# Esokira - Life Planning & Hypothetical Life Events Specification

> **Canonical path:** `docs/features/LIFE-PLANNING.md`  
> **Version:** `0.1.0`  
> **Prepared:** 30 October 2026 (Asia/Jakarta)  
> **Documentation status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Implementation status:** `UNVERIFIED`; repository aplikasi belum diaudit.  
> **Execution evidence:** Flutter, Go, PostgreSQL, API, dan CI tests: `NOT_RUN`. Test IDs pada dokumen ini adalah rencana pengujian, bukan hasil pengujian yang sudah lulus.  
> **Primary scope:** R2 / S-07 / `FR-011`; integration follow-through S-08 / `FR-012`, `FR-013`, `FR-014`.  
> **Primary task:** `EP-S07-02` (epic, harus dipecah menjadi task kecil sebelum implementation approval).  
> **Governing sources:** PRD, Financial Domain Rules, Calculation Specs, Decision Log, User Flows, Screen Specs, and `AGENTS.md`.

---

## 0. Executive contract and non-negotiable boundaries

**Life planning pada Esokira adalah sarana mengevaluasi dampak finansial dari rencana hidup, bukan fasilitas untuk mencatat kejadian sebagai fakta.** Peristiwa seperti rencana menikah, membeli rumah, memiliki anak, kuliah, atau pindah kerja adalah input hipotetis sampai aksi pencatatan aktual dilakukan melalui alur domain terpisah yang disetujui.

Pemisahan berikut adalah **REQUIRED**:

1. `ACTUAL`: account, posted transaction, asset valuation, liability, dan income/expense historis yang benar-benar tercatat.
2. `PLANNED`: life event, goal plan, planned contribution, budget, dan recurring *definition*; belum berarti terjadi atau uang tersedia.
3. `FORECAST`: hasil deterministic projection dari baseline + asumsi + event; bukan saldo bank.
4. `SCENARIO_RESULT`: hasil perhitungan versi tertentu; tidak otomatis mengganti actual, goal, atau versi skenario lain.
5. `STOCHASTIC_RESULT`: kemungkinan output R3; di luar scope implementasi dokumen ini.

**Hard stops:**

- Menambah, mengedit, menyalin, membatalkan, atau menghapus life event **tidak pernah** menulis posted actual transaction, memindahkan uang, membuat loan aktual, mengubah saldo/valuasi aktual, atau menandai goal sebagai funded.
- Tidak ada biaya, penghasilan, bunga, inflasi, pajak, return, cicilan, usia, tanggal, FX, durasi, atau asumsi properti yang diisi diam-diam agar forecast terlihat lengkap (`DR-017`).
- Life event yang belum memenuhi input yang disepakati berstatus `NEEDS_INPUT`/`BLOCKED_FOR_PROJECTION`, bukan dihitung dengan angka nol palsu.
- Cash impact, perubahan asset/liability, dan perubahan net worth harus terklasifikasi; pembelian aset dan dana pinjaman tidak boleh dihitung ganda (`DR-006/009/010/023`).
- Event yang sama tidak boleh terhitung dua kali melalui budget, recurring plan, life event, maupun inherited scenario; identitas occurrence dan source attribution wajib dapat diaudit.
- Baseline, date convention, effect ordering, nominal/real basis, schema, dan persisted scenario-version policy masih mempunyai approval gate `PRD Q-04/Q-05/Q-06`; jangan mempromosikan kandidat menjadi keputusan final.
- `SCENARIO-ENGINE.md` yang belum diterbitkan harus direview dan disetujui sebelum proyeksi R2 production. Dokumen ini **tidak** menetapkan algoritma atau urutan event engine final.
- Semua akses private resource wajib owner-scoped pada Go backend. Flutter bukan trust boundary.
- Jangan menyatakan fitur `DONE`, review/quality gate lulus, atau model approved hanya karena dokumen ini telah disusun.

**Selected foundation:** Flutter/Dart mobile frontend, Go backend, PostgreSQL, monorepo `frontend/` + `backend/` telah ditetapkan pemilik proyek. Library, auth provider, OpenAPI path, data schema, money serialization, event calendar, dan detail deployment tetap mengikuti proses keputusan.

### 0.1 Meaning of specification labels

| Marker | Arti | Perilaku AI coding agent |
|---|---|---|
| `REQUIRED` | Domain invariant/PRD yang telah didefinisikan | Pertahankan; buktikan dengan test saat implementasi tersedia |
| `DEFINED_MATH` | Identitas aritmetika benar dengan asumsi eksplisit | Boleh menulis oracle mandiri; tidak mengunci kebijakan timing |
| `PROPOSED` | Opsi desain, UX, field, kategori, atau arsitektur | Boleh evaluasi/spike; jangan klaim kontrak production |
| `TBD` | Pilihan belum diputuskan | Catat pertanyaan, owner, dampak, dan task yang tertahan |
| `BLOCKED` | Implementasi/publikasi bergantung keputusan atau bukti yang tidak ada | Kerjakan bagian independen saja; laporkan blocker |
| `UNVERIFIED` | Tidak ada inspeksi kode yang terdokumentasi | Jangan menebak status implementasi |

---

## 1. Source-of-truth routing and document links

Read only the sections needed for the task, but never replace authoritative sources with this feature summary.

| Change area | Canonical input | Authority / evidence |
|---|---|---|
| Scope and acceptance | [PRD](../product/PRD.md), [Feature Matrix](../product/FEATURE-MATRIX.md) | `FR-011` R2/S-07; `FR-012/013/014` S-08; no R4 scope |
| Financial meaning | [Financial Domain Rules](../financial/FINANCIAL-DOMAIN-RULES.md) | Especially `DR-001/002/003/004/009/010/011/012/013/014/016/017/018/020/021/023/024` |
| Numbers and assumptions | [Calculation Specs](../financial/CALCULATION-SPECS.md) | `CS-009` deterministic projection, `CS-012` scenario replay; policies TBD |
| Numerical oracles | [Financial Test Cases](../financial/FINANCIAL-TEST-CASES.md) | `FTC-CASH-*`, `FTC-SCEN-*`, `FTC-BUD-*`; distinguish invariant vs conditional |
| Owner decisions | [Decision Log](../planning/DECISION-LOG.md) | PRD `Q-04/Q-05/Q-06/Q-11` and local technical questions |
| Current-data semantics | [Transactions](TRANSACTIONS.md), [Financial Goals](FINANCIAL-GOALS.md) | Actual ledger/goal funding distinct from planning |
| User interactions | [User Flows](../design/USER-FLOWS.md), [Screen Specs](../design/SCREEN-SPECS.md), [Design System](../design/DESIGN-SYSTEM.md) | `UF-18`, `SCR-13`, plus `UF-19/20/21/22` and `SCR-14/15/16` at S-08 |
| Persistence | [Database Design](../architecture/DATABASE-DESIGN.md) | Candidate `life_events`, `scenario_baselines`, `scenario_versions`; migrations not yet authoritative |
| API behavior | [API Guidelines](../architecture/API-GUIDELINES.md) | Auth, errors, revisions, concurrency; sample routes not approved endpoints |
| Go module | [Backend Architecture](../architecture/BACKEND-ARCHITECTURE.md), [backend AGENTS](../../backend/AGENTS.md) | Ownership, business logic, isolation, versioning |
| Flutter module | [Frontend Architecture](../architecture/FRONTEND-ARCHITECTURE.md), [frontend AGENTS](../../frontend/AGENTS.md) | Editor, state machine, accessible timeline, safe retries |
| Security | [Security Privacy](../security/SECURITY-PRIVACY.md), [System Architecture](../architecture/SYSTEM-ARCHITECTURE.md) | Private events, scenario owner, safe logs, data lifecycle |
| Delivery evidence | [Tasks](../planning/TASKS.md), [Roadmap](../planning/MASTER-ROADMAP.md), [Progress](../planning/PROGRESS.md) | `EP-S07-02`; no implied completed/approved work |
| Agent execution | [Root AGENTS](../../AGENTS.md), [Development Workflow](../development/DEVELOPMENT-WORKFLOW.md), [Testing Strategy](../development/TESTING-STRATEGY.md), [Definition of Done](../development/DEFINITION-OF-DONE.md) | Inspect -> Plan -> Gate -> Implement -> Verify -> Handoff |

**Future dependencies (planned, NOT existing or approved):** `docs/features/SIMULATIONS.md` and `docs/financial/SCENARIO-ENGINE.md` will own scenario experiences and engine policies once published. Their future path is cited as text, not a link to an existing artifact. Do not invent their content.

**Wire/schema rule:** an approved `contracts/openapi.yaml` and applied `backend/db/migrations/` are authoritative for concrete API paths and deployed SQL schema when present. Candidate structures in this specification are not permission to create those artifacts without review.

---

## 2. Scope, functional dependencies, and release allocation

| Anchor | Slice | Scope | Acceptance essence | Gate |
|---|---|---|---|---|
| `FR-011` | R2/S-07 | Create/edit a hypothetical dated event with explicit once/recurring effects in a planning draft | Life event persistence, source and assumptions visible, actual unchanged | `EP-S07-02`, `Q-05/Q-06`, validation and authz |
| `FR-010` | R2/S-07 | Budgets/recurring transaction rules, distinct module | Recurring definitions are not posted actuals; safe occurrence identity | `EP-S07-01`, recurring posting policy |
| `FR-012` | R2/S-08 | Deterministic projections consuming life events | Reproducible timeline, dated input/assumptions, no actual writes | Scenario engine + baseline/time policies |
| `FR-013` | R2/S-08 | Scenario drafts, immutable versions and comparison | Old versions unaffected, stale handling | `Q-06`, `CS-012` |
| `FR-014` | R2/S-08 | Forecast and shortfall visualization | Forecast labeled by period and source; incomplete inputs disclosed | Verified `CS-009` result |
| `FR-007/009` | R1, R2 integration | Financial goals and feasibility as inputs to life planning | Goal target and funded actual not overwritten by event | `Q-11` goal-funded policy where needed |
| `FR-015/016/022` | R2/S-09 | Stress, insights, opt-in reminders | Later consumers; no automatic financial action | Own scopes/approvals |

`EP-S07-02` is **one epic**, not an approved implementation task. Break it down after relevant approvals into narrowly testable `TK-S07-*` tasks through a reviewed update of `TASKS.md`; these IDs are not created here. `EP-S07-01` covers budgets/recurring actuals and is not replaced by this spec.

Out of scope: automated salary changes, buying property or issuing a loan in the real world, scheduling real bank transfers, insurance products, tax/legal advice, advisor guarantees, bank sync, credit scoring, property appraisal, auto-posting recurring actual transactions, R3 Monte Carlo/optimizer/retirement, and R4 shared goals/portfolio management.

Primary outcome: an owner can save and revisit a plausible **hypothetical plan** with clearly disclosed assumptions. Only after S-08 approval can the owner request a verified deterministic projection; neither action mutates actual financial records.

---

## 3. Domain glossary and classification

| Concept | Meaning | Source classification | Not equivalent to |
|---|---|---|---|
| `LifeEvent` | User-stated future event, with timeline and proposed effects | `PLANNED` | Actual transaction or a forecast result |
| `PlanningDraft` | Mutable planning artifact, subject to approved owner/version semantics | `PLANNED` | Immutable simulation run |
| `EventOccurrence` | One effective instance of a recurring hypothetical event | `PLANNED` | Recurring posted actual transaction |
| `EventEffect` | Explicit financial consequence within the projection model | `PLANNED_INPUT` | Observed expense, earned salary, bank statement |
| `Baseline` | Identified as-of reference to actual inputs | `ACTUAL_SNAPSHOT_REFERENCE` | A live changing query disguised as a frozen snapshot |
| `ScenarioVersion` | Identified, once-frozen combination of baseline and assumptions | `IMMUTABLE_PLAN_VERSION` (proposed structure) | Rewrite of R1 goal, account, or old run |
| `ProjectionRun` | Reproducible calculation of a specific version | `FORECAST` | Permission to update actual state |
| `FundingAttribution` | Confirmed goal funding under an approved policy | `ACTUAL_DERIVED_CONDITIONAL` | Accumulated planned contributions |
| `CalendarConvention` | Rule for date, timezone, recurrence, and partial periods | `TBD Q-05/Q-06` | A silent implementation default |
| `CashShortfall` | Modeled negative cash/required liquidity gap | `FORECAST` | Posted debt, overdraft or emergency borrowing |

Ownership is assigned and checked on every resource link (event -> draft -> scenario version -> baseline -> run). A valid UUID belonging to another user is never authorization. The same effect must not exist simultaneously as actual, planned scheduled event, and scenario cash flow without explicit de-duplication/source mapping.

---

## 4. Trust boundaries, invariants and prohibited side effects

```text
R1 actual data (Go/PostgreSQL, authenticated owner)
    | explicit read and approved snapshot policy
    v
Identified baseline (as_of + snapshot/fingerprint/version)
    | planning editor constructs/updates isolated draft
    v
Hypothetical life-event set and scenario assumptions
    | validated input; version freeze when approved
    v
Pure deterministic financial engine (Go; S-08)
    | reproducible result + source metadata
    v
Flutter Rencana -> Proyeksi UI (not Actual)

Forbidden backward arrows:
Plan/forecast -> posted actual balances/transactions/assets/debts.
```

| Invariant | Why it matters | Proof required |
|---|---|---|
| `LP-I01` | Editing planning only changes planning resource | Compare actual row count and balances before/after save/delete |
| `LP-I02` | Planned income/expense is never posted actual | Ledger and budget actual totals unchanged |
| `LP-I03` | Same event source/occurrence must not be applied twice | Dedup/aggregate and recurrence tests |
| `LP-I04` | Cross-user event/scenario/baseline access is denied | Nested owner negative tests |
| `LP-I05` | Scenario run is deterministic under equal validated inputs/version | Replay numerical oracles and input fingerprint |
| `LP-I06` | Older run cannot be silently rewritten when actual changes | Explicit stale/rebase/version tests |
| `LP-I07` | Internal transfer principal is not expense or net-worth loss | Source classification and state conservation |
| `LP-I08` | Loan proceeds have matching liability and do not appear as income | Cash/debt/net-worth conservation |
| `LP-I09` | Buying an asset does not count the purchased value twice | Nonduplicated assets + net worth tests |
| `LP-I10` | Goal planned funding does not equal confirmed allocation | R1 goal state unchanged; funded label guarded by Q-11 |
| `LP-I11` | Missing critical inputs cannot become assumed zero | `NEEDS_INPUT`/incomplete and reason trace |
| `LP-I12` | Model results expose money unit, horizon, source, and assumptions | API/UI provenance assertions |
| `LP-I13` | Simulated cash can go negative without inventing a loan | Show shortfall, not auto-borrow or clamp |
| `LP-I14` | Time ordering/recurrence is explicit and reproducible | Boundary cases and same-day collision rules |
| `LP-I15` | Event creation/edit must be safe after ambiguous network outcome | Stable write identity or approved reconciliation flow |
| `LP-I16` | Logs and errors do not disclose sensitive private event details | Privacy/security review and log assertions |

Sources: `DR-001/002/003/006/007/008/009/010/011/012/013/014/016/017/018/020/021/023/024`, plus `NFR-001/002/003/005/006/007/010`. `LP-Ixx` are **local specification anchors**, not new PRD requirements.

---

## 5. Candidate event types and financial-effect grammar

**REQUIRED:** effect parameters must be clear enough to account for modeled behavior without accidental double counting. **PROPOSED:** implement a small typed effect grammar instead of arbitrary user-provided executable expressions.

| Event category example | Possible effect grammar (PROPOSED) | Must not be silently assumed |
|---|---|---|
| Home purchase | `one_off_asset_purchase` with cash down payment, asset recognition, liability principal, fees; optional recurring debt service | Market value, interest rate, tenure, tax, property growth |
| Wedding or family ceremony | `one_off_external_expense` on an explicit date; installments only if separately modeled | Estimated wedding cost or loan automatically |
| Education | `one_off_external_expense`, `recurring_external_expense`, or a schedule of dated costs | Tuition escalation, scholarships, guaranteed return |
| Childcare/household change | Dated recurring additional cost with start/end and category | Number of years, tuition, healthcare assumptions |
| Career change | `recurring_income_replace_or_delta`, optional income interruption and transition costs | New salary, certainty of job transition, taxes |
| Job loss | Explicit negative income shock over time; later via S-09 stress model | Duration, emergency fund, severance by default |
| Moving house | One-time moving expense plus recurring rent change, optionally assets/lease liabilities when scoped | Housing appreciation, deposit recovery, tax |
| Vehicle purchase | Asset + cash outflow + possible liability, explicit future operating costs | Loan rate, depreciation, insurance, resale |

Types can be extended only by model review and test cases. Event labels/categories are **descriptive**, not a license to apply default prices. A one-time fee is not an ongoing recurring fee unless stated. A rate-change effect may replace an existing baseline amount or add a delta, but the choice must be explicit and approved; never add the full new salary on top of the old salary by mistake.

### 5.1 Candidate impact primitives (NOT production wire schema)

| Primitive | Suggested fields | Purpose / safety rule |
|---|---|---|
| `cash_external_inflow` | effective date, amount, currency, source | Hypothetical future income; not posted salary |
| `cash_external_outflow` | effective date, amount, currency, cause | Hypothetical expense; no actual debit |
| `recurring_cash_change` | start, end/count, cadence, amount, currency, replacement/delta policy | Generates dated *planned* effects under approved calendar rules |
| `asset_position_change` | asset kind, effective date, valuation, unit | Different from a cash account; cannot double count account |
| `liability_position_change` | principal, effective date, debt modeling assumptions | Loan receipt and debt balance must match |
| `loan_payment_schedule` | principal, interest, period, amount policy | Interest vs principal must be separated |
| `goal_timing_override` | goal reference, alternative date/plan | Scenario-only, never edit saved R1 goal |
| `assumption_override` | named parameter, unit, effective period | Must declare whether replacement or delta |

Any primitive unsupported by a vetted engine can be stored as a descriptive note only if UI says `Belum dapat dihitung`. Never imply unsupported features have numeric output.

---

## 6. Candidate data contract for a planning life event

The following is **a conceptual contract**, not approved SQL, Dart model, OpenAPI shape, or a final data schema.

| Candidate field | Intent | Validation and data boundary |
|---|---|---|
| `id` | Stable event identity | Server allocated; not sufficient to authorize resource |
| `owner_id` | User ownership | Derived from trusted session; not accepted blindly from mobile |
| `planning_draft_ref` | Which plan owns the event | Resolve owner and parent on every read/write |
| `scenario_version_ref` | If snapshotted in immutable version | Editing requires a new draft/version, not mutation of old version |
| `title` | User-entered event label | Bounded length; escaped in UI/logs; private by default |
| `category` | Optional display grouping | Approved vocabulary, unknown category safely handled |
| `effective_on` or `date_range` | When modeled effect begins/ends | Explicit time/calendar convention, precision/timezone policy |
| `effect_kind` | One-off, recurrence, rate change, or approved primitive | Reject unknown effects rather than interpreting code |
| `effects` | One or more typed impacts | Explicit direction, amount, currency, source, completeness |
| `recurrence` | Optional future occurrence rule | Calendar and horizon bounds TBD; no infinite unbounded loop |
| `model_assumptions` | Tax, fees, loan, inflation, etc. if provided | Distinguish user-entered from model default or unknown |
| `completeness` | `COMPLETE`, `NEEDS_INPUT`, `UNSUPPORTED` (candidate) | State computed from validation, not frontend trust |
| `revision` | Optimistic concurrency on mutable draft | Prevent lost updates; error shape per approved API |
| `schema_version` | Payload migration and interpretation | Old versions remain interpretable or explicitly unsupported |
| `created_at` / `updated_at` | Audit metadata | Not equivalent to effective date |
| `source_event_ref` | Optional previous version/template trace | Avoid reference cycles and accidental duplication |

No final enum, UUID policy, JSONB structure, numeric precision, database migration, or path is selected by this table. Versioned schema should reject unknown numerical meaning and preserve interpretability of old scenario inputs.

### 6.1 Financial money and numeric policy

- Every monetary input must bind a `currency`, amount semantics and supported precision; representation/rounding remains `PRD Q-04` and the financial calculation contract.
- The Go server validates type, unit, sign, non-finite values, numerical overflow, supported currency, and illegal period rates.
- Do not use Flutter `double`/binary floating point as the sole persisted financial truth. UI parsing must be localized yet converted to an approved exact transport representation.
- Only explicitly approved FX rules permit cross-currency aggregation. Incompatible currency input -> `INCOMPATIBLE_CURRENCY` or sanitized equivalent; do not compare literal numbers as if same currency.
- Zero can be a legitimate intentional amount only where allowed; absent numeric value is **not zero**. Capture source (`USER_ENTERED`, `MODEL_APPROVED`, `UNKNOWN`) where useful.

---

## 7. Planning draft, scenario versions and lifecycle

**R2/S-07:** `FR-011` binds life events to a *planning draft/default scenario*. **R2/S-08:** `FR-013` introduces durable versioning and scenario comparisons. Until `Q-06` is resolved, the precise persistence and version transition are `PROPOSED`.

```text
New planning draft
  -> add/edit life events
  -> validate and save draft
  -> (future S-08) select approved baseline
  -> freeze scenario version with input references
  -> calculate immutable run
  -> view forecast and assumptions
  -> edit? create a separate draft/version
  -> actual data changed? mark stale or explicitly rebase
```

| Operation | Allowed change | Forbidden change |
|---|---|---|
| Create plan | Planning container only | Create actual account transaction |
| Create/edit life event | Mutable draft with safe revision | Rewrite frozen run or another user's event |
| Duplicate event | New event identity and explicit scope | Reuse same occurrence twice in one projection inadvertently |
| Remove event from draft | Planning draft removal/archival according to retention policy | Delete posted historical transaction |
| Freeze scenario | Capture references/version/hashes under approved Q-06 | Silent auto-refresh of prior immutable version |
| Compute | Create a run/result tied to frozen validated input | Persist effects as actual ledger postings |
| Rebase | Explicit new baseline/version and change summary | Overwrite old assumptions or old numeric results silently |
| Export/delete | Privacy-reviewed per-user scope | Leak related baseline or event of another owner |

**Unknowns to approve:** Are mutable S-07 planning drafts versioned? How is editing conflict resolved? Are event objects duplicated into frozen versions or referenced through immutable content? How does delete/retention affect older runs? Until answer, document candidate behavior and block production persistence choices that depend on them.

### 7.1 Actual revision and provenance

For each completed run keep or reference: `owner_ref`, `as_of`, baseline ID/fingerprint, scenario/version ID, canonical input hash, engine/model/schema version, explicit assumptions, applied event IDs/versions, horizon, generated-at timestamp, and stale/incomplete metadata. For deterministic runs no random seed is necessary; stochastic R3 would require RNG/seed controls.

A changed actual after `as_of` does not retroactively alter a saved run. If recalculating against newer inputs, create a new identifiable result/version and explain the differences. Staleness detection mechanics remain `Q-06/DB-Q08`, but **silent overwrite is prohibited now**.

---

## 8. Time axis, dates and recurrence conventions

Life planning is date-sensitive. These are **questions to resolve, not silent defaults**:

| Concern | Required explicit policy before production computation | Invalid assumption example |
|---|---|---|
| Effective date vs created timestamp | `effective_on` must be distinct from `created_at`; as-of semantics defined | Using save date for financial impact |
| Date-only vs timestamp | Domain calendar time and timezone specified | Converting a date to UTC midnight and shifting the local day |
| Start/end inclusion | Inclusive/exclusive boundaries, timezone and horizon | Applying event outside forecast horizon |
| Leap years and Feb 29 | Recurrence rule for unavailable day approved | Always falling back to Mar 1 silently |
| Monthly day 29/30/31 | Missing-day behavior approved (skip, clamp, move) | Treat every month as 30 days |
| Same-day effects | Deterministic ordering and precedence | Outcome depends on SQL row order |
| Partial periods | Prorating salary/rent/return/tax documented | Whole-month rate on one-day event |
| One-time vs recurring | Explicit occurrence generation and bounding | Unlimited recurring effects without cap |
| Range vs uncertain date | Unknown timing displayed as range/no-number when unsupported | Selecting nearest day arbitrarily |
| Event in past | Past dates relative to as-of require classification/reconciliation or validation | Adding it as both actual and future impact |
| Cross-period category | Budget/forecast grouping anchored to approved calendar | Double count at month-end |
| Event edit after run | Version/recompute policy, old run unchanged | Replaying old run with new event content |

For a monthly life event, the **calendar policy** must generate a finite sequence within a validated horizon and stable occurrence IDs (conceptually event version + date + rule version). Occurrence identity is for simulation de-duplication; it does not authorize actual recurring payment posting. No scheduler should write actual transactions from life events.

---

## 9. One-time cash effects and asset/liability conservation

A simple one-off external event affects hypothetical cash at its effective date. The exact projection period/timing is governed by `SCENARIO-ENGINE.md` once approved. For numeric **identity examples only**, assume one currency (IDR), no tax/fees/FX/valuation changes unless shown.

| Event | Illustration | Required classification |
|---|---|---|
| Planned ceremony expense | Future cash: 50m -> 40m for an explicit 10m expense | `external_expense=10m` (forecast only), actual unchanged |
| Planned external grant | Future cash: 10m -> 15m for an explicit 5m inflow | `external_income=5m` (forecast only), actual unchanged |
| Move cash A to B | Cash A down 2m, cash B up 2m | Internal transfer, total cash/NW unchanged |
| Buy asset for cash 3m | Cash down 3m, owned asset recognized at 3m | Asset conversion, delta net worth 0 without fee/valuation difference |
| Loan principal 400m received | Cash inflow paired with debt increase 400m | Liability increase, not external earned income |

### 9.1 Home purchase worked accounting example (NOT home valuation model)

Hypothetical starting position: cash IDR 120,000,000, no house or home loan, initial net worth IDR 120,000,000. User models a house cost/initial valuation IDR 500,000,000, cash down payment IDR 100,000,000, and new home-loan principal IDR 400,000,000, all with **no fees, taxes, interest or subsequent growth** for this single-event identity check.

```text
Initial: cash = 120m; house asset = 0; debt = 0; NW = 120m.
Planned purchase: cash -100m; house asset +500m; debt +400m.
Final hypothetical: cash = 20m; house asset = 500m; debt = 400m.
Net worth = cash 20m + house 500m - debt 400m = 120m.
```

There is no realized gain, no 500m earned income, and no 400m salary income. If explicit closing fees IDR 10m are also paid as expense, modeled cash becomes 10m and modeled net worth 110m. The expense is not also subtracted from asset valuation a second time unless the approved valuation convention explicitly requires a different model.

**WARNING:** This example tests asset/liability conservation only. It does not approve a KPR amortization method, interest/accrual schedule, home price forecast, equity valuation, mortgage eligibility, affordability or debt product assumptions. If those inputs are required but unavailable, show a limited event preview and block comprehensive long-term net-worth/projection claims.

---

## 10. Recurring event semantics and FR-010 boundary

**Three separate kinds of recurrence must not be conflated:**

| Mechanism | Owner module | Meaning | Can it post actual? |
|---|---|---|---|
| Recurring **life-event effect** | `FR-011` life planning | Repeated hypothetical cost/income change in scenario | **No** |
| Recurring **transaction rule** | `FR-010` transactions/budget | Scheduled rule for possible real events | Only through separately approved posting policy (`UF-Q10`) |
| Recurring **goal contribution plan** | `FR-007/009` goals | Intended allocation for a future goal | Not by itself; `Q-11` attribution TBD |

An event such as future childcare cost IDR 2m/month from a stated date must produce at most one planned effect for a given approved occurrence. The same childcare amount must not simultaneously enter as `new household expense` **and** a category budget aggregate **and** a separate recurring rule with the same logical source.

For income/rent changes, determine whether an effect is **replacement** or **delta**:

- `REPLACE_EXISTING`: if modeled salary was 10m/month and explicitly becomes 15m/month, resulting future salary is 15m/month, **not 25m**.
- `ADD_DELTA`: if modeled existing salary is 10m/month and explicit increase is +5m/month, resulting future salary is 15m/month.
- `ADDITIONAL_INCOME`: a distinct new income stream can be added, but it must have a separate identity and sufficient provenance.

Those labels are conceptual choices; the final grammar and precedence must be approved. Interruption, negative delta, unpaid leave and gradual transitions require explicit periods, end dates, and gap handling.

---

## 11. Goals and planned contribution interaction

`FINANCIAL-GOALS.md` owns real goal definitions and `Q-11` funding attribution. Life planning may reference a selected goal or a *scenario-only override* for its target/date if approved; neither operation changes R1 saved goal or actual funded progress.

| Example | Allowed | Not allowed |
|---|---|---|
| Marriage changes future savings capacity | Show *projected* changed capacity when cash-flow data/model is complete | Claim monthly contribution was already made |
| House event plus down-payment goal | Link planned cost/goal milestone with explicit attribution and precedence policy | Deduct DP twice, once from event and once from goal plan |
| Career change modifies cash flow | Recalculate hypothetical feasibility after assumptions are approved | Automatically change R1 goal target or contribution record |
| User selects goal as a life milestone | Display due date and target, as planning reference | Count goal target as an external expense automatically |
| User has a planned transfer to savings account | Model transfer neutral to total net worth | Treat transfer principal as income or consumption expense |

If `Q-11` is unresolved, a result can show planned saving contributions **as plans**, but may not claim `funded_amount`, percentage funded, or guaranteed feasibility. When a goal and life-event expense describe the same future purchase, require an approved relationship to avoid recording the milestone cash effect twice.

---

## 12. Budget, financial forecast and repeated source integration

Budgets from `FR-010` are spending limits and comparisons to posted expenses; they are **not** by default additional cash outflows. The event editor must surface whether a planned expense is new or already represented by a baseline recurring spending assumption.

A future cash-flow series (`FR-014`) may consume selected life events only after `CS-009` / `SCENARIO-ENGINE.md` is approved. It should report: period start/end, external inflows/outflows, debt principal vs interest, internal transfer distinction, cash open/close, asset/liability positions, shortfall, warnings, and source events. Forecast negative cash is a modeled shortfall, **not** a posted overdraft or a newly created liability.

Before applying event costs, the engine must identify potential overlap sources: actual historical spending imported into baseline, future baseline recurring assumptions, a life event's recurring effects, the R2 recurring transaction schedule, budget plan, and linked goal contribution. A collision may be a legitimate distinct payment or an accidental duplicate; without an approved resolution rule, flag and request confirmation rather than silently netting or doubling values.

The S-08 output should enable user inspection of relevant event causes for negative forecast periods. Do not produce a definitive `safe to spend`, `ready to buy a house`, or `guaranteed achieve goal` statement from a partially specified plan.

---

## 13. Deterministic scenario integration: inputs, order, output

`FR-011` supplies dated hypothetical events; `FR-012` and `FR-013` own execution and version comparison. This document does **not** define final event ordering. `CS-009` contains an explicitly **PROPOSED** sequence pending `CALC-Q07/Q10/Q11` and `Q-05/Q-06`. It would be incorrect to copy that tentative ordering into production as if approved.

### 13.1 Required input provenance

At minimum, an engine call should be able to identify: owner-authorized baseline, as-of date, referenced actual snapshot/content fingerprint, scenario draft/version, event set with stable identities and effect parameters, selected currency or supported conversion model, horizon/time grid, assumptions including units and date conventions, model/schema/engine versions, and whether any input is incomplete.

### 13.2 Required output provenance

For each generated result, expose a stable run ID, source version, source type `FORECAST`, as-of and projection period, assumptions summary, model/engine version, completeness status, stale status, and a trace to events/effects that caused material changes. Failed or partial output is not equivalent to a computed zero.

### 13.3 Reproducibility and comparator boundaries

Same validated baseline + scenario version + model/engine version and explicit inputs must replay to the same deterministic numerical outputs under the approved precision policy. A comparison of two scenarios must use a compatible baseline/currency/time horizon/nominal-real basis, or explicitly disclose conversion/normalization only after approval. If incompatibility cannot be resolved, block the numerical comparison.

### 13.4 Not every event must be numerically computable

A user may write descriptive life milestones with date/title. In the absence of required approved effect parameters, the event can remain a **narrative plan** and be displayed on a timeline, labeled `Belum cukup data untuk menghitung dampak`. Do not invent market prices, KPR rate, investment return, tax or salary changes. S-07 can provide useful planning even before the S-08 engine is approved.

---

## 14. Assumption management and uncertainty communication

| Assumption | Required evidence | If missing |
|---|---|---|
| One-time cost | Amount, currency, date, user/source confirmation | Do not compute one-time cash impact |
| Ongoing cost | Per occurrence amount, start/end or approved bounded schedule | `NEEDS_INPUT` or unsupported recurrence |
| Salary/income change | Prior stream link, delta/replacement intent, effective dates | Block additive income calculation |
| Home purchase | Cash down payment, asset/liability treatment, optional fees | Preview limited effect only, not complete NW/debt forecast |
| Loan installments | Principal, rate convention, tenor, repayment policy/fees | No debt repayment projection |
| Inflation | Input basis (present real/future nominal), applied once, approved rate | Do not double inflate or make claims of purchasing power |
| Return/interest | Rate unit, compounding, event timing, charges | No unsupported growth projections |
| Tax and fees | Defined items and model inclusion/exclusion | Mark not modeled; do not assume tax-free truth |
| Currency and FX | Approved conversion and valuation dates | Reject/segregate mixed-currency result |
| Baseline completeness | Actual as-of and relevant income/expense/liability data | Partial/insufficient-data state, not false safety |

Disclose user-supplied assumptions distinctly from system-approved defaults; a default cannot be introduced merely because a UI field needs a value. Product copy should state results are estimations under supplied assumptions and may differ from real events. No professional investment, credit, tax or legal guarantees are given.

---

## 15. Flutter UX, flow and accessibility contract

Primary flow `UF-18` from [USER-FLOWS](../design/USER-FLOWS.md) and screen `SCR-13` from [SCREEN-SPECS](../design/SCREEN-SPECS.md) own expected user behavior. Use the [Design System](../design/DESIGN-SYSTEM.md) for approved components and accessible patterns; do not invent visual tokens as final design.

### 15.1 SCR-13.V01 - timeline of planned events

- Header `Rencana hidup` with clear `Rencana` source label and the selected draft/baseline identifier.
- Ordered list/timeline with date, title, event type, modeled/not-modeled indicator, currency unit, and whether assumptions are complete.
- Empty state: explain that adding an event **plans** a future possibility; does not create a transaction.
- Selected event action opens detail/edit and impact preview without creating actual data.
- Screen reader must read date, source status, amount unit, and warnings; a color line alone is insufficient.

### 15.2 SCR-13.V02 - life event editor

- Explicit event kind, date/range, impact kind and amount/unit; category choices are candidate until approved.
- If recurrence is chosen, present preview of occurrences only after recurrence/calendar policy is approved; otherwise disable calculation and preserve narrative draft.
- State difference between `Save planned event` vs `Post real transaction`. The latter is **not** an action on this screen.
- Preview validation/warnings, disclose assumptions, and preserve unsaved changes when safe under approved local persistence policy.
- Owner-scoped resource errors, lost-update conflicts, ambiguous success and offline states must not be shown as ordinary success.

### 15.3 SCR-13.V03 - impact preview

- Show `Perkiraan dampak` and explicit basis: forecast only, as-of, event timing, currency, inclusions/exclusions and model readiness.
- If engine unavailable/unapproved, show event metadata plus `Perhitungan belum tersedia` instead of fabricating chart data.
- If input missing, show field-level `Butuh informasi tambahan` and which parameters are missing.
- If event changes after a result was computed, keep older run identifiable and label stale; no live recalculation against hidden baseline.
- Link to `SCR-16` projection only when S-08 engine is approved and sufficient inputs are available. `SCR-14/15` are separate scenario/version/compare screens and are not silently included in S-07.

### 15.4 Supported states

| State | UI semantics | Prohibited claim |
|---|---|---|
| `EMPTY` | No events; explain planning | `Anda tidak punya biaya masa depan` |
| `DRAFT` | Unsaved/saved hypothetical plan clearly marked | `Pengeluaran tercatat` |
| `NEEDS_INPUT` | Missing required effect data and actions to complete | A financial projection equals 0 |
| `VALIDATED` | Input valid under approved grammar | Simulation/goal succeeded |
| `CALCULATING` | Input snapshot fixed while request runs | Old response overwrites newer selection |
| `FORECAST_READY` | Result with source, period, assumptions | Actual balance changed |
| `STALE` | Result refers to older baseline/version | Up-to-date comparison |
| `ERROR` | Sanitized error, keep safe draft | Display success toast |
| `CONFLICT` | Lost update or simultaneous device edit | Silently overwrite previous user edits |
| `OFFLINE` | View trusted cached draft if policy allows, avoid writes to actual | Offline request successfully committed |
| `UNSUPPORTED` | Specific effect cannot yet be calculated | Show a guessed number |

No floating or fabricated UI sample may be used in production as real user data. Localization and a11y must follow `NFR-005`, and user-sensitive amounts must not be exposed in crash reports or analytics.

---

## 16. API/service boundaries: behavior, not invented routes

**No concrete endpoint, request JSON contract, OpenAPI response, or HTTP status mapping is declared final here.** The following are candidate use cases; implement only after approved contract review and actual repository inspection.

| Candidate Go use case | Responsibility | Mandatory checks |
|---|---|---|
| `CreateLifeEventDraft` | Validate life event ownership, date/effect grammar, and save in planning scope | Identity, scope, revision/write safety, no actual posting |
| `UpdateLifeEventDraft` | Apply owner-authorized edit with conflict check | Parent ownership, revision, preserved old snapshots |
| `GetLifeEvent` / `ListLifeEvents` | Read private planned events for selected draft | Owner-scoped query, no cross-user references |
| `RemoveLifeEventFromDraft` | Remove/archive planned event according to approved policy | No actual deletes, linked run retention review |
| `PreviewLifeEventEffects` | Return normalized proposed effects and validation/completeness | Do not claim numeric result without approved engine |
| `ValidatePlanningDraft` | Verify dependent event references and conflict/timing assumptions | Duplicate source detection, actionable missing fields |
| `FreezeScenarioVersion` (S-08) | Identify immutable version referencing authorized baseline | Q-06, version race, snapshot provenance |
| `RunLifeProjection` (S-08) | Invoke pure Go engine with validated inputs | CS-009, engine readiness, replay, no actual writes |

### 16.1 Candidate service boundary

```text
HTTP transport -> authentication/authorization -> planning application use case
      -> validation and revision control -> planning repository (PostgreSQL)
      -> (when approved S-08) scenario builder -> pure Go projection engine
      -> serialized safe response with source/provenance metadata

NEVER: life-event save -> actual ledger posting service as an implicit side effect.
```

The pure financial engine should accept typed read-only values, not DB connections or HTTP request objects; it cannot depend on Flutter calculations. Service-layer ownership checks are mandatory even if candidate PostgreSQL foreign keys exist. Never create a blanket `catch` that converts model validation errors into a successful zero forecast.

### 16.2 Retries and ambiguous response

S-07 draft writes need deterministic recovery from timeout/lost response; which operations require idempotency keys, optimistic revision, or read-after-timeout reconciliation is an open API decision (`API-GUIDELINES.md`, related `API-Q*`). Avoid auto-replay of a potentially committed write with a fresh ID. Saved version identity and current draft revision must be inspectable after recovery, and no actual mutation may occur in any path.

---

## 17. Database candidate design and isolation requirements

`DATABASE-DESIGN.md` section 12 lists a **candidate** R2 `life_events` table (`id`, `owner_id`, `name`, `planned_on`, `assumptions_jsonb`, `schema_version`) and candidate `scenario_baselines`, `scenarios`, `scenario_versions`. Do not treat that example as an applied migration.

| Storage concern | Required safety behavior | Still undecided |
|---|---|---|
| Owner scoping | Composite owner-safe references or equivalent validated ownership | Exact FK/row design, RLS decision |
| Draft association | Event belongs to a valid owner-scoped planning container | Whether `planning_draft_ref` exists as a separate persisted table |
| Event effect normalization | Schema version and typed semantics preserved | JSONB vs normalized effect rows; migrations |
| Timeline order | Stable ordering by effective date plus explicit tie-break | Which same-day ordering rule has financial meaning |
| Revision control | Concurrent edits do not overwrite one another silently | ETag/If-Match or equivalent and status codes |
| Frozen version | Immutable scenario/version inputs for replay | Snapshot vs versioned read model; `DB-Q08/Q-06` |
| Source de-duplication | Event version + occurrence identity traceable | Canonical occurrence grammar |
| Deletion/export | Include event, draft, version and relevant results per privacy policy | Retention and backup mechanics `SEC-Q*` |
| Performance | Bounded queries and event occurrence explosion protection | Approved event and horizon quotas |

No actual ledger table should accept life event as a posted transaction by switching a Boolean flag. A scenario-specific event storage boundary should prevent accidental inclusion in actual balance aggregates even if a developer writes an insufficient `WHERE` clause.

Use migrations, transactional writes and verification appropriate to the final schema. Indexes, unique constraints, triggers, and foreign keys shown in older docs are suggestions until approved. Internal owner checking is not replaced by UI filters or UUID secrecy.

---

## 18. Security, privacy, lifecycle and abuse cases

Life events can reveal sensitive plans (family, work, education, housing). Treat title, notes, assumptions, detailed amounts and schedule as private financial context even when no real transaction has been recorded.

- Authenticate and authorize **every** read/write on event, parent draft, baseline, scenario version, nested result, export and delete; verify same owner across referenced resources (`DR-003`, `SEC-001` onward as applicable).
- Do not trust client-provided `owner_id`, source event references or `scenario_version_id` to authorize nested resources. Test IDOR and existence leakage through sanitised errors.
- Accept typed bounded values; reject injection in title/notes, overlong strings, unsupported recurrence rules, invalid date ranges and huge generated occurrence counts.
- Use parameterized SQL; treat arbitrary user JSON expressions as data, not executable DSL.
- Do not put planned salaries, relationship/family notes, IDs, precise amounts or tokens into analytics, crash breadcrumbs, tracing payloads or URL query strings.
- Do not include unapproved sensitive lock-screen reminder content. Notifications are a separate opt-in `FR-022` area.
- Exports, deletion, backup retention and account deletion must cover planning artifacts according to approved privacy policy. Never promise immediate disappearance from backup without evidence.
- Store drafts safely when offline only if storage/encryption policy is approved. Device cache is not authoritative.
- Apply rate limits/resource budgets as appropriate to validation/projection; app must not trigger unbounded recurrence expansion or nested scenario loops.
- Explain how changing actual data can mark an old scenario result stale without exposing private details in shared telemetry or error messages.

Security verification must use relevant `ST-001` to `ST-075` cases in the owning security document, augmented by local `LPT-*` tests below; those local IDs do **not** renumber the security matrix.

---

## 19. Reference examples: precise outputs and explicit limitations

These vectors are **documentation-only reference reasoning**. They are not passed tests from an actual Go/Flutter implementation. Treat arithmetic examples as identities conditional on stated assumptions; production timing/precision still needs approved oracles.

### EX-01: Wedding cost without actual posting

Baseline for illustrative future forecast: cash = IDR 50m; one-off future wedding expense = IDR 10m, explicitly modeled within forecast horizon, with no other effects. Hypothetical cash after modeled expense = IDR 40m; hypothetical external expense = IDR 10m. **Actual account balances and actual expense history remain unchanged**. If the event is outside horizon, it must not affect forecast output (`FTC-CASH-09`).

### EX-02: Career change without salary duplication

Modeled salary before effective date = IDR 10m/month; new salary **replaces** old salary at IDR 15m/month starting an explicitly selected month. Modeled salary after date = IDR 15m/month, not IDR 25m/month. With explicit delta +5m/month instead, it may also become IDR 15m/month under a valid delta model. The two representations must not both be applied. This is not proof that the new salary will actually be earned.

### EX-03: Cash shortfall remains visible

Future modeled starting cash IDR 1m; explicit external expense IDR 2m, income IDR 0, no approved loan. Hypothetical closing cash = **negative IDR 1m**. Show `Proyeksi kekurangan dana` / `shortfall`; never clamp to 0 or create an implicit loan (`FTC-CASH-04`, conditional cash policy).

### EX-04: Planned goal and event overlap

A down-payment goal targets IDR 100m and a planned home purchase includes a cash down payment of IDR 100m. These two records describe potentially the **same** future payment. Without explicit source relationship and precedence, system must warn/seek modeling input; it cannot subtract 200m or claim 100m goal contribution has already been funded. Net-worth identity is illustrated in section 9.1, not a projected housing price calculation.

### EX-05: Reproducibility and stale snapshot

S1 references baseline B1 at as-of T1, event E1 version 1, engine version G1. User adds an actual historical expense that changes the relevant baseline. Old S1 remains explainable against B1. New result requires an explicit B2/version or approved stale action, and a new identity; S1 must not silently use current actuals (`FTC-SCEN-01/04/07`).

---

## 20. Testing approach and evidence classification

Test suite names and exact commands must be derived from the real repository after inspection. **No Go, Flutter, PostgreSQL, or CI test has been run as part of authoring this specification.**

Test layers:

1. **Pure domain/unit tests:** effect validation, date recurrence, rate replacement/delta, money units, duplicate detection, deterministic event normalization.
2. **PostgreSQL integration tests:** owner-scoped CRUD, revisions/concurrent edit, draft-only writes, FK ownership, delete/export, transaction rollback.
3. **API contract/security tests:** `FR-011`, authz/IDOR, errors, write retries, optimistic conflict and payload bounds; check actual ledger remains unchanged.
4. **Flutter unit/widget tests:** `UF-18`, `SCR-13` states, no misleading actual labels, accessible timeline, need-input and stale warnings, lost response.
5. **Engine/golden tests (conditional S-08):** `CS-009`, `CS-012`, `FTC-CASH-*`, `FTC-SCEN-*`, replay, asset/debt conservation and projections after model approval.
6. **E2E/release gates:** S-07 planning save/edit with actual unchanged, S-08 approved forecast and comparison, R2 gate evidence per `TESTING-STRATEGY.md` and `DEFINITION-OF-DONE.md`.

### 20.1 Evidence statuses

| Status | Interpretation |
|---|---|
| `NOT_RUN` | No actual execution; never claim PASS |
| `PASS` | Command, environment, artifact/commit, assertions, run evidence attached |
| `FAIL` | Real test executed and failed; with root-cause/issue |
| `BLOCKED` | Cannot execute because approval/contract/environment missing |
| `SKIPPED` | Test not applicable with explicit owner-reviewed reason |
| `UNVERIFIED` | Repository contents not inspected; implementation status unknown |

Test ID `LPT-xxx` exists only within this document. It complements but does not replace upstream `FTC-*`, `TS-*`, `ST-*`, `UXT-*`, or `DOD-*` references.

---

## 21. Local acceptance test matrix: LPT-001 to LPT-040

**All rows are specifications, status `NOT_RUN`.** Cases with `Q-05/Q-06`, time rules or S-08 engine dependencies remain conditional/blocked until approval. Keep upstream numerical references and approved model oracles as the ultimate expected-value source; do not revise a golden case merely to fit code output.

| ID | Layer | Given / when | Expected result | Upstream anchor / gate |
|---|---|---|---|---|
| `LPT-001` | Go/domain | Save event titled wedding for future date | Event remains PLANNED; actual ledger count and balances unchanged | DR-001/002; FTC-SCEN-02 |
| `LPT-002` | Go/domain | Edit event cost from 10m to 12m in draft | Draft effect changes; no actual expense or asset changes | DR-002; UF-18 |
| `LPT-003` | Go/domain | Delete event from draft | Planning event removed/archived; posted history identical | DR-002; UF-18 |
| `LPT-004` | Go/domain | One-off 10m expense in future within valid horizon | Exactly one planned external outflow of 10m | FR-011; CS-009 policy |
| `LPT-005` | Go/domain | Event outside approved horizon | No projected effect in that horizon | FTC-CASH-09 |
| `LPT-006` | Go/domain | Missing required cost is null | NEEDS_INPUT; not treated as explicit 0 | DR-017 |
| `LPT-007` | Go/domain | User explicitly sets valid zero effect where permitted | Validation distinguishes 0 from absent | Q-04; validation |
| `LPT-008` | Go/domain | Invalid money precision/overflow/non-finite input | Reject without persisting invalid numeric impact | DR-004; Q-04 |
| `LPT-009` | Go/domain | Mismatched currencies without approved FX | Refuse merged numeric projection; show incompatible units | DR-014; FTC-SCEN-05 |
| `LPT-010` | Go/domain | Recurring event with approved finite count of 3 | Three distinct planned occurrences; none posted actual | DR-013; Q-05 |
| `LPT-011` | Go/domain | Monthly event on day 31 across shorter month | Follow approved calendar rule or remain BLOCKED | Q-05; LP-Q04 |
| `LPT-012` | Go/domain | Leap-day yearly recurrence | Follow explicit leap rule; no silent move | Q-05; LP-Q04 |
| `LPT-013` | Go/domain | Two events have same effective date | Deterministic approved order or BLOCKED; not SQL-order dependent | CALC-Q10 |
| `LPT-014` | Go/domain | Two sources represent one logical expense | Flag duplicate/require choice; never silently count twice | LP-I03; LP-Q07 |
| `LPT-015` | Go/domain | Income replacement 10m to 15m | Future salary 15m, not 25m, after effective date | LP-I03; LP-Q06 |
| `LPT-016` | Go/domain | Income delta +5m applied to 10m stream | Future salary 15m only once | LP-I03; LP-Q06 |
| `LPT-017` | Go/domain | Transfer 2m between own modeled cash accounts no fee | Total cash/NW neutral, expense 0 | DR-006; FTC-CASH-03 |
| `LPT-018` | Go/domain | Purchase 3m asset from cash valued at 3m | Cash -3m, asset +3m, net worth delta 0 | FTC-CASH-07; DR-009 |
| `LPT-019` | Go/domain | House purchase example 120m/500m/400m | Hypothetical NW remains 120m before fees; no earned income | LP-I08/09; section 9.1 |
| `LPT-020` | Go/domain | Planned goal contribution 500k per period | Does not create actual funding or new assets | DR-011; FTC-CASH-05 |
| `LPT-021` | DB/integration | Create event with authenticated owner and valid parent | Only owner-scoped planning rows created | DR-003; security |
| `LPT-022` | DB/integration | User B reads/updates User A event by ID | Denied; no data leak or mutation | DR-003; ST matrix |
| `LPT-023` | DB/integration | Owner A attaches event to User B draft/baseline | Denied at nested resource boundary | DR-003; ST matrix |
| `LPT-024` | DB/integration | Concurrent edits with same old draft revision | At most one accepted; other conflict, no lost update | Q-06; API-Q08 |
| `LPT-025` | DB/integration | Save draft request times out after commit | Approved reconciliation avoids duplicate events | LP-I15; API policy |
| `LPT-026` | DB/integration | Malformed effect causes failed save transaction | No partially persisted related effect rows | NFR-006 |
| `LPT-027` | DB/integration | Remove event previously included in frozen scenario | Old immutable version/run remains inspectable per approved policy | Q-06; LP-Q09 |
| `LPT-028` | DB/integration | Persist event, inspect posted accounts before/after | No posted transactions, loan, asset or balance writes | DR-001/002 |
| `LPT-029` | API/security | Spoof owner_id in request body | Server uses authenticated owner, rejects spoofed parent | DR-003 |
| `LPT-030` | API/security | Very long title and unbounded recurrence | Bounded rejection; no heavy workload or log leakage | DR-021; NFR-007 |
| `LPT-031` | Flutter/widget | Open SCR-13 empty timeline | Shows Rencana source and descriptive empty state | UF-18; UXT-20 |
| `LPT-032` | Flutter/widget | Event editor has missing amount or date | Actionable NEEDS_INPUT, no fake preview | SCR-13.V02; DR-017 |
| `LPT-033` | Flutter/widget | User presses Save planned event | Never displays posted transaction confirmation | SCR-13; LP-I02 |
| `LPT-034` | Flutter/widget | Accessibility with timeline events and warnings | Readable date/currency/status without color-only coding | NFR-005; UXT-38 |
| `LPT-035` | Flutter/widget | Old preview arrives after user changes selection | Obsolete response ignored; no stale overwrite | UF-19 alternate |
| `LPT-036` | Engine/S-08 | Same baseline/event version/model replay twice | Identical deterministic output and provenance | CS-009/012; FTC-SCEN-01 |
| `LPT-037` | Engine/S-08 | Actual changes after run B1 | Old result references B1; explicit stale/rebase behavior | DR-020; FTC-SCEN-04 |
| `LPT-038` | Engine/S-08 | Projected cash 1m less 2m expense | Cash shortfall -1m; no implicit loan/clamp | FTC-CASH-04 |
| `LPT-039` | Engine/S-08 | Different baseline/currency/horizon compared | Block or explicit approved normalization; no misleading ranking | FTC-SCEN-05/06 |
| `LPT-040` | E2E/R2 | Create event, version, project, inspect result | Provenance recorded and all actual accounts/transactions unchanged | FR-011/012/013; NFR-010 |

### 21.1 Minimum hard-stop regression set

Even before the full scenario engine exists, testing must prove no side effects and valid owner scoping (`LPT-001/002/003/020/021/022/023/028/029/033`). For S-08 numerical release, additionally require approved calendar, duplicate-source, money, housing asset/debt and reproducibility oracles (`LPT-010/011/012/013/014/015/016/017/018/019/035/036/037/038/039/040`).

### 21.2 Required evidence record

Each executed row should link: test ID, responsible module, exact command, relevant commit or artifact ID, environment/database version, sanitized assertion or test report, run timestamp, `PASS/FAIL/BLOCKED/NOT_RUN`, and unresolved policy gaps. Keep raw production financial data out of test snapshots and logs.

---

## 22. Risk register and edge-case controls

| Risk | Impact | Mitigation / release gate |
|---|---|---|
| Hypothetical event accidentally posted as actual | Corrupt balances, mislead user | Storage/service boundary + before/after integration test (`LP-I01/02`) |
| Double counting budget + recurring + event | Inflate future expenses and false shortfall | Source IDs, overlap detection and explicit precedence (`LP-Q07`) |
| Misclassifying house purchase as a loss | Wrong net worth | Cash/asset/liability conservation and fee separation |
| Treating borrowing as salary | Artificial income, wrong affordability | Principal + liability paired, no income classification |
| Wrong salary replacement semantics | Forecast drastically too optimistic | Replacement/delta intent + effect algebra oracle |
| Calendar/interest period drift | Unreproducible outcome | Approved time grid and same-day ordering, boundary tests |
| Unsupported default return/inflation | False financial certainty | `Q-05`, insufficient-data behavior, copy review |
| Hidden scenario rebase | Unreproducible run, no audit | `Q-06`, immutable version identity, stale signal |
| Mixed currencies | Misleading numeric comparisons | Reject or approved FX normalization only |
| X-user scenario data leak | Privacy/security harm | Backend authorization including nested references |
| Overlong recurrence expansion | Resource exhaustion | Approved event horizon/occurrence limits, budget |
| Editing deleted/deactivated event | Lost data or undefined state | Revision/retention and tombstone policy |
| Device offline/ambiguous write | Duplicate or lost planning edit | Read-back/revision reconciliation, no optimistic actual claim |
| Financial advice implied by UI | User acts on model as guarantee | Label uncertainty, limitations and optional alternative framing |

---

## 23. Implementation slices, owners and prerequisites

### 23.1 S-07 entry gate (EP-S07-02)

Before coding production life planning, verify:

1. R1 actual-account and goal contracts and relevant golden tests are reviewed and have **evidence** where applicable; R1 readiness is not inferred from documentation alone.
2. `PRD Q-06` has approved the minimum planning draft/baseline ownership/identity semantics necessary for S-07 persistence; if only a narrative draft is approved, numerical projection remains off.
3. `PRD Q-05` and related model decisions have approved only the event types/amount/time rules actually being released; unsupported types remain descriptive.
4. API and database contracts for owner-scoped planning writes have been reviewed, then implemented via actual OpenAPI/migrations; draft example routes/tables do not count.
5. `EP-S07-02` has been split into bounded tasks with owners, specific DoR/DoD, blockers and test IDs in `TASKS.md`.
6. `SECURITY-PRIVACY.md` review covers event data privacy, export/delete, retention and nested resources.
7. Frontend `SCR-13` behavior and `UF-18` acceptance states have been verified against the relevant design decisions.

**Suggested decomposition (planning proposal, not newly assigned official Task IDs):** owner-scoped event model + validation; draft persistence/revision; conceptual effect normalization; event timeline/editor UI; privacy/security integration; financial boundary regression; R2/S-07 acceptance review.

### 23.2 S-08 dependency for numerical output

Before the app advertises `Hitung proyeksi`, scenario comparison or future-cash forecasts, the owner must approve `docs/financial/SCENARIO-ENGINE.md`, `Q-05/Q-06`, `CS-009/012` timing and reproducibility policy, plus sufficient API/DB contracts and independently verifiable financial test oracles. S-07 narrative timelines may ship **only if** the release scope and UX labeling are explicitly approved.

### 23.3 Change-control path

Any discovered conflict must be raised to the document that owns the subject: Product PRD for requirement behavior; Financial Domain Rules for financial classification; Calculation Specs for math and units; Database Design for candidate persistence; API Guidelines/OpenAPI for transport; Screen Specs for UI; Decision Log for approval decisions. Update derived docs and tests after owner approval, never silently redefine money or ledger behavior here.

---

## 24. Open decisions and owner approval register

The following `LP-Qxx` IDs are **local cross-references**, not duplicate approvals, and do not create new top-level `Q-xx` in PRD. Parent decisions remain authoritative in [Decision Log](../planning/DECISION-LOG.md).

| ID | Parent decision | Open question | Impact / safe default | Status |
|---|---|---|---|---|
| `LP-Q01` | `PRD Q-06` | Life event owned by which planning draft/default scenario and version from S-07? | No approved persistent parent/version model; restrict to reviewable prototype or explicitly approved draft scope | `BLOCKED` |
| `LP-Q02` | `PRD Q-05` | Which initial event categories and typed impacts are authorized for R2? | Block unsupported event calculation; allow descriptive label only when approved | `TBD` |
| `LP-Q03` | `PRD Q-05` | Date-only vs timestamp and local timezone of event effects? | Block date-sensitive computation/calendar semantics | `BLOCKED` |
| `LP-Q04` | `PRD Q-05` | How monthly 29/30/31, leap day, partial periods, and horizons behave? | No production recurrence expansion until approved | `BLOCKED` |
| `LP-Q05` | `PRD Q-04` | Exact money representation, precision, serialization and rounding for event values? | Block production numeric persistence/aggregation choices | `BLOCKED` |
| `LP-Q06` | `PRD Q-05/Q-06` | Income replacement/delta, start/end and conflicting source precedence? | Block salary-change projections without intent/ordering | `BLOCKED` |
| `LP-Q07` | `PRD Q-05/Q-06` | Duplicate/overlap detection across budget, recurring rule, goal and event? | Warn/seek explicit relationship; do not compute double-counted projection | `BLOCKED` |
| `LP-Q08` | `PRD Q-05` | What housing asset valuation, loan amortization, fees, taxes and depreciation are modeled? | Limit to transparent identity illustration; block full house forecast | `BLOCKED` |
| `LP-Q09` | `PRD Q-06` | How drafts freeze, edit and delete while immutable versions/runs remain? | Block final persistence/version design and retention semantics | `BLOCKED` |
| `LP-Q10` | `PRD Q-06` | Snapshot vs immutable references; stale/rebase and actual revision detection? | Block production scenario calculation and comparison | `BLOCKED` |
| `LP-Q11` | `PRD Q-11` | How goal funded actual / contribution plans link to life-event milestone cost? | No funded percentage or automatic goal posting | `BLOCKED` |
| `LP-Q12` | `PRD Q-05` | Nominal vs real target, inflation basis and one-time inflation application? | No unsupported inflation or purchasing-power claims | `BLOCKED` |
| `LP-Q13` | `PRD Q-06` | What optimistic concurrency and retry/read-after-timeout semantics are final? | Block silent overwrite or duplicate saves | `TBD` |
| `LP-Q14` | `PRD Q-06` | Retention, export/delete policy and privacy fields for old drafts and runs? | Require security/lifecycle approval before persistence release | `TBD` |
| `LP-Q15` | `PRD Q-05/Q-06` | Maximum event count, occurrence horizon and execution budget? | Bound server work or block expensive recurrence | `TBD` |
| `LP-Q16` | `PRD Q-06` | Exactly which S-07 narrative preview may ship before S-08 engine? | No fake numeric preview; release gate approval required | `TBD` |

**Review sequence:** lock the minimal S-07 planning ownership scope and data boundaries (`LP-Q01`); approve the subset of event types/timing/money semantics to release (`LP-Q02` to `LP-Q07`); only then approve S-08 engine and data/version mechanics (`LP-Q08` to `LP-Q12`). Finalize concurrency, privacy and resource limits (`LP-Q13` to `LP-Q15`) before production. `LP-Q16` controls the narrative-only S-07 release boundary.

For each approval, record: decision owner, alternatives considered, rationale, effective date, affected FR/DR/CS/UF/SCR/test IDs, migration/rebaseline impact, reviewer identity and link to written approval. No `TBD` entry is automatically resolved by implementing one proposal.

---

## 25. Definition of Ready, Done, and handoff template

### 25.1 Definition of Ready (for a bounded S-07 task)

- [ ] PRD scope `FR-011` and `EP-S07-02` acceptance subset identified; `TASKS.md` has an approved atomic task.
- [ ] Required dependencies and unresolved `LP-Qxx` decisions linked; only the approved subset moves forward.
- [ ] Exact source-of-truth files inspected, including current repository files and active nested `AGENTS.md`.
- [ ] Actual/plan boundary and authorisation plan documented with negative tests.
- [ ] Example payload/effect grammar and dates reviewed; no implicit inflation, price or FX assumptions.
- [ ] Test layers, exact oracles, fixtures and relevant DoD gates identified before code changes.

### 25.2 Definition of Done (task and feature, not this document)

- [ ] Approved requirements implemented with restricted changes and reviewable commits.
- [ ] All actual balances/transactions/asset/liability records remain unaffected by life-event-only actions, backed by executed integration tests.
- [ ] Backend owner and nested-resource checks verified, privacy controls tested, no sensitive data logged.
- [ ] DB schema and API contract match approved applied migrations/OpenAPI; no hypothetical route or JSON assumed to exist.
- [ ] Flutter timeline/editor and error/incomplete/conflict states tested and accessible.
- [ ] Numerical projections only enabled after S-08 approved model and reference tests pass.
- [ ] Run evidence recorded for `LPT-*`, upstream `FTC-*`, `ST-*`, `UXT-*` and gate cases that apply; no unexplained FAIL/BLOCKED claimed as PASS.
- [ ] `TASKS.md` and `PROGRESS.md` reflect actual status, approvals, test output and follow-up work.
- [ ] Product Owner accepts relevant scope, financial semantics and risk/disclosure conditions; release gate reviewed.

### 25.3 Agent handoff example (fill with actual evidence, never invent)

```text
Task / epic: <approved TK-S07-xx or EP-S07-02 scope>
Scope and FR: FR-011; S-07; related FR-010/012 gates
Decision approvals: <links to explicit approval or BLOCKED list>
Changed files: <actual files or NONE>
Schema/OpenAPI changes: <actual migration/spec links or NONE>
Tests run: <exact commands, versions, fixture, PASS/FAIL/NOT_RUN>
No-actual-mutation proof: <test result or NOT_RUN>
Authorization/IDOR evidence: <test result or NOT_RUN>
Known gaps and risks: <specific issue IDs>
Next allowed task: <task id and preconditions>
Implementation state: UNVERIFIED / IN_PROGRESS / PARTIAL / BLOCKED / DONE with evidence
```

---

## Appendix A. Candidate typed effect sketch (illustrative, not code contract)

```text
LifeEventDraft {
  owner_identity: from authenticated server context
  parent_planning_draft: authorized reference
  id, revision, title, category, planned_date_or_range
  effects: [
    {kind: one_off_external_expense,
     date: explicit calendar date,
     money: {amount: exact-value candidate, currency: IDR},
     provenance: user_input}
  ]
  completeness: COMPLETE | NEEDS_INPUT | UNSUPPORTED (candidate)
  schema_version: candidate
}
```

**Example invalid variants:** string containing executable rules; event with nonexistent parent; cross-owner scenario reference; recurring rule with no bound; mixed-currency arithmetic without approved conversion; absent amount silently converted to zero; one-off expense represented as posted transfer.

The final transport schema, enum spelling, error code, money representation and database tables must be decided under API/DB/financial approvals and validated by contract tests. Flutter should never deserialize this pseudo-structure as a production API guarantee.

---

## Appendix B. Anchor checklist and known dependencies

| Anchor family | Why relevant to life planning | State here |
|---|---|---|
| `FR-011`, `UF-18`, `SCR-13`, `UXT-20/38` | Primary life-event editor acceptance | R2/S-07, pending implementation review |
| `FR-010`, `EP-S07-01`, `UF-17` | Recurring and budget source separation | R2/S-07 dependent; actual posting policy TBD |
| `FR-012/013/014`, `CS-009/012` | Future projections, versions, cash-flow shortfall | R2/S-08 BLOCKED pending engine and approvals |
| `DR-001/002/003/011/012/013/014/016/017/018/020/021/023` | Domain boundary, provenance, privacy, timing and conservation | REQUIRED invariants |
| `FTC-CASH-03/04/05/06/07/08/09` | Nonduplication and forecast arithmetic/reference behaviors | Some conditional on Q-05/Q-06 |
| `FTC-SCEN-01..08` | Reproducibility, stale, currency and authorization | Some conditional on baseline/version policy |
| `LP-I01..16`, `LP-Q01..16`, `LPT-001..040` | Local invariants, decisions and test catalog | All local; tests `NOT_RUN` |
| `Q-04/Q-05/Q-06/Q-11` | Money, assumptions/calendar, baseline/version, goal funding | Await owner decisions |

**Document completion != feature completion.** This specification is internally reviewed but still pending owner approval. The repository, actual environment, OpenAPI contract, migrations, app code and test results were not inspected or executed while authoring this file. `PROGRESS.md` is the place to record future implementation evidence only after real execution.
