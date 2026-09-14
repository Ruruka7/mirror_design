# Reverse Page Layout — Workflow

## Phase 0: Browser Extraction

**Agent:** Browser subagent
**Input:** Target URL
**Output:** Screenshot, section map JSON, computed styles summary

### Steps

1. Navigate to URL, wait for full render
2. Take full-page screenshot (save to workspace)
3. Extract page structure via `browser_evaluate`:
   ```javascript
   // Extract all top-level sections
   const sections = [...document.querySelectorAll('header, nav, section, footer, [role="banner"], [role="navigation"], [role="main"], [role="contentinfo"]')].map(el => ({
     tag: el.tagName.toLowerCase(),
     role: el.getAttribute('role'),
     className: el.className?.toString()?.slice(0, 100),
     textPreview: el.textContent?.slice(0, 200)?.replace(/\s+/g, ' '),
     rect: { w: el.offsetWidth, h: el.offsetHeight },
     children: el.children.length
   }));
   ```

4. Extract content inventory: headings, button labels, link text, image alt text
5. Record layout systems: grid templates, flex configurations, max-widths
6. Test responsive: capture at 1920px, 768px widths

### Gate 0

- [ ] Screenshot captured
- [ ] ≥3 page sections identified
- [ ] Content inventory extracted (headings, labels, links)
- [ ] Layout system recorded (grid/flex/max-width)

---

## Phase 1: Page Analysis

**Agent:** Main agent
**Input:** Phase 0 extraction data
**Output:** Section map, content outline

### Steps

1. Build section map: ordered list of page sections with type, content summary, and layout pattern
2. Extract real content text from the page (headings, body copy, button labels, navigation items)
3. Identify the page's narrative flow (what story does the page tell?)
4. Note any unique layout patterns (split sections, card grids, timelines, etc.)

### Gate 1

- [ ] Section map has ≥4 sections
- [ ] Real content extracted (not placeholder)
- [ ] Layout patterns identified

---

## Phase 2: Site Template Generation

**Agent:** Main agent or subagent
**Input:** Phase 1 section map + content
**Output:** `{BrandName}/site-template.html`

### Rules (CRITICAL)

1. **Single HTML file** — all sections in one file, not fragments
2. **CSS link:** `<link rel="stylesheet" href="colors_and_type.css">` (relative, same directory)
3. **Zero hardcoded visual values** — ALL colors, fonts, sizes, spacing use:
   ```css
   color: var(--color-foreground, #000000);
   background: var(--color-background, #ffffff);
   font-family: var(--font-body, sans-serif);
   padding: var(--space-6, 32px) var(--space-5, 24px);
   ```
4. **Use portable variable names** that exist in ALL token files:
   - Colors: `--color-background`, `--color-foreground`, `--color-muted-foreground`, `--color-surface`, `--color-card`, `--color-border`, `--color-primary`, `--color-on-primary`, `--color-primary-hover`, `--color-accent`, `--color-on-accent`
   - Fonts: `--font-display`, `--font-heading`, `--font-body`, `--font-mono`
   - Sizes: `--font-size-display/h1/h2/h3/h4/body/lead/caption/nav/button`
   - Weights: `--font-weight-display/h1/h2/h3/h4/body/lead/nav/button`
   - Spacing: `--space-1` through `--space-8`
   - Radius: `--radius-sm/md/lg/full`
   - Shadows: `--shadow-1` through `--shadow-5`
   - Sizing: `--max-content`, `--size-nav`, `--size-button-sm/md/lg`, `--size-icon-sm/md/lg`
5. **Always provide fallbacks** in `var()` — the template must render even without a token CSS file
6. **Image placeholders** use CSS gradients with variables:
   ```css
   background: linear-gradient(135deg, var(--color-muted, #f5f5f5), var(--color-border, #e5e5e5));
   ```
7. **Responsive** — include media queries for 1024px, 768px breakpoints
8. **Real content** — use the actual text from the source website (headings, body, labels)

### Gate 2

- [ ] Single HTML file created
- [ ] Zero hardcoded color values (grep for `#[0-9a-f]` in style — only in fallbacks allowed)
- [ ] Zero hardcoded font names (only in fallbacks)
- [ ] All sections from source page are present
- [ ] Content is real (not lorem ipsum)
- [ ] Responsive breakpoints exist
- [ ] CSS link is `href="colors_and_type.css"` (relative)

---

## Phase 3: Verification

**Agent:** Browser subagent
**Input:** `site-template.html`
**Output:** Screenshot of rendered template

### Steps

1. Open `site-template.html` in browser
2. Take screenshot
3. Verify: page renders without errors
4. Verify: all sections visible and properly styled
5. Test: swap CSS link to a different brand's tokens → verify restyle works

### Gate 3

- [ ] Page renders without console errors
- [ ] All sections visible
- [ ] Content readable
- [ ] Token swap test passed (different visual style after swap)

---

## Phase 4: Git Deploy

**Agent:** Main agent

### Steps

1. `git add .`
2. `git commit -m "feat: add {BrandName} site template — 1:1 from {source URL}"`
3. `git push origin main`

### Gate 4

- [ ] Commit pushed successfully
- [ ] File exists in remote repository
