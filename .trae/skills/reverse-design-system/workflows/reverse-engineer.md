# Reverse Engineer Workflow

Full phase-by-phase instructions for reverse-engineering a website's design into a complete Design Library.

## Phase 0: Fetch & Extract

### Goal

Download the target website's CSS and extract every design-relevant token value using regex parsing.

### Step 0.1 — Fetch HTML

```powershell
curl -s -L '<target-url>' -o target_page.html
```

If the site is a SPA (Next.js, Nuxt, etc.), the HTML may contain `<link rel="stylesheet">` tags pointing to CDN-hosted CSS bundles. Look for these patterns.

### Step 0.2 — Extract CSS URLs

Parse the HTML for stylesheet links:

```powershell
Select-String -Path target_page.html -Pattern 'href="([^"]*\.css[^"]*)"' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value }
```

For SPAs, also check `_next/static/css/` paths or check the JS bundles for CSS references. Common CDN domains: `web.hycdn.cn`, `cdn.jsdelivr.net`, etc.

### Step 0.3 — Download CSS Files

```powershell
$cssUrls = @('url1', 'url2', ...)
foreach ($url in $cssUrls) {
  $name = $url.Split('/')[-1]
  curl -s -L $url -o "css_$name"
}
```

### Step 0.4 — Extract Tokens

Run a comprehensive extraction script against all downloaded CSS files. Extract these categories:

**Colors** (hex + rgba):

```powershell
[regex]::Matches($allContent, '(#[0-9a-fA-F]{3,8}\b|rgba?\([^)]+\))') |
  ForEach-Object { $_.Value } | Group-Object | Sort-Object Count -Descending
```

**Font families**:

```powershell
[regex]::Matches($allContent, 'font-family\s*:\s*([^;}{]+)') |
  ForEach-Object { $_.Groups[1].Value.Trim() } | Group-Object | Sort-Object Count -Descending
```

**Font sizes**:

```powershell
[regex]::Matches($allContent, 'font-size\s*:\s*([\d.]+(px|rem|em))') |
  ForEach-Object { $_.Groups[1].Value } | Group-Object | Sort-Object Count -Descending
```

**Borders, border-radius, box-shadow, transitions, backdrop-filter, clip-path, letter-spacing, gradients, transforms, z-index, padding, gap** — use the same `Group-Object | Sort-Object Count -Descending` pattern for each property.

**@font-face declarations** — extract to identify custom web fonts.

**CSS custom properties** — extract `--var-name: value` patterns.

### Step 0.5 — Summarize

Record the top values for each category. These become the evidence base for all downstream phases. Key findings to note:

* Dominant background color (usually the most frequent color)

* Primary text color

* Signature/brand accent color(s)

* Font family names (especially custom @font-face fonts)

* Common font-size scale (convert rem to px at 16px root)

* Border patterns (thickness, style, color)

* Border-radius values

* Shadow patterns (glow vs drop)

* Transition durations and easing functions

* Any clip-path polygon shapes

* Letter-spacing values

* Gradient patterns (especially multi-color brand gradients)

### Deliverable

Raw extracted data saved to:
`{tmp_dir}/{BrandName}/phase0-css-extraction.txt`

***

## Phase 1: Brand Analysis

### Goal

Build a structured brand profile JSON that feeds into the design-library-creator pipeline.

### Step 1.1 — Build Brand Profile

Create a brand profile based on Phase 0 evidence:

```json
{
  "productType": "Marketing/Landing",
  "confidence": "high|medium",
  "personality": ["industrial", "post-apocalyptic", ...],
  "language": "zh|en",
  "visualTone": "One-sentence description of the visual language, citing actual extracted values",
  "kitType": "marketing",
  "colorNamingPrefix": "brandname",
  "uiCopySamples": ["actual UI text found on the website"]
}
```

### Rules

* `productType`: almost always `Marketing/Landing` for public-facing websites

* `confidence`: `high` if CSS extraction yielded clear patterns; `medium` if site is heavily JS-rendered

* `personality`: 3-5 keywords derived from visual evidence (e.g., "industrial", "minimal", "editorial")

* `visualTone`: MUST cite real extracted values (e.g., "near-black #191919 ground with #fffa00 signature yellow")

* `uiCopySamples`: actual text found on the website (nav items, CTA buttons, headings) — NEVER fabricated

### Deliverable

`{tmp_dir}/{BrandName}/phase2-brand-analyst.json`

***

## Phase 2: Token Generation

### Goal

Generate `colors_and_type.css` with all design tokens, then derive `css.json`.

### Step 2.1 — Dispatch Token Generation

Dispatch a sub-agent (Task tool) to generate the token CSS. The sub-agent must:

1. Read `file-specs/css-tokens.md` from design-library-creator skill
2. Read the brand profile from Phase 1
3. Generate CSS with:

   * Full 10-step color scales (50-900) for each color group

   * All scales anchored at the real extracted brand color

   * Typography using real font family names from @font-face

   * Spacing, sizing, radius, shadow, transition tokens

   * Dark/light theme blocks as appropriate

   * Every color marked `/* AI-generated */`

   * `@group-priority` comment at top

### Step 2.2 — Generate css.json

```powershell
node "{skill_base}/scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

### Deliverable

```
{output_dir}/colors_and_type.css
{output_dir}/css.json
```

***

## Phase 3: Component Contracts & Previews

### Goal

Generate component JSON contracts and HTML preview pages for each component.

### Step 3.1 — Create Component Index

Write `components/index.json` with the component list. Standard 6 components:

| Slug       | Name       | Category |
| ---------- | ---------- | -------- |
| button     | Button     | action   |
| card       | Card       | surface  |
| input      | Input      | form     |
| badge      | Badge      | status   |
| cta-link   | CTA Link   | action   |
| navigation | Navigation | shell    |

Each entry includes `slug`, `name`, `category`, `confidence`, `variantCount`, `priorityHint`, `keyInsightSeed`.

### Step 3.2 — Write Component Contracts

For each component, write `components/{slug}.json` with:

* `representativeVariants`: 3-4 key variants with real CSS trait values

* `anatomy`: structural elements

* `structurePatterns`: layout patterns

* `usageHints`: priority and a11y guidance

* `doNotInvent`: things NOT to include

* `unknowns`: acknowledged gaps

### Step 3.3 — Generate Component Previews (Parallel)

Dispatch sub-agents in parallel (up to 3 at a time) to generate `preview/component-{slug}.html`. Each sub-agent:

1. Reads `file-specs/preview-pages.md` from design-library-creator
2. Reads `colors_and_type.css` for variable definitions
3. Reads the component JSON contract
4. Writes a self-contained HTML preview using only defined CSS variables

### Step 3.4 — Extract Components CSS

```powershell
node "{skill_base}/scripts/extract-components-css.mjs" "{output_dir}"
```

### Deliverables

```
{output_dir}/components/index.json
{output_dir}/components/{slug}.json (×6)
{output_dir}/preview/component-{slug}.html (×6)
{output_dir}/components.css
```

***

## Phase 4: Documentation & UI Kit

### Goal

Generate README.md, SKILL.md, library-consumption.json, and the Marketing UI Kit.

### Step 4.1 — Generate SKILL.md + library-consumption.json (Parallel)

Dispatch a sub-agent to write:

* `SKILL.md` — YAML frontmatter + quick map + essentials

* `library-consumption.json` — downstream consumption order

### Step 4.2 — Generate README.md (Parallel with 4.1)

Dispatch a sub-agent to write:

* `README.md` — brand narrative with analytical prose sections (Overview, Content Fundamentals, Visual Foundations, Component Patterns, Index, Caveats)

### Step 4.3 — Generate UIKit Plan

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

### Step 4.4 — Generate UIKit HTML

Dispatch a sub-agent to write `ui_kits/marketing/index.html` using the UIKit plan.

### Deliverables

```
{output_dir}/README.md
{output_dir}/SKILL.md
{output_dir}/library-consumption.json
{output_dir}/uikit-plan.json
{output_dir}/ui_kits/marketing/index.html
```

***

## Phase 5: Validate & Deploy

### Step 5.1 — Strip BOMs

All files in the output directory must be UTF-8 without BOM. Run:

```powershell
node -e "const fs=require('fs');const dir='{output_dir}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=d+'/'+e.name;if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',e.name)}}})}walk(dir);console.log('Done')"
```

Also strip BOMs from the temp brand profile:

```powershell
node -e "const fs=require('fs');const dir='{tmp_dir}/{BrandName}';function walk(d){...same logic...}walk(dir);console.log('Done')"
```

### Step 5.2 — Regenerate css.json

After BOM stripping, regenerate css.json:

```powershell
node "{skill_base}/scripts/css-to-json.mjs" "{output_dir}/colors_and_type.css" --output "{output_dir}/css.json"
```

### Step 5.3 — Validate

```powershell
node "{skill_base}/scripts/validate-design-library-output.mjs" "{output_dir}"
```

If validator reports issues:

* Undefined CSS vars → add missing vars to `colors_and_type.css`

* Component slug mismatches → add `slug` field to component JSON

* Missing files → create them

### Step 5.4 — Clean Up

Remove agent-reports and temp files:

```powershell
Remove-Item -Recurse -Force "{output_dir}/agent-reports" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "{tmp_dir}/{BrandName}" -ErrorAction SilentlyContinue
```

### Step 5.5 — Git Commit & Push

```powershell
git add --all
git commit -m "feat: add {BrandName} design system reverse-engineered from {source-url}"
git push origin main
```

### Deliverable

A complete, validated Design Library pushed to the remote repository.

***

## Troubleshooting

### SPA sites with no CSS in HTML

* Check the JS bundle for CSS imports

* Look for `_next/static/css/` (Next.js) or `static/css/` patterns

* Try fetching with a browser user-agent header: `curl -H "User-Agent: Mozilla/5.0 ..."`

### BOM causes JSON parse failures

* Always strip BOMs before running any Node.js script that reads JSON

* The BOM-stripping script in Step 5.1 handles this

### Validator reports undefined CSS vars

* Check `colors_and_type.css` for the variable name

* Add the missing variable with a value derived from Phase 0 extraction data

* Common gap: theme override blocks (.dark / .light) referencing scale tokens that don't exist yet

### UIKit plan generation fails

* Ensure `components/index.json` has no BOM

* Ensure all component JSON files have a `slug` field matching their filename

* Re-run after BOM stripping

### Sub-agent can't write to tmp report path

* Some sub-agents may not have write access to `.trae-cn/work/` paths

* The HTML deliverable is the critical output; the report JSON is optional

* If report fails but HTML succeeds, proceed to next phase

