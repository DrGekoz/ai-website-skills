---
name: seo
description: >
  Deterministic LLM-first SEO audits for websites, blog posts, and GitHub
  repositories. Use when asked to "perform SEO analysis", "run SEO audit",
  "analyze SEO", "check technical SEO", "review schema", "Core Web Vitals",
  "E-E-A-T", "hreflang", "GEO", "AEO", or GitHub repo SEO optimization.
  For full/page/repo audits, run bundled scripts for evidence and return
  prioritized, confidence-labeled fixes.
---

# SEO Skill (Agentic / Claude / Codex)

LLM-first SEO analysis skill with 16 specialized sub-skills, 10 specialist agents, and 33 scripts for website, blog, and GitHub repository optimization.

## Deterministic Trigger Mapping

- If user says `perform seo analysis on <url>` (or similar generic SEO request with a URL), treat it as a **single-URL full audit**.
- If no explicit sub-skill is specified, run the full/page audit path with **LLM-first reasoning** and script-backed evidence.
- For full/page audits, always produce:
  - `FULL-AUDIT-REPORT.md` (detailed findings)
  - `ACTION-PLAN.md` (prioritized fixes)

## Sub-Skills (16)

### 1. Technical SEO
Analyzes site structure: robots.txt, sitemaps, page speed (Core Web Vitals), HTTPS, redirects, hreflang, duplicate content, pagination, and crawlability.

### 2. On-Page SEO
Meta tags, heading structure, keyword usage, content quality, image optimization, internal linking, structured data (JSON-LD), and URL optimization.

### 3. Content Strategy & Optimization
Analyzes existing content for keyword gaps, topical authority, readability, and freshness. Generates content briefs.

### 4. Link Building (Internal & External)
Internal linking structure analysis, external backlink profile, orphan pages, and link equity distribution.

### 5. Keyword Research & Mapping
Identifies primary and secondary keywords, search intent mapping, keyword cannibalization, and SERP feature opportunities.

### 6. Competitor Analysis
Reverse-engineers competitor SEO strategies: backlink gaps, keyword gaps, content gaps, and technical advantages.

### 7. Local SEO (NAP, GMB)
Validates Name/Address/Phone consistency, Google Business Profile optimization, local citations, local keyword opportunities.

### 8. E-E-A-T & YMYL
Expertise, Authoritativeness, Trustworthiness signals. Author bios, citations, fact-checking, medical/financial content rigor.

### 9. GEO / AEO / Answer Engine Optimization
Optimizes for generative AI, voice search, featured snippets, People Also Ask, and answer engines.

### 10. Social Signals & Shareability
Open Graph, Twitter Cards, social meta, share count tracking.

### 11. Schema Markup (Structured Data Validation)
JSON-LD generation and validation for LocalBusiness, FAQ, Article, Product, Review, Service, BreadcrumbList, etc.

### 12. Site Architecture & UX SEO
Navigation depth, information architecture, mobile usability, Core Web Vitals, accessibility (a11y), and page experience signals.

### 13. Video & Image SEO
Alt text, file names, image sitemaps, video schema, lazy loading, WebP/AVIF formats.

### 14. Core Web Vitals (Performance Audit)
LCP, FID/INP, CLS measurement, render-blocking resources, lazy loading, CDN/compression.

### 15. Ghost / De-indexed / Penalty Recovery
Checks for manual actions, noindex tags, robots.txt blocks, thin content, Panda/Penguin penalties.

### 16. Competitor Gap Analysis (GitHub)
- 10+ GitHub-specific scripts for repo SEO
- `github_competitor_research.py`, `github_repo_audit.py`, etc.

## Audit Structure (Full Workflow)

### Step 1 — LLM-first pre-flight
1. Based on trigger mapping, determine exact sub-skills needed
2. List URLs/pages to audit
3. Determine output format (markdown report + HTML visual report)

### Step 2 — Run scripts for evidence
Run relevant scripts from `scripts/` to gather evidence:
- `fetch_page.py <url>` — Fetch and parse a page
- `broken_links.py <url>` — Check for broken links
- `duplicate_content.py <url>` — Check for duplicate/similar content
- `article_seo.py <url>` — Article-level SEO audit
- `entity_checker.py <file>` — Extract semantic entities
- `analyze_visual.py <screenshot>` — Analyze visual page structure

### Step 3 — Execute sub-skill workflows
For each applicable sub-skill:
1. Analyze evidence from scripts
2. Identify issues (severity: critical/high/medium/low)
3. Assign confidence (high/medium/low)
4. Provide specific remediation steps

### Step 4 — Generate deliverables
- `FULL-AUDIT-REPORT.md` — All findings, organized by sub-skill
- `ACTION-PLAN.md` — Prioritized fix list (P0/P1/P2)
- Optional: `SEO-REPORT.html` (via `generate_report.py`)

### Step 5 — Implement fixes (close the loop)
After the report and action plan, implement all P0 and P1 fixes directly in the source files:

1. **Structured data**: Add JSON-LD (LocalBusiness, Organization, WebSite) to every page's `<head>`
2. **Meta fixes**: Fix canonical URLs, add missing twitter:site/creator, OG image dimensions, remove duplicate title tags
3. **Sitemap**: Regenerate to include all live pages, remove 404 pages, update URLs to match preferred domain (www vs non-www)
4. **Robots.txt**: Update sitemap URL, manage AI crawler entries explicitly
5. **Security headers**: Add via `_headers` file or `netlify.toml` (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy)
6. **Redirect broken pages**: Add 301 redirects from dead URLs to live equivalents
7. **Site cleanup**: Remove unreferenced assets (unused icon dirs, dev scripts, screenshots, build artifacts, template files, documentation) — trace all file references from HTML `src`/`href` attributes and delete orphans
8. **Thin content**: Expand pages under 300 words on text-heavy pages

Batch repetitive fixes (JSON-LD, canonical, meta) across all files using a script, not manual find-and-replace per file.

#### Removing unused JS libraries (render-blocking weight reduction)

Heavy CDN libraries (Three.js, GSAP, Framer Motion) are often loaded on every page but never actually called. To remove them safely:

1. **Verify zero usage**: `grep -rn "THREE\|GSAP\|Framer\|gsap\|framer" *.html pages/ 2>/dev/null | grep -v "src=\"https://cdn"` — if only the `<script src>` lines match, the library is unused
2. **Batch remove**: Write a Python script that regex-removes the matching `<script>` lines from all HTML files. Use `re.sub()` with the exact CDN URL pattern
3. **Add async/defer**: Remaining CDN scripts get `async` (Tailwind, which rebuilds from classes) or `defer` (non-critical widgets)

**Pitfall:** A "loaded but unused" library often has zero references in the HTML itself (not even a `Three.js` comment). Always grep across ALL pages, not just index.html.

#### Adding `loading="lazy"` to images (below-fold only)

Batch-add `loading="lazy"` to all images outside `<header>` tags (hero images should be eager for LCP):

```python
# Split on header boundaries
parts = re.split(r'(<header[^>]*>.*?</header>)', content, flags=re.DOTALL)
for i, part in enumerate(parts):
    if part.startswith('<header'):
        continue  # skip header — keep eager
    parts[i] = re.sub(r'(<img\s)(?!.*\bloading=)', r'\1loading="lazy" ', part)
```

Then do a second pass to undo lazy on the first `<img>` after `</header>` (hero image on sub-pages):

```python
first_img = re.search(r'<img\s', after_header)
if 'loading="lazy"' in img_tag[:100]:
    content = content[:start] + content[start:end].replace('loading="lazy" ', '') + content[end:]
```

#### Adding caching headers via netlify.toml

For static sites deployed on Netlify, add granular Cache-Control by pattern:

```toml
# Long-lived immutable cache for fingerprinted/content-hashed assets
[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "*.css"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "*.js"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "*.webp"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

# Shorter cache for HTML (revalidate often)
[[headers]]
  for = "*.html"
  [headers.values]
    Cache-Control = "public, max-age=3600, stale-while-revalidate=86400"

[[headers]]
  for = "/"
  [headers.values]
    Cache-Control = "public, max-age=3600, stale-while-revalidate=86400"
```

#### Fixing the Netlify deploy for static sites

When `netlify deploy --prod` fails, common issues and fixes:

1. **Build command from UI**: Netlify's web UI can set a `build.command` that runs `npm run build`. If the site is static with no build step, create a minimal `package.json` with a no-op build script: `"build": "echo 'Static site'"`.
2. **Functions dependency bundling**: `netlify deploy --prod` tries to bundle Netlify Functions from `netlify/functions/`. If functions use external deps (jszip, @netlify/functions), install them locally: `npm install <dep>`. The CLI needs these available to bundle.
3. **Deploy verification**: After deploy, navigate to the live URL and check `<title>` (the most visible change) to confirm the deploy took effect.

**Pitfall:** Netlify Functions use an older bundler that only resolves deps from the top-level `package.json`. Installing deps globally or in subfolders won't be picked up.

#### Google Fonts consolidation

Sites often load several font families but only use 1-2. Steps:

1. Check `--font-heading` and `--font-body` CSS custom properties to see which fonts are actually declared
2. Remove unreferenced font families from the Google Fonts `<link>` tag across all HTML files
3. Ensure `&display=swap` is appended to the Google Fonts URL
4. Also check `styles.css` for any `@import` of the same fonts — those should have `display=swap` too

### Step 6 — Re-audit (before/after)
Re-run the key scripts (validate_schema, social_meta, security_headers, robots_checker) on the LIVE URL and produce a before/after score comparison.

**Pitfall:** Scripts check the live site, not local files. Changes must be deployed first (netlify deploy, git push, etc.) or the re-audit will show the same scores. Check the local file structure and deployment mechanism before declaring the re-audit complete.

## Evidence-Driven Workflow

Every finding must cite evidence:
- ❌ "Meta description is too short" → ✓ "Meta description is 95 chars (needs 120-155)"
- ❌ "Slow page speed" → ✓ "LCP is 4.2s (threshold: 2.5s)"

## Pitfalls

- **www vs non-www mismatch**: The redirect chain and canonical tag must agree. If the server redirects `www` → `non-www`, the canonical must be `non-www`. Check with `redirect_checker.py`.
- **Local vs live disconnect**: Scripts run against the live URL. If you make local file changes, they won't show up in re-audit results until deployed. Always deploy before re-auditing.
- **Duplicate title tags**: Pages generated by LLMs sometimes end up with two `<title>` tags (one from the template, one injected). Always count them during the audit.
- **Icon/asset directory duplicates**: Sites with multiple icon directories (e.g. `Icons/` and `OrangeIcons/`) may have one completely unreferenced. Grep for `src="DirName/"` across all HTML to confirm which is live.
- **Broken OG image refs**: Case studies and blog pages may hardcode OG image paths that point to deleted or moved assets. Check `og:image` and `twitter:image` values point to real files.
- **JSON-LD must be in every HTML file**: Adding it only to the homepage isn't enough — every page the sitemap lists should have its own schema block.
- **Sitemap gets stale**: After adding/removing pages, update `lastmod` dates to the current date.

## Source

Upstream: https://github.com/Bhanunamikaze/Agentic-SEO-Skill