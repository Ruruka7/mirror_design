# Quality Gates

## Purpose

Define pass/fail criteria between each phase of the reverse-design-system workflow. Each gate must be satisfied before advancing to the next phase. Gates enforce extraction completeness, token integrity, component coverage, documentation completeness, and final validation.

---

## Gate 0 (after Phase 0 extraction)

- [PASS] At least 3 distinct colors found
- [PASS] At least 1 font-family identified
- [PASS] At least 5 font-size values found
- [PASS] At least 1 border or box-shadow pattern found
- [FAIL] If <3 colors → stop, report "sparse CSS", ask user for alternative URL or manual color input
- [FAIL] If no fonts found → proceed with system fallback but warn

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
node "{skill_base}/scripts/validate-design-library-output.mjs" "{output_dir}"
```

## BOM stripping script (MUST run before validation)

```powershell
node -e "const fs=require('fs');const dir='{output_dir}';function walk(d){fs.readdirSync(d,{withFileTypes:true}).forEach(e=>{const p=d+'/'+e.name;if(e.isDirectory()){walk(p)}else{let buf=fs.readFileSync(p);if(buf.length>=3&&buf[0]===0xEF&&buf[1]===0xBB&&buf[2]===0xBF){fs.writeFileSync(p,buf.slice(3));console.log('BOM stripped:',e.name)}}})}walk(dir);console.log('Done')"
```
