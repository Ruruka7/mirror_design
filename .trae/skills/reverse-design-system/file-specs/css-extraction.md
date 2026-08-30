# CSS Extraction Specification

## Purpose

Define exactly what to extract from a website's CSS, how to extract it (exact regex patterns), and in what format to save results. This specification is consumed by the Phase 0 extraction sub-agent of the `reverse-design-system` skill. Every category below MUST be attempted; if a category yields zero matches, record an empty array for that key and continue to the next category — do not abort on an empty category.

All extraction operates on the concatenated raw CSS text (inline `<style>` blocks, linked `.css` files, and computed/inline `style=""` attributes). Before extraction, strip HTML comments `<!-- -->` and CSS comments `/* */` only when they would break a regex match; otherwise preserve them for the `Source:` annotations where relevant.

---

## Required Extraction Categories

Each category below specifies: **what to record**, the **exact regex pattern(s)**, and the **sorting / limit rule**.

### 1. Colors

**What to record:** color value + frequency count.

**Regex patterns:**
- Hex 3-digit: `#([0-9a-fA-F]{3})\b`
- Hex 4-digit: `#([0-9a-fA-F]{4})\b`
- Hex 6-digit: `#([0-9a-fA-F]{6})\b`
- Hex 8-digit: `#([0-9a-fA-F]{8})\b`
- rgba / rgb: `rgba?\(\s*([^)]+)\)`
- hsla / hsl: `hsla?\(\s*([^)]+)\)`

**Sort:** group by normalized value, sort by frequency descending. Keep top 40.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern '#([0-9a-fA-F]{3,8})\b','rgba?\(\s*([^)]+)\)','hsla?\(\s*([^)]+)\)' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Value.Trim().ToLower() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 40 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 2. Font Families

**What to record:** exact `font-family` declaration strings. Pay special attention to `@font-face` custom font names — these are the real brand fonts and MUST be captured verbatim.

**Regex pattern:** `font-family\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 15.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'font-family\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 15 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 3. Font Sizes

**What to record:** value + frequency count. Note rem→px conversion at 16px root (1rem = 16px).

**Regex pattern:** `font-size\s*:\s*(\d+(?:\.\d+)?(?:px|rem|em))`

**Sort:** frequency descending. Keep top 15.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'font-size\s*:\s*(\d+(?:\.\d+)?(?:px|rem|em))' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 15 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 4. Border (width, style, color)

**What to record:** composite `border` declarations — capture the full shorthand string (e.g., `1px solid #191919`).

**Regex pattern:** `border(?:-top|-right|-bottom|-left)?\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'border(?:-top|-right|-bottom|-left)?\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 5. Border-radius

**What to record:** radius values. Note whether the unit is `%` or `px` (and rem/em if present).

**Regex pattern:** `border-radius\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'border-radius\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 6. Box-shadow

**What to record:** full shadow strings. **Classify** each shadow as `"glow"` (pattern: `0 0 Xpx color` — zero x/y offset) or `"drop"` (pattern: non-zero Y offset).

**Regex pattern:** `box-shadow\s*:\s*([^;]+);`

**Classification logic:**
- Glow: offset-x and offset-y are both `0` (e.g., `0 0 10px #fffa00`).
- Drop: offset-y is non-zero (e.g., `0 4px 12px rgba(0,0,0,0.5)`).

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'box-shadow\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object {
    $v = $_.Groups[1].Value.Trim()
    $type = if ($v -match '^0\s+0\s+\d') { 'glow' } else { 'drop' }
    [pscustomobject]@{ value = $v; count = 1; type = $type }
  } |
  Group-Object value | Sort-Object Count -Descending | Select-Object -First 10
```

---

### 7. Transitions

**What to record:** duration + easing function (e.g., `0.3s ease`, `0.4s cubic-bezier(0.4,0,0.2,1)`).

**Regex pattern:** `transition(?:-property|-duration|-timing-function)?\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'transition(?:-property|-duration|-timing-function)?\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 8. Backdrop-filter

**What to record:** filter type (blur, saturate, brightness, etc.) and its value (e.g., `blur(12px)`).

**Regex pattern:** `backdrop-filter\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'backdrop-filter\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 9. Clip-path

**What to record:** polygon coordinates verbatim (e.g., `polygon(0 0, 100% 0, 100% 80%, 80% 100%, 0 100%)`). **Critical for geometric UIs** — clipped corners and angular shapes are signature design elements.

**Regex pattern:** `clip-path\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'clip-path\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 10. Letter-spacing

**What to record:** values. **Classify** as `"tight"` (negative value) or `"wide"` (positive value).

**Regex pattern:** `letter-spacing\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'letter-spacing\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object {
    $v = $_.Groups[1].Value.Trim()
    $type = if ($v -match '^-') { 'tight' } else { 'wide' }
    [pscustomobject]@{ value = $v; count = 1; type = $type }
  } |
  Group-Object value | Sort-Object Count -Descending | Select-Object -First 10
```

---

### 11. CSS Custom Properties (`--var: value`)

**What to record:** all custom property definitions — variable name and value.

**Regex pattern:** `(--[\w-]+)\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10 (or all if fewer).

**PowerShell command template:**
```powershell
$css | Select-String -Pattern '(--[\w-]+)\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object {
    [pscustomobject]@{ name = $_.Groups[1].Value; value = $_.Groups[2].Value.Trim() }
  } | Group-Object name | Sort-Object Count -Descending | Select-Object -First 10
```

---

### 12. Gradients (linear, radial, conic)

**What to record:** full gradient strings. For gradients with multiple color stops, **extract each stop color separately** into a `stops` array.

**Regex pattern:** `(linear|radial|conic)-gradient\(([^)]+)\)`

**Stop extraction:** within each gradient body, find all color tokens using the color regex from category 1.

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern '(linear|radial|conic)-gradient\(([^)]+)\)' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Value } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 13. Z-index

**What to record:** values. **Classify** layers (e.g., `0–9` = base, `10–99` = raised, `100–999` = overlay, `1000+` = modal/toast).

**Regex pattern:** `z-index\s*:\s*(-?\d+)`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'z-index\s*:\s*(-?\d+)' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 14. Padding

**What to record:** common values (shorthand and longhand).

**Regex pattern:** `padding(?:-top|-right|-bottom|-left)?\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'padding(?:-top|-right|-bottom|-left)?\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 15. Gap (flex/grid)

**What to record:** common gap values from flex and grid layouts.

**Regex pattern:** `gap\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'gap\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 16. @font-face

**What to record:** `font-family` name, `src` URL, `font-weight`, and `font-style`.

**Regex pattern (multiline):**
```
@font-face\s*\{[^}]*font-family\s*:\s*['"]?([^'";]+)['"]?[^}]*src\s*:\s*([^;]+);[^}]*font-weight\s*:\s*([^;]+);[^}]*font-style\s*:\s*([^;]+);[^}]*\}
```

> If `font-weight` or `font-style` are absent, record them as `null` — do not skip the entry.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern '@font-face\s*\{[^}]*?font-family\s*:\s*[''"]?([^''";]+)[''"]?[^}]*?src\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object {
    [pscustomobject]@{
      family = $_.Groups[1].Value.Trim()
      src    = $_.Groups[2].Value.Trim()
    }
  }
```

---

### 17. Transform

**What to record:** transform functions (e.g., `translateY(-50%)`, `rotate(45deg)`, `scale(1.05)`).

**Regex pattern:** `transform\s*:\s*([^;]+);`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'transform\s*:\s*([^;]+);' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Groups[1].Value.Trim() } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

### 18. Background Patterns (repeating gradients)

**What to record:** `repeating-linear-gradient` stripe patterns — capture the full string.

**Regex pattern:** `repeating-linear-gradient\(([^)]+)\)`

**Sort:** frequency descending. Keep top 10.

**PowerShell command template:**
```powershell
$css | Select-String -Pattern 'repeating-linear-gradient\(([^)]+)\)' -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Value } |
  Group-Object | Sort-Object Count -Descending | Select-Object -First 10 |
  ForEach-Object { [pscustomobject]@{ value = $_.Name; count = $_.Count } }
```

---

## Output Format

Save results as a single JSON file with the following top-level structure. Every key MUST be present even if its array is empty.

```json
{
  "colors": [
    { "value": "#191919", "count": 142 },
    { "value": "#fffa00", "count": 87 }
  ],
  "fonts": [
    { "value": "'DIN Alternate', sans-serif", "count": 34 }
  ],
  "sizes": [
    { "value": "16px", "count": 56, "rem": "1rem" }
  ],
  "borders": [
    { "value": "1px solid #191919", "count": 12 }
  ],
  "radius": [
    { "value": "4px", "count": 23 }
  ],
  "shadows": [
    { "value": "0 0 10px #fffa00", "count": 8, "type": "glow" },
    { "value": "0 4px 12px rgba(0,0,0,0.5)", "count": 5, "type": "drop" }
  ],
  "transitions": [
    { "value": "0.3s ease", "count": 18 }
  ],
  "backdropFilters": [
    { "value": "blur(12px)", "count": 7 }
  ],
  "clipPaths": [
    { "value": "polygon(0 0, 100% 0, 100% 80%, 80% 100%, 0 100%)", "count": 6 }
  ],
  "letterSpacing": [
    { "value": "-0.02em", "count": 9, "type": "tight" }
  ],
  "customProperties": [
    { "name": "--brand-primary", "value": "#fffa00", "count": 3 }
  ],
  "gradients": [
    { "value": "linear-gradient(135deg, #fffa00, #00ffa2)", "count": 4, "stops": ["#fffa00", "#00ffa2"] }
  ],
  "zIndex": [
    { "value": "1000", "count": 3, "layer": "modal" }
  ],
  "padding": [
    { "value": "12px 24px", "count": 15 }
  ],
  "gap": [
    { "value": "16px", "count": 20 }
  ],
  "fontFace": [
    { "family": "EndfieldTitle", "src": "url(/fonts/endfield-title.woff2)", "weight": "700", "style": "normal" }
  ],
  "transforms": [
    { "value": "translateY(-50%)", "count": 6 }
  ],
  "backgroundPatterns": [
    { "value": "repeating-linear-gradient(45deg, #191919, #191919 10px, #2a2a2a 10px, #2a2a2a 20px)", "count": 2 }
  ]
}
```

---

## Rules

1. **Group by value, sort by frequency descending.** Every array entry includes a `count` field.

2. **Top-N limits:**
   - Colors: top 40
   - Fonts (font-family): top 15
   - Sizes (font-size): top 15
   - All other categories: top 10

3. **@font-face:** extract `font-family` name and `src` URL at minimum. Record `font-weight` and `font-style` when present; use `null` when absent.

4. **Gradients with multiple color stops:** extract each stop color separately into a `stops` array. The `value` field holds the full gradient string; the `stops` array holds individual color values.

5. **Color normalization** (applied before grouping):
   - `#fff` → `#ffffff`
   - `#FFF` → `#ffffff` (lowercase all hex digits)
   - `rgba(0,0,0,.5)` → `rgba(0,0,0,0.5)` (add leading zero to fractional alpha)
   - `RGBA(0, 0, 0, 0.5)` → `rgba(0,0,0,0.5)` (lowercase function name, remove internal spaces)
   - Hex 8-digit with alpha: keep alpha channel (e.g., `#19191980` stays `#19191980`)

6. **rem→px note:** for font-size entries, include a `rem` field showing the rem equivalent when the source value is in px (e.g., `16px` → `rem: "1rem"`), and vice versa. Conversion root is 16px.

---

## Hard Stop Rule

If fewer than 3 distinct colors are found across all color patterns (hex + rgb + hsl after normalization), the CSS is too sparse to reverse-engineer a design system. In this case:

- **Stop immediately.**
- Do not produce the output JSON.
- Report: `"status": "sparse-css", "distinctColors": <N>, "message": "Fewer than 3 distinct colors found — CSS is too sparse for design system extraction."`

This prevents generating a hollow design system from a near-empty stylesheet.
