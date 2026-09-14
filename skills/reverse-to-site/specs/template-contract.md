# Template Contract

Every `site-template.html` file MUST follow these rules. This ensures the template works with ANY token CSS file that follows the Token Contract.

## Rule 1: Single HTML File

The entire page — all sections, all CSS, all content — lives in ONE `.html` file. No external JS. No external CSS except the token link.

## Rule 2: Token CSS Link

The ONLY external CSS reference is:
```html
<link rel="stylesheet" href="colors_and_type.css">
```
Path is relative (same directory). No CDN links. No inline `<style>` that defines colors/fonts.

## Rule 3: Zero Hardcoded Visual Values

ALL visual properties use CSS variables with fallbacks:
```css
/* CORRECT */
color: var(--color-foreground, #000000);
background: var(--color-background, #ffffff);
font-family: var(--font-body, sans-serif);
padding: var(--space-6, 32px) var(--space-5, 24px);
border-radius: var(--radius-md, 12px);
box-shadow: var(--shadow-1, 0 1px 2px rgba(0,0,0,0.1));

/* FORBIDDEN */
color: #333333;
background: #f5f5f5;
font-family: 'Inter', sans-serif;
padding: 32px 24px;
```

The ONLY exception: fallback values inside `var()` may contain hardcoded values.

## Rule 4: Use Only Portable Variable Names

Templates reference variables from the Token Contract's Layer 3 (Portable Aliases) ONLY. Never reference brand-specific variables like `--openai-primary-600` or `--endfield-primary-600`.

## Rule 5: Always Include Fallbacks

Every `var()` must include a fallback:
```css
/* CORRECT */
color: var(--color-foreground, #000000);

/* FORBIDDEN — no fallback */
color: var(--color-foreground);
```

## Rule 6: Image Placeholders

Image areas use CSS gradients built from variables:
```css
.img-placeholder {
  background: linear-gradient(135deg, var(--color-muted, #f5f5f5), var(--color-border, #e5e5e5));
}
```

## Rule 7: Responsive

Include media queries at minimum:
- `@media (max-width: 1024px)` — tablet
- `@media (max-width: 768px)` — mobile

## Rule 8: Real Content

Use actual text from the source website. Headings, body copy, button labels, navigation items, dates, author names — all real. No "Lorem ipsum". No placeholder text.

## Rule 9: Complete Page Structure

The template recreates the source website's full page. It includes:
- Navigation (sticky, with mobile menu)
- Hero section
- All content sections from the source (in order)
- CTA section
- Footer with links

## Rule 10: Body Class for Dark Themes

If the source site is dark-themed, the `<body>` tag has `class="dark"` so that dark-themed token files (which define `.dark` overrides) can apply correctly. Light-themed templates have `<body>` with no class.

## Verification

A site template passes the contract when:
1. `grep -P '#[0-9a-fA-F]{3,8}' file.html` returns matches ONLY inside `var()` fallbacks
2. CSS link is exactly `href="colors_and_type.css"`
3. No references to brand-specific variables (no `--openai-`, `--endfield-`, `--voith-` prefixes)
4. ≥4 distinct sections exist
5. Responsive breakpoints exist
6. All `var()` calls include fallbacks
7. Content is real text from the source website
