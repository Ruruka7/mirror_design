# Quality Gates

## Purpose

This document defines the gate criteria that must be met before transitioning between phases of the `extract-layout` workflow. Each gate is a checkpoint that ensures the output of the preceding phase is complete and valid before the next phase begins.

If a gate fails, the phase must be retried (up to 2 attempts) before consulting `decision-rules.md` for edge-case handling.

---

## Gate 0: Browser Page Extraction

**Phase:** Phase 0 (Browser Page Extraction) → Phase 1 (Layout Analysis)

### Criteria

- [ ] Full-page screenshot captured and saved to `assets/screenshot-full.png`
- [ ] At least 3 sections extracted from the page (Script 1 returned `totalSections >= 3`)
- [ ] Navigation structure mapped (Script 3 returned `found: true` with `navItems` populated)
- [ ] At least 3 responsive breakpoints captured (Script 4 ran successfully at 3+ widths)
- [ ] `intermediate/extraction-output.json` exists, is valid JSON, and contains the `sections`, `componentMap`, `navigation`, and `responsiveBreakpoints` keys

### Failure Action

If fewer than 3 sections are extracted:
1. Check if the page is an SPA with lazy-loaded content (see `decision-rules.md`: SPA with lazy-loaded sections).
2. If SPA: scroll 3 times with 1s delays, re-run Script 1, re-check.
3. If not SPA: proceed with available sections (see `decision-rules.md`: Minimal page).

If navigation is not found:
1. Check if the page uses a non-standard nav structure.
2. Set `navigation.found = false` in the output and proceed.

---

## Gate 1: Layout Analysis

**Phase:** Phase 1 (Layout Analysis) → Phase 2 (Layout Template Generation)

### Criteria

- [ ] All extracted sections classified into a standard type from the catalog (`section-nav`, `section-hero-centered`, `section-hero-split`, `section-feature-grid`, `section-card-row`, `section-testimonials`, `section-pricing`, `section-cta-banner`, `section-footer`)
- [ ] At least 1 grid pattern identified OR confirmed that no grids exist on the page (documented in analysis)
- [ ] Page-level layout system metadata extracted: `maxContentWidth`, `sectionSpacing`, `containerPadding`, `gridGap`
- [ ] `intermediate/layout-analysis.json` written and valid JSON with `classifiedSections`, `gridPatterns`, and `pageSystem` keys

### Failure Action

If sections cannot be classified:
1. Use looser heuristics (match by text content keywords, child element tags, position in page).
2. Assign the closest standard type even if the match is imperfect.
3. Log classification confidence in the analysis output (high/medium/low).

If no grid patterns found:
1. This is valid for pages that use only flex or block layouts.
2. Set `gridPatterns` to an empty array and document that the page uses flex/block only.

---

## Gate 2: Layout Template Generation

**Phase:** Phase 2 (Layout Template Generation) → Phase 3 (UX Flow Mapping)

### Criteria

- [ ] All expected template files exist in `templates/` directory (files for every section type identified in Phase 1)
- [ ] No hardcoded colors in any template file (no hex color values `#XXXXXX` in `<style>` blocks, except in HTML comments)
- [ ] No hardcoded `font-family` values (all use `var(--font-family-*)`)
- [ ] No hardcoded `font-size` values (all use `var(--font-size-*)`)
- [ ] No hardcoded `border-radius` values (all use `var(--radius-*)`)
- [ ] No hardcoded `box-shadow` values (all use `var(--shadow-*)`)
- [ ] Every template has at least one `@media (max-width: 768px)` responsive rule
- [ ] Every template has the HTML comment header (section name, source URL, extraction date)
- [ ] CSS class names match filenames (`.section-{type}` in `section-{type}.html`)
- [ ] No `<script>` tags or inline JavaScript in any template
- [ ] `layout-system.json` generated and valid JSON
- [ ] `page-sections.json` generated and valid JSON

### Validation Method

Run a validation check across all template files:
- Search each file for hex color patterns (`#[0-9a-fA-F]{3,8}`) in `<style>` blocks — should find none
- Search for `font-family:` followed by a literal string (not `var(...)`) — should find none
- Search for `font-size:` followed by a pixel value (not `var(...)`) — should find none
- Verify each file contains `@media` — all must have it
- Verify each file contains `<!-- Section:` comment header — all must have it

### Failure Action

If any template violates the CSS variable rules:
1. Identify the specific violation (file, line, property, hardcoded value).
2. Re-dispatch the responsible sub-agent with the specific violation list.
3. Sub-agent fixes only the violations and rewrites the affected files.

If `layout-system.json` or `page-sections.json` is missing:
1. Generate directly from `layout-analysis.json` (this is a main-agent task, not a sub-agent task).

---

## Gate 3: UX Flow Mapping

**Phase:** Phase 3 (UX Flow Mapping) → Phase 4 (Preview & Validate)

### Criteria

- [ ] `ux/user-journey.json` exists and is valid JSON
- [ ] `user-journey.json` has at least 3 steps in the `steps` array
- [ ] `user-journey.json` has `primaryGoal` (non-empty string)
- [ ] `user-journey.json` has `conversionPath` (non-empty array)
- [ ] `user-journey.json` has `exitPoints` (non-empty array)
- [ ] `user-journey.json` has `navigationPattern` (one of the enumerated values)
- [ ] `ux/interaction-patterns.json` exists and is valid JSON
- [ ] `interaction-patterns.json` has at least 2 entries in the `patterns` array
- [ ] `interaction-patterns.json` has `responsiveBehaviors` with at least 1 entry covering the `768px` breakpoint

### Failure Action

If fewer than 3 user journey steps:
1. Add inferred steps based on section order (every section is at least one step).
2. Re-check.

If fewer than 2 interaction patterns detected:
1. See `decision-rules.md`: No interactions detected.
2. Output an empty `patterns` array and proceed with a note in the file.
3. Alternatively, infer default patterns (sticky nav, hover on cards) from the template data.

---

## Gate 4: Preview & Validate

**Phase:** Phase 4 (Preview & Validate) → Phase 5 (Deploy)

### Criteria

- [ ] `preview.html` exists and renders without JavaScript console errors
- [ ] All section templates visible in the preview (section count matches `page-sections.json`)
- [ ] No zero-height or zero-width sections (all sections have visible content)
- [ ] No unresolved CSS variable warnings (all `var(--*)` references resolve against `sample-tokens.css`)
- [ ] At 1440px width: all sections display in correct order, no overlapping elements
- [ ] At 768px width: grids collapse to 1-2 columns, nav switches to hamburger menu (if applicable)
- [ ] At 375px width: content reflows without horizontal scroll
- [ ] Screenshot captured at 1440px for documentation

### Failure Action

If a section has zero height/width:
1. Check if the template HTML is malformed (unclosed tags, broken structure).
2. Check if the CSS class name doesn't match the HTML element class.
3. Re-dispatch the responsible sub-agent to fix the specific template.

If CSS variables are unresolved:
1. Check `sample-tokens.css` for missing variable definitions.
2. Add any missing variables to the sample tokens file.
3. Re-validate.

If responsive layout breaks:
1. Check the media queries in the specific template.
2. Ensure `grid-template-columns` collapses properly at 768px.
3. Re-dispatch sub-agent to fix the media query.

---

## Gate 5: Deploy

**Phase:** Phase 5 (Deploy) → Workflow Complete

### Criteria

- [ ] `git add` staged all expected output files (templates, JSON files, preview, assets)
- [ ] `git commit` completed successfully with descriptive commit message
- [ ] `git push` completed successfully (exit code 0)
- [ ] All files visible in the remote repository
- [ ] No untracked files remaining in `{outputDir}` (excluding `intermediate/` scratch files)

### Failure Action

If `git push` fails:
1. Check for authentication issues.
2. Check for merge conflicts.
3. Retry push up to 2 times.
4. If still failing, report the git error and mark the deploy as blocked.

---

## Gate Summary Table

| Gate | Phase Transition | Key Criteria | Min Sections | Min Patterns |
|---|---|---|---|---|
| Gate 0 | Phase 0 → 1 | Screenshot, extraction scripts run, JSON valid | 3 | — |
| Gate 1 | Phase 1 → 2 | All sections classified, layout patterns identified | — | — |
| Gate 2 | Phase 2 → 3 | Templates use CSS vars only, responsive rules present, JSON files generated | — | — |
| Gate 3 | Phase 3 → 4 | Journey ≥3 steps, patterns ≥2, responsive behaviors present | — | 2 |
| Gate 4 | Phase 4 → 5 | Preview renders, no broken layouts, responsive verified | — | — |
| Gate 5 | Phase 5 → Done | Git push successful | — | — |
