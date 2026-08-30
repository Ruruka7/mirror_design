# Browser-Based Extraction Specification

> This is the PRIMARY extraction method. CSS file regex extraction (`css-extraction.md`) is a SUPPLEMENT only.

## Purpose

Define how to use browser automation to render a target website, extract real computed styles, analyze DOM structure, capture screenshots, and identify component boundaries — the foundation of true reverse engineering.

## Why Browser, Not curl

| curl + regex | Browser extraction |
|---|---|
| Gets CSS declarations | Gets actual computed values (after cascade + JS) |
| Misses JS-rendered styles | Captures runtime-computed styles |
| No visual context | Full-page screenshots for analysis |
| No DOM structure | Full DOM tree walk with component mapping |
| No layout analysis | Real grid/flex templates, element dimensions |
| No asset capture | All images, SVGs, fonts, icons extracted |
| Static CSS only | Responsive breakpoint testing |
| Guesses component boundaries | Identifies real component elements in DOM |

## Phase 0A: Browser Extraction Steps

### Step 0A.1 — Navigate & Render

Use `browser_use` subagent or `browser_navigate` + `browser_evaluate`:

1. Navigate to target URL
2. Wait for `networkidle` (all resources loaded, no pending requests)
3. Wait for `domcontentloaded`
4. Scroll through entire page to trigger lazy-loaded content
5. Wait 2 seconds for final render settle

```
browser_navigate → target URL
browser_wait_for → "networkidle"
browser_evaluate → scroll-to-bottom script
browser_wait_for → 2000ms
```

### Step 0A.2 — Capture Screenshots

1. Full-page screenshot (above-the-fold + below-the-fold stitched)
2. Viewport-only screenshot (what user sees on load)
3. Per-section screenshots (optional, for complex layouts)

```
browser_take_screenshot → { fullPage: true, path: "{tmp_dir}/{BrandName}/screenshot-full.png" }
browser_take_screenshot → { fullPage: false, path: "{tmp_dir}/{BrandName}/screenshot-viewport.png" }
```

Screenshots serve as:
- Visual reference for design decisions in Phase 1
- Validation comparison for UIKit in Phase 4
- Evidence for component variant selection in Phase 3

### Step 0A.3 — Extract Computed Styles (Core)

Run this JavaScript via `browser_evaluate` on the rendered page:

```javascript
// === COMPUTED STYLE EXTRACTION ===
(function() {
  const results = {
    colors: new Map(),      // value → count
    fonts: new Map(),       // family → count
    fontSizes: new Map(),   // size → count
    fontWeights: new Map(), // weight → count
    lineHeight: new Map(),  // value → count
    letterSpacing: new Map(),
    borderRadius: new Map(),
    borderWidths: new Map(),
    boxShadows: new Map(),
    clipPaths: new Map(),
    backdropFilters: new Map(),
    transitions: new Map(),
    backgrounds: new Map(),  // gradients, images
    paddings: new Map(),
    margins: new Map(),
    gaps: new Map(),
    displays: new Map(),
    positions: new Map(),
    zIndices: new Map(),
    opacities: new Map(),
  };

  const allElements = document.querySelectorAll('*');

  allElements.forEach(el => {
    const cs = window.getComputedStyle(el);

    function tally(map, value) {
      if (!value || value === 'initial' || value === 'inherit' || value === 'none' || value === 'normal') return;
      // Skip transparent/initial
      if (value === 'rgba(0, 0, 0, 0)' || value === 'transparent') return;
      map.set(value, (map.get(value) || 0) + 1);
    }

    tally(results.colors, cs.color);
    tally(results.colors, cs.backgroundColor);
    tally(results.colors, cs.borderColor);
    tally(results.colors, cs.borderTopColor);
    tally(results.colors, cs.borderBottomColor);
    tally(results.colors, cs.outlineColor);
    // Extract colors from box-shadow (first color)
    if (cs.boxShadow && cs.boxShadow !== 'none') {
      const shadowColors = cs.boxShadow.match(/rgba?\([^)]+\)|#[0-9a-fA-F]{3,8}/g);
      if (shadowColors) shadowColors.forEach(c => tally(results.colors, c));
    }
    // Extract colors from background (gradients)
    if (cs.background && cs.background !== 'none' && cs.background !== 'rgba(0, 0, 0, 0)') {
      const bgColors = cs.background.match(/rgba?\([^)]+\)|#[0-9a-fA-F]{3,8}/g);
      if (bgColors) bgColors.forEach(c => tally(results.colors, c));
      tally(results.backgrounds, cs.background);
    }

    tally(results.fonts, cs.fontFamily);
    tally(results.fontSizes, cs.fontSize);
    tally(results.fontWeights, cs.fontWeight);
    tally(results.lineHeight, cs.lineHeight);
    tally(results.letterSpacing, cs.letterSpacing);
    tally(results.borderRadius, cs.borderRadius);
    tally(results.borderWidths, cs.borderTopWidth);
    tally(results.boxShadows, cs.boxShadow);
    tally(results.clipPaths, cs.clipPath);
    tally(results.backdropFilters, cs.backdropFilter || cs.webkitBackdropFilter);
    tally(results.transitions, cs.transition);
    tally(results.paddings, cs.padding);
    tally(results.margins, cs.margin);
    tally(results.gaps, cs.gap);
    tally(results.displays, cs.display);
    tally(results.positions, cs.position);
    tally(results.zIndices, cs.zIndex);
    tally(results.opacities, cs.opacity);
  });

  // Convert Maps to sorted arrays
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
    borderWidths: mapToSorted(results.borderWidths, 10),
    boxShadows: mapToSorted(results.boxShadows, 10),
    clipPaths: mapToSorted(results.clipPaths, 8),
    backdropFilters: mapToSorted(results.backdropFilters, 5),
    transitions: mapToSorted(results.transitions, 10),
    backgrounds: mapToSorted(results.backgrounds, 10),
    paddings: mapToSorted(results.paddings, 15),
    margins: mapToSorted(results.margins, 10),
    gaps: mapToSorted(results.gaps, 10),
    displays: mapToSorted(results.displays, 8),
    positions: mapToSorted(results.positions, 5),
    zIndices: mapToSorted(results.zIndices, 8),
    opacities: mapToSorted(results.opacities, 5),
    totalElementsScanned: allElements.length,
  };
})();
```

### Step 0A.4 — Extract DOM Structure & Component Boundaries

```javascript
// === DOM STRUCTURE & COMPONENT IDENTIFICATION ===
(function() {
  const components = [];

  // Component selectors to search for
  const componentSelectors = {
    navigation: ['nav', 'header', '[role="navigation"]', '.nav', '.navbar', '.header', '.menu'],
    button: ['button', '[role="button"]', '.btn', '.button', '.cta', '[type="submit"]'],
    card: ['.card', '[class*="card"]', '[class*="Card"]', '.panel', '.tile', '.item-card'],
    input: ['input', 'textarea', 'select', '.input', '[class*="input"]', '[class*="Input"]'],
    badge: ['.badge', '.tag', '.chip', '.label', '[class*="badge"]', '[class*="Badge"]'],
    ctaLink: ['a', '.link', '.cta-link', '[class*="link"]', '[class*="cta"]'],
    hero: ['[class*="hero"]', '[class*="Hero"]', '.banner', '[class*="banner"]'],
    footer: ['footer', '.footer', '[class*="footer"]'],
  };

  Object.entries(componentSelectors).forEach(([type, selectors]) => {
    const found = new Set();
    selectors.forEach(sel => {
      document.querySelectorAll(sel).forEach(el => {
        // Skip if already found (avoid duplicates)
        const key = el.tagName + '|' + el.className + '|' + el.getBoundingClientRect().top;
        if (found.has(key)) return;
        found.add(key);

        const cs = window.getComputedStyle(el);
        const rect = el.getBoundingClientRect();

        // Only include visible elements
        if (rect.width === 0 || rect.height === 0) return;
        if (cs.display === 'none' || cs.visibility === 'hidden') return;

        components.push({
          type: type,
          tag: el.tagName.toLowerCase(),
          classes: (el.className || '').toString().split(' ').filter(c => c).slice(0, 5),
          text: (el.textContent || '').trim().substring(0, 100),
          computed: {
            background: cs.backgroundColor,
            color: cs.color,
            fontFamily: cs.fontFamily.split(',')[0].replace(/['"]/g, '').trim(),
            fontSize: cs.fontSize,
            fontWeight: cs.fontWeight,
            lineHeight: cs.lineHeight,
            letterSpacing: cs.letterSpacing,
            borderRadius: cs.borderRadius,
            border: cs.border,
            boxShadow: cs.boxShadow,
            clipPath: cs.clipPath !== 'none' ? cs.clipPath : undefined,
            backdropFilter: (cs.backdropFilter || cs.webkitBackdropFilter) !== 'none' ? (cs.backdropFilter || cs.webkitBackdropFilter) : undefined,
            display: cs.display,
            gap: cs.gap !== 'normal' ? cs.gap : undefined,
            flexDirection: cs.flexDirection !== 'row' ? cs.flexDirection : undefined,
            alignItems: cs.alignItems !== 'normal' ? cs.alignItems : undefined,
            justifyContent: cs.justifyContent !== 'normal' ? cs.justifyContent : undefined,
            padding: cs.padding,
            transition: cs.transition !== 'all 0s ease 0s' ? cs.transition : undefined,
            textTransform: cs.textTransform !== 'none' ? cs.textTransform : undefined,
            opacity: cs.opacity !== '1' ? cs.opacity : undefined,
          },
          dimensions: {
            width: Math.round(rect.width),
            height: Math.round(rect.height),
          },
          childrenCount: el.children.length,
          childTags: Array.from(el.children).slice(0, 5).map(c => c.tagName.toLowerCase()),
        });
      });
    });
  });

  return {
    totalComponents: components.length,
    components: components.slice(0, 60),  // Limit to 60 for readability
  };
})();
```

### Step 0A.5 — Extract Layout Systems

```javascript
// === LAYOUT SYSTEM EXTRACTION ===
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
        gridTemplateRows: cs.gridTemplateRows !== 'none' ? cs.gridTemplateRows : undefined,
        gridGap: cs.gridGap !== '0px' ? cs.gridGap : undefined,
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

### Step 0A.6 — Extract CSS Custom Properties (Runtime)

```javascript
// === CSS CUSTOM PROPERTIES FROM :ROOT ===
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

  // Also check body and common container elements
  ['body', '[class*="container"]', '[class*="wrapper"]'].forEach(sel => {
    const el = document.querySelector(sel);
    if (!el) return;
    const styles = window.getComputedStyle(el);
    for (let i = 0; i < styles.length; i++) {
      const prop = styles[i];
      if (prop.startsWith('--') && !seen.has(prop)) {
        seen.add(prop);
        const value = styles.getPropertyValue(prop).trim();
        if (value) vars[prop] = value;
      }
    }
  });

  return { customPropertyCount: Object.keys(vars).length, properties: vars };
})();
```

### Step 0A.7 — Extract Assets

```javascript
// === ASSET EXTRACTION ===
(function() {
  return {
    images: Array.from(document.querySelectorAll('img')).map(img => ({
      src: img.src,
      alt: img.alt,
      naturalWidth: img.naturalWidth,
      naturalHeight: img.naturalHeight,
    })),
    svgs: Array.from(document.querySelectorAll('svg')).map(svg => ({
      viewBox: svg.getAttribute('viewBox'),
      width: svg.getAttribute('width'),
      height: svg.getAttribute('height'),
      classList: svg.className?.baseVal || svg.className || '',
      pathCount: svg.querySelectorAll('path, circle, rect, polygon, line, ellipse').length,
      innerHTML: svg.innerHTML.substring(0, 200),  // First 200 chars for pattern matching
    })),
    pictureSources: Array.from(document.querySelectorAll('source')).map(s => ({
      srcset: s.srcset,
      media: s.media,
      type: s.type,
    })),
    backgroundImages: Array.from(document.querySelectorAll('*')).filter(el => {
      const cs = window.getComputedStyle(el);
      return cs.backgroundImage && cs.backgroundImage !== 'none';
    }).slice(0, 20).map(el => ({
      tag: el.tagName.toLowerCase(),
      backgroundImage: window.getComputedStyle(el).backgroundImage.substring(0, 200),
    })),
    fontLinks: Array.from(document.querySelectorAll('link[rel="stylesheet"], link[rel="preload"], style')).map(l => ({
      rel: l.rel,
      href: l.href || undefined,
      type: l.type || undefined,
      innerHTML: l.tagName === 'STYLE' ? l.innerHTML.substring(0, 300) : undefined,
    })),
    loadedFonts: (document.fonts ? Array.from(document.fonts) : []).map(f => ({
      family: f.family,
      weight: f.weight,
      style: f.style,
      status: f.status,
    })),
    favicon: document.querySelector('link[rel="icon"]')?.href || document.querySelector('link[rel="shortcut icon"]')?.href,
  };
})();
```

### Step 0A.8 — Responsive Breakpoint Testing

```javascript
// === RESPONSIVE ANALYSIS ===
// Run at multiple viewport widths to capture breakpoint behavior

async function analyzeResponsive() {
  const widths = [1920, 1440, 1024, 768, 375];
  const results = [];

  for (const width of widths) {
    // Resize browser (via browser_evaluate or CDP)
    // Then capture layout at each width
    const body = document.body;
    const cs = window.getComputedStyle(body);
    const nav = document.querySelector('nav, header, [role="navigation"]');
    const navCs = nav ? window.getComputedStyle(nav) : null;

    results.push({
      viewportWidth: width,
      bodyWidth: body.offsetWidth,
      navHeight: nav ? nav.offsetHeight : null,
      navDisplay: navCs ? navCs.display : null,
      // Check if hamburger menu appears at this width
      hasMobileMenu: !!document.querySelector('[class*="hamburger"], [class*="menu-toggle"], [class*="mobile-menu"]'),
    });
  }

  return results;
}
```

### Step 0A.9 — Extract Section Structure (Page Sections)

```javascript
// === PAGE SECTION STRUCTURE ===
(function() {
  // Identify major page sections by common patterns
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
      if (rect.width === 0 || rect.height < 50) return;  // Skip tiny elements

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
      });
    });
  });

  // Sort by top position
  sections.sort((a, b) => a.top - b.top);

  return {
    totalSections: sections.length,
    sections: sections.slice(0, 20),
  };
})();
```

## Output Format

All extraction results should be consolidated into a single JSON:

```json
{
  "source": "https://target-url.com",
  "extractionDate": "2026-08-30",
  "screenshots": ["screenshot-full.png", "screenshot-viewport.png"],
  "computedStyles": { /* Step 0A.3 output */ },
  "domComponents": { /* Step 0A.4 output */ },
  "layoutSystems": [ /* Step 0A.5 output */ ],
  "cssCustomProperties": { /* Step 0A.6 output */ },
  "assets": { /* Step 0A.7 output */ },
  "responsiveBreakpoints": [ /* Step 0A.8 output */ ],
  "pageSections": { /* Step 0A.9 output */ },
  "totalElementsScanned": 1234,
  "keyFindings": {
    "dominantBackground": "#hexvalue",
    "primaryText": "#hexvalue",
    "brandAccent": "#hexvalue",
    "secondaryAccent": "#hexvalue or null",
    "tertiaryAccent": "#hexvalue or null",
    "displayFont": "FontName",
    "bodyFont": "FontName",
    "monoFont": "FontName or null",
    "baseFontSize": "16px",
    "spacingBase": "4px or 8px",
    "primaryBorderRadius": "0px / 4px / 8px / etc",
    "shadowStyle": "glow | drop | mixed | none",
    "hasClipPath": true,
    "hasBackdropFilter": true,
    "darkTheme": true,
    "layoutSystem": "grid | flex | mixed",
    "gridPattern": "repeat(auto-fit, minmax(...)) or null",
    "maxContentWidth": "1200px or null",
    "navHeight": "72px",
    "buttonHeight": "40px",
    "inputHeight": "44px",
    "responsiveBreakpoints": ["768px", "1024px"],
    "customPropertyCount": 42
  }
}
```

## Key Findings Derivation Rules

1. **dominantBackground**: Most frequent `backgroundColor` value across all elements
2. **primaryText**: Most frequent `color` value (excluding background)
3. **brandAccent**: Color with highest saturation that appears in backgrounds or borders of interactive elements (buttons, links). NOT the dominant background.
4. **secondaryAccent**: Second distinct high-saturation color found in computed styles
5. **tertiaryAccent**: Third distinct high-saturation color (if exists)
6. **displayFont**: Font family used on largest heading elements (h1, [class*="title"], [class*="hero"])
7. **bodyFont**: Font family used on body/p elements
8. **monoFont**: Font family containing "mono", "code", "consolas", or "grotesk" in name
9. **baseFontSize**: fontSize of body element
10. **spacingBase**: GCD of all padding/gap values (usually 4px or 8px)
11. **darkTheme**: true if dominantBackground luminance < 0.3
12. **shadowStyle**: classify by pattern — "0 0" prefix = glow, Y-offset prefix = drop

## Merge with CSS File Extraction (Phase 0B)

After browser extraction, optionally run CSS file regex extraction (`css-extraction.md`) as supplement:
- Browser computed styles are AUTHORITATIVE (they reflect actual rendering)
- CSS file values are SUPPLEMENTARY (they show what was declared, which may differ from rendered)
- If conflict: browser value wins
- CSS file extraction adds: @font-face declarations, :root custom property definitions, media query breakpoints
- Browser extraction adds: actual rendered dimensions, DOM structure, component boundaries, asset URLs, layout systems

## Validation Checklist

- [ ] Full-page screenshot captured
- [ ] Computed styles extracted from all elements
- [ ] At least 3 component types identified in DOM
- [ ] Layout system (grid/flex) patterns captured
- [ ] CSS custom properties extracted from runtime
- [ ] Assets (images, SVGs, fonts) catalogued
- [ ] At least 1 responsive width tested
- [ ] Page section structure mapped
- [ ] Key findings JSON completed with all required fields
