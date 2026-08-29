---
name: endfield-design
description: Use this skill to generate well-branded interfaces and assets for Endfield — industrial post-apocalyptic marketing landing experience. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping marketing UIs.
user-invocable: true
---

# Endfield Design Skill

Read the `README.md` file within this skill, and explore the other available files.

If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out
and create static HTML files for the user to view. If working on production code, you can
copy assets and read the rules here to become an expert in designing with this brand.

If the user invokes this skill without any other guidance, ask them what they want to build
or design, ask some questions, and act as an expert designer who outputs HTML artifacts
_or_ production code, depending on the need.

## Quick map

- `README.md` — brand context, content fundamentals, visual foundations (read first)
- `css.json` — structured token understanding source (read this to understand tokens)
- `colors_and_type.css` — drop-in runtime CSS variables; link it, do not read it to understand tokens when css.json exists
- `components.css` — aggregated component CSS extracted from preview pages
- `components/index.json` — component index and cross-component patterns
- resolved component sources — consume in priority order: `preview/component-{slug}.html` first, `components/{slug}.json` for intent/variants, and `components/_evidence/{slug}.json` as fallback evidence. UIKit may read `_evidence/` when preview is insufficient, but preview DOM/CSS remains the first source when present.
- `preview/` — small HTML cards illustrating the foundations and components
- `library-consumption.json` — recommended downstream read order

## Essentials at a glance

- Primary `#fffa00` hazard yellow on near-black `#191919` ground; neon green `#00ffa2` and magenta `#ff1aac` are the only secondaries — no warm accents, no default gradients.
- Radius is 0 / 4 / 8 — geometric and restrained; 42px reserved for pill status chips only, never cards or containers.
- Density first: 44px input height, 32 / 40 / 48px buttons; spacing base 10px scaling 10 → 80.
- Type: **Novecentosanswide** for display headings, **Gilroy** for body (fallback Segoe UI / Roboto / PingFang SC), **SpaceGrotesk** for mono / eyebrow labels, **ProtestStrike** for mega impact.
- Voice: bilingual CN-first, industrial, terse, no emoji — "终末地 / ENDFIELD".
- Shadows are glow-based, never soft drop-shadow at rest; brand glow `0 0 10px #fff000` signals primary / hover actions.
- Signature: geometric clip-path cutouts + diagonal hazard stripe patterns on surfaces — angular, never rounded.
- Motion overshoots: `cubic-bezier(0.2, 0, 0.13, 1.13)` emphasized ease for high-energy feedback.
