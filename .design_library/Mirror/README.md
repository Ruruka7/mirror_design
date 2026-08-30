# Mirror Design System

## Overview

This design library is a research reconstruction of the public Mirror / CONSTRUCT blog at `https://emaostudio.online/`.
It captures the site's black-and-white editorial language for AI-assisted interfaces, technical notes, open-source project indexes, and personal content archives.
The package includes color and typography tokens, spacing and motion decisions, six reusable component contracts, standalone previews, and a content-oriented marketing UI kit.
The source page was inspected as rendered HTML on 2026-08-30; values in this library are observations or clearly marked inferences from that page.

## Content Fundamentals

The voice is personal, concise, and research-oriented. Chinese carries the main narrative, while compact English labels such as `PERSONAL BLOG`, `FEATURED`, `OPEN SOURCE`, `AI NOTES`, and `AUTO-SUMMARIZED` provide an index-like layer.
Observed interface copy includes “AI 叙事引擎 / 技术笔记 / 开源项目”, “开始阅读”, “关于我”, “查看全部”, “更多日常”, “全部项目”, and “持续更新”.
Titles can be reflective or technical, while metadata stays short and factual: dates, reading time, category, and tags.
Prefer direct nouns and short clauses over promotional claims. Keep punctuation restrained, preserve the author's first-person tone in personal sections, and use uppercase English only as a compact section marker or system label.

## Visual Foundations

### Color

The default canvas is `--color-background` at `#ffffff`, with `--color-foreground` and `--color-primary` anchored to `#0a0a0a`. The main secondary ink is `--mirror-secondary-600` at `#404040`, while `--color-text-secondary` resolves to the quieter `#404040` family used for summaries and metadata. `--color-surface` resolves to `#f5f5f5`, and `--color-muted` resolves to `#e5e5e5`, creating the two gray levels used for cards, filters, and hover washes.

The neutral scale moves from `#ffffff` through `#fafafa`, `#f5f5f5`, `#e5e5e5`, `#d4d4d4`, `#a3a3a3`, `#737373`, `#525252`, and `#262626` to `#0a0a0a`. Rules use `--color-border` and `--line-thin`/`--line-heavy`, while the lighter `--color-border-subtle` uses `#d4d4d4`. The stylesheet exposes `#ef4444` and `#dc2626` for destructive states, but no colored success, warning, or info language was visible in the homepage; the corresponding library fallbacks therefore stay monochrome.

### Typography

Display headings use `--font-family-display`, led by `Inter Tight Variable` and falling back through `Space Grotesk Variable`, `Noto Sans SC`, `PingFang SC`, and `Microsoft YaHei`. Body copy uses `--font-family-body`, led by `Inter Variable`; compact dates and technical labels use `--font-family-mono`, led by `JetBrains Mono Variable`.

The observed scale is deliberately sparse: `--font-size-display` is `128px`, `--font-size-h1` is `48px`, `--font-size-h2` is `24px`, `--font-size-h3` is `20px`, `--font-size-body` is `16px`, `--font-size-caption` is `14px`, and `--font-size-mono` is `12px`. Display and H1 weights use `900`, section headings use `700`, body text uses `400`, and captions use `500`. Display line-height is `0.85`, body line-height is `1.5`, and labels use the tighter `1.45` rhythm. Uppercase labels use the positive `--tracking-wider` spacing.

### Spacing and layout

The spacing base is `4px`, exposed as `--space-1`; the working rhythm steps through `8px`, `12px`, `16px`, `24px`, `32px`, `48px`, `64px`, and the observed hero breathing room of `80px`. The homepage combines flex rows for shell and metadata with border-separated grids: three featured article cards, three project cards, and a two-column notes/inspiration block.

The measured navigation height is `--size-nav-height` at `58px`. The utility control is `36px`, the medium action reference is `44px`, and the large reading CTA is `60px`. The reusable input height is `40px` as a documented inference because the homepage itself does not expose a visible text field.

### Radius and elevation

The system is intentionally square. `--radius-none`, `--radius-sm`, `--radius-md`, `--radius-lg`, and `--radius-xl` all resolve to `0px`, matching the page's rectangular cards, buttons, tags, and navigation. `--radius-full` and `--radius-pill` retain `9999px` only for exceptional circular or pill-like affordances.

No meaningful box shadow was observed in the rendered homepage. `--shadow-1` through `--shadow-5` therefore resolve to `none`; separation comes from `--line-thin`, `--line-heavy`, `--line-hair`, surface contrast, and generous whitespace rather than elevation.

### Motion

Motion is restrained and functional. `--transition-fast` uses `0.15s` with `cubic-bezier(0.4, 0, 0.2, 1)` for links, borders, backgrounds, and arrow movement; `--transition-base` uses `0.2s` for utility controls. The `0.3s` smooth token is reserved for longer filter-like transitions. Hover states mostly change background, underline, or border weight instead of introducing translation or glow.

## Component Patterns

The six reusable components are Button for primary and utility actions, Card for article/project/note summaries, Input for archive and filter fields, Badge for categories and metadata, CTA Link for section exits and content discovery, and Navigation for the fixed top shell plus inverse footer. Together they preserve the blog's dense index language: square edges, black-and-white contrast, compact mono metadata, and rule-based grouping.

## Index

- `colors_and_type.css`
- `css.json`
- `metadata.json`
- `components.css`
- `components/index.json`
- `components/{button,card,input,badge,cta-link,navigation}.json`
- `preview/component-{button,card,input,badge,cta-link,navigation}.html`
- `ui_kits/marketing/index.html`
- `ui_kits/marketing/quality-report.json`
- `uikit-plan.json`
- `SKILL.md`
- `library-consumption.json`

## Caveats / Known Substitutions

1. The source site loads variable font faces named `Inter Variable`, `Inter Tight Variable`, `Space Grotesk Variable`, and `JetBrains Mono Variable`; this library names those families first but does not bundle font files.
2. The homepage exposes no visible input field, so the Input contract and its `40px` height are reusable inferences from the shared CSS language rather than direct component measurements.
3. The source stylesheet contains destructive red values `#ef4444` and `#dc2626`, but no rendered success, warning, or info hues were observed; those semantic scales are monochrome fallbacks and should be replaced when a target page provides stronger evidence.
4. Browser viewport override did not produce reliable narrow-width measurements in this session; responsive notes are based on the source stylesheet's `40rem`, `48rem`, and `64rem` media queries plus the desktop DOM.
5. Image files and source-site assets are not copied into this library. The UI Kit uses neutral CSS-only placeholders so the package remains reusable without external dependencies.
