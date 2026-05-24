# Playbook: Add a product

Reusable script for generating the article "How to add products to the Catalog". When this playbook runs, it walks through the steps in the sandbox, captures screenshots, and produces the article draft.

## Metadata

- **Helpdesk category**: `catalog`
- **Slug**: `how-to-add-products`
- **Persona**: novice user
- **Type**: step-by-step
- **Publishing flow**: draft → visual review → publish manually

## Code recon — where to look

Before the walkthrough, search for (results go into `recon.md`):

1. **Endpoint**: `grep -rn "products.*post\|router.*products" $BACKEND_REPO/src`
2. **Input schema**: search for `productSchema`, `ProductInput`, `CreateProductDto`
3. **Image validation**: search for `image`, `upload`, `multipart`, MB limits
4. **Error codes**: `grep -rn "PRODUCT_\|throw new.*Product" $BACKEND_REPO/src`
5. **Per-plan quotas**: `$BACKEND_REPO/packages/plans` or similar

Extract for the recon: required name (min/max chars), SKU (regex), price, image format (extensions and size), max number of images, per-plan limits.

## Sandbox walkthrough

Precondition: logged in as the test account, on the seed test store's dashboard, with the seed catalog present.

### Step 1 — Catalog home screen

- Navigate to `/dashboard/products`
- Capture a full-page screenshot (`step-01-catalog.webp`)
- Annotation: red border on the **New product** button (top-right corner)
  - Use `browser_highlight` before the screenshot
- In the article: short intro on when to create a product

### Step 2 — Open the form

- Click **New product**
- Wait for the form to open (3s timeout)
- Capture screenshot (`step-02-empty-form.webp`)
- Annotation: none (an empty form is self-explanatory)

### Step 3 — Required fields

- Fill in Name: `Sample Product A`
- Fill in SKU: `SAMPLE-001-TEST`
- Fill in Price: `89.00`
- Capture screenshot (`step-03-form-filled.webp`)
- Annotation: **numbered circles** on each field in order (1=Name, 2=SKU, 3=Price)
  - Positions come from each input's `getBoundingClientRect`

### Step 4 — Upload images

- Click **Add images**
- Upload 3 fictional images (from the seed in `fixtures/`)
- Capture screenshot of the filled gallery (`step-04-images.webp`)
- Annotation: red border around the gallery
- **Important**: cite from the recon the accepted formats and max size here

### Step 5 — Save

- Click **Save product**
- Wait for the success toast (5s timeout)
- Capture screenshot of the toast (`step-05-toast.webp`)
- Annotation: **cropped zoom-in** on the success toast
  - approximate bbox: bottom-right corner, padding 40

### Step 6 — Confirmation in the Catalog

- Wait for the redirect back to the catalog
- Capture screenshot of the newly created product in the list (`step-06-catalog-with-product.webp`)
- Annotation: red border on the new product's row

## Cover

Use the screenshot from step 1 (full catalog), cropped to 1280x640.

## Article structure

Follow the `step-by-step` template from `../references/style-guide.md`:

```
# How to add products to the Catalog

[1-2 sentences about the goal]

## Before you start

- Your store must be connected to the app
- Have your product images ready (formats: JPG, PNG, WEBP — max X MB)
  <!-- src: recon.md, "Limits" section -->

## Step by step

### 1. Open the Catalog
[text] → [screenshot step-01]

### 2. Click "New product"
[text] → [screenshot step-02]

### 3. Fill in the required fields
[text explaining Name, SKU, Price]
[screenshot step-03 with numbering]

> **About the SKU:** [friendly explanation of the regex]
> <!-- src: packages/schemas/src/product.ts:11 — regex ^[A-Z0-9-]+$ -->

### 4. Add images
[text explaining image limits]
[screenshot step-04]

> **Image limits** <!-- src: recon.md -->
> - Formats: JPG, PNG, WEBP (HEIC from iPhone is converted automatically)
> - Max size: 5 MB per image
> - Up to 10 images per product

### 5. Save
[text] → [screenshot step-05]

### 6. Done!
Your product appears in the Catalog:
[screenshot step-06]

## Your plan's quotas

<!-- src: recon.md, "Quotas per plan" section -->
| Plan | Products |
|---|---|
| Free | 50 |
| Starter | 500 |
| Growth | 5,000 |
| Pro | Unlimited |

## See also
- [Accepted image formats](/catalog/accepted-image-formats)
- [Bulk-import products](/catalog/bulk-import)

## Didn't work?
- "SKU already registered" → [link]
- "Image too large" → [link]
```

## Errors to document separately

Each becomes its own article in the `errors` category:

- `PRODUCT_SKU_DUPLICATE` → article "SKU already registered"
- `PRODUCT_IMAGE_TOO_LARGE` → article "Image exceeds maximum size"
- `QUOTA_EXCEEDED` → article "Product limit reached"

Each of those has its own playbook.
