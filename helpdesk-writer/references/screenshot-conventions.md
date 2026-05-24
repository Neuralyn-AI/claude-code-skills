# Screenshot Conventions

Standards to keep screenshots consistent across the whole helpdesk. Consistency is what makes the helpdesk look professional.

## Base Playwright config

Before any screenshot, configure the browser via Playwright MCP:

- **Viewport**: match the user's actual visible browser area with the window **maximized** — the goal is that the screenshot shows only what the user can currently see, not content scrolled below the fold. Ask the user for their screen dimensions if you don't know them (common monitors: 1920×1080, 2560×1440, 3840×2160) and store them in `./projects/<project_slug>/.helpdesk.env` as `SCREEN_WIDTH` / `SCREEN_HEIGHT`. **Always use `fullPage: false`** — capturing the full scrollable page produces oversized images and shows content the user wasn't looking at.
- **Device scale factor**: 2 (high-resolution screenshots for retina displays)
- **Locale**: `en-US` (or the article's target language)
- **Timezone**: `UTC` (or the project's default)
- **Color scheme**: `light` (light mode, the default most users see)

If the project ships an embedded widget that runs on a customer site, also capture at:

- **Mobile**: 390 x 844 (standard iPhone), DSF 3
- **Desktop**: 1280 x 800, DSF 2

## File format

Save all screenshots — raw captures, processed assets, thumbnails, covers — as **WebP** (`.webp`). At quality 90 it's visually indistinguishable from PNG for UI screenshots, at roughly 1/10 the file size. Smaller files mean faster reads, lighter draft folders, and less bandwidth at publish time.

**Playwright doesn't output WebP natively.** Two-step capture pattern:

1. Capture with Playwright to a temporary PNG (e.g. `raw/_tmp.png`).
2. Convert + clean up:

   ```bash
   python scripts/annotate.py convert --in raw/_tmp.png --out raw/step-NN.webp
   rm raw/_tmp.png
   ```

All other `annotate.py` subcommands (`number`, `arrow`, `box`, `crop`, `blur`, `composite`, `thumb`) write WebP automatically when the `--out` path ends in `.webp`.

## Before any screenshot

Always wait for the page to fully render before capturing. Without this, async-loaded panels (ambient images, lazy-rendered cards, deferred fetches) come out blank in the screenshot.

```js
await page.waitForLoadState('networkidle');
await page.waitForTimeout(5000);
window.scrollTo(0, 0);
```

The extra 5-second wait after `networkidle` is intentional — `networkidle` alone is not enough on dashboards that defer renders after the initial fetch.

## Thumbnails for agent inspection

**Every time the skill captures or saves a screenshot, generate a thumbnail.** When the agent later needs to inspect a screenshot — to plan the next step, verify an element is on screen, or confirm what was captured — it must read the **thumb**, not the raw or processed asset. Full-resolution screenshots burn token budget; thumbnails preserve enough detail for confirmation at a fraction of the cost.

**Spec:**

- **Max width:** 700 px (height scales proportionally)
- **Format:** PNG
- **Location:** `./projects/<project_slug>/drafts/<slug>/thumbs/<same-name>.webp`
- **Naming:** identical to the source file in `raw/` or `assets/`

**When to (re)generate:**

- Immediately after a new `raw/` capture
- Immediately after writing a processed image to `assets/` — overwrite the thumb so it reflects the latest annotated version

**Command:**

```bash
python scripts/annotate.py thumb \
  --in raw/step-01.webp \
  --out thumbs/step-01.webp
```

**Article output references `assets/`, not `thumbs/`.** The thumb is purely an agent-inspection optimization; the published article links to the full-resolution annotated images in `assets/`.

## Naming

Files in `./projects/<project_slug>/drafts/<slug>/raw/` and `./projects/<project_slug>/drafts/<slug>/assets/`:

```
step-NN-short-description.webp

Examples:
step-01-catalog.webp
step-02-new-product.webp
step-03-form-filled.webp
```

Use 2-digit numbering (`01`, `02`...) so the files sort correctly.

The **annotated** version goes in `assets/` with the same name as the one in `raw/`. Never overwrite the raw — you may need to reprocess with a different annotation.

## Annotation types and when to use each

### A. Red border on the element (Playwright's native highlight)

**When to use:** giving context with the full screen and showing where the element is. It's the most subtle annotation — good for the first screenshot of each step.

**How to apply:** before capturing, call `browser_highlight` from Playwright MCP on the target element. The highlight injects CSS into the page (`border: 3px solid #E53935; border-radius: 5px;`) that shows up in the screenshot.

After capturing, call `browser_remove_highlight` to clean up.

**Example:**
> "To start, click **Register product** in the top-right corner of the **Catalog**:"
> ![catalog screen with the Register product button highlighted](assets/step-01-catalog.webp)

### B. Numbered circles + arrows

**When to use:** steps with multiple sub-elements in sequence (e.g. filling 3 form fields in order). Shows the order visually.

**How to apply:** capture the clean screenshot, then call `scripts/annotate.py`:

```bash
python scripts/annotate.py number \
  --in raw/step-02-form.webp \
  --out assets/step-02-form.webp \
  --xy 250,180 --n 1

# Add more numbers to the same file, overwriting:
python scripts/annotate.py number \
  --in assets/step-02-form.webp \
  --out assets/step-02-form.webp \
  --xy 250,280 --n 2

python scripts/annotate.py number \
  --in assets/step-02-form.webp \
  --out assets/step-02-form.webp \
  --xy 250,380 --n 3
```

Coordinates come from the element's bounding box (`browser_snapshot` returns this, or run `getBoundingClientRect` via `browser_run_code_unsafe`).

Place the circle **slightly overlapping** the top-left corner of the element, not on top of the text or icon.

### C. Cropped element capture

**When to use:** any time the screenshot focuses on a specific element (a button, a toggle, a form field, a card, a small detail) instead of the full screen. Common cases: zoom-ins on toggles, status icons, menu badges, numeric values; isolated form fields; contextual highlights of a single button.

**Rules that always apply:**

- **20 px padding on every side.** Compute from `getBoundingClientRect()`. No padding = the crop hugs the element's edge and looks amateur.
- **Centered horizontally** in the article when the helpdesk renders HTML.
- **`max-width: 70%`** so the cropped image doesn't dominate the article column.
- **Link to a full-page screenshot** of the same view, opened in a new tab. A reader who needs context for where the element sits in the UI can click through.

**Two captures per element:**

1. **Cropped + padded** → inline image
2. **Viewport overview** (`fullPage: false`, full visible browser area) → link target

The link target is *not* a `fullPage: true` capture — that would include content scrolled below the fold and produce an oversized image. The reader just needs to see where the element sits within the currently-visible screen.

**Two ways to crop the element capture:**

- Direct cropped capture via Playwright `clip` (preferred when you know the bbox at capture time):

  ```js
  const rect = await elementHandle.boundingBox();
  await page.screenshot({
    clip: {
      x: rect.x - 20, y: rect.y - 20,
      width: rect.width + 40, height: rect.height + 40,
    },
    path: 'raw/step-04-toggle.webp',
  });
  ```

- Post-process a full screenshot with `scripts/annotate.py crop` (use when you already have the raw shot):

  ```bash
  python scripts/annotate.py crop \
    --in raw/step-04-toggle.webp \
    --out assets/step-04-toggle.webp \
    --bbox 1080,420,1250,490 \
    --padding 20
  ```

**HTML embedding pattern (when the helpdesk renders inline HTML):**

```html
<p style="text-align: center;">
  <a href="{full_page_url}" target="_blank">
    <img src="{cropped_url}" alt="..." style="max-width: 70%" />
  </a>
</p>
```

**Markdown-only helpdesks:** drop the centering and `max-width` rules (the platform handles layout). Keep the padding rule, and add a plain "View full page" link below the image:

```markdown
![alt](cropped.webp)

[View full page](full-page.webp)
```

If you want to highlight inside the crop, chain with `box`:

```bash
python scripts/annotate.py box \
  --in assets/step-04-toggle.webp \
  --out assets/step-04-toggle.webp \
  --bbox 80,30,180,90
```

### D. Mix: border + number

For complex steps that need to both highlight the element AND show order. Combine A + B on the same screenshot:

1. `browser_highlight` on the main element → screenshot
2. `annotate.py number` on each sub-element

### E. Blur (masking data)

**ALWAYS use when:**

- Emails that look real (even in the sandbox)
- Tax IDs, phone numbers, addresses
- End-customer names (buyers)
- Tokens, long IDs, API keys
- Real billing amounts

How to apply:

```bash
python scripts/annotate.py blur \
  --in raw/step-05-profile.webp \
  --out assets/step-05-profile.webp \
  --bbox 150,300,500,330
```

**When NOT to blur:** fictional sample names created in the sandbox (e.g. `Test Product A`, `Demo Item B`). Those stay — they add realism.

### F. Side-by-side composition (before/after)

For showing transformations: initial state vs. final state. Useful for conceptual articles ("what the app does to your input").

```bash
python scripts/annotate.py composite \
  --in raw/before.webp,raw/after.webp \
  --out assets/comparison.webp \
  --labels "Before,After"
```

## Article cover

Every article has a cover: a screenshot of the feature's main screen, **with no annotations**, at 1280 x 640. Crop to that aspect ratio with `crop`:

```bash
python scripts/annotate.py crop \
  --in raw/cover-raw.webp \
  --out assets/cover.webp \
  --bbox 0,80,1280,720
```

File name: always `cover.webp`.

## Test data — what shows up in the screenshots

In the sandbox, populate with fictional but realistic-sounding data. Pick names and stick with them so screenshots across different articles feel coherent.

Guidelines:

- Use a single fictional brand / company name across all articles
- Reuse the same sample records (products, customers, orders, etc.) from article to article
- Keep the data stable — if you change a seed name later, old screenshots will look inconsistent next to new ones

## Reproducibility

If an article needs to be updated later (UI changed, screenshot got stale), the playbook script should be **re-runnable** and produce equivalent screenshots. So:

- Use semantic selectors (`getByRole`, `getByLabel`), not brittle XPath
- Manage data state via reproducible seeds
- Always the same viewport and DSF
