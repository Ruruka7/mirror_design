# 组合演示 / Combinations

## 概念

每个品牌的资产分为两类：

1. **设计令牌** (`colors_and_type.css`) — 颜色、字体、间距、阴影、圆角
2. **站点模板** (`site-template.html`) — 完整的网站页面，1:1 还原官网结构

站点模板中所有视觉值都使用 `var(--name, fallback)` 语法引用 CSS 变量。
换一行 `<link>` 链接，整个网站就变成完全不同的视觉风格。

## 可用资产

| 品牌 | 令牌文件 | 站点模板 |
|------|---------|---------|
| OpenAI | `OpenAI/colors_and_type.css` | `OpenAI/site-template.html` |
| Endfield | `Endfield/colors_and_type.css` | `Endfield/site-template.html` |
| Voith | `Voith/colors_and_type.css` | `Voith/site-template.html` |

## 组合矩阵

| 布局 ↓ \ 令牌 → | OpenAI | Endfield | Voith |
|------------------|--------|----------|-------|
| OpenAI 布局 | 原始 | `openai-layout_endfield-tokens/` | — |
| Endfield 布局 | `endfield-layout_openai-tokens/` | 原始 | — |
| Voith 布局 | — | `voith-layout_endfield-tokens/` | 原始 |

## 如何创建新组合

```powershell
# 1. 创建目录
New-Item -ItemType Directory -Force -Path ".design_library/combinations/{layout-brand}_layout_{token-brand}_tokens"

# 2. 复制站点模板
$content = Get-Content ".design_library/{layout-brand}/site-template.html" -Raw

# 3. 替换 CSS 链接
$content = $content -replace 'href="colors_and_type\.css"', 'href="../../{token-brand}/colors_and_type.css"'

# 4. 如果用 Endfield 令牌（暗色），添加 dark class
$content = $content -replace '<body>', '<body class="dark">'

# 5. 保存
Set-Content -Path ".design_library/combinations/{layout-brand}_layout_{token-brand}_tokens/index.html" -Value $content -Encoding UTF8
```

## 设计原则

- 站点模板零硬编码颜色——全部走 CSS 变量
- 令牌文件提供可移植别名（`--color-background`、`--color-primary` 等）
- 所有变量都有 fallback 值，确保跨令牌兼容
- 组合后的页面是完整的、可直接打开的网站
