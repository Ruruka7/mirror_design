# Git Deployment Conventions

## Purpose

Standardize local Git commits, branch, remote-safety, and .gitignore rules for reverse-engineered design system output. Consistent conventions keep the repository history readable and prevent temp artifacts or unrelated workspace changes from leaking into commits.

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

## Local-first repository safety

- Treat the current workspace as the only default scope.
- Verify the repository root with `git rev-parse --show-toplevel` before staging.
- Never assume the current remote, repository owner, or default branch belongs to the user.
- Do not push by default. A validated local commit is the normal completion state.
- Stage only the generated output and explicitly related files; do not use `git add --all` blindly.

## Branch strategy

- Default: keep the current local branch; do not switch branches or create a remote branch without an explicit request.
- For major rework: suggest a local branch such as `refactor/{brandname}-tokens` only when useful; do not push it unless explicitly requested.
- Never invent `main`, branch names, remote names, or repository URLs.

## Explicit remote sync

Push only when the user explicitly requests it for this task and provides or confirms the target repository, remote, and branch. Before pushing:

1. Confirm the workspace root.
2. Inspect the configured remotes and current branch.
3. Match the requested repository, remote, and branch exactly.
4. Confirm that only intended commits are ahead.

If any target is unclear, stop and ask. Never push to `origin/main` as a default.

---

## .gitignore rules

- Ignore: `agent-reports/`, `*.tmp`, `phase0-*.txt`, `phase1-*.txt`, `ef_*.css`, `css_*.css`, `target_page.html`
- Do NOT ignore: `.design_library/`, `skills/`, `README.md`, `LICENSE`

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

## Git identity and remote checks

- For local-only work, no remote or SSH check is needed.
- If an explicit push is requested, verify the configured identity and target remote before pushing.
- If the explicit push fails with permission denied, check the SSH configuration and verify that the correct key is used.
