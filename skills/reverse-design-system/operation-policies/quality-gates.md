# Quality Gates

## Purpose

Define pass/fail criteria between each phase of the reverse-design-system workflow. Each gate must be satisfied before advancing to the next phase. Gates enforce extraction completeness, token integrity, component coverage, documentation completeness, and final validation.

---

## Gate 0 (after Phase 0A browser extraction)

- [PASS] Full-page screenshot captured successfully
- [PASS] Computed styles extracted from at least 50 elements
- [PASS] At least 3 distinct background colors found in computed styles
- [PASS] At least 2 distinct font-family values identified
- [PASS] At least 1 component type identified in DOM (navigation, button, card, etc.)
- [PASS] At least 1 layout system (grid or flex) pattern captured
- [PASS] Page section structure mapped (at least 3 sections identified)
- [WARN] If CSS custom properties not found at runtime → proceed, use CSS file extraction
- [WARN] If responsive breakpoint test skipped → proceed, note in README
- [FAIL] If screenshot capture fails → try curl fallback, if curl also fails → stop, inform user
- [FAIL] If <3 colors in computed styles AND <3 colors in CSS files → stop, report sparse
- [FAIL] If no fonts identified in computed styles AND no @font-face in CSS → proceed with system fallback, warn

---

## Gate 0B (after Phase 0B CSS supplement)

- [PASS] CSS files downloaded and parsed (if accessible)
- [WARN] If CSS files not accessible → proceed with browser-only data
- [PASS] @font-face declarations extracted (if CSS accessible)
- [PASS] :root custom properties extracted from CSS (cross-reference with browser runtime)

---

## Gate 1 (after Phase 1 brand profile)

- [PASS] Brand profile has all required fields
- [PASS] visualTone contains at least one hex reference
- [PASS] personality has 3-5 items
- [PASS] uiCopySamples has at least 5 items
- [PASS] colorNamingPrefix is lowercase, no spaces

---

## Gate 2 (after Phase 2 token generation)

- [PASS] colors_and_type.css exists and is non-empty
- [PASS] css.json generated successfully
- [PASS] All 10-step scales present (50-900) for each color group
- [PASS] Each color group has exactly one /* @primary */ marker
- [PASS] @group-priority comment present
- [PASS] No @import for custom fonts
- [WARN] If undefined CSS variables found in theme blocks → fix before proceeding

---

## Gate 3 (after Phase 3 components)

- [PASS] components/index.json exists with 6 components
- [PASS] Each component has a {slug}.json with all required fields
- [PASS] Each component has a preview/component-{slug}.html
- [PASS] components.css extracted successfully
- [PASS] All preview HTML files use correct CSS link path (../colors_and_type.css)
- [WARN] If any preview has undefined CSS vars → fix

---

## Gate 4 (after Phase 4 docs + UIKit)

- [PASS] README.md exists with all 6 required sections
- [PASS] SKILL.md exists with YAML frontmatter
- [PASS] library-consumption.json exists
- [PASS] ui_kits/marketing/index.html exists
- [PASS] UIKit uses correct CSS link path (../../colors_and_type.css)

---

## Gate 5 (final validation)

- [PASS] All BOMs stripped from all files
- [PASS] Validator script exits with code 0
- [PASS] css.json regenerated after BOM strip
- [PASS] No agent-reports or temp files left in output dir
- [PASS] Git commit and push successful

---

## Validator usage

```powershell
node "scripts/validate-design-library-output.mjs" "{output_dir}"
```

## BOM stripping script (MUST run before validation)

```powershell
node -e "const fs=require('fs');const dir='{output_dir}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=d+'/'+e.name;if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',e.name)}}})}walk(dir);console.log('Done')"
```
