# AGENTS.md — Esokira Root Agent Instructions

> **Path:** `AGENTS.md` (monorepo root)  
> **Version:** `0.1.0` — 2026-10-30 (Asia/Jakarta)  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Applies to:** entire repository; nested `frontend/AGENTS.md` / `backend/AGENTS.md` may add compatible area-specific rules later  
> **Purpose:** agent execution rules and documentation router, **not** new product requirements or approval of undecided architecture

## 1. Mission and non-negotiable context

**Esokira** is a mobile personal financial life-planning and simulation application. Implement only a clearly scoped, authorized user task. Never invent features, financial formulas, data-model policies, security exemptions, or production integrations to make a task appear complete.

**Explicitly selected:** Flutter/Dart mobile frontend, Go backend, PostgreSQL primary database, and a monorepo containing `/frontend` and `/backend`.

**Not necessarily approved:** modular-monolith implementation details, authentication provider, Go router/ORM/query generator, Flutter state management, job queue, final financial ledger, money precision policy, exact REST/OpenAPI payloads, brand/design tokens, Monte Carlo model, optimizer objective, or cloud deployment. Read their decision status in the relevant docs.

**All 15 source documents are currently `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`.** This status is *not* Product Owner approval, and a documentation diagram or example is not evidence that code exists. The repository may currently be documentation-only. Inspect the actual workspace before stating implementation status.

Communicate with the Product Owner primarily in **Bahasa Indonesia**; use consistent technical terms, requirement IDs, and code identifiers.

## 2. Documentation router — topic owners

Read this root file and **only the relevant portions** of the owner documents for the assigned task. Do not dump all long Markdown files into context at once.

| Topic / authority | Canonical file |
|---|---|
| Purpose, audience, non-goals | `docs/product/PRODUCT-VISION.md` |
| Features, acceptance, `FR-*`, `EN-*`, `NFR-*` | `docs/product/PRD.md` |
| Releases R0–R4, feature dependencies | `docs/product/FEATURE-MATRIX.md` |
| Financial meanings, `DR-*`, actual vs planned | `docs/financial/FINANCIAL-DOMAIN-RULES.md` |
| Formulas, time/rate/units, `CS-*` | `docs/financial/CALCULATION-SPECS.md` |
| Numeric reference/golden tests, `FTC-*` | `docs/financial/FINANCIAL-TEST-CASES.md` |
| Overall system and trust boundaries | `docs/architecture/SYSTEM-ARCHITECTURE.md` |
| Go modules, transactions, worker | `docs/architecture/BACKEND-ARCHITECTURE.md` |
| Flutter modules, async state, offline | `docs/architecture/FRONTEND-ARCHITECTURE.md` |
| PostgreSQL entities/constraints/candidate SQL | `docs/architecture/DATABASE-DESIGN.md` |
| API policy, idempotency, errors, serialization | `docs/architecture/API-GUIDELINES.md` |
| Tokens, UI components, accessibility | `docs/design/DESIGN-SYSTEM.md` |
| Journeys and errors, `UF-*` | `docs/design/USER-FLOWS.md` |
| Screen/UI contracts, `SCR-*`, `UXT-*` | `docs/design/SCREEN-SPECS.md` |
| Privacy, authentication, authorization, `ST-*` | `docs/security/SECURITY-PRIVACY.md` |

**Additional files only when they actually exist:** `docs/INDEX.md`, `docs/planning/TASKS.md`, `docs/planning/PROGRESS.md`, `docs/planning/DECISION-LOG.md`, `docs/architecture/ADR/`, `contracts/openapi.yaml`, `backend/db/migrations/`, and nested area `AGENTS.md` files. These are planned paths, **not** claims about existing files.

- **Before Go/API/database work:** PRD + affected DR/CS/FTC, Backend Architecture, Database Design, API Guidelines, Security & Privacy.
- **Before financial calculation work:** PRD + Financial Domain Rules, Calculation Specs, Golden Tests; review backend boundary.
- **Before Flutter work:** Screen Specs, User Flows, Design System, Frontend Architecture, relevant API/security contracts.
- **Before auth, export, deletion or secrets:** Security & Privacy and relevant NFR/FR; inspect all affected trust boundaries.
- **Before foundation or CI setup:** PRD R0, Feature Matrix, System Architecture and targeted backend/frontend/security docs.

**Authority belongs to the topic owner, not the last file read.** An *approved* OpenAPI file later becomes the machine-checkable API schema; approved, applied SQL migrations establish deployed DB structure. An implementation mismatch is drift to report, **not** permission to silently override approved business requirements.

## 3. Decision statuses and conflict handling

| Label | Required agent behavior |
|---|---|
| `DECIDED` | Explicitly owner-selected; use unless formally superseded |
| `REQUIRED` | Domain/security invariant; do not weaken |
| `PROPOSED` | Candidate; may compare or isolate a prototype, **not** silently lock into production |
| `TBD` | Missing decision; state impact/options and request approval if blocking |
| `BLOCKED` | Do not implement/release the blocked production path; proceed on independent authorized work |
| `REVIEWED_INTERNAL` | Documentation QA only; **not** owner approval |
| `UNVERIFIED` | Existence/behavior/test status is not established |

If docs conflict: report **paths, sections/IDs, impact, recommendation, and the exact owner decision needed**. Stop only dependent work. A message such as "lanjut" approves continuation of the requested task, **not every pending architecture/policy choice**. Never treat silence, sample code, API examples, or proposed SQL as approval.

When a decision is explicitly approved, record owner, date, rationale, affected requirements, and the needed documentation/API/DB/test changes in an existing decision log or a proposed new one if authorized. Do not invent approval history.

## 4. Release and scope gates

| Release | Product slice | Constraint |
|---|---|---|
| `R0` | Monorepo, identity/API/DB scaffolding, tooling and foundational tests | Do not choose unresolved auth or ledger policies silently |
| `R1` | Actual accounts/transactions, assets/debts, net worth, goals, compound calculations | Golden tests, ownership and safe mutation semantics required |
| `R2` | Budgets, life events, deterministic projections, scenario compare, cash flow, stress tests | Hypothetical actions must not modify actual data |
| `R3` | Goal Optimizer, Monte Carlo, Retirement/FIRE, Financial GPS | Validate model, dependency gates, reproducibility and heavy-job limits first |
| `R4` | Shared goals and portfolio exploration | **Discovery only**; no build without explicit approval/privacy review |

A planned release is **not** a completion claim or deadline. Respect dependency order in `FEATURE-MATRIX.md`; no placeholder API or dummy response may be presented as finished functionality.

## 5. Financial correctness invariants — always apply

1. **Actual, planned, forecast, scenario and simulation result are different data classes.** A simulator or goal plan **must never post actual transactions**.
2. **Planned contributions are not funded goal progress.** Only approved real funding semantics may change the displayed actual-funded amount.
3. An internal transfer (without fees) must not be counted as income or expense and must not change net worth; genuine fees/interest require explicit treatment.
4. Opening balances set the initial position; **do not record them again as income**. Never double count the same cash or asset in net worth.
5. Account balances use **exact monetary representation**, not binary floating point as source of truth. Minor units vs decimal and rounding policy remain subject to approval.
6. Clearly define currency, units, percentage/fraction, nominal vs effective returns, annual/monthly periods, contribution timing, timezone/cutoff and negative/zero cases. **Never silently choose these in production calculations**.
7. Financial engines should be pure domain code, independent of HTTP, database, UI and network. Generative AI is **not** an authoritative financial calculation engine.
8. Persist assumptions, baseline/data revision, engine version and random seed (if stochastic) to support deterministic replay and stale-result detection.
9. Do not present forecasts as certain outcomes or stochastic success rates as guarantees. Surface model assumptions and limits.
10. Apply relevant `DR-*`, `CS-*`, `FTC-*` and security ownership checks. Where the statistical model or financial policy is `TBD`, mark the affected operation **BLOCKED** rather than improvising it.

## 6. Security, privacy and actual financial writes

- Treat Flutter as **untrusted**. Identify the owner from a **server-verified principal**, not a client-supplied `user_id`, URL or header. Enforce ownership on reads, writes, lists, nested resources, exports and worker jobs.
- Fail closed for invalid token, missing authorization context, cross-user references, or broken security configuration.
- Never put secrets, raw tokens, credentials, real financial records or environment keys into git, Flutter builds, logs, telemetry, fixtures, screenshots or AI prompts. Use synthetic test data.
- Actual financial mutations must be **atomic** and protected from double posting. A timeout gives an **unknown outcome**, not permission to blindly retry or report success/failure.
- Require validation, rate/abuse controls where appropriate, safe error messages, and bounded queries/jobs. Do not expose private finance details in stack traces or notification previews.
- Data export/deletion are privileged operations needing ownership checks, auditable behavior, and explicit retention/backup rules before production.
- Refer to `docs/security/SECURITY-PRIVACY.md`. A security checklist is not evidence of penetration testing, legal approval, OWASP certification or PDP compliance.
- Do not add trading, real payments, bank integrations or regulated financial advice without an explicitly approved new scope.

## 7. Implementation boundaries

### Go / PostgreSQL / workers

- Keep transport, use-case/domain, persistence and core financial algorithms separate. Use module-aligned packages; **do not add microservices** without a justified approved design.
- Multi-step actual money writes require a DB transaction. Implement ownership, constraints, concurrency control, rollback and idempotency according to approved domain/DB design.
- `DATABASE-DESIGN.md` includes **illustrative SQL only**; it is not an approved migration. Use versioned migration + integration tests after data-model approval.
- Router, SQL tooling, queue and auth integration choices listed in docs are candidates. Do not install them just because they appear in an example.
- If an R3 worker is approved, enforce bounded concurrency/cost, per-user job ownership, retry/deduplication, cancellation/timeouts, status lifecycle and retention.

### Flutter / UX

- Follow the existing screen IDs (`SCR-00`–`SCR-22`) and flows (`UF-00`–`UF-31`) unless explicitly revised. Keep service/state logic out of widget rendering.
- Never mutate actual balances locally as authoritative truth; do not mask failed API writes with optimistic fake success. Handle pending/unknown/recovery states.
- Show differences between *actual/forecast*, *planned/funded*, *baseline/scenario*; mark stale data and expose assumptions. Use accessible financial charts, readable IDR formats and approved design tokens.
- No speculative routes, forms, screen copy, state-management package or permanent mock endpoint outside the authorized PRD/Screen Specs scope.

### API contracts

- `API-GUIDELINES.md` is a convention document; **example endpoints are not live APIs**. Create/update `contracts/openapi.yaml` only within an approved contract task.
- Coordinate any API changes across Go, Flutter clients, validation, errors, idempotency, money/time serialization and contract tests.
- Prefer backward-compatible changes. Breaking API/DB changes need approved migration and rollout plans.

## 8. Per-task agent workflow

**A. Inspect** — inspect Git/worktree state, actual files, nested agent instructions, source and existing tests. Read relevant docs by section/ID. Distinguish observed implementation from proposals.

**B. Plan and gate** — before nontrivial edits, report:

```text
TASK / release / FR IDs:
In scope / out of scope:
Verified code/docs and relevant references:
Approved decisions / unresolved blockers:
Expected changed files / API / DB impact:
Tests and safety checks actually available:
```

For material blockers: present options and ask for an owner decision for that **specific** scope; continue only independent, authorized tasks. Never silently settle financial/security/legal/product decisions.

**C. Implement** — change the smallest coherent slice, preserve unrelated behavior, and avoid broad refactors. Update tests, contracts, migrations or docs only where the authorized change requires them. Do not add new dependencies, telemetry, external services, destructive scripts, or secrets without an explicit decision.

**D. Verify** — run relevant tests, inspect failures and exit codes, review the diff, check sensitive data exposure. Financial changes need golden vectors and edge cases; actual writes need rollback, duplicate/retry, concurrent update and cross-user tests; Flutter changes need loading/error/empty/stale/accessibility states.

**E. Handoff** — state exactly what changed, what was tested, what is unverified, outstanding decisions/risks and next authorized step. Update `TASKS.md` / `PROGRESS.md` / ADR **only if those files exist and the task permits changes**. Never fabricate progress or claim approval.

## 9. Commands and test evidence

**Inspect manifests and available tools first.** Commands below are conditional examples, not proof that repository source code or CI exists.

| If available | Candidate check | Note |
|---|---|---|
| `backend/go.mod` | `cd backend && go test ./...` | PostgreSQL integration tests may require disposable test DB |
| Changed Go source | `gofmt -l <files>` | Report/fix formatting within task scope |
| `frontend/pubspec.yaml` | `cd frontend && flutter analyze` | Needs configured Flutter SDK/dependencies |
| Flutter tests | `cd frontend && flutter test` | Skipped tests are not passing evidence |
| Changed Dart source | `dart format --output=none --set-exit-if-changed <files>` | Use actual changed paths |
| Approved OpenAPI contract | Validate contract using a confirmed tool | Do not invent executable lint targets |
| Approved SQL migrations | Run existing migrations on disposable DB | Never test destructive migrations against live data |
| Markdown-only task | Check paths, references, code fences, decision status | No claim of Flutter/Go/security tests |

Do not assume `make`, Docker Compose, CI, local PostgreSQL or generated code exists. Never delete, weaken or skip failing tests to report green status.

## 10. Definition of Ready / Done

**Ready** requires: clearly scoped owner task, release and requirement IDs, approved policy where necessary, documented dependencies, and test plan. Unresolved critical assumptions must be isolated or blocked.

**Done** requires, where applicable: accepted scope, correct monetary invariants, server ownership checks, updated API/DB/client contract, tested normal and boundary/error/retry cases, security review, no leaked secrets, evidence of real commands and tests, and accurate documentation changes.

Use `PARTIAL`, `BLOCKED` or `UNVERIFIED` if any necessary gate is missing. **Never declare DONE because a UI renders or sample JSON parses.**

Report to owner in this format:

```text
Status: DONE | PARTIAL | BLOCKED | UNVERIFIED
Scope + requirement IDs:
Files changed:
Verified behavior:
Tests run (commands + actual results):
Not run / reasons:
Open decisions and risks:
Next authorized step:
```

## 11. Maintenance and approval

Keep this root file concise and linked to detailed source documents; later nested `AGENTS.md` files must not weaken security/financial invariants. Review after approved changes to auth, money precision/ledger, API contract, release scope, simulation methodology or deployment.

**Owner approval is pending for this version.** The file can serve as a reviewed draft, but it does **not** authorize production coding or convert existing `PROPOSED/TBD` choices into `DECIDED` choices.
