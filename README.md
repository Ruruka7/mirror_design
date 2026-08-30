# Mirror Design

> 面向学习、原型设计和 AI 辅助界面生成的逆向设计系统资料库。
>
> A collection of reverse-engineered design-system references and agent-friendly UI kits for study, prototyping, and design-system workflows.

[中文](#中文) · [English](#english)

## 中文

### 项目简介

Mirror Design 将公开网站和品牌的视觉特征整理成可阅读、可预览、可被工具消费的设计系统资料。每套资料库都尽量同时提供：

- 视觉基础：颜色、字体、间距、圆角、阴影和动效 token
- 组件契约：组件意图、结构、变体和使用边界
- HTML 预览：可直接打开的组件示例
- Marketing UI Kit：一套完整的营销落地页示例
- 机器可读文件：JSON token、组件索引、消费顺序和质量报告
- 品牌文档：语气、文案、视觉基础、已知替代方案和局限

这是一个研究与原型项目，不是任何品牌的官方设计系统，也不代表相关品牌的认可或授权。

### 目前包含的设计库

| 设计库 | 视觉方向 | 入口 |
| --- | --- | --- |
| OpenAI | 极简白底、黑色主行动、克制的产品型排版 | [文档](./.design_library/OpenAI/README.md) · [Marketing UI Kit](./.design_library/OpenAI/ui_kits/marketing/index.html) |
| Voith | 工业工程、海洋蓝与青色强调、机构化信息层级 | [文档](./.design_library/Voith/README.md) · [Marketing UI Kit](./.design_library/Voith/ui_kits/marketing/index.html) |
| Endfield | 近黑底、危险黄、霓虹强调、几何切角与工业警示纹理 | [文档](./.design_library/Endfield/README.md) · [Marketing UI Kit](./.design_library/Endfield/ui_kits/marketing/index.html) |

每套设计库当前记录 6 个核心组件：`button`、`card`、`input`、`badge`、`cta-link`、`navigation`。

### 快速开始

```bash
git clone https://github.com/Ruruka7/mirror_design.git
cd mirror_design
```

本项目不依赖包管理器，也没有必须执行的构建步骤。可以直接用浏览器打开 `preview/` 下的 HTML；如果需要更稳定地加载相对路径和外部字体，可以用任意静态文件服务器托管仓库根目录。

常用入口：

- [OpenAI 组件预览](./.design_library/OpenAI/preview/)
- [Voith 组件预览](./.design_library/Voith/preview/)
- [Endfield 组件预览](./.design_library/Endfield/preview/)
- [逆向设计系统工作流](./skills/reverse-design-system/SKILL.md)

### 如何消费一套设计库

推荐按照以下顺序阅读：

1. `README.md`：先了解品牌语气、视觉意图和已知局限
2. `SKILL.md`：查看面向 AI 代理的快速规则
3. `css.json`：读取结构化 token 语义
4. `colors_and_type.css`：在 HTML 或原型中加载运行时 CSS 变量
5. `components/index.json`：查看组件清单与跨组件规律
6. `components/{slug}.json`：读取具体组件契约
7. `preview/component-{slug}.html`：以渲染结果检查组件形态
8. `ui_kits/marketing/index.html`：查看组合后的营销页面示例

其中，`css.json` 适合程序或代理理解 token，`colors_and_type.css` 适合实际链接到页面中；组件预览优先于文字描述，用于确认最终的 DOM 与视觉表现。

### 目录结构

```text
mirror_design/
├── .design_library/
│   ├── OpenAI/
│   ├── Voith/
│   └── Endfield/
├── skills/
│   └── reverse-design-system/
├── LICENSE
└── README.md
```

单套设计库的常用文件：

```text
<library>/
├── README.md                         # 品牌背景、语气、视觉基础和局限
├── SKILL.md                          # AI 代理入口与快速规则
├── colors_and_type.css               # 运行时 CSS token
├── css.json                          # 结构化 token
├── components.css                    # 汇总后的组件 CSS
├── components/{slug}.json            # 组件契约
├── components/index.json             # 组件索引
├── preview/component-{slug}.html     # 单组件 HTML 预览
├── ui_kits/marketing/index.html      # 营销 UI Kit
├── library-consumption.json          # 推荐消费顺序
├── uikit-plan.json                   # UI Kit 组合计划
└── ui_kits/marketing/quality-report.json
```

### `reverse-design-system` 工作流

`skills/reverse-design-system/` 是一套独立的工作流规范，覆盖：

- 浏览器与 CSS 信息提取
- 品牌画像和视觉 token 生成
- 组件识别、契约定义和 HTML 预览
- 文档与 Marketing UI Kit 生成
- BOM、JSON、CSS 变量、组件覆盖率和质量报告检查
- Git 提交、清理和交付约定

它更接近可复用的研究方法和代理规则，而不是一个已经发布到包管理器的开发工具。

### 已知局限与来源说明

- 所有设计库都是基于公开页面观察、结构化整理和部分推断的重建版本，不保证像素级一致。
- HTML 预览和 Marketing UI Kit 已使用中性占位文案与示例数据；目录名和研究文档中的来源标签仅用于资料索引。
- 各库文档中的置信度、测量日期、替代方案和 caveats 以对应目录为准；部分色阶或暗色主题明确标记为推断或 AI 生成。
- 仓库不包含 OpenAI Sans、Novecentosanswide、ProtestStrike 等可能受许可限制的字体文件；部分预览会使用系统回退或外部字体/CDN。
- Voith 的质量报告保留了 `menu` 和 `table` 固定插槽缺失警告。
- 在生产项目中使用前，请自行核对最新品牌规范、字体许可、图标许可、外部 CDN 条款和相关网站的使用条款。

### 贡献

欢迎提交问题和 Pull Request。新增设计库时，建议至少同时提供品牌 README、`SKILL.md`、token 文件、组件契约、HTML 预览、Marketing UI Kit 和质量报告，并确保相对路径与 JSON 格式可用。

### 许可证与第三方权利

本仓库中由作者编写、且不属于第三方内容的代码、文档、JSON schema、CSS、HTML 和工作流规范，除另有说明外，按 [MIT License](./LICENSE) 发布。

OpenAI、Voith、Endfield/终末地等名称、商标、标识、原网站文案、字体、图标、视觉资产，以及 README 或预览中引用的外部资源，仍归各自权利人所有；MIT License 不授予这些第三方内容的商标权、品牌授权或再分发权。本项目与相关品牌没有隶属、赞助或官方合作关系。

## English

Mirror Design is a research and prototyping repository that turns observations from public websites and brands into structured design-system references. Each library combines design tokens, component contracts, standalone HTML previews, a marketing UI kit, machine-readable JSON, and usage documentation for AI-assisted interface work.

The repository currently includes:

- **OpenAI** — minimal white canvas, black primary actions, restrained product typography
- **Voith** — industrial engineering language, marine blue, cyan accents, institutional hierarchy
- **Endfield** — near-black surfaces, hazard yellow, neon accents, angular cut-outs and warning patterns

Every library documents six core components: `button`, `card`, `input`, `badge`, `cta-link`, and `navigation`. Start with the library `README.md`, then read `SKILL.md`, `css.json`, the runtime CSS, component contracts, previews, and finally the marketing UI kit.

The repository has no package-manager dependency or required build step. Preview HTML files can be opened directly or served from the repository root with any static file server.

Preview HTML and marketing UI kits intentionally use neutral placeholder copy and sample data. Library names and source labels remain only as research indexes.

`skills/reverse-design-system/` contains the reusable workflow for browser/CSS extraction, brand analysis, token generation, component contracts, documentation, UI kits, quality gates, and Git delivery.

### Important legal boundary

The MIT License applies only to the original repository materials that the author is able to license. Brand names, trademarks, logos, source-site copy, fonts, icons, visual assets, and external resources remain the property of their respective owners. This project is unofficial and does not imply affiliation, endorsement, or permission to use third-party brands in production.

See [LICENSE](./LICENSE) for the full license text and each library's README for provenance and caveats.
