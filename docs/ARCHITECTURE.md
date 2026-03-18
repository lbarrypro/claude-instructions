# Architecture

## Overview

This repository is a **documentation-only project**. Its sole purpose is to distribute `CLAUDE.md` — a set of operating instructions for Claude AI sessions.

## Structure

```
/
├── CLAUDE.md              # Main operating instructions (source of truth)
├── README.md              # Project overview and usage
├── CHANGELOG.md           # Change history
├── .claudeignore          # Files excluded from Claude's context
├── tasks/
│   └── todo.md            # Work-in-progress tracking
└── docs/
    ├── ARCHITECTURE.md    # This file
    ├── CONVENTIONS.md     # Naming and style conventions for this repo
    ├── lessons.md         # Mistakes made and rules derived
    ├── RECAP.md           # Stakeholder-facing progress summary
    ├── STACK.md           # Tech stack (N/A — doc-only repo)
    ├── ADR/               # Architecture Decision Records
    └── phases/            # Per-phase documentation
```

## Design Decisions

- `CLAUDE.md` is the single source of truth for Claude behavior
- All other files in this repo serve to maintain the repo itself per the conventions defined in `CLAUDE.md`
- No code, no runtime dependencies — purely markdown

## Rollback

Any change is reversible via `git revert`. No deployment required.
