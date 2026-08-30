# Component Preview HTML Specification

## Purpose

This specification defines the structure for `preview/component-{slug}.html` files. Each preview page renders a single component's representative variants in isolation, using only tokens from `colors_and_type.css`. Previews are the primary visual evidence for a component and must be openable directly in any browser with zero build step and zero external dependencies.

## File Naming

- One preview per component: `preview/component-button.html`, `preview/component-card.html`, etc.
- The `{slug}` segment MUST match the `slug` field in `component/{slug}.json`.

## Required HTML Structure

### Document skeleton

```html
<!DOCTYPE html>
<html lang="{brand-language}">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{BrandName} — {ComponentName}</title>
  <link rel="stylesheet" href="../colors_and_type.css">
</head>
<body>
  <!-- preview content -->
</body>
</html>
```

### `<head>` rules

- `lang` attribute on `<html>` MUST match the brand's language (e.g. `en`, `zh-CN`).
- `<meta charset="UTF-8">` is required.
- `<meta name="viewport" ...>` is required for responsive rendering.
- `<title>` MUST follow the pattern `{BrandName} — {ComponentName}` (em dash separator).
- The CSS link MUST be exactly: `<link rel="stylesheet" href="../colors_and_type.css">`. No other path, no query string, no alternate filename.
- NO inline `<style>` blocks EXCEPT demo-specific layout styling for the preview container itself (e.g. `.preview-container`, `.preview-grid`). The inline styles must NOT redefine component tokens.
- NO external JavaScript. NO external CSS besides the token file. NO CDN links. NO web fonts loaded via `<link>` or `@import` — fonts come from the brand tokens only.
- NO `<script>` tags of any kind.

### `<body>` rules

- The body background MUST match the brand `--background` (or `--color-background`) token.
- Set via inline style or a preview-only rule: `body { background: var(--background); }` (acceptable as demo layout styling).

## Layout Structure

The preview body uses the following structural classes. These are layout scaffolding, not component tokens, and may be defined in the permitted inline `<style>` block.

### `.preview-container`
- `max-width: 1200px`
- `margin: 0 auto` (centered)
- `padding: 48px`
- Wraps all preview content.

### `.preview-header`
- Displays the component name and brand name.
- Example markup:
  ```html
  <header class="preview-header">
    <h1>{ComponentName}</h1>
    <p>{BrandName} · Component Preview</p>
  </header>
  ```

### `.preview-section`
- Each representative variant lives in its own `.preview-section`.
- Each section has a visible label (the variant name from the contract).

### `.preview-grid`
- Grid layout for showing multiple variants side by side.
- Use `display: grid` with a sensible column count (e.g. `grid-template-columns: repeat(auto-fill, minmax(260px, 1fr))`).

### `.preview-dark-surface`
- A container whose background is `var(--color-surface)` (or the brand surface token).
- Used to demonstrate components on a raised surface distinct from the page background.

## Component Rendering Rules

1. **Token-only styling.** Each variant MUST use ONLY CSS variables from `colors_and_type.css`. The trait values defined in `component/{slug}.json` are applied as inline styles or a scoped class block, but every value must be a `var()` reference (per the Trait Value Rules in `component-contract.md`).
2. **Visually functional.** Components must render as real, interactive-looking elements — not placeholder boxes. Buttons look like buttons, inputs look like inputs.
3. **Interactive states via pure CSS.** Hover and focus states MUST be implemented with `:hover` and `:focus` / `:focus-visible` selectors. NO JavaScript for interactivity.
4. **Brand copy.** Text content should use `uiCopySamples` from the brand where appropriate. Do not fabricate copy that contradicts the brand voice.
5. **Readability.** All text must be readable against its background. Foreground colors must contrast with their backgrounds (use the brand's `--color-on-*` tokens which are already paired correctly).
6. **Variant count.** At least 3 variants MUST be shown, corresponding to the `representativeVariants` in the component contract.
7. **No screenshots.** Variants are rendered with live CSS, not `<img>` tags.

## Validation Checklist

Before a preview file is considered complete, verify every item:

- [ ] CSS link path is exactly `../colors_and_type.css`.
- [ ] No undefined CSS variables — every `var(--...)` resolves in `colors_and_type.css`.
- [ ] `body` background uses `--background` or `--color-background`.
- [ ] At least 3 variants shown.
- [ ] All text readable (contrast check — no token-on-token clashes).
- [ ] No external resources besides the token CSS (no JS, no CDN, no web fonts, no images).
- [ ] `<title>` follows `{BrandName} — {ComponentName}` pattern.
- [ ] `lang` attribute matches brand language.
- [ ] Hover and focus states are implemented with pure CSS selectors.
- [ ] Each variant section has a visible label.
