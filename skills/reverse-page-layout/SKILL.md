---
name: "reverse-page-layout"
description: "Reverse-engineer a website's complete page structure from its rendered DOM. Produces a single reusable HTML site template that recreates the full page — all visual values use CSS variables with fallbacks, so swapping the token CSS link instantly restyles the entire site. Invoke when user asks to extract page layout, full site structure, or says '逆向页面布局'/'抓页面结构'/'extract layout from URL'. Do NOT invoke for token-only extraction (use reverse-design-system)."
---

# Reverse Page Layout

Reverse-engineer any website's **complete page structure** from its rendered DOM — then produce a single reusable HTML **site template** that recreates the full page. All visual values use `var(--name, fallback)` syntax, so swapping one `<link>` to a different token CSS instantly restyles the entire site.

## Relationship to reverse-design-system

| Skill | Extracts | Output |
|-------|----------|--------|
| `reverse-design-system` | Colors, fonts, spacing, shadows, radius, transitions | `colors_and_type.css`, `css.json`, component contracts |
| `reverse-page-layout` (this) | Full page structure, section composition, navigation, UX flow, content layout | **`site-template.html`** (single complete page) |

**Combine**: Pick any token system (`colors_and_type.css`) + any site template (`site-template.html`) → change one CSS link → complete new website.

## When to Invoke

- User provides a URL and asks to extract page layout or full site structure
- User says "逆向页面布局" / "抓页面结构" / "extract layout from URL"
- User wants a reusable complete website template from a real website
- User wants to combine a token system with a layout to create a new site

## When NOT to Invoke

- User wants only colors/fonts/tokens → use `reverse-design-system`
- User wants to create a design system from scratch → use `design-library-creator`
- User wants page/UI design on canvas → use `solo-design`

## Output

A single file: `{BrandName}/site-template.html`

This file is a **complete, standalone website page** that:
1. Links to `colors_and_type.css` via relative path (same directory)
2. Uses `var(--name, fallback)` for ALL visual values (zero hardcoded colors/fonts)
3. Recreates the source website's full page structure (all sections in one HTML)
4. Contains real content from the source website (Chinese or original language)
5. Is responsive (desktop + mobile breakpoints)
6. Works when linked to ANY brand's `colors_and_type.css`

## Quick Map

| Phase | Action | Agent | Gate |
|-------|--------|-------|------|
| Phase 0 | Browser extraction — screenshot, computed styles, DOM structure, layout systems | Browser subagent | Gate 0 |
| Phase 1 | Page analysis — section map, content inventory, UX flow | Main agent | Gate 1 |
| Phase 2 | Site template generation — single HTML with all sections, CSS variables only | Main agent / subagent | Gate 2 |
| Phase 3 | Verification — open in browser, screenshot, compare to source | Browser subagent | Gate 3 |
| Phase 4 | Git deploy | Main agent | Gate 4 |

## Combination Usage

```powershell
# Create a combination: Brand A's layout + Brand B's tokens
New-Item -ItemType Directory -Force -Path ".design_library/combinations/{A}-layout_{B}-tokens"
$content = Get-Content ".design_library/{A}/site-template.html" -Raw
$content = $content -replace 'href="colors_and_type\.css"', 'href="../../{B}/colors_and_type.css"'
# If using dark-themed tokens (like Endfield), add class="dark" to body
Set-Content -Path ".design_library/combinations/{A}-layout_{B}-tokens/index.html" -Value $content -Encoding UTF8
```
