# Endfield Design System

Endfield（终末地）设计系统——面向工业末日废土风格的市场营销与落地页。系统专为高能量、几何切割、危险标识视觉语言构建，服务于品牌的预约转化、世界观展示与角色推广场景。

> 工业末日废土暗色 UI：危险黄主色，霓虹绿与品红次级强调，几何 clip-path 切口，对角危险条纹图案，彩色辉光阴影。基于近黑 #191919 地面与 #ffffff 文字、#fffa00 签名黄。

## Source

- **品牌:** Endfield / 终末地
- **产品类型:** Marketing / Landing
- **Kit 类型:** marketing
- **置信度:** high

## What this design system covers

- **Foundations** — 8 组色阶（每组 9 级）、4 字体族、10 级间距、6 级圆角、5 层辉光阴影
- **Components** — 6 个已文档化组件：button、card、input、badge、cta-link、navigation
- **Preview** — 每组件一张 HTML 预览卡，位于 `preview/` 目录

## CONTENT FUNDAMENTALS

### Voice & tone

Endfield 的语言是双语的——中文为主、英文为辅，两者并置制造工业标识感。语调克制而紧迫，像终端指令而非营销话术：动词短促（"探索""领取""预约"），名词技术化（"ARK牌号""开发组"）。几乎不用感叹号；情绪靠视觉的辉光与切口传达，而非文字修辞。导航与标签一律大写英文（ENDFIELD），正文用中文，形成"代码层 / 人文层"的双轨。UI 内无 emoji。

### Concrete copy examples (lifted from the bundle)

- 品牌名（英文标识）: *"ENDFIELD"*
- 品牌名（中文）: *"终末地"*
- 主导航/行动词: *"探索"*
- 转化行动词: *"领取"*
- 转化行动词: *"预约"*
- 内容分区: *"世界观"*
- 内容分区: *"角色"*
- 页脚归属: *"开发组"*
- 社交引导: *"关注我们"*
- 技术名词: *"ARK牌号"*

### When generating copy

- 行动词保持二字优先（探索、领取、预约），避免"立即点击领取"之类冗余修饰
- 英文标识全部大写（ENDFIELD），与中文并置时英文在前
- 技术名词保留原文不翻译（ARK牌号）
- 正文用中文，导航与 eyebrow 标签用大写英文

## VISUAL FOUNDATIONS

### Color

品牌主色是危险黄 `--endfield-primary-600: #fffa00`，一种高饱和工业警示色，在近黑底上产生最高对比与签名辉光（`--shadow-5: 0 0 10px #fff000`）。主色阶 9 级从 `#ffffcc`（primary-50，近白黄）到 `#737000`（primary-900，暗橄榄），但实际使用集中在 500–600 段——更深的 700+ 用于浅底 badge 的文字色。

次级强调有两支：霓虹绿 `--endfield-accent-600: #00ffa2` 与品红 `--endfield-tertiary-600: #ff1aac`。绿用于状态确认与 card eyebrow 标签；品红用于 error 辉光与 tertiary badge。两者不是品牌色——它们是功能信号色，出现频率远低于主色黄。

中性色 9 级从 `#f2f2f2`（neutral-50）到 `#191919`（neutral-900），后者即默认背景 `--background`。工作中性集中在 neutral-700 `#333333`（边框 `--rule` / `--border`）、neutral-800 `#2e2e2e`（surface / card 底）、neutral-400 `#888888`（muted 文字）。前景纯白 `#ffffff`。

语义色与主色阶同构：success `#00e676`、warning `#ffb300`、error `#ff1a4d`、info `#00d4ff`，各 9 级渐变。Warning 的 600 值 `#ffb300` 比主色黄更橙，用于真正的警示状态而非品牌表达。整体色感：近黑地面上三色荧光——黄是身份、绿是确认、品红是危险。没有暖灰，没有蓝紫，没有渐变背景。辉光是唯一的"深度"手段。

### Typography

四个字体族，各司其职。**Novecentosanswide**（`--font-display`）——标题与导航专用，几何无衬线，字宽偏阔，大写时最具标识感。权重用 Bold（700）与 DemiBold（600）。该字体为品牌授权字体，本地环境通常缺失（见 Caveats）。

**Gilroy**（`--font-body`）——正文与 UI 文字。权重覆盖 Light（400）到 ExtraBold（700），但正文默认 400。Fallback 链已内建 `Segoe UI, Roboto, PingFang SC, Microsoft YaHei`，中文回退到苹方/雅黑。**SpaceGrotesk**（`--font-mono`）——等宽与 eyebrow 标签，用于代码、数据型小字、card-cta 链接，14px / 400。**ProtestStrike**（`--font-impact`）——仅 mega 级标题（130px），冲击型展示字，uppercase。

字阶从 mega 130px / 0.9 行高开始，经 display 64px、h1 48px、h2 40px、h3 32px、h4 24px，到 lead 22px、body 16px、caption 12px。大字一律 uppercase + 负字距（tight `-0.03em` 到 tightest `-0.08em`），制造压缩紧凑的工业排版。正文行高 1.6 保证可读，但标题行高压到 1.05–1.2，近乎贴行。Eyebrow 用 mono 字体 + 0.08em 正字距 + uppercase，是整个系统里唯一放开字距的场景。

### Spacing

间距不遵循 4px 基数，而是一组观察值：`--space-1: 10px` 起步，经 12、16、18、20、24、32、40，到 60 与 80。10px 是最小工作间距（badge 内 padding、card 内 gap），24px 是标准卡片内距，80px 是大节段间距。控件高度独立定义：button sm 32px / md 40px / lg 48px，input 44px，nav 72px。

### Radius

圆角是刻意的工业语言，不是装饰。`--radius-sm: 4px` 用于 input、nav-logo mark 等小控件——棱角分明。`--radius-md: 8px` 极少使用。没有 12px 或 16px 的"卡片圆角"——卡片用 `clip-path` 切角而非圆角。`--radius-lg: 42px` 被别名给 `--radius-pill`，但实际 pill 场景由 `--radius-full: 9999px` 承担（badge 即用 full）。圆角与切角是两套并行的语言：小件用圆角，大件用切角。

### Shadow / Elevation

5 层，但不是传统投影——是辉光。`--shadow-1` 到 `--shadow-4` 都是 `0 0` 偏移的黑色半透明 spread，从 .5rem 到 2rem，营造暗色表面上的微弱浮起。真正的签名是 `--shadow-5: 0 0 10px #fff000`——黄色品牌辉光，专用于 primary button、glow card、input focus 态。其余 colored glow（accent 绿、tertiary 品红）在组件内联定义而非 token 化。无辉光即无深度——rest 态几乎不投影。

### Borders, Backgrounds, Animation, Iconography

边框统一 1px、颜色 `--border: #333333`（neutral-700），在近黑底上几乎不可见但划定边界。Outline button 用 2px 实边。背景没有渐变——唯一图案是 card-media 的对角危险条纹 `repeating-linear-gradient`，4px 透明 + 1px 黄 5% 叠加。动画以 `--transition-fast: .2s ease` 为主，hover 多为 `filter: brightness()` 微调或辉光增强，不位移、不变形。缓动 `--ease-emphasized: cubic-bezier(0.2, 0, 0.13, 1.13)` 带 overshoot，用于强调性交互。图标尺寸 16 / 24 / 32px 三级，与 button 尺寸对应。

## Component Patterns

- **Button** (`preview/component-button.html` · `components.css` §Button) — 尺寸 sm/md/lg（32/40/48px），变体 primary/accent/outline/ghost。`clip-path` 切右下角 8px 是签名特征；primary 带黄色辉光 `--shadow-5`，ghost 去掉切角回归纯文字链接。
- **Card** (`preview/component-card.html` · `components.css` §Card) — 变体 dark/glow/compact。dark 与 glow 均用 12px 切角 `clip-path`；glow 版换黄色辉光 + primary 边框。compact 是唯一用圆角（`--radius-sm`）的变体。card-media 自带危险条纹底纹。
- **Input** (`preview/component-input.html` · `components.css` §Input) — 44px 高，`--radius-sm` 圆角，focus 态亮黄色边框 + `--shadow-5` 辉光；error 态品红边框 + 品红辉光。
- **Badge** (`preview/component-badge.html` · `components.css` §Badge) — pill 形（`--radius-full`），caption 字号 + h4 字重 + uppercase。6 色变体，用浅底深字（如 primary-100 底 + primary-900 字）。
- **Cta Link** (`preview/component-cta-link.html` · `components.css` §Cta Link) — 大写 + 负字距 + 箭头 indicator `→`。变体 default（黄）/accent（绿）/subtle（去大写、降字重，用于次要操作）。
- **Navigation** (`preview/component-navigation.html` · `components.css` §Navigation) — 72px 高，display 字体 + uppercase。logo mark 是 28px 黄色方块（`--radius-sm`）。active 态用 2px 黄色底边线，不用背景填充。

## Index

- `README.md` — 本文件，品牌叙事与视觉基础
- `SKILL.md` — AI 代理技能入口
- `colors_and_type.css` — 所有 token 的 CSS 变量（颜色、字体、间距、圆角、阴影、动画）
- `components.css` — 从预览页提取的组件 CSS
- `css.json` — 结构化 token JSON
- `library-consumption.json` — 下游消费读序
- `preview/` — 6 张组件 HTML 预览卡
- `components/index.json` + `components/{slug}.json` — 组件契约

## Caveats / known substitutions

1. **Novecentosanswide** 为品牌授权字体，本地环境普遍缺失。CSS fallback 仅到 `sans-serif`——如需精确还原则需自备字体文件；否则标题将回退至系统无衬线，字宽与标识感会显著降低。
2. **ProtestStrike** 同为非通用展示字体，仅用于 mega 级（130px）。缺失时回退至 `Gilroy-ExtraBold`，冲击力减弱但字重可保。
3. **Gilroy** 为免费字体但可能未预装。Fallback 链已内建 `Segoe UI → Roboto → PingFang SC → Microsoft YaHei`，跨平台可用但字形细节有差异。
4. **SpaceGrotesk** 为 Google Fonts 免费字体，可在线引入；如离线使用需本地安装。
5. 所有色阶 token 均为 AI 生成（CSS 注释已标注 `/* AI-generated */`），非从设计源文件精确提取——使用时以语义别名为准（`--primary`、`--accent` 等），不要直接硬编码原始 hex。
6. 暗色为默认主题；`.light` 覆盖类存在但未在组件预览中验证——亮色态为推断值。
