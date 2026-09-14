# Reverse-to-Site — Full Pipeline

## Overview

Given a website URL, produce two portable assets: (1) `colors_and_type.css` — a token file with all visual values as CSS variables using portable names, and (2) `site-template.html` — a complete standalone HTML page recreating the source site's full structure using only `var(--name, fallback)` references. These two assets are decoupled: any token file works with any template, enabling cross-brand combinations by changing a single `<link>` tag.

## Prerequisites

- Browser access (for page rendering and computed style extraction)
- File write access (for creating HTML/CSS files)
- Git access (for version control, optional)

---

## Phase 1: Token Extraction

### 1.1 Browser Extraction

Navigate to the target URL, wait for full render, then extract computed styles via JavaScript.

**Steps:**

1. Navigate to target URL
2. Wait for `networkidle` (all resources loaded, no pending requests)
3. Scroll through entire page to trigger lazy-loaded content
4. Wait 2 seconds for final render settle
5. Take full-page screenshot (for visual reference)
6. Run the computed style extraction script below via `browser_evaluate`

**Computed Style Extraction JavaScript:**

```javascript
(function() {
  const results = {
    colors: new Map(),
    fonts: new Map(),
    fontSizes: new Map(),
    fontWeights: new Map(),
    lineHeight: new Map(),
    letterSpacing: new Map(),
    borderRadius: new Map(),
    boxShadows: new Map(),
    backgrounds: new Map(),
    paddings: new Map(),
    margins: new Map(),
    gaps: new Map(),
    transitions: new Map(),
  };

  const allElements = document.querySelectorAll('*');

  allElements.forEach(el => {
    const cs = window.getComputedStyle(el);

    function tally(map, value) {
      if (!value || value === 'initial' || value === 'inherit' || value === 'none' || value === 'normal') return;
      if (value === 'rgba(0, 0, 0, 0)' || value === 'transparent') return;
      map.set(value, (map.get(value) || 0) + 1);
    }

    // Colors: background, text, border
    tally(results.colors, cs.backgroundColor);
    tally(results.colors, cs.color);
    tally(results.colors, cs.borderColor);
    tally(results.colors, cs.borderTopColor);
    tally(results.colors, cs.borderBottomColor);
    // Extract colors from box-shadow
    if (cs.boxShadow && cs.boxShadow !== 'none') {
      const shadowColors = cs.boxShadow.match(/rgba?\([^)]+\)|#[0-9a-fA-F]{3,8}/g);
      if (shadowColors) shadowColors.forEach(c => tally(results.colors, c));
    }
    // Extract colors from background gradients
    if (cs.background && cs.background !== 'none' && cs.background !== 'rgba(0, 0, 0, 0)') {
      const bgColors = cs.background.match(/rgba?\([^)]+\)|#[0-9a-fA-F]{3,8}/g);
      if (bgColors) bgColors.forEach(c => tally(results.colors, c));
      tally(results.backgrounds, cs.background);
    }

    // Fonts: family, size, weight, line-height
    tally(results.fonts, cs.fontFamily);
    tally(results.fontSizes, cs.fontSize);
    tally(results.fontWeights, cs.fontWeight);
    tally(results.lineHeight, cs.lineHeight);
    tally(results.letterSpacing, cs.letterSpacing);

    // Spacing: padding, margin, gap
    tally(results.paddings, cs.padding);
    tally(results.margins, cs.margin);
    tally(results.gaps, cs.gap);

    // Radius and shadows
    tally(results.borderRadius, cs.borderRadius);
    tally(results.boxShadows, cs.boxShadow);

    // Transitions
    tally(results.transitions, cs.transition);
  });

  function mapToSorted(m, limit = 40) {
    return Array.from(m.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, limit)
      .map(([value, count]) => ({ value, count }));
  }

  return {
    colors: mapToSorted(results.colors, 40),
    fonts: mapToSorted(results.fonts, 15),
    fontSizes: mapToSorted(results.fontSizes, 25),
    fontWeights: mapToSorted(results.fontWeights, 10),
    lineHeight: mapToSorted(results.lineHeight, 10),
    letterSpacing: mapToSorted(results.letterSpacing, 10),
    borderRadius: mapToSorted(results.borderRadius, 15),
    boxShadows: mapToSorted(results.boxShadows, 10),
    backgrounds: mapToSorted(results.backgrounds, 10),
    paddings: mapToSorted(results.paddings, 15),
    margins: mapToSorted(results.margins, 10),
    gaps: mapToSorted(results.gaps, 10),
    transitions: mapToSorted(results.transitions, 10),
    totalElementsScanned: allElements.length,
  };
})();
```

**Also extract CSS custom properties from `:root`:**

```javascript
(function() {
  const rootStyles = window.getComputedStyle(document.documentElement);
  const vars = {};
  const seen = new Set();

  for (let i = 0; i < rootStyles.length; i++) {
    const prop = rootStyles[i];
    if (prop.startsWith('--') && !seen.has(prop)) {
      seen.add(prop);
      const value = rootStyles.getPropertyValue(prop).trim();
      if (value) vars[prop] = value;
    }
  }

  return { customPropertyCount: Object.keys(vars).length, properties: vars };
})();
```

**Quality gate: ≥3 background colors, ≥2 font families, ≥5 spacing values.** If fewer, see Fallback Chain.

### 1.2 CSS File Extraction

Download external CSS files, parse `@font-face`, `:root` variables, and media queries. This supplements browser extraction with source CSS declarations.

**Steps:**

1. Download the HTML source via `curl`:

```powershell
curl -s -L '<target-url>' `
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' `
  -o 'target_page.html'
```

2. Extract `<link>` CSS URLs:

```powershell
$html = Get-Content -Path 'target_page.html' -Raw
[regex]::Matches($html, 'href="([^"]*\.css[^"]*)"') |
  ForEach-Object { $_.Groups[1].Value } | Sort-Object -Unique
```

3. Download each CSS file:

```powershell
$cssUrls = @('https://...', 'https://...')
$idx = 0
foreach ($url in $cssUrls) {
  $idx++
  curl -s -L $url -H 'User-Agent: Mozilla/5.0' -o "css_$idx.css"
}
```

4. Concatenate all CSS files and extract:
   - `@font-face` declarations (font-family name, src URL, weight, style)
   - `:root` CSS custom property definitions
   - `@media` query breakpoints
   - Raw gradient strings

**Fallback if browser fails:** Use `curl` to download HTML, extract `<link>` CSS URLs, download and parse CSS files with regex. This yields lower quality (no computed values, no DOM structure) but is still usable.

### 1.3 Token Generation

Create `colors_and_type.css` following the Token Contract. The file MUST define all tokens as CSS custom properties on `:root`.

**Required sections (in order):**

1. `@font-face` declarations (if custom fonts found) — NEVER use `@import` for custom fonts
2. `@group-priority` comment listing token group hierarchy
3. `:root` block — all token definitions (default theme)
4. `.dark` block — if the brand is dark-dominant (repeat semantic + portable aliases only)
5. `.light` block — if a light override is needed (optional)

**Color scale rules:**
- Full 10-step scale (50, 100, 200, 300, 400, 500, 600, 700, 800, 900) per color group
- Pattern: `--{prefix}-{role}-{step}` (e.g., `--brand-primary-600`)
- One step per group marked `/* @primary */` — the anchor, using the real extracted brand color
- Anchor has `/* Source: {url} */` comment
- All other steps marked `/* AI-generated */`

**Semantic aliases (pattern: `--{role}`):**

```css
--primary: var(--brand-primary-600);
--accent: var(--brand-accent-500);
--foreground: var(--brand-neutral-50);
--background: var(--brand-neutral-950);
```

**Portable aliases (REQUIRED — pattern: `--color-{role}`):**

These are the variable names that `site-template.html` consumes. Every token file MUST define all of these:

| Category | Portable Variable Names |
|----------|------------------------|
| Colors | `--color-background`, `--color-foreground`, `--color-muted-foreground`, `--color-surface`, `--color-card`, `--color-border`, `--color-muted`, `--color-primary`, `--color-on-primary`, `--color-primary-hover`, `--color-secondary`, `--color-accent`, `--color-on-accent` |
| Fonts | `--font-display`, `--font-heading`, `--font-body`, `--font-mono` |
| Font sizes | `--font-size-display`, `--font-size-h1`, `--font-size-h2`, `--font-size-h3`, `--font-size-h4`, `--font-size-body`, `--font-size-lead`, `--font-size-caption`, `--font-size-nav`, `--font-size-button` |
| Font weights | `--font-weight-display`, `--font-weight-h1`, `--font-weight-h2`, `--font-weight-h3`, `--font-weight-h4`, `--font-weight-body`, `--font-weight-lead`, `--font-weight-nav`, `--font-weight-button` |
| Spacing | `--space-1` (0.25rem/4px), `--space-2` (0.5rem/8px), `--space-3` (0.75rem/12px), `--space-4` (1rem/16px), `--space-5` (1.5rem/24px), `--space-6` (2rem/32px), `--space-7` (3rem/48px), `--space-8` (4rem/64px) |
| Radius | `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-full` |
| Shadows | `--shadow-1`, `--shadow-2`, `--shadow-3`, `--shadow-4`, `--shadow-5` |
| Sizing | `--max-content`, `--size-nav`, `--size-button-sm`, `--size-button-md`, `--size-button-lg`, `--size-icon-sm`, `--size-icon-md`, `--size-icon-lg` |

**Example token file skeleton:**

```css
/* @group-priority
   1. Color scales (brand primary, secondary, accent, neutral, semantic)
   2. Semantic color aliases
   3. Portable color aliases (--color-*)
   4. Typography (font-family, font-size, font-weight, line-height)
   5. Spacing
   6. Sizing
   7. Radius
   8. Shadows
   9. Transitions & easing
   10. Letter spacing
*/

:root {
  /* --- Color Scales --- */
  --brand-primary-50:  #fffdf0;  /* AI-generated */
  /* ... full 10-step scale ... */
  --brand-primary-600: #fffa00;  /* @primary */ /* Source: https://example.com */

  /* --- Semantic Aliases --- */
  --primary: var(--brand-primary-600);
  --foreground: var(--brand-neutral-50);
  --background: var(--brand-neutral-950);

  /* --- Portable Aliases (REQUIRED) --- */
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-on-primary: #ffffff;
  --color-primary-hover: var(--brand-primary-500);
  --color-surface: var(--brand-neutral-900);
  --color-card: var(--brand-neutral-900);
  --color-border: var(--brand-neutral-800);
  --color-muted: var(--brand-neutral-800);
  --color-muted-foreground: var(--brand-neutral-400);
  --color-secondary: var(--brand-secondary-600);
  --color-accent: var(--brand-accent-500);
  --color-on-accent: #ffffff;

  /* --- Typography --- */
  --font-display: 'BrandTitle', sans-serif;
  --font-heading: 'BrandTitle', sans-serif;
  --font-body: 'DIN Alternate', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  --font-size-display: 3.5rem;
  --font-size-h1: 2.5rem;
  --font-size-h2: 2rem;
  --font-size-h3: 1.5rem;
  --font-size-h4: 1.25rem;
  --font-size-body: 1rem;
  --font-size-lead: 1.125rem;
  --font-size-caption: 0.75rem;
  --font-size-nav: 0.875rem;
  --font-size-button: 0.875rem;

  --font-weight-display: 800;
  --font-weight-h1: 700;
  --font-weight-h2: 700;
  --font-weight-h3: 600;
  --font-weight-h4: 600;
  --font-weight-body: 400;
  --font-weight-lead: 400;
  --font-weight-nav: 500;
  --font-weight-button: 600;

  /* --- Spacing --- */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.5rem;
  --space-6: 2rem;
  --space-7: 3rem;
  --space-8: 4rem;

  /* --- Radius --- */
  --radius-sm: 2px;
  --radius-md: 4px;
  --radius-lg: 8px;
  --radius-full: 9999px;

  /* --- Shadows --- */
  --shadow-1: 0 1px 2px rgba(0,0,0,0.3);      /* drop */
  --shadow-2: 0 4px 12px rgba(0,0,0,0.5);     /* drop */
  --shadow-3: 0 0 8px rgba(255,250,0,0.3);    /* glow */
  --shadow-4: 0 0 16px rgba(255,250,0,0.5);   /* glow */
  --shadow-5: 0 0 10px #fff000;              /* glow, brand-colored */

  /* --- Sizing --- */
  --max-content: 1200px;
  --size-nav: 72px;
  --size-button-sm: 32px;
  --size-button-md: 40px;
  --size-button-lg: 48px;
  --size-icon-sm: 16px;
  --size-icon-md: 24px;
  --size-icon-lg: 32px;

  /* --- Transitions --- */
  --transition-fast: 0.15s ease;
  --transition-base: 0.3s ease;
  --transition-smooth: 0.4s cubic-bezier(0.4, 0, 0.2, 1);

  /* --- Letter Spacing --- */
  --tracking-tight: -0.01em;
  --tracking-tighter: -0.02em;
  --tracking-tightest: -0.04em;
}
```

**Quality gate: all portable variables from the contract are defined.** Run the checklist:
- [ ] All color groups have complete 10-step scales (50-900)
- [ ] Each color group has exactly one step marked `/* @primary */`
- [ ] All portable aliases (`--color-*`) are defined
- [ ] Font families use real extracted `@font-face` names (no generic primary)
- [ ] Spacing tokens (`--space-1` through `--space-8`) are present
- [ ] Shadow tokens (`--shadow-1` through `--shadow-5`) are present
- [ ] No `@import` is used for custom fonts

### 1.4 Token Verification

Verify that the token CSS file is valid and all variables resolve when consumed.

**Steps:**

1. Create a minimal test HTML that links to the token CSS:

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="colors_and_type.css">
</head>
<body>
  <div id="test" style="background: var(--color-background); color: var(--color-foreground); font-family: var(--font-body); padding: var(--space-4); border-radius: var(--radius-md);">
    Token test — if this has a background color, tokens resolved.
  </div>
</body>
</html>
```

2. Open the test HTML in a browser
3. Verify: the `#test` div has a visible background color (tokens resolved)
4. Check the browser console for any CSS warnings about undefined variables
5. **Swap test:** change the `<link>` to a different brand's `colors_and_type.css` — verify the test div restyles correctly

**Fix:** Add any missing portable variable definitions. Fix typos in variable names. Ensure every `var()` reference in `.dark` or `.light` blocks resolves to a definition in `:root`.

---

## Phase 2: Layout Extraction

### 2.1 Browser Extraction

Capture full page structure: section list, content text, layout patterns.

**Steps:**

1. Navigate to the target URL (reuse the browser session from Phase 1.1)
2. Ensure the page is fully rendered and scrolled
3. Run the section structure extraction script below
4. Run the content inventory extraction script below
5. Run the layout system extraction script below

**Page Section Extraction JavaScript:**

```javascript
(function() {
  const sectionSelectors = [
    'section', 'header', 'footer', 'main',
    '[class*="section"]', '[class*="Section"]',
    '[class*="hero"]', '[class*="Hero"]',
    '[class*="banner"]', '[class*="Banner"]',
    '[class*="content"]', '[class*="Content"]',
    '[role="banner"]', '[role="contentinfo"]',
  ];

  const sections = [];
  const seen = new Set();

  sectionSelectors.forEach(sel => {
    document.querySelectorAll(sel).forEach(el => {
      const rect = el.getBoundingClientRect();
      if (rect.width === 0 || rect.height < 50) return;

      const key = el.tagName + '|' + rect.top;
      if (seen.has(key)) return;
      seen.add(key);

      const cs = window.getComputedStyle(el);
      sections.push({
        tag: el.tagName.toLowerCase(),
        classes: (el.className || '').toString().split(' ').filter(c => c).slice(0, 3),
        height: Math.round(rect.height),
        top: Math.round(rect.top),
        background: cs.backgroundColor !== 'rgba(0, 0, 0, 0)' ? cs.backgroundColor : undefined,
        padding: cs.padding !== '0px' ? cs.padding : undefined,
        maxWidth: cs.maxWidth !== 'none' ? cs.maxWidth : undefined,
        childCount: el.children.length,
        textPreview: (el.textContent || '').trim().substring(0, 200).replace(/\s+/g, ' '),
      });
    });
  });

  sections.sort((a, b) => a.top - b.top);

  return {
    totalSections: sections.length,
    sections: sections.slice(0, 20),
  };
})();
```

**Content Inventory Extraction JavaScript:**

```javascript
(function() {
  // Headings
  const headings = [];
  document.querySelectorAll('h1, h2, h3, h4, h5, h6').forEach(el => {
    const text = (el.textContent || '').trim();
    if (text) headings.push({ tag: el.tagName.toLowerCase(), text: text.substring(0, 200) });
  });

  // Body copy (paragraphs)
  const paragraphs = [];
  document.querySelectorAll('p').forEach(el => {
    const text = (el.textContent || '').trim();
    if (text.length > 20) paragraphs.push(text.substring(0, 300));
  });

  // Button labels
  const buttons = [];
  document.querySelectorAll('button, [role="button"], .btn, .button, .cta, a[class*="btn"], a[class*="button"]').forEach(el => {
    const text = (el.textContent || '').trim();
    if (text) buttons.push(text.substring(0, 100));
  });

  // Navigation items
  const navItems = [];
  document.querySelectorAll('nav a, nav li, header a, [role="navigation"] a').forEach(el => {
    const text = (el.textContent || '').trim();
    if (text && text.length < 50) navItems.push(text);
  });

  // Footer links
  const footerLinks = [];
  document.querySelectorAll('footer a, [role="contentinfo"] a').forEach(el => {
    const text = (el.textContent || '').trim();
    if (text) footerLinks.push(text.substring(0, 100));
  });

  // Image alt text
  const images = [];
  document.querySelectorAll('img').forEach(img => {
    images.push({ src: img.src, alt: img.alt, width: img.naturalWidth, height: img.naturalHeight });
  });

  return {
    headings: headings,
    paragraphs: paragraphs.slice(0, 30),
    buttons: [...new Set(buttons)].slice(0, 20),
    navItems: [...new Set(navItems)].slice(0, 20),
    footerLinks: [...new Set(footerLinks)].slice(0, 20),
    images: images.slice(0, 30),
  };
})();
```

**Layout System Extraction JavaScript:**

```javascript
(function() {
  const layouts = [];

  document.querySelectorAll('*').forEach(el => {
    const cs = window.getComputedStyle(el);
    const display = cs.display;

    if (display === 'grid' || display === 'flex' || display === 'inline-flex' || display === 'inline-grid') {
      const rect = el.getBoundingClientRect();
      if (rect.width === 0 || rect.height === 0) return;

      layouts.push({
        tag: el.tagName.toLowerCase(),
        classes: (el.className || '').toString().split(' ').filter(c => c).slice(0, 3),
        display: display,
        gridTemplateColumns: cs.gridTemplateColumns !== 'none' ? cs.gridTemplateColumns : undefined,
        gap: cs.gap !== 'normal' ? cs.gap : undefined,
        flexDirection: cs.flexDirection,
        flexWrap: cs.flexWrap !== 'nowrap' ? cs.flexWrap : undefined,
        alignItems: cs.alignItems !== 'normal' ? cs.alignItems : undefined,
        justifyContent: cs.justifyContent !== 'normal' ? cs.justifyContent : undefined,
        childCount: el.children.length,
        width: Math.round(rect.width),
        height: Math.round(rect.height),
      });
    }
  });

  return layouts.slice(0, 30);
})();
```

**Quality gate: ≥4 sections identified, real content extracted (headings, buttons, nav items are not empty).**

### 2.2 Content Inventory

Extract all text content from the page. This content will be used directly in the site template — NEVER use placeholder text.

**Steps:**

1. From the Phase 2.1 extraction output, compile:
   - All headings in order (h1 through h6) — these define the page's narrative structure
   - All paragraph text — body copy for each section
   - All button/CTA labels — exact text strings
   - All navigation items — the site's menu structure
   - All footer links — organized by column if applicable
2. Organize content by section: match headings and body copy to their parent section from the section extraction
3. Note the page's narrative flow: what story does the page tell from top to bottom?
4. Record any unique layout patterns: split sections, card grids, timelines, feature lists

**Critical:** Use the REAL content from the source website. Do not invent, paraphrase, or use lorem ipsum. If content is in Chinese, keep it in Chinese. If in English, keep it in English.

### 2.3 Site Template Generation

Create `site-template.html` following the Template Contract.

**CRITICAL RULES:**

1. **Single HTML file** — all sections in one file, not fragments. The file must be a complete, standalone web page.
2. **CSS link:** `<link rel="stylesheet" href="colors_and_type.css">` (relative path, same directory). No other CSS files. No CDN links. No inline `<style>` for visual values (layout scaffolding only is acceptable).
3. **ALL visual values use `var(--name, fallback)` — ZERO hardcoded colors/fonts.** Every color, font family, font size, spacing, radius, and shadow must reference a CSS variable:
   ```css
   color: var(--color-foreground, #000000);
   background: var(--color-background, #ffffff);
   font-family: var(--font-body, sans-serif);
   padding: var(--space-6, 32px) var(--space-5, 24px);
   border-radius: var(--radius-md, 4px);
   box-shadow: var(--shadow-2, 0 4px 12px rgba(0,0,0,0.1));
   ```
4. **Use ONLY portable variable names** (listed in the Token Contract, Phase 1.3). These are the names that exist in ALL token files:
   - Colors: `--color-background`, `--color-foreground`, `--color-muted-foreground`, `--color-surface`, `--color-card`, `--color-border`, `--color-primary`, `--color-on-primary`, `--color-primary-hover`, `--color-accent`, `--color-on-accent`, `--color-secondary`, `--color-muted`
   - Fonts: `--font-display`, `--font-heading`, `--font-body`, `--font-mono`
   - Sizes: `--font-size-display` / `h1` / `h2` / `h3` / `h4` / `body` / `lead` / `caption` / `nav` / `button`
   - Weights: `--font-weight-display` / `h1` / `h2` / `h3` / `h4` / `body` / `lead` / `nav` / `button`
   - Spacing: `--space-1` through `--space-8`
   - Radius: `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-full`
   - Shadows: `--shadow-1` through `--shadow-5`
   - Sizing: `--max-content`, `--size-nav`, `--size-button-sm` / `md` / `lg`, `--size-icon-sm` / `md` / `lg`
5. **Always include fallback values** in `var()` — the template must render even without a token CSS file linked.
6. **Image areas use CSS gradients with variables** (no external image URLs):
   ```css
   .hero-image {
     background: linear-gradient(135deg, var(--color-muted, #f5f5f5), var(--color-border, #e5e5e5));
   }
   ```
7. **Responsive breakpoints at 1024px and 768px** — include media queries:
   ```css
   @media (max-width: 1024px) { /* tablet adjustments */ }
   @media (max-width: 768px) { /* mobile adjustments */ }
   ```
8. **Real content from source website** — use the actual headings, body copy, button labels, and navigation items extracted in Phase 2.2. Do NOT use placeholder text.

**Template skeleton:**

```html
<!DOCTYPE html>
<html lang="{source-language}">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{BrandName}</title>
  <link rel="stylesheet" href="colors_and_type.css">
  <style>
    /* Layout scaffolding only — NO visual values here */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    .container { max-width: var(--max-content, 1200px); margin: 0 auto; padding: 0 var(--space-4, 16px); }

    /* Navigation */
    .nav {
      position: sticky; top: 0; z-index: 100;
      background: var(--color-surface, #ffffff);
      border-bottom: 1px solid var(--color-border, #e5e5e5);
      height: var(--size-nav, 72px);
      display: flex; align-items: center;
    }
    .nav-brand { font-family: var(--font-display, sans-serif); font-size: var(--font-size-h4, 1.25rem); font-weight: var(--font-weight-h4, 600); color: var(--color-foreground, #000); }
    .nav-items { display: flex; gap: var(--space-4, 16px); margin-left: auto; }
    .nav-items a { color: var(--color-muted-foreground, #666); text-decoration: none; font-size: var(--font-size-nav, 0.875rem); font-weight: var(--font-weight-nav, 500); }

    /* Hero */
    .hero { padding: var(--space-8, 64px) 0; background: var(--color-background, #ffffff); }
    .hero h1 { font-family: var(--font-display, sans-serif); font-size: var(--font-size-display, 3.5rem); font-weight: var(--font-weight-display, 800); color: var(--color-foreground, #000); }
    .hero p { font-size: var(--font-size-lead, 1.125rem); color: var(--color-muted-foreground, #666); margin-top: var(--space-3, 12px); }
    .hero-cta {
      display: inline-block; margin-top: var(--space-5, 24px);
      padding: var(--space-3, 12px) var(--space-6, 32px);
      background: var(--color-primary, #0066ff); color: var(--color-on-primary, #fff);
      font-size: var(--font-size-button, 0.875rem); font-weight: var(--font-weight-button, 600);
      border-radius: var(--radius-md, 4px); text-decoration: none;
      transition: var(--transition-fast, 0.15s ease);
    }
    .hero-cta:hover { background: var(--color-primary-hover, #0052cc); }

    /* Sections */
    .section { padding: var(--space-7, 48px) 0; }
    .section h2 { font-family: var(--font-heading, sans-serif); font-size: var(--font-size-h2, 2rem); font-weight: var(--font-weight-h2, 700); color: var(--color-foreground, #000); }
    .section p { font-family: var(--font-body, sans-serif); font-size: var(--font-size-body, 1rem); color: var(--color-muted-foreground, #666); line-height: 1.6; }

    /* Cards */
    .card-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: var(--space-4, 16px); }
    .card { background: var(--color-card, #f9f9f9); border: 1px solid var(--color-border, #e5e5e5); border-radius: var(--radius-lg, 8px); padding: var(--space-5, 24px); }
    .card h3 { font-size: var(--font-size-h3, 1.5rem); font-weight: var(--font-weight-h3, 600); color: var(--color-foreground, #000); }

    /* Footer */
    .footer { background: var(--color-surface, #f5f5f5); border-top: 1px solid var(--color-border, #e5e5e5); padding: var(--space-6, 32px) 0; }
    .footer a { color: var(--color-muted-foreground, #666); text-decoration: none; font-size: var(--font-size-caption, 0.75rem); }

    @media (max-width: 1024px) { /* tablet adjustments */ }
    @media (max-width: 768px) {
      .nav-items { display: none; } /* mobile menu */
      .hero h1 { font-size: var(--font-size-h1, 2.5rem); }
    }
  </style>
</head>
<body>
  <!-- All sections with REAL content from source site -->
  <nav class="nav"> ... </nav>
  <section class="hero"> ... </section>
  <section class="section"> ... </section>
  <footer class="footer"> ... </footer>
</body>
</html>
```

### 2.4 Layout Verification

Verify the site template renders correctly and works with any token set.

**Steps:**

1. Open `site-template.html` in a browser (linked to its own `colors_and_type.css`)
2. Take a screenshot — verify the page renders without errors
3. Verify: all sections are visible and properly styled
4. Verify: content is readable (no contrast issues)
5. Verify: responsive breakpoints work (resize to 1024px and 768px)
6. **Token swap test:** change the `<link>` to a DIFFERENT brand's `colors_and_type.css`:
   ```html
   <!-- Original -->
   <link rel="stylesheet" href="colors_and_type.css">
   <!-- Swapped -->
   <link rel="stylesheet" href="../OtherBrand/colors_and_type.css">
   ```
7. Open the swapped version — verify the layout is intact but the visual style changed completely
8. **Fix:** If any hardcoded values are found, replace them with `var(--name, fallback)`. If any variables are missing fallbacks, add them.

**Quality gate:**
- [ ] Page renders without console errors
- [ ] All sections visible and properly styled
- [ ] Content is readable
- [ ] Responsive breakpoints exist (1024px and 768px)
- [ ] Zero hardcoded color values (grep for `#[0-9a-f]` in style — only in fallbacks allowed)
- [ ] Zero hardcoded font names (only in fallbacks)
- [ ] CSS link is `href="colors_and_type.css"` (relative)
- [ ] Token swap test passed (different visual style after swap, layout intact)

---

## Phase 3: Assembly & Combination

### 3.1 Create Combinations

Create cross-brand combinations: Brand A's layout + Brand B's tokens.

**Steps:**

1. Create a combinations directory:
   ```powershell
   New-Item -ItemType Directory -Force -Path "combinations/{A}-layout_{B}-tokens"
   ```
2. Copy the site template and update the CSS link path:
   ```powershell
   $content = Get-Content "{A}/site-template.html" -Raw
   $content = $content -replace 'href="colors_and_type\.css"', 'href="../../{B}/colors_and_type.css"'
   # If Brand B's tokens are dark-themed, add class="dark" to <body>
   if ($isDarkTheme) {
     $content = $content -replace '<body>', '<body class="dark">'
   }
   Set-Content -Path "combinations/{A}-layout_{B}-tokens/index.html" -Value $content -Encoding UTF8
   ```
3. Repeat for each desired combination (e.g., A-layout + B-tokens, B-layout + A-tokens)

**Bash equivalent:**
```bash
mkdir -p "combinations/{A}-layout_{B}-tokens"
sed 's|href="colors_and_type.css"|href="../../{B}/colors_and_type.css"|' \
  "{A}/site-template.html" > "combinations/{A}-layout_{B}-tokens/index.html"
# For dark-themed tokens:
sed -i 's|<body>|<body class="dark">|' "combinations/{A}-layout_{B}-tokens/index.html"
```

### 3.2 Verify Combinations

Open each combination in a browser and verify rendering.

**Steps:**

1. Open `combinations/{A}-layout_{B}-tokens/index.html` in a browser
2. Take a screenshot
3. Verify: Brand A's layout structure is intact
4. Verify: Brand B's visual style is applied (colors, fonts, spacing, shadows)
5. Verify: no console errors about undefined CSS variables
6. If dark theme: verify dark mode is applied correctly
7. Repeat for each combination

### 3.3 Git Deploy

Stage and commit all assets.

**Steps:**

```powershell
git status --short --branch
git add "{BrandName}/colors_and_type.css" "{BrandName}/site-template.html"
git add "combinations/"  # if combinations were created
git commit -m "feat: add {BrandName} reverse-engineered assets — tokens + site template from {source-domain}"
git push origin main  # only if remote push is explicitly requested by user
```

**Commit message format:** `feat: add {BrandName} reverse-engineered assets — tokens + site template from {source-domain}`

> Remote push is opt-in. Only push when the user explicitly requests it. Never assume `origin` or `main` belong to the user.

---

## Quality Gates Summary

| Gate | Phase | Criteria | Pass | Fail Action |
|------|-------|----------|------|-------------|
| 1.1 | Token Extraction | ≥3 background colors, ≥2 font families, ≥5 spacing values extracted | Proceed to 1.2 | See Fallback Chain |
| 1.2 | CSS File Extraction | CSS files downloaded and parsed (or documented as skipped for SPA) | Proceed to 1.3 | Proceed with browser data only |
| 1.3 | Token Generation | All portable variables defined, 10-step color scales present, no `@import` for fonts | Proceed to 1.4 | Fix missing variables, re-check |
| 1.4 | Token Verification | Token test HTML renders with visible colors, swap test passes | Proceed to Phase 2 | Add missing definitions, fix typos |
| 2.1 | Layout Extraction | ≥4 sections identified, real content extracted | Proceed to 2.2 | Check for SPA, use alternative selectors |
| 2.2 | Content Inventory | Headings, buttons, nav items are non-empty with real text | Proceed to 2.3 | Re-extract content, check page render |
| 2.3 | Template Generation | Single HTML file, zero hardcoded colors, CSS link is relative, responsive breakpoints exist | Proceed to 2.4 | Replace hardcoded values with `var()`, add fallbacks |
| 2.4 | Layout Verification | Renders without errors, token swap test passes, layout intact after swap | Proceed to Phase 3 | Fix undefined variables, add fallbacks |
| 3.1 | Combinations | Combination files created with correct CSS link paths | Proceed to 3.2 | Fix path replacement regex |
| 3.2 | Combination Verification | Each combination renders with correct layout + different visual style | Proceed to 3.3 | Check for missing portable variables in target token file |
| 3.3 | Git Deploy | Files committed successfully | Done | Check git status, fix file paths |

---

## Fallback Chain

If the primary extraction method fails, follow this chain in order:

### Fallback 1: Browser fails → curl HTML → parse CSS

If `browser_use` or `browser_navigate` / `browser_evaluate` is unavailable or crashes:

1. Download the HTML source via `curl`:
   ```powershell
   curl -s -L '<target-url>' -H 'User-Agent: Mozilla/5.0' -o 'target_page.html'
   ```
2. Extract `<link>` CSS URLs from the HTML
3. Download each CSS file via `curl`
4. Parse CSS files with regex to extract: colors, fonts, font sizes, border-radius, box-shadows, transitions, `@font-face`, `:root` variables, media queries
5. Build the token file from CSS declarations only (no computed values, no DOM structure)
6. For layout: parse the HTML structure manually to identify sections, headings, buttons
7. Set quality to lower confidence — note in the output that browser extraction was unavailable

### Fallback 2: CSS file not accessible → browser computed styles only

If CSS files cannot be fetched (403, 404, or SPA with no `<link>` tags):

1. Rely entirely on browser computed styles from Phase 1.1
2. `@font-face` src URLs will be missing — use `document.fonts` API to identify font families
3. Raw `:root` variable definitions will be missing — use runtime `getComputedStyle(document.documentElement)` to extract them
4. Media query breakpoints will be missing — test responsive by resizing the browser viewport
5. Note the gaps in the token file's comments

### Fallback 3: Page requires auth → ask user for saved HTML

If the target URL requires authentication or is behind a login wall:

1. Stop the automated extraction
2. Ask the user to:
   - Save the fully rendered page as HTML (Ctrl+S in browser, choose "Webpage, Complete")
   - Or take full-page screenshots and provide them
   - Or provide the page's CSS files manually
3. Parse the saved HTML/CSS files with the same regex patterns from Phase 1.2
4. For layout, use the saved HTML structure directly
5. Note in the output that extraction was from saved files, not live browser

### Fallback 4: SPA with no SSR → browser is the only option

If the site is a client-rendered SPA (Next.js, Nuxt, Vue) with a bare `<div id="root">` shell:

1. `curl` will return an empty shell — skip CSS file extraction entirely
2. Browser extraction (Phase 1.1) is the only option — the browser renders the page, JS injects styles
3. `getComputedStyle()` captures the real values after JS rendering
4. `document.fonts` API reports loaded font families
5. `@font-face` src URLs may be missing — note as `unknowns`
