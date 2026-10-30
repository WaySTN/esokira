# Esokira - Go Backend Agent Instructions

> **Canonical path:** `backend/AGENTS.md`  
> **Version:** `0.1.0` - 2026-10-30 (Asia/Jakarta)  
> **Status:** `REVIEWED_INTERNAL - PENDING_OWNER_APPROVAL`  
> **Scope:** every file under `backend/` (Go, SQL, migrations, generation, workers, tests, build/deployment files)  
> **Parent:** [`../AGENTS.md`](../AGENTS.md) remains authoritative and must be followed. This file adds narrower rules, never overrides security or financial invariants.  
> **Purpose:** executable working rules for an AI coding agent, **not** approval to start production coding, not a finalized database schema, and not proof a Go project exists.

## 1. Mission, approved stack, and hard boundaries

Build the backend for **Esokira**, a personal financial life-planning and simulation app. Only implement an **explicitly assigned and appropriately approved slice**. Preserve financial correctness, ownership isolation, auditable changes, and reproducible calculations.

- **DECIDED by the owner:** Go backend; PostgreSQL as the primary database; monorepo with `frontend/` (Flutter) and `backend/`.
- **Architecture direction, not independently approved:** modular monolith, one backend Go module, feature-oriented packages, pure financial engine, REST/JSON with versioned OpenAPI contracts, and a separately runnable worker when required.
- **PROPOSED / TBD, not defaults to auto-install:** `chi` or `net/http` router; `pgx/v5`, `sqlc`, migration runner; authentication provider; database ledger design; exact money storage and wire format; PostgreSQL job queue/River; hosting; logging/observability vendors; algorithmic models and statistical assumptions.
- **Out of scope without a new approval:** microservices, Redis/message bus as a reflex, bank connections, actual investments/trades, payment processing, financial advice by an LLM, R4 shared-goal authorization, or arbitrary platform integrations.

**Do not assume implementation exists.** The repository could be documentation-only. Inspect worktree, `backend/go.mod`, source packages, applied migrations, OpenAPI files, test runners, and CI before reporting what is implemented. Use Bahasa Indonesia for owner-facing reports; preserve source identifiers (`BE-Qxx`, `FR-xxx`, `DR-xxx`, `CS-xxx`) exactly.

## 2. Source-of-truth routing (read narrowly, not all at once)

Before changing backend code, read `../AGENTS.md`, then the **relevant sections** of the files below. Relative paths here start from `backend/`.

| When working on | Required source documents |
|---|---|
| Backend structure, dependency boundaries, workers | `../docs/architecture/BACKEND-ARCHITECTURE.md`; `../docs/architecture/SYSTEM-ARCHITECTURE.md` |
| Feature behavior, release dependency | `../docs/product/PRD.md` (target `FR-*` / `EN-*` / `NFR-*`); `../docs/product/FEATURE-MATRIX.md` |
| Actual accounts, transfer, opening balance, assets, debt, goals | `../docs/financial/FINANCIAL-DOMAIN-RULES.md` (target `DR-*`); `../docs/architecture/DATABASE-DESIGN.md` |
| Compound, projection, KPR, optimizer, Monte Carlo or financial GPS | `../docs/financial/CALCULATION-SPECS.md` (target `CS-*`); `../docs/financial/FINANCIAL-TEST-CASES.md` (`FTC-*`) |
| HTTP request/response, errors, versioning, idempotency | `../docs/architecture/API-GUIDELINES.md`; approved `../contracts/openapi.yaml` **only if present** |
| PostgreSQL schema, ownership, migration, query plans | `../docs/architecture/DATABASE-DESIGN.md`; approved applied migration files **only if present** |
| Identity, access control, secrets, export/delete, worker abuse | `../docs/security/SECURITY-PRIVACY.md` (including `ST-*` tests) |
| A Flutter-facing contract or UX error semantics | `../docs/architecture/FRONTEND-ARCHITECTURE.md`; affected `../docs/design/USER-FLOWS.md` and `SCREEN-SPECS.md` |

**Authority by topic:** PRD owns product acceptance; Financial Domain Rules own the meaning of money/actual/plan/goal; Calculation Specs and Golden Tests own mathematical contracts; Database Design owns candidate persistence intent; API Guidelines own conventions; an **approved** OpenAPI contract and **approved applied** SQL migrations become machine-checkable implementation references. No Markdown example is automatically a live endpoint or executable migration.

Planned files such as `../docs/INDEX.md`, `../docs/planning/DECISION-LOG.md`, `../contracts/openapi.yaml`, `db/migrations/`, `cmd/api/main.go`, and `go.mod` may not exist. Never fabricate their existence. If a referenced file is missing, report it and use only verified context.

## 3. Approval semantics and backend decision gates

Use the parent statuses: `DECIDED`, `REQUIRED`, `PROPOSED`, `TBD`, `BLOCKED`, `UNVERIFIED`. `REVIEWED_INTERNAL` means **reviewed draft**, not owner sign-off. An instruction such as "gas buat markdown selanjutnya" approves creation of that Markdown only, **not** all open architectural choices.

`BACKEND-ARCHITECTURE.md` section 18 records **17 backend decisions**; do not rename these IDs or silently convert them to approved. The table below is an **index**, not a second decision log.

| ID | Decision still requiring owner approval (check owner document) | Affected work |
|---|---|---|
| `BE-Q01` | Authentication provider, token/session rules and user mapping | R0/R1 identity |
| `BE-Q02` | Ledger, posting/correction/reversal, audit lifecycle | Actual financial writes |
| `BE-Q03` | Exact amount representation and Go/SQL/JSON precision/rounding | All financial amounts |
| `BE-Q04` | Effective/recorded/as-of dates, cutoff and timezone | Balances, reporting and forecasts |
| `BE-Q05` | Goal funding attribution vs planning-only amount | Goal progress / feasibility |
| `BE-Q06` | Router, OpenAPI generation and error envelope | HTTP/API foundation |
| `BE-Q07` | Idempotency key, replay, TTL, concurrency / retry protocol | Actual writes and async submission |
| `BE-Q08` | Driver/query tooling, migration runner, transaction pattern | PostgreSQL foundation |
| `BE-Q09` | Scenario baseline snapshot, versioning, stale/rebase semantics | R2 scenarios |
| `BE-Q10` | Worker queue, atomic enqueue, retry, cancel, retention | R3 heavy jobs |
| `BE-Q11` | Optimizer objective, constraints, penalty/infeasibility semantics | R3 goal allocation |
| `BE-Q12` | Monte Carlo distribution, seed, RNG, percentiles and versioning | R3 stochastic engine |
| `BE-Q13` | Export/delete process, privacy retention/backup | R1 privacy lifecycle |
| `BE-Q14` | Notification provider, consent, schedule and payload | R2 reminders |
| `BE-Q15` | Runtime quotas, timeouts, load/latency budgets | Availability and abuse resistance |
| `BE-Q16` | Owner approval of release scope and locale assumptions | Release gating |
| `BE-Q17` | Deployment, backup, restore and production environment | Operations |

Cross-reference relevant `DB-Qxx`, `API-Qxx`, `SEC-Qxx`, `ARCH-Qxx`, `FDR-Qxx`, and `PRD Q-xx` **in their owner documents**. Do not maintain competing status copies. When relevant approval is missing, summarize the question, options, blast radius, and test implications; mark only affected production behavior `BLOCKED`. Continue unrelated, explicitly authorized tasks (e.g., pure-Go golden test harness) where possible. An isolated throwaway prototype is allowed only when explicitly requested and clearly not production behavior.

## 4. Release gates and implementation order

| Release | Intended backend work | Mandatory gate |
|---|---|---|
| `R0` / `S-00` - `S-01` | Repo/build/test foundation, API/DB tooling, auth contract scaffolding, independent financial test harness | Decisions needed by each slice approved; no fake runnable code or fabricated migrations |
| `R1` / `S-02` - `S-06` | Identity/profile; actual accounts/transfers; assets, debt, net worth; goals/compound; export/delete | Approved auth, ledger, exact money, privacy, golden tests and cross-user/atomicity tests |
| `R2` / `S-07` - `S-09` | Budgets/recurrence, life events, deterministic projections, scenario versions, cash flow, stress, insights/reminders | Actual/plan isolation; approved snapshot and staleness semantics; reproducible forecasts |
| `R3` / `S-10` - `S-12` | Goal optimizer, bounded jobs, Monte Carlo, retirement, Financial GPS | Approved model assumptions, seed/replay, computational caps and model-validation tests |
| `R4` | Shared goals / portfolio exploration | **Discovery only**, no production implementation |

`S-*` are proposed delivery slices from `FEATURE-MATRIX.md`, not evidence of completion or a sprint commitment. Never insert mock algorithm results into production to satisfy a screen or an endpoint. Do not build advanced modules before required R1/R2 dependencies and relevant model approvals.

## 5. Package structure, imports, and dependency direction

**Candidate organization** (not an instruction to create empty folders):

```text
backend/
  cmd/
    api/                 # API executable, only when approved
    worker/              # optional R3 executable, only when required
  internal/
    bootstrap/           # dependency wiring and configuration
    platform/            # DB, HTTP plumbing, auth adapters, telemetry
    modules/
      accounts/          # feature handlers, use cases, persistence adapter
      transactions/
      goals/
      scenarios/
      simulations/
    finance/
      money/             # exact money/domain types
      compound/          # deterministic pure calculations
      projection/
      allocation/
      montecarlo/
  db/
    migrations/          # reviewed versioned migrations, if approved
    queries/             # SQL source, if chosen
    generated/           # generated code, if chosen
  go.mod                 # one module at first; only if repo has/needs it
```

**Dependency policy:** HTTP transport -> application/use case -> domain abstractions; persistence adapters implement declared interfaces; pure financial engine depends on plain values and approved domain types, not HTTP, PostgreSQL, logging, time-of-day globals, app secrets, or Flutter. Prefer composition at a single bootstrap entrypoint. Avoid cross-import cycles and a giant `utils` or `services` package with mixed responsibilities.

- `cmd/*` wires dependencies and starts processes. No financial rules or private credentials hardcoded there.
- Handler: decode + validate transport shape, resolve authenticated principal, call use case, map result/error. **No business math, SQL, or authorization shortcuts.**
- Use case: validate domain transitions, enforce owner identity and idempotency, orchestrate DB transaction, invoke pure engine if appropriate.
- Repository/adapters: parameterized SQL, constraints, owner filters, and transaction-bound queries; no HTTP types in data access.
- Domain/finance: deterministic business rules, typed values, testable calculations. No implicit I/O or network calls.
- Worker (when approved): separate process boundary but shared domain/engine. It cannot bypass ownership/privacy controls just because it runs in the backend.

Keep interfaces near consumers where they clarify test boundaries; do not abstract every trivial function. Respect existing package organization in code if it exists: propose bounded refactors instead of unrequested rewrites. **Do not create a second Go module, `go.work`, Kafka, Redis, or microservices by default.**

## 6. HTTP transport and authentication/authorization

- Use a **provider-agnostic** verified identity/principal boundary until `BE-Q01`/`API-Q06`/`SEC-Q01` are approved. Do not embed Firebase/OIDC logic everywhere or invent a token format.
- Every protected read/write/list/export/job/result must derive the owner from a **server-verified principal**. Never use client-supplied `user_id`, email, query string, request header, path ID, or unverified JWT claims as an ownership grant.
- Always check ownership on nested references and resource combinations (e.g., goal + account, transaction + account, scenario + baseline). Treat resource-existence disclosure policy as `TBD`; never leak cross-user metadata.
- Authenticate before protected use cases and fail closed when verification or configuration fails. Never enable an auth bypass in production or rely only on Flutter route guards.
- Request boundaries must validate schema, lengths, units, enums, dates, rates, currencies, and resource caps. Return stable typed errors without dumping user data or stack traces.
- API examples in `API-GUIDELINES.md` are **proposed**. Do not create endpoints, error codes, response envelopes, or a new `contracts/openapi.yaml` without an approved API-contract task. Never declare a route implemented solely because it is documented.
- External HTTP must be bounded by request size, server/client timeouts, cancellation, and per-user abuse limits; exact thresholds require decision/benchmark (`BE-Q15`).
- Changes in API behavior require coordinated OpenAPI/client/validation/error/contract tests. Do not break Flutter clients by silently renaming fields, monetary units, status codes, or route semantics.

## 7. PostgreSQL integrity and money writes

**A posted actual financial change is a privileged, atomic domain event**, not a generic `UPDATE balance = ...` action. See `DATABASE-DESIGN.md` and `FINANCIAL-DOMAIN-RULES.md`; candidate DDL there is **not an approved migration**.

1. **Exact values:** account balances and financial posting use approved exact money representation. Do not use binary `float32/float64` as the ledger source of truth. Unit (IDR whole rupiah, minor units, `NUMERIC`) and JSON encoding remain `BE-Q03`/`DB-Q02`/`API-Q03`.
2. **Opening balance:** initial position, not income. Do not post a duplicate opening transaction while initializing an account.
3. **Internal transfer:** move value between owned accounts in one DB transaction; no fictitious income/expense or changed net worth when there are no fees. Real fees/FX require separate explicit approved rules.
4. **Atomicity:** validate both account ownerships, record movement(s), update any approved aggregate, write idempotency/audit evidence and commit together. Any failure must roll back the entire operation. Transaction boundaries belong at the application use case.
5. **Concurrency:** use explicit lock/order or isolation strategy as approved, handle conflict and retry safely; do not rely on a single-process mutex for a multi-instance database.
6. **Idempotency:** timeouts can mean `UNKNOWN_OUTCOME`. Never blindly post again. Implement agreed key/fingerprint/replay/TTL/recovery rules before financial posting endpoints go live.
7. **Corrections:** never silently edit historical posted records in ways that violate approved audit/reversal semantics. Do not use hard deletion as an ad hoc correction.
8. **Ownership / anti-double-counting:** scope read/write queries to verified owner; distinguish account cash from separate asset valuations and debt outstanding; do not count an internal transfer as additional wealth.
9. **Migrations:** use versioned, reviewed, additive-first changes when possible; test upgrades against disposable PostgreSQL, including constraints, indexes, rollback or documented forward-recovery. Production destructive operations require explicit authorization and backup/recovery planning.
10. **No fake green tests:** SQLite or in-memory fakes cannot replace real PostgreSQL integration coverage for locking, isolation, constraints, migration and idempotency semantics.

SQL must be parameterized. Avoid interpolating user input into identifiers, sorting, filters or JSON-path expressions without strict allowlists. Close rows, handle `ErrNoRows`, propagate `context.Context`, check commit errors and avoid leaking connections or transactions. RLS is a **proposed defense in depth**, not a substitute for application object authorization; if enabled, verify pooling/session-role behavior and test against a non-bypass database role.

## 8. Pure-Go financial engine rules

**Mathematical source:** `FINANCIAL-DOMAIN-RULES.md`, `CALCULATION-SPECS.md`, and `FINANCIAL-TEST-CASES.md`. Read the specific `DR-*`, `CS-*`, `FTC-*` IDs for each algorithm, not general summaries.

- Keep inputs explicit and typed: amount/currency, units, rates (percent vs fraction, nominal/effective), compounding interval, as-of/timezone, contribution timing, term, rounding and assumptions.
- A mathematically valid formula **does not** choose the product policy. For example, contribution at period start vs end and rounding each period vs at final output remain decision-gated.
- Reject or explicitly handle zero return, negative return, zero periods, huge periods, invalid date/rate, insufficient cash, divide-by-zero and overflow; never hide `NaN`, `Inf` or silently clamp an error to zero.
- Prefer deterministic pure functions with no DB, HTTP, network, clock, randomness, global mutable variables or hidden I/O. Keep numeric-policy and display formatting separate.
- Preserve cash-flow conservation; separate **actual**, **planned**, **goal funded**, **scenario forecast**, and **stochastic result** as distinct types/meaning. A scenario, calculator or insight must never post an actual transaction.
- Test against independently checked golden vectors with explicit tolerance or exact-decimal equality as relevant. Property/invariant tests supplement, **not replace**, golden reference cases.
- If a model/policy remains `TBD` or `BLOCKED`, write a non-executable spec or explicitly isolated exploratory spike only when authorized; do not ship fabricated compound assumptions, success probabilities or recommended allocation as production output.

**R2/R3-specific:** record baseline revision, input snapshot or reference with its validity semantics, scenario version, assumptions, algorithm version and deterministic seed (where randomness is used). A changed actual baseline must not silently produce an apparently current old result. Monte Carlo needs approved RNG/distribution/correlation and statistical validation; optimizer needs approved objective/constraints and truthful infeasibility. Explainability cannot change the underlying computed values.

## 9. Workers, simulation jobs, and reliability (R3 only when approved)

- Do not introduce a queue/worker just because a folder exists in a diagram. `BE-Q10`, `DB-Q09`, `API-Q09`, and `SEC-Q16` gate job design.
- If an approved async simulation is requested: creation, enqueue and durable status must not lose work across process crashes. Transactional enqueue or outbox is a design option, **not a settled policy**.
- Every job has an authorized owner, bounded input size/path count/horizon, execution deadline, resource quota, cancellation policy, and safe retry semantics. Apply ownership to polling/download/cancel APIs too.
- Avoid running thousands of financial paths synchronously inside an unbounded public HTTP handler. Limit worker concurrency and memory as measured, not as invented constants.
- Worker retries must not double-post actual records, duplicate paid work, or publish contradictory final results. Make completion idempotent and status transitions well defined.
- Treat simulation artifacts as sensitive personal financial data: restrict tenant access, limit retention, and avoid raw input/result payloads in logs.
- State the run version, assumptions, seed, timestamps and stale/superseded status when returning results. Do not present model rates as guaranteed predictions.

## 10. Sensitive data, secrets, observability and privacy

- Never log/request-dump real financial amounts, free-text transaction descriptions, access/refresh tokens, email identifiers, raw simulation inputs, bank-like account identifiers, passwords or secrets; use sanitized metadata and synthetic fixtures only.
- Keep env secrets outside Git and artifacts; never ship hardcoded developer DB passwords/auth bypasses as production defaults. Never send confidential user data to external AI APIs or monitoring providers without an explicit approved decision and privacy basis.
- Protect DB and infrastructure using least-privilege roles, encrypted transport, credential rotation, restricted network exposure and audited access as approved; do not claim these exist without operational proof.
- Backups are not verified recovery. Production readiness needs documented data lifecycle, backup/restore drill evidence, retention and deletion/backup policy (`BE-Q13/17`, `SEC-Q10`-`SEC-Q14`).
- Export/delete and any sensitive account changes require authenticated ownership checks and additional safeguards chosen in `SECURITY-PRIVACY.md`; never return sensitive exports via public/long-lived links.
- Follow the applicable `ST-*` specs for cross-user abuse, malformed input, duplicate posting, auth expiry, secret exposure and worker cancellation. A passing unit test is not a security audit or legal compliance certificate.

## 11. Verification by change type (run only commands that exist)

Inspect `go.mod`, toolchain, build scripts, existing test directory and CI config first. **The commands below are candidate checks, not claims of installed tools or completed tests.**

| Work type | Minimum meaningful checks after implementation |
|---|---|
| Go source | `gofmt -l <changed-go-files>`; `cd backend && go test ./...` **if module/source exists** |
| Significant concurrency/shared state | Targeted `go test -race ./...` where toolchain supports it; stress/failure tests for locks and cancellation |
| Backend lint/static analysis | `cd backend && go vet ./...` if configured/available; chosen lint job if actually installed |
| Pure financial algorithm | Matching `FTC-*` golden tests; zero/negative/overflow edges; deterministic replay as applicable |
| SQL/migration/query changes | Migration application on disposable PostgreSQL; constraints/FKs/index and ownership tests; `sqlc` generation check only if approved and configured |
| Actual posting/transfer/idempotency | Real PostgreSQL tests for full rollback, duplicate-key replay, same-key/different-payload conflict, parallel writes, unknown-outcome recovery and other-owner denial |
| Auth or private data APIs | Valid/expired/invalid token, cross-user resource IDs, nested ownership, 401/403/404 disclosure policy, export/delete and secret-leak tests |
| OpenAPI / DTO | Validate approved schema with confirmed tooling, contract tests, Go/Flutter monetary/time compatibility and error shapes |
| Background job | Crash/restart replay, idempotent completion, bounded resource use, cancel/retry/retention/tenant isolation |
| Documentation-only change | Confirm paths, source statuses, IDs, links, code fences and ZIP/file integrity if packaged; **no claim of Go tests** |

Never claim `PASS` for a test that was skipped, missing, blocked or only visually inspected. Never change expected values just to match output. A successful `go test` without a real DB does not establish transaction or locking correctness.

## 12. Per-task workflow and reporting contract

Before changing anything:

1. **Inspect:** `git status`, repo tree, changed modules, existing migrations/contracts/tests and applicable nested instructions; record what is actually present.
2. **Read targeted sources:** identify release/slice and `FR-*`, `EN-*`, `NFR-*`, `DR-*`, `CS-*`, `FTC-*`, `BE-Qxx`, `DB-Qxx`, `API-Qxx`, `SEC-Qxx` as needed.
3. **Plan:** define smallest end-to-end scope, interfaces touched, expected files, DB/API impact, tests and unresolved decisions. Review with owner for consequential changes.
4. **Implement:** make the smallest authorized change; no unrelated refactor, dependency upgrade, hidden side effects or invented endpoints.
5. **Verify:** execute available checks and log command + exit/result; inspect diff for leaked data, incorrect imports, contract drift, concurrency and data-integrity risks.
6. **Handoff:** report achieved behavior, files changed, tests actually run, blockers, decision approvals still needed and recommended next task. Update `TASKS.md` / `PROGRESS.md` / ADR only if present and authorized.

**Pre-implementation plan format:**

```text
Task / release / FR-EN-NFR IDs:
Observed repo state (not assumptions):
In-scope / out-of-scope:
Document sections and BE/DB/API/SEC decisions:
File/API/migration impact:
Acceptance tests and safety risks:
Questions or blockers to owner:
```

**Final handoff format:**

```text
Status: DONE | PARTIAL | BLOCKED | UNVERIFIED
Task IDs and scope:
Files changed:
Behavior independently verified:
Tests executed (actual command/result):
Tests NOT executed (and why):
Open decisions, risks and dependencies:
Next authorized step:
```

If task cannot proceed due to an unapproved critical policy, say `BLOCKED` for that specific behavior and continue independent authorized work when appropriate. Do not pretend a partial API, schema draft or static architecture diagram is a production feature.

## 13. Backend Definition of Ready and Definition of Done

**Ready (for an implementation task):** (a) owner-authorized scope/release/requirement; (b) relevant policy decisions approved; (c) source docs and real code inspected; (d) acceptance and test strategy identified; (e) API/migration/rollout implications known; (f) security and data ownership not left implicit.

**Done:** correct behavior and errors; approved invariants preserved; auth/ownership enforced; financial computations match golden cases; real PostgreSQL integration evidence where DB behavior matters; error/unknown outcome and concurrency/retry states tested; API/DB/Flutter contracts updated when relevant; safe logging and privacy checks; actual test commands and results reported; no unrelated changes or secret leakage.

Use `PARTIAL`, `BLOCKED` or `UNVERIFIED` if required evidence is unavailable. **A compile success, generated code, plausible diagram, happy-path unit test, or successful HTTP 200 response is not sufficient alone.**

## 14. Maintenance, change control, and approval status

- This file is a **derived agent instruction**, subordinate to the root rules and the authoritative topic documents. Do not duplicate entire PRD/domain/DB/API specs here.
- After a real owner decision, update the **owning** decision register/ADR and affected contracts, tests and instructions as separately authorized; keep `BE-Qxx` traceability intact.
- Before release, cross-check behavior against approved `FR/EN/NFR/DR/CS/FTC/ST` requirements and current deployed schema. Reject unauthorized contract drift, silent irreversible migrations, misleading model output or privacy regression.
- Revisit these rules after changes to auth, ledger, monetary representation, idempotency, simulation methodology, worker model, deployment, data retention or compliance review.

**Owner approval for `backend/AGENTS.md` v0.1.0 is pending.** This document is a reviewed draft and does not itself approve `BE-Q01`-`BE-Q17`, migrations, dependencies or production release.
