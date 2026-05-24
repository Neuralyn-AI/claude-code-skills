# Publishing Rules

When to publish directly vs. when to leave as a draft. The rule of thumb: the more sensitive the content, the stricter the review.

## Decision matrix

| Article type | Default flow | Why |
|---|---|---|
| **New step-by-step / how-to** | Draft → visual review in the helpdesk → publish manually | Screenshots need human eyes. A wrong image generates support tickets. |
| **Troubleshooting / error explanation** | Always draft | Badly written error text makes the experience worse. Always review first. |
| **Conceptual / "what is" / FAQ** | Publish directly after chat approval | Low risk, no critical screenshots. |
| **Update to existing article** | Draft with a diff vs. the current version | You need to compare before overwriting. |
| **Format / limit reference** | Draft | A wrong number (wrong limit) creates the wrong expectation. |
| **Article with billing/plan data** | Draft with mandatory approval | Commercial commitment — one wrong word becomes a problem. |

"Draft" = `status: draft` or the equivalent on your helpdesk platform. Not visible to the public until it becomes `published`.

## Before any publication (draft or direct)

Mandatory checks:

1. **All HTML evidence comments have been removed.** Grep the final markdown: `grep -n '<!--' article.md` should return zero lines.
2. **All screenshots have been processed** (in `assets/`, not `raw/`).
3. **Sensitive data has been blurred** (do this visually: open each asset, scan it).
4. **No forgotten placeholders**: `grep -in 'TODO\|FIXME\|XXX\|\[\.\.\.\]'`.

If any of these fail, go back and fix it. Only then publish.

## Image upload approach

Some helpdesk APIs/MCPs accept base64 images inline with the article body. Others require you to upload images first to a CDN and reference URLs in the body. Pick the right flow for the target platform — and **document the decision in this file the first time you run the skill against it**, so future runs don't have to re-discover it.

## After publishing (or creating a draft)

Return to the user in the chat:

- Final status (`draft` or `published`)
- Edit URL in the helpdesk dashboard (if available)
- Public URL (if published)
- List of attached images (with local paths for future reference)
- A summary of **what was verified in the code** (for confidence)

Example response:

```
✓ Article created as a draft in the helpdesk MCP/API.

Title: How to Add Products
Slug: how-to-add-products
Edit URL: https://app.helpdesk.com/portals/my-helpdesk/categories/.../articles/.../edit
Images: 6 attached from assets/

Next step: review visually in the helpdesk and change the status to Published.
```

## Automatic updates (future)

Don't implement now, but worth planning: the skill could run in "verification" mode periodically — re-run the article's playbook, compare old vs. new screenshots via pixelmatch, and open a GitHub issue when the UI has drifted enough to invalidate the article. Leave that for v2.
