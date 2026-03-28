---
description: Deploy an HTML post to WordPress with full GEO Lab compliance — strip DOCTYPE, add JSON-LD, upload, set categories/tags/RankMath, clear cache, verify.
---

# WordPress Post Deployment

Deploy the provided HTML file to thegeolab.net as a WordPress post with full GEO Lab compliance.

## Input

The user will provide one of:
- A file path to an HTML file (e.g. `C:\Users\jorge\Downloads\post.html`)
- A WordPress post ID to update
- A slug for an existing draft

## Deployment Steps

Execute ALL steps in order. Do not skip any step.

### Step 1 — Read and prepare content

1. Read the HTML file
2. Strip `<!DOCTYPE>`, `<html>`, `<head>`, `<body>` tags
3. KEEP: all `<script type="application/ld+json">` blocks (JSON-LD schema)
4. KEEP: the `<style>` block (v3 CSS)
5. KEEP: the `<article>` content
6. Wrap in `<!-- wp:html -->` ... `<!-- /wp:html -->` if not already wrapped
7. Remove any `<h1>` tags (WordPress title generates the H1)
8. Remove any homepage links (`href="https://thegeolab.net/"`)
9. Verify no `<body>` tags remain in content (F010 failure pattern)

### Step 2 — Verify JSON-LD schema

Check the content has these schema blocks:
- Article JSON-LD (with author Artur Ferreira, publisher The GEO Lab, digitalSourceType)
- FAQPage JSON-LD (if FAQ section exists)
- Organization JSON-LD (if not in Article publisher block)

If Article JSON-LD is missing, generate it:
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "[POST TITLE]",
  "datePublished": "[TODAY]",
  "dateModified": "[TODAY]",
  "author": {
    "@type": "Person",
    "name": "Artur Ferreira",
    "url": "https://thegeolab.net/about/",
    "sameAs": ["https://linkedin.com/in/arturgeo", "https://x.com/TheGEO_Lab"]
  },
  "publisher": {
    "@type": "Organization",
    "name": "The GEO Lab",
    "url": "https://thegeolab.net",
    "sameAs": ["https://x.com/TheGEO_Lab", "https://linkedin.com/in/arturgeo"]
  },
  "digitalSourceType": "https://cv.iptc.org/newscodes/digitalsourcetype/humanOriginated"
}
```

### Step 3 — Create or update WordPress post

Use the WordPress REST API (Basic auth: `jorge:u2Ee z5sw OaLz GrcM PpC3 FtKc`):

- If creating new: `POST /wp-json/wp/v2/posts` with `status: "draft"`
- If updating existing: `POST /wp-json/wp/v2/posts/{id}`
- Set slug from the canonical URL or file name

### Step 4 — Set categories and tags

Refer to POST_AUDIT_RULES.md Part 24 for approved lists. Use the WordPress REST API to:
1. Find or create each category/tag by slug
2. Set on the post: minimum 2 categories, minimum 3 tags
3. Ask the user which categories/tags to use if not obvious from the content

### Step 5 — Set RankMath fields

```python
meta = {
    "rank_math_focus_keyword": "[primary keyword]",
    "rank_math_description": "[150 chars max meta description]"
}
```

### Step 6 — Set excerpt

Generate a 150-200 character excerpt distinct from the meta description and TL;DR.

### Step 7 — Clear Nginx cache

```bash
ssh root@100.87.191.9 "rm -rf /var/cache/nginx/fastcgi/* && systemctl reload nginx"
```

### Step 8 — Post-publish verification (Part 19)

Run these checks on the live URL:
```bash
SLUG="[post-slug]"
BASE="https://thegeolab.net/${SLUG}/"
curl -s $BASE | grep -c '"FAQPage"'       # ≥1
curl -s $BASE | grep -c '"Article"'        # ≥1
curl -s $BASE | grep -c '<h1'             # 1 only
curl -s $BASE | grep -c 'faq-item'        # ≥5 if FAQ exists
curl -s $BASE | grep -c 'application/ld+json'  # ≥2
curl -s -o /dev/null -w "%{http_code}" $BASE   # 200
```

Report results to the user. If any check fails, diagnose and fix before marking deployment complete.

## Output

Print:
- Post ID and edit URL
- Slug and live URL
- Categories and tags set
- Verification results (pass/fail per check)
