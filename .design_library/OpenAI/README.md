# OpenAI Design System

A design system reconstruction of **OpenAI** — the public-facing marketing and product experience for AI research, APIs, and consumer products.
The system is purpose-built for high-trust landing pages and editorial product surfaces where research credibility meets minimalist product storytelling.

> *"有什么可以帮忙的？"*

## Source

- **Official page:** `https://openai.com/zh-Hans-CN/`
- **Measured date:** 2026-08-29
- **Page title:** OpenAI | Research & Deployment
- **Language:** 中文简体
- **Brand owner:** OpenAI

## What this design system covers

- **Foundations** — color (black-on-white with reserved green accent), typography (measured scale), spacing (4px base), radius, shadow
- **Components** — 6 core components: Button, Navigation, Card, Badge, CTA Link, Input
- **Sample kit** — Marketing landing UI kit (`ui_kits/marketing/index.html`)

---

## CONTENT FUNDAMENTALS

### Voice & tone

OpenAI's voice is research-led and editorially restrained: Chinese-first, formal but accessible, and deliberately sparse. Sentences are short. Product labels avoid hype and favor verbs of action and discovery. There is no emoji, no exclamation marks, and no conversational filler. The tone treats the user as a peer — someone who is either building with AI or trying to understand it.

### Concrete copy examples (lifted from the source)

- Hero prompt: *"有什么可以帮忙的？"*
- Input placeholder: *"给 ChatGPT 发送消息"*
- Hero pills: *"了解 ChatGPT Business"*, *"与 ChatGPT 对话"*, *"研究"*, *"API 平台"*, *"更多"*
- Primary CTA: *"试用 ChatGPT"*
- Secondary CTA: *"登录"*
- Section titles: *"最新动态"*, *"客户案例"*, *"最新研究"*, *"OpenAI 企业解决方案"*, *"开始使用 ChatGPT"*
- Link labels: *"查看更多"*, *"查看全部"*, *"下载"*
- Brand statement: *"通往 AGI 之路的开创性研究"*

### When generating copy

- Prefer verb-first CTAs over noun labels: "开始使用" not "使用入口".
- Keep navigation labels to 2–4 characters where possible.
- Avoid emoji, decorative punctuation, or sales language.
- Match the bilingual engineering context: Chinese UI labels, Latin reserved for product names and code.

---

## Visual Foundations

### Color

The current OpenAI homepage is built on near-pure contrast: **#000000** text on a **#ffffff** ground. The primary interactive surface is **black** (`#000000`) with **white** text; the secondary surface is a near-transparent black at **rgba(0, 0, 0, 0.04)** with black text. The legacy brand green **#10a37f** is retained only as `--brand-green` for reserved accent use — the current homepage almost never uses it as a primary color.

- **Background:** `#ffffff`
- **Foreground:** `#000000`
- **Muted foreground:** `rgba(0, 0, 0, 0.6)`
- **Primary button:** `#000000` background + `#ffffff` text
- **Secondary button:** `rgba(0, 0, 0, 0.04)` background + `#000000` text
- **Border:** `#e5e5e5` / `rgba(0, 0, 0, 0.08)`
- **Reserved accent:** `#10a37f` (`--brand-green`)

Semantic colors are conventional but desaturated: success `#16a34a`, warning `#d97706`, error `#dc2626`, info `#475569`. The overall vibe is gallery-white restraint; it signals confidence through absence rather than ornament.

### Typography

The primary face is the custom **OpenAI Sans** stack — `"OpenAI Sans SC", "OpenAI Sans", "OpenAI Sans Variable Scripts", sans-serif` — used for display, heading, and body. **Source Code Pro** handles mono contexts such as API references or inline code.

Measured scale (2026-08-29):

| Token | Size | Line-height | Weight | Letter-spacing |
|---|---|---|---|---|
| Display / H1 | 28px | 34px | 600 | 0.3px |
| H2 | 20.78px | 25.44px | 500 | -0.17px |
| H3 / card title | 16.78px | 24px | 500 | -0.17px |
| H4 | 16px | 24px | 500 | -0.17px |
| Body | 17px | 28px | 400 | -0.17px |
| Lead | 17px | 28px | 400 | -0.17px |
| Caption / meta / pill | 13px | 20px | 500 | — |
| Button / CTA | 14px | — | 500 | — |
| Mono | 14px | 20px | 400 | — |

The scale is compact and product-first: display text is 28px rather than oversized editorial, body is 17px for readable Chinese, and metadata is 13px/500. Headings use medium weight (500–600) rather than heavy bold.

### Spacing

The base unit is **4px**, producing tokens of 4, 8, 12, 16, 24, 32, 48, and 64px. Component heights land on multiples of 4 (button sm 32px, md 40px, lg 48px; input 40px; navigation 68px). Max content width is capped at **1200px**.

### Radius

- **8px** — small controls and compact containers.
- **12px** — standard cards and medium surfaces.
- **16px** — large panels or prominent containers.
- **24px** — input containers.
- **40px** — primary and secondary buttons.
- **9999px** — pills and badges only.

The radius ladder is soft but controlled: buttons are fully pill-shaped, input containers are large rounded shells, and cards sit at 12–16px.

### Shadow / Elevation

The only prominent measured shadow belongs to the **input container**:

`0 3px 6px rgba(0, 0, 0, 0.04), 0 4px 80px 8px rgba(0, 0, 0, 0.04), 0 0 1px rgba(0, 0, 0, 0.62)`

Cards carry almost no shadow. Additional elevation tokens are reserved for hover/float/modal/overlay layers:

1. **Card** — `0 1px 2px rgba(0, 0, 0, 0.06), 0 1px 1px rgba(0, 0, 0, 0.04)`
2. **Card Hover** — `0 4px 8px -2px rgba(15, 15, 15, 0.10)`
3. **Float** — `0 8px 24px -8px rgba(15, 15, 15, 0.18)`
4. **Modal** — `0 16px 40px -12px rgba(15, 15, 15, 0.24)`
5. **Overlay** — `0 24px 60px -20px rgba(15, 15, 15, 0.30)`

A `.dark` theme inverts the palette and strengthens shadow opacity for OLED surfaces.

### Borders and backgrounds

- Borders are universally **1px solid #e5e5e5**, used to separate cards and navigation bars without visual weight.
- Backgrounds stay in the white-to-off-white range: `#ffffff` for canvas, `#fafafa` for surface, `#f5f5f5` for muted fills. Dark mode swaps to `#0f0f0f` canvas and `#171717` surfaces.
- No gradients are used in the core system; color interest comes from pure black/white contrast.

---

## Component Patterns

| Component | File | Key Insight |
|---|---|---|
| Button | `preview/component-button.html` | Primary is **black fill with white text**, 40px pill radius; secondary uses `rgba(0,0,0,0.04)` fill with black text. |
| Navigation | `preview/component-navigation.html` | 68px transparent bar with OpenAI logo, sparse centered links, search, login, and "试用 ChatGPT" CTA. |
| Card | `preview/component-card.html` | Image on top, title and meta below, white background, 12–16px radius, almost no shadow. |
| Badge | `preview/component-badge.html` | 9999px pill, 13px caption, light gray fill; used for categorization and status. |
| CTA Link | `preview/component-cta-link.html` | Text link with trailing arrow; primary or muted, no underline at rest. |
| Input | `preview/component-input.html` | 24px rounded container with layered shadow, wrapping a borderless input; placeholder uses muted foreground. |

---

## Index

- `README.md` — this file
- `SKILL.md` — agent skill manifest
- `colors_and_type.css` — color, type, radius, shadow, and spacing tokens (measured 2026-08-29)
- `components.css` — aggregated component CSS extracted from previews
- `css.json` — structured token representation
- `components/index.json` — component index and cross-component patterns
- `preview/` — small HTML component previews
- `ui_kits/marketing/` — interactive marketing landing kit
- `library-consumption.json` — recommended downstream read order

---

## Caveats / known substitutions

1. **OpenAI Sans** is a proprietary typeface and is not publicly distributed. The token stack uses `"OpenAI Sans SC", "OpenAI Sans", "OpenAI Sans Variable Scripts", sans-serif` as the intended face, with **Inter** loaded from Google Fonts as a practical fallback. For code contexts, **Source Code Pro** is loaded from Google Fonts.
2. Icons in preview pages reference `lucide-static` via CDN as a placeholder. OpenAI's actual iconography is custom and should be replaced with exported SVGs before production use.
3. The dark mode token block is included in `colors_and_type.css`, but the marketing UI kit is light-first; dark-mode components were inferred rather than observed from the source pages.
4. Fixed slots for **menu** and **table** are missing from the source bundle; the library covers the six observed marketing components only.
5. Measured values are taken from a single homepage snapshot (2026-08-29). Sub-pages or future redesigns may shift radius, shadow, or type sizes.
