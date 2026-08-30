---
name: "reverse-design-system"
description: "Reverse-engineer a website's CSS to extract real design tokens, then generate a complete Design Library through the design-library-creator pipeline. Invoke when user provides a URL and asks to reverse/extract/reconstruct a design system from a live website."
---

# Reverse Design System

Reverse-engineer any website's visual design language from its live CSS, then generate a complete Design Library using the `design-library-creator` pipeline.

## When to Invoke

- User provides a URL and asks to reverse-engineer / extract / reconstruct a design system from it
- User says "从官网逆向" / "从网站提取设计" / "reverse engineer design from URL"
- User wants a design system based on a real website's actual CSS values (not AI-guessed)

## When NOT to Invoke

- User already has a Figma export, design spec bundle, or brand guide → use `design-library-creator` directly
- User wants to create a design system from scratch without a reference URL → use `design-library-creator`
- User wants to edit an existing design system → use `design-library-creator` (refine route)
- User wants page/UI design, not a token system → use `solo-design`

## Prerequisites

- Target URL must be publicly accessible (no auth-required pages)
- `design-library-creator` skill must be available in the environment
- Node.js runtime with the design-library-creator scripts available at:
  `c:\Users\25230\.trae-cn\builtin\design\default\skills\design-library-creator\scripts\`

## Read Order

| File | When | Purpose |
|------|------|---------|
| `SKILL.md` (this file) | Always | Route, trigger conditions, pipeline overview |
| `workflows/reverse-engineer.md` | After confirming this skill | Full phase-by-phase execution instructions |

## Pipeline Overview

```
Phase 0: Fetch & Extract     ← UNIQUE to this skill
    curl target website → parse CSS → extract real tokens
    (colors, fonts, sizes, borders, shadows, gradients, clip-paths, transitions)
         │
         ▼
Phase 1: Brand Analysis      ← Bridge to design-library-creator
    Build brand profile from extracted data
    (productType, personality, visualTone, language, kitType, uiCopySamples)
         │
         ▼
Phase 2-4: Design Library Generation  ← Delegated to design-library-creator
    Token CSS → css.json → Components → Previews → Docs → UIKit
         │
         ▼
Phase 5: Validate & Deploy   ← Post-processing
    Strip BOMs → validate → git commit → git push
```

## Key Principles

1. **No hallucination**: Every color, font, size, shadow value MUST come from the target website's real CSS. Never invent values.
2. **Evidence-first**: Extract raw CSS data before making any design decisions. The data drives the system.
3. **Full delegation for generation**: Phases 2-4 use the `design-library-creator` pipeline exactly as-is. Do not reinvent token generation, component contracts, or preview authoring.
4. **BOM safety**: All generated files must be UTF-8 without BOM. Run a BOM-stripping pass before validation.
5. **Git-ready**: Final output should be committable to a design-systems repository without modification.

## Quick Start

1. Read `workflows/reverse-engineer.md` for full phase instructions
2. Confirm the target URL with the user
3. Execute Phase 0 (fetch + extract)
4. Execute Phase 1 (brand analysis)
5. Hand off to `design-library-creator` pipeline for Phases 2-4
6. Execute Phase 5 (validate + deploy)

## Output Location

```
{workspace}/.design_library/{BrandName}/
  colors_and_type.css      # Token system (from real CSS values)
  css.json                 # Derived token JSON
  components.css            # Extracted component styles
  components/
    index.json             # Component index
    {slug}.json            # Individual component contracts
  preview/
    component-{slug}.html   # Component previews
  ui_kits/marketing/
    index.html             # Marketing UI Kit
  README.md                # Brand narrative
  SKILL.md                 # Skill entry point
  library-consumption.json # Consumption guide
```
