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

---

## Rule: Browser automation fails (browser_use subagent errors)

- First fallback: try browser_navigate + browser_evaluate directly (without subagent)
- Second fallback: try WebFetch to get rendered HTML
- Third fallback: curl + css-extraction.md only (CSS file regex)
- If all fail: inform user, ask them to manually save the page's CSS

---

## Rule: JS-heavy SPA with no server-side rendering

- curl will return empty shell HTML
- Browser is the ONLY viable extraction method
- Must wait for networkidle + additional 2s for lazy content
- Check for common SPA frameworks: Next.js (_next/), Nuxt (_nuxt/), React (react-root), Vue (#app)

---

## Rule: Page requires interaction before content loads

- Some sites show a loading screen or cookie banner first
- Use browser_click to dismiss cookie/consent banners
- Use browser_scroll to trigger scroll animations
- Wait for content after each interaction

---

## Rule: Page has infinite scroll or lazy-loaded sections

- Scroll to bottom 3 times with 1s pauses
- Capture computed styles after each scroll (styles may change)
- Record which sections appeared after scroll

---

## Rule: Computed style conflict (browser vs CSS file)

- Browser computed style ALWAYS wins
- CSS file value is recorded as "declared" in README caveats
- If significant discrepancy (e.g., different font family): note in README, investigate why (media query, JS override, CSS-in-JS)

---

## Rule: Multiple page templates on same domain

- If user gives homepage URL, also check /about, /pricing, /contact
- Extract from each page, merge results
- Different pages may reveal different component variants

---

## Rule: Page behind paywall or geo-restriction

- Try with different user-agent
- If still blocked: inform user, ask for alternative URL or saved HTML
