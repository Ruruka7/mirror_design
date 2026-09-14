# Mirror Design

[English](#english) | [中文](#中文)

***

## English

A personal collection of reverse-engineered and reconstructed design systems, built for reference, learning, and backup.

Each library includes both **design tokens** (colors, fonts, spacing, components) and **layout templates** (page sections, UX flows, interaction patterns). Combine any token system with any layout system to generate a new website.

### Design Systems

| Library                                 | Source                                                                              | Style                                                                         |
| --------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [OpenAI](./.design_library/OpenAI/)     | [openai.com](https://openai.com/zh-Hans-CN/)                                        | Minimal white-canvas with single green accent                                 |
| [Voith](./.design_library/Voith/)       | [voith.com](https://www.voith.com/corp-en/about-us/markets-locations/china-cn.html) | Industrial marine-blue corporate engineering                                  |
| [Endfield](./.design_library/Endfield/) | [endfield.hypergryph.com](https://endfield.hypergryph.com/)                         | Industrial post-apocalyptic dark UI with triple-accent (yellow/green/magenta) |

### Each Library Includes

**Design Tokens** — extracted from real CSS

* Token system (`colors_and_type.css` + `css.json`)
* Component contracts (`components/*.json`)
* Component previews (`preview/*.html`)
* Marketing UI Kit (`ui_kits/marketing/index.html`)

**Layout Templates** — extracted from rendered DOM

* Page section map (`layouts/page-sections.json`)
* Layout system config (`layouts/layout-system.json`)
* Reusable HTML section templates (`layouts/section-*.html`) — all use CSS variables, no hardcoded styling
* Combined preview (`layouts/preview.html`)
* UX user journey (`ux/user-journey.json`)
* Interaction patterns (`ux/interaction-patterns.json`)

### Skills / Workflows

| Skill | Extracts | Output |
|-------|----------|--------|
| `reverse-design-system` | Colors, fonts, spacing, shadows, components | Token system + component contracts |
| `reverse-page-layout` | Page sections, layout grids, UX flows, interactions | HTML layout templates + UX JSON |

### Structure

```
mirror_design/
  .design_library/
    OpenAI/
      colors_and_type.css          # Tokens
      components/                   # Component contracts
      preview/                      # Component previews
      ui_kits/marketing/            # Marketing UI Kit
      layouts/                      # Page layout templates (CSS vars only)
        section-nav.html
        section-hero-centered.html
        section-feature-grid.html
        ...
        preview.html                # Combined preview
      ux/                           # UX flows
        user-journey.json
        interaction-patterns.json
    Voith/                          # same structure
    Endfield/                       # same structure
  skills/
    reverse-design-system/          # Token extraction workflow
    reverse-page-layout/            # Layout extraction workflow
```

***

## 中文

个人收藏的逆向工程与重建设计系统集合，用于参考、学习和备份。

每个系统同时包含**设计令牌**（颜色、字体、间距、组件）和**布局模板**（页面分区、UX 流程、交互模式）。任意令牌系统 + 任意布局系统 = 一个新网站。

### 设计系统

| 设计系统                                    | 来源                                                                                  | 风格                      |
| --------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------- |
| [OpenAI](./.design_library/OpenAI/)     | [openai.com](https://openai.com/zh-Hans-CN/)                                        | 极简白底配单一绿色强调             |
| [Voith](./.design_library/Voith/)       | [voith.com](https://www.voith.com/corp-en/about-us/markets-locations/china-cn.html) | 工业海洋蓝企业风格               |
| [Endfield](./.design_library/Endfield/) | [endfield.hypergryph.com](https://endfield.hypergryph.com/)                         | 工业废土暗色 UI，三重强调色（黄/绿/品红） |

### 每个系统包含

**设计令牌** — 从真实 CSS 提取

* 令牌系统（`colors_and_type.css` + `css.json`）
* 组件契约（`components/*.json`）
* 组件预览（`preview/*.html`）
* Marketing UI Kit（`ui_kits/marketing/index.html`）

**布局模板** — 从渲染后 DOM 提取

* 页面分区结构（`layouts/page-sections.json`）
* 布局系统配置（`layouts/layout-system.json`）
* 可复用 HTML 区块模板（`layouts/section-*.html`）— 全部使用 CSS 变量，无硬编码样式
* 合并预览（`layouts/preview.html`）
* UX 用户旅程（`ux/user-journey.json`）
* 交互模式（`ux/interaction-patterns.json`）

### 技能 / 工作流

| 技能 | 提取内容 | 产出 |
|------|---------|------|
| `reverse-design-system` | 颜色、字体、间距、阴影、组件 | 令牌系统 + 组件契约 |
| `reverse-page-layout` | 页面分区、布局网格、UX 流程、交互 | HTML 布局模板 + UX JSON |

### 目录结构

```
mirror_design/
  .design_library/
    OpenAI/
      colors_and_type.css          # 令牌
      components/                   # 组件契约
      preview/                      # 组件预览
      ui_kits/marketing/            # Marketing UI Kit
      layouts/                      # 页面布局模板（仅 CSS 变量）
        section-nav.html
        section-hero-centered.html
        section-feature-grid.html
        ...
        preview.html                # 合并预览
      ux/                           # UX 流程
        user-journey.json
        interaction-patterns.json
    Voith/                          # 同构
    Endfield/                       # 同构
  skills/
    reverse-design-system/          # 令牌提取工作流
    reverse-page-layout/            # 布局提取工作流
```

***

## License / 许可证

MIT
