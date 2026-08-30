# DOM to Component Mapping Specification

## Purpose

Define how to map real DOM elements to design system component types. This specification is consumed by the Phase 0A browser extraction step (see `browser-extraction.md`, Step 0A.4). It tells the extraction script which HTML selectors to match, which elements to skip, which computed properties to extract per component type, and how to classify visual variants (e.g., button → primary/secondary/outline/ghost based on background, border, text color).

The 6 standard components are: Navigation, Button, Card, Input, Badge, CTA Link.

---

## Component Identification Rules

For each of the 6 standard components, the following rules apply:

### General Rules (all components)

1. **Match selectors in order.** Each component has a list of CSS selectors. Try them in the listed order; the first selector that matches an element claims it.
2. **Deduplicate.** If an element is already claimed by a previous component type, skip it. Use a `Set` keyed by the element's unique signature (tag + class list + bounding rect top).
3. **Visibility filters.** Skip any element that meets ANY of these conditions:
   - `getComputedStyle(el).display === 'none'`
   - `getComputedStyle(el).visibility === 'hidden'`
   - `getComputedStyle(el).opacity === '0'`
   - `rect.width === 0` or `rect.height === 0`
   - Element is an `<svg>`, `<path>`, `<circle>`, etc. that is a child of an already-claimed component (avoid double-counting icon paths as badges).
4. **Cap results.** Collect at most 15 instances per component type to keep output readable. Prefer elements higher in the DOM order.
5. **Extract the following for every matched element** (common to all types):
   - `tag` — lowercased tag name
   - `classes` — first 5 class tokens
   - `text` — trimmed textContent, capped at 100 characters
   - `dimensions` — `{ width, height }` from `getBoundingClientRect()`, rounded
   - `childrenCount` — `el.children.length`
   - `childTags` — first 5 child tag names

### Per-Component Computed Properties

Each component type has additional computed properties to extract beyond the common set:

| Component | Extra computed properties to extract |
|---|---|
| Navigation | `display`, `flexDirection`, `alignItems`, `justifyContent`, `gap`, `padding`, `position`, `zIndex`, `height` (from rect), `backdropFilter` |
| Button | `backgroundColor`, `color`, `border`, `borderRadius`, `padding`, `fontSize`, `fontWeight`, `textTransform`, `letterSpacing`, `cursor`, `transition`, `clipPath` |
| Card | `backgroundColor`, `border`, `borderRadius`, `boxShadow`, `padding`, `display` (grid/flex?), `gap`, `clipPath` |
| Input | `backgroundColor`, `color`, `border`, `borderRadius`, `padding`, `fontSize`, `lineHeight`, `height` (from rect), `placeholder` attribute |
| Badge | `backgroundColor`, `color`, `border`, `borderRadius`, `padding`, `fontSize`, `fontWeight`, `textTransform`, `letterSpacing`, `lineHeight` |
| CTA Link | `color`, `textDecoration`, `fontSize`, `fontWeight`, `textTransform`, `letterSpacing`, `borderBottom` (underline indicator), `transition`, `cursor` |

---

## Component Selector Patterns

Each component's selector list, tried in order:

### Navigation
`nav`, `header`, `[role="navigation"]`, `.nav`, `.navbar`, `.header`, `.menu`

### Button
`button`, `[role="button"]`, `.btn`, `.button`, `.cta`, `[type="submit"]`, `a[class*="btn"]`

### Card
`.card`, `[class*="card"]`, `.panel`, `.tile`, `.item`, `article`

### Input
`input`, `textarea`, `select`, `.input`, `[class*="input"]`, `[class*="field"]`

### Badge
`.badge`, `.tag`, `.chip`, `.label`, `[class*="badge"]`, `[class*="tag"]`

### CTA Link
`a`, `.link`, `.cta-link`, `[class*="cta"]`, `a[class*="button"]`

> **Note on `<a>` ambiguity:** The `a` tag matches both CTA Link and Button (`a[class*="btn"]`). The Button selector list includes `a[class*="btn"]` before the CTA Link `a` catch-all, so link-styled buttons are claimed as Buttons first. Plain `<a>` tags without button-like classes fall through to CTA Link.

---

## Variant Classification Rules

After identifying a component element, classify its visual variant from computed styles. Variant names feed into the component contract's `representativeVariants` in Phase 3.

### Button Variants

Classify by computed `background-color` and `border`:

| Variant | Rule |
|---|---|
| **Solid fill (primary)** | `background-color` is non-transparent (`rgba(...,0)` excluded) AND has high saturation. This is the dominant call-to-action button. |
| **Outline** | `background-color` is transparent AND `border-width >= 1px` (border is visible). |
| **Ghost** | `background-color` is transparent AND `border-width` is `0px` or border color matches background (effectively no border). Relies on hover/focus state changes. |
| **Text link** | Element is an `<a>` or `[role="button"]` that looks like text — `background-color` transparent, no border, `cursor: pointer`, and font size matches body text. Typically has `text-decoration: underline` or underline on hover. |

Classification pseudocode:
```javascript
function classifyButton(el, cs) {
  const bgTransparent = cs.backgroundColor === 'rgba(0, 0, 0, 0)' || cs.backgroundColor === 'transparent';
  const borderWidth = parseFloat(cs.borderTopWidth) || 0;
  const hasBorder = borderWidth >= 1;

  if (!bgTransparent) return 'primary';
  if (hasBorder) return 'outline';
  if (el.tagName === 'A' || el.getAttribute('role') === 'button') return 'text-link';
  return 'ghost';
}
```

### Card Variants

Classify by `box-shadow` and `border`:

| Variant | Rule |
|---|---|
| **Elevated** | `box-shadow` is present (not `none`). Shadow provides visual depth. |
| **Outlined** | `box-shadow` is `none` AND `border-width >= 1px` (border defines the card edge). |
| **Flat** | `box-shadow` is `none` AND `border-width` is `0px`. Relies on background-color contrast for separation. |

Classification pseudocode:
```javascript
function classifyCard(cs) {
  const hasShadow = cs.boxShadow && cs.boxShadow !== 'none';
  const borderWidth = parseFloat(cs.borderTopWidth) || 0;

  if (hasShadow) return 'elevated';
  if (borderWidth >= 1) return 'outlined';
  return 'flat';
}
```

### Other Components

Navigation, Input, Badge, and CTA Link do not have formal variant classification rules in this spec. Their variants are derived in Phase 3 from the collected computed traits (e.g., input variant = filled vs outlined based on background + border; badge variant = solid vs outline based on background transparency).

---

## Layout Structure Extraction

Beyond individual components, extract the page's layout structure to inform the UIKit layout in Phase 4.

### Container / Wrapper Elements

Identify elements that act as content-width containers:
- Check `max-width` — values like `1200px`, `1280px`, `1400px` indicate a constrained container
- Check `margin: 0 auto` or `margin-inline: auto` — indicates centering
- Check `padding` — containers typically have horizontal padding (e.g., `0 24px`)
- Record the `max-width` value as `maxContentWidth` in key findings

### Grid Template Patterns

Extract `grid-template-columns` and `grid-template-rows` values. Look for:
- `repeat()` function usage (e.g., `repeat(3, 1fr)`)
- `minmax()` function (e.g., `minmax(280px, 1fr)`)
- `auto-fit` or `auto-fill` keywords (responsive auto-grid)
- Fixed column counts (e.g., `1fr 1fr 1fr` = 3-column grid)
- Named grid lines (rare but significant)

Record the most common grid pattern as `gridPattern` in key findings.

### Flex Layout Patterns

Record for each flex container:
- `flex-direction` (row vs column)
- `flex-wrap` (nowrap vs wrap)
- `align-items` (center, flex-start, flex-end, stretch)
- `justify-content` (center, space-between, space-around, space-evenly, flex-start)
- `gap` value

### Section Hierarchy Mapping

Map the vertical sequence of major page sections. Common marketing page hierarchy:

```
hero → features → testimonials → CTA → footer
```

Record the detected section order and each section's:
- Tag and classes
- Height (from `getBoundingClientRect()`)
- Background color
- `max-width` (if constrained)
- `padding`

This hierarchy feeds directly into the UIKit layout in Phase 4.

---

## Output Format

The DOM component mapping produces a JSON array. Each entry represents one identified component instance:

```json
[
  {
    "type": "button",
    "variant": "primary",
    "tag": "button",
    "classes": ["btn", "btn-primary", "cta"],
    "text": "立即下载",
    "computed": {
      "backgroundColor": "rgb(255, 250, 0)",
      "color": "rgb(25, 25, 25)",
      "border": "0px solid rgb(255, 250, 0)",
      "borderRadius": "0px",
      "padding": "12px 32px",
      "fontSize": "14px",
      "fontWeight": "700",
      "textTransform": "uppercase",
      "letterSpacing": "0.04em",
      "cursor": "pointer",
      "transition": "all 0.3s ease",
      "clipPath": "polygon(0 0, 100% 0, 100% 80%, 80% 100%, 0 100%)"
    },
    "dimensions": { "width": 160, "height": 44 },
    "childrenCount": 0,
    "childTags": []
  },
  {
    "type": "card",
    "variant": "elevated",
    "tag": "div",
    "classes": ["feature-card", "card"],
    "text": "智能终端 跨平台无缝协作",
    "computed": {
      "backgroundColor": "rgb(26, 26, 26)",
      "border": "1px solid rgb(42, 42, 42)",
      "borderRadius": "8px",
      "boxShadow": "0 4px 24px rgba(0, 0, 0, 0.5)",
      "padding": "24px",
      "display": "flex",
      "gap": "16px",
      "clipPath": "none"
    },
    "dimensions": { "width": 320, "height": 180 },
    "childrenCount": 3,
    "childTags": ["div", "h3", "p"]
  },
  {
    "type": "navigation",
    "variant": null,
    "tag": "header",
    "classes": ["site-header", "navbar"],
    "text": "首页 产品 下载 关于",
    "computed": {
      "display": "flex",
      "flexDirection": "row",
      "alignItems": "center",
      "justifyContent": "space-between",
      "gap": "0px",
      "padding": "0 48px",
      "position": "fixed",
      "zIndex": "100",
      "height": "72px",
      "backdropFilter": "blur(12px)"
    },
    "dimensions": { "width": 1920, "height": 72 },
    "childrenCount": 4,
    "childTags": ["a", "a", "a", "a"]
  },
  {
    "type": "badge",
    "variant": null,
    "tag": "span",
    "classes": ["tag", "tag-new"],
    "text": "NEW",
    "computed": {
      "backgroundColor": "rgb(0, 255, 162)",
      "color": "rgb(25, 25, 25)",
      "border": "0px",
      "borderRadius": "999px",
      "padding": "2px 10px",
      "fontSize": "11px",
      "fontWeight": "700",
      "textTransform": "uppercase",
      "letterSpacing": "0.06em",
      "lineHeight": "1.6"
    },
    "dimensions": { "width": 48, "height": 20 },
    "childrenCount": 0,
    "childTags": []
  }
]
```

---

## Rules

1. **Selector order matters.** Earlier selectors in a component's list take priority. An `a.btn` is claimed by Button (via `a[class*="btn"]`) before CTA Link (via `a`).
2. **Visibility is non-negotiable.** Never include `display: none`, `width: 0`, `height: 0`, or `opacity: 0` elements. They pollute frequency counts and produce phantom components.
3. **Cap at 15 per type.** If a page has 100 buttons, record the first 15 (in DOM order). This is enough for variant classification and keeps the output JSON under 50KB.
4. **Variant is nullable.** Components without a formal classification rule (navigation, input, badge, cta-link) set `variant: null`. Phase 3 derives their variants from the computed traits.
5. **Text is capped.** Always cap `text` at 100 characters to avoid massive JSON blobs from content-heavy elements.
6. **Clip-path is conditional.** Only include `clipPath` in computed output when it is not `none`. This signals a geometric design language.
