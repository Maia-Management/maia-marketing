# maia-marketing Audit Report
**Date:** 2026-05-15
**Health Score:** 7/10

## Summary
The site is well-structured, mobile-first, fully bilingual, and free of secrets or NIT bugs. The biggest outstanding problems are: a stale staging file (`live_fetch.html`) that is publicly indexed with old failing contrast values and mojibake text; a second team-card with placeholder initials and a generic "Marketing Lead" name that should be filled in or removed; and form labels on Step 2 that are English-only despite the default Spanish language. No placeholder copy or Lorem ipsum was found in any shipped template.

---

## Top 5 Fixes (Priority Order)
1. **Delete or block `live_fetch.html`** — it is publicly indexed (no noindex), has a BOM, contains double-encoded UTF-8 mojibake throughout the Spanish copy, and ships the old low-contrast colour values that were already fixed in `index.html`.
2. **Fill in or remove the "Marketing Lead" placeholder team card** — the initials bubble shows "MM" and the name is literally "Marketing Lead". This is visible to every visitor.
3. **Add bilingual labels to Step 2 form fields** — `name`, `whatsapp`, and `language` labels are hard-coded in English while the page defaults to Spanish.
4. **Add `required` attributes and native validation to form inputs** — the form uses JS `alert()` validation only; if JS fails, empty submissions go through.
5. **Commit unstaged changes or clean the working tree** — six files are modified but not committed (`_redirects`, `index.html`, `live_fetch.html`, `netlify.toml`, `robots.txt`, `thank-you.html`).

---

## Issues

### P0 — Critical (fix before any launch/traffic)

| # | Issue | File + Line | Fix |
|---|-------|-------------|-----|
| 1 | `live_fetch.html` has `<meta name="robots" content="index, follow">` — it will be crawled and indexed as a duplicate of `index.html`, diluting SEO | `live_fetch.html` line 9 | Add `noindex, nofollow` or delete the file; add a Netlify redirect rule away from it |
| 2 | `live_fetch.html` has a UTF-8 BOM (`EF BB BF`) causing `Â©`, `Â¿`, `Ã©`, `â€"` mojibake throughout all Spanish strings | `live_fetch.html` throughout (confirmed at lines 1946, 1994, 1997) | Delete the file or, if kept, re-save as UTF-8 without BOM; the canonical `index.html` is fine |
| 3 | `live_fetch.html` ships old, WCAG-failing contrast tokens (`--muted: #7F9AB4` = 6.71:1 against `#080C10`; `--faint: #7890A8` = 5.94:1) that were already corrected in `index.html` | `live_fetch.html` lines 120–125 (vs `index.html` lines 120–125) | Delete file; confirmed fix is already in `index.html` |

### P1 — High (fix within this sprint)

| # | Issue | File + Line | Fix |
|---|-------|-------------|-----|
| 4 | Second team card shows placeholder avatar initials "MM" and title "Marketing Lead" with no real name — visible to all visitors | `index.html` lines 1761–1762 | Replace with real person name/photo, or remove the card until the person is confirmed |
| 5 | Step 2 form labels (`name`, `whatsapp`, `language`) are hard-coded English only; page defaults to Spanish | `index.html` lines 1899, 1903, 1907 | Wrap each label text in `<span data-lang="en">…</span>` / `<span data-lang="es">…</span>` pairs, matching the pattern used in Step 1 |
| 6 | Form inputs have no `required` attribute — JS validation uses `alert()` calls only; if script fails, bare submissions pass | `index.html` lines 1877, 1900, 1904 | Add `required` to `name` and `whatsapp` inputs; add `required` to `goal` and `language` selects |
| 7 | Six files are modified but not committed to git (`_redirects`, `index.html`, `live_fetch.html`, `netlify.toml`, `robots.txt`, `thank-you.html`) | repo root — `git status` | Review and commit genuine changes; delete `live_fetch.html` before committing |
| 8 | The "Our Work" section and team bio both claim "14 brands" but only 11 distinct brand tiles are rendered (missing: Sushi Pop, and two others from the CLAUDE.md brand list) | `index.html` lines 1784, 1793–1848 | Either add the missing tiles or change the copy from "14" to the accurate count |
| 9 | `hreflang` tags for `en` and `es` both point to the same URL (`https://maia-marketing.com/`) — technically correct for a single-URL bilingual site, but Google may flag it as misconfigured since they expect distinct URLs per language | `index.html` lines 13–15 | If single-URL bilingual is intentional, this is acceptable; if separate language URLs are planned, update accordingly |

### P2 — Medium (fix this month)

| # | Issue | File + Line | Fix |
|---|-------|-------------|-----|
| 10 | `sitemap.xml` does not include `thank-you.html` — acceptable since it is noindex, but `/thank-you` has a canonical and could be confirmed excluded | `sitemap.xml` | Confirm intentional omission; add an XML comment noting it is noindex |
| 11 | `privacy.html` and `terms.html` have no `<link rel="author">` tag (present on `index.html`) | `privacy.html`, `terms.html` | Add `<link rel="author" href="https://maia-marketing.com">` to both |
| 12 | `privacy.html` and `terms.html` have no Structured Data / JSON-LD | both files | Add at minimum an `Organization` schema block, consistent with `index.html` |
| 13 | `assets/images/team-strategy.png` (1.6 MB PNG) is present in the repo and is referenced only inside a commented-out HTML block — adds unnecessary repo weight | `index.html` lines 1460–1462 (commented) | Delete the file if the section is permanently removed; convert to WebP if it will be restored |
| 14 | `consent-banner.js` initialises `window.dataLayer` and `window.gtag` but no GTM or GA4 snippet is present in any HTML file — consent infrastructure is wired up with nothing downstream to control | `index.html`, `consent-banner.js` | Either connect GTM/GA4 snippets after the banner script, or document the intentional analytics-free state |
| 15 | `live_fetch.html` is tracked in git and has uncommitted changes — it will be deployed to production as a live page until removed | repo | Delete via `git rm live_fetch.html` and commit |
| 16 | `role="menu"` on the mobile drawer (`#mobileMenu`) is semantically incorrect — `menu` role is for application menus, not navigation lists | `index.html` line 1413 | Change to `role="navigation"` or remove the role; the `<nav>` parent already provides landmark context |

### P3 — Low / Nice-to-have

| # | Issue | File + Line | Fix |
|---|-------|-------------|-----|
| 17 | The `<select>` options in the goal dropdown are English-only (e.g. "More leads & inquiries", "WhatsApp sales automation") — no Spanish equivalents | `index.html` lines 1883–1887 | Add bilingual option text using the same `data-lang` display toggle pattern |
| 18 | `thank-you.html` does not load `consent-banner.js` — a user who lands on thank-you without having visited index first will have no consent banner | `thank-you.html` (head, no consent script) | Add `<script src="/consent-banner.js?v=20260430a" defer></script>` to thank-you head |
| 19 | The `lang-toggle` ARIA label on `index.html` is static (`"EN ES language toggle"`) rather than dynamic, unlike the improved version in `thank-you.html` which updates to reflect current state | `index.html` line 1404 | Port the dynamic `aria-label` update logic from `thank-you.html` lines 31–34 to `index.html` |
| 20 | Back button arrow in Step 2 form points the wrong direction: `→ Back` should be `← Back` | `index.html` line 1918 | Change `→ Back` to `← Back` |
| 21 | `og:locale` is `en_US` on all pages but the default language is Spanish (`es`) — minor signal mismatch | `index.html` line 30, `privacy.html`, `terms.html` | Change to `es_CO` or add `og:locale:alternate` for `en_US` |
| 22 | Ecosystem tile for Mapané Marine links to `https://mapana-marine.com` but the display URL shows `mapana-marine.com` — consistent, but verify domain is live | `index.html` line 1828 | Confirm domain resolves; `mapana-marine.com` is the canonical from CLAUDE.md |

---

## Verified OK

- **WhatsApp number:** All public-facing CTAs use `wa.me/19034598763` exclusively. No personal numbers found. Checked across `index.html`, `live_fetch.html`, `privacy.html`, `terms.html`, `thank-you.html`, and `consent-banner.js`.
- **NIT number:** All occurrences correctly show `901.862.977-7`. No `-1` suffix bug found anywhere in the repo.
- **Secrets / credentials:** No API keys, tokens, `.env` files, passwords, or OAuth secrets found in any tracked file. `.env` is listed in `.gitignore`.
- **Viewport meta tag:** Present and correct (`width=device-width, initial-scale=1.0`) on all four HTML files.
- **Responsive CSS:** Mobile-first breakpoints at 640px, 768px, 900px. No fixed-width elements wider than the viewport found. `overflow-x: hidden` on `html`, `body`, and `main`.
- **Image optimisation:** Hero image uses WebP format with `srcset` (480w / 720w / 1200w), correct `width`/`height` attributes, and `loading="lazy"`. `og-image.jpg` is 154 KB — within acceptable range.
- **Alt text:** All `<img>` tags in `index.html` have descriptive, bilingual alt text. WhatsApp FAB SVG uses `aria-label` on the anchor.
- **Heading hierarchy:** Single `<h1>` per page, followed by `<h2>` and `<h3>` in correct order. No heading levels skipped.
- **Duplicate IDs:** None found.
- **ARIA landmarks:** `<nav>` has `aria-label="Main navigation"`. All major sections use `aria-labelledby` pointing to their heading IDs. Form has `aria-label`.
- **Colour contrast (index.html):** Updated CSS variables documented as WCAG AAA compliant (`--ink` ~14.6:1, `--ink-2` ~10.4:1, `--ink-3` ~7.97:1, `--muted` ~9.46:1). Accent `#19A898` on dark backgrounds passes AA for large text.
- **robots.txt:** Valid. `Allow: /` with correct sitemap reference.
- **sitemap.xml:** Covers homepage, privacy, and terms. Dates and priorities are reasonable.
- **Canonical tags:** Correct absolute URLs on all pages.
- **netlify.toml:** CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, and Permissions-Policy headers are all present. www → apex redirect configured. Cache-Control rules for images and JS are appropriate.
- **Netlify forms:** `data-netlify="true"`, `netlify-honeypot`, and hidden `form-name` field are all correctly implemented. Fetch posts to `/` (self), compatible with CSP `form-action 'self'`.
- **Internal links:** `/privacy`, `/terms`, and `/` all resolve correctly. Netlify redirects cover `.html` extension variants.
- **No Lorem ipsum / placeholder copy:** All body copy and marketing claims appear real and production-ready.
- **No committed .env files:** Confirmed; only `.gitignore` reference.
