# Token CSS Specification

## Purpose

Define the structure, naming conventions, and content rules for `colors_and_type.css` — the central token file generated in Phase 2. This file is read by the Phase 2 token-generation sub-agent. It specifies the exact section order, CSS variable naming patterns, color scale rules, typography rules, shadow rules, and a validation checklist.

The output file MUST be valid CSS that defines all design tokens as CSS custom properties (variables) on `:root` and optional theme class blocks (`.dark`, `.light`).

---

## Required Sections (in order)

The file MUST contain these sections in the exact order below. Each section is separated by a comment header.

### 1. Font-Face Declarations (or font-family usage)

- **NEVER use `@import` for custom web fonts.** `@import` is blocked by performance rules and breaks the build.
- If the extraction found `@font-face` declarations with local/custom font names, declare the `@font-face` blocks here using the real `font-family` name and `src` URL from extraction.
- If the fonts are already loaded by the website's existing stylesheet (external), reference them directly via `font-family` without re-declaring `@font-face`.
- System fallbacks (`sans-serif`, `monospace`) are placed **last** in font stacks, never as the primary family.

### 2. `@group-priority` Comment

A comment block that lists the token groups in priority order. This communicates the design system's token hierarchy to consumers.

```css
/* @group-priority
   1. Color scales (brand primary, secondary, accent, neutral, semantic)
   2. Semantic color aliases
   3. Portable color aliases (--color-*)
   4. Typography (font-family, font-size, font-weight, line-height)
   5. Spacing
   6. Sizing (buttons, inputs, icons)
   7. Radius
   8. Shadows (glow + drop)
   9. Transitions & easing
   10. Letter spacing
*/
```

### 3. `:root` Block — All Tokens

The `:root` selector contains every token definition. This is the default theme. Order tokens by group priority (see section 2).

### 4. `.dark {}` Block (if dark theme is default)

If the brand profile indicates a dark-dominant UI (`personality` includes `dark` or `post-apocalyptic`, and the background color from extraction is `#1a1a1a` or darker), repeat the semantic aliases here with dark-theme-appropriate values.

Only **semantic aliases** and **portable aliases** are repeated in `.dark` — color scales stay in `:root`.

### 5. `.light {}` Block (if light override is needed)

If the brand supports a light variant (e.g., the extraction found light surface colors), define a `.light` block that overrides semantic aliases for the light theme. If no light variant is warranted, omit this block entirely.

---

## CSS Variable Naming Rules

### Color Scales

Pattern: `--{prefix}-{role}-{step}`

- `{prefix}` = the brand's `colorNamingPrefix` (e.g., `endfield`)
- `{role}` = the color group name (e.g., `primary`, `secondary`, `accent`, `neutral`)
- `{step}` = `50 | 100 | 200 | 300 | 400 | 500 | 600 | 700 | 800 | 900`

Example: `--endfield-primary-600`

### Semantic Aliases

Pattern: `--{role}`

Short, theme-aware aliases that reference scale steps. These are what components consume.

Examples:
- `--primary: var(--endfield-primary-600);`
- `--accent: var(--endfield-accent-500);`
- `--foreground: var(--endfield-neutral-50);`
- `--background: var(--endfield-neutral-950);`

### Portable Aliases

Pattern: `--color-{role}`

Framework-agnostic aliases that any component library can consume. These MUST be defined in addition to semantic aliases.

Examples:
- `--color-primary`, `--color-secondary`, `--color-accent`
- `--color-surface`, `--color-foreground`, `--color-background`
- `--color-border`, `--color-muted`

### Typography

| Token pattern | Example |
|---|---|
| `--font-family-{display\|body\|mono\|impact}` | `--font-family-display`, `--font-family-body` |
| `--font-size-{display\|h1\|h2\|h3\|h4\|lead\|body\|caption\|mono}` | `--font-size-h1`, `--font-size-body` |
| `--font-weight-{display\|h1\|h2\|h3\|h4\|body\|caption\|mono}` | `--font-weight-display`, `--font-weight-body` |
| `--line-height-{display\|h1\|h2\|h3\|body\|caption}` | `--line-height-h1` |

### Spacing

Pattern: `--space-{1-8}`

Values based on the extracted rem/px spacing pattern. Map the most frequent extracted padding/gap values to steps 1–8.

Example:
```css
--space-1: 0.25rem;   /* 4px  */
--space-2: 0.5rem;    /* 8px  */
--space-3: 0.75rem;   /* 12px */
--space-4: 1rem;      /* 16px */
--space-5: 1.5rem;     /* 24px */
--space-6: 2rem;      /* 32px */
--space-7: 3rem;      /* 48px */
--space-8: 4rem;      /* 64px */
```

### Sizing

| Token pattern | Example |
|---|---|
| `--size-button-{sm\|md\|lg}` | `--size-button-md` |
| `--size-input-height` | `--size-input-height: 48px;` |
| `--size-icon-{sm\|md\|lg}` | `--size-icon-md` |

### Radius

Pattern: `--radius-{none\|sm\|md\|lg\|full\|pill}`

Example:
```css
--radius-none: 0;
--radius-sm: 2px;
--radius-md: 4px;
--radius-lg: 8px;
--radius-full: 9999px;
--radius-pill: 9999px;
```

### Shadows

Pattern: `--shadow-{1-5}`

Each shadow classified as `glow` or `drop` with a comment.

Example:
```css
--shadow-1: 0 1px 2px rgba(0,0,0,0.3);                          /* drop */
--shadow-2: 0 4px 12px rgba(0,0,0,0.5);                          /* drop */
--shadow-3: 0 0 8px rgba(255,250,0,0.3);                         /* glow */
--shadow-4: 0 0 16px rgba(255,250,0,0.5);                        /* glow */
--shadow-5: 0 0 10px #fff000;                                    /* glow, brand-colored */
```

### Transitions

| Token pattern | Example |
|---|---|
| `--transition-{fast\|base\|smooth}` | `--transition-fast: 0.15s ease;` |
| `--ease-{natural\|linear}` | `--ease-natural: cubic-bezier(0.4, 0, 0.2, 1);` |

### Letter Spacing

Pattern: `--tracking-{tight\|tighter\|tightest}`

Example:
```css
--tracking-tight: -0.01em;
--tracking-tighter: -0.02em;
--tracking-tightest: -0.04em;
```

---

## Color Scale Rules

1. **FULL 10-step scale** (50, 100, 200, 300, 400, 500, 600, 700, 800, 900) for every color group. No group may have fewer than 10 steps.

2. **Exactly one token per group** is marked with `/* @primary */` — this is the anchor step (the step closest to the real brand color).

3. **Anchor value MUST be the real extracted brand color.** The `@primary` step's hex value must match (or be the closest step to) a color found in Phase 0 extraction.

4. **All colors marked `/* AI-generated */`** after the value, except the anchor which has a Source comment.

5. **Source comment for anchor:**
   ```css
   --endfield-primary-600: #fffa00; /* @primary */ /* Source: https://endfield.hypergryph.com */
   ```

6. **Scale progression:** 50 = lightest tint, 900 = darkest shade. The scale must progress monotonically in lightness (50 is the lightest, 900 is the darkest for a given hue).

7. **Semantic colors:** define full scales for:
   - `success` (green-based)
   - `warning` (yellow-based)
   - `error` / `danger` (red-based)
   - `info` (blue-based)

Example of a complete color group:
```css
--endfield-primary-50:  #fffdf0;  /* AI-generated */
--endfield-primary-100: #fffab8;  /* AI-generated */
--endfield-primary-200: #fff680;  /* AI-generated */
--endfield-primary-300: #fff34d;  /* AI-generated */
--endfield-primary-400: #fff11a;  /* AI-generated */
--endfield-primary-500: #fffa00;  /* AI-generated */
--endfield-primary-600: #fffa00;  /* @primary */ /* Source: https://endfield.hypergryph.com */
--endfield-primary-700: #ccc800;  /* AI-generated */
--endfield-primary-800: #999600;  /* AI-generated */
--endfield-primary-900: #666400;  /* AI-generated */
```

---

## Typography Rules

1. **Font families MUST use real `@font-face` names from extraction.** If the extraction found a custom font named `EndfieldTitle`, the `--font-family-display` value must include `'EndfieldTitle'` as the primary family.

2. **NEVER use generic fallbacks as primary.** Forbidden primary values:
   - `"System Sans"`
   - `"系统字体栈"`
   - `system-ui` as the sole/primary family
   - Any unnamed generic family as the first item in the stack

3. **System fallbacks go LAST** in the font stack:
   ```css
   --font-family-display: 'EndfieldTitle', 'Impact', sans-serif;
   --font-family-body: 'DIN Alternate', sans-serif;
   --font-family-mono: 'JetBrains Mono', monospace;
   ```

4. **Font sizes MUST match extracted values.** Convert rem→px at 16px root when the source uses rem, and rem→px when the source uses px. The token value should preserve the original unit if the extraction was predominantly in one unit.

5. **Line-height values** must come from extraction. If line-height was not extracted, use standard ratios (1.5 for body, 1.1–1.2 for display) and mark them `/* AI-generated */`.

6. **Font weights** must match extracted values. Common weights: 400 (regular), 500 (medium), 600 (semibold), 700 (bold), 800/900 (heavy).

---

## Shadow Rules

1. **Classify each shadow** as `glow` (pattern: `0 0 Xpx color` — zero x/y offset) or `drop` (non-zero Y offset). Add a trailing comment.

2. **Include brand-colored glows** using extracted accent colors. These are signature visual elements for high-energy/dark UIs.

3. **Example with brand-colored glow:**
   ```css
   --shadow-5: 0 0 10px #fff000;  /* glow, brand-colored */
   ```

4. Provide 5 shadow tokens (`--shadow-1` through `--shadow-5`), mixing glow and drop types based on what was found in extraction.

---

## Validation Checklist

The Phase 2 sub-agent MUST self-verify every item below before finalizing the file:

- [ ] All color groups have complete 10-step scales (50–900)
- [ ] Each color group has exactly one step marked `/* @primary */`
- [ ] All non-anchor colors have `/* AI-generated */` comment
- [ ] Anchor colors have a `/* Source: {url} */` comment
- [ ] No `@import` is used for custom fonts
- [ ] No undefined CSS variables are referenced in `.dark` or `.light` theme blocks (every `var(--x)` must resolve to a token defined in `:root`)
- [ ] `@group-priority` comment is present and lists all token groups
- [ ] Portable aliases are defined (`--color-*`, `--radius-*`, `--type-*` or `--font-*`)
- [ ] Font families use real extracted `@font-face` names (no generic primary)
- [ ] Semantic color groups (success, warning, error, info) are present
- [ ] Shadow tokens are classified as glow or drop
- [ ] Spacing tokens (`--space-1` through `--space-8`) are present
- [ ] Letter-spacing tokens (`--tracking-*`) are present
- [ ] Transition and easing tokens are present

If any checklist item fails, the sub-agent must fix the file before reporting completion.
