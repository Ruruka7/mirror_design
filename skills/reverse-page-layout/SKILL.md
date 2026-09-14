---
name: "reverse-page-layout"
description: "Reverse-engineer a website's page structure, layout patterns, UX flows, and UI composition from its rendered DOM. Produces reusable HTML layout templates that use CSS variables — combine with any design token system to generate a complete website. Invoke when user asks to extract page layout, UI patterns, UX flows, or says '逆向页面布局'/'抓页面结构'/'extract layout from URL'. Do NOT invoke for token-only extraction (use reverse-design-system)."
---

# Reverse Page Layout

Reverse-engineer any website's **page structure, layout systems, UX flows, and UI composition** from its rendered DOM — then produce reusable HTML layout templates that can be combined with any design token system.

## Relationship to reverse-design-system

| Skill | Extracts | Output |
|-------|----------|--------|
| `reverse-design-system` | Colors, fonts, spacing, shadows, radius, transitions | `colors_and_type.css`, `css.json`, component contracts |
| `reverse-page-layout` (this) | Page sections, layout grids, component placement, UX flows, interaction patterns | `layouts/*.html`, `page-sections.json`, `layout-system.json`, `ux/user-journey.json` |

**Combine**: Pick any token system + any layout system → generate a complete new website.

## When to Invoke

- User provides a URL and asks to extract page layout, UI patterns, or UX flows
- User says "逆向页面布局" / "抓页面结构" / "extract layout from URL"
- User wants reusable HTML layout templates from a real website
- User wants to combine a token system with a layout system to create a new site

## When NOT to Invoke

- User wants only colors/fonts/tokens → use `reverse-design-system`
- User wants to create a design system from scratch → use `design-library-creator`
- User wants page/UI design on canvas → use `solo-design`

## Prerequisites

- Browser automation capability (browser_use subagent or browser_navigate/evaluate/screenshot)
- Target URL must be publicly accessible
- Optional: existing token CSS for previewing layouts (if combining)

## Read Order

| File | When | Purpose |
|------|------|---------|
| `SKILL.md` (this file) | Always | Route, pipeline overview |
| `workflows/extract-layout.md` | After confirming skill | Full phase-by-phase execution |
| `file-specs/page-section-extraction.md` | Phase 0 | DOM section extraction JavaScript, output format |
| `file-specs/layout-template-spec.md` | Phase 2 | HTML template structure, CSS variable rules, naming |
| `file-specs/ux-flow-spec.md` | Phase 3 | UX journey JSON schema, interaction pattern catalog |
| `operation-policies/quality-gates.md` | Between phases | Gate criteria |
| `operation-policies/decision-rules.md` | Edge cases | SPA, lazy-load, multi-page handling |

## Pipeline Overview

```
Phase 0: Browser Page Extraction       [Browser Subagent]
    Navigate → render → screenshot → extract sections → extract layouts → extract interactions
    Deliverable: phase0-page-extraction.json
         │
         ▼
Phase 1: Layout Analysis               [Main Agent]
    Analyze section hierarchy, identify layout patterns, map component placement
    Deliverable: phase1-layout-analysis.json
         │
         ▼
Phase 2: Layout Template Generation    [Sub-Agents ×3 parallel]
    Generate reusable HTML templates per section type (hero, feature grid, CTA, footer, etc.)
    All styling uses CSS variables — NO hardcoded colors/fonts
    Deliverable: layouts/*.html, layout-system.json
         │
         ▼
Phase 3: UX Flow & Interaction Mapping [Sub-Agent ×1]
    Map user journey, interaction patterns, scroll behavior, navigation flow
    Deliverable: ux/user-journey.json, ux/interaction-patterns.json
         │
         ▼
Phase 4: Preview & Validate            [Main Agent]
    Preview each template with a sample token system → validate → fix
    Deliverable: layouts/preview.html (combined preview)
         │
         ▼
Phase 5: Deploy                        [Main Agent]
    Git commit → push
```

## Key Principles

1. **Layout-only, no styling**: Templates use CSS variables exclusively. Zero hardcoded colors, fonts, or visual styling. Only layout properties (grid, flex, gap, padding, max-width) are baked in.
2. **Real structure**: Every layout template must mirror the actual DOM structure from the target site — real section order, real grid patterns, real component placement.
3. **Composable**: Any `layouts/*.html` file must work with ANY `colors_and_type.css` token system. Link a different token CSS → get a different visual style on the same layout.
4. **Section-based**: Layouts are organized by page section (hero, features, testimonials, pricing, CTA, footer), not as one monolithic page. Users pick sections to assemble.
5. **Responsive captured**: Each template includes media queries extracted from the real site's breakpoint behavior.

## Output Location

```
{workspace}/.design_library/{BrandName}/
  layouts/
    page-sections.json           # Section structure map (order, type, height, layout)
    layout-system.json           # Layout system tokens (grid templates, gaps, max-widths)
    section-hero.html            # Hero section template (CSS vars only)
    section-nav.html             # Navigation template
    section-features.html        # Feature grid/card row template
    section-testimonials.html    # Testimonial template (if exists)
    section-pricing.html         # Pricing template (if exists)
    section-cta.html             # CTA banner template
    section-footer.html          # Footer template
    preview.html                 # Combined preview (all sections with sample tokens)
  ux/
    user-journey.json            # User flow map
    interaction-patterns.json    # Interaction catalog
```

## Quick Start

1. Read `workflows/extract-layout.md` for full instructions
2. Confirm target URL and brand name
3. Execute Phase 0 → Gate 0 → Phase 1 → Gate 1 → Phase 2 → Gate 2 → Phase 3 → Gate 3 → Phase 4 → Gate 4 → Phase 5
