---
name: "reverse-design-system"
description: "Evidence-first workflow for AI agents: given an authorized official website URL, automatically extract its visual language and generate a reusable design-system asset package. Invoke when user provides a URL and asks to reverse/extract/reconstruct a design system, or says '从官网逆向'/'从网站提取设计'/'reverse engineer design from URL'. Do NOT invoke for Figma exports, from-scratch systems without a reference URL, or page/UI design."
---

# Reverse Design System

An evidence-first, reusable workflow for AI agents. Given an authorized official website URL, extract the site's rendered visual language and turn it into a complete design-system asset package: tokens, component contracts, HTML previews, a marketing UI kit, documentation, and quality metadata.

## When to Invoke

- User provides a URL and asks to reverse-engineer / extract / reconstruct a design system from it
- User says "从官网逆向" / "从网站提取设计" / "reverse engineer design from URL"
- User wants a design system based on a real website's actual CSS values (not AI-guessed)
- User wants to replicate a website's design language as a reusable token system

## When NOT to Invoke

- User already has a Figma export, design spec bundle, or brand guide → use a design library generator directly
- User wants to create a design system from scratch without a reference URL → use a design library generator
- User wants to edit an existing design system → use a design library generator (refine route)
- User wants page/UI design, not a token system → use `solo-design`

## Prerequisites

- Target URL must be publicly accessible (no auth-required pages), and the user must be authorized to research and use the source material
- The host AI Agent must be able to read this workflow, inspect a rendered site, create files, and produce structured output.
- Browser automation is preferred; if it is unavailable, use user-provided screenshots or another permitted inspection method, record the limitation, and lower confidence.

## Read Order

| File | When | Purpose |
|------|------|---------|
| `SKILL.md` (this file) | Always | Route, trigger conditions, pipeline overview, protocol actions |
| `workflows/reverse-engineer.md` | After confirming skill | Full phase-by-phase execution with exact commands and gates |
| `file-specs/css-extraction.md` | Phase 0 | Extraction regex patterns, required token categories, output format |
| `file-specs/browser-extraction.md` | Phase 0A | Browser extraction JavaScript snippets, output format |
| `file-specs/dom-component-mapping.md` | Phase 0A | DOM→component type mapping rules |
| `file-specs/asset-extraction.md` | Phase 0A | Asset URL capture, SVG/font/image extraction |
| `file-specs/brand-profile.md` | Phase 1 | Brand profile JSON schema, field definitions, validation rules |
| `file-specs/token-css.md` | Phase 2 | Token CSS structure, naming conventions, color scale rules |
| `file-specs/component-contract.md` | Phase 3 | Component JSON schema, variant spec, anatomy rules |
| `file-specs/preview-page.md` | Phase 3 | Preview HTML structure, CSS link rules, layout constraints |
| `file-specs/uikit.md` | Phase 4 | UIKit HTML structure, section layout, component display rules |
| `file-specs/documentation.md` | Phase 4 | README structure, SKILL.md format, library-consumption schema |
| `file-specs/metadata-and-quality.md` | Phase 2 start + Phase 4 end | metadata.json and quality-report.json schemas |
| `operation-policies/quality-gates.md` | Between phases | Phase gate criteria, validator usage, acceptance checklist |
| `operation-policies/decision-rules.md` | Edge cases | SPA handling, auth-required sites, minimal CSS, language detection |
| `operation-policies/agent-dispatch.md` | Phase 2-4 | Sub-agent dispatch templates, parallel batch rules, query contracts |
| `operation-policies/git-deploy.md` | Phase 5 | Commit conventions, branch strategy, .gitignore rules |

> [FORBIDDEN] Main Agent MUST NOT pre-read sub-agent-only constraint files. Sub-agents receive those paths inside Task queries.

## Main Agent Read Budget

| Scope | Budget | Allowed Reads |
|-------|--------|---------------|
| Phase 0A entry | 1 | `file-specs/browser-extraction.md` |
| Phase 0B entry | 1 | `file-specs/css-extraction.md` |
| Phase 0C entry | 0 | no reads (merge logic is in workflow) |
| Phase 1 entry | 1 | `file-specs/brand-profile.md` |
| Phase 2 entry | 1 | `file-specs/token-css.md` |
| Phase 2 start | 1 | `file-specs/metadata-and-quality.md` |
| Phase 3 entry | 1 each | `file-specs/component-contract.md`, `file-specs/preview-page.md` |
| Phase 4 entry | 1 each | `file-specs/uikit.md`, `file-specs/documentation.md` |
| Phase 5 entry | 1 each | `operation-policies/quality-gates.md`, `operation-policies/git-deploy.md` |
| Edge cases | 1 | `operation-policies/decision-rules.md` |
| Sub-agent dispatch | 1 | `operation-policies/agent-dispatch.md` |

Hard stops:
- If Phase 0 extraction yields <3 distinct colors: stop, report sparse CSS, ask user for alternative URL
- If Phase 1 brand profile cannot determine a primary accent color: stop, ask user to specify
- Do NOT re-read file-specs once a phase is dispatched to sub-agents

## Sub-Agent Scope

| Files | Assigned To |
|------|-------------|
| `file-specs/browser-extraction.md` | Browser extraction sub-agent (browser_use) |
| `file-specs/dom-component-mapping.md` | Browser extraction sub-agent (component identification JS) |
| `file-specs/asset-extraction.md` | Browser extraction sub-agent (asset capture JS) |
| `file-specs/token-css.md` | Phase 2 token generation sub-agent |
| `file-specs/component-contract.md` | Phase 3 component data sub-agent |
| `file-specs/preview-page.md` | Phase 3 preview generation sub-agents (parallel) |
| `file-specs/uikit.md` | Phase 4 UIKit sub-agent |
| `file-specs/documentation.md` | Phase 4 documentation sub-agents |
| `operation-policies/agent-dispatch.md` | Main Agent only (dispatch orchestration) |
| `operation-policies/quality-gates.md` | Main Agent only (gate verification) |
| `operation-policies/decision-rules.md` | Main Agent only (edge case handling) |

## Protocol Actions

During reverse-engineering, Main Agent submits artifacts to the runtime via the following protocol actions ONLY when the runtime tool list explicitly exposes an action/tool of the same name:

| Action | When | Params |
|--------|------|--------|
| `generate_skill_files` | After each Phase completes | `{ files: [{ path: string, role: string }] }` |
| `update_tokens` | Token system assembled (end of Phase 2) | `{ tokensExtracted: number }` |
| `update_components` | Component data exported (end of Phase 3) | `{ componentsAnalyzed: number }` |
| `complete` | Finalize after all Gates pass (Phase 5) | `{ stats: { tokensExtracted, componentsAnalyzed, filesGenerated } }` |

## Pipeline Overview

```
Phase 0A: Browser Extraction        [Browser Subagent] ← PRIMARY
    Navigate → render → screenshots → computed styles → DOM walk → assets → responsive
    Deliverable: phase0a-browser-extraction.json
         │
         ▼
Phase 0B: CSS File Extraction      [Main Agent] ← SUPPLEMENT
    curl CSS files → regex extract → @font-face, :root vars, media queries
    Deliverable: phase0b-css-extraction.txt
         │
         ▼
Phase 0C: Merge & Summarize        [Main Agent]
    Merge browser + CSS data → produce key findings
    Deliverable: phase0-css-extraction.json (consolidated)
         │
         ▼
Phase 1: Brand Analysis            [Main Agent]
    Build brand profile from Phase 0 evidence
    Deliverable: phase1-brand-analyst.json (structured profile)
         │
         ▼
Phase 2: Token Generation          [Sub-Agent ×1]
    Generate colors_and_type.css → derive css.json
    Deliverable: colors_and_type.css, css.json
         │
         ▼
Phase 3: Components & Previews    [Main Agent + Sub-Agents ×3 parallel]
    Write component index + contracts → generate previews → extract components.css
    Deliverable: components/index.json, components/{slug}.json ×6, preview/*.html ×6, components.css
         │
         ▼
Phase 4: Documentation & UIKit    [Sub-Agents ×3 parallel → Sub-Agent ×1]
    README + SKILL + consumption → UIKit plan → UIKit HTML
    Deliverable: README.md, SKILL.md, library-consumption.json, ui_kits/marketing/index.html
         │
         ▼
Phase 5: Validate & Deploy         [Main Agent]
    Strip BOMs → validate → fix → git commit → git push
    Deliverable: validated, committed Design Library
```

## Key Principles

1. **No hallucination**: Every color, font, size, shadow value MUST come from the target website's real CSS. Never invent values. If extraction is sparse, report it rather than fabricating.
2. **Evidence-first**: Extract raw CSS data before making any design decisions. The data drives the system, not the other way around.
3. **Full delegation for generation**: Phases 2-4 use the design library generator pipeline exactly as-is. Do not reinvent token generation, component contracts, or preview authoring.
4. **BOM safety**: All generated files must be UTF-8 without BOM. Run a BOM-stripping pass before any validation or JSON parsing step.
5. **Git-ready**: Final output should be committable to a design-systems repository without modification. Commit messages follow conventional-commits format.
6. **Parallel where possible**: Component previews (Phase 3) and documentation (Phase 4) should be dispatched as parallel sub-agent batches to minimize wall-clock time.
7. **Browser-first**: Computed styles from a rendered page are the source of truth. CSS file declarations are supplementary. Never rely on CSS declarations alone when browser rendering is available.

## Output Location

```
{workspace}/.design_library/{BrandName}/
  colors_and_type.css           # Token system (all values from real CSS)
  css.json                      # Derived token JSON (auto-generated)
  metadata.json                 # Library identity (id, name, version)
  components.css                # Extracted component styles (auto-generated)
  components/
    index.json                  # Component index (6 standard components)
    button.json                 # Component contracts
    card.json
    input.json
    badge.json
    cta-link.json
    navigation.json
  preview/
    component-button.html        # Component previews
    component-card.html
    component-input.html
    component-badge.html
    component-cta-link.html
    component-navigation.html
  ui_kits/marketing/
    index.html                  # Marketing UI Kit
    quality-report.json         # UIKit quality metrics
  uikit-plan.json               # UIKit plan (auto-generated)
  README.md                     # Brand narrative documentation
  SKILL.md                      # Skill entry point
  library-consumption.json      # Downstream consumption guide
```

Phase 0A browser-extraction artifacts are written to a temp working directory (not the final library output):

```
{tmp_dir}/{BrandName}/
  screenshots/                 # Full-page and viewport screenshots captured during browser extraction
    full-page.png
    viewport-mobile.png
    viewport-tablet.png
    viewport-desktop.png
```

## Quick Start

1. Read `workflows/reverse-engineer.md` for full phase instructions
2. Read `file-specs/css-extraction.md` before Phase 0
3. Confirm the target URL with the user
4. Execute Phase 0 → Gate 0 → Phase 1 → Gate 1 → Phase 2 → Gate 2 → Phase 3 → Gate 3 → Phase 4 → Gate 4 → Phase 5
5. Each gate is defined in `operation-policies/quality-gates.md`
