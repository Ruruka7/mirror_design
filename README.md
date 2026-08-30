# Mirror Design

> 从官网自动提取真实视觉规律，让 AI 生成可复用设计系统与页面方案的工作流和模板资产。
>
> An official-site-driven workflow that lets AI extract visual patterns from a website and turn them into reusable design-system assets.

[中文](#中文) · [English](#english)

## 中文

### 这是什么

Mirror Design 的主体不是某一个品牌的设计库，而是一条可复用的 AI 设计工作流：

> 输入一个允许研究和使用的官网地址 → AI 提取页面证据 → 生成设计系统 → 产出可直接复用的页面与组件资产。

仓库里的 OpenAI、Voith、Endfield 目录是这条工作流产出的模板样例，用来展示最终资产长什么样；真正可复用的核心是：

- `skills/reverse-design-system/`：从官网生成设计资产包的 AI 工作流
- `file-specs/`：token、组件契约、预览页、UI Kit 和质量报告的结构规范
- `operation-policies/`：证据优先、决策、质量门禁和 Git 交付规则
- `.design_library/`：可被后续页面项目直接消费的模板化输出

### 核心流程

| 阶段 | AI 自动完成的工作 | 主要产出 |
| --- | --- | --- |
| 1. Extract | 打开目标官网，读取渲染后的 computed styles、CSS、DOM 组件、响应式状态和可用资产信息 | 页面证据、截图、样式与组件观察结果 |
| 2. Analyze | 从证据整理品牌画像、语气、色彩、字体、间距、圆角、阴影和组件规律，并标注未知项 | 品牌画像、关键发现、决策依据 |
| 3. Generate | 生成 token CSS/JSON、组件契约、HTML 预览、Marketing UI Kit、文档和消费顺序 | 可复用设计系统资产包 |
| 4. Validate | 检查 BOM、JSON、CSS 变量、组件覆盖率、相对路径和实际渲染结果 | 可提交、可交接、可继续迭代的输出 |

核心原则是 evidence-first：能从目标页面实际读取的值优先使用，无法确认的内容明确标记为推断，不用想象值冒充官网事实。

### 可复用资产包

一条工作流不只生成一张页面，而是生成一套可以被 AI、设计师和前端项目反复消费的资产：

| 资产 | 用途 |
| --- | --- |
| `colors_and_type.css` | 页面运行时直接加载的 CSS token |
| `css.json` | 供 AI 或工具理解的结构化 token |
| `components/*.json` | 组件意图、变体、结构和使用边界 |
| `preview/*.html` | 单组件视觉与交互参考 |
| `ui_kits/marketing/index.html` | 组合后的完整页面方案 |
| `README.md` / `SKILL.md` | 品牌语气、设计原则和 AI 使用入口 |
| `quality-report.json` | 组件覆盖率、交互状态和已知警告 |

因此，换一个官网地址，就可以得到另一套风格不同、结构统一、可继续消费的设计资产包。

### 当前模板样例

这些目录是已经整理好的输出模板。预览页面中的品牌名、产品名和示例数据已替换为中性内容；目录名与研究文档中的来源标签仅用于资料索引。

| 模板 | 风格方向 | 入口 |
| --- | --- | --- |
| OpenAI | 极简白底、黑色主行动、克制的产品型排版 | [设计库文档](./.design_library/OpenAI/README.md) · [Marketing UI Kit](./.design_library/OpenAI/ui_kits/marketing/index.html) |
| Voith | 工业工程、海洋蓝与青色强调、机构化信息层级 | [设计库文档](./.design_library/Voith/README.md) · [Marketing UI Kit](./.design_library/Voith/ui_kits/marketing/index.html) |
| Endfield | 近黑底、危险黄、霓虹强调、几何切角与工业警示纹理 | [设计库文档](./.design_library/Endfield/README.md) · [Marketing UI Kit](./.design_library/Endfield/ui_kits/marketing/index.html) |

每套模板当前包含 6 个核心组件：`button`、`card`、`input`、`badge`、`cta-link`、`navigation`。

### 如何让 AI 使用这套工作流

在支持浏览器自动化、文件写入和设计资产生成的 AI Agent 环境中，将官网 URL 和本仓库的工作流入口一起提供：

```text
请使用 skills/reverse-design-system/SKILL.md。
根据以下官网 URL 生成一套可复用的设计系统资产：
<official-site-url>

要求：
1. 以官网实际渲染结果和可读取 CSS 为主要证据；
2. 完成 Extract → Analyze → Generate → Validate 全流程；
3. 输出 token、组件契约、HTML 预览、Marketing UI Kit、文档和质量报告；
4. 无法确认的值标记为推断，不要伪造官网事实；
5. 只使用我有权研究和使用的页面、文案与资产。
```

完整的阶段说明、输入输出和门禁规则见 [reverse-design-system 工作流](./skills/reverse-design-system/SKILL.md)。

### 目录结构

```text
mirror_design/
├── skills/
│   └── reverse-design-system/
│       ├── SKILL.md
│       ├── workflows/
│       ├── file-specs/
│       └── operation-policies/
├── .design_library/
│   ├── OpenAI/              # 模板样例
│   ├── Voith/               # 模板样例
│   └── Endfield/            # 模板样例
├── LICENSE
└── README.md
```

单套设计库的通用输出结构：

```text
<library>/
├── README.md
├── SKILL.md
├── colors_and_type.css
├── css.json
├── components.css
├── components/{slug}.json
├── components/index.json
├── preview/component-{slug}.html
├── ui_kits/marketing/index.html
├── library-consumption.json
├── uikit-plan.json
└── ui_kits/marketing/quality-report.json
```

### 运行前提与边界

- 目标官网需要可访问；登录、验证码、地区限制或强反爬页面可能需要用户提供替代页面或截图。
- 完整自动化执行需要宿主 AI Agent 提供浏览器自动化、文件读写和相应生成运行时；本仓库本身没有 `package.json` 或必须执行的构建脚本。
- 页面预览可以直接打开，或用任意静态文件服务器托管仓库根目录。
- 这是设计语言的研究性重建，不保证像素级一致；每套库的置信度、测量日期、推断值和已知警告以对应文档为准。
- 生产使用前，请自行确认目标网站的访问权限、字体/图标/图片许可、文案使用权和品牌规范。

### 许可证与第三方权利

本仓库中由作者编写、且不属于第三方内容的代码、文档、JSON schema、CSS、HTML 和工作流规范，除另有说明外，按 [MIT License](./LICENSE) 发布。

OpenAI、Voith、Endfield/终末地等名称、商标、标识、原网站文案、字体、图标、视觉资产，以及文档中引用的外部资源，仍归各自权利人所有；MIT License 不授予这些第三方内容的商标权、品牌授权或再分发权。本项目与相关品牌没有隶属、赞助或官方合作关系。

使用这套工作流时，请只处理你有权研究、改编和使用的官网及资产，并对最终生成物进行人工版权与商标审查。

## English

### What this is

Mirror Design is an official-site-driven AI workflow for turning a permitted website URL into a reusable design-system asset package. The current OpenAI, Voith, and Endfield directories are example templates produced by that workflow; they are not the core product.

The reusable core is:

- `skills/reverse-design-system/` — the end-to-end workflow for extraction, analysis, generation, and validation
- `file-specs/` — reusable schemas for tokens, components, previews, UI kits, and quality reports
- `operation-policies/` — evidence, decision, quality-gate, and Git delivery rules
- `.design_library/` — template-style output packages that downstream projects can consume

### Workflow

1. **Extract** rendered styles, CSS, DOM component patterns, responsive states, and asset metadata from the target site.
2. **Analyze** the evidence into a brand profile, content tone, visual foundations, component patterns, and explicit unknowns.
3. **Generate** CSS/JSON tokens, component contracts, HTML previews, a marketing UI kit, documentation, and a consumption guide.
4. **Validate** BOM, JSON, CSS variables, component coverage, paths, and rendered output.

The workflow is evidence-first: confirmed values come from the target page, while inferred values are labeled instead of being presented as observed facts.

### Reuse

Give a permitted official-site URL to an AI Agent that can browse, read/write files, and run the required generation runtime. The result is a reusable package containing tokens, machine-readable contracts, previews, a complete UI kit, documentation, and quality metadata—not just a one-off page.

The repository has no `package.json` or required build step. See [the workflow entry point](./skills/reverse-design-system/SKILL.md) for the full protocol and [the template libraries](./.design_library/) for examples.

### Legal boundary

The MIT License applies only to original repository materials that the author is able to license. Brand names, trademarks, logos, source-site copy, fonts, icons, visual assets, and external resources remain the property of their respective owners. The project is unofficial and does not imply affiliation, endorsement, or permission to use third-party brands in production.

Preview pages intentionally use neutral placeholder copy and sample data. Use the workflow only with source pages and assets you are authorized to research and use. See [LICENSE](./LICENSE) for the full license text.
