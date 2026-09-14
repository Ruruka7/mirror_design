# Mirror Design

[English](#english) | [中文](#中文)

***

## English

A personal collection of reverse-engineered design systems, built for reference, learning, and backup.

Each library includes two portable assets: **design tokens** (`colors_and_type.css`) and a **complete site template** (`site-template.html`). All visual values in templates use `var(--name, fallback)` syntax — swap one CSS link to instantly restyle an entire site.

### Design Systems

| Library                                 | Source                                                                              | Style                                                                         |
| --------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [OpenAI](./.design_library/OpenAI/)     | [openai.com](https://openai.com/zh-Hans-CN/)                                        | Minimal white-canvas with single green accent                                 |
| [Voith](./.design_library/Voith/)       | [voith.com](https://www.voith.com/corp-en/about-us/markets-locations/china-cn.html) | Industrial marine-blue corporate engineering                                  |
| [Endfield](./.design_library/Endfield/) | [endfield.hypergryph.com](https://endfield.hypergryph.com/)                         | Industrial post-apocalyptic dark UI with triple-accent (yellow/green/magenta) |

### Each Library Includes

**Design Tokens** — extracted from real CSS

* `colors_and_type.css` — all visual values (colors, fonts, spacing, shadows, radius)
* `css.json` — token JSON (derived from CSS)
* `components/` — component contracts (JSON)
* `preview/` — component HTML previews
* `ui_kits/marketing/` — marketing UI kit

**Site Template** — complete page, 1:1 from source

* `site-template.html` — full standalone website page, all sections in one file, zero hardcoded colors

**Combinations** — cross-brand layout + token swaps

* `combinations/` — ready-to-open HTML files mixing different layouts with different token sets

### Skill / Workflow

| Skill | What it does | Output |
|-------|-------------|--------|
| `reverse-to-site` | Full pipeline: browser extract → tokens → layout → site template → combination | `colors_and_type.css` + `site-template.html` |

### Structure

```
mirror_design/
  .design_library/
    OpenAI/
      colors_and_type.css          # Design tokens
      site-template.html           # Complete site template (all CSS vars)
      components/                   # Component contracts
      preview/                      # Component previews
      ui_kits/marketing/            # Marketing UI Kit
    Voith/                          # same structure
    Endfield/                       # same structure
    combinations/                   # Cross-brand demos
      openai-layout_endfield-tokens/
      endfield-layout_openai-tokens/
      voith-layout_endfield-tokens/
  skills/
    reverse-to-site/
      SKILL.md                      # Entry point
      workflows/
        full-pipeline.md            # Complete A-to-Z workflow
      specs/
        token-contract.md           # Portable variable contract
        template-contract.md        # Site template rules
        combination-guide.md        # How to create combinations
      reference/
        variable-reference.md       # Complete variable lookup table
```

***

## 中文

个人收藏的逆向设计系统集合，用于参考、学习和备份。

每个系统包含两类可移植资产：**设计令牌**（`colors_and_type.css`）和**完整站点模板**（`site-template.html`）。模板中所有视觉值使用 `var(--name, fallback)` 语法 — 换一行 CSS 链接即可瞬间改变整站风格。

### 设计系统

| 设计系统                                    | 来源                                                                                  | 风格                      |
| --------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------- |
| [OpenAI](./.design_library/OpenAI/)     | [openai.com](https://openai.com/zh-Hans-CN/)                                        | 极简白底配单一绿色强调             |
| [Voith](./.design_library/Voith/)       | [voith.com](https://www.voith.com/corp-en/about-us/markets-locations/china-cn.html) | 工业海洋蓝企业风格               |
| [Endfield](./.design_library/Endfield/) | [endfield.hypergryph.com](https://endfield.hypergryph.com/)                         | 工业废土暗色 UI，三重强调色（黄/绿/品红） |

### 每个系统包含

**设计令牌** — 从真实 CSS 提取

* `colors_and_type.css` — 所有视觉值（颜色、字体、间距、阴影、圆角）
* `css.json` — 令牌 JSON（从 CSS 派生）
* `components/` — 组件契约（JSON）
* `preview/` — 组件 HTML 预览
* `ui_kits/marketing/` — Marketing UI Kit

**站点模板** — 完整页面，1:1 还原源站

* `site-template.html` — 完整独立网站页面，所有区块在一个文件内，零硬编码颜色

**组合演示** — 跨品牌布局 + 令牌互换

* `combinations/` — 可直接打开的 HTML 文件，混合不同布局与不同令牌

### 技能 / 工作流

| 技能 | 功能 | 产出 |
|------|------|------|
| `reverse-to-site` | 完整管线：浏览器提取 → 令牌 → 布局 → 站点模板 → 组合 | `colors_and_type.css` + `site-template.html` |

### 目录结构

```
mirror_design/
  .design_library/
    OpenAI/
      colors_and_type.css          # 设计令牌
      site-template.html           # 完整站点模板（全部 CSS 变量）
      components/                   # 组件契约
      preview/                      # 组件预览
      ui_kits/marketing/            # Marketing UI Kit
    Voith/                          # 同构
    Endfield/                       # 同构
    combinations/                   # 跨品牌组合演示
      openai-layout_endfield-tokens/
      endfield-layout_openai-tokens/
      voith-layout_endfield-tokens/
  skills/
    reverse-to-site/
      SKILL.md                      # 入口
      workflows/
        full-pipeline.md            # 完整 A-to-Z 工作流
      specs/
        token-contract.md           # 可移植变量契约
        template-contract.md        # 站点模板规则
        combination-guide.md        # 组合创建指南
      reference/
        variable-reference.md       # 完整变量查找表
```

***

## License / 许可证

MIT
