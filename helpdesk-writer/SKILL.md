---
name: helpdesk-writer
description: Writes user-facing helpdesk articles for an app and publishes them to a Helpdesk System through an API or MCP server. Use when the user asks to create, update, or draft helpdesk documentation, a tutorial, a how-to, a step-by-step guide, or an article explaining a feature, error message, format, limit, or workflow. The skill cross-references two sources of truth (the backend/frontend codebase and the live UI via Playwright MCP), captures annotated screenshots, and publishes to a MCP/API chosen by the user, only after the user reviews the draft. Articles are written in EN-US by default, or a language informed by the user.
---

# Helpdesk Writer

Drafts helpdesk articles backed by technically verified information: cross-references the product's code and the live UI in a sandbox, then applies the voice guide before writing a single line. Publishes as a draft through the MCP/API supplied by the user.

## When to activate

Requests to write, update, or draft:

- A tutorial / step-by-step / how-to for the app
- An explanation of a feature, flow, or product concept
- A troubleshooting article / error explanation
- A reference for a data format, limit, or requirement
- An FAQ or conceptual article

## Environment prerequisites

- **Playwright MCP** installed: `claude mcp add playwright npx @playwright/mcp@latest`
- **A helpdesk MCP or API.** Ask the user whether they're using an MCP server or a direct API. If MCP, it should already be set up. If API, ask the user for the URL and credentials.
- **Project name.** Ask the user for a name for the app/project.
- **Project slug folder.** Create a folder `./projects/<project_slug>/`. All files related to the project go there (drafts, credentials, etc.).
- **Repos cloned locally** (frontend + backend monorepo) — paths in `./projects/<project_slug>/.helpdesk.env`
- **Sandbox credentials** in `.helpdesk.env` (never hardcoded)
- **Python 3 with Pillow**: `pip install Pillow`

## Master workflow

For any article request, run these steps in order. **Do not skip steps.** The hard rule is: **no technical claim makes it into the article without traceable evidence — from the code or the UI.**

### 1. Briefing

Ask the user, in a single round:

- The article's topic and helpdesk category
- The target persona (e.g. novice user, power user, developer integrating)
- Whether it's a new article or an update (if update, the ID/slug of the existing article)

If the article type is obvious from the request, infer it instead of asking.

### 2. Code recon (read `references/code-recon.md`)

Before touching the UI, do recon on the code:

- Identify the relevant files (handlers, schemas, validators, error maps, constants)
- Extract: accepted formats, numeric limits, exhaustive list of possible errors, defaults, conditional flows
- Save your findings to `./projects/<project_slug>/drafts/<slug>/recon.md` with `path:line` for each piece of evidence

If the feature has no corresponding code, **warn the user** and do not invent.

#### 2.1 API recon

- Try to connect to the helpdesk MCP or API supplied by the user.
- Identify the endpoints and fields of that API or MCP.
- Save your findings to `./projects/<project_slug>/drafts/<slug>/api-or-mcp-recon.md`.
- Highlight which fields are required.
- Check how image upload works (upload first and reference URLs, or send base64 inline with the article).

### 3. UI walkthrough (read `references/test-environment.md` and `references/screenshot-conventions.md`)

Log into the sandbox via Playwright MCP using the credentials from the env file. For each step of the script:

- Navigate to the screen
- Capture the screenshot (full page or element, per convention)
- For native annotations: use `browser_highlight` before the screenshot when applicable
- Capture with Playwright to a temporary PNG (e.g. `raw/_tmp.png`), then convert to WebP and clean up: `python scripts/annotate.py convert --in raw/_tmp.png --out raw/step-NN.webp && rm raw/_tmp.png`. Playwright doesn't output WebP natively, so this two-step keeps the canonical `raw/` file in the project format.
- **Immediately generate a thumbnail** to `./projects/<project_slug>/drafts/<slug>/thumbs/step-NN.webp` (`python scripts/annotate.py thumb --in raw/step-NN.webp --out thumbs/step-NN.webp`). Whenever you need to inspect a screenshot you took, always read the thumb — never the raw or assets version — to save tokens.

If the UI contradicts the code (e.g. the code says it accepts up to 100 products but the UI says "max. 50"), **stop and report it to the user**. It could be a bug, double validation, or stale docs.

### 4. Image post-processing (read `references/screenshot-conventions.md`)

Call `scripts/annotate.py` to apply the right annotation to each step:

- **Red border** (subtle overlay via native highlight): overview of the screen with the target element highlighted (add 5px extra space to each side of the overlay).
- **Numbered circles + arrows**: when the step has multiple sub-elements in sequence.
- **Cropped element capture**: focus on a specific element (button, toggle, form field, badge) — always padded, centered, and linked to a full-page version. See `references/screenshot-conventions.md` for the full pattern.

Save the processed images to `./projects/<project_slug>/drafts/<slug>/assets/step-NN.webp`, and **regenerate the matching thumbnail** in `./projects/<project_slug>/drafts/<slug>/thumbs/step-NN.webp` so the thumb reflects the annotated version.

### 5. Writing the article (read `references/style-guide.md`)

Compose the markdown following the standard structure for the article type. Inviolable rules:

- Every technical claim needs an HTML comment with its source: `<!-- src: backend/api/products.ts:42 — limit 100 -->`
- Every screenshot has text before it (introducing what the reader will see) and text after it (what to do next)
- Errors are explained as **what happened, why it happened, what to do** — in that order

### 6. Human review

Show the draft in the chat (rendered markdown + list of generated screenshots). **Never publish directly.** Wait for explicit approval.

If the user asks for changes, revise and show it again.

### 7. Publishing (read `references/publishing-rules.md`)

Based on the article type, choose the mode (draft vs. publish directly). Before upload, strip **all HTML evidence comments** from the final markdown. They're for audit, not for the reader.

Call the MCP or API with the fields it requires — typically title, slug, category, body (HTML or clean markdown; check the API and the user), meta description, image attachments, etc.

## Reference files

| File | When to read |
|---|---|
| `references/code-recon.md` | Before step 2 |
| `references/test-environment.md` | Before step 3 |
| `references/screenshot-conventions.md` | Steps 3 and 4 |
| `references/style-guide.md` | Step 5 |
| `references/publishing-rules.md` | Step 7 |
| `api-or-mcp-recon.md` (per-project draft) | Step 7 |
| `playbooks/*.md` | When the user asks for an article whose topic already has a playbook |
| `examples/*.md` | For reference on the final format |

## Scripts

- `scripts/annotate.py` — image annotations and format conversion (number, arrow, box, crop, blur, composite, thumb, convert). Output format follows the `--out` file extension (`.webp` for screenshots). Run `python scripts/annotate.py --help`.

## Drafts structure

```
./projects/<project_slug>/drafts/<slug>/
├── recon.md              # code evidence (step 2)
├── outline.md            # script with steps (steps 1+2)
├── api-or-mcp-recon.md   # how to use the helpdesk API/MCP
├── raw/                  # raw screenshots from Playwright
├── assets/               # annotated screenshots (final, referenced by the article)
├── thumbs/               # 700px-wide thumbnails — read these when the agent needs to inspect
└── article.md            # article markdown (with evidence comments)
```

Keep the `./projects/<project_slug>/drafts/<slug>/` folder around even after publishing — it's the audit trail and the starting point for future updates.

## Principles

1. **Code truth > UI truth > impression.** In that order.
2. **Every technical claim has a trace** (`path:line`) in the draft.
3. **Never publish without human review.** Even prior approval doesn't carry over to a new article.
4. **If the code and UI contradict, stop and report.** Don't pick a side on your own.
5. **Write for the end user, not the developer.** The final article must not read like a GitHub issue.
