# Extract Layout Workflow

## Purpose

This is the main execution workflow for the `reverse-page-layout` skill. It defines the phase-by-phase process for extracting page layout, UX, and UI patterns from a target website and producing reusable HTML layout templates.

The workflow is executed by the main agent and may dispatch sub-agents for parallel template generation.

---

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `url` | string | yes | The target website URL to extract layout patterns from |
| `outputDir` | string | yes | The directory path where all output files will be written |
| `brand` | string | no | Optional brand name override; otherwise inferred from the page |

---

## Phase 0: Browser Page Extraction

**Goal:** Navigate to the target URL, capture a screenshot, run the four extraction scripts from `page-section-extraction.md`, and save the combined output to JSON.

### Steps

1. **Navigate to the URL.**
   ```
   browser_navigate(url)
   ```
   Wait for `networkidle` event (or equivalent — all network requests settled, async content loaded).

2. **Set viewport to desktop width.**
   ```
   browser_resize(1440, 1080)
   ```
   Wait 1000ms for any resize-triggered reflow to settle.

3. **Capture full-page screenshot.**
   ```
   browser_screenshot(fullPage=true)
   ```
   Save the screenshot to `{outputDir}/assets/screenshot-full.png`.

   Also capture viewport-only screenshots at each breakpoint:
   ```
   browser_resize(1920, 1080) → screenshot → screenshot-1920.png
   browser_resize(1440, 1080) → screenshot → screenshot-1440.png
   browser_resize(1024, 1080) → screenshot → screenshot-1024.png
   browser_resize(768, 1080) → screenshot → screenshot-768.png
   browser_resize(375, 812) → screenshot → screenshot-375.png
   ```

4. **Run Script 1: Page Section Structure.**
   ```
   browser_evaluate(script_1_page_section_structure)
   ```
   Save result to `{outputDir}/intermediate/extraction-sections.json`.

5. **Run Script 2: Component Placement Map.**
   ```
   browser_evaluate(script_2_component_placement_map)
   ```
   Save result to `{outputDir}/intermediate/extraction-components.json`.

6. **Run Script 3: Navigation Structure.**
   ```
   browser_evaluate(script_3_navigation_structure)
   ```
   Save result to `{outputDir}/intermediate/extraction-navigation.json`.

7. **Run Script 4: Responsive Breakpoints** (5 iterations).

   For each width in `[1920, 1440, 1024, 768, 375]`:
   ```
   browser_resize(width, 1080)
   wait 500ms
   browser_evaluate(script_4_responsive_breakpoints)
   ```
   Collect all 5 results into an array.

   Save result to `{outputDir}/intermediate/extraction-responsive.json`.

8. **Merge all extraction outputs.**

   Combine the four JSON files into a single `extraction-output.json` following the Output Format defined in `page-section-extraction.md`. Write to:
   ```
   {outputDir}/intermediate/extraction-output.json
   ```

### Gate 0 Criteria

Before proceeding to Phase 1, verify:
- [ ] Full-page screenshot captured and saved
- [ ] At least 3 sections extracted (Script 1 returned `totalSections >= 3`)
- [ ] Navigation structure mapped (Script 3 returned `found: true`)
- [ ] At least 3 responsive breakpoints captured (Script 4 ran at 3+ widths)
- [ ] `extraction-output.json` exists and is valid JSON

**If Gate 0 fails:** See `decision-rules.md` for edge-case handling (minimal pages, SPAs with lazy-loaded content).

---

## Phase 1: Layout Analysis

**Goal:** Analyze the extraction output, classify each section into a standard type, identify grid/flex layout patterns, and produce a structured analysis that feeds template generation.

### Steps

1. **Read the extraction output.**
   Read `{outputDir}/intermediate/extraction-output.json`.

2. **Classify each section.**

   For each entry in `sections`, determine the section type from the standard set:
   - `section-nav` — if tag is `header` or `nav`, or classes contain `nav`, `header`, `banner`
   - `section-hero-centered` — if first major content section with centered text + CTA, height > 400px
   - `section-hero-split` — if first major content section with 2-column flex/grid (text + visual)
   - `section-feature-grid` — if layout is grid with 2-4 columns and children are card-like
   - `section-card-row` — if layout is flex row or grid with horizontal card layout
   - `section-testimonials` — if text content contains testimonial-like quotes or classes match
   - `section-pricing` — if text content contains price-like patterns ($, /mo, plan, tier)
   - `section-cta-banner` — if section has a single CTA focus, full-width background, short height
   - `section-footer` — if tag is `footer` or at bottom of page with link groups

3. **Identify layout patterns.**

   For each section, extract:
   - Grid template pattern (e.g., `repeat(3, 1fr)`, `1fr 1fr`, `auto-fit minmax`)
   - Flex direction and alignment
   - Section spacing (padding values)
   - Max-width / container width
   - Gap values

4. **Produce analysis output.**

   Write the classified analysis to:
   ```
   {outputDir}/intermediate/layout-analysis.json
   ```

   Structure:
   ```json
   {
     "source": "url",
     "classifiedSections": [
       {
         "order": 0,
         "sectionType": "section-nav",
         "originalTag": "header",
         "originalClasses": "site-header sticky",
         "layoutType": "flex",
         "layoutDetails": {
           "display": "flex",
           "justifyContent": "space-between",
           "alignItems": "center",
           "position": "sticky"
         },
         "gridPattern": null,
         "flexDirection": "row",
         "gap": "0px",
         "padding": "0px 24px",
         "maxWidth": "1200px",
         "height": 80,
         "childCount": 3,
         "responsiveChanges": "nav → hamburger at 768px"
       }
     ],
     "gridPatterns": [
       { "name": "3-col-auto", "template": "repeat(auto-fit, minmax(300px, 1fr))", "usage": "section-feature-grid" }
     ],
     "pageSystem": {
       "maxContentWidth": "1200px",
       "sectionSpacing": "64px 0",
       "containerPadding": "0 24px",
       "gridGap": "24px"
     }
   }
   ```

### Gate 1 Criteria

Before proceeding to Phase 2, verify:
- [ ] All sections classified into standard types (each has a `sectionType` from the standard set)
- [ ] At least 1 grid pattern identified (or confirmed no grids exist)
- [ ] Page-level layout system metadata extracted (maxContentWidth, sectionSpacing, gridGap)
- [ ] `layout-analysis.json` written and valid

**If Gate 1 fails:** Review the extraction output for missed sections; reclassify using looser heuristics.

---

## Phase 2: Layout Template Generation

**Goal:** Generate all section HTML templates, the `layout-system.json`, and `page-sections.json` index file.

### Dispatch: 3 Parallel Sub-Agents

Dispatch three sub-agents in parallel. Each sub-agent reads:
- `file-specs/layout-template-spec.md` (the template spec with rules and examples)
- `intermediate/layout-analysis.json` (the classified sections to template)

Each sub-agent is responsible for generating specific section templates.

#### Sub-Agent 1: Hero + Nav Templates

**Task:** Generate the following template files:
- `templates/section-nav.html`
- `templates/section-hero-centered.html`
- `templates/section-hero-split.html`

**Instructions for sub-agent:**
1. Read `file-specs/layout-template-spec.md` for rules and structure.
2. Read `intermediate/layout-analysis.json` for the classified nav and hero sections.
3. For each section type, create a self-contained HTML file following the template structure.
4. Bake in the layout properties (grid-template, gap, padding, flex-direction, min-height, max-width) extracted from the analysis.
5. Use CSS variables for all visual properties (colors, fonts, sizes, radii, shadows).
6. Include `@media (max-width: 768px)` responsive rules based on the responsive breakpoint data.
7. Use `{curly brace}` placeholder syntax for text content.
8. Write each file to `{outputDir}/templates/section-{type}.html`.

#### Sub-Agent 2: Feature Grid + Card Row + Testimonials Templates

**Task:** Generate the following template files:
- `templates/section-feature-grid.html`
- `templates/section-card-row.html`
- `templates/section-testimonials.html`

**Instructions:** Same as Sub-Agent 1, but for these section types. Pay special attention to:
- Grid template patterns for feature grids (use the `gridPatterns` from analysis)
- Card hover effects (CSS `:hover` only, no JS)
- Testimonial layout (quote + attribution + optional avatar)

#### Sub-Agent 3: CTA + Pricing + Footer Templates

**Task:** Generate the following template files:
- `templates/section-cta-banner.html`
- `templates/section-pricing.html`
- `templates/section-footer.html`

**Instructions:** Same as Sub-Agent 1, but for these section types. Pay special attention to:
- CTA banner full-width background with centered content
- Pricing card layout with feature lists (checkmarks) and highlighted "recommended" tier
- Footer multi-column link layout

### After Sub-Agents Complete: Generate System Files

Once all three sub-agents have completed, the main agent generates two additional files:

1. **`layout-system.json`** — Read from `layout-analysis.json` `pageSystem` and `gridPatterns` fields. Write to `{outputDir}/layout-system.json` following the schema in `layout-template-spec.md`.

2. **`page-sections.json`** — Build from the classified sections, mapping each to its template file. Write to `{outputDir}/page-sections.json` following the schema in `layout-template-spec.md`.

### Gate 2 Criteria

Before proceeding to Phase 3, verify:
- [ ] All template files exist in `templates/` directory (9 files: nav, hero-centered, hero-split, feature-grid, card-row, testimonials, pricing, cta-banner, footer)
- [ ] No hardcoded colors in any template (search for `#` followed by hex digits in `<style>` blocks — none should exist except in comments)
- [ ] No hardcoded font-family, font-size, border-radius, or box-shadow values
- [ ] Every template has at least one `@media (max-width: 768px)` rule
- [ ] Every template has the HTML comment header (section name, source URL, extraction date)
- [ ] CSS class names match filenames (`.section-{type}` in `section-{type}.html`)
- [ ] `layout-system.json` generated and valid
- [ ] `page-sections.json` generated and valid

**If Gate 2 fails:** Re-dispatch the failing sub-agent with specific feedback on which rule was violated.

---

## Phase 3: UX Flow Mapping

**Goal:** Generate `user-journey.json` and `interaction-patterns.json` by analyzing the extraction output and the live page for interaction cues.

### Dispatch: 1 Sub-Agent

Dispatch one sub-agent to handle the UX flow mapping.

**Task:** Generate the following files:
- `{outputDir}/ux/user-journey.json`
- `{outputDir}/ux/interaction-patterns.json`

**Instructions for sub-agent:**
1. Read `file-specs/ux-flow-spec.md` for the JSON schemas and field rules.
2. Read `intermediate/extraction-output.json` for section data, component map, navigation, and responsive breakpoints.
3. Read `intermediate/layout-analysis.json` for classified section types.
4. Optionally use the browser to inspect the live page for interaction cues:
   - Hover over cards/buttons to detect hover effects
   - Scroll slowly to detect scroll-triggered animations
   - Observe auto-rotating content (carousels, testimonials)
   - Check for video backgrounds in hero/banner sections
5. Construct `user-journey.json`:
   - Infer `brand` from page title, logo, or meta tags
   - Map the user's path from hero → through sections → to conversion CTA
   - Write at least 3 steps (minimum per Gate 3)
   - Identify the primary conversion path and exit points
   - Determine `navigationPattern` from the navigation extraction data
6. Construct `interaction-patterns.json`:
   - Catalog each detected interaction pattern
   - Include at least 2 patterns (minimum per Gate 3)
   - Document responsive behaviors at each breakpoint
7. Write both files to `{outputDir}/ux/`.

### Gate 3 Criteria

Before proceeding to Phase 4, verify:
- [ ] `user-journey.json` exists, is valid JSON, and has at least 3 steps in the `steps` array
- [ ] `user-journey.json` has `primaryGoal`, `conversionPath` (non-empty array), `exitPoints` (non-empty array), and `navigationPattern`
- [ ] `interaction-patterns.json` exists, is valid JSON, and has at least 2 entries in the `patterns` array
- [ ] `interaction-patterns.json` has `responsiveBehaviors` with at least 1 entry covering 768px

**If Gate 3 fails:** See `decision-rules.md` for the no-interactions-detected fallback (output empty patterns array, proceed).

---

## Phase 4: Preview & Validate

**Goal:** Create a `preview.html` file that assembles all section templates with a sample token CSS applied, verify rendering in the browser, and confirm no broken layouts.

### Steps

1. **Create sample token CSS.**

   Write a sample token CSS file to `{outputDir}/assets/sample-tokens.css` that defines all the CSS variables referenced by the templates with placeholder values:
   ```css
   :root {
     --primary: #2563eb;
     --primary-foreground: #ffffff;
     --secondary: #64748b;
     --background: #ffffff;
     --foreground: #0f172a;
     --muted: #64748b;
     --color-surface: #f8fafc;
     --border: #e2e8f0;
     --font-family-display: 'Inter', system-ui, sans-serif;
     --font-family-body: 'Inter', system-ui, sans-serif;
     --font-family-mono: 'JetBrains Mono', monospace;
     --font-size-h1: 48px;
     --font-size-h2: 36px;
     --font-size-h3: 24px;
     --font-size-body: 16px;
     --font-size-lead: 20px;
     --font-size-caption: 14px;
     --font-weight-h1: 700;
     --font-weight-body: 400;
     --tracking-tight: -0.02em;
     --space-2: 8px;
     --space-3: 12px;
     --space-4: 16px;
     --space-6: 24px;
     --space-8: 32px;
     --space-12: 48px;
     --space-16: 64px;
     --radius-sm: 4px;
     --radius-md: 8px;
     --radius-lg: 16px;
     --shadow-1: 0 1px 2px rgba(0,0,0,0.05);
     --shadow-2: 0 4px 6px rgba(0,0,0,0.1);
     --shadow-3: 0 10px 15px rgba(0,0,0,0.15);
     --transition-fast: 0.2s ease;
   }
   ```

2. **Create preview.html.**

   Assemble all section templates in page order (from `page-sections.json`) into a single HTML file:
   - `<head>` links `sample-tokens.css` and includes a `<title>` with the brand name.
   - `<body>` contains each section template's HTML inline (the `<style>` blocks are scoped by class so they won't conflict).
   - Replace `{placeholder}` text with realistic sample content.
   - Write to `{outputDir}/preview.html`.

3. **Render and validate in browser.**
   ```
   browser_navigate("file:///{outputDir}/preview.html")
   browser_resize(1440, 1080)
   browser_screenshot(fullPage=true)
   ```
   Visually verify:
   - All sections render in correct order
   - No overlapping or broken layouts
   - CSS variables resolve (no missing variable warnings)
   - Responsive: resize to 768px and 375px, verify grids collapse and nav switches to hamburger

4. **Run validation script.**

   Execute a validation script via `browser_evaluate` that checks:
   ```javascript
   (() => {
     const sections = document.querySelectorAll('[class*="section-"]');
     const issues = [];
     sections.forEach((s) => {
       const rect = s.getBoundingClientRect();
       if (rect.height === 0) issues.push(`${s.className}: zero height`);
       if (rect.width === 0) issues.push(`${s.className}: zero width`);
       // Check for unresolved CSS variables
       const computed = getComputedStyle(s);
       // ... additional checks
     });
     return { sectionCount: sections.length, issues: issues };
   })();
   ```

### Gate 4 Criteria

Before proceeding to Phase 5, verify:
- [ ] `preview.html` renders without errors
- [ ] All section templates visible in the preview (section count matches `page-sections.json`)
- [ ] No zero-height or zero-width sections
- [ ] No unresolved CSS variable warnings in console
- [ ] Responsive: at 768px, grids collapse to 1-2 columns, nav switches to hamburger
- [ ] Responsive: at 375px, content reflows without horizontal scroll
- [ ] Screenshot captured at 1440px for documentation

**If Gate 4 fails:** Identify the specific template(s) causing issues, re-dispatch the responsible sub-agent to fix, then re-validate.

---

## Phase 5: Deploy

**Goal:** Commit all generated files to git and push to the remote repository.

### Steps

1. **Stage all output files.**
   ```bash
   git add {outputDir}/templates/section-*.html
   git add {outputDir}/layout-system.json
   git add {outputDir}/page-sections.json
   git add {outputDir}/ux/user-journey.json
   git add {outputDir}/ux/interactions-patterns.json
   git add {outputDir}/preview.html
   git add {outputDir}/assets/sample-tokens.css
   git add {outputDir}/assets/screenshot-*.png
   git add {outputDir}/intermediate/extraction-output.json
   git add {outputDir}/intermediate/layout-analysis.json
   ```

2. **Commit.**
   ```bash
   git commit -m "feat(reverse-page-layout): extract layout templates from {url}

   - {N} section templates generated
   - layout-system.json with grid patterns
   - user-journey.json with {M} steps
   - interaction-patterns.json with {K} patterns
   - preview.html for validation
   "
   ```

3. **Push.**
   ```bash
   git push origin main
   ```

### Gate 5 Criteria

- [ ] `git push` completed successfully (exit code 0)
- [ ] All files are in the remote repository
- [ ] No untracked files remaining in `{outputDir}` (except `intermediate/` which may contain additional scratch files)

---

## Output File Manifest

After the workflow completes, the following files should exist in `{outputDir}`:

```
{outputDir}/
├── templates/
│   ├── section-nav.html
│   ├── section-hero-centered.html
│   ├── section-hero-split.html
│   ├── section-feature-grid.html
│   ├── section-card-row.html
│   ├── section-testimonials.html
│   ├── section-pricing.html
│   ├── section-cta-banner.html
│   └── section-footer.html
├── ux/
│   ├── user-journey.json
│   └── interaction-patterns.json
├── assets/
│   ├── sample-tokens.css
│   ├── screenshot-full.png
│   ├── screenshot-1920.png
│   ├── screenshot-1440.png
│   ├── screenshot-1024.png
│   ├── screenshot-768.png
│   └── screenshot-375.png
├── intermediate/
│   ├── extraction-sections.json
│   ├── extraction-components.json
│   ├── extraction-navigation.json
│   ├── extraction-responsive.json
│   ├── extraction-output.json
│   └── layout-analysis.json
├── layout-system.json
├── page-sections.json
└── preview.html
```

---

## File Specs Read Order

Sub-agents dispatched in Phases 2 and 3 must read these spec files before starting:

| Sub-Agent | Must Read |
|---|---|
| Phase 2 (all 3 sub-agents) | `file-specs/layout-template-spec.md` |
| Phase 3 (1 sub-agent) | `file-specs/ux-flow-spec.md` |
| All phases | `operation-policies/quality-gates.md`, `operation-policies/decision-rules.md` |

---

## Error Handling

If any phase fails to pass its gate criteria after 2 attempts:

1. Check `operation-policies/decision-rules.md` for the applicable edge-case policy.
2. If the edge case is covered, follow the prescribed action and continue.
3. If the edge case is NOT covered, log the issue in the final output under `caveats` and proceed with best-effort output.
4. Never abort the workflow entirely — always produce as much output as possible.
