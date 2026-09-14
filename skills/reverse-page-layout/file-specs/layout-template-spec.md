# Layout Template HTML Specification

## Purpose

This document defines the structure, rules, and conventions for reusable HTML section templates produced by the `reverse-page-layout` skill. Each template is a self-contained HTML file (with an embedded `<style>` block) that captures a page-level layout pattern extracted from a real website. Templates use generic CSS variable names so they can be themed by any token system.

---

## Core Rules (CRITICAL)

These rules are non-negotiable. Any template that violates them must be regenerated before passing Gate 2.

### Rule 1: ALL styling uses CSS variables

NEVER hardcode colors, fonts, font-sizes, border-radius, shadows, or any visual property that a design token system would provide. Every visual value must reference a CSS variable.

**Forbidden:**
```css
color: #2563eb;              /* hardcoded color */
font-family: 'Inter', sans-serif;  /* hardcoded font */
font-size: 48px;            /* hardcoded size */
border-radius: 12px;        /* hardcoded radius */
box-shadow: 0 4px 6px rgba(0,0,0,0.1);  /* hardcoded shadow */
```

**Required:**
```css
color: var(--primary);
font-family: var(--font-family-display);
font-size: var(--font-size-h1);
border-radius: var(--radius-md);
box-shadow: var(--shadow-2);
```

### Rule 2: Layout properties are baked in

Structural layout values are baked directly into the template's `<style>` block — they are NOT tokenized. This includes:
- `grid-template-columns` (e.g., `repeat(3, 1fr)`, `1fr 1fr`)
- `gap` between grid/flex items (e.g., `24px`, `var(--space-6)`)
- `padding` on sections (e.g., `var(--space-12) var(--space-6)`)
- `max-width` on containers (e.g., `1200px`)
- `flex-direction` (e.g., `row`, `column`)
- `min-height` on hero sections (e.g., `600px`)
- `align-items`, `justify-content`

These values are layout-specific (they define the *structure*, not the *visual theme*) and are therefore part of the template itself.

### Rule 3: CSS variable naming

Templates use GENERIC variable names that any token system provides. The following variables are expected to be defined by the consuming token CSS:

**Colors:**
- `var(--primary)` — primary brand color
- `var(--primary-foreground)` — text on primary background
- `var(--secondary)` — secondary brand color
- `var(--background)` — page background
- `var(--foreground)` — primary text color
- `var(--muted)` — muted text color
- `var(--color-surface)` — elevated surface background
- `var(--border)` — border color

**Typography:**
- `var(--font-family-display)` — display/heading font
- `var(--font-family-body)` — body text font
- `var(--font-family-mono)` — monospace font
- `var(--font-size-h1)` — H1 size
- `var(--font-size-h2)` — H2 size
- `var(--font-size-h3)` — H3 size
- `var(--font-size-body)` — body text size
- `var(--font-size-lead)` — lead/subtitle text size
- `var(--font-size-caption)` — small/caption text size
- `var(--font-weight-h1)` — heading weight
- `var(--font-weight-body)` — body weight
- `var(--tracking-tight)` — tight letter-spacing for headings

**Spacing (optional — can be baked in or tokenized):**
- `var(--space-2)` through `var(--space-12)` — spacing scale

**Other:**
- `var(--radius-sm)`, `var(--radius-md)`, `var(--radius-lg)` — border radii
- `var(--shadow-1)`, `var(--shadow-2)`, `var(--shadow-3)` — box shadows
- `var(--transition-fast)` — quick transition timing

### Rule 4: Section structure

Each template is a complete `<section>` element with semantic HTML. Use appropriate semantic tags (`<nav>`, `<header>`, `<footer>`, `<article>`, `<aside>`, `<figure>`, `<figcaption>`) within the section where applicable.

### Rule 5: No JavaScript

Templates are pure HTML + CSS only. No `<script>` tags. No inline event handlers. No `data-*` attributes that imply JS behavior. Animations use CSS `@keyframes` and `:hover` pseudo-classes only.

### Rule 6: Responsive

Every template includes media queries for the breakpoints identified in the Phase 0 extraction. The standard breakpoints are:
- Mobile: `max-width: 375px`
- Tablet: `max-width: 768px`
- Desktop: `max-width: 1024px`
- Wide: `min-width: 1440px`

Templates must gracefully degrade on smaller screens (grids collapse, flex stacks vertically, hero shrinks, etc.).

---

## Template Structure

Each template file follows this structure:

```html
<!-- Section: Hero Centered -->
<!-- Source: https://example.com -->
<!-- Layout: centered hero with heading, subtext, CTA -->
<!-- Extracted: 2026-09-14 -->
<section class="section-hero-centered">
  <style>
    .section-hero-centered {
      min-height: 600px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: var(--space-6);
      padding: var(--space-12) var(--space-6);
      text-align: center;
      background: var(--background);
    }
    .section-hero-centered h1 {
      font-family: var(--font-family-display);
      font-size: var(--font-size-h1);
      font-weight: var(--font-weight-h1);
      line-height: 1.1;
      max-width: 800px;
      color: var(--foreground);
      letter-spacing: var(--tracking-tight);
      margin: 0;
    }
    .section-hero-centered .hero-subtext {
      font-family: var(--font-family-body);
      font-size: var(--font-size-lead);
      max-width: 600px;
      color: var(--foreground);
      opacity: 0.8;
      margin: 0;
    }
    .section-hero-centered .cta-group {
      display: flex;
      gap: var(--space-3);
      margin-top: var(--space-2);
    }
    .section-hero-centered .cta-primary {
      display: inline-flex;
      align-items: center;
      padding: var(--space-3) var(--space-6);
      background: var(--primary);
      color: var(--primary-foreground);
      font-family: var(--font-family-body);
      font-size: var(--font-size-body);
      font-weight: var(--font-weight-body);
      border-radius: var(--radius-md);
      border: none;
      cursor: pointer;
      transition: var(--transition-fast);
    }
    .section-hero-centered .cta-primary:hover {
      opacity: 0.9;
    }
    .section-hero-centered .cta-secondary {
      display: inline-flex;
      align-items: center;
      padding: var(--space-3) var(--space-6);
      background: transparent;
      color: var(--foreground);
      font-family: var(--font-family-body);
      font-size: var(--font-size-body);
      border: 2px solid var(--border);
      border-radius: var(--radius-md);
      text-decoration: none;
      transition: var(--transition-fast);
    }
    .section-hero-centered .cta-secondary:hover {
      border-color: var(--primary);
    }
    @media (max-width: 768px) {
      .section-hero-centered {
        min-height: 400px;
        padding: var(--space-8) var(--space-4);
      }
      .section-hero-centered .cta-group {
        flex-direction: column;
        width: 100%;
      }
      .section-hero-centered .cta-primary,
      .section-hero-centered .cta-secondary {
        justify-content: center;
      }
    }
    @media (max-width: 375px) {
      .section-hero-centered h1 {
        font-size: var(--font-size-h2);
      }
    }
  </style>
  <h1>{Heading text}</h1>
  <p class="hero-subtext">{Subtext that describes the value proposition in one or two sentences}</p>
  <div class="cta-group">
    <button class="cta-primary">{Primary CTA}</button>
    <a href="#" class="cta-secondary">{Secondary CTA}</a>
  </div>
</section>
```

---

## Section Types (Standard Set)

The following section types are the standard catalog. Each type has a fixed filename and CSS class name.

| # | Section Type | Filename | CSS Class | Description |
|---|---|---|---|---|
| 1 | Navigation | `section-nav.html` | `.section-nav` | Sticky/fixed navigation bar with logo, links, CTA |
| 2 | Hero Centered | `section-hero-centered.html` | `.section-hero-centered` | Centered hero with heading, subtext, CTA buttons |
| 3 | Hero Split | `section-hero-split.html` | `.section-hero-split` | Split layout: text content left, visual/image right |
| 4 | Feature Grid | `section-feature-grid.html` | `.section-feature-grid` | Grid of feature cards (2-4 columns) |
| 5 | Card Row | `section-card-row.html` | `.section-card-row` | Horizontal row of cards (scrollable or fixed) |
| 6 | Testimonials | `section-testimonials.html` | `.section-testimonials` | Testimonial quotes with avatar and attribution |
| 7 | Pricing | `section-pricing.html` | `.section-pricing` | Pricing tier cards with feature lists |
| 8 | CTA Banner | `section-cta-banner.html` | `.section-cta-banner` | Full-width call-to-action banner |
| 9 | Footer | `section-footer.html` | `.section-footer` | Footer with link columns, social, copyright |

### Naming Convention

- **File:** `section-{type}.html` (e.g., `section-hero-centered.html`)
- **CSS class:** `.section-{type}` (must match the filename's stem)
- **Self-contained:** Each file includes its own `<style>` block scoped to its class name
- **HTML comments at top:** Section name, source URL, layout description, extraction date

---

## Example: Section Nav Template

```html
<!-- Section: Navigation -->
<!-- Source: https://example.com -->
<!-- Layout: sticky nav with logo left, links center, CTA right -->
<!-- Extracted: 2026-09-14 -->
<section class="section-nav">
  <style>
    .section-nav {
      position: sticky;
      top: 0;
      z-index: 100;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: var(--space-4) var(--space-6);
      background: var(--background);
      border-bottom: 1px solid var(--border);
      max-width: 100%;
    }
    .section-nav .nav-inner {
      display: flex;
      align-items: center;
      justify-content: space-between;
      width: 100%;
      max-width: 1200px;
      margin: 0 auto;
      gap: var(--space-6);
    }
    .section-nav .nav-logo {
      font-family: var(--font-family-display);
      font-size: var(--font-size-h3);
      font-weight: var(--font-weight-h1);
      color: var(--foreground);
      text-decoration: none;
      white-space: nowrap;
    }
    .section-nav .nav-links {
      display: flex;
      align-items: center;
      gap: var(--space-6);
      list-style: none;
      margin: 0;
      padding: 0;
    }
    .section-nav .nav-links a {
      font-family: var(--font-family-body);
      font-size: var(--font-size-body);
      color: var(--foreground);
      text-decoration: none;
      opacity: 0.8;
      transition: var(--transition-fast);
    }
    .section-nav .nav-links a:hover {
      opacity: 1;
      color: var(--primary);
    }
    .section-nav .nav-cta {
      padding: var(--space-2) var(--space-4);
      background: var(--primary);
      color: var(--primary-foreground);
      font-family: var(--font-family-body);
      font-size: var(--font-size-body);
      border-radius: var(--radius-md);
      border: none;
      text-decoration: none;
      cursor: pointer;
      white-space: nowrap;
    }
    .section-nav .nav-mobile-toggle {
      display: none;
    }
    @media (max-width: 768px) {
      .section-nav .nav-links { display: none; }
      .section-nav .nav-mobile-toggle { display: flex; }
      .section-nav .nav-inner { gap: var(--space-3); }
    }
  </style>
  <div class="nav-inner">
    <a href="#" class="nav-logo">{Brand}</a>
    <nav>
      <ul class="nav-links">
        <li><a href="#">{Link 1}</a></li>
        <li><a href="#">{Link 2}</a></li>
        <li><a href="#">{Link 3}</a></li>
        <li><a href="#">{Link 4}</a></li>
      </ul>
    </nav>
    <a href="#" class="nav-cta">{CTA Label}</a>
    <button class="nav-mobile-toggle" aria-label="Menu">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <line x1="3" y1="6" x2="21" y2="6"/>
        <line x1="3" y1="12" x2="21" y2="12"/>
        <line x1="3" y1="18" x2="21" y2="18"/>
      </svg>
    </button>
  </div>
</section>
```

---

## Example: Section Feature Grid Template

```html
<!-- Section: Feature Grid -->
<!-- Source: https://example.com -->
<!-- Layout: 3-column auto-fit grid of feature cards -->
<!-- Extracted: 2026-09-14 -->
<section class="section-feature-grid">
  <style>
    .section-feature-grid {
      padding: var(--space-16) var(--space-6);
      background: var(--background);
    }
    .section-feature-grid .feature-grid-inner {
      max-width: 1200px;
      margin: 0 auto;
    }
    .section-feature-grid .feature-grid-header {
      text-align: center;
      margin-bottom: var(--space-12);
    }
    .section-feature-grid .feature-grid-header h2 {
      font-family: var(--font-family-display);
      font-size: var(--font-size-h2);
      font-weight: var(--font-weight-h1);
      color: var(--foreground);
      letter-spacing: var(--tracking-tight);
      margin: 0 0 var(--space-3);
    }
    .section-feature-grid .feature-grid-header p {
      font-family: var(--font-family-body);
      font-size: var(--font-size-lead);
      color: var(--muted);
      max-width: 600px;
      margin: 0 auto;
    }
    .section-feature-grid .feature-grid-cards {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: var(--space-6);
    }
    .section-feature-grid .feature-card {
      background: var(--color-surface);
      border: 1px solid var(--border);
      border-radius: var(--radius-md);
      padding: var(--space-6);
      transition: var(--transition-fast);
    }
    .section-feature-grid .feature-card:hover {
      box-shadow: var(--shadow-2);
      transform: translateY(-2px);
    }
    .section-feature-grid .feature-card .feature-icon {
      width: 48px;
      height: 48px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: var(--radius-md);
      background: var(--primary);
      color: var(--primary-foreground);
      margin-bottom: var(--space-4);
    }
    .section-feature-grid .feature-card h3 {
      font-family: var(--font-family-display);
      font-size: var(--font-size-h3);
      color: var(--foreground);
      margin: 0 0 var(--space-2);
    }
    .section-feature-grid .feature-card p {
      font-family: var(--font-family-body);
      font-size: var(--font-size-body);
      color: var(--muted);
      margin: 0;
      line-height: 1.6;
    }
    @media (max-width: 768px) {
      .section-feature-grid .feature-grid-cards {
        grid-template-columns: 1fr;
      }
    }
  </style>
  <div class="feature-grid-inner">
    <div class="feature-grid-header">
      <h2>{Section Heading}</h2>
      <p>{Section subtext describing the features below}</p>
    </div>
    <div class="feature-grid-cards">
      <article class="feature-card">
        <div class="feature-icon">{icon}</div>
        <h3>{Feature Title}</h3>
        <p>{Feature description text}</p>
      </article>
      <article class="feature-card">
        <div class="feature-icon">{icon}</div>
        <h3>{Feature Title}</h3>
        <p>{Feature description text}</p>
      </article>
      <article class="feature-card">
        <div class="feature-icon">{icon}</div>
        <h3>{Feature Title}</h3>
        <p>{Feature description text}</p>
      </article>
    </div>
  </div>
</section>
```

---

## layout-system.json Schema

In addition to the section template files, the skill generates a `layout-system.json` file that captures the page-level layout system metadata. This file is consumed by downstream tooling to understand the overall page structure.

```json
{
  "maxContentWidth": "1200px",
  "sectionSpacing": "64px 0",
  "containerPadding": "0 24px",
  "gridGap": "24px",
  "breakpoints": {
    "mobile": "375px",
    "tablet": "768px",
    "desktop": "1024px",
    "wide": "1440px"
  },
  "gridPatterns": [
    {
      "name": "3-col-auto",
      "template": "repeat(auto-fit, minmax(300px, 1fr))",
      "usage": "feature grids"
    },
    {
      "name": "2-col-50-50",
      "template": "1fr 1fr",
      "usage": "split hero"
    },
    {
      "name": "4-col-fixed",
      "template": "repeat(4, 1fr)",
      "usage": "card rows, stat bars"
    },
    {
      "name": "1-col-stack",
      "template": "1fr",
      "usage": "mobile layout, CTA banner"
    }
  ],
  "sectionOrder": [
    "section-nav",
    "section-hero-centered",
    "section-feature-grid",
    "section-testimonials",
    "section-cta-banner",
    "section-footer"
  ],
  "sourceUrl": "https://example.com"
}
```

---

## page-sections.json Schema

A companion index file that maps each extracted section to its template file and key properties:

```json
{
  "source": "https://example.com",
  "sections": [
    {
      "order": 0,
      "type": "section-nav",
      "templateFile": "section-nav.html",
      "layoutType": "flex",
      "layoutDetails": {
        "display": "flex",
        "justifyContent": "space-between",
        "alignItems": "center",
        "position": "sticky"
      },
      "height": 80,
      "backgroundColor": "var(--background)"
    },
    {
      "order": 1,
      "type": "section-hero-centered",
      "templateFile": "section-hero-centered.html",
      "layoutType": "flex",
      "layoutDetails": {
        "display": "flex",
        "flexDirection": "column",
        "alignItems": "center",
        "justifyContent": "center",
        "minHeight": "600px"
      },
      "height": 600,
      "backgroundColor": "var(--background)"
    }
  ],
  "totalSections": 6
}
```

---

## Validation Checklist (Gate 2)

Before a template is considered complete, verify:

- [ ] No hardcoded colors anywhere in the file (search for `#` in style block)
- [ ] No hardcoded font-family values (must use `var(--font-family-*)`)
- [ ] No hardcoded font-size values (must use `var(--font-size-*)`)
- [ ] No hardcoded border-radius values (must use `var(--radius-*)`)
- [ ] No hardcoded box-shadow values (must use `var(--shadow-*)`)
- [ ] Layout properties (grid-template, gap, padding, flex-direction, min-height) are baked in
- [ ] At least one `@media` query present for `max-width: 768px`
- [ ] HTML comment header with section name, source URL, and extraction date
- [ ] CSS class name matches filename stem (e.g., `.section-hero-centered` in `section-hero-centered.html`)
- [ ] Semantic HTML tags used appropriately (`<nav>`, `<article>`, `<section>`, `<footer>`)
- [ ] No `<script>` tags or inline JavaScript
- [ ] Placeholder text uses `{curly brace}` syntax for easy find-and-replace
