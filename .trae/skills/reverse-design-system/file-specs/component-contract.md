# Component Contract JSON Schema

## Purpose

This specification defines the structure for `component/{slug}.json` files. Each file is a machine-readable contract that captures one component's anatomy, variant dimensions, representative variants, and trait values. Contracts are the single source of truth used by preview HTML, the UI Kit, and documentation generation. They must be fully derivable from `colors_and_type.css` and `_evidence/` assets — no invented values.

## Standard 6 Components

Every reverse-design-system library MUST produce contracts for exactly these six components:

| slug         | category   | description                                           |
| ------------ | ---------- | ----------------------------------------------------- |
| button       | action     | Primary interactive trigger                           |
| card         | surface    | Container surface for grouped content                |
| input        | form       | Text-entry field and related form controls           |
| badge        | status     | Compact status / category label                      |
| cta-link     | action     | Call-to-action link, visually distinct from body link |
| navigation   | shell      | Top-level site or app navigation shell                |

No additional components may be invented. If a component cannot be reverse-engineered from evidence, record the gap in `unknowns` rather than fabricating it.

## Top-Level Schema

Each `component/{slug}.json` file is a single JSON object with the following fields.

### `slug`
- Type: `string`
- MUST match the filename without extension (e.g. the file `button.json` has `slug: "button"`).
- MUST be one of the six standard slugs above.

### `schemaVersion`
- Type: `number`
- ALWAYS `2`. Indicates this contract format.

### `name`
- Type: `string`
- Human-readable display name (e.g. `"Button"`, `"CTA Link"`).

### `category`
- Type: `string`
- One of: `action` | `surface` | `form` | `status` | `shell`.

### `sourceKind`
- Type: `string`
- Always `"from-scratch"` — contracts are authored from analyzed evidence, not extracted from a packaged library.

### `confidence`
- Type: `string`
- One of: `high` | `medium`.
  - `high` — every trait is directly backed by a token or clear evidence.
  - `medium` — one or more traits are inferred from adjacent patterns; document the inference in `unknowns`.

### `semanticTypeCandidates`
- Type: `string[]`
- Candidate semantic roles this component may play (e.g. `["primary-action", "submit", "confirm"]` for button).

### `variantDimensions`
- Type: `object`
- Keys are dimension names (e.g. `"intent"`, `"size"`, `"state"`); values are `string[]` enumerating the values along that dimension.
- Example:
  ```json
  "variantDimensions": {
    "intent": ["primary", "secondary", "ghost"],
    "size": ["md", "lg"],
    "state": ["default", "hover", "focus", "disabled"]
  }
  ```

### `representativeVariants`
- Type: `array` of 3–4 variant objects.
- Each variant object:
  - `name` — `string`, human label (e.g. `"Primary — default"`).
  - `whySelected` — `string`, design rationale for why this variant is representative (not boilerplate).
  - `traits` — `object` of CSS property → value pairs. Values MUST follow the Trait Value Rules below.
  - `childrenDigest` — `string[]`, expected child elements (e.g. `["icon", "label"]`).

### `anatomy`
- Type: `string[]`
- Structural elements that compose the component (e.g. `["container", "leading-icon", "label", "trailing-caret"]`).

### `structurePatterns`
- Type: `string[]`
- Layout patterns the component relies on (e.g. `"inline-flex row"`, `"flex column gap-md"`).

### `usageHints`
- Type: `object`
  - `priorityHint` — `string`, when to prefer this component over alternatives.
  - `a11y` — `string`, accessibility requirements (focus management, aria, contrast).

### `doNotInvent`
- Type: `string[]`
- Explicitly excluded features (e.g. `["loading spinner", "icon-only variant"]`). Use this to prevent hallucinated capabilities.

### `unknowns`
- Type: `string[]`
- Acknowledged gaps where evidence was insufficient (e.g. `"disabled state color inferred, no evidence found"`).

## Trait Value Rules

The `traits` object on each variant is the most strictly governed part of the contract.

1. **All color, font, spacing, radius, shadow, and transition values MUST use `var()` references** to tokens defined in `colors_and_type.css`.
   - Correct: `"background": "var(--color-primary)"`, `"border-radius": "var(--radius-md)"`.
   - Incorrect: `"background": "#5B8CFF"`.
2. **Direct hex values are ONLY permitted when no token exists** for that value in `colors_and_type.css`. This must be rare; if a hex is used, record the gap in `unknowns` explaining why no token covers it.
3. **`clip-path` values MUST be raw polygon strings**, not token references:
   - Correct: `"clip-path": "polygon(0 0, 100% 0, 100% 85%, 50% 100%, 0 85%)"`.
4. **Numeric scale values** (font-size, width, height, padding) MUST reference spacing/type scale tokens via `var()`. Bare numbers like `"padding: 12px"` are not allowed unless no token exists.
5. **Composite shorthand values** may combine tokens, e.g. `"box-shadow": "var(--shadow-md), inset 0 0 0 1px var(--color-surface-border)"`.
6. **No `url()` references** to external assets in traits. Images/icons are described in `childrenDigest`, not embedded.

## Complete Example — `button.json` (Endfield reference)

```json
{
  "slug": "button",
  "schemaVersion": 2,
  "name": "Button",
  "category": "action",
  "sourceKind": "from-scratch",
  "confidence": "high",
  "semanticTypeCandidates": ["primary-action", "submit", "confirm", "navigate-forward"],
  "variantDimensions": {
    "intent": ["primary", "secondary", "ghost"],
    "size": ["md", "lg"],
    "state": ["default", "hover", "focus", "disabled"]
  },
  "representativeVariants": [
    {
      "name": "Primary — default",
      "whySelected": "The dominant CTA used across hero and forms; defines the brand accent at rest.",
      "traits": {
        "background": "var(--color-primary)",
        "color": "var(--color-on-primary)",
        "font-family": "var(--font-body)",
        "font-size": "var(--font-size-body)",
        "font-weight": "var(--font-weight-bold)",
        "letter-spacing": "var(--letter-spacing-wide)",
        "padding": "var(--space-3) var(--space-6)",
        "border-radius": "var(--radius-md)",
        "border": "none",
        "transition": "var(--transition-fast)"
      },
      "childrenDigest": ["label"]
    },
    {
      "name": "Primary — hover",
      "whySelected": "Reveals the accent interaction lift and brightness shift that anchors hover language.",
      "traits": {
        "background": "var(--color-primary-hover)",
        "color": "var(--color-on-primary)",
        "box-shadow": "var(--shadow-md)",
        "transform": "translateY(-2px)"
      },
      "childrenDigest": ["label"]
    },
    {
      "name": "Secondary — default",
      "whySelected": "Demonstrates the surface + border treatment used for non-primary actions.",
      "traits": {
        "background": "var(--color-surface)",
        "color": "var(--color-text-primary)",
        "border": "1px solid var(--color-surface-border)",
        "padding": "var(--space-3) var(--space-6)",
        "border-radius": "var(--radius-md)"
      },
      "childrenDigest": ["label"]
    },
    {
      "name": "Ghost — default",
      "whySelected": "Low-emphasis tertiary control; validates transparent + text-only treatment.",
      "traits": {
        "background": "transparent",
        "color": "var(--color-text-secondary)",
        "padding": "var(--space-2) var(--space-4)",
        "border-radius": "var(--radius-sm)"
      },
      "childrenDigest": ["label"]
    }
  ],
  "anatomy": ["container", "label"],
  "structurePatterns": ["inline-flex row", "center-aligned", "gap var(--space-2) when icon present"],
  "usageHints": {
    "priorityHint": "Use primary for the single most important action per view; secondary for supporting actions; ghost for tertiary or dismissive actions.",
    "a11y": "Must show a visible :focus ring using --color-focus; disabled state must set aria-disabled and reduce opacity; minimum touch target 44px."
  },
  "doNotInvent": ["loading spinner", "icon-only variant", "split-button menu", "progress fill"],
  "unknowns": ["disabled state opacity value inferred from general practice; no explicit evidence in _evidence/."]
}
```

## Field Presence Requirements

| Field                          | Required | Notes                                   |
| ------------------------------ | -------- | --------------------------------------- |
| slug                           | yes      | matches filename                        |
| schemaVersion                  | yes      | always `2`                              |
| name                           | yes      | —                                       |
| category                       | yes      | enum                                    |
| sourceKind                     | yes      | always `"from-scratch"`                 |
| confidence                     | yes      | `high` or `medium`                      |
| semanticTypeCandidates         | yes      | at least 1 entry                        |
| variantDimensions              | yes      | at least 1 dimension                     |
| representativeVariants         | yes      | 3–4 entries                             |
| anatomy                        | yes      | —                                       |
| structurePatterns              | yes      | —                                       |
| usageHints                     | yes      | both `priorityHint` and `a11y`          |
| doNotInvent                    | yes      | may be empty array but must be present  |
| unknowns                       | yes      | may be empty array but must be present  |

## Validation Notes

- JSON must be valid and parseable; trailing commas are not permitted.
- Every `var()` token referenced in `traits` must resolve to a definition in `colors_and_type.css`. Unresolved references invalidate the contract.
- `representativeVariants` count must be between 3 and 4 inclusive.
- If `confidence` is `medium`, `unknowns` must contain at least one entry explaining the inference.
