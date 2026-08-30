# Documentation Specification

## Purpose

This specification defines the format and content rules for three documentation artifacts produced by the reverse-design-system skill:

1. `README.md` — the human-readable system overview.
2. `SKILL.md` — the agent-facing quick-reference skill card.
3. `library-consumption.json` — the machine-readable consumption guide.

These documents are generated strictly from `colors_and_type.css`, `css.json`, `_evidence/`, and `uiCopySamples`. They must never duplicate component variant details or serve as a token reference table.

> **Note:** Two additional non-documentation files are also generated alongside the docs: `metadata.json` (library identity / provenance) and `quality-report.json` (validation results). These are NOT documentation files but identity/quality files that must exist alongside the docs. Their schemas are defined in `file-specs/metadata-and-quality.md`.

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

SKILL.md is the agent-facing quick-reference skill card. It MUST follow the exact structure below.

### YAML Frontmatter
```yaml
---
name: {brandname-lowercase}-design
description: Use this skill to generate well-branded interfaces and assets for {BrandName} — {brand design language summary}. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping {kit type} UIs.
user-invocable: true
---
```

- `name` — `{brandname-lowercase}-design` (e.g. `endfield-design`, `openai-design`).
- `description` — a real description of the brand's design language, not a generic template. Summarize the visual character (e.g. "industrial post-apocalyptic marketing landing experience") and the kit's intended use.

### Body Structure

The body MUST contain, in this order:

1. **Title line** — `# {BrandName} Design Skill`.
2. **Description paragraph** — one short paragraph explaining how to use the skill: read `README.md` first for design intent, copy assets from the library, link `colors_and_type.css` into HTML, etc.
3. **`## Quick map`** — a bulleted file navigation list (not a table) with these entries:
   - `README.md` — brand context, content fundamentals, visual foundations
   - `css.json` — structured token understanding source
   - `colors_and_type.css` — drop-in runtime CSS variables
   - `components.css` — aggregated component CSS
   - `components/index.json` — component index and cross-component patterns
   - resolved component sources — consume in priority order: preview first, then JSON, then `_evidence/` fallback
   - `preview/` — small HTML cards for each component
   - `library-consumption.json` — recommended downstream read order
4. **`## Essentials at a glance`** — 8–10 bullets of key design decisions, each with the real value:
   - Each bullet is one design decision with the real value, e.g.:
     - `Primary accent: --color-primary (#5B8CFF).`
     - `Body type: --font-body at --font-size-body, weight --font-weight-regular.`
   - ALL values from `colors_and_type.css`.

## library-consumption.json Structure

```json
{
  "brand": "Endfield",
  "language": "zh",
  "kitType": "marketing",
  "readOrder": [
    { "file": "README.md", "purpose": "Brand context, content fundamentals, visual foundations — read first for design intent" },
    { "file": "SKILL.md", "purpose": "Skill entry point and quick essentials" },
    { "file": "css.json", "purpose": "Structured token understanding source — read to understand color / type / spacing / radius values" },
    { "file": "colors_and_type.css", "purpose": "Runtime CSS variables — link into HTML; do not read for token understanding when css.json exists" },
    { "file": "components/index.json", "purpose": "Component index and cross-component patterns" },
    { "file": "components.css", "purpose": "Aggregated component CSS extracted from preview pages" }
  ],
  "components": ["button", "card", "input", "badge", "cta-link", "navigation"],
  "componentSourcePriority": {
    "first": "preview/component-{slug}.html",
    "second": "components/{slug}.json",
    "fallback": "components/_evidence/{slug}.json"
  },
  "notes": [
    "css.json is the token understanding source; colors_and_type.css is the runtime link source.",
    "Preview DOM/CSS is the first source for component fidelity; component JSON provides intent/variants only.",
    "Dark theme is default; .light class overrides tokens for light mode.",
    "All color, radius, and spacing values originate from colors_and_type.css — do not introduce values not present there."
  ]
}
```

### Field rules
- `brand` — the brand name (string).
- `language` — the primary content language of the library (e.g. `zh`, `en`).
- `kitType` — the kit type (e.g. `marketing`).
- `readOrder` — ordered array of objects, each with `file` + `purpose`, defining the recommended downstream read order for an agent.
- `components` — array of component slugs included in the library.
- `componentSourcePriority` — object with `first`/`second`/`fallback` keys defining the ranked source paths for component truth (highest priority first).
- `notes` — array of key consumption notes (strings, 3–5 entries).

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
