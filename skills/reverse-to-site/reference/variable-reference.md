# Variable Reference

Complete list of portable CSS variables that ALL token files define and ALL site templates use.

## How to Use

In a site template, always use this syntax:
```css
property: var(--variable-name, fallback-value);
```

## Colors

| Variable | Purpose | Light Fallback | Dark Fallback |
|----------|---------|---------------|---------------|
| --color-background | Page background | #ffffff | #191919 |
| --color-foreground | Primary text | #000000 | #ffffff |
| --color-muted-foreground | Secondary text | rgba(0,0,0,0.6) | rgba(255,255,255,0.6) |
| --color-surface | Raised surface | #ffffff | #2e2e2e |
| --color-card | Card background | #ffffff | #2e2e2e |
| --color-muted | Muted surface | rgba(0,0,0,0.04) | rgba(255,255,255,0.08) |
| --color-border | Borders | #e5e5e5 | #424242 |
| --color-primary | Primary actions | #000000 | #fffa00 |
| --color-on-primary | Text on primary | #ffffff | #191919 |
| --color-primary-hover | Hover state | #333333 | #fffb00 |
| --color-accent | Accent highlights | #10a37f | #00ffa2 |
| --color-on-accent | Text on accent | #ffffff | #191919 |
| --color-secondary | Secondary surface | rgba(0,0,0,0.04) | rgba(255,255,255,0.12) |
| --color-on-secondary | Text on secondary | #000000 | #ffffff |

## Typography — Font Families

| Variable | Purpose | Common Fallback |
|----------|---------|------------------|
| --font-display | Large display headings | sans-serif |
| --font-heading | Section headings | sans-serif |
| --font-body | Body text, UI | sans-serif |
| --font-mono | Code, labels, metadata | monospace |

## Typography — Sizes

| Variable | Purpose | Common Fallback |
|----------|---------|------------------|
| --font-size-display | Hero display | 64px |
| --font-size-h1 | Section title | 48px |
| --font-size-h2 | Subsection | 32px |
| --font-size-h3 | Card title | 24px |
| --font-size-h4 | Small heading | 20px |
| --font-size-body | Body text | 16px |
| --font-size-lead | Lead paragraph | 18px |
| --font-size-caption | Captions, metadata | 13px |
| --font-size-nav | Navigation links | 15px |
| --font-size-button | Button text | 14px |

## Typography — Weights

| Variable | Common Fallback |
|----------|------------------|
| --font-weight-display | 700 |
| --font-weight-h1 | 700 |
| --font-weight-h2 | 600 |
| --font-weight-h3 | 600 |
| --font-weight-h4 | 600 |
| --font-weight-body | 400 |
| --font-weight-lead | 400 |
| --font-weight-nav | 500 |
| --font-weight-button | 500 |

## Spacing

| Variable | Value |
|----------|-------|
| --space-1 | 4px |
| --space-2 | 8px |
| --space-3 | 12px |
| --space-4 | 16px |
| --space-5 | 24px |
| --space-6 | 32px |
| --space-7 | 48px |
| --space-8 | 64px |

## Sizing

| Variable | Purpose | Common Fallback |
|----------|---------|------------------|
| --max-content | Max content width | 1200px |
| --size-nav | Navigation height | 64px |
| --size-button-sm | Small button height | 32px |
| --size-button-md | Medium button height | 40px |
| --size-button-lg | Large button height | 48px |
| --size-icon-sm | Small icon | 16px |
| --size-icon-md | Medium icon | 20px |
| --size-icon-lg | Large icon | 24px |

## Radius

| Variable | Common Fallback |
|----------|------------------|
| --radius-sm | 4px |
| --radius-md | 8px |
| --radius-lg | 16px |
| --radius-full | 9999px |

## Shadows

| Variable | Purpose | Common Fallback |
|----------|---------|------------------|
| --shadow-1 | Card default | 0 1px 2px rgba(0,0,0,0.1) |
| --shadow-2 | Card hover | 0 4px 8px rgba(0,0,0,0.1) |
| --shadow-3 | Floating element | 0 8px 24px rgba(0,0,0,0.15) |
| --shadow-4 | Modal | 0 16px 40px rgba(0,0,0,0.2) |
| --shadow-5 | Overlay | 0 24px 60px rgba(0,0,0,0.25) |

## Quick Copy

For convenience, here are the most commonly used patterns:

```css
/* Body */
body {
  background: var(--color-background, #ffffff);
  color: var(--color-foreground, #000000);
  font-family: var(--font-body, sans-serif);
  font-size: var(--font-size-body, 16px);
  line-height: var(--line-height-body, 1.6);
}

/* Button */
.btn {
  background: var(--color-primary, #000000);
  color: var(--color-on-primary, #ffffff);
  font-family: var(--font-body, sans-serif);
  font-size: var(--font-size-button, 14px);
  font-weight: var(--font-weight-button, 500);
  padding: 0 var(--space-5, 24px);
  height: var(--size-button-md, 40px);
  border-radius: var(--radius-full, 9999px);
  border: none;
  cursor: pointer;
}
.btn:hover {
  background: var(--color-primary-hover, #333333);
}

/* Card */
.card {
  background: var(--color-card, #ffffff);
  border: 1px solid var(--color-border, #e5e5e5);
  border-radius: var(--radius-md, 12px);
  box-shadow: var(--shadow-1, 0 1px 2px rgba(0,0,0,0.1));
}
.card:hover {
  box-shadow: var(--shadow-2, 0 4px 8px rgba(0,0,0,0.1));
}

/* Container */
.container {
  max-width: var(--max-content, 1200px);
  margin: 0 auto;
  padding: 0 var(--space-5, 24px);
}

/* Nav */
.nav {
  height: var(--size-nav, 64px);
  background: var(--color-surface, #ffffff);
  border-bottom: 1px solid var(--color-border, #e5e5e5);
}
```
