---
name: mirror-design
description: Use this skill to generate interfaces and content surfaces in the Mirror / CONSTRUCT blog language — monochrome editorial layouts, square rules, compact metadata, and technical archive patterns for personal AI and open-source projects.
user-invocable: true
---

# Mirror Design Skill

Read `README.md` first for design intent and content tone. Then consume `css.json` for token understanding, link `colors_and_type.css` into HTML, and use the preview/component contracts as the reusable source for Mirror-style interfaces.

## Quick map

- `README.md` — brand context, content fundamentals, visual foundations
- `css.json` — structured token understanding source
- `colors_and_type.css` — drop-in runtime CSS variables
- `components.css` — aggregated component CSS
- `components/index.json` — component index and cross-component patterns
- `components/{slug}.json` — resolved component intent and evidence-backed variants
- `preview/` — small HTML cards for each component
- `library-consumption.json` — recommended downstream read order

## Essentials at a glance

- **Canvas:** `--color-background` is `#ffffff`; the default page is light and paper-like.
- **Ink:** `--color-primary` and `--color-foreground` resolve to `#0a0a0a`.
- **Secondary text:** `--color-text-secondary` uses the `#404040` family; muted surfaces use `#e5e5e5`.
- **Display type:** `--font-display` leads with `Inter Tight Variable` and falls back to `Space Grotesk Variable`.
- **Body type:** `--font-body` leads with `Inter Variable`; mono metadata uses `JetBrains Mono Variable`.
- **Headline scale:** use `--font-size-display` at `128px`, `--font-size-h1` at `48px`, then `24px` and `20px` section steps.
- **Control language:** `--size-nav-height` is `58px`; compact utilities use `36px`; large CTAs use `60px`.
- **Edges:** `--radius-none` through `--radius-xl` are `0px`; use square cards, buttons, tags, and rules.
- **Spacing:** `--space-1` is `4px`; the workhorse steps are `8px`, `16px`, `24px`, and `32px`.
- **Motion:** `--transition-fast` is `0.15s cubic-bezier(0.4, 0, 0.2, 1)`; prefer background, underline, and border changes over shadows.
