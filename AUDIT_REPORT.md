# Site Audit Report — micahgao.com

Branch: `site-audit` · Date: 2026-09-12 · Not deployed; all commits local for review.

## 1. Broken images: root cause and fix

**Stack**: plain static site (two hand-written HTML files with inline CSS/JS, no build system, no dependencies). Images served as static files from `photos/`. Hosted on Cloudflare Pages (project `portfoliosite`), auto-deployed on push to `main` from github.com/micah872/portfoliosite.

**Root cause (two stacked problems):**

1. `.gitignore` contained `photos/`, ignoring the entire photos directory. The original eight photos only worked because they were committed before the rule existed (tracked files survive gitignore). Every photo added since was silently excluded from git. Commit `fdccdc2` shipped HTML referencing `06.jpeg`, `09.jpeg`, `10.jpeg` and deleted the tracked `06.jpg`, so the deploy pointed at three files that existed only on this machine.
2. Cloudflare Pages' SPA fallback masked the failure: requests for the missing images returned **HTTP 200 with `text/html`** (the index page) instead of 404, which is why browsers showed a broken-image icon rather than an obvious missing-file error.

**Fix**: removed the `photos/` ignore rule; committed the three photos (resized to max 1600px, EXIF stripped, re-encoded); added a `404.html`, which switches Cloudflare Pages from SPA fallback to real 404s so a missing asset can never masquerade as HTML again.

**Verified**: all 10 photos are valid JPEG (magic bytes `FF D8 FF`), all 16 site URLs return 200 with correct content-type from a local server, and headless Chrome loads both pages with zero failed requests.

## 2. Still broken in production: the domain binding (needs you)

`micahgao.com` is still serving the deployment frozen at **Aug 31** (it still contains the removed ski header), while `portfoliosite.pages.dev` is current. Cloudflare builds every push successfully; the custom domain is just attached to something stale. **Nothing merged from this branch will appear on micahgao.com until you rebind the domain to the `portfoliosite` Pages project in the Cloudflare dashboard.** This is the highest-priority item in this report and only you can do it.

## 3. Issues found and status

| # | Severity | Issue | File | Status |
|---|----------|-------|------|--------|
| 1 | High | New photos never tracked by git (`photos/` in .gitignore) | .gitignore | **Fixed** |
| 2 | High | micahgao.com bound to stale Aug 31 deployment | Cloudflare dashboard | **Left for you** (not repo-fixable) |
| 3 | High | Missing assets soft-200 as HTML (no 404 page) | 404.html | **Fixed** (new page) |
| 4 | Medium | New photos 2.0–4.2 MB each (12.4 MB page) | photos/*.jpeg | **Fixed** (now 0.29–0.37 MB) |
| 5 | Medium | Original photos oversized with EXIF metadata | photos/*.jpg | **Fixed** (same treatment; no GPS data was present in any photo) |
| 6 | Medium | All gallery images had empty `alt` | gallery.html | **Fixed** (descriptive alt on all 10) |
| 7 | Medium | Theme toggle accessible name didn't include visible text (WCAG 2.5.3) | index.html, gallery.html | **Fixed** (aria-label tracks state) |
| 8 | Medium | Color contrast 3.4:1 on `--fg-faint` text (needs 4.5:1) | both pages | **Left** (design token; see §5) |
| 9 | Low | No canonical, og:url/type, or Twitter tags; gallery had no OG at all | both pages | **Fixed** |
| 10 | Low | No robots.txt or sitemap.xml | new files | **Fixed** |
| 11 | Low | No security headers | _headers | **Fixed** (nosniff, frame-deny, referrer-policy, permissions-policy; verifiable only after deploy) |
| 12 | Low | LCP image was `loading="lazy"` (delays discovery) | gallery.html | **Fixed** (first two tiles eager, first `fetchpriority=high`) |
| 13 | Low | No og:image / share image | index.html | **Left** (picking a share image is your call) |
| 14 | Info | Gallery ships 1600px images into ~200px tiles | gallery.html | **Left** (real fix is srcset thumbnails; structural, your call) |
| 15 | Info | Inline CSS unminified (~7 KiB savings) | both pages | **Left** (single-file maintainability beats 7 KiB) |
| 16 | Info | Mixed `.jpg`/`.jpeg` extensions in photos/ | photos/ | **Left** (renaming would churn refs for zero user benefit) |
| 17 | Info | No .gitattributes; git warns about LF/CRLF on every touch | repo | **Left** (harmless; add `* text=auto eol=lf` if the warnings annoy you) |

N/A audit items for this stack: build warnings (no build), hydration (no framework), `npm audit` (no dependencies), next/image config (not used). HTTPS redirect is handled by Cloudflare automatically.

## 4. Content consistency check (report only, nothing edited)

**Site copy:**

| Term | Location | Finding |
|------|----------|---------|
| Hackney title | index.html:856 | `accounting & technology intern` — does not match required "Software Engineering & Corporate Accounting Intern" |
| "Minor" | index.html:816 | `B.S. Informatics + Accounting & Business Minor` — your rule says "Accounting Certificate" |
| Hackney dates | index.html:857 | `may 2026 – aug 2026` — **correct** |
| "3.8" | index.html:831 | False positive: coordinates inside the GitHub icon's SVG path |

**Hosted resume PDF (Micah_Gao_Resume.pdf)** — this file is the biggest offender against your own rules:

| Term | Finding |
|------|---------|
| "3.8" | `GPA: 3.8/4.0` in Education — rule says no GPA anywhere on the site |
| "May 2029" / "2029" | `Expected May 2029` — rule says no expected graduation date on the site resume |
| "AWS" | Listed in Engineering skills — on your do-not-claim list |
| "Minor" | `Kelley Accounting & Business Minors` — rule says "Accounting Certificate" |
| Hackney title/dates | `Software Engineering & Corporate Accounting Intern`, May 2026 – August 2026 — **correct** |

No hits anywhere for: PyTorch, Tableau, Power BI, Snowflake, Foundry, Docker, Kubernetes, Stellantis, Cummins, Toyota, "May 2028", "2028". If the site must not show a GPA or graduation date, the fix is swapping in a site-specific PDF export; say the word and I'll wire it in.

## 5. Design/copy observations (not changed)

- **Contrast**: `--fg-faint` fails WCAG AA on both themes at small sizes. Drop-in compliant values that keep the same hue: dark theme `#6b675e` → `#827d72` (≈4.6:1 on `#0f0f0d`); light theme `#999` → `#757575` (≈4.5:1 on `#fafafa`). One-line change each in both HTML files if you want it.
- The `currently:` heading got a colon in your latest edit while other section headings (`experience`, `projects`) have none.
- OG share image: pick one (or a simple branded card) and I'll add `og:image` + `twitter:image`.

## 6. Lighthouse before/after (local server, mobile emulation, slow-4G throttling)

| Page | Performance | Accessibility | Best practices | SEO |
|------|-------------|---------------|----------------|-----|
| index before | 98 | 95 | 100 | 100 |
| index after | **100** | 95 | 100 | 100 |
| gallery before | 75 | 95 | 100 | 100 |
| gallery after | 75* | 95 | 100 | 100 |

*Gallery specifics: LCP 43.1s → **11.9s**, total bytes 12,493 KiB → **~2,900 KiB**, interactive 44.7s → ~12s. The score stays 75 because Lighthouse's slow-4G simulation penalizes serving 1600px images into 200px tiles; srsets/thumbnails (item 14) is the remaining lever. Accessibility stays 95 solely because of the contrast token (item 8); the two other a11y audits that failed before (label mismatch, alt text) now pass. Real-world performance on Cloudflare (brotli, HTTP/2, no throttling) will feel far better than these simulated numbers.

Console/overflow check (headless Chrome, index + gallery + 404 at 375px and 768px): zero console errors or warnings, zero page errors, zero failed requests, no horizontal overflow on any page at either width.

External links: all return 200 except LinkedIn (999 = LinkedIn's bot block, not a dead link; verify by hand). All internal links, anchors, and the resume PDF resolve.

## 7. Commands used to verify

```powershell
# serve locally
python -m http.server 8123

# every URL: status + content type
curl -s -o /dev/null -w "%{http_code} %{content_type} /$u\n" "http://localhost:8123/$u"   # for all 16 URLs

# image signatures (magic bytes) and EXIF
python -c "from PIL import Image; ..."   # Pillow: dimensions, getexif(), GPS tag 34853

# Lighthouse (before and after)
npx lighthouse http://localhost:8123/index.html --output=json --chrome-flags="--headless=new" --only-categories=performance,accessibility,best-practices,seo
npx lighthouse http://localhost:8123/gallery.html ...

# console errors + horizontal overflow at 375/768
node browsercheck.js   # puppeteer-core against installed Chrome

# external links
curl -s -o /dev/null -w "%{http_code}" -L -A "Mozilla/5.0 ..." <each external URL>

# content consistency
grep -n across *.html/*.txt/*.xml + pypdf text extraction of Micah_Gao_Resume.pdf
```

## 8. Commits on this branch

1. `update languages list and currently section` — your uncommitted working-tree edits, committed as-is
2. `stop ignoring photos/ so new gallery images get tracked`
3. `add missing gallery photos, resized to 1600px and stripped of metadata`
4. `add alt text, social meta tags, canonical links, and theme toggle a11y fix`
5. `add 404 page, robots.txt, sitemap, and security headers`
6. `resize original gallery photos to 1600px max and strip metadata`
7. `load first gallery tiles eagerly so LCP image is discovered early`
8. `add site audit report` (this file)

After merging to main and pushing: verify on **portfoliosite.pages.dev** first (it always reflects the latest build), then fix the domain binding (§2) and verify micahgao.com matches.
