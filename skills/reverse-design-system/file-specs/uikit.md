# UI Kit HTML Specification

## Purpose

This specification defines the structure for `ui_kits/marketing/index.html`. The UI Kit is the consolidated, navigable showcase of the entire reverse-engineered design system: colors, typography, all six components, design tokens, and marketing page patterns rendered live. It must open directly in any browser with zero build step and zero external dependencies.

## CSS Link

```html
<link rel="stylesheet" href="../../colors_and_type.css">
```

The path MUST be exactly `../../colors_and_type.css` because `index.html` lives two levels below the library root. No alternate path is permitted.

## Required Sections (in order)

The body MUST contain these sections in exactly this order. Each section is an anchor target for the sidebar navigation.

### 1. UIKit Header
- Brand name.
- Library name.
- Label: `"Marketing UI Kit"`.
- Example:
  ```html
  <header id="top">
    <h1>{BrandName}</h1>
    <p>Marketing UI Kit</p>
  </header>
  ```

### 2. Color Palette Section
- All color groups from `colors_and_type.css` displayed as swatches.
- Each swatch shows: the token name, the hex value, and a filled background using that token.
- Group by semantic family (e.g. primary, surface, text, status).

### 3. Typography Section
- Display, headings, body, caption, and mono specimens.
- Each specimen shows: the actual font family name, weight, size, and line-height — rendered live, not as text descriptions.
- Use the brand `--font-*` and `--font-size-*` tokens.

### 4. Component Showcase
- All 6 components (button, card, input, badge, cta-link, navigation) rendered with their representative variants.
- Each component in its own card (see Component Display Rules below).

### 5. Design Tokens Section
- Visualizations of spacing, radius, shadow, and transition tokens.
- Spacing: sized boxes demonstrating each `--space-*` step.
- Radius: rounded rectangles demonstrating each `--radius-*` value.
- Shadow: surfaces demonstrating each `--shadow-*` value.
- Transition: an element demonstrating `--transition-*` on `:hover`.

### 6. Marketing Patterns
- Hero section mock — using brand layout, headline, and CTA.
- Card grid — 3-up card layout.
- CTA section — full-width accent band with a primary button.
- Footer — multi-column footer with links.

## Sidebar Navigation

- Fixed left sidebar, `width: 220px`.
- Links to each section using anchor `href="#section-id"` with smooth scroll (`scroll-behavior: smooth` on `html`).
- Active section highlight (via `:target` pseudo-class or CSS-only approach; NO JavaScript).
- Sidebar collapses on mobile (below a breakpoint, e.g. `768px`): becomes a top bar or hidden, with main content taking full width.

## Layout

- Main content area: `margin-left: 220px`, `max-width: 960px`, `padding: 48px`.
- Dark background matching brand `--background` token.
- Responsive: below the mobile breakpoint, `margin-left: 0` and sidebar collapses.
- Sidebar is `position: fixed`, `top: 0`, `left: 0`, `height: 100vh`, `overflow-y: auto`.

## Component Display Rules

- Each of the 6 components is shown in a "card" — a `.ui-kit-card` container with:
  - A label (component name).
  - A variant list — each representative variant rendered with actual CSS.
- Variants are rendered with live CSS (NOT screenshots). The trait values from the component contracts are applied directly.
- Hover and focus states are demonstrated with `:hover` / `:focus` selectors so a reviewer can see the interaction live.
- All values use CSS variables from `colors_and_type.css`.

## Quality Rules

- **NO inline styles** EXCEPT for the sidebar width (a hardcoded `220px` fallback is acceptable for the fixed sidebar; everywhere else use tokens).
- **NO external resources** besides `colors_and_type.css`. No CDN, no web fonts, no images, no JS libraries.
- **NO JavaScript** — pure CSS only. Interactivity (hover, focus, active section highlight) must be CSS-only.
- **All text readable** against backgrounds. Use the brand's paired `--color-on-*` tokens.
- The UI Kit MUST render correctly when opened directly in a browser via `file://` — no server, no build, no network.

## Structure Skeleton

```html
<!DOCTYPE html>
<html lang="{brand-language}">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{BrandName} — Marketing UI Kit</title>
  <link rel="stylesheet" href="../../colors_and_type.css">
</head>
<body>
  <aside class="ui-kit-sidebar" style="width:220px">
    <nav>
      <a href="#top">Header</a>
      <a href="#colors">Color Palette</a>
      <a href="#typography">Typography</a>
      <a href="#components">Components</a>
      <a href="#tokens">Design Tokens</a>
      <a href="#patterns">Marketing Patterns</a>
    </nav>
  </aside>
  <main class="ui-kit-main">
    <header id="top"><!-- UIKit Header --></header>
    <section id="colors"><!-- Color Palette --></section>
    <section id="typography"><!-- Typography --></section>
    <section id="components"><!-- Component Showcase --></section>
    <section id="tokens"><!-- Design Tokens --></section>
    <section id="patterns"><!-- Marketing Patterns --></section>
  </main>
</body>
</html>
```

## Validation Checklist

- [ ] CSS link path is exactly `../../colors_and_type.css`.
- [ ] All 6 sections present in the specified order.
- [ ] All 6 components rendered with live CSS (not screenshots).
- [ ] Sidebar is 220px fixed and collapses on mobile.
- [ ] No JavaScript present anywhere.
- [ ] No external resources besides the token CSS.
- [ ] All `var()` references resolve in `colors_and_type.css`.
- [ ] Page renders correctly via `file://`.
