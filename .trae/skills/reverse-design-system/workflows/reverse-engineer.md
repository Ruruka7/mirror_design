# Reverse Engineer Workflow

This is the complete execution playbook for reverse-engineering a website's design into a Design Library. Follow every phase in order. Do not skip gates. Each phase builds on evidence from the previous one.

## Prerequisites

Before starting, confirm all of the following with the user or from the task context:

* **Target URL** — the full homepage URL to reverse-engineer (e.g., `https://endfield.hypergryph.com/`).

* **Output directory** — `{workspace}/.design_library/{BrandName}/`. This is where the final Design Library lives. The `{BrandName}` folder name should match the brand's English name in PascalCase or lowercase (e.g., `endfield`, `Arknights`).

* **Temp directory** — `{tmp_dir}/{BrandName}/`. All intermediate artifacts (downloaded HTML, CSS bundles, extraction reports, brand profile) go here. This directory is cleaned up at the end.

* **`skill_base`** **path** — the absolute path to the `design-library-creator` skill directory. All `scripts/*.mjs` references use this path. Confirm it exists before Phase 2.

> **Decision point:** If any prerequisite is missing, stop and ask the user. Do not guess the URL or brand name.

***

## Phase 0: Fetch & Extract

**Goal:** Download the target website's HTML and all CSS bundles, then run a comprehensive regex extraction to capture every design-relevant token value. This is the evidence base for all downstream phases.

### Step 0.1 — Fetch HTML

Download the target page HTML using curl with a browser user-agent header (some sites block default curl agents).

```powershell
$targetUrl = '<target-url>'
curl -s -L $targetUrl `
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36' `
  -o '{tmp_dir}/{BrandName}/target_page.html'
```

**Expected output:** A file `target_page.html` in the temp directory (typically 50KB–2MB for a marketing landing page).

**What to look for in the HTML:**

* `<link rel="stylesheet" href="...">` tags — direct CSS file references.

* `_next/static/css/` paths — Next.js CSS bundle references.

* `<style>` blocks — inline CSS (rare on marketing sites but possible).

* CDN domains in href attributes (e.g., `web.hycdn.cn`, `cdn.jsdelivr.net`).

> **Decision point (SPA detection):** If the HTML is mostly empty (under 5KB) or contains only a `<div id="root">` / `<div id="__next">` with no CSS links, the site is a client-rendered SPA. See the [Troubleshooting: SPA sites](#spa-sites-with-no-css-in-html) section — you may need to use a browser to fetch the rendered DOM.

### Step 0.2 — Extract CSS URLs

Parse the downloaded HTML for all CSS stylesheet references:

```powershell
$html = Get-Content -Path '{tmp_dir}/{BrandName}/target_page.html' -Raw
[regex]::Matches($html, 'href="([^"]*\.css[^"]*)"') |
  ForEach-Object { $_.Groups[1].Value } |
  Sort-Object -Unique
```

**Expected output:** A list of relative or absolute CSS URLs, one per line.

**Common CDN patterns to check:**

* `https://web.hycdn.cn/{brand}/official-v*/_next/static/css/*.css` — Hypergryph CDN.

* `/_next/static/css/*.css` — Next.js local bundle (prepend the origin domain).

* `https://cdn.jsdelivr.net/...` — jsdelivr CDN.

* `https://fonts.googleapis.com/css?...` — Google Fonts (skip these, they are font imports, not style bundles).

**Fallback: check JS bundles for CSS imports.** If the HTML has no `<link>` stylesheet tags but has `<script src="...">` tags, the CSS may be loaded dynamically by JS. Download a JS bundle and search for `.css` string literals:

```powershell
[regex]::Matches($jsContent, '["''][^"'']*\.css[^"'']*["'']') |
  ForEach-Object { $_.Groups[0].Value.Trim('"''') }
```

> **Decision point:** If zero CSS URLs are found after all methods, Gate 0 will fail. See Troubleshooting.

### Step 0.3 — Download CSS Files

Download each CSS file found in Step 0.2. Use a `foreach` loop with a consistent naming convention:

```powershell
$cssUrls = @('https://...', 'https://...')
$idx = 0
foreach ($url in $cssUrls) {
  $idx++
  $outFile = "{tmp_dir}/{BrandName}/css_$idx.css"
  curl -s -L $url `
    -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' `
    -o $outFile
  Write-Host "Downloaded: $outFile ($((Get-Item $outFile).Length) bytes)"
}
```

**Naming convention:** All temp CSS files are prefixed `css_` followed by an index number (e.g., `css_1.css`, `css_2.css`). This avoids filename collisions from CDN bundles that often have hash-based names.

**Expected output:** Multiple `css_*.css` files in the temp directory. Verify each file is non-empty (a 0-byte file means the download failed — retry or check the URL).

### Step 0.4 — Extract All Tokens

Run ONE comprehensive PowerShell script that extracts ALL 18 token categories from the downloaded CSS. Before writing this script, **read** **`file-specs/css-extraction.md`** for the exact category list and regex patterns.

The script must:

1. **Concatenate** all `css_*.css` files into a single `$allContent` string.
2. **Run regex** for each of the 18 categories using the patterns from `css-extraction.md`.
3. **Group-Object** the results for each category.
4. **Sort-Object Count -Descending** to rank by frequency.
5. **Output summary tables** to the console (top 20 per category).
6. **Save** the full output to `{tmp_dir}/{BrandName}/phase0-css-extraction.txt`.

```powershell
# --- Phase 0 Comprehensive CSS Extraction Script ---
$cssDir = '{tmp_dir}/{BrandName}'
$cssFiles = Get-ChildItem -Path $cssDir -Filter 'css_*.css'
$allContent = ($cssFiles | ForEach-Object { Get-Content $_.FullName -Raw }) -join "`n"
$report = @()
$report += "=== PHASE 0: CSS EXTRACTION SUMMARY ==="
$report += "Source: <target-url>"
$report += "Date: $(Get-Date -Format 'yyyy-MM-dd')"
$report += "CSS Files: $($cssFiles.Count) bundles"
$report += ""

# Helper: extract + group + sort + format a category
function Extract-Category($label, $pattern, $groupIdx, $top = 20) {
  $matches = [regex]::Matches($allContent, $pattern, 'IgnoreCase')
  $values = if ($groupIdx -gt 0) {
    $matches | ForEach-Object { $_.Groups[$groupIdx].Value.Trim() }
  } else {
    $matches | ForEach-Object { $_.Value }
  }
  $grouped = $values | Group-Object | Sort-Object Count -Descending
  $lines = @()
  $lines += "=== $label (Top $top) ==="
  $grouped | Select-Object -First $top | ForEach-Object {
    $lines += ('{0,-16} {1,5}' -f $_.Name, $_.Count)
  }
  $lines += ""
  return $lines
}

$report += Extract-Category 'COLORS (hex + rgba)' '(#[0-9a-fA-F]{3,8}\b|rgba?\([^)]+\))' 0 20
$report += Extract-Category 'FONTS' 'font-family\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'FONT SIZES' 'font-size\s*:\s*([\d.]+(px|rem|em))' 1 15
$report += Extract-Category 'FONT WEIGHTS' 'font-weight\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'BORDERS' 'border\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'BORDER-RADIUS' 'border-radius\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'BOX-SHADOW' 'box-shadow\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'TEXT-SHADOW' 'text-shadow\s*:\s*([^;}{]+)' 1 5
$report += Extract-Category 'LETTER-SPACING' 'letter-spacing\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'LINE-HEIGHT' 'line-height\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'TRANSITIONS' 'transition\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'TRANSFORMS' 'transform\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'BACKDROP-FILTER' 'backdrop-filter\s*:\s*([^;}{]+)' 1 5
$report += Extract-Category 'CLIP-PATH' 'clip-path\s*:\s*([^;}{]+)' 1 5
$report += Extract-Category 'GRADIENTS' '(linear-gradient|radial-gradient|repeating-linear-gradient)\([^)]+\)' 0 5
$report += Extract-Category 'Z-INDEX' 'z-index\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'PADDING' 'padding\s*:\s*([^;}{]+)' 1 10
$report += Extract-Category 'GAP' 'gap\s*:\s*([^;}{]+)' 1 10

# @font-face declarations
$report += "=== @FONT-FACE ==="
[regex]::Matches($allContent, '@font-face\s*\{[^}]*font-family\s*:\s*[''"]?([^''"};]+)') |
  ForEach-Object { $_.Groups[1].Value.Trim() } | Sort-Object -Unique |
  ForEach-Object { $report += $_ }

# CSS custom properties
$report += ""
$report += "=== CSS CUSTOM PROPERTIES (Top 20) ==="
[regex]::Matches($allContent, '(--[a-zA-Z0-9-]+)\s*:' ) |
  ForEach-Object { $_.Groups[1].Value } | Group-Object | Sort-Object Count -Descending |
  Select-Object -First 20 | ForEach-Object { $report += ('{0,-40} {1,5}' -f $_.Name, $_.Count) }

$out = $report -join "`n"
$out | Set-Content -Path "$cssDir\phase0-css-extraction.txt" -Encoding UTF8
Write-Host $out
```

**Expected output:** A detailed text report saved to `phase0-css-extraction.txt` and echoed to the console. The report has top values for every token category, sorted by frequency. See `examples/extraction-output-example.txt` for what a real output looks like.

### Step 0.5 — Summarize Findings

Read the extraction report and record these **12 key findings** — they are REQUIRED inputs for Phase 1:

1. **Dominant background color** — the most frequently occurring color (usually a dark or light base).
2. **Primary text color** — the 2nd most frequent, usually white or near-white (the inverse of the background).
3. **Brand accent color(s)** — distinct from bg/text, high saturation (e.g., hazard yellow, neon green). These become `--primary`, `--accent`, `--tertiary`.
4. **Font family names** — from `@font-face` declarations and `font-family` values. Identify display vs body vs mono.
5. **Common font-size scale** — convert rem to px at 16px root (1rem = 16px). These become the typography scale tokens.
6. **Border patterns** — thickness, style, color frequency.
7. **Border-radius values** — look for scale steps (0, 4px, 8px, 50%).
8. **Shadow patterns** — distinguish glow shadows (colored, `0 0`) from drop shadows (offset, gray).
9. **Transition durations** — common timing values (0.2s, 0.3s) and easing functions (ease, ease-in-out).
10. **clip-path shapes** — polygon shapes indicate geometric/angular design language.
11. **Letter-spacing values** — negative (tight) vs positive (loose) tracking.
12. **Gradient patterns** — extract color stops; multi-color brand gradients are critical signature elements.

Write a `=== KEY FINDINGS ===` section at the end of the extraction report with all 12 items, citing real hex values and font names.

### Gate 0

**Read** **`operation-policies/quality-gates.md`** **Gate 0 criteria.** Check:

* [ ] At least 1 CSS file was successfully downloaded (non-empty).

* [ ] Extraction report exists and has content for colors, fonts, font-sizes, shadows, transitions.

* [ ] A dominant background color is identifiable.

* [ ] At least 1 brand accent color is identifiable (distinct from bg/text).

* [ ] At least 1 custom web font (@font-face) is identified, OR system fonts are clearly dominant.

> **If FAIL:** Stop. Inform the user that CSS extraction did not yield enough evidence. Ask for an alternative URL, a different page on the same site, or a screenshot-based approach.
>
> **If PASS:** Proceed to Phase 1.

***

## Phase 1: Brand Analysis

**Goal:** Build a structured brand profile JSON that captures the design language in human-readable terms. This file feeds into the design-library-creator pipeline as the creative brief.

### Step 1.1 — Build Brand Profile

**Read** **`file-specs/brand-profile.md`** for the complete schema and the keyword bank for `personality` values.

Construct a JSON object with all fields, populating from Phase 0 evidence:

```json
{
  "productType": "Marketing/Landing",
  "confidence": "high",
  "personality": ["industrial", "post-apocalyptic", "high-energy", "geometric", "aggressive"],
  "language": "zh",
  "visualTone": "Industrial post-apocalyptic dark UI with hazard-yellow #fffa00 primary, neon green #00ffa2 secondary, magenta #ff1aac tertiary, built on near-black #191919 ground with #fff text. Geometric clip-path cutouts, diagonal hazard stripe patterns, colored glow shadows.",
  "kitType": "marketing",
  "colorNamingPrefix": "brandname",
  "uiCopySamples": ["actual nav items", "CTA button text", "section headings", "footer links", "brand slogans"]
}
```

**Field rules:**

* `productType`: almost always `Marketing/Landing` for public-facing websites.

* `confidence`: `high` if CSS extraction yielded clear, consistent patterns. `medium` if the site is heavily JS-rendered and CSS was sparse.

* `personality`: 3–5 keywords from the keyword bank in `brand-profile.md`. Derive from visual evidence — dark industrial → "industrial", "post-apocalyptic"; clean minimal → "minimal", "editorial".

* `visualTone`: MUST cite real extracted hex values and font names. This is the creative north star for all token generation.

* `colorNamingPrefix`: lowercase brand name, used as the CSS variable prefix (e.g., `endfield`).

* `uiCopySamples`: 5–10 actual text strings found on the website — nav items, CTA labels, headings, footer text. NEVER fabricate these. Read them from the HTML.

See `examples/brand-profile-example.json` for a real reference.

### Step 1.2 — Save Brand Profile

Save the brand profile to:

```
{tmp_dir}/{BrandName}/phase2-brand-analyst.json
```

> **Note:** The filename is `phase2-brand-analyst.json` (not `phase1`) because the downstream `design-library-creator` scripts expect this exact filename. This is a non-negotiable naming convention.

### Gate 1

**Read** **`quality-gates.md`** **Gate 1 criteria.** Check:

* [ ] All required fields present: `productType`, `confidence`, `personality`, `language`, `visualTone`, `kitType`, `colorNamingPrefix`, `uiCopySamples`.

* [ ] `visualTone` cites at least 2 real hex values from Phase 0 extraction.

* [ ] `personality` has 3–5 keywords from the keyword bank.

* [ ] `uiCopySamples` has 5+ items, all from actual page content.

* [ ] File is valid JSON (no BOM, no trailing commas).

> **If FAIL:** Fix the specific failing field. Usually this means visualTone is too vague or uiCopySamples are fabricated.
>
> **If PASS:** Proceed to Phase 2.

***

## Phase 2: Token Generation

**Goal:** Generate `colors_and_type.css` with all design tokens (color scales, typography, spacing, shadows, etc.), then derive `css.json` from it.

### Step 2.1 — Dispatch Token Sub-Agent

**Read** **`operation-policies/agent-dispatch.md`** for the Phase 2 sub-agent dispatch template.

Dispatch 1 sub-agent (`general_purpose_task`) with these instructions:

* **Read:** `file-specs/token-css.md` (the token CSS specification from design-library-creator).

* **Read:** `{tmp_dir}/{BrandName}/phase2-brand-analyst.json` (the brand profile from Phase 1).

* **Write:** `{output_dir}/colors_and_type.css`.

* **CSS budget:** ≤250 lines. If the sub-agent exceeds this, it must consolidate tokens.

**The sub-agent must generate:**

* Full 10-step color scales (50–900) for each color group (primary, accent, tertiary, neutral, surface).

* All scales anchored at the real extracted brand color (e.g., if `--primary` is `#fffa00`, the 500 step IS `#fffa00`).

* Typography tokens using real font family names from `@font-face`.

* Spacing, sizing, radius, shadow, transition tokens derived from Phase 0 frequency data.

* Dark/light theme blocks as appropriate (`.dark`, `.light`).

* Every AI-generated color value marked with `/* AI-generated */`.

* `@group-priority` comment at the top defining CSS insertion order.

* `@primary`, `@accent` markers on the brand color variables.

* `@import` for Google Fonts only (custom web fonts are loaded by the consuming page, not imported in the CSS).

* NO `@import` for `@font-face` fonts — they are referenced by name only.

### Step 2.2 — Generate css.json

After the sub-agent writes `colors_and_type.css`, derive the JSON token map:

```powershell
node "{skill_base}/scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

**Expected output:** A `css.json` file containing all CSS variables parsed into a structured JSON map. This file is consumed by downstream scripts and the validator.

### Gate 2

**Read** **`quality-gates.md`** **Gate 2 criteria.** Check:

* [ ] `colors_and_type.css` exists and is ≤250 lines.

* [ ] All 10-step scales (50–900) present for each color group.

* [ ] `@primary` and `@accent` markers present on the correct variables.

* [ ] `@group-priority` comment present at the top.

* [ ] No `@import` for `@font-face` fonts (Google Fonts `@import` is OK).

* [ ] `css.json` generated successfully without errors.

* [ ] All CSS variables used in theme blocks (`.dark`/`.light`) are defined elsewhere in the file.

> **If FAIL (undefined CSS vars in theme blocks):** Add the missing variable definitions to `colors_and_type.css`, then re-run `css-to-json.mjs`. Common gap: theme override blocks reference `--scale-*` tokens that the sub-agent forgot to define.
>
> **If PASS:** Proceed to Phase 3.

***

## Phase 3: Component Contracts & Previews

**Goal:** Define 6 standard component contracts as JSON, then generate HTML preview pages for each, and extract the combined `components.css`.

### Step 3.1 — Create Component Index

Write `{output_dir}/components/index.json` with the 6 standard components:

```json
{
  "schemaVersion": 2,
  "sourceKind": "from-scratch",
  "libraryName": "{BrandName}",
  "components": [
    { "slug": "button", "name": "Button", "category": "action", "confidence": "high", "variantCount": 5, "priorityHint": "Primary CTA, hazard yellow fill", "keyInsightSeed": "Angular clip-path corner, uppercase, tight tracking" },
    { "slug": "card", "name": "Card", "category": "surface", "confidence": "high", "variantCount": 3, "priorityHint": "Dark surface card with border", "keyInsightSeed": "Near-black bg, thin border, glow on hover" },
    { "slug": "input", "name": "Input", "category": "form", "confidence": "medium", "variantCount": 2, "priorityHint": "Minimal dark input", "keyInsightSeed": "Transparent bg, bottom border focus state" },
    { "slug": "badge", "name": "Badge", "category": "status", "confidence": "high", "variantCount": 3, "priorityHint": "Small status indicators", "keyInsightSeed": "Pill shape, brand color fills" },
    { "slug": "cta-link", "name": "CTA Link", "category": "action", "confidence": "high", "variantCount": 2, "priorityHint": "Text link with underline animation", "keyInsightSeed": "Uppercase, tight tracking, color transition on hover" },
    { "slug": "navigation", "name": "Navigation", "category": "shell", "confidence": "high", "variantCount": 1, "priorityHint": "Top horizontal nav bar", "keyInsightSeed": "Horizontal flex, uppercase links, active indicator" }
  ]
}
```

**Schema:** `schemaVersion` (int), `sourceKind` (always `from-scratch` for reverse-engineered), `libraryName` (brand name), `components[]` array. Each component has: `slug`, `name`, `category`, `confidence`, `variantCount`, `priorityHint`, `keyInsightSeed`.

### Step 3.2 — Write Component Contracts

For each of the 6 components, write `{output_dir}/components/{slug}.json`.

**Read** **`file-specs/component-contract.md`** for the complete schema.

Each contract must include:

* `slug` — matches the filename (e.g., `button` → `button.json`).

* `schemaVersion` — `2`.

* `name`, `category`, `sourceKind`, `confidence`.

* `semanticTypeCandidates` — possible semantic type labels.

* `variantDimensions` — dimensions that define variants (e.g., `variant: [primary, accent, outline]`, `size: [sm, md, lg]`).

* `representativeVariants` — 3–4 key variants, each with `name`, `whySelected`, `traits` (CSS property → value using `var()` references), `childrenDigest`.

* `anatomy` — structural element list (e.g., `["root", "label", "clip-path-corner"]`).

* `structurePatterns` — layout patterns observed.

* `usageHints` — `priorityHint` and `a11y` guidance.

* `doNotInvent` — things NOT to include (e.g., `["icons inside button", "loading spinners"]`).

* `unknowns` — acknowledged gaps (e.g., `["exact clip-path angle degree"]`).

**All trait values MUST use** **`var()`** **references** to tokens defined in `colors_and_type.css`. Never hardcode hex values in a contract — always reference `var(--primary)`, `var(--color-on-primary)`, etc.

See `examples/component-contract-example.json` for a real reference.

### Step 3.3 — Generate Component Previews (Parallel)

**Read** **`agent-dispatch.md`** for the Phase 3 parallel dispatch template.

Dispatch 3 sub-agents in parallel, 2 components each:

| Sub-agent | Components                | Output files                                                           |
| --------- | ------------------------- | ---------------------------------------------------------------------- |
| Batch 1   | `button` + `card`         | `preview/component-button.html`, `preview/component-card.html`         |
| Batch 2   | `input` + `badge`         | `preview/component-input.html`, `preview/component-badge.html`         |
| Batch 3   | `cta-link` + `navigation` | `preview/component-cta-link.html`, `preview/component-navigation.html` |

Each sub-agent:

* Reads the component JSON contract(s) it is assigned.

* Reads `colors_and_type.css` for available CSS variable definitions.

* Writes a self-contained HTML preview page showing all variants of the component(s).

* **CSS link (REQUIRED in every preview):**

  ```html
  <link rel="stylesheet" href="../colors_and_type.css">
  ```

  The path is `../colors_and_type.css` because previews live in `preview/` and the CSS lives in the output root.

### Step 3.4 — Extract Components CSS

After all 6 previews are generated, extract the inline CSS from all preview HTMLs into a single `components.css`:

```powershell
node "{skill_base}/scripts/extract-components-css.mjs" "{output_dir}"
```

**Expected output:** `{output_dir}/components.css` containing all component-specific styles extracted from the preview HTML files. This file is consumed by the UIKit plan generator in Phase 4.

### Gate 3

Check:

* [ ] 6 component JSON files exist in `components/` (one per slug).

* [ ] 6 preview HTML files exist in `preview/` (one per slug).

* [ ] `components.css` exists and is non-empty.

* [ ] All preview HTMLs have `<link rel="stylesheet" href="../colors_and_type.css">`.

* [ ] All component JSONs have a `slug` field matching their filename.

* [ ] All trait values in contracts use `var()` references (no hardcoded hex).

> **If FAIL:** Fix the specific issue — usually a missing CSS link in a preview, or a hardcoded hex value in a contract. Regenerate the affected file(s).
>
> **If PASS:** Proceed to Phase 4.

***

## Phase 4: Documentation & UIKit

**Goal:** Generate the UIKit plan, then produce all documentation files (README, SKILL, library-consumption) and the Marketing UI Kit HTML page.

### Step 4.1 — Generate UIKit Plan

Run the UIKit plan generator and validator in sequence:

```powershell
node "{skill_base}/scripts/generate-uikit-plan.mjs" `
  --component-index "{output_dir}/components/index.json" `
  --components-dir "{output_dir}/components" `
  --brand-data "{tmp_dir}/{BrandName}/phase2-brand-analyst.json" `
  --available-vars "{output_dir}/colors_and_type.css" `
  --components-css "{output_dir}/components.css" `
  --out "{output_dir}/uikit-plan.json"

node "{skill_base}/scripts/validate-uikit-plan.mjs" `
  --plan "{output_dir}/uikit-plan.json" `
  --component-index "{output_dir}/components/index.json" `
  --components-dir "{output_dir}/components" `
  --out "{output_dir}/uikit-plan.json"
```

**Expected output:** `{output_dir}/uikit-plan.json` — a validated plan describing which components and variants to include in the Marketing UI Kit, with layout instructions.

> **Decision point:** If `generate-uikit-plan.mjs` fails, check that `components/index.json` has no BOM and all component JSONs have a `slug` field. See Troubleshooting.

### Step 4.2 — Generate Documentation (Parallel)

**Read** **`agent-dispatch.md`** for the Phase 4a parallel dispatch template.

Dispatch 2 sub-agents in parallel:

**Sub-agent A** writes:

* `{output_dir}/SKILL.md` — YAML frontmatter + quick map + essentials (token lists, component list, usage rules).

* `{output_dir}/library-consumption.json` — downstream consumption order (which files to import and in what sequence).

**Sub-agent B** writes:

* `{output_dir}/README.md` — brand narrative with analytical prose sections: Overview, Content Fundamentals, Visual Foundations, Component Patterns, Index, Caveats.

**Both sub-agents:**

* Read `file-specs/documentation.md` for the schema and content requirements.

* Do NOT read individual component JSON files — they work from the component index and brand profile only.

* Read `colors_and_type.css` for token names.

### Step 4.3 — Generate UIKit

**Read** **`agent-dispatch.md`** for the Phase 4b dispatch template.

Dispatch 1 sub-agent that:

* **Reads:** `file-specs/uikit.md`, all 6 component JSON contracts, `{tmp_dir}/{BrandName}/phase2-brand-analyst.json`, `{output_dir}/colors_and_type.css`, `{output_dir}/uikit-plan.json`.

* **Writes:** `{output_dir}/ui_kits/marketing/index.html`.

* The HTML must be a self-contained marketing page showcasing all components in context (hero, feature cards, CTA section, navigation, footer).

* **CSS link (REQUIRED):**

  ```html
  <link rel="stylesheet" href="../../colors_and_type.css">
  ```

  The path is `../../colors_and_type.css` because the UIKit lives in `ui_kits/marketing/`, two levels below the output root.

### Gate 4

Check:

* [ ] `README.md` exists and has all 6 prose sections.

* [ ] `SKILL.md` exists with YAML frontmatter.

* [ ] `library-consumption.json` exists and is valid JSON.

* [ ] `uikit-plan.json` exists and passed validation.

* [ ] `ui_kits/marketing/index.html` exists.

* [ ] UIKit HTML has `<link rel="stylesheet" href="../../colors_and_type.css">`.

> **If FAIL:** Fix the specific missing file or incorrect CSS link path. Regenerate as needed.
>
> **If PASS:** Proceed to Phase 5.

***

## Phase 5: Validate & Deploy

**Goal:** Strip BOMs from all files, regenerate css.json, run the final validator, clean up temp artifacts, and push to git.

### Step 5.1 — Strip BOMs (ALL files)

Sub-agents often write files with a UTF-8 BOM (byte order mark) that breaks downstream JSON parsing. Strip BOMs from ALL files in both the output directory AND the temp directory.

**Read** **`operation-policies/quality-gates.md`** for the exact BOM-stripping script. The script must walk all subdirectories recursively:

```powershell
# Strip BOM from output_dir
node -e "const fs=require('fs');const dir='{output_dir}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=require('path').join(d,e.name);if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',p)}}})}walk(dir);console.log('BOM strip complete for output_dir')"

# Strip BOM from tmp_dir/{BrandName}
node -e "const fs=require('fs');const dir='{tmp_dir}/{BrandName}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=require('path').join(d,e.name);if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',p)}}})}walk(dir);console.log('BOM strip complete for tmp_dir')"
```

**Expected output:** Console messages listing any files that had BOMs stripped. If no files had BOMs, only the "complete" message appears.

### Step 5.2 — Regenerate css.json

After BOM stripping (which may have modified `colors_and_type.css`), regenerate `css.json`:

```powershell
node "{skill_base}/scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

### Step 5.3 — Validate

Run the final design library validator:

```powershell
node "{skill_base}/scripts/validate-design-library-output.mjs" "{output_dir}"
```

**Expected output:** Either `PASS` (exit code 0) or a list of issues.

**If issues are reported, fix and re-run:**

* **Undefined CSS vars** → add the missing variable to `colors_and_type.css`, then re-run Step 5.2 (regenerate css.json).

* **Component slug mismatches** → add or fix the `slug` field in the component JSON so it matches the filename.

* **Missing files** → create the missing file (trace back to the phase that should have produced it).

* **Invalid JSON** → usually a BOM issue; re-run Step 5.1.

Re-run the validator until exit code is 0. Do not proceed to Step 5.4 until validation passes.

### Step 5.4 — Clean Up

Remove all temporary artifacts:

```powershell
# Remove agent-reports from output (if any sub-agents created them)
Remove-Item -Recurse -Force "{output_dir}/agent-reports" -ErrorAction SilentlyContinue

# Remove the temp directory for this brand
Remove-Item -Recurse -Force "{tmp_dir}/{BrandName}" -ErrorAction SilentlyContinue

# Remove stray temp CSS files and HTML in the working directory
Remove-Item -Force "{workspace}/ef_*.css" -ErrorAction SilentlyContinue
Remove-Item -Force "{workspace}/css_*.css" -ErrorAction SilentlyContinue
Remove-Item -Force "{workspace}/target_page.html" -ErrorAction SilentlyContinue
```

> **Decision point:** Verify `{tmp_dir}/{BrandName}` is gone before proceeding. Temp files should never end up in the git commit.

### Step 5.5 — Git Commit & Push

**Read** **`operation-policies/git-deploy.md`** for full git conventions (branch naming, commit message format).

```powershell
cd {workspace}
git add --all
git commit -m "feat: add {BrandName} design system reverse-engineered from {source-domain}"
git push origin main
```

**Commit message convention:** `feat: add {BrandName} design system reverse-engineered from {source-domain}`. Replace `{BrandName}` with the actual brand name and `{source-domain}` with the website domain (e.g., `endfield.hypergryph.com`).

> **Decision point:** If `git push` fails due to authentication or branch protection, inform the user. The Design Library files are still saved locally and complete — only the remote push is blocked.

### Gate 5 (Final)

Check:

* [ ] Validator passed with exit code 0.

* [ ] No temp files remain in `{workspace}` or `{tmp_dir}`.

* [ ] `git push` succeeded (or user was informed of push failure).

* [ ] All expected deliverables exist in `{output_dir}`:
  * `colors_and_type.css`, `css.json`

  * `components/index.json`, `components/{slug}.json` (×6), `components.css`

  * `preview/component-{slug}.html` (×6)

  * `README.md`, `SKILL.md`, `library-consumption.json`

  * `uikit-plan.json`, `ui_kits/marketing/index.html`

**Report final stats to the user:**

* Files generated (count)

* Tokens extracted (color scales, font families, etc.)

* Components analyzed (6 contracts, 6 previews)

* UIKit page generated

* Git commit hash

***

## Troubleshooting

### SPA sites with no CSS in HTML

If the target site is a client-rendered SPA (Next.js, Nuxt, Vue) and the initial HTML has no `<link rel="stylesheet">` tags:

1. **Read** **`operation-policies/decision-rules.md`** for the SPA decision tree.
2. Check the JS bundle for CSS imports (see Step 0.2 fallback).
3. Try fetching with a full browser user-agent string.
4. If CSS is loaded at runtime by JS and no URLs appear in the bundle, use the browser automation plugin to fetch the rendered DOM and extract `<link>` tags from the post-render HTML.
5. As a last resort, use the `web-perf` or `stark` skill to capture a screenshot and reverse-engineer from visual analysis.

### BOM causes JSON parse failures

Sub-agents running on Windows may produce files with a UTF-8 BOM (3 bytes: `EF BB BF`) at the start. This breaks:

* `JSON.parse()` in Node.js scripts (`generate-uikit-plan.mjs`, `validate-uikit-plan.mjs`).

* The CSS-to-JSON parser.

**Fix:** Run the BOM-stripping script from Step 5.1 on the affected directory. Then re-run the failing script. See `quality-gates.md` for the exact BOM script.

### Validator reports undefined CSS vars

The validator (`validate-design-library-output.mjs`) checks that every `var(--xxx)` reference used in theme blocks and component previews has a corresponding `:root` definition in `colors_and_type.css`.

**Fix:**

1. Read the validator output — it lists the exact undefined variable names.
2. Open `colors_and_type.css` and add the missing definitions.
3. Derive values from the Phase 0 extraction data (frequency tables).
4. Regenerate `css.json` (Step 5.2).
5. Re-run the validator (Step 5.3).

Common gap: theme override blocks (`.dark` / `.light`) reference `--scale-*` tokens that the token sub-agent forgot to define because the brand only had a dark theme.

### UIKit plan generation fails

If `generate-uikit-plan.mjs` throws an error:

1. Strip BOM from `components/index.json` (Step 5.1 script, targeted at the components dir).
2. Ensure every component JSON has a `slug` field that exactly matches its filename (e.g., `button.json` → `"slug": "button"`).
3. Verify `phase2-brand-analyst.json` is valid JSON (no BOM, no trailing comma).
4. Re-run the plan generator.

### Sub-agent can't write report

Some sub-agents may not have write access to `.trae-cn/work/` paths for their internal report JSON. This is expected behavior:

* The **HTML deliverable** (preview page, UIKit page) is the critical output — it must be written to `{output_dir}/` which is the user workspace.

* The **report JSON** (sub-agent self-report) is optional metadata.

* If the report fails to write but the HTML succeeds, proceed to the next phase. Do not block on a missing report file.

