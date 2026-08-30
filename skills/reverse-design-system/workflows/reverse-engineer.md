# Reverse Engineer Workflow

This is the complete execution playbook for reverse-engineering a website's design into a Design Library. Follow every phase in order. Do not skip gates. Each phase builds on evidence from the previous one.

## Prerequisites

Before starting, confirm all of the following:

* **Target URL** — the full homepage URL to reverse-engineer (e.g., `https://endfield.hypergryph.com/`).
* **Output directory** — `{workspace}/.design_library/{BrandName}/`. Where the final Design Library lives.
* **Temp directory** — `{tmp_dir}/{BrandName}/`. All intermediate artifacts go here. Cleaned up at the end.
* **Scripts directory** — `scripts/` directory within this repository. Contains Node.js `.mjs` scripts for token generation, validation, and UIKit planning. All `scripts/*.mjs` references use this path.
* **Browser automation available** — confirm the `browser_use` subagent (or `browser_navigate` / `browser_evaluate` / `browser_take_screenshot` tools) is available. This is the **PRIMARY** extraction method. If unavailable, the workflow falls back to `curl` + CSS regex only (lower quality — see Troubleshooting).

> **Decision point:** If any prerequisite is missing, stop and ask the user. Do not guess the URL or brand name.

***

## Phase 0: Extract (Browser-Primary)

**Goal:** Render the target website in a real browser, extract real computed styles, DOM structure, component boundaries, layout systems, and assets. Then supplement with CSS file regex extraction. Merge both into a unified key findings report.

Phase 0 is split into: **0A** (browser, primary), **0B** (CSS files, supplement), **0C** (merge & summarize).

### Phase 0A — Browser Extraction (PRIMARY)

**Read `file-specs/browser-extraction.md`** for exact JavaScript snippets. Execute via `browser_use` subagent or direct `browser_navigate` + `browser_evaluate` + `browser_take_screenshot` calls.

#### Step 0A.1 — Navigate & Render

```
browser_navigate → <target-url>
browser_wait_for → "networkidle"
browser_evaluate → scroll-to-bottom script (trigger lazy-loaded content)
browser_wait_for → 2000ms (final render settle)
```

Navigate to the target URL, wait for `networkidle`, scroll the entire page to trigger lazy-loaded content, then wait 2 seconds for final render settle.

> **SPA detection:** If the page renders as a bare `<div id="root">` shell and never populates, see Troubleshooting: JS-rendered page with no SSR.

#### Step 0A.2 — Capture Screenshots

```
browser_take_screenshot → { fullPage: true,  path: "{tmp_dir}/{BrandName}/screenshot-full.png" }
browser_take_screenshot → { fullPage: false, path: "{tmp_dir}/{BrandName}/screenshot-viewport.png" }
```

Screenshots serve as visual reference for Phase 1 design decisions, Phase 3 component variant selection, and Phase 5 UIKit validation.

#### Step 0A.3 — Extract Computed Styles (Core)

Run the computed-style extraction JavaScript from **`browser-extraction.md` Step 0A.3** via `browser_evaluate`. Walks every element, calls `getComputedStyle()`, tallies frequencies for 20+ property categories (colors, fonts, font sizes, font weights, border radius, box shadows, clip-paths, backdrop filters, transitions, paddings, gaps, z-indices, etc.).

Save to `{tmp_dir}/{BrandName}/phase0a-computed-styles.json`.

#### Step 0A.4 — Extract DOM Components

Run the DOM component identification JavaScript from **`browser-extraction.md` Step 0A.4**. Matches elements against the 6 standard component selector patterns (Navigation, Button, Card, Input, Badge, CTA Link), extracts computed traits, dimensions, and children.

**Read `file-specs/dom-component-mapping.md`** for exact selector lists, visibility filters, per-component computed properties, and variant classification rules (button → primary/outline/ghost/text-link; card → elevated/outlined/flat).

Save to `{tmp_dir}/{BrandName}/phase0a-dom-components.json`.

#### Step 0A.5 — Extract Layout Systems

Run the layout extraction JavaScript from **`browser-extraction.md` Step 0A.5**. Captures grid templates (`repeat()`, `minmax()`, `auto-fit`, `auto-fill`), flex direction/wrap/align/gap, and container `max-width` patterns.

Save to `{tmp_dir}/{BrandName}/phase0a-layouts.json`.

#### Step 0A.6 — Extract CSS Custom Properties (Runtime)

Run the `:root` variable extraction JavaScript from **`browser-extraction.md` Step 0A.6**. Reads all `--*` custom properties from `document.documentElement` computed styles at runtime.

#### Step 0A.7 — Extract Assets

Run the asset extraction JavaScript from **`browser-extraction.md` Step 0A.7**.

**Read `file-specs/asset-extraction.md`** for full asset categorization (raster images, SVG icons, fonts, background patterns, favicons, video/media), URL normalization rules, and download rules (do NOT download during extraction — record URLs and metadata only).

Save to `{tmp_dir}/{BrandName}/phase0a-assets.json`.

#### Step 0A.8 — Responsive Breakpoint Testing

Run the responsive analysis JavaScript from **`browser-extraction.md` Step 0A.8** at viewport widths `[1920, 1440, 1024, 768, 375]`. Records layout changes, nav height, and mobile menu appearance per breakpoint.

#### Step 0A.9 — Extract Section Structure

Run the page section extraction JavaScript from **`browser-extraction.md` Step 0A.9**. Maps the vertical sequence of major sections (hero → features → testimonials → CTA → footer) with heights, backgrounds, and max-widths.

### Phase 0B — CSS File Extraction (SUPPLEMENT)

> Supplement to Phase 0A. Only run if CSS files can be fetched via `curl`. For SPAs with no SSR, skip entirely.

#### Step 0B.1 — Fetch HTML & CSS URLs

```powershell
curl -s -L '<target-url>' `
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' `
  -o '{tmp_dir}/{BrandName}/target_page.html'

$html = Get-Content -Path '{tmp_dir}/{BrandName}/target_page.html' -Raw
[regex]::Matches($html, 'href="([^"]*\.css[^"]*)"') |
  ForEach-Object { $_.Groups[1].Value } | Sort-Object -Unique
```

**Fallback — JS bundles for CSS imports:** If no `<link>` tags, search JS bundles: `[regex]::Matches($jsContent, '["''][^"'']*\.css[^"'']*["'']') | ForEach-Object { $_.Groups[0].Value.Trim('"''') }`

> If zero CSS URLs found and the site is an SPA, skip Phase 0B. Browser extraction (0A) already captured computed styles.

#### Step 0B.2 — Download CSS Files

```powershell
$cssUrls = @('https://...', 'https://...')
$idx = 0
foreach ($url in $cssUrls) {
  $idx++
  curl -s -L $url -H 'User-Agent: Mozilla/5.0' -o "{tmp_dir}/{BrandName}/css_$idx.css"
}
```

Verify each file is non-empty.

#### Step 0B.3 — Run Regex Extraction

**Read `file-specs/css-extraction.md`** for exact regex patterns and PowerShell templates for all 18 token categories. Concatenate all `css_*.css` files and extract: colors, fonts, font sizes, borders, border-radius, box-shadow, transitions, backdrop-filter, clip-path, letter-spacing, custom properties, gradients, z-index, padding, gap, `@font-face`, transforms, background patterns.

Save to `{tmp_dir}/{BrandName}/phase0b-css-extraction.json`.

> **What CSS extraction uniquely adds:** `@font-face` src URLs, raw `:root` variable definitions as authored, media query breakpoints, raw gradient strings before browser normalization.

### Phase 0C — Merge & Summarize

Merge Phase 0A (browser) and Phase 0B (CSS) into a unified key findings report.

#### Merge Priority Rules

**Read `file-specs/css-extraction.md` → "Merge Priority" section.** Summary:

1. **Browser computed styles = AUTHORITATIVE.** Actual rendering after cascade + JS.
2. **CSS file declarations = SUPPLEMENTARY.** What was declared in source.
3. **When conflict: browser wins.** Computed > declared.
4. **CSS file extraction adds:** `@font-face` src URLs, `:root` variable definitions, media query breakpoints, raw gradient strings.
5. **Browser extraction adds:** rendered values, DOM structure, component boundaries, asset URLs, layout systems, responsive behavior.

#### Step 0C.1 — Produce Key Findings

Consolidate all extraction data into **14 key findings** (REQUIRED inputs for Phase 1):

1. **Dominant background color** — most frequent `backgroundColor` from browser computed styles.
2. **Primary text color** — most frequent `color` value (excluding background).
3. **Brand accent color(s)** — high-saturation colors on interactive elements. Become `--primary`, `--accent`, `--tertiary`.
4. **Font family names** — from `document.fonts` API + `@font-face`. Identify display vs body vs mono.
5. **Common font-size scale** — from computed `fontSize`. Convert rem→px at 16px root.
6. **Border patterns** — thickness, style, color from computed `border`.
7. **Border-radius values** — scale steps from computed `borderRadius`.
8. **Shadow patterns** — glow (`0 0`) vs drop (offset) from computed `boxShadow`.
9. **Transition durations & easing** — from computed `transition`.
10. **clip-path shapes** — from computed `clipPath` (geometric design language).
11. **Letter-spacing values** — tight (negative) vs loose (positive) from computed `letterSpacing`.
12. **Gradient patterns** — color stops from computed `background` + raw CSS gradient strings.
13. **Layout system** — grid pattern, max content width, nav/button/input heights from `phase0a-layouts.json`.
14. **DOM component map & assets** — 6 component types with real selectors, computed traits, dimensions, variant classifications (`phase0a-dom-components.json`); asset catalog with SVG icons, font URLs, brand gradients, favicon (`phase0a-assets.json`).

Write `=== KEY FINDINGS ===` citing real hex values, font names, layout metrics. Save to `{tmp_dir}/{BrandName}/phase0-key-findings.json`.

### Gate 0

**Read `operation-policies/quality-gates.md` Gate 0 criteria.** Check:

* [ ] Browser extraction completed — `phase0a-computed-styles.json` has content for colors, fonts, font-sizes, shadows, transitions.
* [ ] At least 3 component types identified in DOM (from `phase0a-dom-components.json`).
* [ ] Screenshots captured (full-page + viewport).
* [ ] CSS file extraction completed if CSS was fetchable (may be skipped for SPAs).
* [ ] A dominant background color is identifiable.
* [ ] At least 1 brand accent color is identifiable (distinct from bg/text).
* [ ] At least 1 custom web font identified (via `document.fonts` or `@font-face`), OR system fonts clearly dominant.
* [ ] Key findings JSON saved with all 14 items.

> **If FAIL:** Check Troubleshooting for fallback to curl-only. If curl also fails, ask for an alternative URL or screenshot-based approach.
>
> **If PASS:** Proceed to Phase 1.

***

## Phase 1: Brand Analysis

**Goal:** Build a structured brand profile JSON from Phase 0 key findings — now enriched with DOM component mapping, layout systems, and asset URLs.

### Step 1.1 — Build Brand Profile

**Read `file-specs/brand-profile.md`** for the complete schema and personality keyword bank. Populate a JSON object with: `productType`, `confidence`, `personality`, `language`, `visualTone`, `kitType`, `colorNamingPrefix`, `uiCopySamples`.

Key field rules:
- `confidence`: `high` if browser extraction yielded clear patterns; `medium` if heavily JS-rendered and CSS was sparse.
- `personality`: 3–5 keywords from the keyword bank. Derive from visual evidence.
- `visualTone`: MUST cite real extracted hex values and font names. Also reference layout system, component patterns, and asset observations. This is the creative north star for all token generation.
- `colorNamingPrefix`: lowercase brand name (CSS variable prefix).
- `uiCopySamples`: 5–10 actual text strings from the DOM — read from `phase0a-dom-components.json` component text fields. NEVER fabricate.

See `examples/brand-profile-example.json` for a real reference.

### Step 1.2 — Save Brand Profile

Save to `{tmp_dir}/{BrandName}/phase2-brand-analyst.json`. (Filename is `phase2-brand-analyst.json` — not `phase1` — because downstream generator scripts expect this exact name. Non-negotiable.)

### Gate 1

Check: all required fields present; `visualTone` cites ≥2 real hex values; `personality` has 3–5 keywords; `uiCopySamples` has 5+ items from actual DOM content; valid JSON (no BOM, no trailing commas).

> **If FAIL:** Fix the specific failing field. Usually visualTone is too vague or uiCopySamples are fabricated.
>
> **If PASS:** Proceed to Phase 2.

***

## Phase 2: Token Generation

**Goal:** Generate `colors_and_type.css` with all design tokens, then derive `css.json`.

### Step 2.1 — Dispatch Token Sub-Agent

**Read `operation-policies/agent-dispatch.md`** for the Phase 2 dispatch template.

Dispatch 1 sub-agent (`general_purpose_task`) with these inputs:

* **Read:** `file-specs/token-css.md`, `{tmp_dir}/{BrandName}/phase2-brand-analyst.json`, `{tmp_dir}/{BrandName}/phase0a-computed-styles.json` (browser-extracted computed styles — AUTHORITATIVE), `{tmp_dir}/{BrandName}/phase0b-css-extraction.json` (CSS file extraction — supplementary `@font-face` src URLs, if available).
* **Write:** `{output_dir}/colors_and_type.css`.
* **CSS budget:** ≤250 lines.

The sub-agent must generate: full 10-step color scales (50–900) per group (primary, accent, tertiary, neutral, surface), anchored at the real browser-computed brand color; typography tokens using real font family names from `document.fonts` / `@font-face`; spacing/sizing/radius/shadow/transition tokens from computed style frequency data; dark/light theme blocks; `/* AI-generated */` markers on AI-generated values; `@group-priority` comment; `@primary`/`@accent` markers; `@import` for Google Fonts only (no `@import` for `@font-face` fonts).

### Step 2.2 — Generate css.json

```powershell
node "scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

### Gate 2

Check: `colors_and_type.css` ≤250 lines; all 10-step scales present; `@primary`/`@accent` markers correct; `@group-priority` present; no `@import` for `@font-face` fonts; `css.json` generated; all theme-block vars defined elsewhere in file.

> **If FAIL (undefined CSS vars):** Add missing definitions, re-run `css-to-json.mjs`.
>
> **If PASS:** Proceed to Phase 3.

***

## Phase 3: Component Contracts & Previews

**Goal:** Define 6 standard component contracts from real DOM component data, generate HTML previews, extract `components.css`.

### Step 3.1 — Create Component Index

Write `{output_dir}/components/index.json` with the 6 standard components. Populate `priorityHint` and `keyInsightSeed` from **real DOM component data** in `{tmp_dir}/{BrandName}/phase0a-dom-components.json` — actual computed traits (clip-path, text-transform, shadow, border-radius, backdrop-filter, height, etc.).

### Step 3.2 — Write Component Contracts

For each component, write `{output_dir}/components/{slug}.json`. **Read `file-specs/component-contract.md`** for the schema.

**Key change:** Contracts now reference **real DOM component data** — actual selectors that matched, actual computed traits, actual dimensions (width/height in px). Variant classifications come from `dom-component-mapping.md` rules (button → primary/outline/ghost/text-link; card → elevated/outlined/flat). Include: `slug`, `schemaVersion: 2`, `name`, `category`, `sourceKind`, `confidence`, `semanticTypeCandidates`, `variantDimensions`, `representativeVariants` (3–4 with `traits` using `var()` refs), `anatomy`, `structurePatterns`, `usageHints`, `doNotInvent`, `unknowns`.

**All trait values MUST use `var()` references** to tokens in `colors_and_type.css`. Never hardcode hex. See `examples/component-contract-example.json`.

### Step 3.3 — Generate Component Previews (Parallel)

**Read `agent-dispatch.md`** Phase 3 template. Dispatch 3 sub-agents in parallel:

| Batch | Components | Output files |
|---|---|---|
| 1 | `button` + `card` | `preview/component-button.html`, `preview/component-card.html` |
| 2 | `input` + `badge` | `preview/component-input.html`, `preview/component-badge.html` |
| 3 | `cta-link` + `navigation` | `preview/component-cta-link.html`, `preview/component-navigation.html` |

Each reads the component JSON contract(s), `colors_and_type.css`, and **the original screenshots** (`screenshot-full.png`) as visual reference. Writes self-contained HTML previews. **CSS link (REQUIRED):** `<link rel="stylesheet" href="../colors_and_type.css">`

### Step 3.4 — Extract Components CSS

```powershell
node "scripts/extract-components-css.mjs" "{output_dir}"
```

### Gate 3

Check: 6 component JSONs + 6 preview HTMLs exist; `components.css` non-empty; all previews have CSS link; all JSONs have `slug` matching filename; all trait values use `var()` (no hardcoded hex); contracts cite real DOM data from Phase 0A.

> **If PASS:** Proceed to Phase 4.

***

## Phase 4: Documentation & UIKit

**Goal:** Generate UIKit plan, documentation files, and the Marketing UI Kit HTML page.

### Step 4.1 — Generate UIKit Plan

```powershell
node "scripts/generate-uikit-plan.mjs" `
  --component-index "{output_dir}/components/index.json" `
  --components-dir "{output_dir}/components" `
  --brand-data "{tmp_dir}/{BrandName}/phase2-brand-analyst.json" `
  --available-vars "{output_dir}/colors_and_type.css" `
  --components-css "{output_dir}/components.css" `
  --out "{output_dir}/uikit-plan.json"

node "scripts/validate-uikit-plan.mjs" `
  --plan "{output_dir}/uikit-plan.json" `
  --component-index "{output_dir}/components/index.json" `
  --components-dir "{output_dir}/components" `
  --out "{output_dir}/uikit-plan.json"
```

### Step 4.2 — Generate Documentation (Parallel)

**Read `agent-dispatch.md`** Phase 4a template. Dispatch 2 sub-agents:

**Sub-agent A:** `{output_dir}/SKILL.md` (YAML frontmatter + essentials), `{output_dir}/library-consumption.json` (consumption order).
**Sub-agent B:** `{output_dir}/README.md` (Overview, Content Fundamentals, Visual Foundations, Component Patterns, Index, Caveats).

Both read `file-specs/documentation.md` and `colors_and_type.css`. Neither reads individual component JSONs.

### Step 4.3 — Generate UIKit

**Read `agent-dispatch.md`** Phase 4b template. Dispatch 1 sub-agent that reads `file-specs/uikit.md`, all 6 component contracts, brand profile, `colors_and_type.css`, `uikit-plan.json`, and **the original website screenshots** (`screenshot-full.png`, `screenshot-viewport.png`) as visual reference for matching layout and component placement. Writes `{output_dir}/ui_kits/marketing/index.html`.

**CSS link (REQUIRED):** `<link rel="stylesheet" href="../../colors_and_type.css">`

### Gate 4

Check: `README.md` has 6 prose sections; `SKILL.md` has YAML frontmatter; `library-consumption.json` valid; `uikit-plan.json` passed validation; `ui_kits/marketing/index.html` exists with correct CSS link.

> **If PASS:** Proceed to Phase 5.

***

## Phase 5: Validate & Deploy

**Goal:** Strip BOMs, regenerate css.json, validate, optional screenshot comparison, clean up, push to git.

### Step 5.1 — Strip BOMs (ALL files)

**Read `quality-gates.md`** for the BOM-stripping script. Walk all subdirectories in both `{output_dir}` and `{tmp_dir}/{BrandName}`:

```powershell
node -e "const fs=require('fs');const dir='{output_dir}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=require('path').join(d,e.name);if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',p)}}})}walk(dir)"

node -e "const fs=require('fs');const dir='{tmp_dir}/{BrandName}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=require('path').join(d,e.name);if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',p)}}})}walk(dir)"
```

### Step 5.2 — Regenerate css.json

```powershell
node "scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

### Step 5.3 — Validate

```powershell
node "scripts/validate-design-library-output.mjs" "{output_dir}"
```

Re-run until exit code 0. Common fixes: undefined CSS vars → add to `colors_and_type.css`; slug mismatches → fix `slug` field; invalid JSON → re-run BOM strip.

### Step 5.4 — Optional: Screenshot Comparison

Take a screenshot of the generated UIKit page via `browser_take_screenshot`. Compare visually against original website screenshots (`screenshot-full.png`) from Phase 0A. Check: color palette matches, typography ballpark, component arrangement follows same section hierarchy, border-radius/shadow/clip-path patterns consistent.

> Optional validation. If UIKit diverges significantly, note in README "Caveats". Do not block on minor visual differences — the UIKit is an interpretation, not a pixel-perfect clone.

### Step 5.5 — Clean Up

```powershell
Remove-Item -Recurse -Force "{output_dir}/agent-reports" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "{tmp_dir}/{BrandName}" -ErrorAction SilentlyContinue
Remove-Item -Force "{workspace}/ef_*.css","{workspace}/css_*.css","{workspace}/target_page.html" -ErrorAction SilentlyContinue
```

### Step 5.6 — Git Commit & Push

**Read `operation-policies/git-deploy.md`** for conventions.

```powershell
cd {workspace}
git add --all
git commit -m "feat: add {BrandName} design system reverse-engineered from {source-domain}"
git push origin main
```

> If `git push` fails (auth or branch protection), inform the user. Files are saved locally and complete.

### Gate 5 (Final)

Check: validator exit code 0; no temp files remain; `git push` succeeded (or user informed); all deliverables exist in `{output_dir}` (`colors_and_type.css`, `css.json`, `components/index.json`, `components/{slug}.json` ×6, `components.css`, `preview/component-{slug}.html` ×6, `README.md`, `SKILL.md`, `library-consumption.json`, `uikit-plan.json`, `ui_kits/marketing/index.html`).

**Report to user:** Files generated, tokens extracted, components analyzed, UIKit page generated, git commit hash.

***

## Troubleshooting

### browser_use fails → fall back to curl + css-extraction.md only

If the `browser_use` subagent is unavailable, crashes, or the site blocks automated browsers:

1. Skip Phase 0A entirely. Run Phase 0B (CSS file extraction) as the sole method using `curl` + regex patterns in `css-extraction.md`.
2. Set `confidence: "medium"` in the brand profile — CSS-only extraction misses runtime-computed values, DOM structure, and component boundaries.
3. Component contracts (Phase 3) will lack real DOM data — derive `keyInsightSeed` from CSS patterns only.
4. Skip Step 5.4 (screenshot comparison) — no screenshots to compare against.

### JS-rendered page with no SSR → browser is the only option

If the site is a client-rendered SPA (Next.js, Nuxt, Vue) and the initial HTML has no `<link rel="stylesheet">` tags:

1. **curl won't work.** The HTML is a bare `<div id="root">` / `<div id="__next">` shell. CSS is injected at runtime by JavaScript.
2. **Browser extraction (Phase 0A) is the only option.** The browser renders the page, JS injects styles, and `getComputedStyle()` captures the real values.
3. Skip Phase 0B (no CSS files to fetch via curl).
4. The `document.fonts` API will still report loaded fonts. `@font-face` src URLs may be missing (note as `unknowns` in key findings).

### BOM causes JSON parse failures

Sub-agents on Windows may produce UTF-8 BOM (`EF BB BF`). This breaks `JSON.parse()` and the CSS-to-JSON parser. **Fix:** Run the BOM-stripping script (Step 5.1), then re-run the failing script.

### Validator reports undefined CSS vars

Read validator output for exact variable names. Add missing definitions to `colors_and_type.css` (derive from Phase 0 data). Regenerate `css.json` (Step 5.2), re-run validator. Common gap: theme blocks reference `--scale-*` tokens the sub-agent forgot to define.

### UIKit plan generation fails

Strip BOM from `components/index.json`. Ensure every component JSON has `slug` matching filename. Verify `phase2-brand-analyst.json` is valid JSON. Re-run generator.

### Sub-agent can't write report

The **HTML deliverable** (preview/UIKit page) is critical — it must write to `{output_dir}/`. The **report JSON** is optional metadata. If the report fails but the HTML succeeds, proceed.
