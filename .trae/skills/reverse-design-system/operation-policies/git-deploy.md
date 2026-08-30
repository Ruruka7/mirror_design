# Git Deployment Conventions

## Purpose

Standardize git commit, branch, and .gitignore rules for reverse-engineered design system output. Consistent conventions keep the repository history readable and prevent temp artifacts from leaking into commits.

---

## Commit message format (conventional commits)

New system:

```
feat: add {BrandName} design system reverse-engineered from {source-domain}
```

Example:

```
feat: add Endfield design system reverse-engineered from endfield.hypergryph.com
```

### When updating an existing system

```
fix: update {BrandName} {what-changed} based on {evidence}
```

Example:

```
fix: update Endfield button component clip-path based on re-extraction
```

### When adding skill/workflow files

```
feat: add reverse-design-system skill with full reverse-engineering workflow
```

---

## Branch strategy

- Default: work on `main` branch directly (personal repo)
- For major rework: create branch `refactor/{brandname}-tokens`
- Merge back to main after validation passes

---

## .gitignore rules

- Ignore: `agent-reports/`, `*.tmp`, `phase0-*.txt`, `phase1-*.txt`, `ef_*.css`, `css_*.css`, `target_page.html`
- Do NOT ignore: `.design_library/`, `.trae/skills/`, `README.md`, `LICENSE`

### Standard .gitignore content

```
# Agent reports and temp files
**/agent-reports/
*.tmp

# Phase 0 extraction artifacts
ef_*.css
css_*.css
target_page.html
endfield_page.html

# OS files
.DS_Store
Thumbs.db
```

---

## Pre-commit checklist

- [ ] All BOMs stripped
- [ ] Validator passed (exit code 0)
- [ ] No agent-reports directory in output
- [ ] No temp files (ef_*.css, css_*.css, etc.)
- [ ] css.json regenerated after final CSS changes

---

## Git identity

- Use the SSH-configured identity for the target repo
- Verify with: `ssh -T git@github.com` or `ssh -T git@github-{host-alias}`
- If push fails with permission denied: check SSH config, verify correct key is used
