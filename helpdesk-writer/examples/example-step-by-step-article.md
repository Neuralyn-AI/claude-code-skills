# How to add products to the Catalog

Adding products is the first step to start managing your storefront in the app. You can do it in under 2 minutes.

## Before you start

- Your store must be connected to the app (if it's not yet, see [Installing the widget](/widget/install)).
- Have your product images ready. <!-- src: recon.md "Image limits" -->
  - Accepted formats: **JPG, PNG, WEBP**. HEIC images (from iPhone) are converted automatically.
  - Max size: **5 MB** per image.
  - Up to **10 images** per product.

## Step by step

### 1. Open the Catalog

In the sidebar of the dashboard, click **Catalog**:

![Dashboard home with the Catalog item highlighted in the sidebar](assets/step-01-catalog.webp)

### 2. Click **New product**

The button sits in the top-right corner of the screen:

![New product button in the top-right corner](assets/step-02-new-product.webp)

### 3. Fill in the required fields

You need to fill in three fields to create the product:

![Form with numbered fields](assets/step-03-form.webp)

**1. Name** — how the product appears to the buyer. Between 3 and 120 characters. <!-- src: packages/schemas/src/product.ts:8 -->

**2. SKU** — the product's unique code in your store. Uppercase letters, numbers, and hyphens only (e.g. `SHIRT-BLUE-001`). <!-- src: packages/schemas/src/product.ts:11 — regex ^[A-Z0-9-]+$ -->

**3. Price** — in dollars. Use a period for cents (e.g. `89.00`). <!-- src: apps/api/src/routes/products.ts:67 — price_cents conversion -->

> **About the SKU:** if you already use product codes in your store, reuse them. It makes connecting the Catalog with your inventory easier later. If you don't have an SKU scheme yet, a good convention is **category-model-number** (`DRESS-LUM-002` for the Lumen Dress, model 2).

### 4. Add images

Click **Add images** and select the product photos:

![Upload area with 3 images loaded](assets/step-04-images.webp)

The first image is the **main one** — it's the one that appears as the product's primary image in your storefront. The others become variations the buyer can browse.

### 5. Save

Click **Save product** at the bottom of the form. You'll see a confirmation in the corner of the screen:

![Success toast "Product registered"](assets/step-05-toast.webp)

### 6. Done!

Your product now appears in the Catalog and is available in your storefront:

![Catalog list with the newly registered product highlighted](assets/step-06-confirmation.webp)

## Your plan's quotas

Each plan has a limit on how many products you can keep in the Catalog: <!-- src: recon.md "Quotas per plan" -->

| Plan | Product limit |
|---|---|
| Free | 50 |
| Starter | 500 |
| Growth | 5,000 |
| Pro | Unlimited |

When you get close to the limit, a warning shows up in the Catalog. You can upgrade your plan any time at **Settings → Plan**.

## See also

- [Accepted image formats](/catalog/accepted-image-formats)
- [Import many products at once](/catalog/bulk-import)
- [Edit or remove a product](/catalog/edit-product)

## Didn't work?

- **"SKU already registered"** — you already have a product with that same code. [What to do](/errors/sku-duplicate).
- **"Image too large"** — one of the images is over 5 MB. [How to reduce](/errors/image-too-large).
- **"Plan limit reached"** — you've hit the product quota. [See your options](/errors/quota-exceeded).
