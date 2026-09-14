# Token Contract

Every `colors_and_type.css` file MUST define these portable variables. This is the contract that enables any token set to work with any site template.

## Layer 1: Brand-Specific Variables

Brand-specific variables use a prefix (e.g., `--openai-*`, `--endfield-*`, `--voith-*`). These hold the raw measured values.

Example:
```css
--openai-primary-600: #000000;
--endfield-primary-600: #fffa00;
```

## Layer 2: Semantic Aliases

Map brand variables to semantic names:
```css
--primary: var(--brand-primary-600);
--background: var(--brand-neutral-900);
--foreground: #ffffff;
--border: var(--brand-neutral-700);
```

## Layer 3: Portable Aliases (CRITICAL)

These are the variables that site templates reference. EVERY token CSS file MUST define ALL of these:

### Colors (14 variables)

| Variable | Purpose | Example Fallback |
|----------|---------|-------------------|
| `--color-background` | Page background | `#ffffff` or `#191919` |
| `--color-foreground` | Body text | `#000000` or `#ffffff` |
| `--color-muted-foreground` | Secondary text | `rgba(0,0,0,0.6)` |
| `--color-surface` | Raised surface | `#ffffff` |
| `--color-card` | Card background | same as surface |
| `--color-muted` | Muted surface | `rgba(0,0,0,0.04)` |
| `--color-border` | Border color | `#e5e5e5` |
| `--color-primary` | Primary button/link | `#000000` |
| `--color-on-primary` | Text on primary | `#ffffff` |
| `--color-primary-hover` | Hover state | `#333333` |
| `--color-accent` | Accent color | `#10a37f` |
| `--color-on-accent` | Text on accent | `#ffffff` |
| `--color-secondary` | Secondary surface | `rgba(0,0,0,0.04)` |
| `--color-on-secondary` | Text on secondary | `#000000` |

### Typography (16 variables)

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `--font-display` | Display/headline font | `"Tiempos Headline", serif` |
| `--font-heading` | Heading font | `"Inter", sans-serif` |
| `--font-body` | Body copy font | `"Inter", sans-serif` |
| `--font-mono` | Monospace/code font | `"Söhne Mono", monospace` |
| `--font-size-display` | Display size | `3.5rem` |
| `--font-size-h1` | H1 size | `2.5rem` |
| `--font-size-h2` | H2 size | `2rem` |
| `--font-size-h3` | H3 size | `1.5rem` |
| `--font-size-h4` | H4 size | `1.25rem` |
| `--font-size-body` | Body size | `1rem` |
| `--font-size-lead` | Lead paragraph size | `1.125rem` |
| `--font-size-caption` | Caption/small size | `0.875rem` |
| `--font-size-nav` | Navigation size | `0.875rem` |
| `--font-size-button` | Button size | `0.875rem` |
| `--font-weight-heading` | Heading weight | `600` |
| `--font-weight-body` | Body weight | `400` |

### Spacing (8 variables)

| Variable | Value |
|----------|-------|
| `--space-1` | `4px` |
| `--space-2` | `8px` |
| `--space-3` | `12px` |
| `--space-4` | `16px` |
| `--space-5` | `24px` |
| `--space-6` | `32px` |
| `--space-7` | `48px` |
| `--space-8` | `64px` |

### Sizing (7 variables)

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `--max-content` | Max content width | `1280px` |
| `--size-nav` | Navigation height | `64px` |
| `--size-button-sm` | Small button height | `32px` |
| `--size-button-md` | Medium button height | `40px` |
| `--size-button-lg` | Large button height | `48px` |
| `--size-icon-sm` | Small icon size | `16px` |
| `--size-icon-md` | Medium icon size | `24px` |
| `--size-icon-lg` | Large icon size | `32px` |

### Radius (4 variables)

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `--radius-sm` | Small radius | `4px` |
| `--radius-md` | Medium radius | `8px` |
| `--radius-lg` | Large radius | `16px` |
| `--radius-full` | Full/pill radius | `9999px` |

### Shadows (5 variables)

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `--shadow-1` | Subtle shadow | `0 1px 2px rgba(0,0,0,0.05)` |
| `--shadow-2` | Light shadow | `0 2px 4px rgba(0,0,0,0.08)` |
| `--shadow-3` | Medium shadow | `0 4px 12px rgba(0,0,0,0.10)` |
| `--shadow-4` | Elevated shadow | `0 8px 24px rgba(0,0,0,0.12)` |
| `--shadow-5` | Heavy shadow | `0 16px 48px rgba(0,0,0,0.16)` |

## Dark Theme Support

If the brand has a dark theme, define a `.dark` class that overrides the portable aliases:
```css
.dark {
  --color-background: #191919;
  --color-foreground: #ffffff;
  /* ... */
}
```

## Verification

A token CSS file passes the contract when:
1. All 14 color portable aliases are defined
2. All 16 typography portable aliases are defined
3. All 8 spacing variables are defined
4. All 7 sizing variables are defined
5. All 4 radius variables are defined
6. All 5 shadow variables are defined
7. If dark theme exists, `.dark` class overrides the relevant aliases
