# Voith Design System

A design system reconstruction of **Voith** — the digital presence of an industrial engineering group serving paper, energy, and transport markets.
The system is purpose-built for Chinese-language marketing and landing pages that must read trustworthy, engineering-led, and clean.

> *"世纪渊源，植根发展"*

## Source

- **Brand owner:** Voith Group / 福伊特集团
- **Language:** Chinese (zh) with occasional English navigational labels
- **Product type:** Marketing / Landing
- **Kit type:** website

## What this design system covers

- **Foundations** — marine-cyan color logic, Space Grotesk + Inter typography, 4px spacing base, 4/8/12/16/9999px radius, five shadow levels
- **Components** — 6 documented components: Button, Card, Input, Badge, CTA Link, Navigation
- **Previews** — standalone HTML cards under `preview/`

# CONTENT FUNDAMENTALS

## Voice & tone

The voice is formal, institutional, and third-person: the brand speaks as a long-standing engineering authority rather than a chatty product. Copy is concise and noun-driven, preferring section titles and label-style phrasing over long persuasive sentences. Chinese dominates the UI, while English appears only in navigational asides such as "Locations in China". Emoji, exclamation marks, and casual contractions are absent. The tone should feel like a corporate annual report translated into interface copy: precise, proud, and quietly global.

### Concrete copy examples (from the source)

- Hero/tagline: *"世纪渊源，植根发展"*
- Primary navigation: *"关于福伊特集团"*
- Careers section: *"职业机会"*
- News section: *"企业新闻"*
- Compliance: *"合规服务台"*
- Global locator: *"Locations in China"*

### When generating copy

- Keep labels in Chinese unless the source explicitly uses English.
- Prefer institution-level verbs (发展、服务、解决方案) over conversion-driven verbs.
- Avoid product-hype adjectives; let nouns and numbers carry authority.
- Do not invent CTA labels like "立即购买"; use measured CTAs such as "了解更多" or "了解详情".

# VISUAL FOUNDATIONS

### Color

The palette is built around deep marine blues on a white canvas, with a single high-energy cyan accent and a restrained neutral scale. The brand primary resolves to `--voith-blue-600` (#00567e), a saturated navy used for navigation active states, primary buttons, links, and headline emphasis. The accent is `--voith-cyan-500` (#00e5e5), a near-neon cyan that powers focus rings, hover fills, and attention badges without drifting into playful territory.

The blue scale runs from `--voith-blue-50` (#e8f4f9) to `--voith-blue-900` (#03223c), a deliberate deep-to-pale sequence that supports everything from subtle backgrounds to hero headline color. The cyan scale is similarly full-spectrum, from `--voith-cyan-50` (#e6fcfc) to `--voith-cyan-900` (#024b5a), but it is used sparingly so it never competes with the primary navy. Neutrals span `--voith-neutral-50` (#f8f8f8) through `--voith-neutral-900` (#191c1d), with `--voith-neutral-200` (#e8eaea) serving as the default rule and border color, `--voith-neutral-500` (#a9acac) for muted placeholders, and `--voith-neutral-700` (#747778) for secondary text.

Semantic colors are drawn from the same family logic: success resolves to `--voith-success-600` (#0d8f59), warning to `--voith-warning-600` (#d99e00), error to `--voith-error-600` (#d9002b), and info aligns with the cyan line at `--voith-info-500` (#05a2c2). The overall vibe is cool, technical, and confident — color is used to guide hierarchy rather than decorate.

### Typography

The system imports two faces from Google Fonts: **Space Grotesk** (500/600/700) for display and headings, and **Inter** (400/500/600/700) for body and UI text. **JetBrains Mono** is specified for monospace code or data snippets. This pairing gives the industrial brand a contemporary geometric headline without sacrificing readability in long body passages.

Space Grotesk carries display text at 68px/700 with a tight -0.02em letter-spacing, h1 at 40px/700, and h2-h4 at 32px/24px/20px with weights 600-700. Inter handles body at 16px/400 with a 1.6 line-height, lead at 18px, caption at 12px, and eyebrow labels at 11px/600 with 0.08em letter-spacing and uppercase transform. Monospace is set at 14px/400. The strategy is high contrast in weight between headings and body, not size alone, which keeps the page feeling editorial rather than sales-y.

### Spacing

The base unit is 4px. Tokens run `--space-1` (4px), `--space-2` (8px), `--space-3` (12px), `--space-4` (16px), `--space-5` (24px), `--space-6` (32px), `--space-7` (40px), and `--space-8` (48px). Default controls follow a 32/40/48px height model: small buttons are 32px, medium buttons and inputs are 40px, and large buttons are 48px. Navigation is taller at 72px to anchor the page. Cards use 24px-32px internal padding, creating generous whitespace that keeps the engineering content from feeling dense.

### Radius

Radius is intentionally restrained. `--radius-sm` is 4px for tight controls and small tags; `--radius-md` is 8px for buttons, inputs, and tinted cards; `--radius-lg` is 12px for default cards; `--radius-xl` is 16px for larger surfaces; and `--radius-full` is 9999px reserved for pills such as badges. There are no 20px or 24px radius tokens — the system prefers crisp edges softened only slightly.

### Shadow / Elevation

There are five shadow layers, all cast in a deep navy alpha so they feel cool rather than warm. Level 1 (`--shadow-1`) is `0 1px 2px rgba(3,34,60,0.06), 0 1px 1px rgba(3,34,60,0.04)` and is used for cards at rest. Level 2 (`--shadow-2`) is `0 4px 8px -2px rgba(3,34,60,0.10)` for card hover. Level 3 (`--shadow-3`) is `0 8px 24px -8px rgba(3,34,60,0.18)` for floating elements. Level 4 (`--shadow-4`) is `0 16px 40px -12px rgba(3,34,60,0.24)` for modals. Level 5 (`--shadow-5`) is `0 24px 60px -20px rgba(3,34,60,0.30)` for overlays. The philosophy is whisper-quiet at rest and decisive at the top layer; no heavy ambient shadow is used for ordinary surfaces.

### Borders, backgrounds, animation, iconography

Borders are consistently 1px and use `--voith-neutral-200` for rules and card borders, switching to `--voith-neutral-800` in dark mode. Backgrounds default to white (`#ffffff`) with `--voith-blue-50` as the tinted surface for highlighted cards; dark mode flips to `#0b1215` with light text. Motion is minimal: transitions are 0.15s for background, border-color, color, and filter, plus a 2px solid cyan focus ring with 2px offset. Iconography follows three sizes — 16px, 24px, and 32px — and is implemented as inline SVG or symbol references; the current CTA Link preview uses a 16px arrow icon from `lucide-static` via CDN.

# Component Patterns

| Component | Preview | Contract | CSS Source | Key Facts | Key Insight |
|---|---|---|---|---|---|
| Button | `preview/component-button.html` | `components/button.json` | `components.css` — Button | 32/40/48px heights; primary/secondary/ghost variants; focus ring 2px cyan | Primary fills use navy #00567e; hover shifts to cyan #05a2c2, turning a utility color into the brand signature |
| Card | `preview/component-card.html` | `components/card.json` | `components.css` — Card | Default card 12px radius + shadow-1; tinted variant 8px radius + blue-50 background | Cards are content-forward: eyebrow, h3, body, CTA link in a single vertical rhythm |
| Input | `preview/component-input.html` | `components/input.json` | `components.css` — Input | 40px height, 240px default width, hover-to-cyan focus ring | Error state uses a subtle red-50 fill rather than a loud border, keeping the form industrial-calm |
| Badge | `preview/component-badge.html` | `components/badge.json` | `components.css` — Badge | 28px min-height pill, 4px/12px padding, 12px caption text | Color is semantic and legible on light backgrounds; no dark badge variant is defined |
| CTA Link | `preview/component-cta-link.html` | `components/cta-link.json` | `components.css` — Cta Link | Default/accent/subtle styles; 16px arrow icon slot; underline on hover for default | The default CTA underlines only on hover, matching the restrained "discover more" voice of the brand |
| Navigation | `preview/component-navigation.html` | `components/navigation.json` | `components.css` — Navigation | 72px height, logo left, links center, utility right; active link gets 2px cyan bottom border | Nav hover uses the cyan accent as a background wash, making the entire bar feel responsive without changing type color |

# Index

- `README.md` — this file
- `colors_and_type.css` — CSS variables for color, type, radius, shadow, and spacing
- `components.css` — aggregated component CSS extracted from preview pages
- `components/` — component contract JSON files and index
- `preview/` — small HTML cards for each documented component
- `SKILL.md` — agent skill manifest
- `css.json` — structured token representation for programmatic consumption
- `library-consumption.json` — recommended downstream read order for agents

# Caveats / known substitutions

1. **Space Grotesk and Inter** are loaded from Google Fonts CDN. If the environment blocks external fonts, the fallbacks are Arial for body and the browser's default sans-serif for headings, which loses the geometric character of Space Grotesk.
2. **JetBrains Mono** is referenced as the monospace face but is not imported in `colors_and_type.css`; in practice it will only render if already installed locally or if the consuming project adds the import.
3. The Lucide arrow icon in the CTA Link preview is pulled from `lucide-static` CDN. For offline or locked-down builds, replace it with an equivalent inline SVG.
4. Several color stops are marked `/* AI-generated */` in the CSS; they form a coherent scale but should be validated against official Voith brand guidelines before production use.
5. The dark mode values are inferred from the light palette and may not match Voith's actual dark-theme specifications.
6. Input width is fixed at 240px in the preview CSS; real forms should override this with responsive widths or grid-based sizing.
