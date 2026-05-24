# Code Recon — extracting truth from the code

This is the most important step of the skill, and the one that separates a good article from a generic one. Before writing, find out what the code actually does.

## Principle

**No technical claim makes it into the article without having been found in the code.** Don't write "the limit is 100 products" unless you've seen a constant `MAX_PRODUCTS = 100` in the repo. Instead, find it and cite it.

If you can't find the evidence, two paths:

1. Report it to the user ("I couldn't find that constant — can you point me to where it lives?")
2. Drop the claim from the article

Never guess.

## Where things live (map this at the start of the project)

When the skill is set up for a new project, ask the user to fill in `.helpdesk.env` with the repo paths:

```bash
BACKEND_REPO=/home/path/code/backend
FRONTEND_REPO=/home/path/code/dashboard
```

The skill assumes the patterns below. If the monorepo has a different layout, update this section.

### Backend

| What to look for | Search pattern | Why |
|---|---|---|
| Input validation schemas | `z.object`, `zod` | Defines the exact shape of accepted data: types, sizes, required fields |
| Limit constants | `MAX_`, `LIMIT_`, `_LIMIT`, `const.*=.*\d+` | Hardcoded numeric limits (quotas, sizes) |
| Error codes | `throw new`, `HTTPException`, `ApiError`, `ErrorCode` | Exhaustive list of errors the backend can emit |
| Error messages | `message:`, `errorCode:`, files like `errors.ts` or `messages.ts` | Literal text the frontend shows |
| Defaults | `??`, `defaultValue`, `default:` | Values that kick in when a field is omitted |
| Shared types | `types/`, `schemas/`, `shared/` | API contracts |
| Routing | `app.get(`, `app.post(`, `route(`, `Hono` | Maps endpoint → handler |
| Quotas / billing | terms like `quota`, `usage`, `plan`, `tier` | Per-plan limits (free/starter/growth/pro) |

### Frontend

| What to look for | Pattern | Why |
|---|---|---|
| UI strings / i18n | locale files like `en-US.json`, `translations`, `t(` | Literal text the user sees — reuse it verbatim in the article |
| Client-side validators | `react-hook-form`, `zod`, `validate` | May be more restrictive than the backend |
| Rendered error messages | grep for the backend's `errorCode` | How the error actually appears visually |
| Feature flags | `flag`, `enabled`, `experimental` | Tells you whether the feature is on for all plans |
| Error components | `Error`, `Toast`, `Alert` | How the error is presented |

## Search strategy

Don't try to read the whole repo. Do targeted searches:

1. **Start from the feature's name in the UI.** If the article is about "register product", `grep -rin "register product\|create product\|new product"` in the frontend leads you to the component. From the component, find which endpoint it calls. From the endpoint, jump to the handler in the backend.

2. **Use Glob + Grep aggressively.** Instead of reading whole files, search precisely:

   ```bash
   # Find the endpoint's handler
   grep -rn "products.*post\|POST.*products" $BACKEND_REPO/src

   # Find the validation schema
   grep -rn "productSchema\|ProductInput" $BACKEND_REPO/src

   # Find every error this handler can throw
   grep -rn "throw\|ApiError" $BACKEND_REPO/src/routes/products/
   ```

3. **Follow imports.** Found the schema? Read it whole. See what it imports (sub-schemas, validation regex, enums). Found the constant? Check where it's used.

4. **Check shared types.** In a monorepo, frontend and backend often share types. Verify whether what you're documenting is the shared type or a transformation.

## How to record the evidence

After the recon, create `./projects/<project_slug>/drafts/<slug>/recon.md` with this format:

```markdown
# Recon: <article slug>

## Main endpoint
- POST /api/products
  src: `my-backend/apps/api/src/routes/products.ts:34`

## Input schema
- `name`: string, 3-120 chars, required
  src: `my-backend/packages/schemas/src/product.ts:8`
- `sku`: string, regex `^[A-Z0-9-]+$`, 1-50 chars, required
  src: `packages/schemas/src/product.ts:11`
- `price_cents`: integer ≥ 0, required
  src: `packages/schemas/src/product.ts:14`
- `images`: array of URLs, max 10 items
  src: `packages/schemas/src/product.ts:18`

## Limits
- Maximum products per batch upload: 100
  src: `apps/api/src/routes/products.ts:42` (const `BATCH_LIMIT`)
- Images per product: 10
  src: `packages/schemas/src/product.ts:18`
- Max image size: 5 MB (enforced in the Worker)
  src: `apps/api/src/middleware/upload.ts:23`

## Possible errors
| Code | When it happens | Message (localized) |
|---|---|---|
| `PRODUCT_SKU_DUPLICATE` | SKU already exists for the tenant | "This SKU is already registered..." (src: `frontend/locales/en-US.json:142`) |
| `PRODUCT_IMAGE_TOO_LARGE` | Image > 5 MB | "The image exceeds the limit..." |
| `QUOTA_EXCEEDED` | Tenant hit the plan's limit | depends on the plan (see `plans.ts:18`) |

## Quotas per plan
- Free: 50 products
- Starter: 500 products
- Growth: 5,000 products
- Pro: unlimited
  src: `my-backend/packages/plans/src/limits.ts:8-22`

## Conditional behavior
- If the image is HEIC, it's converted to JPEG before storage
  src: `apps/api/src/services/image-processor.ts:55`
- If `price_cents` is omitted, the default is 0 but the product doesn't show up on the storefront
  src: `apps/api/src/services/product-visibility.ts:12`
```

This `recon.md` is the source of every technical claim in the article. Each `<!-- src: ... -->` in the draft points to a line in this recon.

## When the code contradicts the UI

Common case. Real examples:

- The code says it accepts 100 products, the UI says "max. 50" → double validation, and the UI is stricter for a good reason (or because of a bug). Stop and ask.
- The code emits an `INVALID_FORMAT` error, but the frontend shows "Unknown error" → missing translation. Document the message the user **will see** (not the one the code emits), and make a mental note to report it to the team.
- The UI shows a button with no matching route in the backend → feature flag, mocked, or dead feature.

In every case: **report to the user before writing**. Don't pick a side on your own.

## What NOT to document

Some things you'll find in the code that **don't belong in the article**:

- Internal endpoints (admin, debug)
- Validations that exist but are about to be removed (TODO, deprecated)
- Implementation details that don't affect the user (which queue it uses, which cache, which AI provider)
- Internal error codes that never reach the frontend
- Test constants (`MAX_PRODUCTS_DEV = 999999`)

The rule: **does the end user need to know this to use the product?** If not, leave it out.
