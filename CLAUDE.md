# CLAUDE.md — Agent Operating Instructions

> This file defines how Claude should behave in this project. Read it fully at the start of every session.

---

## 0. Context Loading (Session Start)

Before doing anything, load the project context in this exact order (**macro → micro, static → dynamic**):

| # | File | Why |
|---|------|-----|
| 1 | `README.md` | Global vision, stack summary, how to run the project |
| 2 | `CHANGELOG.md` | History of changes, current state of the project |
| 3 | `docs/STACK.md` | Tech choices and their rationale in detail |
| 4 | `docs/ARCHITECTURE.md` | System design and global structure |
| 5 | `docs/ADR/` | Architectural decisions in chronological order (NNN) |
| 6 | `docs/API.md` | Contracts and endpoints (if touching the API) |
| 7 | `docs/CONVENTIONS.md` | Code conventions — read before writing anything |
| 8 | `docs/phases/` | Development phases in order, to understand the thread |
| 9 | `tasks/todo.md` | Current work in progress and plan state |
| 10 | `docs/lessons.md` | Past mistakes — read last to contextualize everything above |

- Check that `README.md` exists at the project root — if missing or empty, generate and propose it before starting work
- If any of these files don't exist yet, create them before starting work

### Existing Docs Migration Protocol

If `docs/` already contains markdown files that don't match the naming convention defined in this file, **do not rename or move anything silently**. Apply the following process:

1. **Inventory**: List all existing `.md` files found in `docs/` and read their content
2. **Analyse**: For each file, determine its likely purpose based on content
3. **Propose**: Present a migration plan to the user in this format:

```
Found X unrecognized doc(s) in docs/:

- notes.md → content seems to cover tech stack choices
  → Proposed action: merge into docs/STACK.md

- decisions.md → content spans architecture decisions and API contracts
  → Proposed action: split into docs/ADR/001-*.md + docs/API.md

- misc.md → purpose unclear
  → Question: what does this file cover? (stack / architecture / lessons / conventions / API / other?)
```

4. **Wait for confirmation**: Do not rename, move, merge, or delete any file until the user has validated the proposed plan
5. **Execute**: Apply the approved actions — merge content into the correct canonical files, then delete the originals
6. **Never silently overwrite**: If a target file already exists, show a diff of what would be added before writing

> **Rule**: When in doubt about a file's purpose, ask. One targeted question is better than a wrong assumption.

---

## 0.1 Profil Utilisateur

> Calibre le comportement de Claude pour cette session.

- **Rôle** : Développeur senior, vision produit forte, décisions fonctionnelles autonomes
- **Expertise technique** : Solide sur le code métier ; pas d'expertise DevOps, infra, sécurité, tests, architecture système
- **Attentes** :
  - Solutions clé en main sur les domaines non maîtrisés
  - Explications courtes, sans jargon infra
  - Toujours proposer avant d'agir — **c'est l'utilisateur qui valide, toujours**
  - Format préféré pour les décisions techniques : ADR pré-rempli à soumettre à validation

---

## 0.2 Domaines Techniques — Proposition obligatoire

Sur les domaines listés ci-dessous, Claude **propose une solution complète et argumentée**, puis **attend validation explicite** avant d'implémenter :

| Domaine | Ce que Claude fait |
|---------|-------------------|
| **DevOps / CI-CD** | Propose pipeline, config Docker, stratégie de déploiement |
| **Sécurité** | Propose audit OWASP, config headers, gestion secrets |
| **Architecture** | Propose patterns, découpage services, structure de dossiers |
| **Tests** | Propose stratégie, coverage cible, fixtures |
| **Performance** | Propose indexing, caching, pagination |
| **Base de données** | Propose schema design, migrations, indexing |
| **Dépendances** | Propose ajout/suppression de libs avec justification |

**Format de proposition systématique :**
```
Domaine : <DevOps / Sécurité / Architecture / ...>
Proposition : <ce que je propose de faire>
Pourquoi : <justification courte>
Impact : <ce que ça change, risques éventuels>
→ Valider pour continuer ?
```

> Règle absolue : aucune décision technique structurante n'est prise sans validation explicite de l'utilisateur.

---

## 1. Plan Node Default

- Enter plan mode for **ANY non-trivial task** (3+ steps or architectural decisions)
- Write the plan to `tasks/todo.md` with checkable items before touching any code
- If something goes sideways: **STOP and re-plan immediately** — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

---

## 2. Subagent Strategy

- Use subagents liberally to keep the main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

---

## 3. Self-Improvement Loop

- After **ANY correction** from the user: update `docs/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review `docs/lessons.md` at session start for the relevant project

---

## 4. Verification Before Done

- **Never mark a task complete without proving it works**
- Diff behavior between main and your changes when relevant
- Ask yourself: *"Would a staff engineer approve this?"*
- Run tests, check logs, demonstrate correctness
- See **Definition of Done** section below

---

## 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask *"is there a more elegant way?"*
- If a fix feels hacky: *"Knowing everything I know now, implement the elegant solution"*
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

---

## 6. Autonomous Bug Fixing

- When given a bug report: **just fix it.** Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

---

## 7. Coding Best Practices & Design Patterns

### General Principles

- Follow **SOLID** principles in all object-oriented code:
  - **S**ingle Responsibility: one class/module = one reason to change
  - **O**pen/Closed: open for extension, closed for modification
  - **D**ependency Inversion: depend on abstractions, not concretions
- Apply **DRY** (Don't Repeat Yourself): extract duplication into shared abstractions only when it appears 3+ times and the abstraction is stable
- Apply **KISS** (Keep It Simple): the simplest working solution is the right one until proven otherwise
- Apply **YAGNI** (You Aren't Gonna Need It): don't build for hypothetical futures

### Design Patterns — When to Use

Apply design patterns when they solve a real, present problem — not to demonstrate knowledge:

| Pattern | Use when |
|---------|----------|
| **Repository** | Abstracting data access from business logic |
| **Factory / Factory Method** | Object creation logic is complex or varies by context |
| **Strategy** | Multiple interchangeable algorithms or behaviors |
| **Observer / Event Emitter** | Decoupling producers from consumers of events |
| **Decorator** | Adding behavior to objects without subclassing |
| **Adapter** | Wrapping an incompatible interface into an expected one |
| **Command** | Encapsulating actions for undo/redo, queuing, or logging |
| **Singleton** | Exactly one instance needed (use sparingly — prefer DI) |
| **Builder** | Constructing complex objects step by step |
| **Middleware / Chain of Responsibility** | Processing pipelines with pluggable steps |

### Rules

- **Name it**: if you apply a pattern, name it explicitly in the code (class name, file name, or comment) so it's recognizable
- **Justify it**: if a pattern adds indirection, it must earn its complexity — document why in the relevant ADR or phase doc
- **Don't force it**: a plain function or simple class is better than a pattern applied for its own sake
- **Prefer composition over inheritance** in all cases where both could work
- **Code review question**: before finalizing, ask *"does this code read like idiomatic [language] to a senior engineer?"*

---

## 8. Robustness, Performance & Resource Management

### Memory Management

- **No memory leaks**: every resource opened must be closed — use `finally`, `using`, `defer`, context managers, or RAII depending on the language
- **Avoid unbounded collections**: cap queues, lists, and maps that grow over time; set explicit `maxSize` limits
- **Prefer streaming over buffering**: for large data (files, API responses), process in chunks rather than loading everything into memory
- **Release references explicitly** when objects are long-lived (event listeners, timers, subscriptions) — unsubscribe and clear on teardown
- **Use weak references** (`WeakMap`, `WeakRef`) for caches or secondary indexes that should not prevent garbage collection
- **Profile before optimizing**: don't guess at memory pressure — measure with heap snapshots, memory profilers, or APM tools

### Caching Strategy

- **Cache at the right layer**: distinguish L1 (in-process), L2 (shared/Redis), L3 (CDN/HTTP) — pick the layer that matches the data's scope and lifetime
- **Always define TTL and eviction policy**: no cache entry lives forever; LRU is the default eviction strategy unless data has natural expiry
- **Cache-aside by default**: read from cache → on miss, load from source → populate cache — never write-through unless consistency is critical
- **Invalidate deliberately**: cache invalidation must be an explicit design decision, not an afterthought — document the strategy in the relevant ADR
- **Cache only stable, expensive data**: don't cache what's cheap to compute or changes on every request
- **Key design matters**: cache keys must be deterministic, scoped, and versioned (include schema version or hash when relevant) to avoid stale reads after deploys
- **Never cache errors** unless intentional (negative caching) — always document if you do

### Resilience & Error Handling

- **Fail fast, recover gracefully**: validate inputs at boundaries, surface errors early, provide meaningful fallbacks downstream
- **Apply Circuit Breaker** for calls to external services — stop hammering a failing dependency, return a degraded response instead
- **Timeouts everywhere**: every network call, DB query, and external API call must have an explicit timeout — never rely on default or infinite
- **Retry with exponential backoff + jitter** for transient failures; cap total retry attempts and always log each attempt
- **Bulkhead isolation**: isolate resource pools (thread pools, connection pools) per subsystem so one slow dependency doesn't starve the others
- **Graceful degradation**: design features to work partially when dependencies are unavailable (cached data, reduced functionality, safe defaults)

### Performance

- **Measure first**: instrument before optimizing — use tracing, profiling, and benchmarks to confirm where the bottleneck actually is
- **Avoid N+1 queries**: batch or eager-load related data at the data access layer; use dataloaders or join queries where appropriate
- **Lazy load by default, eager load when measured**: don't load data until needed, but flip to eager when profiling proves it's faster
- **Paginate everything**: no endpoint or query returns unbounded result sets — enforce a default and maximum page size
- **Async for I/O, sync for CPU**: use async/non-blocking I/O for network and disk; use workers/threads for CPU-intensive work
- **Connection pooling**: always use pooled connections for databases and HTTP clients — never open a raw connection per request

### Observability (Robustness enabler)

- **Structured logging**: log JSON (or equivalent) with consistent fields — `level`, `timestamp`, `traceId`, `service`, `message`
- **Log at boundaries**: log at every entry/exit of a significant operation (API request, job start/end, cache hit/miss, external call)
- **Metrics for what matters**: track error rates, latency percentiles (p50/p95/p99), cache hit ratio, queue depth — not just "is it up"
- **Distributed tracing**: propagate trace IDs across service calls; correlate logs and spans with a single `traceId`
- **Alerts on symptoms, not causes**: alert on user-visible impact (error rate spike, latency degradation) rather than internal signals alone

### Rules

- Any new feature touching data access, external I/O, or shared state **must include a cache/memory strategy decision** — document it, even if the decision is "no cache needed because X"
- Performance-sensitive paths must have a benchmark or load test before shipping
- When in doubt between memory efficiency and readability: **readability wins** unless profiling proves otherwise — premature optimization is a bug

---

## 9. Security by Design

### Secrets Management
- **Never commit secrets**: no API keys, tokens, passwords in code or git history — ever
- `.env.example` must always be up to date; `.env` must be in `.gitignore`
- Use a secret manager (Vault, AWS Secrets Manager, Doppler) in production — never rely on `.env` files
- Rotate secrets immediately if accidentally exposed

### OWASP Top 10 — Always check before shipping
- **Injection** (SQL, command, LDAP): use parameterized queries and ORM — never string-concatenate user input into queries
- **Broken Auth**: use short-lived JWTs, secure httpOnly cookies, always hash passwords with bcrypt/argon2
- **Sensitive Data Exposure**: never log PII, tokens, or passwords — mask in all outputs
- **XSS**: sanitize and escape all user input rendered in HTML; set Content-Security-Policy headers
- **CSRF**: use CSRF tokens or SameSite cookies for state-changing requests
- **Security Misconfiguration**: disable debug mode in prod, set security headers (HSTS, X-Frame-Options, etc.)
- **Vulnerable Dependencies**: run `npm audit` / `pip audit` / equivalent before every release

### Principle of Least Privilege
- Every service, user, and process gets only the permissions it needs — nothing more
- Database users must not have DDL rights in production
- API keys scoped to minimum required endpoints

### Input Validation
- Validate all inputs at the boundary (schema validation, type coercion) — trust nothing from outside
- Reject unknown fields; enforce max lengths and type constraints
- Return generic error messages to clients; log the real cause server-side

### Dependency Auditing
- Run security audits as part of CI — block merge on HIGH/CRITICAL vulnerabilities
- Review all new dependencies before adding (stars, last commit, maintainers)
- Prefer well-maintained, widely-used libraries over niche alternatives

---

## 10. Testing Strategy

### The Pyramid
- **Unit tests** (base, most tests): pure functions, business logic, isolated from I/O
- **Integration tests** (middle): test real interactions — DB, cache, message queue
- **E2E tests** (top, fewest): critical user journeys only — expensive to write and maintain

### Rules
- **Coverage threshold**: aim for ≥80% on business logic; don't chase 100% — test behavior, not implementation
- **No mocking the DB in integration tests**: use a real test DB or transaction rollbacks — mock/prod divergence hides bugs
- **Test naming**: `it("should <behavior> when <condition>")` — tests document intent
- **Arrange-Act-Assert**: structure every test explicitly in three blocks
- **One assertion per test** (where practical): multiple assertions → multiple tests with focused names
- **Test data via factories/fixtures**: never hardcode IDs or dates; use deterministic seeds
- **Don't test framework code**: don't write tests that only verify the ORM or the router work — test your logic
- **Regression tests**: every bug fixed must have a test that reproduces it first

---

## 11. CI/CD

### Required Gates (must pass before merge)
1. **Lint** — no warnings, no style violations
2. **Type check** — no type errors (if typed language)
3. **Unit tests** — 100% pass
4. **Integration tests** — 100% pass
5. **Security audit** — no HIGH/CRITICAL vulnerabilities
6. **Build** — artifact builds successfully

### Environment Strategy
- **local → staging → production** — no direct-to-prod deploys
- Staging must mirror production config (same infra, same secrets shape, same data shape)
- Deploy to staging on every merge to `develop`; deploy to prod only from tagged releases

### Zero-Downtime Deploys
- Use rolling deploys or blue/green — never take the service down to deploy
- Database migrations must be backward-compatible with the previous version of the code
- Deploy code first, then run migrations — never the reverse

### Rollback Strategy
- Every deploy must be reversible in under 5 minutes
- Keep previous Docker image / artifact available for at least 2 versions back
- Document the rollback procedure in `docs/ARCHITECTURE.md`

---

## 12. Dependency Management

- **Always commit lock files** (`package-lock.json`, `poetry.lock`, etc.) — reproducible builds are non-negotiable
- **Pin exact versions** in production apps; use ranges only in libraries
- **Review before adding**: before adding a new dependency, ask — can this be done in 20 lines without it?
- **Update regularly**: run dependency updates in a dedicated `chore/deps-YYYY-MM` branch monthly
- **Audit on every PR**: CI must run security audits automatically
- **Remove unused deps**: dead dependencies are attack surface — delete them

---

## 13. Database & Migrations

- **Never modify a migration after it's merged** — create a new one instead
- **Backward-compatible migrations**: new columns must have defaults or be nullable — old code must still run against the new schema
- **Separate deploy from migrate**: code deploy and migration run are independent steps — never couple them
- **No data migrations in schema migrations**: schema changes and data backfills are separate migration files
- **Index before constraint**: add indexes before adding foreign key constraints on large tables
- **Document destructive operations**: dropping a column or table requires an ADR — irreversible actions need deliberate sign-off
- **Seed data is code**: seed scripts must be idempotent (safe to run multiple times)

---

## 14. API Versioning & Backward Compatibility

- **Semver for all public contracts**: MAJOR = breaking, MINOR = additive, PATCH = fix
- **Never remove or rename a field without a deprecation period** — add the new field, deprecate the old, remove after N releases
- **Version in the URL** (`/v1/`, `/v2/`) for REST APIs with breaking changes
- **Deprecation header**: return `Deprecation: true` and `Sunset: <date>` headers on deprecated endpoints
- **Contract tests**: if you have consumers (internal or external), use contract tests (Pact or equivalent) to catch breaking changes before merge
- **Changelog entry for every API change** — consumers must know what changed

---

## 15. Technical Debt

- **Document debt, don't hide it**: use `// TODO(username): explanation` with a link to a tracking issue
- **No silent workarounds**: if a fix is temporary, mark it explicitly — `// TEMP: reason, remove after X`
- **Debt review**: include a debt review item in every phase — identify what was cut and track it
- **Repay debt before it compounds**: a debt item that blocks 3+ features must be addressed in the next phase
- **The boy scout rule**: leave the code slightly better than you found it — one small cleanup per PR is acceptable

---

## 16. Release Process

- **Tag every release**: `git tag v1.2.3` on `main` after merge — tags are immutable
- **Release notes** = CHANGELOG section for that version — no separate document needed
- **Hotfix process**: `fix/hotfix-<issue>` branched from `main`, merged back into both `main` and `develop`
- **Release checklist**:
  - [ ] All CI checks green on `develop`
  - [ ] CHANGELOG `[Unreleased]` section promoted to new version
  - [ ] `README.md` version badge updated if applicable
  - [ ] Staging deploy validated
  - [ ] Tagged and pushed
  - [ ] Production deploy + smoke test

---

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `docs/lessons.md` after corrections

---

## Git Workflow

### Mode PoC

Si le projet est en phase **PoC / Prototype** :

- Commit direct sur `main` autorisé
- Pas de branche `develop`, pas de PR obligatoire
- Documentation minimale : `README.md` + `tasks/todo.md` suffisent
- **Dès qu'un PoC passe en "produit"** (validation, premier utilisateur réel, ou décision de continuer) → appliquer le workflow complet ci-dessous sans exception

> Indiquer explicitement le mode du projet dans `README.md` : `**Mode : PoC**` ou `**Mode : Produit**`.

---

### Branch Strategy

- Main branches: `main` (production), `develop` (integration)
- **Every development phase or sub-phase must have its own branch**, created from `develop`:

```
develop → feature/<phase>-<short-description>
develop → fix/<issue-short-description>
develop → chore/<task-short-description>
```

Examples:
```
feature/phase1-auth-setup
feature/phase2-1-api-layer
fix/phase2-token-refresh
```

- **Never commit directly to `develop` or `main`**
- Merge only via Pull Request, after review and all checks passing

### Pull Request Requirements

Each PR must include:

**Title**: `[Phase X.Y] Short description of what was done`

**Body checklist**:
```markdown
## Summary
<!-- What was built, why, and how -->

## Phase Documentation
- [ ] `docs/phases/phase-X.Y-*.md` created and up to date
- [ ] `docs/phases/phase-X.Y-test-plan.md` created, all test cases executed and status filled
- [ ] `docs/phases/phase-X.Y-user-doc.md` created and covers all user-visible features
- [ ] `docs/RECAP.md` updated — PO section and CTO section filled for this phase
- Link: docs/phases/phase-X.Y-short-description.md

## Changes
- [ ] List of files modified and reason
- [ ] Any schema / API contract changes

## Tests
- [ ] Unit tests written and passing
- [ ] Integration tests written and passing
- [ ] Edge cases covered
- [ ] Test output / coverage attached or linked

## Security
- [ ] No secrets committed
- [ ] Inputs validated at boundaries
- [ ] `npm audit` / equivalent run — no HIGH/CRITICAL issues
- [ ] OWASP Top 10 self-review done (if touching auth, DB, or user input)

## Definition of Done
- [ ] Feature works as specified
- [ ] No regressions on existing tests
- [ ] Code reviewed for elegance (no hacky fixes)
- [ ] `docs/phases/phase-X.Y-*.md` finalized (status set to Done)
- [ ] `CHANGELOG.md` updated with relevant entries
- [ ] `README.md` updated if setup, structure, or public interface changed
- [ ] Relevant `docs/` files updated if needed
- [ ] `tasks/todo.md` updated
- [ ] `docs/lessons.md` updated if a mistake was caught

## Screenshots / Logs (if applicable)
```

---

## Project Documentation (`docs/`)

All technical decisions and reference documents live in `docs/`. Never scatter architecture decisions in code comments or chat history.

| File | Purpose |
|------|---------|
| `docs/STACK.md` | Tech stack, chosen libraries, hosting, rationale |
| `docs/ARCHITECTURE.md` | High-level system design, diagrams |
| `docs/ADR/` | Architecture Decision Records (one file per decision) |
| `docs/lessons.md` | Mistakes made + rules derived to prevent recurrence |
| `docs/CONVENTIONS.md` | Naming conventions, code style, folder structure |
| `docs/API.md` | API contracts, endpoints, payload shapes |
| `docs/phases/` | One documentation file per development phase (see below) |
| `docs/phases/phase-X.Y-test-plan.md` | Cahier de tests par phase (cas, données, résultats attendus) |
| `docs/phases/phase-X.Y-user-doc.md` | Documentation utilisateur par phase (guides, flux, captures) |
| `docs/RECAP.md` | Récapitulatif vivant pour le PO (livraisons) et le CTO (points techniques) |
| `CHANGELOG.md` | Chronological log of all significant changes (root of project) |
| `README.md` | Project overview, setup instructions, usage (root of project) |

### README

A `README.md` must exist at the **root of every project**. If it doesn't exist, Claude must create it before any other work begins.

**At session start**: if `README.md` is missing or empty, generate it from available context (project name, existing code, `docs/STACK.md`, `docs/ARCHITECTURE.md`, etc.) and propose it to the user for review before writing it.

**Required structure:**

````markdown
# Project Name

> One-line description of what this project does and for whom.

## Overview
Brief explanation of the project's purpose, context, and main features (3–5 sentences max).

## Tech Stack
Main technologies used — link to `docs/STACK.md` for details.

## Prerequisites
What needs to be installed before running the project (Node version, env vars, etc.).

## Getting Started
```bash
# Clone
git clone ...

# Install dependencies
npm install

# Configure environment
cp .env.example .env

# Run locally
npm run dev
```

## Project Structure
```
src/
docs/
tasks/
```
Brief explanation of the main folders.

## Documentation
- [Stack & Architecture](docs/STACK.md)
- [Architecture Decisions](docs/ADR/)
- [API Reference](docs/API.md)
- [Conventions](docs/CONVENTIONS.md)
- [Changelog](CHANGELOG.md)

## Contributing
Branch naming, PR process — see Git Workflow in `CLAUDE.md`.

## License
````

**Rules:**
- `README.md` is a living document — update it whenever the setup, structure, or features change significantly
- Keep it concise and developer-facing: no marketing copy, no fluff
- If information already exists in `docs/`, link to it rather than duplicating it
- Update the README as part of any PR that changes the public interface, setup steps, or project structure

### CHANGELOG

A `CHANGELOG.md` must be maintained at the **root of every project**, following the [Keep a Changelog](https://keepachangelog.com) format:

```markdown
# Changelog

All notable changes to this project will be documented here.
Format: [Keep a Changelog](https://keepachangelog.com) — [Semantic Versioning](https://semver.org)

## [Unreleased]

## [1.2.0] - YYYY-MM-DD
### Added
- ...
### Changed
- ...
### Fixed
- ...
### Removed
- ...
```

**Rules:**
- Update `CHANGELOG.md` as part of **every PR** — never retroactively
- Each merged PR maps to at least one entry under the appropriate section (`Added`, `Changed`, `Fixed`, `Removed`, `Security`)
- The `[Unreleased]` section accumulates entries until a version is tagged
- Never leave `CHANGELOG.md` empty or stale — it is a first-class project artifact

### Per-Phase Documentation (`docs/phases/`)

Every development phase (and significant sub-phase) must have a dedicated documentation file created **before the PR is opened**:

**Naming**: `docs/phases/phase-X.Y-short-description.md`

Examples:
```
docs/phases/phase-1-project-bootstrap.md
docs/phases/phase-2-1-api-layer.md
docs/phases/phase-2-2-auth-integration.md
```

**Required structure:**

```markdown
# Phase X.Y — Short Description

**Status**: In Progress | Done | Abandoned
**Branch**: `feature/phaseX-Y-short-description`
**PR**: #XX (link once created)
**Date**: YYYY-MM-DD

## Goal
What this phase aims to deliver and why.

## Scope
- What is included
- What is explicitly excluded (avoids scope creep)

## Technical Decisions
Key choices made during this phase (link to ADRs if relevant).

## Implementation Notes
Anything worth knowing for future reference: gotchas, non-obvious choices, workarounds.

## Tests
How the phase was validated. What was covered, what was left out and why.
Link to test plan: `docs/phases/phase-X.Y-test-plan.md`

## User Documentation
Summary of what was documented for end users.
Link to user doc: `docs/phases/phase-X.Y-user-doc.md`

## Result
What was actually delivered. Differences from the initial goal, if any.

## CHANGELOG entries
Entries added to CHANGELOG.md as part of this phase.
```

**Rules:**
- The phase doc is created at branch creation, filled progressively, and finalized before the PR is merged
- The PR description links to the corresponding `docs/phases/phase-X.Y-*.md` file
- **A cahier de tests (`phase-X.Y-test-plan.md`) and a user doc (`phase-X.Y-user-doc.md`) are mandatory companion files** — created before implementation begins, completed before the PR is merged
- If a phase is abandoned or pivoted, document the reason in the file and set status to `Abandoned`

### Cahier de Tests (`docs/phases/phase-X.Y-test-plan.md`)

Every phase and sub-phase must produce a test plan written **before implementation starts** and completed **before the PR is merged**.

**Naming**: `docs/phases/phase-X.Y-test-plan.md`

**Required structure:**

```markdown
# Phase X.Y — Cahier de Tests

**Phase**: X.Y — Short Description
**Status**: Draft | In Progress | Validated
**Date**: YYYY-MM-DD

## Scope
What features / user stories are covered by this test plan.

## Test Environment
- Runtime version, OS, browser(s) if applicable
- Test database / fixtures strategy
- Required environment variables or configuration

## Test Cases

### TC-001 — <Short test case name>
| Field | Value |
|-------|-------|
| **Category** | Unit \| Integration \| E2E \| Manual |
| **Preconditions** | State required before the test runs |
| **Input / Steps** | Step-by-step actions or input data |
| **Expected Result** | Exact expected outcome |
| **Actual Result** | (filled during execution) |
| **Status** | Pass \| Fail \| Blocked \| Skipped |
| **Notes** | Observations, links to failing logs, etc. |

<!-- Repeat for each test case -->

## Edge Cases & Negative Tests
List of boundary conditions, invalid inputs, and error scenarios covered.

## Non-Regression
List of existing test suites that must still pass after this phase:
- [ ] `<test file or suite name>` — reason it could be affected

## Coverage Summary
| Layer | Target | Achieved |
|-------|--------|----------|
| Unit | ≥80% | — |
| Integration | key paths | — |
| E2E | critical journeys | — |

## Known Gaps
What was NOT tested and why (time constraint, dependency, out of scope).
```

**Rules:**
- Every feature added or modified in the phase must have at least one test case entry
- Negative / error-path test cases are mandatory for any user-facing or API-facing feature
- Mark test cases with `Status: Fail` if they do not pass — never delete failing cases before the PR
- The Coverage Summary must be filled before marking the phase `Done`

---

### Documentation Utilisateur (`docs/phases/phase-X.Y-user-doc.md`)

Every phase that delivers user-visible functionality must produce a user documentation file **before the PR is merged**.

**Naming**: `docs/phases/phase-X.Y-user-doc.md`

**Required structure:**

```markdown
# Phase X.Y — Documentation Utilisateur

**Phase**: X.Y — Short Description
**Audience**: End user \| Admin \| Developer \| All
**Status**: Draft | Review | Published
**Date**: YYYY-MM-DD

## Overview
What this feature does and who it is for (2–4 sentences, no jargon).

## Prerequisites
What the user needs before using this feature (account, permissions, installed tools, etc.).

## Getting Started
Step-by-step guide to use the feature for the first time.

1. Step one — brief explanation
2. Step two — brief explanation
3. ...

> Include screenshots, CLI output, or UI mockups where helpful.

## Features & Usage

### <Feature or screen name>
Description of what this feature/screen does.

**How to use:**
1. ...
2. ...

**Expected result:** ...

<!-- Repeat per feature -->

## Common Workflows
Describe the most frequent end-to-end flows a user will follow.

### Workflow: <Name>
1. ...
2. ...
3. ...

## Error Messages & Troubleshooting

| Error / Symptom | Cause | Resolution |
|-----------------|-------|------------|
| "..." | ... | ... |

## FAQ
**Q: ...**
A: ...

## Known Limitations
What the feature does not do yet (and the phase or ticket where it will be addressed).

## Related Documentation
- Link to API doc if relevant
- Link to other phase user docs if chained
```

**Rules:**
- Written in the target user's language (match the project's language setting)
- No internal jargon, implementation details, or class names — write for the persona who will use the feature
- Screenshots or annotated CLI output are required for any UI or interactive workflow
- If the feature has no user-visible surface (e.g. a pure infra change), write a minimal doc explaining impact on existing behaviour and link it in the phase file
- Updated whenever the feature's behaviour changes in a subsequent phase

---

### Recap Stakeholders (`docs/RECAP.md`)

`docs/RECAP.md` is a **living document** updated at the end of every phase and sub-phase. It consolidates the project's progress into two audience-specific views: one for the Product Owner, one for the CTO.

**Rules:**
- Created at project bootstrap (even if initially empty) and kept up to date throughout the project
- Updated **before closing the PR** for each phase — it is part of the Definition of Done
- Written in the project's language (match the user-facing docs language)
- The PO section must be jargon-free — no class names, no infrastructure details
- The CTO section must be precise and honest — surface risks, debt, and open questions explicitly
- **For each phase, explicitly identify what can be demonstrated to management (direction)** — a working UI flow, a feature end-to-end, a report, etc. If nothing is demo-ready (pure infra phase), state it explicitly with a one-line explanation. This field is mandatory in the PO section.

**Naming**: `docs/RECAP.md` (single file, cumulative — never one file per phase)

**Required structure:**

```markdown
# Project Recap

> Last updated: YYYY-MM-DD — Phase X.Y

---

## Product Owner View

> What has been built, phase by phase. Written in plain language, focused on user value and delivery.

### Phase 1 — <Short name>
**Status**: Done | In Progress | Abandoned
**Delivered:**
- <User-facing feature or capability, one bullet per item>
- ...

**Not delivered / deferred:**
- <What was descoped and why — one line each>

**Notable changes vs. initial scope:**
- <Any pivot, cut, or addition vs. the original goal>

**Démo direction :**
- <Ce qui peut être montré à la direction : flux UI, feature bout-en-bout, rapport, etc.>
- <Si rien n'est démontrable (phase infra pure) : l'indiquer explicitement en une ligne>

<!-- Repeat one ### block per phase and sub-phase -->

---

## CTO View

> Technical highlights, architectural decisions, risks, and debt. One entry per phase. Written for a senior engineer who needs the full picture fast.

### Phase 1 — <Short name>
**Key technical decisions:**
- <Decision taken and rationale — link to ADR if applicable>
- ...

**Risks & open questions:**
- <Any known technical risk, assumption, or unresolved question>

**Technical debt introduced:**
- <What was cut or deferred, with a TODO/ticket reference>

**Performance / security notes:**
- <Anything impacting latency, throughput, attack surface, or compliance>

**Breaking changes:**
- <Schema changes, API contract changes, config changes — anything that could break consumers>

<!-- Repeat one ### block per phase and sub-phase -->
```

---

### ADR Format (`docs/ADR/NNN-title.md`)

```markdown
# ADR-NNN: Title

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Deprecated

## Context
Why is this decision needed?

## Decision
What was decided?

## Consequences
What are the trade-offs?
```

---

## Definition of Done

A task is **done** only when ALL of the following are true:

- [ ] The feature/fix behaves exactly as specified
- [ ] Tests are written, passing, and pushed to the branch
- [ ] No existing tests are broken
- [ ] Security checklist reviewed (OWASP Top 10, no secrets committed, inputs validated)
- [ ] No HIGH/CRITICAL vulnerabilities in dependencies (`npm audit` / equivalent)
- [ ] The PR is created with the full checklist filled out
- [ ] `docs/phases/phase-X.Y-*.md` exists, is complete, and status is set to `Done`
- [ ] `docs/phases/phase-X.Y-test-plan.md` exists, all test cases filled, Coverage Summary completed
- [ ] `docs/phases/phase-X.Y-user-doc.md` exists and covers all user-visible features of the phase
- [ ] `docs/RECAP.md` updated — PO section lists all deliverables, CTO section covers decisions, risks, and debt for this phase
- [ ] `CHANGELOG.md` has been updated with at least one entry for this phase
- [ ] Relevant `docs/` files are updated
- [ ] `tasks/todo.md` reflects completion
- [ ] `README.md` updated if setup, structure, or features changed

---

## Hooks — Automatismes de session

Les hooks Claude Code sont configurés dans `.claude/settings.json`. Ils s'exécutent automatiquement sur des événements de session.

### Hooks actifs

| Événement | Déclencheur | Action |
|-----------|-------------|--------|
| `PostToolUse` | Après `Edit` ou `Write` | Rappel de vérifier lint/tests si un linter est configuré |
| `PreToolUse` | Avant `Bash` avec commandes destructives | Avertissement avant `rm -rf`, `drop`, `reset --hard`, `force push` |
| `Stop` | Fin de session | Rappel de mettre à jour `tasks/todo.md` et `docs/lessons.md` |

### Règles d'intégration dans le process

- Si un hook bloque une action : **ne pas contourner** (`--no-verify`, etc.) — investiguer la cause
- Si un hook échoue sur le lint : corriger avant de continuer, ne pas ignorer
- Le hook `Stop` est un filet de sécurité — ne pas attendre qu'il le rappelle pour mettre à jour les docs

> La config complète des hooks est dans `.claude/settings.json` à la racine du projet.

---

## Skills — Commandes réutilisables

Les skills sont des commandes slash personnalisées définies dans `.claude/commands/`. Ils peuvent être invoqués par Claude (subagents inclus) ou par l'utilisateur.

### Skills disponibles

| Commande | Usage |
|----------|-------|
| `/new-feature <description>` | Initialise une nouvelle feature : plan, docs de phase, test plan, user doc |
| `/prep-pr` | Génère le corps de PR complet avec checklist remplie |
| `/review-security` | Audit OWASP sur les fichiers modifiés dans la session courante |
| `/debt-review` | Liste tous les `TODO`, `TEMP`, `FIXME` avec contexte et ticket suggéré |
| `/explain-tech <terme>` | Explique un concept technique en langage produit, sans jargon infra |

> Les skills sont définis dans `.claude/commands/<nom>.md`. Tout subagent peut les invoquer.

### Skills et agents spécialisés par projet

Chaque projet a ses propres besoins. Au démarrage d'un nouveau projet (ou à la demande), Claude **analyse les spécifications et crée les skills et agents adaptés** à ce contexte précis.

**Catalogue de rôles spécialisés :**

#### Rôles universels (tout projet)

| Rôle / Agent | Ce qu'il fait |
|--------------|---------------|
| **Architecte logiciel** | Conçoit la structure globale, découpage en services, choix des patterns |
| **CTO** | Évalue les risques techniques, la dette, les décisions structurantes, la roadmap tech |
| **Product Owner** | Rédige les specs fonctionnelles, les user stories, les critères d'acceptance |
| **DevOps** | Configure CI/CD, Docker, déploiements, monitoring, rollback |
| **Senior Backend** | Implémente la logique métier, les APIs, les accès données |
| **Senior Frontend** | Implémente les interfaces, la gestion d'état, les intégrations API |
| **Expert UI/UX** | Conçoit les flows utilisateurs, la hiérarchie visuelle, l'accessibilité |
| **Testeur qualité (QA)** | Rédige les cahiers de tests, exécute les scénarios, identifie les régressions |
| **Expert sécurité** | Audite le code, les dépendances, les configurations, les accès |
| **Expert performance** | Profile, identifie les goulots, propose les optimisations mesurées |
| **DBA (Expert base de données)** | Schema design, query optimization, stratégie de migration — distinct du backend qui code |
| **Technical Writer** | Rédige la doc API, guides d'intégration, READMEs — pour les développeurs consommateurs |
| **Expert conformité / RGPD** | Consentement, rétention, droit à l'oubli, mentions légales — dès qu'il y a des données utilisateurs |

#### Rôles fréquents selon la stack

| Rôle / Agent | Ce qu'il fait |
|--------------|---------------|
| **Expert intégrations tierces** | APIs externes, webhooks, OAuth, Stripe — contrats, idempotence, retry strategies |
| **Expert mobile (iOS / Android / RN)** | Navigation, permissions, offline, push notifications |
| **Expert accessibilité (a11y)** | WCAG, lecteurs d'écran, navigation clavier, contrastes — distinct de l'UX |
| **Expert i18n / l10n** | Traductions, formats dates/monnaies, RTL — projets multi-langues |

#### Rôles spécialisés par domaine

| Rôle / Agent | Ce qu'il fait |
|--------------|---------------|
| **Data Engineer / Analyste** | Pipelines de données, reporting, dashboards, ETL, modélisation analytique |
| **Expert IA / LLM** | Prompting, RAG, fine-tuning, évaluation de sorties — projets intégrant des modèles |
| **FinOps / Expert coût cloud** | Optimise les coûts AWS/GCP/Azure — intervient dès qu'il y a de l'infra cloud |
| **Expert SEO technique** | SSR, balises meta, Core Web Vitals, sitemap, structured data — projets marketing/contenu |
| **Expert monitoring / observabilité** | Instrumente Datadog, Grafana, alerting — distinct du DevOps qui pipeline |
| **Expert infrastructure / Cloud Architect** | VPC, IAM, scaling, disaster recovery — infra complexe, distinct du DevOps |

#### Stack de départ recommandée (projet SaaS type)

Au démarrage d'un projet SaaS, proposer systématiquement ces rôles :

```
Architecte logiciel, Senior Backend, Senior Frontend,
DBA, DevOps, QA, Expert sécurité, Expert conformité/RGPD,
Expert intégrations tierces, Technical Writer
```

Les autres rôles s'ajoutent au besoin selon les specs du projet.

**Règle de création :**

Au démarrage d'un projet ou d'une phase significative, Claude doit :

1. **Analyser les specs** du projet (stack, domaines fonctionnels, contraintes)
2. **Identifier les rôles nécessaires** parmi les exemples ci-dessus ou en créer de nouveaux si le projet l'exige
3. **Proposer la liste des skills et agents** à créer, avec leur périmètre exact
4. **Attendre validation** avant de créer quoi que ce soit
5. **Créer les fichiers** dans `.claude/commands/<projet>/<nom>.md` pour les skills, et documenter les agents dans `docs/ARCHITECTURE.md`

**Naming convention :**
```
.claude/commands/<projet>/<role>-<action>.md

Exemples :
.claude/commands/myapp/cto-risk-review.md
.claude/commands/myapp/qa-test-plan.md
.claude/commands/myapp/ux-flow-review.md
```

> Un agent spécialisé est un subagent auquel on donne un contexte de rôle précis, un périmètre strict, et les skills correspondants. Il ne doit jamais sortir de son périmètre sans escalader à l'utilisateur.

---

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Docs are truth**: If it's not in `docs/`, it doesn't exist as a decision.
- **Tests are proof**: If it's not tested, it's not done.
- **Patterns serve the code**: Apply design patterns and best practices (SOLID, DRY, KISS) when they reduce complexity — never to show off. Name and justify every pattern introduced.
- **Resources are finite**: every cache has a TTL, every collection has a bound, every connection has a pool. Design with limits from day one.
- **Resilience is a feature**: timeouts, retries, circuit breakers, and graceful degradation are not optional polish — they are part of the definition of done for any networked or I/O-bound feature.
- **Security is not a phase**: it's built in from the first line of code — never retrofitted.
- **Migrations are contracts**: once merged, a migration is immutable — the schema is a public API.
- **CI is the authority**: if CI says it's broken, it's broken — don't ship around it.

