# Judge.me Installation Guide for Gravity Designs (Sofine Theme)

**Date:** 2026-03-09
**Store:** gravity-designs.com (d20e1f.myshopify.com)
**Theme:** Sofine by Alothemes (legacy, non-OS 2.0)

---

## 1. Overview

Judge.me is a product reviews app for Shopify. It collects customer reviews, displays star ratings, and injects structured data (JSON-LD) for Google rich snippets. It works with any Shopify theme, including legacy themes like Sofine.

The store currently has dead code from the discontinued Shopify Product Reviews (SPR) app. This guide covers installing Judge.me and cleaning up that legacy code.

---

## 2. Why Judge.me

| Criteria | Judge.me (Free Plan) |
|----------|---------------------|
| Cost | $0/month (Forever Free plan) |
| SEO rich snippets | Yes — auto-injects `aggregateRating` and `Review` schema |
| Review request emails | Yes — automatic post-purchase emails |
| Widget weight | ~30 KB async loaded, minimal impact |
| Photo/video reviews | Yes (paid plan only for video) |
| Import from SPR | Yes — built-in CSV importer |
| Theme compatibility | Works with all themes, including legacy Liquid |

For a small store with few products, the free plan covers everything needed.

---

## 3. Installation Steps

### 3.1 Install from Shopify App Store

1. Go to https://apps.shopify.com/judgeme
2. Click **Add app**
3. You will be redirected to your Shopify admin. Click **Install app**
4. On the Judge.me setup screen, select **Forever Free** plan
5. Complete the onboarding wizard:
   - Store name and logo (auto-detected)
   - Email sender name
   - Review request timing (recommended: 14 days after fulfillment)

### 3.2 Automatic Theme Installation

After installing the app:

1. Go to **Judge.me > Settings > Theme Installation**
2. Click **Install Judge.me widgets on your theme**
3. Select your active Sofine theme
4. Click **Install**

Judge.me will automatically inject its script tag into `theme.liquid` and add widget snippets. This works for most themes, including Sofine.

### 3.3 Verify the Script Tag

After auto-install, check that this line was added to `layout/theme.liquid`, typically before `</head>`:

```liquid
<!-- Judge.me script is injected automatically via the app embed -->
```

If using Online Store 2.0 app embeds (Sofine may partially support this):

1. Go to **Online Store > Themes > Customize**
2. Click **App embeds** (left sidebar, puzzle icon)
3. Toggle **Judge.me** to ON
4. Save

---

## 4. Theme Integration — Widget Placement

Judge.me provides two types of widgets:

- **Review Widget** — full review list with form (goes on product page)
- **Preview Badge** — star rating + count (goes anywhere: product cards, product page header)

### 4.1 Automatic Widgets

After installation, Judge.me typically auto-places:
- The **Review Widget** below the product form on product pages
- The **Preview Badge** near the product title

### 4.2 Manual Liquid Placement (if auto-install misses something)

#### Preview Badge (star rating)

Place this wherever you want to show stars + review count:

```liquid
<div class="jdgm-widget jdgm-preview-badge"
     data-id="{{ product.id }}"
     data-auto-load="false">
</div>
```

**Common locations in this theme:**
- `sections/main-product.liquid` — near the product title (around line 66-70, replacing the old SPR badge)
- `snippets/product-list-item.liquid` — inside product cards on collection pages
- `snippets/product-grid-item-masonry.liquid` — masonry grid product cards

#### Review Widget (full reviews section)

Place this where you want the full review list + submission form:

```liquid
<div class="jdgm-widget jdgm-review-widget"
     data-id="{{ product.id }}">
</div>
```

**Best location in this theme:** Inside `sections/product-page-tab.liquid`, replacing the old SPR reviews tab content (line 82-90).

### 4.3 Replacing the SPR Reviews Tab

In `sections/product-page-tab.liquid`, the `tab_reviews` block (line 81-90) currently renders dead SPR code. To make it work with Judge.me:

**Before (dead SPR code):**
```liquid
{%- when 'tab_reviews' -%}
    <div id="tab_review" class="tab-panel">
      <div class="custom-review">
        <div class="row">
          <div id="shopify-product-reviews" class="col-xs-12 col-sm-12 col-md-12 col-lg-12" data-id="{{product.id}}">
            {{ product.metafields.spr.reviews }}
          </div>
        </div>
      </div>
    </div>
```

**After (Judge.me widget):**
```liquid
{%- when 'tab_reviews' -%}
    <div id="tab_review" class="tab-panel">
      <div class="custom-review">
        <div class="row">
          <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
            <div class="jdgm-widget jdgm-review-widget"
                 data-id="{{ product.id }}">
            </div>
          </div>
        </div>
      </div>
    </div>
```

---

## 5. Recommended Widget Placement

| Location | Widget | File | Notes |
|----------|--------|------|-------|
| Product page — near title | Preview Badge | `sections/main-product.liquid` (line ~67) | Replace SPR badge div |
| Product page — tab section | Review Widget | `sections/product-page-tab.liquid` (line ~82) | Replace SPR tab content |
| Collection product cards | Preview Badge | `snippets/product-list-item.liquid` | Add after product title/price |
| Homepage featured products | Preview Badge | Relevant homepage snippet | Add after product title |
| Sticky add-to-cart bar | Preview Badge | `sections/sticky_add_to_cart.liquid` | Optional |

### Enabling the Reviews Tab

The reviews tab block (`tab_reviews`) was removed from `templates/product.json`. To re-enable it:

1. Go to **Online Store > Themes > Customize > Product page**
2. In the **Product Page Tab** section, click **Add block**
3. Select **Reviews** tab
4. Save

Or manually add the block back in `templates/product.json` under the `product-page-tab` section:

```json
{
  "type": "tab_reviews",
  "settings": {
    "title_reviews": "Reviews"
  }
}
```

### Enabling the Star Rating on Product Page

The `show_reviews` block exists in `templates/product.json` but is currently disabled (`"disabled": true`). To enable it:

1. Go to **Online Store > Themes > Customize > Product page**
2. Find the **Reviews** block in the main product section
3. Toggle its visibility ON (click the eye icon)
4. Save

Note: Once Judge.me is installed, it will render its own badge inside the `.shopify-product-reviews-badge` span, so the existing block markup in `main-product.liquid` (line 66-70) should work without changes. Judge.me hooks into elements with `data-id` attributes.

---

## 6. SEO Configuration

### 6.1 Judge.me Structured Data

Judge.me automatically injects `aggregateRating` and individual `Review` entries into the page as JSON-LD. You do NOT need to manually add this to `snippets/product-json-ld.liquid`.

To verify Judge.me handles structured data:

1. Go to **Judge.me > Settings > SEO**
2. Ensure **Rich Snippets** is enabled (it is by default)
3. Judge.me will add a separate `<script type="application/ld+json">` block with review data

### 6.2 Avoiding Duplicate Schema

Your existing `snippets/product-json-ld.liquid` outputs a `Product` schema without `aggregateRating`. Judge.me adds its own `Product` schema with ratings. This can cause duplicate `Product` entities.

**Recommended approach:** Keep your existing `product-json-ld.liquid` as-is. Google can merge multiple JSON-LD blocks for the same product. However, if you see warnings in Google Search Console, you have two options:

**Option A — Let Judge.me handle all Product schema:**
Remove or disable the render call to `product-json-ld` in your theme layout/templates, and let Judge.me's schema be the sole source.

**Option B — Add aggregateRating to your custom schema (advanced):**
Judge.me stores review data in metafields. You can reference them:

```liquid
{%- if product.metafields.judgeme.badge_text != blank -%}
  {%- assign jm_rating = product.metafields.judgeme.badge_text | split: ' out of ' -%}
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "{{ jm_rating[0] }}",
    "bestRating": "5",
    "reviewCount": "{{ product.metafields.judgeme.review_count }}"
  },
{%- endif -%}
```

### 6.3 Verify Rich Snippets

1. Publish a test review (see section 7)
2. Wait a few minutes for Judge.me to update metafields
3. Visit your product page
4. Test with Google Rich Results Test: https://search.google.com/test/rich-results
5. Enter the product URL and check for `Product` with `aggregateRating`
6. Also check with Schema Validator: https://validator.schema.org/

---

## 7. Testing

### 7.1 Start Dev Server

```bash
shopify theme dev --store d20e1f.myshopify.com
```

Preview at http://127.0.0.1:9292

Note: Judge.me widgets may not load on localhost since the app injects scripts based on the live store domain. Test on the live preview URL provided by `shopify theme dev` or on the published theme.

### 7.2 Submit a Test Review

1. Go to a product page on your live store (or preview theme)
2. Scroll to the reviews section
3. Click **Write a Review**
4. Fill in: name, email, star rating (5 stars), title, body
5. Submit
6. Go to **Judge.me dashboard > Reviews** to approve it
7. Refresh the product page — stars and review should appear

### 7.3 Verify Star Badges

- Check the product page for the preview badge (stars near title)
- Check collection pages for star badges on product cards
- Check that the review count in the tab header updates

### 7.4 Validate Structured Data

```bash
# Quick check — view page source and search for aggregateRating
# On the live product page, Ctrl+U, then Ctrl+F for "aggregateRating"
```

Use Google Rich Results Test:
1. Go to https://search.google.com/test/rich-results
2. Enter a product URL that has at least one review
3. Confirm `Product` entity shows `aggregateRating`

### 7.5 Performance Check

1. Run Lighthouse on a product page (Chrome DevTools > Lighthouse)
2. Check that Judge.me scripts load async and don't block rendering
3. Compare Total Blocking Time before and after installation

---

## 8. Performance Considerations

- **Judge.me loads asynchronously** — its main script uses `async` or `defer`, so it does not block page rendering
- **Widget rendering** is lazy — badges only initialize when the DOM element is present
- **Limit widgets per page** — avoid placing more than one Review Widget on a single page
- **Preview badges are lightweight** — safe to use on collection pages with many products
- **If performance degrades:** In Judge.me settings, disable the floating "Write a Review" tab (it adds an extra DOM element on every page)

---

## 9. Maintenance

### 9.1 Review Moderation

- Go to **Judge.me > Reviews** in Shopify admin
- Set moderation to **Manual** (recommended) or **Auto-publish**
- Enable email notifications for new reviews

### 9.2 Spam Prevention

- Judge.me has built-in spam filtering
- Enable CAPTCHA: **Judge.me > Settings > Review Form > Enable reCAPTCHA**
- Set minimum review body length to reduce low-effort spam

### 9.3 Import Existing Reviews

If you have reviews from the old SPR app:

1. Export from SPR (if you still have access): CSV format
2. Go to **Judge.me > Settings > Import**
3. Upload the CSV file
4. Map columns (product handle, reviewer name, email, rating, title, body, date)

### 9.4 Export and Backup

- **Judge.me > Settings > Export** — download all reviews as CSV
- Do this periodically as a backup
- Useful if you ever switch review apps

---

## 10. Troubleshooting

| Problem | Solution |
|---------|----------|
| Stars don't appear on product page | Check that the `show_reviews` block is enabled in product.json (currently `"disabled": true`). Go to Theme Customize and toggle it ON. |
| Review widget not showing in tab | Re-add the `tab_reviews` block to the product-page-tab section (it's currently removed from product.json). |
| Judge.me widgets not loading on localhost | Normal — test on the live preview URL from `shopify theme dev` output instead. |
| Duplicate Product schema in Google | See section 6.2 — either remove your custom `product-json-ld.liquid` or let both coexist (Google handles it). |
| "Write a Review" button does nothing | Check browser console for JS errors. Ensure Judge.me script is loaded. Check for jQuery conflicts (Sofine uses jQuery). |
| Stars show on product page but not on collection cards | Add the Preview Badge widget manually to `snippets/product-list-item.liquid`. |
| Email requests not sending | Check **Judge.me > Settings > Email** — verify sender email and timing. Ensure orders are being fulfilled (emails trigger on fulfillment). |
| Reviews tab shows count of 0 | The old `reviews-count.liquid` snippet reads SPR metafields. Replace the count logic or just use Judge.me's preview badge which shows its own count. |
| CSS conflicts with theme styles | Add custom CSS overrides in `assets/custom.css.liquid`. Judge.me classes are prefixed with `jdgm-`. |

---

## 11. Cleaning Up Old SPR Code

The Shopify Product Reviews (SPR) app was discontinued in May 2024. The following files contain dead SPR code that should be cleaned up.

### Files to Modify

#### 1. `snippets/product-get-review.liquid`
**Action:** Replace entire contents or delete file.

This file renders the SPR badge:
```liquid
<div class="spr...">
    <span class="shopify-product-reviews-badge" data-id="{{ id }}">
```

**Replace with** (if any template still includes it):
```liquid
{%- comment -%}
  Legacy SPR badge — replaced by Judge.me.
  Judge.me auto-renders badges via its own script.
  Keep this file empty to avoid breaking {% include %} calls.
{%- endcomment -%}
<div class="jdgm-widget jdgm-preview-badge" data-id="{{ id }}"></div>
```

#### 2. `snippets/reviews-count.liquid`
**Action:** Replace entire contents or delete file.

This file parses SPR metafields to extract review count:
```liquid
{%- assign product_reviews_count = 0 -%}
{%- if product.metafields.spr.reviews -%}
  ...
```

**Replace with:**
```liquid
{%- comment -%}
  Legacy SPR review count — replaced by Judge.me.
  Judge.me displays counts automatically via its widgets.
{%- endcomment -%}
{%- assign product_reviews_count = 0 -%}
```

Note: Keep the `product_reviews_count` assignment so the tab header in `product-page-tab.liquid` doesn't break. Judge.me's badge will show the actual count.

#### 3. `sections/product-page-tab.liquid` (lines 81-90)
**Action:** Replace SPR review container with Judge.me widget.

See section 4.3 above for the exact before/after code.

#### 4. `sections/main-product.liquid` (lines 66-70)
**Action:** Optionally replace the SPR badge markup.

Current code:
```liquid
{%- when 'show_reviews' -%}
    <div class="star-rating">
      <span class="shopify-product-reviews-badge" data-id="{{ product.id }}"></span>
      <a href="#tab_product" class="write-review">{{'products.product.create_review' | t}}</a>
    </div>
```

Judge.me can hook into `.shopify-product-reviews-badge` elements automatically, so this may work as-is. If it doesn't, replace with:

```liquid
{%- when 'show_reviews' -%}
    <div class="star-rating">
      <div class="jdgm-widget jdgm-preview-badge" data-id="{{ product.id }}"></div>
      <a href="#tab_product" class="write-review">{{'products.product.create_review' | t}}</a>
    </div>
```

#### 5. `sections/sticky_add_to_cart.liquid`
**Action:** Check for SPR references and replace if present.

#### 6. Other files with SPR references (low priority)
These files contain SPR-related CSS or minor references. They are not critical but can be cleaned up:
- `snippets/product-list-item.liquid`
- `snippets/product-grid-item-masonry.liquid`
- `snippets/product_menu.liquid`
- `snippets/product-outofstock.liquid`
- `snippets/product_demo.liquid`
- `snippets/product_demo_column.liquid`
- `snippets/pr_group_loop.liquid`
- `snippets/product_tabs.liquid`
- `assets/style.product_page.css.liquid` (SPR-specific CSS)
- `assets/style.product.css.liquid` (SPR-specific CSS)
- `config/settings_schema.json` (SPR loading text setting)

Search for these patterns across the codebase to find all references:
```bash
grep -r "spr" --include="*.liquid" --include="*.json" -l
grep -r "shopify-product-reviews" --include="*.liquid" -l
grep -r "metafields.spr" --include="*.liquid" -l
```

### Cleanup Order

1. Install Judge.me first and verify it works
2. Replace `product-page-tab.liquid` tab_reviews content (highest impact)
3. Replace `product-get-review.liquid` (used across product cards)
4. Replace `reviews-count.liquid` (used for tab count display)
5. Optionally update `main-product.liquid` show_reviews block
6. Clean up CSS files last (lowest priority, purely cosmetic)

---

## Quick Reference

```
Install app:     https://apps.shopify.com/judgeme
Dashboard:       Shopify Admin > Apps > Judge.me
Dev server:      shopify theme dev --store d20e1f.myshopify.com
Preview:         http://127.0.0.1:9292
Rich test:       https://search.google.com/test/rich-results
Schema test:     https://validator.schema.org/
```
