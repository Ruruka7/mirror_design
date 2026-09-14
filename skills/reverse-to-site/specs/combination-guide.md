# Combination Guide

How to combine any token set with any site template to create a new website.

---

## 1. The Combination Formula

The core mechanic of the reverse-to-site skill is **cross-combination**:

```
Site Template (A) + Token CSS (B) = New Website with A's structure and B's visual style
```

- **Site Template (A)** provides the HTML structure, layout, component composition, and content slots.
- **Token CSS (B)** provides the visual identity — colors, typography, spacing, shadows, radii.
- The **combination** is a new `index.html` that uses A's body but links to B's `colors_and_type.css`.

Because every token file defines the **same set of portable CSS variables** (see the [Variable Reference](../reference/variable-reference.md)), and every site template consumes only those variables, any template can be paired with any token set without editing a single line of HTML markup.

---

## 2. How to Create a Combination

### Prerequisites

- A site template exists at `.design_library/{A}/site-template.html`
- A token CSS exists at `.design_library/{B}/colors_and_type.css`
- You know whether token B is **dark-themed** or **light-themed**

### PowerShell Script

```powershell
# 1. Create directory
New-Item -ItemType Directory -Force -Path ".design_library/combinations/{A}-layout_{B}-tokens"

# 2. Read source template
$content = Get-Content ".design_library/{A}/site-template.html" -Raw

# 3. Replace CSS link
$content = $content -replace 'href="colors_and_type\.css"', 'href="../../{B}/colors_and_type.css"'

# 4. Handle dark theme
# If token B is dark-themed (like Endfield), add class="dark" to body:
$content = $content -replace '<body>', '<body class="dark">'
# If token B is light-themed and template has class="dark", remove it:
$content = $content -replace '<body class="dark">', '<body>'

# 5. Update title
$content = $content -replace '<title>.*?</title>', '<title>{A} layout x {B} tokens</title>'

# 6. Save
Set-Content -Path ".design_library/combinations/{A}-layout_{B}-tokens/index.html" -Value $content -Encoding UTF8
```

### Step-by-Step Explanation

| Step | Action | Why |
|------|--------|-----|
| 1 | Create the combination directory | Each combination gets its own folder under `combinations/` |
| 2 | Read the source template HTML | We need the full markup to re-link the CSS |
| 3 | Replace the CSS href | Point from the template's own tokens to the target token set's CSS |
| 4 | Handle dark theme class | Dark token sets require `class="dark"` on `<body>` so `:root` dark overrides activate |
| 5 | Update the `<title>` | Makes the combination identifiable in the browser tab |
| 6 | Save as `index.html` | Standard entry point for easy preview |

### Naming Convention

Combination directories follow the pattern:

```
{TemplateBrand}-layout_{TokenBrand}-tokens
```

Examples:
- `OpenAI-layout_Endfield-tokens`
- `Endfield-layout_OpenAI-tokens`
- `OpenAI-layout_OpenAI-tokens`

---

## 3. Bash / Linux Equivalent

For non-Windows environments, the same transformation can be done with `sed`:

```bash
mkdir -p ".design_library/combinations/{A}-layout_{B}-tokens"
sed -e 's|href="colors_and_type.css"|href="../../{B}/colors_and_type.css"|' \
    -e 's|<body>|<body class="dark">|' \
    ".design_library/{A}/site-template.html" \
    > ".design_library/combinations/{A}-layout_{B}-tokens/index.html"
```

**Notes on the bash version:**
- The `<body>` → `<body class="dark">` substitution should only be applied when token B is dark-themed. If B is light-themed, omit that `-e` line or use the inverse:
  ```bash
  -e 's|<body class="dark">|<body>|'
  ```
- To also update the title, add:
  ```bash
  -e 's|<title>.*</title>|<title>{A} layout x {B} tokens</title>|'
  ```

---

## 4. Verification

After creating a combination, verify it renders correctly:

1. **Open the file** — Navigate to `.design_library/combinations/{A}-layout_{B}-tokens/index.html` and open it in a browser.
2. **Visual check** — Confirm:
   - The page structure matches template A's layout (header, hero, cards, footer, etc.)
   - The colors, fonts, spacing, and shadows reflect token B's visual identity
   - No unstyled elements or broken layouts
3. **Dark theme check** — If token B is dark-themed, verify the background is dark and text is light. If it still looks light, the `class="dark"` step was missed.
4. **Responsive check** — Resize the browser to confirm the template's responsive breakpoints still work with the new tokens.

If something looks wrong:
- **Wrong colors** → Check the CSS href path is correct (`../../{B}/colors_and_type.css`)
- **Dark theme not applied** → Ensure `<body class="dark">` is present
- **Missing styles entirely** → Confirm the token CSS file exists at the linked path

---

## 5. Available Combinations

The current design library contains **3 brands**, each with both a site template and a token CSS. This yields a **3 x 3 = 9** combination matrix.

### Combination Matrix

| | **OpenAI tokens** | **Anthropic tokens** | **Endfield tokens** |
|---|---|---|---|
| **OpenAI layout** | OpenAI layout x OpenAI tokens | OpenAI layout x Anthropic tokens | OpenAI layout x Endfield tokens |
| **Anthropic layout** | Anthropic layout x OpenAI tokens | Anthropic layout x Anthropic tokens | Anthropic layout x Endfield tokens |
| **Endfield layout** | Endfield layout x OpenAI tokens | Endfield layout x Anthropic tokens | Endfield layout x Endfield tokens |

### Directory Names

| | OpenAI tokens | Anthropic tokens | Endfield tokens |
|---|---|---|---|
| **OpenAI layout** | `OpenAI-layout_OpenAI-tokens` | `OpenAI-layout_Anthropic-tokens` | `OpenAI-layout_Endfield-tokens` |
| **Anthropic layout** | `Anthropic-layout_OpenAI-tokens` | `Anthropic-layout_Anthropic-tokens` | `Anthropic-layout_Endfield-tokens` |
| **Endfield layout** | `Endfield-layout_OpenAI-tokens` | `Endfield-layout_Anthropic-tokens` | `Endfield-layout_Endfield-tokens` |

### Theme Notes

| Token Brand | Theme | Requires `class="dark"` |
|-------------|-------|----------------------|
| OpenAI | Light | No |
| Anthropic | Light | No |
| Endfield | Dark | Yes |

When the template and tokens are from the same brand (the diagonal), the result is the "native" combination — essentially the original website.

---

## 6. When to Create Combinations

Create a combination when the user:

- **Wants to see a design in a different style** — "Show me the OpenAI layout but with Endfield's dark sci-fi aesthetic."
- **Wants to prototype a new site quickly** — "Build a landing page using Anthropic's structure but OpenAI's clean black-and-white identity."
- **Wants to compare visual languages** — "Generate all 9 combinations so I can compare how each brand's structure looks under each brand's tokens."
- **Is exploring brand flexibility** — "Which template works best with a dark token set?"
- **Needs a variant for A/B testing** — "Create two versions of this page with different token sets."

### Tips

- The **diagonal** combinations (same brand layout + same brand tokens) reproduce the original site — useful as a baseline for comparison.
- **Cross-combinations** (different brand layout + different brand tokens) are where the most interesting design exploration happens.
- When a dark token set is applied to a light-themed template, check that all components have dark-aware styling. If any component hardcodes a light color, it will clash — this is a signal that the template needs a fix to be fully portable.
- You can create combinations programmatically by looping over all template-token pairs, which is useful for generating the full matrix at once.

---

## See Also

- [Variable Reference](../reference/variable-reference.md) — Complete lookup table of all portable CSS variables
