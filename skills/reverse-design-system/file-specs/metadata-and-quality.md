# Metadata & Quality Report Specification

Defines two required JSON files that exist in every design library but were missing from the workflow output spec.

---

## metadata.json

A minimal identity file at the root of every design library.

### Location

```
{output_dir}/metadata.json
```

### Schema

```json
{
  "id": "string — unique identifier for this library (alphanumeric + dots/hyphens)",
  "name": "string — brand name, matches the directory name",
  "version": "number — integer, starts at 1, increments on each update"
}
```

### Field Rules

| Field | Type | Required | Rule |
|-------|------|----------|------|
| `id` | string | yes | Generate a random 12-char ID like `H0DPM77H.7AKY4`. Can use base36 of a timestamp + random suffix. Must be unique per library. |
| `name` | string | yes | Exact brand name, matches the directory name under `.design_library/`. Case-sensitive. |
| `version` | number | yes | Start at `1`. Increment by 1 each time the library is regenerated or updated. |

### Generation Timing

Created at the **start** of Phase 2 (before token generation), so all downstream phases can reference it. Updated (version bump) at Phase 5 if the library already existed.

### Example (from Endfield)

```json
{
  "id": "H0DPM77H.7AKY4",
  "name": "Endfield",
  "version": 7
}
```

### Example (from OpenAI)

```json
{
  "id": "WCL18KCES_THRU",
  "name": "OpenAI",
  "version": 16
}
```

---

## quality-report.json

A quality metrics file inside the UIKit directory, recording what went into the UIKit generation.

### Location

```
{output_dir}/ui_kits/marketing/quality-report.json
```

### Schema

```json
{
  "schemaVersion": 1,
  "screensGenerated": "number — count of distinct UI screens/sections in the UIKit",
  "coreComponentsUsed": ["string array — slugs of components used in the UIKit"],
  "supportComponentsUsed": ["string array — additional non-core components, usually empty"],
  "previewClassReuseRate": "number — 0 to 1, fraction of UIKit CSS that reuses preview class names",
  "hasProductContext": "boolean — true if the UIKit uses real brand copy from uiCopySamples",
  "inventedComponents": ["string array — components that had to be invented (not from extraction), usually empty"],
  "renderedFromEvidence": ["string array — evidence sources cited, may be empty"],
  "warnings": ["string array — any quality warnings, may be empty"]
}
```

### Field Rules

| Field | Type | Required | Rule |
|-------|------|----------|------|
| `schemaVersion` | number | yes | Always `1`. |
| `screensGenerated` | number | yes | Count of major sections in the UIKit (typically 3-6: hero, features, component showcase, CTA, footer). |
| `coreComponentsUsed` | string[] | yes | Must include all 6 standard slugs: `["button", "card", "input", "badge", "cta-link", "navigation"]`. |
| `supportComponentsUsed` | string[] | yes | Additional components beyond the 6 standard. Usually `[]`. |
| `previewClassReuseRate` | number | yes | Target `0.8` or higher. Below `0.5` means UIKit is mostly custom CSS, not reusing preview patterns. |
| `hasProductContext` | boolean | yes | `true` if UIKit contains actual brand copy (from uiCopySamples), `false` if using placeholder text. |
| `inventedComponents` | string[] | yes | List any components that were not derived from extraction data. Should be `[]` for a faithful reverse-engineer. |
| `renderedFromEvidence` | string[] | yes | Cite evidence sources (e.g., `["phase0a-dom-components.json", "screenshot-full.png"]`). May be `[]`. |
| `warnings` | string[] | yes | Any quality concerns. May be `[]` if all good. |

### Generation Timing

Created at the **end** of Phase 4, after the UIKit HTML is written. The UIKit sub-agent fills in the metrics based on what it actually used.

### Example (from Endfield)

```json
{
  "schemaVersion": 1,
  "screensGenerated": 3,
  "coreComponentsUsed": ["button", "navigation", "card", "badge", "cta-link", "input"],
  "supportComponentsUsed": [],
  "previewClassReuseRate": 0.9,
  "hasProductContext": true,
  "inventedComponents": [],
  "renderedFromEvidence": [],
  "warnings": []
}
```

### Validation Checklist

- [ ] `metadata.json` exists at library root
- [ ] `metadata.json` has `id`, `name`, `version` fields
- [ ] `metadata.json` `name` matches directory name
- [ ] `quality-report.json` exists in `ui_kits/marketing/`
- [ ] `quality-report.json` has all 9 required fields
- [ ] `coreComponentsUsed` includes all 6 standard slugs
- [ ] `previewClassReuseRate` >= 0.5
- [ ] `hasProductContext` is `true`
