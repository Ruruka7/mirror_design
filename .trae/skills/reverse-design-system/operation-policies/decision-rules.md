# Decision Rules

## Purpose

Handle edge cases during reverse-engineering. These rules standardize how the workflow resolves ambiguous or hostile source conditions so that behavior is consistent across runs.

---

## Rule: SPA site (Next.js, Nuxt, React) with no CSS in HTML

- Check for `_next/static/css/` paths (Next.js)
- Check for `static/css/` paths (Nuxt)
- Check JS bundle for CSS import strings
- Try fetching with browser user-agent:
  ```
  curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
  ```
- If all fail: use browser_use subagent to render the page and extract computed styles

---

## Rule: Auth-required site

- Do NOT attempt to bypass auth
- Inform user the site requires authentication
- Ask user to provide an alternative public URL or export the CSS manually

---

## Rule: Minimal CSS (inline styles, styled-components)

- If <3 colors after extraction: try WebFetch to get rendered HTML
- If still sparse: ask user to provide a different page on same site
- Last resort: ask user to paste CSS manually

---

## Rule: Language detection

- Check `<html lang="...">` attribute
- Check for Chinese characters in page content
- Check for Japanese (hiragana/katakana) characters
- Default to `"zh"` if Chinese characters present, `"en"` otherwise

---

## Rule: Multiple accent colors

- If extraction reveals 2-3 distinct accent colors with similar frequency:
  - Classify as primary (most frequent), secondary (2nd), tertiary (3rd)
  - All three get full 10-step scales
  - `@group-priority` lists all three
- If 4+ accents: select top 3 by frequency, note others in README caveats

---

## Rule: Dark vs Light default theme

- If dominant background color is dark (luminance < 0.3): dark is default, add `.light{}` override
- If dominant background is light (luminance >= 0.3): light is default, add `.dark{}` override
- Use relative luminance: `L = 0.2126*R + 0.7152*G + 0.0722*B` (normalized 0-1)

---

## Rule: Custom font unavailable locally

- Note in README "Caveats / known substitutions" section
- Provide system fallback stack (`Segoe UI, Roboto, PingFang SC, Microsoft YaHei, sans-serif`)
- The font-family declaration in CSS still names the real font first

---

## Rule: clip-path extraction

- If clip-path polygon shapes found: record exact coordinates
- These become component trait values (e.g., button corner cut)
- If no clip-path found: omit from component traits, use border-radius instead
