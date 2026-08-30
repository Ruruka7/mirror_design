# Brand Profile JSON Schema

## Purpose

Define the brand profile JSON that bridges Phase 0 extraction data to Phase 2 token generation. The brand profile is a compact, human-readable summary of the website's visual identity, derived from the raw CSS extraction JSON (see `css-extraction.md`). It tells the token-generation sub-agent what kind of product this is, what personality the design conveys, and what the real brand colors are — so generated tokens stay anchored to source evidence rather than drifting toward generic defaults.

The brand profile MUST be grounded in extracted data. Fields that cite colors, text, or visual properties MUST reference real values found during Phase 0 extraction. Fabricated content is a hard error.

---

## JSON Schema

```json
{
  "productType": "Marketing/Landing",
  "confidence": "high|medium",
  "personality": ["keyword1", "keyword2", "keyword3"],
  "language": "zh|en|ja|ko|...",
  "visualTone": "One-sentence description citing real extracted values.",
  "kitType": "marketing",
  "colorNamingPrefix": "brandname-lowercase",
  "uiCopySamples": ["actual UI text from the website"]
}
```

---

## Field Definitions

| Field | Type | Required | Description | Validation Rule |
|---|---|---|---|---|
| `productType` | string | yes | The product category. Always `"Marketing/Landing"` for public-facing websites. | Must equal `"Marketing/Landing"`. |
| `confidence` | string | yes | How confidently the design system can be extracted, based on color diversity. | Must be `"high"` (>20 distinct colors) or `"medium"` (3–20 distinct colors). Never `"low"` — if <3 colors, stop (see hard stop rule in `css-extraction.md`). |
| `personality` | array of strings | yes | 3–5 keyword descriptors of the visual style, derived from extracted evidence. | Must contain 3–5 items. Each item must be from the keyword bank (see below). |
| `language` | string | yes | Primary language of the website's UI text. | ISO 639-1 code: `"zh"`, `"en"`, `"ja"`, `"ko"`, etc. Detect from actual page text content. |
| `visualTone` | string | yes | A single-sentence description of the visual identity. MUST cite at least one real hex color value from extraction. | Must contain at least one `#` hex color reference. |
| `kitType` | string | yes | The design-kit type this profile maps to. | Must be `"marketing"`. |
| `colorNamingPrefix` | string | yes | Brand name in lowercase, used as the CSS variable prefix in Phase 2 tokens. | Lowercase, no spaces, no hyphens at start/end (e.g., `"endfield"`, `"openai"`). |
| `uiCopySamples` | array of strings | yes | 5–10 actual text strings found on the website (nav items, CTA buttons, headings, labels). | Must have at least 5 items. NEVER fabricate — every string must appear in the extracted page text. |

---

## ProductType

Always `"Marketing/Landing"` for public-facing websites. This skill targets marketing pages, landing pages, and product microsites — not dashboards or admin panels. If the website is clearly an app/dashboard (e.g., contains data tables, form-heavy CRUD layouts), still set `"Marketing/Landing"` because the extraction workflow and token set are optimized for marketing contexts.

---

## Confidence

Determined by the count of distinct colors found in Phase 0 extraction (after normalization):

| Distinct Colors | Confidence |
|---|---|
| > 20 | `"high"` |
| 3–20 | `"medium"` |
| < 3 | **Stop** — do not produce a brand profile. Report sparse CSS. |

`"low"` is never used. If fewer than 3 distinct colors are found, the extraction is too sparse and the entire pipeline halts per the hard stop rule in `css-extraction.md`.

---

## Personality

Derive 3–5 keywords from the visual evidence. Select from the keyword bank below — do not invent keywords outside this list:

| Keyword | Visual evidence that justifies it |
|---|---|
| `minimal` | Very few colors (≤5), generous whitespace, thin borders |
| `editorial` | Serif or high-contrast display fonts, magazine-style layout |
| `industrial` | Monospace/technical fonts, sharp corners, mechanical aesthetic |
| `post-apocalyptic` | Dark backgrounds, hazard colors (yellow/orange), distressed textures |
| `corporate` | Blue/grey palette, conservative spacing, sans-serif |
| `aggressive` | High contrast, bold weights, heavy shadows, bright accents on dark |
| `soft` | Rounded corners, pastel palette, low contrast, gentle shadows |
| `geometric` | Clip-path polygons, angular shapes, grid-based layouts |
| `organic` | Curved borders, natural color palette, irregular spacing |
| `high-energy` | Bright saturated colors, glows, animations, gradients |
| `calm` | Muted palette, low saturation, slow transitions, ample spacing |
| `technical` | Monospace fonts, code-like elements, data-dense, precise grids |
| `luxury` | Gold/black palette, serif display fonts, fine spacing, subtle shadows |
| `playful` | Rounded shapes, bright multi-color palette, varied type sizes |
| `dark` | Dark background dominant (#1a1a1a or darker as primary surface) |
| `bright` | Light background dominant (#f5f5f5 or lighter as primary surface) |
| `monochrome` | Single-hue palette with varied lightness |
| `colorful` | 4+ distinct hues actively used as accents |
| `brutalist` | Raw/unstyled look, exposed borders, default fonts, high contrast |
| `futuristic` | Glows, gradients, neon accents, dark surfaces, sci-fi aesthetic |

Select keywords that are **mutually consistent** — do not pick contradictory pairs (e.g., `calm` + `high-energy` together is invalid).

---

## Language

Detect from the website's actual text content:

- Chinese text (汉字) → `"zh"`
- English text → `"en"`
- Japanese text (hiragana/katakana/kanji mixed) → `"ja"`
- Korean text (hangul) → `"ko"`

If the site is multilingual, pick the dominant UI language (the language of the primary navigation and hero heading).

---

## VisualTone

A single dense sentence that captures the visual identity. **MUST cite at least one real hex color value** from the Phase 0 extraction. Reference the primary, secondary, and background colors with their actual hex codes.

**Example (good):**
> "Industrial post-apocalyptic dark UI with hazard-yellow #fffa00 primary, neon green #00ffa2 secondary, magenta #ff1aac tertiary, built on near-black #191919 ground with #ffffff text."

**Example (invalid — no hex reference):**
> "Dark industrial design with yellow accents." ← Missing hex values; rejected.

---

## ColorNamingPrefix

The brand name in lowercase with no spaces, used as the CSS variable prefix in Phase 2 token generation (e.g., `--endfield-primary-600`). Extract the brand name from the page title, logo alt text, or og:site_name meta tag.

Examples: `"endfield"`, `"openai"`, `"linear"`, `"vercel"`.

---

## uiCopySamples

Collect 5–10 real text strings from the website. Prioritize:

1. Primary navigation items
2. CTA button labels
3. Hero / section headings
4. Footer links or labels

**NEVER fabricate.** Every string must be verbatim from the rendered page text. If the site has fewer than 5 distinct text strings (extremely unlikely for a marketing page), supplement with button labels and link text — but never invent text that does not appear on the page.

---

## Validation Rules

| # | Rule |
|---|---|
| 1 | `productType` must be `"Marketing/Landing"` |
| 2 | `confidence` must be `"high"` or `"medium"` (never `"low"`) |
| 3 | `personality` must have 3–5 items, each from the keyword bank |
| 4 | `visualTone` must contain at least one hex color reference (`#` followed by hex digits) |
| 5 | `uiCopySamples` must have at least 5 items |
| 6 | `colorNamingPrefix` must be lowercase with no spaces |
| 7 | `language` must be a valid ISO 639-1 code |
| 8 | `kitType` must be `"marketing"` |

If any validation rule fails, the brand profile is invalid and the pipeline must not proceed to Phase 2.

---

## Example Output

Reference example, derived from the Endfield (终末地) marketing site:

```json
{
  "productType": "Marketing/Landing",
  "confidence": "high",
  "personality": ["industrial", "post-apocalyptic", "high-energy", "geometric", "aggressive"],
  "language": "zh",
  "visualTone": "Industrial post-apocalyptic dark UI with hazard-yellow #fffa00 primary, neon green #00ffa2 secondary, magenta #ff1aac tertiary, built on near-black #191919 ground with #fff text.",
  "kitType": "marketing",
  "colorNamingPrefix": "endfield",
  "uiCopySamples": ["ENDFIELD", "终末地", "探索", "领取", "预约", "世界观", "角色", "开发组"]
}
```
