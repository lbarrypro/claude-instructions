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

## 1. Plan Mode Default

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

### Built-in Subagent Types

| Type | Tools | Use when |
|------|-------|----------|
| `Explore` | Read-only | Fast codebase search, multi-location research |
| `Plan` | Read-only | Architecture planning, design decisions |
| `general-purpose` | All tools | Complex multi-step tasks |

### Custom Subagents (`.claude/agents/`)

Create project-specific agents with restricted tools and system prompts:
- **code-reviewer**: Read-only (`Glob, Grep, Read`), triggered automatically after changes
- **db-reader**: Bash only, with a `PreToolUse` hook blocking any non-SELECT SQL
- **parallel-research**: Multiple Explore agents in parallel for independent investigations

### `/batch` — Large-scale Parallel Execution

Built-in skill that orchestrates 5–30 agents in isolated Git worktrees simultaneously.
Use for: large refactors, cross-codebase analysis, parallel feature spikes.

```
/batch Refactor all API handlers to use the new error format
```

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

- Follow **SOLID** principles: Single Responsibility, Open/Closed, Dependency Inversion
- Apply **DRY**: extract duplication into shared abstractions only when it appears 3+ times and the abstraction is stable
- Apply **KISS**: the simplest working solution is the right one until proven otherwise
- Apply **YAGNI**: don't build for hypothetical futures

### Design Patterns — When to Use

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

- **Name it**: if you apply a pattern, name it explicitly in the code so it's recognizable
- **Justify it**: if a pattern adds indirection, it must earn its complexity — document why in the ADR
- **Don't force it**: a plain function or simple class is better than a pattern applied for its own sake
- **Prefer composition over inheritance** in all cases where both could work
- Before finalizing, ask *"does this code read like idiomatic [language] to a senior engineer?"*

---

## 8. Robustness, Performance & Resource Management

### Memory Management
- **No memory leaks**: every resource opened must be closed — use `finally`, `using`, `defer`, context managers, or RAII
- **Avoid unbounded collections**: cap queues, lists, and maps that grow over time; set explicit `maxSize` limits
- **Prefer streaming over buffering**: for large data, process in chunks rather than loading everything into memory
- **Release references explicitly** when objects are long-lived (event listeners, timers, subscriptions) — unsubscribe on teardown
- **Use weak references** (`WeakMap`, `WeakRef`) for caches or secondary indexes that should not prevent GC
- **Profile before optimizing**: measure with heap snapshots or APM tools — don't guess

### Caching Strategy
- **Cache at the right layer**: L1 (in-process), L2 (shared/Redis), L3 (CDN/HTTP) — pick the layer matching the data's scope
- **Always define TTL and eviction policy**: LRU is the default; no cache entry lives forever
- **Cache-aside by default**: read from cache → on miss, load from source → populate cache
- **Invalidate deliberately**: cache invalidation must be an explicit design decision — document the strategy in the ADR
- **Cache only stable, expensive data**: don't cache what's cheap to compute or changes on every request
- **Key design matters**: cache keys must be deterministic, scoped, and versioned to avoid stale reads after deploys
- **Never cache errors** unless intentional (negative caching) — always document if you do

### Resilience & Error Handling
- **Fail fast, recover gracefully**: validate inputs at boundaries, surface errors early, provide meaningful fallbacks
- **Apply Circuit Breaker** for calls to external services
- **Timeouts everywhere**: every network call, DB query, and external API call must have an explicit timeout
- **Retry with exponential backoff + jitter** for transient failures; cap total retry attempts and always log each attempt
- **Bulkhead isolation**: isolate resource pools per subsystem so one slow dependency doesn't starve the others
- **Graceful degradation**: design features to work partially when dependencies are unavailable

### Performance
- **Measure first**: instrument before optimizing — confirm where the bottleneck actually is
- **Avoid N+1 queries**: batch or eager-load related data; use dataloaders or join queries
- **Paginate everything**: no endpoint or query returns unbounded result sets
- **Async for I/O, sync for CPU**: use workers/threads for CPU-intensive work
- **Connection pooling**: always use pooled connections for databases and HTTP clients

### Observability
- **Structured logging**: log JSON with consistent fields — `level`, `timestamp`, `traceId`, `service`, `message`
- **Log at boundaries**: every entry/exit of a significant operation (API request, job start/end, external call)
- **Metrics for what matters**: error rates, latency percentiles (p50/p95/p99), cache hit ratio, queue depth
- **Distributed tracing**: propagate trace IDs across service calls; correlate logs and spans with a single `traceId`
- **Alerts on symptoms, not causes**: alert on user-visible impact, not internal signals alone

### Rules
- Any new feature touching data access, external I/O, or shared state **must include a cache/memory strategy decision** — document it, even if "no cache needed because X"
- Performance-sensitive paths must have a benchmark or load test before shipping
- When in doubt between memory efficiency and readability: **readability wins** unless profiling proves otherwise

---

## 9. Security by Design

### Secrets Management
- **Never commit secrets**: no API keys, tokens, passwords in code or git history — ever
- `.env.example` must always be up to date; `.env` must be in `.gitignore`
- Use a secret manager (Vault, AWS Secrets Manager, Doppler) in production
- Rotate secrets immediately if accidentally exposed

### OWASP Top 10 — Always check before shipping
- **Injection** (SQL, command, LDAP): use parameterized queries and ORM — never string-concatenate user input
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

### Form Validation — Mandatory Front + Back
Every form must validate on BOTH client (UX) and server (security). Never rely on only one.
→ Full rules: see `.claude/rules/forms.md`

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
- **No mocking the DB in integration tests**: use a real test DB or transaction rollbacks
- **Test naming**: `it("should <behavior> when <condition>")`
- **Arrange-Act-Assert**: structure every test explicitly in three blocks
- **Test data via factories/fixtures**: never hardcode IDs or dates; use deterministic seeds
- **Regression tests**: every bug fixed must have a test that reproduces it first

→ Full testing rules when editing test files: see `.claude/rules/tests.md`

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

Rules when editing migration files → see `.claude/rules/migrations.md`

Summary:
- Never modify a merged migration — create a new one
- Backward-compatible migrations only: new columns must have defaults or be nullable
- Separate deploy from migrate — never couple them
- No data migrations in schema migrations
- Dropping a column or table requires an ADR

---

## 14. API Versioning & Backward Compatibility

- **Semver for all public contracts**: MAJOR = breaking, MINOR = additive, PATCH = fix
- **Never remove or rename a field without a deprecation period** — add new field, deprecate old, remove after N releases
- **Version in the URL** (`/v1/`, `/v2/`) for REST APIs with breaking changes
- **Deprecation header**: return `Deprecation: true` and `Sunset: <date>` headers on deprecated endpoints
- **Contract tests**: use Pact or equivalent to catch breaking changes before merge
- **Changelog entry for every API change**

---

## 15. Technical Debt

- **Document debt, don't hide it**: use `// TODO(username): explanation` with a link to a tracking issue
- **No silent workarounds**: if a fix is temporary, mark it — `// TEMP: reason, remove after X`
- **Debt review**: include a debt review item in every phase — identify what was cut and track it
- **Repay debt before it compounds**: a debt item that blocks 3+ features must be addressed in the next phase
- **The boy scout rule**: leave the code slightly better than you found it — one small cleanup per PR

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

## 17. Hooks — Lifecycle Automation

Configure in `.claude/settings.json` (project, committed) or `.claude/settings.local.json` (local, gitignored).

### Key Events

| Event | Trigger | Typical Use |
|-------|---------|-------------|
| `PreToolUse` | Before any tool runs | Block dangerous commands, validate SQL |
| `PostToolUse` | After tool succeeds | Auto-format, run linter, log actions |
| `SessionStart` | Session begin / after `/compact` | Re-inject critical context |
| `Notification` | Claude is idle, waiting for input | Desktop alert |
| `Stop` | Claude finishes responding | Verify work before accepting |

### Hook Types
- **`command`**: Shell script. Exit `0` = allow, exit `2` = block with message, other = log warning
- **`prompt`**: Single-turn LLM decision — returns `{"ok": true/false, "reason": "..."}`
- **`agent`**: Full subagent for complex verification (up to 50 tool turns)

### Recommended Setup
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\"" }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "echo \"$CLAUDE_TOOL_INPUT\" | jq -e '.command | test(\"rm -rf\")' && exit 2 || exit 0" }]
      }
    ]
  }
}
```

Add a `PreToolUse` hook on `Edit|Write` to block edits to `.env`, `*.lock`, `migrations/*` unless explicitly approved.

---

## 18. Skills — Slash Commands

Store in `.claude/skills/` (project) or `~/.claude/skills/` (personal).

### Built-in Skills Worth Using

| Skill | Purpose |
|-------|---------|
| `/batch <instruction>` | Parallel agents across worktrees — large refactors |
| `/simplify` | Spawns 3 review agents, aggregates findings, applies fixes |
| `/loop [interval] <prompt>` | Repeat a prompt on a timer (e.g. `/loop 5m check deploy`) |

### Creating a Project Skill

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Fetch issue #$ARGUMENTS with `gh issue view $ARGUMENTS`.
Reproduce the bug, fix it, write a regression test, open a PR.
```

- Skills with `disable-model-invocation: true` are **manual only** (e.g. production deploy checklist)
- Use `context: fork` for read-only analysis skills to keep the main context clean

---

## 19. MCP Servers — External Tool Integrations

Configure in `.mcp.json` (project scope, committed) or `~/.claude.json` (user scope).

```bash
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/
claude mcp add --scope user --transport http sentry https://mcp.sentry.io/
claude mcp add --transport stdio db-reader -- node scripts/db-readonly-mcp.js
```

| Server | Scope | Value |
|--------|-------|-------|
| GitHub | project | Read PRs, issues, diffs |
| Sentry | user | Pull production errors into context |
| PostgreSQL | local | Query the dev DB without leaving the session |
| Jira/Linear | user | Link issues to code changes |

- MCP servers at **project scope** go in `.mcp.json` — commit it so the team shares integrations
- Scope sensitive servers (prod DB, secrets) to **local** only — never commit credentials

---

## 20. `.claudeignore` — Controlling Claude's File Access

Every project **must** include a `.claudeignore` file at the root. It tells Claude which files to exclude from context (reads, searches, indexing).

→ Baseline template: `docs/templates/claudeignore`

### Rules
- **Create `.claudeignore` at project bootstrap** — before writing any code
- **Commit it** — project-level configuration that the whole team benefits from
- **Never ignore `docs/`, `tasks/`, `CLAUDE.md`, `CHANGELOG.md`, `README.md`**
- **Add stack-specific entries** as the project grows (e.g. `.terraform/`, `__generated__/`)
- **Do not ignore test files** — Claude must be able to read and write tests
- **Review and update** whenever a new tool or build step is added

---

## 21. `.gitignore` — Controlling What Git Tracks

Every project **must** include a `.gitignore` file at the root. Create it before the first commit.

→ Baseline template: `docs/templates/gitignore`

### Rules
- **Create `.gitignore` at project bootstrap** — before `git init` or the first `git add`
- **Never commit secrets**: if accidentally committed, rotate immediately and rewrite history (`git filter-repo`)
- **Commit `.env.example`**: always provide a documented, secret-free template
- **Do not ignore lock files**: `package-lock.json`, `poetry.lock`, `go.sum`, etc. must be committed
- **Do not ignore `docs/`, `tasks/`, `CHANGELOG.md`, `README.md`**
- **Review with `git status --short`** before every first commit on a new project

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

### Branch Strategy

- Main branches: `main` (production), `develop` (integration)
- **Every development phase or sub-phase must have its own branch**, created from `develop`:

```
develop → feature/<phase>-<short-description>
develop → fix/<issue-short-description>
develop → chore/<task-short-description>
```

- **Never commit directly to `develop` or `main`** — except during PoC phase (see below)
- Merge only via Pull Request, after review and all checks passing

> **Exception PoC**: During Proof of Concept (no stable prod yet), it's acceptable to merge feature branches directly on `main` or commit directly to `main` when working alone. Reintroduce `develop` once a v1 is deployed to production.

### Pull Request Requirements

**Title**: `[Phase X.Y] Short description of what was done`

**Body**: use template at `docs/templates/pr-checklist.md`

---

## Project Documentation (`docs/`)

All technical decisions and reference documents live in `docs/`. Never scatter architecture decisions in code comments or chat history.

| File | Purpose |
|------|---------|
| `docs/STACK.md` | Tech stack, chosen libraries, hosting, rationale |
| `docs/ARCHITECTURE.md` | High-level system design, diagrams |
| `docs/ADR/` | Architecture Decision Records — template: `docs/templates/adr.md` |
| `docs/lessons.md` | Mistakes made + rules derived to prevent recurrence |
| `docs/CONVENTIONS.md` | Naming conventions, code style, folder structure |
| `docs/API.md` | API contracts, endpoints, payload shapes |
| `docs/phases/` | One doc per phase — template: `docs/templates/phase.md` |
| `docs/phases/phase-X.Y-test-plan.md` | Test plan per phase — template: `docs/templates/test-plan.md` |
| `docs/phases/phase-X.Y-user-doc.md` | User doc per phase — template: `docs/templates/user-doc.md` |
| `docs/RECAP.md` | Living recap for PO and CTO — template: `docs/templates/recap.md` |
| `docs/templates/` | All document templates |
| `CHANGELOG.md` | Chronological log of all changes — template: `docs/templates/changelog.md` |
| `README.md` | Project overview, setup, usage — template: `docs/templates/readme.md` |
| `.claude/rules/` | Path-specific rules (forms, tests, migrations) |
| `.claude/settings.json` | Project-level permissions and hooks (committed) |
| `.mcp.json` | Project MCP server configuration (committed) |

### README Rules
- Must exist at the root of every project — create before any other work
- At session start: if missing or empty, generate from context and propose to user before writing
- Living document — update whenever setup, structure, or features change significantly
- Keep concise and developer-facing; link to `docs/` rather than duplicating content

### CHANGELOG Rules
- Update as part of **every PR** — never retroactively
- Each merged PR maps to at least one entry (`Added`, `Changed`, `Fixed`, `Removed`, `Security`)
- The `[Unreleased]` section accumulates entries until a version is tagged

### Per-Phase Documentation Rules
- Create the phase doc **before the PR is opened**, fill progressively, finalize before merge
- `test-plan.md` and `user-doc.md` are mandatory companion files — created before implementation, completed before merge
- If a phase is abandoned or pivoted, document the reason and set status to `Abandoned`

### Test Plan Rules
- Write before implementation starts, complete before PR is merged
- Every feature must have at least one test case entry
- Negative / error-path test cases are mandatory for any user-facing or API-facing feature
- Mark failing cases with `Status: Fail` — never delete them before the PR

### User Doc Rules
- Written in the target user's language — no jargon, no class names
- Screenshots or annotated CLI output required for any UI or interactive workflow
- If no user-visible surface (pure infra change), write a minimal doc explaining impact on existing behaviour

### RECAP Rules (`docs/RECAP.md`)
- Created at project bootstrap, updated at end of every phase — part of Definition of Done
- PO section: jargon-free, focused on user value; must include **Démo direction** (what can be shown to management)
- CTO section: precise and honest — surface risks, debt, and open questions explicitly

### Path-Specific Rules (`.claude/rules/`)
Rules that only activate when Claude opens matching files:
```
.claude/rules/forms.md      → src/**/*.{tsx,jsx,ts,js}   (form validation)
.claude/rules/tests.md      → **/*.{test,spec}.{ts,js}   (testing rules)
.claude/rules/migrations.md → migrations/**               (DB migration rules)
```

### Auto Memory
Claude automatically takes notes across sessions in `~/.claude/projects/<project>/memory/MEMORY.md`.
- First 200 lines loaded every session — no manual action required
- Complement to CLAUDE.md: CLAUDE.md = team standards (committed), auto memory = session learnings (local)

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
- [ ] `docs/RECAP.md` updated — PO section lists all deliverables, CTO section covers decisions, risks, and debt
- [ ] `CHANGELOG.md` has been updated with at least one entry for this phase
- [ ] Relevant `docs/` files are updated
- [ ] `tasks/todo.md` reflects completion
- [ ] `README.md` updated if setup, structure, or features changed

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
