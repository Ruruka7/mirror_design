# Asset Extraction Specification

## Purpose

Define how to extract all visual assets from a rendered page. This specification is consumed by the Phase 0A browser extraction step (see `browser-extraction.md`, Step 0A.7). It covers raster images, SVG icons, fonts, background patterns, favicons, and video/media.

The goal is to catalogue asset URLs and metadata — NOT to download everything. Downloading is deferred to Phase 2 where only the needed assets are fetched.

---

## Asset Categories

### 1. Raster Images

**Sources:** `<img>` tags, `<picture>` `<source>` elements, CSS `background-image` URLs.

**Extract per image:**
- `src` — the resolved absolute URL of the image
- `srcset` — full srcset string (for `<img>` and `<source>`)
- `naturalWidth` / `naturalHeight` — intrinsic pixel dimensions (from `img.naturalWidth`)
- `alt` — alt text (accessibility + content context)
- `format` — infer from URL or `type` attribute: `jpg`, `png`, `webp`, `avif`, `gif`
- `loading` — `lazy`, `eager`, or `null`

**Classify by role:**
- **Hero image** — large image (naturalWidth > 800) in a `[class*="hero"]` or `[class*="banner"]` section
- **Product image** — medium image inside a card or product container
- **Decorative image** — small image with empty `alt` or `role="presentation"`
- **Icon-as-image** — small image (naturalWidth <= 64) used as an icon

### 2. SVG Icons

**Sources:** inline `<svg>` elements in the DOM.

**Extract per SVG:**
- `viewBox` — the viewBox attribute (e.g., `"0 0 24 24"`)
- `width` / `height` — attribute values (may differ from rendered size)
- `classList` — class tokens (for icon-name inference)
- `pathCount` — number of `path`, `circle`, `rect`, `polygon`, `line`, `ellipse` children
- `fill` — computed fill color (from `getComputedStyle`)
- `stroke` — computed stroke color
- `strokeWidth` — computed stroke-width if line-style icons
- `innerHTML` — raw inner SVG markup, **truncated to 500 characters**

**Record icon size patterns:**
- Collect all `width`/`height` attribute pairs and tally them
- Common patterns: `16x16`, `20x20`, `24x24`, `32x32`
- The most frequent size is the design system's base icon size (usually 24px)

**Line vs filled icons:**
- Line icons: have `stroke` with no `fill` (or `fill="none"`)
- Filled icons: have `fill` with no `stroke` (or `stroke="none"`)
- Record this as `iconStyle: "line" | "filled" | "mixed"`

### 3. Fonts

**Sources:** `@font-face` declarations in CSS files, `<link>` stylesheet tags (Google Fonts etc.), and the browser's `document.fonts` API.

**From browser `document.fonts` API:**
- Iterate `document.fonts` (FontFaceSet)
- For each loaded font face, record: `family`, `weight`, `style`, `status` (`loaded` | `loading` | `unloaded`)

**From CSS `@font-face` declarations** (via `css-extraction.md`, category 16):
- `font-family` name
- `src` URL (the actual font file URL — woff2/woff/ttf)
- `font-weight`
- `font-style`

**From `<link>` tags:**
- `rel` — `stylesheet`, `preload`, `prefetch`
- `href` — the URL (Google Fonts CSS URL, or direct font file URL)
- `type` — MIME type if present

**Record font usage in computed styles:**
- Cross-reference the `fonts` frequency map from computed style extraction (Step 0A.3)
- Mark which `font-family` names are actually used in rendered elements (vs declared but unused)
- Identify: display font (used on h1/h2), body font (used on p/body), mono font (name contains "mono"/"code"/"consolas")

### 4. Background Patterns

**Sources:** CSS `background-image` with gradients or repeating patterns.

**Extract per pattern:**
- `backgroundImage` — the full computed `background-image` string (e.g., `linear-gradient(135deg, #fffa00, #00ffa2)`)
- `tag` and `classes` — the element it's applied to (for context)
- Truncate to 200 characters

**Identify pattern types:**
- **Brand gradient** — multi-color `linear-gradient` or `radial-gradient` with 2+ color stops. These become token values (`--gradient-brand`).
- **Hazard stripes** — `repeating-linear-gradient` with alternating solid colors (e.g., `repeating-linear-gradient(45deg, #191919, #191919 10px, #2a2a2a 10px, #2a2a2a 20px)`)
- **Noise textures** — `url(data:...)` or `url(...noise.png)` with low opacity
- **Solid color fallback** — `background-image: none` but `background-color` set (not a pattern, skip)

### 5. Favicon and Meta Icons

**Sources:** `<link>` tags in `<head>`.

**Extract:**
- `favicon` — URL from `link[rel="icon"]` or `link[rel="shortcut icon"]`
- `appleTouchIcon` — URL from `link[rel="apple-touch-icon"]`
- `ogImage` — URL from `meta[property="og:image"]`
- `manifest` — URL from `link[rel="manifest"]` (PWA icon source)

### 6. Video / Media

**Sources:** `<video>` and `<audio>` elements.

**Extract per video:**
- `src` or `<source>` URLs
- `poster` — poster image URL (shown before play)
- `width` / `height` — intrinsic or attribute dimensions
- `autoplay`, `loop`, `muted`, `controls` — boolean attributes
- `preload` — `none`, `metadata`, `auto`

> Media is rare on marketing landing pages but present on product showcase sites. Record it when found; it informs the "media" personality keyword in Phase 1.

---

## Asset URL Normalization

All extracted URLs must be normalized before recording:

| URL type | Normalization rule |
|---|---|
| **Relative URL** (e.g., `/fonts/myfont.woff2`, `../images/hero.jpg`) | Resolve against the page base URL using `new URL(relativeUrl, pageUrl).href`. Record the absolute URL. |
| **Protocol-relative** (e.g., `//cdn.example.com/img.png`) | Prepend `https:` → `https://cdn.example.com/img.png` |
| **Absolute CDN URL** (e.g., `https://web.hycdn.cn/...`) | Record as-is. Note the CDN domain. |
| **Data URI** (e.g., `data:image/svg+xml;base64,...`) | Record as-is but add `"inline": true` flag. Do not attempt to download. |
| **Blob URL** (e.g., `blob:https://example.com/uuid`) | Note as `"runtime": true` — generated at runtime, cannot be downloaded. Skip in Phase 2 download. |
| **Empty / missing** | Record as `null`. Do not crash. |

Normalization pseudocode:
```javascript
function normalizeUrl(rawUrl, pageUrl) {
  if (!rawUrl || rawUrl === 'undefined' || rawUrl === '[object Module]') return null;
  if (rawUrl.startsWith('data:')) return { url: rawUrl, inline: true };
  if (rawUrl.startsWith('blob:')) return { url: rawUrl, runtime: true, skip: true };
  try {
    return { url: new URL(rawUrl, pageUrl).href };
  } catch {
    return { url: rawUrl };
  }
}
```

---

## Download Rules

**Critical principle: Do NOT download all assets during extraction.** Downloading every image, font, and icon during Phase 0A is too slow and produces gigabytes of unnecessary files. Instead:

1. **Record URLs and metadata only** during Phase 0A extraction.
2. **Font files:** record the `src` URL from `@font-face` for potential download in Phase 2. Phase 2's token sub-agent decides whether to fetch the font file (usually it references fonts by name and lets the consuming page load them).
3. **SVG icons:** inline the SVG markup directly — it is already in the DOM, no download needed. Truncate to 500 chars per icon.
4. **Raster images:** record URLs only. Download only if the Phase 4 UIKit sub-agent needs a specific hero image as visual reference (optional).
5. **Brand gradients:** no download needed — they are CSS strings, recorded as text.
6. **Favicon:** record URL only. Optionally download in Phase 5 for the README.

> **Rationale:** A typical marketing page has 20-50 images, 5-15 font files, and 10-30 SVG icons. Downloading all during extraction would add 30-120 seconds and 50-500MB of temp files. The extraction phase should be fast (< 10 seconds) and produce a JSON catalog only.

---

## Output Format

All asset data is consolidated into a single JSON object with arrays per category:

```json
{
  "images": [
    {
      "src": "https://example.com/images/hero.jpg",
      "srcset": "https://example.com/images/hero.jpg 1x, https://example.com/images/hero@2x.jpg 2x",
      "naturalWidth": 1920,
      "naturalHeight": 1080,
      "alt": "Product hero banner",
      "format": "jpg",
      "loading": "eager",
      "role": "hero"
    }
  ],
  "svgs": [
    {
      "viewBox": "0 0 24 24",
      "width": "24",
      "height": "24",
      "classList": "icon icon-arrow-right",
      "pathCount": 2,
      "fill": "rgb(255, 250, 0)",
      "stroke": "none",
      "strokeWidth": "0px",
      "innerHTML": "<path d=\"M5 12h14M13 6l6 6-6 6\" stroke=\"currentColor\" stroke-width=\"2\" fill=\"none\"/>"
    }
  ],
  "pictureSources": [
    { "srcset": "https://example.com/images/hero.webp 1x", "media": "(min-width: 768px)", "type": "image/webp" }
  ],
  "backgroundImages": [
    { "tag": "div", "classes": ["hero-bg"], "backgroundImage": "linear-gradient(135deg, #fffa00, #00ffa2)" }
  ],
  "fonts": {
    "loadedViaAPI": [
      { "family": "\"Inter\"", "weight": "400", "style": "normal", "status": "loaded" },
      { "family": "\"Inter\"", "weight": "700", "style": "normal", "status": "loaded" }
    ],
    "fontFaceDeclarations": [
      { "family": "Inter", "src": "url(/fonts/inter-var.woff2)", "weight": "400", "style": "normal" }
    ],
    "linkTags": [
      { "rel": "stylesheet", "href": "https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" }
    ],
    "usedInComputedStyles": ["Inter", "sans-serif"],
    "displayFont": "Inter",
    "bodyFont": "Inter",
    "monoFont": null
  },
  "favicon": "https://example.com/favicon.ico",
  "appleTouchIcon": "https://example.com/apple-touch-icon.png",
  "ogImage": "https://example.com/og-image.jpg",
  "videos": [],
  "iconSizePattern": { "24x24": 18, "16x16": 5, "32x32": 2 },
  "iconStyle": "line"
}
```

---

## Asset Usage in Design Library

The extracted assets feed into downstream phases:

| Asset | Where it's used | How |
|---|---|---|
| **SVG icons** | Component previews (Phase 3) | SVG markup can become component preview decorations — inlined directly into preview HTML as decorative elements (e.g., an arrow icon inside a CTA button). Truncated markup is sufficient. |
| **Font URLs** | Token CSS (Phase 2) | `@font-face` src URLs go into token CSS `@font-face` or README caveats. The token CSS references fonts by family name; the consuming page provides the actual `@font-face` or `<link>`. |
| **Brand gradients** | Token values (Phase 2) | Multi-color gradients become `--gradient-brand` or `--gradient-accent` token values in `colors_and_type.css`. |
| **Favicon / og:image** | README (Phase 4) | Referenced in the README's "Visual Foundations" section as evidence of brand identity. |
| **Screenshots** | UIKit visual reference (Phase 4) | Full-page and viewport screenshots serve as visual reference for the UIKit sub-agent to match layout, color, and component arrangement. |
| **Raster images** | UIKit (optional, Phase 4) | Hero image URLs may be referenced in the UIKit HTML as background images to match the original site's visual feel. Not downloaded — hotlinked or replaced with placeholders. |
| **Icon size pattern** | Token sizing (Phase 2) | The most common icon size (e.g., 24px) becomes `--icon-size-base` in the token CSS. |
| **Icon style** | Component contracts (Phase 3) | "line" vs "filled" informs the `keyInsightSeed` for button and badge components. |
