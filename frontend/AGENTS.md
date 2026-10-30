# Esokira — Flutter Frontend Agent Instructions

> **Canonical path:** `frontend/AGENTS.md`  
> **Version:** `0.1.0` — 2026-10-30 (Asia/Jakarta)  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Scope:** all files below `frontend/`, including Flutter/Dart, Android, iOS, assets, tests, and frontend build configuration  
> **Parent instructions:** [`../AGENTS.md`](../AGENTS.md) **always apply**. This file adds frontend-specific rules; it never overrides or weakens them.  
> **Audience:** AI coding agents implementing explicitly authorized frontend tasks.

## 1. Mission and scope boundary

Build Esokira's mobile experience as a **trustworthy interface for personal financial planning and simulation**. Flutter/Dart is **DECIDED**; the Go backend and PostgreSQL database are **DECIDED**. Flutter is an **untrusted client**, not an accounting ledger, authorization service, statistical engine, or direct PostgreSQL client.

The repository may still contain documentation only. **Inspect actual files first**; do not claim that a widget, dependency, OpenAPI contract, backend endpoint, test suite, CI job, or mobile app already exists merely because it is documented.

### Non-negotiable outcomes

1. Only implement the scope explicitly requested by the Product Owner and allowed by the release/requirements.
2. Show **actual, planned, funded, forecast, scenario, and model-derived** values as different concepts; never silently merge them.
3. Use **server-confirmed** financial changes; never turn a failed/unknown request into a displayed success.
4. Never fabricate balances, forecast curves, success probabilities, historical data, backend responses, or compliance claims in production UI.
5. Keep the financial domain engine and privileged validation in Go; Flutter may format values and perform explicitly labeled, approved non-authoritative previews.
6. Protect private financial data throughout login, logout, app switching, local storage, logging, notifications, and deep links.
7. Do not select libraries or finalize `PROPOSED`/`TBD` product decisions silently.
8. **Review is not approval.** Existing documents marked `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` are reviewed drafts, not blanket authorization to code or deploy.

Communicate with the Product Owner in **Bahasa Indonesia**; keep source identifiers, `FR-*`, `FE-Q*`, `SCR-*`, `UF-*`, and error/decision labels consistent.

## 2. Context router: read only what the task needs

Read `../AGENTS.md` first, then inspect the changed feature's files, tests, and **targeted sections** of the appropriate source documents. The following paths are relative to `frontend/`:

| Task | Required reference documents |
|---|---|
| All production Flutter tasks | `../docs/architecture/FRONTEND-ARCHITECTURE.md`; `../docs/product/PRD.md`; `../docs/product/FEATURE-MATRIX.md` |
| Screen/navigation/layout | `../docs/design/SCREEN-SPECS.md`; `../docs/design/USER-FLOWS.md`; `../docs/design/DESIGN-SYSTEM.md` |
| API client/request/response | `../docs/architecture/API-GUIDELINES.md`; approved `../contracts/openapi.yaml` **if it exists**; relevant Go/domain contracts |
| Account, transfer, dashboard, goal amounts | `../docs/financial/FINANCIAL-DOMAIN-RULES.md`; applicable `../docs/financial/CALCULATION-SPECS.md`; `../docs/financial/FINANCIAL-TEST-CASES.md` |
| Financial charts/simulations | Above financial documents, `../docs/design/SCREEN-SPECS.md`, and relevant `CS-*`/`FTC-*` |
| Login/session, data deletion/export, privacy/cache | `../docs/security/SECURITY-PRIVACY.md`; `../docs/architecture/SYSTEM-ARCHITECTURE.md` |
| Feature identity and user outcomes | `../docs/product/PRODUCT-VISION.md`, applicable `FR-*`, `UF-*`, and `SCR-*` |

**Topic authority:** PRD owns product requirements; Financial Domain Rules own financial meaning; Calculation Specs/golden tests own math; API Guidelines plus **approved OpenAPI** own HTTP contract; Screen Specs and User Flows own UX behavior; Design System owns UI token proposals; Security & Privacy owns security/privacy controls. An example endpoint or Dart class is **not** proof of production implementation or approval.

If a referenced file is missing in the actual repo, report it and inspect what exists; do not fabricate the missing file or content. Never ingest every long Markdown file indiscriminately when only one feature is being changed.

## 3. Decision register and blocking rules

Use the parent decision meanings: `DECIDED`, `REQUIRED`, `PROPOSED`, `TBD`, `BLOCKED`, `UNVERIFIED`. Only an explicit owner decision advances a candidate to approved status; the word **"gas"** authorizes the **requested next task**, not every unresolved decision.

Key decisions to inspect in `FRONTEND-ARCHITECTURE.md` §18:

| ID | Unresolved subject | Agent restriction |
|---|---|---|
| `FE-Q01` | State management (Riverpod, BLoC, other) | Do **not** install/adopt one as a final standard without approval. |
| `FE-Q02` | Router (for example `go_router`) | Do not finalize route package/guards based only on examples. |
| `FE-Q03` | HTTP client + serialization/codegen | Do not invent DTO formats or generated-client workflows. |
| `FE-Q04` | Authentication provider and secure storage | Do not build a guessed login provider/token scheme. |
| `FE-Q05` | Offline cache/writes/conflicts | Offline financial writes remain blocked without explicit policy. |
| `FE-Q06` | Exact monetary, rate, date, timezone formats | Do not guess API representation or rounding rules. |
| `FE-Q07` | Local calculation previews | Do not create a second authoritative financial calculator in Dart. |
| `FE-Q08` / `FE-Q09` | Navigation, design tokens, chart component/library | Do not lock proposed UX or add chart packages without review. |
| `FE-Q10`, `FE-Q11`, `FE-Q12` | Simulation job transport, notifications, export/delete behavior | Require backend contract, consent and privacy decisions before building these flows. |
| `FE-Q13`, `FE-Q14`, `FE-Q15`, `FE-Q16` | Target devices, localization, telemetry, performance budgets | Do not invent minimum OS, locale policy, analytics, or performance targets. |
| `FE-Q17`, `FE-Q18` | Stale scenario comparison and funded-goal semantics | Follow approved backend baseline/funding definitions; do not infer status from UI drafts. |

**R0/S-01 readiness:** resolve `FE-Q01`–`FE-Q03` and the required environment/security boundaries for the affected implementation. **R1 login** additionally depends on `FE-Q04`; **R1 actual money screens** depend on approved ledger/precision/API policies and `FE-Q06`.

When a necessary decision is missing, report its ID, options, impact, and proposed recommendation. Continue independent approved work, but mark the affected production behavior **BLOCKED**. Do not create a mock that pretends the decision has been made.

## 4. Frontend architecture and code organization

**Proposed direction, not a mandated package choice:** feature-first modules with separation of presentation, state/controller or view-model, repository, typed remote data source, and UI-safe models. Reuse existing repository conventions when they exist; suggest a limited change rather than silently reorganizing everything.

```text
frontend/
  lib/
    app/                 # app bootstrap, environment-aware routing/theme
    core/                # approved shared API, auth, formatting, errors
    features/
      <feature>/
        presentation/    # widgets, pages, UI state
        domain/          # screen-safe models, interfaces if needed
        data/            # DTO mapping, repository implementations
  test/                  # unit/widget/contract fixtures when authorized
```

This **illustrates possible boundaries only**. It is not proof these directories exist and must not trigger creation of every folder for a tiny task.

- Widgets render and forward user intents; **no HTTP, SQL, token parsing, net-worth formula, Monte Carlo, or business posting inside `build()`**.
- UI state/controller handles loading, validation presentation, retry eligibility, and navigation. Repositories own data access and API error mapping.
- Keep DTOs, UI models, and form draft models separate. Do not pass untyped `Map<String,dynamic>` from raw JSON deep into widget trees.
- Keep errors as typed states/codes, not logic derived from error-message string matching.
- Avoid multiple parallel global state-management patterns without an approved architectural reason.
- Depend only on packages actually chosen through approved decisions. Inspect `pubspec.yaml` and `pubspec.lock` before editing either.
- Make changes feature-local. No unrelated massive refactors, package upgrades, native permissions, platform manifests, or routing rewrites without explicit scope.

## 5. Screen IDs, flow IDs, and release gates

Preserve **conceptual** `SCR-00`–`SCR-22` and `UF-00`–`UF-31` identifiers from Screen Specs/User Flows. Screen IDs are not mandated route strings, Dart class names, nor API operation IDs. Each screen group can contain detail/form/result subviews without inventing unrelated canonical IDs.

| Release | Screen groups and intended slice | Gate |
|---|---|---|
| `R0` | `SCR-00` session gate; safe `SCR-22` recovery shell | Foundation and relevant auth/session contracts. |
| `R1` | `SCR-01`–`SCR-11`: auth, profile, actual finance, transfers, goals, compound, settings | Confirmed API behavior, ownership, money accuracy, user-visible states. |
| `R2` | `SCR-12`–`SCR-17`: budgets, life events, scenario versions, comparison, forecasting, insights | Scenario/baseline separation, stale/version semantics and consent. |
| `R3` | `SCR-18`–`SCR-21`: optimization, Monte Carlo, retirement, Financial GPS | Approved model, bounded jobs, reproducible results, uncertainty UX. |
| `R4` | `FR-023`/`FR-024`: shared goals/portfolio discovery | **Not authorized for production build**. |

For each assigned `SCR-xx`, find the primary `UF-xx`, applicable `UXT-xx` from Screen Specs and `FR-*` in PRD. Never expose a nonfunctional future feature as an active tab/action or build entire R2/R3 screens under an R1 task. Prototypes must have clear separation from production builds and user data.

## 6. Exact money, financial provenance, and safe charting

- Treat Go-confirmed actual financial data as the source of truth; never write an account balance as an authoritative local mutation.
- Preserve **exact** integer/decimal money representation and currency as defined by approved API/domain contracts. Avoid `double` as the source of truth for stored balances or serialized monetary amounts.
- Keep **missing** distinct from **zero** and **negative**. Parse errors must fail visibly rather than silently substitute `0` or today's date.
- Display amount, sign, unit/currency, period or `asOf`, and meaningful source (`Aktual`, `Rencana`, `Proyeksi`, `Skenario`, model output). Use Indonesian number/date formatting only per approved locale/date policy; formatting never changes the underlying amount.
- A transfer is **not income or expense** absent legitimate fees; opening balance is **not income**. Do not double-count assets or show planned goal contributions as already funded.
- Charts must distinguish actual, forecast, and scenario using text labels **plus** line/marker style, not color alone; preserve negative values and show accessible tabular/text equivalents.
- Risk/success rate is a **model-relative statistic**, not an assurance; show assumptions, horizon, engine/version and warnings where provided.
- Avoid manufacturing chart trend lines from a single value, deriving a future time series without an approved engine, or re-running Go calculations in Dart for a prettier graph.
- Local previews, if approved, must be **explicitly non-authoritative** and validated against Go golden fixtures; do not persist them as official simulation output.

## 7. API boundary, mutations, and error recovery

All private data access goes through the authenticated Go API. **Flutter never connects directly to PostgreSQL** and never trusts client-provided IDs as proof of ownership.

### Read path

1. API -> typed DTO parse/validate -> repository mapping -> presentation state.
2. Map contract/API failures into safe, stable UI errors; never reveal stack traces or raw backend financial payloads.
3. Respect pagination, filtering, cancellation/disposal, per-user cache keys, and response ordering to avoid displaying results from a previous session or filter.
4. An API shape is **not** final until the approved OpenAPI contract exists. Do not invent endpoint paths/headers/responses from illustrative examples in `API-GUIDELINES.md`.

### Actual write path

1. Validate form locally for UX, but do not replace server validation.
2. Disable duplicate user taps during submission; this does **not** replace server idempotency.
3. Use approved intent/idempotency semantics for transactions/transfers. If the network times out, result = **unknown/awaiting confirmation**, not a definite failure or success.
4. Recover by the approved read-by-intent/status flow if the API supports it; never retry a posting with a fresh identity by default.
5. Show confirmation, update/invalidate related views, and clear draft **only after an authoritative server outcome**.
6. Treat `401`, `403/404`, `409`, `422`, `429`, `5xx`, timeout, and parse errors as distinct recoverable/terminal classes according to the approved contract.

### Scenarios and heavy simulations

- Scenario edits produce **draft/versions**, never real transactions.
- Keep results tied to scenario version and source baseline revision. When actuals change, mark superseded calculations **stale** according to approved backend metadata; do not overwrite historical results silently.
- Use explicit job state (`draft/submitting/queued/running/succeeded/failed/cancelled`) for R3. Do not invent progress percentages or success output while the worker is running.
- Poll/cancel/retry only under an approved transport/backoff/auth lifecycle; stop polling and clear sensitive in-memory state on logout/session change.

## 8. Authentication and privacy-by-design

- Treat the mobile bundle/device as potentially compromised. Never embed service account keys, server secrets, database credentials, or private environment values in Flutter code, `.env`, asset files, test fixtures, or native manifests.
- Backend must verify principal and resource ownership. Client route guards are **UX controls only**; they are not server authorization.
- Do not invent Firebase/OIDC/email/Google/Apple login flows until `FE-Q04` and its backend security contract are decided.
- Tokens, if used, must follow an approved platform-secure storage and refresh/revocation design. Never persist them to plain shared preferences, logcat, console, error reporting, screenshots, or analytics.
- Clear/invalidate user-scoped views, requests, polling, caches, drafts (according to approved retention policy), and deep-link pending state on logout/account switch. Prevent late responses from repopulating private screens.
- Offline read cache is **TBD**; offline financial writes are **BLOCKED** until encryption, conflict, idempotency, and reconciliation are approved. Never represent cached data as fresh without source/freshness labels.
- Reverify authorization before opening private deep links, notifications, exports, and delete requests. Keep lock-screen previews and app-switcher surfaces privacy-safe where the approved platform policy permits.
- No real user financial data in AI prompts, public test fixtures, screenshots, remote telemetry, chart demos, logs, or console. Prefer synthetic, labeled fixtures.

## 9. UI, accessibility, and content standards

- Follow approved semantic tokens and shared components from `DESIGN-SYSTEM.md`; the proposed navy/teal/off-white palette is **not automatically approved**. Do not scatter literal colors, custom typography, iconography, or arbitrary spacing screen by screen.
- Prefer information hierarchy: current actual position -> context/source -> insight/next action. Avoid gradient-heavy dashboard cards, meaningless AI decorations, decorative charts without units, and finance KPI numbers with no as-of/source label.
- Provide functional empty, loading, disabled, success, failed, stale, partially loaded, and unknown-write states as appropriate; no dead controls or fake loading/success.
- Forms require precise field labels, clear optional/required cues, local validation, keyboard/focus handling, unsaved-change behavior, and safe retry instructions.
- Use localization-ready strings; Bahasa Indonesia-first is **PROPOSED** pending locale approval (`FE-Q14`). Do not scatter non-translatable literal business copy across widgets.
- Meet relevant WCAG 2.2/Flutter accessibility expectations: screen-reader semantics, meaningful focus order, adequate contrast, scalable text, touch targets, error announcements, and non-color chart alternatives.
- Layout must handle smaller devices, large text, negative/very large money amounts, system light/dark mode where approved, and connectivity changes. Device matrix and exact breakpoints remain **TBD**.
- Notifications must be opt-in under the agreed policy and never expose salary, balances, goals, or debts on a lock screen by default.

## 10. Performance, lifecycle, and reliability

- No network calls, blocking computation, or expensive decoding inside widget `build()`.
- List large financial records using approved pagination/lazy rendering; avoid downloading entire histories to calculate totals client-side.
- Use server-supplied aggregates for financial KPIs where provided. Downsample **chart presentation** only, never alter official summary totals or financial results.
- Cancel or ignore obsolete async responses after disposal, filter switches, auth changes, scenario version changes, or logout.
- Do not run official Monte Carlo/optimizer computation in the main Flutter isolate. Heavy R3 work belongs to approved Go simulation services/workers.
- Measure performance before claiming a target. Startup, p95, frame time, memory, bundle size, polling interval, and device matrix are **TBD** until requirements/measurement approve them.

## 11. Test matrix and evidence gates

For affected screens, use the predefined **`FE-T01`–`FE-T22`** in Frontend Architecture and **`UXT-01`–`UXT-40`** in Screen Specs; map exact test IDs to the task rather than claiming all are implemented. Add focused tests for:

| Area | Minimum behaviors to demonstrate |
|---|---|
| Session/authorization | Expired token, unauthenticated/private route, logout/cache wipe, cross-user resource denied. |
| Exact financial display | Null vs zero, negative values, huge amounts, decimal/locale parsing, actual vs planned, goal funding. |
| Mutation safety | Double tap, timeout/ambiguous outcome, server conflict, retry identity, no false success. |
| API/DTO | Missing fields, invalid currencies/rates, typed errors, stale/out-of-order response. |
| Scenario | Draft vs actual isolation, version/baseline mismatch, stale result labels, no actual posting. |
| R3 jobs | Queued/running/failed/cancelled/succeeded, no fake progress, logout cancels polling. |
| UX/accessibility | Keyboard, focus, text scaling, empty/error/offline, semantics, financial chart alternative. |
| Privacy | No token/PII/amount logging, private notification, proper cleanup after identity change. |

**If a Flutter project is present and commands apply**, inspect installed toolchain and manifests, then run the relevant commands and record actual results:

```bash
cd frontend
flutter pub get
dart format --output=none --set-exit-if-changed lib test
flutter analyze
flutter test
```

Use targeted paths when `test/` or some commands do not yet exist. Integration tests need a configured environment/emulator; contract tests need an **approved** API schema. Never claim a check passed if it was skipped, could not run, or only succeeded against uncontrolled dummy data. Root `../AGENTS.md` defines the authoritative overall evidence rules.

## 12. Per-task execution protocol

### Before changing files

1. `Inspect`: check actual Git/worktree and `frontend/` structure, existing nested instructions, SDK/package manifests, current implementation and tests.
2. `Trace`: identify release `R0`–`R3`, `FR-*`, `SCR-*`, `UF-*`, `UXT-*`, any `CS-*`/`DR-*`, and approved API operations relevant to the task.
3. `Gate`: list proposed/TBD decisions blocking only the affected production behavior. Do not block unrelated authorized work.
4. `Plan`: state intended changes, involved routes/state/API interactions, test coverage, and any privacy/contract implications **before substantial code edits**.

### While implementing

- Make the smallest coherent vertical slice. Preserve existing behavior, and inspect every dependency/contract touched.
- Do not modify `backend/`, `contracts/`, `docs/`, or production infra without scope approval; propose cross-boundary changes instead.
- Do not invent success states, mutate actuals through scenario flows, or conceal test failures.
- Prefer existing reusable components and approved libraries; avoid unrequested code generation, framework rewrites or package installation.

### After implementing

- Run relevant formatting/analyze/unit/widget/integration/contract checks **when available**; record commands, pass/fail, skips, and blockers.
- Inspect diff for unintended source files, auth leaks, API drift, mock data, formatting, accessibility and new permissions.
- Update task/progress logs **only if they exist and the task permits edits**. Do not claim an approval or feature completion merely because a screen renders.

## 13. Definition of Ready / Done and handoff

**Ready:** authorized scope; current release/FR/screen/flow; decisions resolved for that slice; source/contract identified; validation, states, privacy controls and tests defined. A task without approved API or monetary policy may still produce a clearly labeled **prototype**, never a purported production financial feature.

**Done:** approved behavior demonstrated; server-confirmed actuals; correct units/provenance; loading/error/empty/ambiguous/stale states implemented as relevant; authorization/cache isolation preserved; accessible/localized design; actual checks executed and recorded; contracts/docs updated only with authorization. If some gate fails, status is **`PARTIAL`, `BLOCKED`, or `UNVERIFIED`**, not `DONE`.

Use this final report format:

```text
Status: DONE | PARTIAL | BLOCKED | UNVERIFIED
Scope / release / FR / SCR / UF / UXT IDs:
Changed frontend files:
Approved decisions used:
Verified screen behavior and backend boundary:
Tests run (exact commands, exit results):
Tests not run / why:
Privacy, financial correctness, and accessibility checks:
Open decisions and risks:
Next authorized action:
```

## 14. Maintenance and approval

This file should remain a **short operational rulebook**. Detailed screen maps, colors, formulas, and API examples belong in their authoritative documents, not duplicated permanently here. If an approved decision changes a referenced `FE-Qxx`, `FR-*`, `SCR-*`, contract, domain rule or release boundary, update the authoritative document and relevant tests before amending this file.

**Current disposition:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`. This document does **not** approve Riverpod, BLoC, `go_router`, Dio, a login provider, colors, chart packages, offline writes, a financial model, or an API payload. It is an agent-governance draft for `frontend/`, not evidence of working Flutter source code.
