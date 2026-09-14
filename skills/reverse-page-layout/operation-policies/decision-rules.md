# Decision Rules

## Purpose

This document defines the decision rules for edge cases encountered during the `reverse-page-layout` extraction workflow. When a phase fails to meet its gate criteria after 2 attempts, consult this document for the prescribed action.

The principle is: never abort the workflow entirely. Always produce as much output as possible and document any caveats in the final deliverable.

---

## Edge Case 1: SPA with Lazy-Loaded Sections

**Trigger:** Gate 0 fails because fewer than 3 sections are extracted, and the page is a Single Page Application (SPA) that loads content on scroll.

**Detection:**
- The page URL changes without full page reload (client-side routing).
- `documentHeight` is small on initial load but grows after scrolling.
- Section count is low (< 3) despite a visible full-page screenshot showing more content.
- Framework indicators in the DOM (`data-reactroot`, `__next`, `__nuxt`, Angular attributes).

**Action:**
1. Scroll down by one viewport height (`window.scrollBy(0, window.innerHeight)`).
2. Wait 1000ms for lazy-loaded content to render.
3. Repeat scroll + wait cycle 3 times total.
4. Re-run Script 1 (Page Section Structure).
5. Re-check Gate 0 criteria.
6. If sections are now >= 3, proceed normally.
7. If still < 3, fall through to Edge Case 3 (Minimal page).

**Caveats to log:**
- `"spaLazyLoadHandled": true` in `extraction-output.json`
- Note that some sections may have been loaded post-initial-render and may have different computed styles than they would at initial load.

---

## Edge Case 2: Multi-Page Site

**Trigger:** The extracted page is a homepage that links to distinct inner pages (e.g., a pricing page, a features page) with different section layouts that should be captured.

**Detection:**
- Navigation items link to different paths (e.g., `/pricing`, `/features`, `/about`).
- The homepage itself has < 5 sections but nav links suggest richer pages.
- The user or dispatching agent explicitly requested multi-page extraction.

**Action:**
1. Complete extraction on the homepage normally (Phase 0 through Phase 2).
2. Identify 1 inner page with the most distinct section types (prefer `/pricing` or `/features`).
3. Run Phase 0 extraction on the inner page.
4. Merge the inner page's sections into the section catalog.
5. For any new section type not already captured, generate a new template.
6. Do NOT overwrite homepage templates — only add new ones.
7. In `layout-system.json`, set `"multiPage": true` and `"innerPages": [url]`.

**Caveats to log:**
- `"multiPageExtracted": true` in the output manifest.
- Note which sections came from which page.

---

## Edge Case 3: Minimal Page (Few Sections)

**Trigger:** Gate 0 fails because fewer than 3 sections are extracted, and the page is NOT an SPA (no lazy loading detected).

**Detection:**
- Page is a simple landing page, coming soon page, or 404.
- `totalSections` is 1 or 2.
- No framework indicators.

**Action:**
1. Proceed with the available sections.
2. Generate templates only for the section types that exist.
3. In `page-sections.json`, set `"minimalPage": true`.
4. Skip Gate 0's minimum 3 sections requirement — override to accept >= 1 section.
5. Continue through all phases normally.

**Caveats to log:**
- `"minimalPage": true` in the output.
- Note the actual section count.

---

## Edge Case 4: No Interactions Detected

**Trigger:** Gate 3 fails because fewer than 2 interaction patterns are detected on the page.

**Detection:**
- The page is mostly static with no scroll animations, hover effects, or auto-rotating content.
- No CSS transition properties on interactive elements.
- No IntersectionObserver or scroll listener references.

**Action:**
1. Output an empty `patterns` array in `interaction-patterns.json`:
   ```json
   {
     "brand": "...",
     "patterns": [],
     "responsiveBehaviors": [...],
     "note": "No dynamic interaction patterns detected on this page."
   }
   ```
2. Still generate `responsiveBehaviors` (these are layout-based, not interaction-based).
3. Proceed to Phase 4 (Preview & Validate).
4. Override Gate 3's minimum 2 patterns requirement — accept 0 patterns with the note.

**Caveats to log:**
- `"noInteractionsDetected": true` in `interaction-patterns.json`.

---

## Edge Case 5: Responsive Testing Fails

**Trigger:** Script 4 (Responsive Breakpoints) fails at one or more widths, or the browser cannot resize to a specific width.

**Detection:**
- `browser_resize` returns an error for a specific width.
- The page throws JavaScript errors when resized.
- The layout breaks catastrophically at a specific width (e.g., all content disappears).

**Action:**
1. Note which breakpoints failed in the output.
2. Proceed with the breakpoints that succeeded.
3. If only desktop (1440px) succeeded:
   - Set `"responsiveTesting": "desktop-only"` in the extraction output.
   - In templates, include standard responsive media queries (768px, 375px) based on best practices rather than observed behavior.
   - Note in `interaction-patterns.json` `responsiveBehaviors` that the changes are inferred, not observed.
4. Continue through all phases.

**Caveats to log:**
- `"responsiveTesting": "partial"` or `"desktop-only"` in the extraction output.
- List which breakpoints failed and why.

---

## Edge Case 6: Non-Standard Navigation

**Trigger:** Script 3 (Navigation Structure) returns `found: false`.

**Detection:**
- No `<nav>`, `<header>`, `[role="banner"]`, or class-based nav elements found.
- The page may use a non-semantic structure (e.g., a `<div>` with navigation links).

**Action:**
1. Search for anchor tag clusters: find the topmost `<div>` or `<ul>` containing 3+ `<a>` tags.
2. Treat that element as the navigation.
3. If still not found, set `navigation: { found: false }` in the output.
4. Generate a `section-nav.html` template based on standard patterns (logo + links + CTA) even without extraction data.
5. Note in the output that the nav template is based on best practices, not extraction.

**Caveats to log:**
- `"navInferred": true` if the nav was reconstructed from non-semantic elements.
- `"navTemplateIsBestPractice": true` if no nav data was available at all.

---

## Edge Case 7: Page Requires Authentication

**Trigger:** The target URL redirects to a login page or returns a 401/403.

**Detection:**
- After navigation, the URL changes to a login/auth page.
- The page content contains login forms, "sign in" prompts, or access-denied messages.
- HTTP status indicates authentication is required.

**Action:**
1. Do NOT attempt to authenticate or enter credentials.
2. Extract whatever content is visible on the login/landing page.
3. If the login page itself has interesting layout (e.g., split login with branding), extract that.
4. Note `"authRequired": true` in the output.
5. Proceed with the available content.

**Caveats to log:**
- `"authRequired": true` in the extraction output.
- Note that the extracted content may be a login/gate page, not the intended content.

---

## Edge Case 8: Page Loads Very Slowly

**Trigger:** The page takes > 10 seconds to reach `networkidle`.

**Detection:**
- `networkidle` timeout.
- Many pending network requests.
- Page content is incomplete after the standard wait.

**Action:**
1. Wait an additional 10 seconds.
2. If still not idle, proceed with whatever has loaded.
3. Capture the screenshot at the current state.
4. Run extraction scripts on the available content.
5. Note `"slowLoad": true` and `"loadWaitTime": "20s"` in the output.

**Caveats to log:**
- `"slowLoad": true` in the extraction output.
- Note that some content may not have fully loaded.

---

## Fallback Policy

For any edge case NOT covered by the rules above:

1. **Never abort.** Produce as much output as possible.
2. **Log the issue.** Add a `caveats` array to `extraction-output.json` describing the unexpected situation.
3. **Use best judgment.** Apply standard patterns and reasonable defaults where extraction data is missing.
4. **Document assumptions.** Any inferred or defaulted values must be noted in the output metadata.
5. **Proceed through all phases.** Even with degraded data, generate templates, UX files, and preview.
