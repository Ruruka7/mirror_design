---
name: "reverse-to-site"
description: "Reverse-engineer any website into two reusable assets — design tokens (colors_and_type.css) and a complete site template (site-template.html) — then combine any token set with any layout to generate a complete new website. Covers the full pipeline: browser extraction → token generation → layout extraction → site template → cross-brand combination. Invoke when user says '逆向'/'reverse engineer'/'抓设计'/'抓页面'/'从网站提取' and wants to reuse the design, or wants to combine a token set with a layout to make a new site. Do NOT invoke for from-scratch design system creation without a source URL."
---

# Reverse-to-Site

## What This Skill Does

Reverse-engineer any website into two portable assets that combine into new sites.

## The Two Assets

| Asset | File | Description |
|-------|------|-------------|
| Design Tokens | `colors_and_type.css` | All visual values: colors, fonts, spacing, shadows, radius. Uses CSS variables with portable names that work across brands. |
| Site Template | `site-template.html` | A complete standalone HTML page recreating the source site's full structure. All visual values use `var(--name, fallback)`. Zero hardcoded colors/fonts. |

## Combination Principle

Change one `<link>` to swap token sets. The same layout instantly becomes a completely different visual style. Any `colors_and_type.css` works with any `site-template.html` — they are decoupled by design.

```html
<!-- Swap this single line to restyle the entire site -->
<link rel="stylesheet" href="colors_and_type.css">
```

## Pipeline Map

| Phase | Name | Description |
|-------|------|-------------|
| Phase 1 | Token Extraction | Browser → CSS parse → `colors_and_type.css` |
| Phase 2 | Token Verification | Swap test → fix missing variables |
| Phase 3 | Layout Extraction | Browser → DOM structure → `site-template.html` |
| Phase 4 | Layout Verification | Render test → token swap test → fix |
| Phase 5 | Assembly & Deployment | Create combinations → git push |

## When to Invoke

- User says "逆向" / "reverse engineer" / "抓设计" / "抓页面" / "从网站提取"
- User provides a URL and wants to reuse the website's design as tokens or layout
- User wants to extract design tokens (colors, fonts, spacing) from a real website
- User wants to extract a complete page layout from a real website
- User wants to combine a token set with a layout to create a new site
- User wants to create a cross-brand combination (Brand A layout + Brand B tokens)

## When NOT to Invoke

- User wants to create a design system from scratch without a source URL → use `design-library-creator`
- User wants page/UI design on a canvas → use `solo-design`
- User has a Figma export or brand guide → use `design-library-creator` directly
- User wants to edit an existing design system → use `design-library-creator` (refine route)

## File Structure

The skill produces the following output per brand:

```
{BrandName}/
  colors_and_type.css    ← Design tokens
  site-template.html     ← Complete site template
  css.json               ← Token JSON (derived)
  components/             ← Component contracts (optional)
  preview/                ← Component previews (optional)
```

## Combination Usage

```powershell
# Create a combination: Brand A's layout + Brand B's tokens
New-Item -ItemType Directory -Force -Path "combinations/{A}-layout_{B}-tokens"
$content = Get-Content "{A}/site-template.html" -Raw
$content = $content -replace 'href="colors_and_type\.css"', 'href="../../{B}/colors_and_type.css"'
# If using dark-themed tokens, add class="dark" to <body>
Set-Content -Path "combinations/{A}-layout_{B}-tokens/index.html" -Value $content -Encoding UTF8
```

Open `combinations/{A}-layout_{B}-tokens/index.html` in a browser. Brand A's layout now wears Brand B's visual style.

## Reference Files

| File | Purpose |
|------|---------|
| `workflows/full-pipeline.md` | Complete A-to-Z workflow playbook with JavaScript extraction snippets, token contract details, template contract rules, quality gates, and fallback chain. Any AI agent can follow this from start to finish. |
