# Deep Page Analysis: 5paisa.com/stocks/sbin-share-price

**Page Type:** Programmatic stock detail page (State Bank of India / SBIN)  
**Industry:** Financial Services — YMYL (Your Money or Your Life)  
**Analysis Date:** June 20, 2026

---

## Page Scorecard

| Category        | Score   | Bar              |
|----------------|---------|------------------|
| **Overall**     | 64/100  | `████████░░░░░░░` |
| On-Page SEO     | 72/100  | `██████████░░░░░` |
| Content Quality | 68/100  | `█████████░░░░░░` |
| Technical       | 52/100  | `███████░░░░░░░░` |
| Schema          | 62/100  | `████████░░░░░░░` |
| Images          | 70/100  | `█████████░░░░░░` |

---

## Issues Found

### 🔴 Critical

#### 1. Page Response Time: 5.77 Seconds

The page took 5.77s to fully deliver — measured from a local curl request, meaning real-user LCP on mobile India will be substantially worse. Stock pages are time-sensitive and users expect near-instant price visibility; slow TTFB will hurt engagement.

**Root causes detected:**
- 6 render-blocking CSS files loaded synchronously (global header/footer, Owl Carousel, dashboard dropdown, views module, custom stylesheet)
- 6 synchronous JS scripts with no `async` or `defer` (Drupal core loaders, OTP autofill, sticky form)
- mPulse/BOOMR real-user monitoring script adds pre-load overhead
- Drupal dynamic cache shows `MISS` on first hit (server-side cache header)

**Fix:** Defer all non-critical JS. Bundle and inline critical CSS. Set Drupal page cache TTL to at least 1 hour for stock detail pages. The live price data should load asynchronously via JS after the static frame paints.

---

### 🟠 High

#### 2. Twitter Card Tags Completely Missing

The page has zero `twitter:card`, `twitter:title`, or `twitter:image` meta tags. When SBIN share price is shared on X/Twitter (common for traders), the link will render as a bare URL — no preview, no engagement.

**Fix — add to `<head>`:**

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@5paisa" />
<meta name="twitter:title" content="SBI Share Price Today: Live State Bank of India (SBIN)" />
<meta name="twitter:description" content="SBI share price today with live NSE/BSE updates, market cap, fundamentals, technicals, 52-week range and historical performance." />
<meta name="twitter:image" content="https://images.5paisa.com/MarketIcons/SBIN.png" />
```

---

#### 3. Schema `@type: "Webpage"` — Invalid Capitalisation

The second JSON-LD block uses `"@type": "Webpage"` instead of the correct `"WebPage"`. Schema.org is case-sensitive. Google's Rich Results parser will silently fail to parse this type, forfeiting any entity association benefit.

**Fix:** Change `"Webpage"` → `"WebPage"` in the schema.

---

#### 4. BreadcrumbList Item 3 Uses Malformed Name

The breadcrumb schema has:
```json
{ "position": 3, "name": "Sbin share price", "item": "https://www.5paisa.com/stocks/sbin-share-price" }
```
This renders in Google breadcrumb rich results as "Sbin share price" — inconsistent capitalisation and ticker formatting.

**Fix:**
```json
{ "position": 3, "name": "State Bank of India Share Price", "item": "https://www.5paisa.com/stocks/sbin-share-price" }
```

---

#### 5. `og:type` Value Uses Non-Standard Capitalisation

The page sets `og:type` as `"Website"` (capital W). Some crawlers and social preview tools may reject capitalised values.

**Fix:** Change to `content="website"`.

---

#### 6. No `dateModified` on WebPage Schema

Financial data pages must signal freshness. There is no `dateModified` property, and there is no visible "Last Updated" timestamp on the page. For a YMYL stock page where data changes intraday, Google's QRG specifically looks for update signals to assess content freshness and E-E-A-T.

**Fix:** Add `"dateModified": "<ISO-8601 timestamp>"` to the schema, updated each time live data refreshes. Also show a human-readable timestamp in the visible content (e.g., *"Data as of: 20 Jun 2026, 20:16 IST"*).

---

### 🟡 Medium

#### 7. Title Tag Is 67 Characters — Over the 60-Char Best Practice

**Current title:** `SBI Share Price Today: Live State Bank of India NSE/BSE` (67 chars)

This will truncate in desktop SERPs (~600px). It also repeats "SBI Share Price" and "State Bank of India Share Price", wasting crawl space on a repeated keyword.

**Suggested titles:**
- `SBI Share Price Live | State Bank of India NSE/BSE` (58 chars)
- `SBI Share Price Today — Live NSE/BSE | 5paisa` (47 chars, punchy)

---

#### 8. Missing Corporation Schema for State Bank of India

The page describes SBIN the company, but there is no `Corporation` schema — only a `FinancialProduct` schema that names the page, not the entity. Competitors like Moneycontrol use structured Corporation schema which powers knowledge panel associations.

**Suggested addition:**
```json
{
  "@context": "https://schema.org",
  "@type": "Corporation",
  "@id": "https://www.wikidata.org/wiki/Q1524990",
  "name": "State Bank of India",
  "alternateName": "SBI",
  "tickerSymbol": "SBIN",
  "exchange": "NSE",
  "url": "https://www.sbi.co.in/",
  "sameAs": [
    "https://www.wikidata.org/wiki/Q1524990",
    "https://en.wikipedia.org/wiki/State_Bank_of_India"
  ],
  "legalName": "State Bank of India",
  "foundingDate": "1955-07-01",
  "addressCountry": "IN",
  "industry": "Banking"
}
```

---

#### 9. FAQPage Schema — No Longer Generates Rich Results

The page uses `FAQPage` schema with 4 Q&As. FAQ rich results are now limited and may not appear as expandable accordions in SERPs. However, the structured Q&A retains value as an AI Overview and LLM citation signal — keep the schema, do not remove it, but revisit the choice of questions.

The current 4 questions (Share Price, Market Cap, P/E, intrinsic value) will have stale schema answers between crawls. Consider whether static FAQ answers (e.g., *"How to invest in SBI shares?"*) serve better for durable AI citations.

---

#### 10. `og:image` Uses a 96×96px Company Icon Logo

The `og:image` points to the same 96×96 stock ticker icon used in the UI. Social platforms require a minimum of 600×315px for proper Open Graph image previews. A tiny logo will render as a micro-thumbnail or be discarded entirely by WhatsApp, LinkedIn, and Facebook.

**Fix:** Generate a dedicated social preview card per stock page (1200×630px) showing the stock name, current price, and 5paisa branding. This is a template-level fix affecting all stock pages.

---

#### 11. 2 Images Missing Width/Height Dimensions (CLS Risk)

The `pointer.svg` image (used twice) has no `width` or `height` attributes:
```html
<img src=".../pointer.svg" alt="...">  <!-- no width/height -->
```
Without explicit dimensions, the browser cannot reserve layout space, causing Cumulative Layout Shift when the SVG loads.

**Fix:** Add `width` and `height` attributes, or use CSS `aspect-ratio` to reserve space.

---

#### 12. `cache-control: public` With No Explicit `max-age`

The page response headers show `cache-control: public` without an explicit `max-age`. The `Expires` header gives ~10 minutes. For a stock detail page served at scale, this short cache window means frequent cache misses and Drupal re-renders — contributing to slow TTFB.

**Fix:** Set `cache-control: public, max-age=1800` for the HTML shell, and load live prices via XHR that bypasses the HTML cache.

---

### 🔵 Low

#### 13. No Visible "Last Updated" Timestamp in Body Content

YMYL financial pages should clearly display when data was last refreshed. The FAQ section contains "As on 20 June, 2026 | 20:16" but there is no prominent page-level timestamp (e.g., near the price) in a `<time datetime="...">` element visible to crawlers.

---

#### 14. `meta name="robots"` Tag Absent

While the absence of a robots tag defaults to index, it's best practice for programmatic pages to explicitly declare this — especially on Drupal CMS where a module misconfiguration could accidentally inject `noindex`.

**Fix:**
```html
<meta name="robots" content="index, follow, max-snippet:-1, max-video-preview:-1, max-image-preview:large" />
```

---

#### 15. Breadcrumb Schema Uses Non-Canonical Homepage URL

Breadcrumb item 1 uses `"item": "https://www.5paisa.com"` (no trailing slash), while the canonical homepage is `https://www.5paisa.com/` (with trailing slash). This can cause URL deduplication noise in GSC.

---

## Full Findings Summary

| # | Issue | Category |
|---|-------|----------|
| 1 | Page response time 5.77s — render-blocking assets | Performance |
| 2 | Twitter Card tags completely missing | Technical |
| 3 | `@type: "Webpage"` invalid capitalisation | Schema |
| 4 | BreadcrumbList item 3 name: "Sbin share price" | Schema |
| 5 | `og:type = "Website"` (should be lowercase) | Technical |
| 6 | No `dateModified` on WebPage schema + no visible timestamp | Content/Schema |
| 7 | Title 67 chars, "Share Price" repeated | On-Page SEO |
| 8 | Missing Corporation schema for SBI the entity | Schema |
| 9 | FAQPage schema won't produce rich results | Schema |
| 10 | `og:image` is 96×96px logo — too small for social cards | Images |
| 11 | 2 images missing width/height dimensions | Images |
| 12 | `cache-control: public` with only 10-min implicit window | Technical |
| 13 | No visible `<time>` timestamp for last data refresh | Content |
| 14 | Meta robots tag absent from page | Technical |
| 15 | Breadcrumb schema homepage URL missing trailing slash | Schema |

---

## What's Working Well

- ✅ H1 is clean and singular: "State Bank of India" — matches search intent exactly
- ✅ Canonical is self-referencing and correct
- ✅ Hreflang properly implemented across 5 languages (`en-IN`, `hi-IN`, `mr-IN`, `bn-IN`, `x-default`)
- ✅ All 8 images have descriptive alt text — zero missing
- ✅ BreadcrumbList has 3 levels with proper positions
- ✅ About SBI section provides unique, crawlable static content that differentiates from thin stock-ticker-only pages
- ✅ External links to authoritative sources (Business Line) — strong E-E-A-T signal
- ✅ `FinancialProduct` schema provides a domain-relevant type signal
- ✅ LCP candidate image (SBIN.png) correctly marked as hero image
- ✅ Word count ~1,847 is solid for a programmatic stock page (minimum ~500 for this type)
- ✅ Similar Stocks section with descriptive anchor text creates strong internal link equity

---

## Schema Additions — Ready to Use

### Fix 1: Correct WebPage Schema

```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "@id": "https://www.5paisa.com/stocks/sbin-share-price",
  "name": "SBI Share Price Today: Live State Bank of India NSE/BSE",
  "description": "SBI share price today with live NSE/BSE updates, market cap, fundamentals, technicals, 52-week range, financials, news, and historical performance data.",
  "url": "https://www.5paisa.com/stocks/sbin-share-price",
  "dateModified": "2026-06-20T20:16:00+05:30",
  "publisher": {
    "@type": "Organization",
    "name": "5paisa",
    "logo": {
      "@type": "ImageObject",
      "url": "https://storage.googleapis.com/5paisalogonew.svg"
    }
  },
  "breadcrumb": {
    "@id": "https://www.5paisa.com/stocks/sbin-share-price#breadcrumb"
  }
}
```

### Fix 2: Corrected BreadcrumbList

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "@id": "https://www.5paisa.com/stocks/sbin-share-price#breadcrumb",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "5Paisa Homepage",
      "item": "https://www.5paisa.com/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Stocks",
      "item": "https://www.5paisa.com/stocks"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "State Bank of India Share Price",
      "item": "https://www.5paisa.com/stocks/sbin-share-price"
    }
  ]
}
```

### Fix 3: Corporation Schema for SBI Entity

```json
{
  "@context": "https://schema.org",
  "@type": "Corporation",
  "@id": "https://www.wikidata.org/wiki/Q1524990",
  "name": "State Bank of India",
  "alternateName": ["SBI", "SBIN"],
  "tickerSymbol": "SBIN",
  "exchange": "NSE",
  "legalName": "State Bank of India",
  "foundingDate": "1955-07-01",
  "url": "https://www.sbi.co.in/",
  "addressCountry": "IN",
  "industry": "Banking",
  "numberOfEmployees": {
    "@type": "QuantitativeValue",
    "value": 230000
  },
  "sameAs": [
    "https://en.wikipedia.org/wiki/State_Bank_of_India",
    "https://www.wikidata.org/wiki/Q1524990",
    "https://www.nseindia.com/get-quotes/equity?symbol=SBIN"
  ]
}
```

---

## Prioritised Action Plan

| Timeline | Action |
|----------|--------|
| **This week** | Fix `@type: "Webpage"` → `"WebPage"` (applies to all ~5k stock pages) |
| **This week** | Fix BreadcrumbList item 3 capitalisation |
| **This week** | Add Twitter Card meta tags (template-level) |
| **This week** | Fix `og:type` from `"Website"` → `"website"` |
| **This week** | Add `dateModified` to WebPage schema (update on refresh) |
| **2–3 weeks** | Performance: defer non-critical JS, bundle CSS, increase page cache TTL |
| **2–3 weeks** | Generate proper 1200×630 OG social preview images (template) |
| **1 month** | Add Corporation schema for each listed company (from stock master data) |
| **1 month** | Add `<time datetime="...">` element timestamp visibly on page |
| **Backlog** | Shorten title template to under 60 characters |
| **Backlog** | Add explicit meta robots tag to stock pages |

---

## How to Verify Fixes Succeeded

- **Schema:** Google Rich Results Test → expect BreadcrumbList to pass with zero errors
- **Twitter Card:** Twitter Card Validator → expect preview to render
- **Performance:** Repeat `curl -w "%{time_total}"` after caching changes — target <2s TTFB
- **dateModified:** Google Search Console → URL Inspection should surface updated timestamp

**Leading indicator to monitor:** Track GSC CTR for `/stocks/*-share-price` template pages. Schema fixes + Twitter Card will improve click-through; performance fixes will improve mobile ranking position.
