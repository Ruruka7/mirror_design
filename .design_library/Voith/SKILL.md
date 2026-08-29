---
name: voith-design
description: Use this skill to generate well-branded interfaces and assets for Voith — industrial engineering corporate marketing/landing experiences. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping website UIs.
user-invocable: true
---

# Voith Design Skill

Read the `README.md` file within this skill, and explore the other available files.

If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out and create static HTML files for the user to view. If working on production code, you can copy assets and read the rules here to become an expert in designing with this brand.

If the user invokes this skill without any other guidance, ask them what they want to build or design, ask some questions, and act as an expert designer who outputs HTML artifacts _or_ production code, depending on the need.

## Quick map

- `README.md` — brand context, content fundamentals, visual foundations (read first)
- `css.json` — structured token understanding source
- `colors_and_type.css` — drop-in runtime CSS variables; link it, do not read it to understand tokens when `css.json` exists
- `components/` — compact component contracts for intent and variants
- `components/_evidence/` — raw Figma evidence; use only when preview is insufficient
- `preview/` — small HTML cards illustrating foundations and components
- `ui_kits/{type}/` — full click-thru recreation (use as reference for layout, density, patterns)
- `library-consumption.json` — recommended downstream read order

> Consume component sources in priority order: `preview/component-{slug}.html` first, `components/{slug}.json` for intent/variants, and `components/_evidence/{slug}.json` as fallback evidence. Use `_evidence/` only when preview DOM/CSS is insufficient.

## Essentials at a glance

- **Primary brand blue:** `#00567E` (deep marine). Used for key actions, links, and headings; pairs with a white canvas.
- **Energy accent:** `#00E5E5` (cyan). Reserved for focus rings, hover states, and high-impact highlights — the only bright accent in an otherwise restrained palette.
- **Radius system:** `4px` for controls, `8px` for cards, `16px` for large surfaces, `9999px` only for pills/badges.
- **Control height:** buttons and inputs are `40px` by default; compact buttons are `32px`.
- **Spacing base:** `4px` with an `8px` grid — `16px`, `24px`, and `32px` are the workhorse steps.
- **Typography:** `Space Grotesk` for display and headings, `Inter` for body copy, `JetBrains Mono` for code.
- **Voice:** Chinese-first, bilingual when needed; professional, industrial, and clean — no emoji in product UI.
- **Shadows:** deep-blue-tinted elevations, five levels from `0 1px 2px` to `0 24px 60px`; used sparingly, never purely decorative.
- **Dark mode signature:** near-black `#0B1215` background with `cyan-400` accents and `blue-400` primary — preserves the marine engineering mood.
