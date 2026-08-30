# Sub-Agent Dispatch Templates

## Purpose

Standardize how sub-agents are dispatched for each phase. Consistent dispatch contracts guarantee that sub-agents receive the same inputs, constraints, and return expectations regardless of which phase they serve.

---

## General dispatch rules

- Max 3 parallel sub-agents at a time
- Each sub-agent gets: task description, mandatory reads, output path, CSS link path, brand name, language, product type, forbidden tools list, return report path, completion contract
- Sub-agents MUST read constraint files before writing
- Sub-agents MUST NOT use TodoWrite, Skill, RunCommand, SearchCodebase

---

## Phase 2 — Token Generation (1 sub-agent)

- Task: "Generate token CSS for brand {BrandName}"
- Mandatory reads: `file-specs/token-css.md`, `colors_and_type.css` (if exists), brand profile JSON
- Output: `{output_dir}/colors_and_type.css`
- Must include: all color scales, typography, spacing, radius, shadows, transitions, letter-spacing
- Return: `writtenFiles[]`, `tokenCount`, `flatTokenSummary`, `warnings[]`

---

## Phase 3 — Component Previews (3 parallel sub-agents, 2 components each)

- Batch 1: `button` + `card`
- Batch 2: `input` + `badge`
- Batch 3: `cta-link` + `navigation`
- Each sub-agent reads: `file-specs/preview-page.md`, `colors_and_type.css`, component JSON
- Output: `preview/component-{slug}.html`
- CSS link: `<link rel="stylesheet" href="../colors_and_type.css">`
- Return: `writtenFiles[]`, `warnings[]`, `undefinedCssVars` count

---

## Phase 4a — Documentation (2 parallel sub-agents)

- Sub-agent A: `SKILL.md` + `library-consumption.json`
- Sub-agent B: `README.md`
- Both read: `file-specs/documentation.md`, brand profile, `colors_and_type.css`, `components.css`
- Do NOT read component JSON files
- Return: `writtenFiles[]`, `warnings[]`

---

## Phase 4b — UIKit (1 sub-agent, after 4a)

- Reads: `file-specs/uikit.md`, brand profile, `colors_and_type.css`, component index, all component JSONs, `uikit-plan.json`, `README.md`, `SKILL.md`
- Output: `ui_kits/marketing/index.html`
- CSS link: `<link rel="stylesheet" href="../../colors_and_type.css">`
- Return: `writtenFiles[]`, `warnings[]`

---

## Dispatch query template

Structure each sub-agent query with these sections:

1. Task description (1-2 lines)
2. MANDATORY READS (file list with read order)
3. OUTPUT (exact file path)
4. CSS link path (EXACT string)
5. BrandName, Language, ProductType
6. Forbidden tools list
7. ReturnReportFileAbs path
8. Completion contract (what to write, what to return)
9. Final response format (e.g., "已完成组件预览。")

---

## Sub-agent response rules

- Do NOT return JSON in final response (write it to report file instead)
- Do NOT return file paths or analysis in final response
- Final response should be a short confirmation like "已完成组件预览。" or "已完成 Token 生成。"
