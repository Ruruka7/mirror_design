---
name: openai-design
description: Use this skill to generate well-branded interfaces for OpenAI. Contains colors, type, fonts, assets, and UI kit for prototyping marketing UIs.
user-invocable: true
---
# OpenAI Design Skill

Read the `README.md` file within this skill, and explore the other available files.

If creating visual artifacts, copy assets out and create static HTML files. If working on production code, read the rules here to become an expert in designing with this brand.

When resolving components, prefer `preview/component-{slug}.html` first, then `components/{slug}.json` for intent and variants. `components/_evidence/` is a fallback when preview is insufficient.

## Quick map

- `README.md` — brand context, content fundamentals, visual foundations (read first)
- `colors_and_type.css` — drop-in CSS variables for colors, type, radius, shadow, spacing
- `css.json` — structured token understanding source
- `components.css` — aggregated component styles extracted from preview pages
- `components/index.json` — component index + cross-component patterns
- `uikit-plan.json` — component whitelist and UIKit planner output
- `library-consumption.json` — recommended downstream read order
- `preview/` — small HTML cards illustrating foundations and components
- `ui_kits/marketing/` — full marketing landing-page recreation

## Essentials at a glance

- **Primary interactive color is black** `#000000` on white text for buttons and key CTAs; secondary actions use `rgba(0,0,0,0.04)` fill with black text.
- **Reserved accent** `#10a37f` (`--brand-green`) is kept for legacy/emphasis only; the current homepage rarely uses it.
- **Radius** `8 / 12 / 16 / 24 / 40 / 9999px` — 40px pill buttons, 24px input containers, 12–16px cards, 9999px badges/pills.
- **Default control heights:** 40px buttons, 40px inputs, 68px navigation; spacing unit is 4px.
- **Type:** custom `"OpenAI Sans SC", "OpenAI Sans", "OpenAI Sans Variable Scripts", sans-serif` stack; 28px display/H1, 20.78px H2, 16.78px H3, 17px body, 13px caption, 14px button. **Source Code Pro** for mono.
- **Voice:** Chinese-first marketing copy, professional, authoritative, and restrained; no emoji in product UI.
- **Shadows are extremely restrained** in light mode; the signature shadow belongs to the rounded input container. Cards are almost flat.
- **Signature pattern:** centered hero prompt "有什么可以帮忙的？" above a shadowed rounded input, flanked by black/gray pill suggestions, sparse 68px navigation.

## Components

| Slug | Name | Key Insight |
|------|------|-------------|
| button | Button | Primary black fill / white text, secondary near-transparent gray fill, both 40px pill radius; 32/40/48px sizes. |
| card | Card | Image-top editorial card, white background, 12–16px radius, title + meta, minimal shadow. |
| input | Input | 24px rounded container with layered shadow, borderless native input, muted placeholder. |
| badge | Badge | 9999px pill, 13px caption, light gray fill; for categorization and status. |
| cta-link | CTA Link | Text link with trailing arrow; primary black or muted, no underline at rest. |
| navigation | Navigation | 68px transparent bar, OpenAI logo, sparse links, search, login, "试用 ChatGPT" CTA. |
