# Conventions

## File Naming

- All documentation files: `UPPER_CASE.md` for root-level docs (README, CHANGELOG, CLAUDE)
- Phase docs: `phase-X.Y-short-description.md` in `docs/phases/`
- ADRs: `NNN-short-title.md` in `docs/ADR/` (e.g. `001-use-markdown.md`)
- Test plans: `phase-X.Y-test-plan.md`
- User docs: `phase-X.Y-user-doc.md`

## Branch Naming

```
claude/<short-description>-<session-id>
feature/<phase>-<short-description>
fix/<issue-short-description>
chore/<task-short-description>
```

## Commit Messages

- Lowercase imperative: `feat: add .claudeignore`, `fix: correct typo in README`
- Prefix: `feat`, `fix`, `docs`, `chore`, `refactor`

## Language

- All technical documentation: English
- User-facing docs: match the target audience's language

## Markdown Style

- Use `##` for top-level sections within a file (never `#` — that's the title)
- Code blocks always specify language: ` ```bash `, ` ```markdown `
- Tables for structured comparisons
- Checkboxes (`- [ ]`) for actionable items
