# Documentation Specification

## Purpose

This specification defines the format and content rules for three documentation artifacts produced by the reverse-design-system skill:

1. `README.md` — the human-readable system overview.
2. `SKILL.md` — the agent-facing quick-reference skill card.
3. `library-consumption.json` — the machine-readable consumption guide.

These documents are generated strictly from `colors_and_type.css`, `css.json`, `_evidence/`, and `uiCopySamples`. They must never duplicate component variant details or serve as a token reference table.

## README.md Structure

The README MUST contain exactly the following sections, in this order, with no additional sections.

### 1. Overview (~10 lines)
- Brand name.
- Source URL the system was reverse-engineered from.
- A one-paragraph statement of what the system covers (color, type, spacing, 6 components, marketing patterns).

### 2. Content Fundamentals (~15 lines)
- Voice & tone description derived from `uiCopySamples`.
- Copy examples quoted directly from `uiCopySamples` — never fabricated.
- Copy generation rules (e.g. preferred sentence length, formality, punctuation habits observed in samples).

### 3. Visual Foundations (~50–60 lines)
This is the analytical core. It MUST be **all analytical prose with embedded values**, NOT tables. Each subsection weaves actual token values into sentences.

- **Color** — dominant color, accent color, the color scale, and semantic color usage. Name the actual token names and their roles.
- **Typography** — actual font family names (never "System Sans"), weights, the type scale, line-height values, and letter-spacing values.
- **Spacing** — base unit, the spacing scale, and standard component heights.
- **Radius** — radius scale and where each step is used.
- **Shadow** — shadow scale and elevation language.
- **Motion** — transition tokens and their duration/easing.

Every value cited MUST come from `colors_and_type.css`.

### 4. Component Patterns (~10 lines)
- A brief overview paragraph naming the 6 components and their one-line roles. No variant details.

### 5. Index (~8 lines)
- A file listing of the generated library (paths only, one line each).

### 6. Caveats / Known Substitutions (~10 lines)
- Font substitution warnings (e.g. if a brand font was unavailable and a fallback was used).
- Token gaps — any value that could not be derived and was inferred.

## README Rules

- **ALL values must come from `colors_and_type.css`** — zero hallucination. If a value is not in the token file, it does not appear in the README.
- **Copy examples MUST come from `uiCopySamples`** — never fabricated marketing copy.
- **Typography MUST name actual font families** — never use generic placeholders like "System Sans" or "sans-serif".
- **NO tables with more than 6 rows.** Prefer prose. Small reference tables (≤6 rows) are acceptable only when prose would be less clear.
- **NO installation section.** The system is a reverse-engineered library, not an installable package.
- **NO license section.**
- **NO API reference.**
- **NO content that duplicates SKILL.md.** The README is analytical prose; SKILL.md is a compact decision card.

## SKILL.md Structure

SKILL.md is the agent-facing quick-reference. It MUST be 25–50 lines.

### YAML Frontmatter
```yaml
---
name: reverse-design-system-{brand}
description: Reverse-engineered design system for {BrandName} covering color, type, spacing, and 6 components.
user-invocable: true
---
```

### Quick Map (file navigation table)
A small table (≤6 rows) listing key files and their purpose:
- `colors_and_type.css` — tokens
- `css.json` — structured token data
- `components/index.json` — component index
- `preview/` — component previews
- `ui_kits/marketing/index.html` — UI Kit

### Essentials (8–10 design decision bullets)
- Each bullet is one design decision with the real value, e.g.:
  - `Primary accent: --color-primary (#5B8CFF).`
  - `Body type: --font-body at --font-size-body, weight --font-weight-regular.`
- ALL values from `colors_and_type.css`.

## library-consumption.json Structure

```json
{
  "readOrder": [
    "colors_and_type.css",
    "css.json",
    "components/index.json",
    "components/button.json",
    "components/card.json",
    "components/input.json",
    "components/badge.json",
    "components/cta-link.json",
    "components/navigation.json",
    "preview/",
    "ui_kits/marketing/index.html"
  ],
  "componentSourcePriority": [
    "preview HTML",
    "components/{slug}.json",
    "_evidence/"
  ],
  "notes": [
    "Read colors_and_type.css first; every trait value is a var() reference into it.",
    "Prefer preview HTML for visual truth; fall back to component JSON for trait values.",
    "Consult _evidence/ only when a trait's origin is disputed."
  ]
}
```

### Field rules
- `readOrder` — ordered list of files/paths an agent should read to fully understand the system.
- `componentSourcePriority` — ranked sources for component truth, highest priority first.
- `notes` — key consumption notes (3–5 entries).

## Generation Constraints

- **Do NOT read the component directory or individual component JSON files when generating docs.** Documentation is generated from `colors_and_type.css`, `css.json`, and `uiCopySamples` only. Component contracts inform previews and the UI Kit, not the prose documentation.
- The documentation generator must remain agnostic to individual variant trait values — those belong to contracts and previews, not to README/SKILL prose.

## Forbidden Content

The following MUST NOT appear in README.md or SKILL.md:

- **Component variant details** — no enumeration of per-variant traits, states, or anatomy in the prose docs.
- **Token system tables** — no full token reference tables; values appear inline in prose only.
- **Color hex scale listings** — no full color swatch lists; describe dominant/accent/semantic colors in prose.
- **Typography tables** — no full type-scale tables; describe the scale in prose.
- **Version info** — no version numbers or changelogs.
- **Design principles** — no abstract principle sections; the system documents what IS, not philosophy.
- **Usage guide sections** — no step-by-step usage instructions; that belongs to SKILL.md essentials, not README.
