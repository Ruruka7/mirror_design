# Page Section Extraction Specification

## Purpose

This document defines the JavaScript extraction scripts used by the `reverse-page-layout` skill to extract page structure, component placement, navigation structure, and responsive behavior from a rendered DOM. All scripts are executed via `browser_evaluate` against the target URL after the page has fully loaded and settled.

The output of these scripts feeds directly into Phase 1 (Layout Analysis) and Phase 2 (Layout Template Generation). Each script returns a JSON-serializable object that is saved to an intermediate JSON file for downstream consumption.

---

## Pre-conditions

Before running any extraction script, ensure:

1. The page has been navigated to via `browser_navigate(url)`.
2. `networkidle` (or equivalent) has been reached — all async content is loaded.
3. A full-page screenshot has been captured for visual reference.
4. The viewport is set to desktop width (1440px) for the initial extraction pass.

---

## Script 1: Page Section Structure

Extracts all major page sections (header, hero, features, CTA, footer, etc.) from the rendered DOM. For each section it captures the HTML tag, classes, geometry, computed background, padding, container/max-width, layout type, grid or flex details, child element count and tags, and the first 200 characters of text content. Results are sorted by vertical position (top to bottom).

### Extraction Script

```javascript
(() => {
  const sections = [];
  const selectors = [
    'section', 'header', 'footer', 'main',
    '[class*="section"]', '[class*="hero"]', '[class*="banner"]',
    '[class*="cta"]', '[class*="feature"]', '[class*="pricing"]',
    '[class*="testimonial"]', '[class*="content-wrapper"]',
    '[role="banner"]', '[role="contentinfo"]', '[role="main"]'
  ];
  const sectionEls = document.querySelectorAll(selectors.join(', '));

  const seen = new Set();
  sectionEls.forEach((el) => {
    if (seen.has(el)) return;
    // Skip elements nested inside another captured section
    let parent = el.parentElement;
    let isNested = false;
    while (parent) {
      if (seen.has(parent)) { isNested = true; break; }
      parent = parent.parentElement;
    }
    if (isNested && el.tagName.toLowerCase() !== 'section' && el.tagName.toLowerCase() !== 'header' && el.tagName.toLowerCase() !== 'footer') return;
    seen.add(el);

    const rect = el.getBoundingClientRect();
    const computed = window.getComputedStyle(el);
    const parentRect = el.parentElement ? el.parentElement.getBoundingClientRect() : rect;

    // Determine layout type
    const display = computed.display;
    let layoutType = 'block';
    if (display.includes('grid')) layoutType = 'grid';
    else if (display.includes('flex')) layoutType = 'flex';

    // Grid details
    let gridTemplateColumns = null;
    let gridTemplateRows = null;
    let gridColumnCount = 0;
    if (layoutType === 'grid') {
      gridTemplateColumns = computed.gridTemplateColumns;
      gridTemplateRows = computed.gridTemplateRows;
      gridColumnCount = computed.gridTemplateColumns.split(' ').filter(v => v && v !== 'none' && !v.includes('0px')).length;
    }

    // Flex details
    let flexDirection = null;
    let alignItems = null;
    let justifyContent = null;
    let flexWrap = null;
    if (layoutType === 'flex') {
      flexDirection = computed.flexDirection;
      alignItems = computed.alignItems;
      justifyContent = computed.justifyContent;
      flexWrap = computed.flexWrap;
    }

    // Resolve background color (walk up parents if transparent)
    let bg = computed.backgroundColor;
    if (bg === 'rgba(0, 0, 0, 0)' || bg === 'transparent') {
      let parent = el.parentElement;
      while (parent && (bg === 'rgba(0, 0, 0, 0)' || bg === 'transparent')) {
        const parentBg = window.getComputedStyle(parent).backgroundColor;
        if (parentBg !== 'rgba(0, 0, 0, 0)' && parentBg !== 'transparent') {
          bg = parentBg;
          break;
        }
        parent = parent.parentElement;
      }
      if (bg === 'rgba(0, 0, 0, 0)' || bg === 'transparent') bg = '#ffffff';
    }

    const children = Array.from(el.children);
    const childTags = children.map(c => c.tagName.toLowerCase());

    sections.push({
      tag: el.tagName.toLowerCase(),
      classes: el.className || '',
      id: el.id || '',
      height: Math.round(rect.height),
      topOffset: Math.round(rect.top + window.scrollY),
      bottomOffset: Math.round(rect.bottom + window.scrollY),
      backgroundColor: bg,
      padding: computed.padding,
      paddingTop: computed.paddingTop,
      paddingBottom: computed.paddingBottom,
      paddingLeft: computed.paddingLeft,
      paddingRight: computed.paddingRight,
      maxWidth: computed.maxWidth,
      containerWidth: Math.round(rect.width),
      parentWidth: Math.round(parentRect.width),
      layoutType: layoutType,
      display: display,
      gridTemplateColumns: gridTemplateColumns,
      gridTemplateRows: gridTemplateRows,
      gridColumnCount: gridColumnCount,
      flexDirection: flexDirection,
      alignItems: alignItems,
      justifyContent: justifyContent,
      flexWrap: flexWrap,
      gap: computed.gap,
      childCount: children.length,
      childTags: childTags,
      textContentPreview: el.textContent.trim().substring(0, 200),
    });
  });

  // Sort by vertical position (top to bottom)
  sections.sort((a, b) => a.topOffset - b.topOffset);

  // Page-level metadata
  const bodyComputed = window.getComputedStyle(document.body);
  const htmlComputed = window.getComputedStyle(document.documentElement);

  return {
    sections: sections,
    totalSections: sections.length,
    pageMaxWidth: bodyComputed.maxWidth !== 'none' ? bodyComputed.maxWidth : (htmlComputed.maxWidth !== 'none' ? htmlComputed.maxWidth : 'none'),
    pageBackground: bodyComputed.backgroundColor !== 'rgba(0, 0, 0, 0)' && bodyComputed.backgroundColor !== 'transparent' ? bodyComputed.backgroundColor : '#ffffff',
    viewportWidth: window.innerWidth,
    viewportHeight: window.innerHeight,
    documentHeight: Math.round(document.documentElement.scrollHeight),
  };
})();
```

---

## Script 2: Component Placement Map

For each section identified in Script 1, extracts the individual components placed within it. Each component is classified by type, its position within the section's layout, its dimensions, a text preview, and computed layout properties. A responsive-behavior hint is inferred by checking whether the element reflows at narrower widths.

### Extraction Script

```javascript
(() => {
  const selectors = [
    'section', 'header', 'footer', 'main',
    '[class*="section"]', '[class*="hero"]', '[class*="banner"]',
    '[class*="cta"]', '[class*="feature"]', '[class*="pricing"]',
    '[class*="testimonial"]', '[role="banner"]', '[role="contentinfo"]'
  ];
  const sectionEls = document.querySelectorAll(selectors.join(', '));

  const componentMap = [];
  const componentTypes = ['button', 'card', 'input', 'badge', 'cta-link', 'navigation', 'image', 'heading', 'paragraph', 'icon', 'list', 'form', 'video', 'avatar', 'tag', 'logo', 'label', 'divider', 'spacer'];

  function classifyComponent(el) {
    const tag = el.tagName.toLowerCase();
    const cls = (el.className || '').toLowerCase();
    const role = el.getAttribute('role') || '';
    const text = el.textContent.trim().substring(0, 100);

    if (tag === 'button' || role === 'button' || cls.includes('btn') || cls.includes('button')) return 'button';
    if (tag === 'a' && (cls.includes('btn') || cls.includes('cta') || cls.includes('button'))) return 'cta-link';
    if (tag === 'input' || tag === 'textarea' || tag === 'select' || tag === 'form') return 'input';
    if (tag === 'nav' || cls.includes('nav')) return 'navigation';
    if (tag === 'img' || tag === 'picture' || tag === 'svg' || cls.includes('logo')) return 'image';
    if (tag === 'video') return 'video';
    if (tag === 'h1' || tag === 'h2' || tag === 'h3' || tag === 'h4' || tag === 'h5' || tag === 'h6') return 'heading';
    if (tag === 'p') return 'paragraph';
    if (tag === 'ul' || tag === 'ol') return 'list';
    if (tag === 'span' && (cls.includes('badge') || cls.includes('tag') || cls.includes('pill'))) return 'badge';
    if (cls.includes('card')) return 'card';
    if (cls.includes('avatar')) return 'avatar';
    if (cls.includes('divider') || tag === 'hr') return 'divider';
    if (cls.includes('icon')) return 'icon';
    if (cls.includes('spacer')) return 'spacer';
    if (tag === 'a') return 'cta-link';
    if (tag === 'label') return 'label';
    return 'container';
  }

  function getPosition(el, parent) {
    const parentRect = parent.getBoundingClientRect();
    const elRect = el.getBoundingClientRect();
    const parentComputed = window.getComputedStyle(parent);

    if (parentComputed.display.includes('grid')) {
      const colStart = getComputedStyle(el).gridColumnStart;
      const rowStart = getComputedStyle(el).gridRowStart;
      return { type: 'grid-cell', gridColumn: colStart, gridRow: rowStart };
    }
    if (parentComputed.display.includes('flex')) {
      const order = getComputedStyle(el).order;
      return { type: 'flex-order', order: parseInt(order) || 0 };
    }
    return {
      type: 'flow',
      offsetLeft: Math.round(elRect.left - parentRect.left),
      offsetTop: Math.round(elRect.top - parentRect.top),
    };
  }

  function inferResponsive(el, parentComputed) {
    // Quick heuristic: if element width percentage is high and parent is flex/grid,
    // it likely stacks on mobile
    const width = el.getBoundingClientRect().width;
    const parentWidth = el.parentElement.getBoundingClientRect().width;
    const widthRatio = width / parentWidth;
    if (parentComputed.display.includes('grid') || parentComputed.display.includes('flex')) {
      if (widthRatio > 0.6) return 'likely-stacks-on-mobile';
      return 'likely-reflows';
    }
    return 'stable';
  }

  const seenSections = new Set();
  sectionEls.forEach((sectionEl) => {
    if (seenSections.has(sectionEl)) return;
    seenSections.add(sectionEl);

    const sectionRect = sectionEl.getBoundingClientRect();
    const sectionTop = Math.round(sectionRect.top + window.scrollY);
    const sectionClasses = sectionEl.className || sectionEl.tagName.toLowerCase();
    const sectionComputed = window.getComputedStyle(sectionEl);

    const directChildren = Array.from(sectionEl.children);
    // Also look one level deeper for components inside wrapper divs
    const allDescendants = [];
    directChildren.forEach((child) => {
      const childType = classifyComponent(child);
      if (childType !== 'container') {
        allDescendants.push({ el: child, type: childType, parent: sectionEl, depth: 1 });
      } else {
        // Dig into wrapper
        Array.from(child.children).forEach((grandchild) => {
          const gcType = classifyComponent(grandchild);
          if (gcType !== 'container') {
            allDescendants.push({ el: grandchild, type: gcType, parent: child, depth: 2 });
          } else {
            Array.from(grandchild.children).forEach((ggc) => {
              const ggcType = classifyComponent(ggc);
              if (ggcType !== 'container') {
                allDescendants.push({ el: ggc, type: ggcType, parent: grandchild, depth: 3 });
              }
            });
          }
        });
      }
    });

    allDescendants.forEach(({ el, type, parent, depth }) => {
      const rect = el.getBoundingClientRect();
      const computed = window.getComputedStyle(el);
      const pos = getPosition(el, parent);

      componentMap.push({
        section: sectionClasses.substring(0, 80),
        sectionTopOffset: sectionTop,
        componentType: type,
        depth: depth,
        position: pos,
        width: Math.round(rect.width),
        height: Math.round(rect.height),
        textContent: el.textContent.trim().substring(0, 100),
        display: computed.display,
        gap: computed.gap !== 'normal' ? computed.gap : null,
        alignItems: computed.alignItems !== 'normal' ? computed.alignItems : null,
        justifyContent: computed.justifyContent !== 'normal' ? computed.justifyContent : null,
        responsiveHint: inferResponsive(el, sectionComputed),
        tag: el.tagName.toLowerCase(),
        classes: el.className ? el.className.substring(0, 120) : '',
      });
    });
  });

  return {
    componentMap: componentMap,
    totalComponents: componentMap.length,
    typeBreakdown: componentMap.reduce((acc, c) => {
      acc[c.componentType] = (acc[c.componentType] || 0) + 1;
      return acc;
    }, {}),
  };
})();
```

---

## Script 3: Navigation Structure

Extracts the full navigation structure including nav items, logo placement, CTA button presence, mobile menu trigger, layout direction, and sticky/fixed behavior.

### Extraction Script

```javascript
(() => {
  const navEl = document.querySelector('nav, header, [role="banner"], [class*="navbar"], [class*="nav-bar"], [class*="navigation"]');
  if (!navEl) {
    return { found: false, message: 'No navigation element detected on the page.' };
  }

  const navRect = navEl.getBoundingClientRect();
  const navComputed = window.getComputedStyle(navEl);

  // Extract nav items
  const navItems = [];
  const linkEls = navEl.querySelectorAll('a, [role="menuitem"]');
  linkEls.forEach((link) => {
    const text = link.textContent.trim().substring(0, 50);
    const href = link.getAttribute('href') || '';
    const lr = link.getBoundingClientRect();
    navItems.push({
      text: text,
      href: href,
      position: 'inline',
      offsetX: Math.round(lr.left - navRect.left),
      offsetY: Math.round(lr.top - navRect.top),
      width: Math.round(lr.width),
    });
  });

  // Detect logo
  const logoEl = navEl.querySelector('img[class*="logo"], svg[class*="logo"], [class*="logo"], [class*="brand"]');
  let logoPlacement = null;
  if (logoEl) {
    const lr = logoEl.getBoundingClientRect();
    logoPlacement = {
      tag: logoEl.tagName.toLowerCase(),
      classes: logoEl.className || '',
      offsetX: Math.round(lr.left - navRect.left),
      offsetY: Math.round(lr.top - navRect.top),
      width: Math.round(lr.width),
      height: Math.round(lr.height),
    };
  }

  // Detect CTA button in nav
  const ctaEl = navEl.querySelector('button[class*="cta"], a[class*="cta"], a[class*="btn"], button[class*="btn"], button[class*="button"], a[class*="button"]');
  let ctaButton = null;
  if (ctaEl) {
    const cr = ctaEl.getBoundingClientRect();
    ctaButton = {
      tag: ctaEl.tagName.toLowerCase(),
      text: ctaEl.textContent.trim().substring(0, 50),
      href: ctaEl.getAttribute('href') || '',
      offsetX: Math.round(cr.left - navRect.left),
      offsetY: Math.round(cr.top - navRect.top),
      width: Math.round(cr.width),
      height: Math.round(cr.height),
    };
  }

  // Detect mobile menu trigger
  const mobileTriggerEl = navEl.querySelector('[class*="hamburger"], [class*="menu-toggle"], [class*="mobile-menu"], [class*="burger"], button[aria-label*="menu"], button[aria-expanded]');
  let mobileMenu = null;
  if (mobileTriggerEl) {
    mobileMenu = {
      tag: mobileTriggerEl.tagName.toLowerCase(),
      classes: mobileTriggerEl.className || '',
      ariaLabel: mobileTriggerEl.getAttribute('aria-label') || '',
      ariaExpanded: mobileTriggerEl.getAttribute('aria-expanded') || '',
      visible: mobileTriggerEl.getBoundingClientRect().width > 0,
    };
  }

  // Determine layout
  let layout = 'horizontal';
  const display = navComputed.display;
  if (display.includes('flex')) {
    const dir = navComputed.flexDirection;
    const justify = navComputed.justifyContent;
    if (dir.includes('column')) {
      layout = 'vertical';
    } else if (justify === 'center') {
      layout = 'horizontal-centered';
    } else if (justify === 'space-between') {
      layout = 'space-between';
    } else {
      layout = 'horizontal';
    }
  }

  // Sticky/fixed behavior
  let stickyBehavior = 'static';
  const position = navComputed.position;
  if (position === 'fixed') stickyBehavior = 'fixed';
  else if (position === 'sticky') stickyBehavior = 'sticky';
  else if (position === 'absolute') stickyBehavior = 'absolute';

  // Check if nav is at top of page
  const isAtTop = navRect.top <= 10;

  return {
    found: true,
    navTag: navEl.tagName.toLowerCase(),
    navClasses: navEl.className || '',
    navHeight: Math.round(navRect.height),
    navWidth: Math.round(navRect.width),
    layout: layout,
    display: display,
    position: position,
    stickyBehavior: stickyBehavior,
    isAtTop: isAtTop,
    navItems: navItems,
    navItemCount: navItems.length,
    logoPlacement: logoPlacement,
    ctaButton: ctaButton,
    mobileMenuTrigger: mobileMenu,
    backgroundColor: navComputed.backgroundColor !== 'rgba(0, 0, 0, 0)' ? navComputed.backgroundColor : 'transparent',
  };
})();
```

---

## Script 4: Responsive Breakpoints

Tests the page at five widths: 1920, 1440, 1024, 768, and 375. At each width it re-extracts section heights, detects layout changes (grid to flex to stack), notes navigation changes, and records content reflow patterns.

> **Important:** This script must be run once per breakpoint width. Before each run, call `browser_resize(width, height)` where height is 1080, then wait 500ms for reflow to settle, then execute this script. Collect results for all five widths into the `responsiveBreakpoints` array in the final output.

### Extraction Script

```javascript
(() => {
  const currentWidth = window.innerWidth;
  const selectors = [
    'section', 'header', 'footer', 'main',
    '[class*="section"]', '[class*="hero"]', '[class*="banner"]',
    '[class*="cta"]', '[class*="feature"]', '[class*="pricing"]',
    '[class*="testimonial"]', '[role="banner"]', '[role="contentinfo"]'
  ];
  const sectionEls = document.querySelectorAll(selectors.join(', '));

  const sectionData = [];
  sectionEls.forEach((el) => {
    const rect = el.getBoundingClientRect();
    const computed = window.getComputedStyle(el);
    const display = computed.display;

    let layoutType = 'block';
    if (display.includes('grid')) layoutType = 'grid';
    else if (display.includes('flex')) layoutType = 'flex';

    let gridCols = null;
    if (layoutType === 'grid') {
      gridCols = computed.gridTemplateColumns;
    }

    let flexDir = null;
    let flexWrap = null;
    if (layoutType === 'flex') {
      flexDir = computed.flexDirection;
      flexWrap = computed.flexWrap;
    }

    sectionData.push({
      tag: el.tagName.toLowerCase(),
      classes: (el.className || '').substring(0, 80),
      height: Math.round(rect.height),
      width: Math.round(rect.width),
      layoutType: layoutType,
      gridTemplateColumns: gridCols,
      flexDirection: flexDir,
      flexWrap: flexWrap,
      childCount: el.children.length,
    });
  });

  // Navigation state at this width
  const navEl = document.querySelector('nav, header, [role="banner"]');
  let navState = null;
  if (navEl) {
    const navRect = navEl.getBoundingClientRect();
    const navComputed = window.getComputedStyle(navEl);
    const mobileTrigger = navEl.querySelector('[class*="hamburger"], [class*="menu-toggle"], [class*="burger"], button[aria-label*="menu"]');
    navState = {
      height: Math.round(navRect.height),
      navItemsVisible: navEl.querySelectorAll('a:not([class*="hamburger"]):not([class*="burger"])').length,
      mobileMenuVisible: mobileTrigger ? mobileTrigger.getBoundingClientRect().width > 0 : false,
      layout: navComputed.display.includes('flex') ? navComputed.justifyContent : 'unknown',
    };
  }

  // Detect overflow / horizontal scroll
  const hasHorizontalScroll = document.documentElement.scrollWidth > window.innerWidth;

  // Content reflow detection
  const reflowNotes = [];
  if (currentWidth <= 768) {
    sectionData.forEach((s) => {
      if (s.layoutType === 'grid' && s.gridTemplateColumns) {
        const cols = s.gridTemplateColumns.split(' ').length;
        if (cols === 1) reflowNotes.push(`${s.classes.substring(0, 40)}: grid collapsed to 1 col`);
      }
      if (s.layoutType === 'flex' && s.flexDirection && s.flexDirection.includes('column')) {
        reflowNotes.push(`${s.classes.substring(0, 40)}: flex stacked vertically`);
      }
    });
    if (navState && navState.mobileMenuVisible) {
      reflowNotes.push('navigation: hamburger menu active');
    }
  }

  return {
    breakpoint: currentWidth,
    viewportHeight: window.innerHeight,
    sectionHeights: sectionData.map(s => ({
      section: s.classes || s.tag,
      height: s.height,
      layoutType: s.layoutType,
      gridColumns: s.gridTemplateColumns,
      flexDirection: s.flexDirection,
      flexWrap: s.flexWrap,
      childCount: s.childCount,
    })),
    navigation: navState,
    hasHorizontalScroll: hasHorizontalScroll,
    reflowNotes: reflowNotes,
  };
})();
```

---

## Output Format

After running all four scripts (Script 4 is run five times — once per breakpoint), the agent assembles the following combined JSON object and saves it to the extraction output file:

```json
{
  "source": "https://example.com",
  "extractedAt": "2026-09-14T12:00:00Z",
  "sections": [
    {
      "tag": "header",
      "classes": "site-header sticky",
      "id": "masthead",
      "height": 80,
      "topOffset": 0,
      "bottomOffset": 80,
      "backgroundColor": "#ffffff",
      "padding": "0px 24px",
      "paddingTop": "0px",
      "paddingBottom": "0px",
      "paddingLeft": "24px",
      "paddingRight": "24px",
      "maxWidth": "1200px",
      "containerWidth": 1200,
      "parentWidth": 1440,
      "layoutType": "flex",
      "display": "flex",
      "gridTemplateColumns": null,
      "gridTemplateRows": null,
      "gridColumnCount": 0,
      "flexDirection": "row",
      "alignItems": "center",
      "justifyContent": "space-between",
      "flexWrap": "nowrap",
      "gap": "0px",
      "childCount": 3,
      "childTags": ["div", "nav", "div"],
      "textContentPreview": "Brand | Features Pricing About | Get Started"
    }
  ],
  "componentMap": [
    {
      "section": "hero-banner",
      "sectionTopOffset": 80,
      "componentType": "heading",
      "depth": 2,
      "position": { "type": "flex-order", "order": 0 },
      "width": 580,
      "height": 72,
      "textContent": "Build Better Products Faster",
      "display": "block",
      "gap": null,
      "alignItems": null,
      "justifyContent": null,
      "responsiveHint": "stable",
      "tag": "h1",
      "classes": "hero-title"
    }
  ],
  "navigation": {
    "found": true,
    "navTag": "header",
    "navClasses": "site-header sticky",
    "navHeight": 80,
    "navWidth": 1440,
    "layout": "space-between",
    "display": "flex",
    "position": "sticky",
    "stickyBehavior": "sticky",
    "isAtTop": true,
    "navItems": [
      { "text": "Features", "href": "/features", "position": "inline", "offsetX": 200, "offsetY": 28, "width": 80 }
    ],
    "navItemCount": 5,
    "logoPlacement": { "tag": "img", "classes": "logo", "offsetX": 0, "offsetY": 20, "width": 120, "height": 40 },
    "ctaButton": { "tag": "a", "text": "Get Started", "href": "/signup", "offsetX": 1300, "offsetY": 24, "width": 120, "height": 36 },
    "mobileMenuTrigger": { "tag": "button", "classes": "hamburger", "ariaLabel": "Open menu", "ariaExpanded": "false", "visible": false },
    "backgroundColor": "#ffffff"
  },
  "responsiveBreakpoints": [
    {
      "breakpoint": 1920,
      "viewportHeight": 1080,
      "sectionHeights": [ { "section": "hero-banner", "height": 600, "layoutType": "flex", "gridColumns": null, "flexDirection": "row", "flexWrap": "nowrap", "childCount": 2 } ],
      "navigation": { "height": 80, "navItemsVisible": 5, "mobileMenuVisible": false, "layout": "space-between" },
      "hasHorizontalScroll": false,
      "reflowNotes": []
    },
    {
      "breakpoint": 375,
      "viewportHeight": 812,
      "sectionHeights": [ { "section": "hero-banner", "height": 400, "layoutType": "flex", "gridColumns": null, "flexDirection": "column", "flexWrap": "wrap", "childCount": 2 } ],
      "navigation": { "height": 64, "navItemsVisible": 0, "mobileMenuVisible": true, "layout": "space-between" },
      "hasHorizontalScroll": false,
      "reflowNotes": ["hero-banner: flex stacked vertically", "navigation: hamburger menu active"]
    }
  ],
  "totalSections": 7,
  "pageMaxWidth": "1200px",
  "pageBackground": "#ffffff"
}
```

---

## Execution Order

1. Navigate to the target URL and wait for `networkidle`.
2. Set viewport to 1440 x 1080.
3. Capture full-page screenshot.
4. Run **Script 1** (Page Section Structure) — save result to `extraction-sections.json`.
5. Run **Script 2** (Component Placement Map) — save result to `extraction-components.json`.
6. Run **Script 3** (Navigation Structure) — save result to `extraction-navigation.json`.
7. For each width in `[1920, 1440, 1024, 768, 375]`:
   a. Call `browser_resize(width, 1080)`.
   b. Wait 500ms for reflow.
   c. Run **Script 4** (Responsive Breakpoints).
   d. Append result to the `responsiveBreakpoints` array.
8. Merge all four script outputs into a single `extraction-output.json` following the Output Format above.

This combined JSON file is the input to Phase 1 (Layout Analysis).
