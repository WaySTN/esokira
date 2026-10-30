# Esokira — Feature Matrix & Delivery Dependencies

> **Dokumen:** `docs/product/FEATURE-MATRIX.md`  
> **Versi:** `0.1.0`  
> **Tanggal penyusunan:** 26 Oktober 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Sumber kebutuhan:** [`PRD.md`](./PRD.md) v0.1.0  
> **Sumber visi:** [`PRODUCT-VISION.md`](./PRODUCT-VISION.md) v0.1.0  
> **Tujuan:** peta fitur, prioritas, urutan teknis, dan gate agar AI coding agent tidak membangun modul secara acak

## 0. Aturan penggunaan matrix

1. **PRD adalah sumber definisi fitur dan acceptance criteria.** Matrix ini hanya indeks, rencana dependensi, serta status pekerjaan; jangan menggunakan ringkasan matrix untuk mengganti PRD.
2. **Belum ada repository source code yang diaudit.** Status implementasi semua item adalah `UNVERIFIED`, **bukan** klaim sudah atau belum dibuat.
3. **Tidak ada requirement yang memperoleh approval Product Owner secara otomatis.** Semua prioritas/release di sini masih *proposed*, walaupun sudah direview konsistensi internalnya.
4. AI coding agent harus membaca `PRD.md#FR-xxx` (cari ID heading), dokumen teknis relevan, dan test cases sebelum membuat implementasi.
5. Fitur dengan financial modeling yang belum dispesifikasikan dapat direncanakan, tetapi **tidak boleh disebut production-ready**.
6. Jangan mengubah prioritas, release, atau dependency tanpa memperbarui PRD/decision log dan melakukan review dampak.

### Legenda

| Nilai | Arti |
|---|---|
| `P0` | Kebutuhan inti MVP, bukan berarti kode dapat langsung dibuat. |
| `P1` | Planning V1, sesudah core stabil. |
| `P2` | Advanced, butuh engine teruji dan model spesifik. |
| `P3` | Discovery backlog; tidak boleh diimplementasi tanpa spesifikasi baru. |
| `R0` | Engineering Foundation; `R1` MVP; `R2` Planning V1; `R3` Advanced; `R4` discovery. |
| `SPEC_GATED` | Butuh dokumen/domain/design approval untuk bisa diklaim selesai. |
| `DISCOVERY` | Belum ada acceptance siap implementasi. |
| `UNVERIFIED` | Belum ada audit repo; **jangan** dianggap `NOT_STARTED` tanpa inspeksi. |

## 1. Matrix fitur R1 — MVP

| ID | Fitur | Pilar | Prioritas | Depends on | Readiness gate |
|---|---|---|---|---|---|
| **FR-001** | Identity, session, data isolation | V-01 | P0 | EN-002, EN-003, EN-006 | Provider auth diputuskan; authz & cross-user tests. |
| **FR-002** | Financial profile / onboarding | V-01 | P0 | FR-001 | Field required/optional, consent, validation. |
| **FR-003** | Financial accounts & opening balance | V-01 | P0 | FR-001, FR-002, EN-002 | Domain model balance/ownership. |
| **FR-004** | Income, expense, transfer | V-01 | P0 | FR-003 | Posting/transfer/reversal rules & atomic tests. |
| **FR-005** | Assets & liabilities | V-01 | P0 | FR-002, EN-002 | Anti-double-counting & valuation policy. |
| **FR-006** | Net worth & dashboard | V-01 | P0 | FR-003, FR-004, FR-005 | Golden cases net worth and period aggregation. |
| **FR-007** | Financial goals | V-02 | P0 | FR-002 | Model goals; goal funding/progress policy (Q-11). |
| **FR-008** | Compound calculator | V-02 | P0 | EN-004 | Approved formula, rate conventions, golden tests. |
| **FR-009** | Goal contribution/feasibility | V-02 | P0 | FR-007, FR-008, FR-004 | Capacity definition, infeasible tests, formula specs. |
| **FR-021** | Data export & deletion | V-01 | P0 | FR-001, FR-003, FR-004, FR-005, FR-007 | Retention, authorization, secure deletion & export test. |

**Acceptance untuk seluruh R1:** semua FR P0 harus memenuhi acceptance di PRD, data pengguna tidak bocor silang, hasil finansial lulus golden test, dan perhitungan yang bersifat simulasi diberi label jelas.

## 2. Matrix fitur R2 — Planning V1

| ID | Fitur | Pilar | Prioritas | Depends on | Readiness gate |
|---|---|---|---|---|---|
| **FR-010** | Budgets & recurring transactions | V-02 | P1 | FR-004 | Recurring idempotency, period & budget semantics. |
| **FR-011** | Life event planning | V-03 | P1 | FR-007 | Event effects + planning-draft model; no actual mutation. |
| **FR-012** | Deterministic projection | V-03 | P1 | FR-003, FR-004, FR-005, FR-007, FR-008, FR-010, FR-011 | Scenario engine spec + reproducibility tests. |
| **FR-013** | Scenario builder & comparison | V-03 | P1 | FR-011, FR-012 | Baseline/versioning policy, actual isolation tests. |
| **FR-014** | Future cash-flow forecast | V-03 | P1 | FR-012 | Shortfall semantics & future-vs-actual labels. |
| **FR-015** | Financial stress test | V-04 | P1 | FR-013, FR-014 | Shock model and cash resilience cases. |
| **FR-016** | Explainable rule-based insights | V-05 | P1 | FR-006, FR-009, FR-014, FR-015 | Documented decision rules, insufficient-data behavior. |
| **FR-022** | Notifications & reminders | V-02 | P1 | FR-007, FR-016 | Opt-in, privacy, provider + frequency TBD. |

**Acceptance untuk seluruh R2:** skenario dapat dibandingkan dan direproduksi tanpa mengubah data aktual, seluruh asumsi penting dapat diinspeksi, dan insight menyebut alasan serta batasannya.

## 3. Matrix fitur R3 — Advanced

| ID | Fitur | Pilar | Prioritas | Depends on | Readiness gate |
|---|---|---|---|---|---|
| **FR-017** | Multi-goal allocation optimizer | V-04 | P2 | FR-007, FR-009, FR-012, FR-013 | Objective, constraints, infeasible reference cases. |
| **FR-018** | Monte Carlo simulation | V-04 | P2 | FR-012, FR-013, EN-005 | Model risk, seed, statistical golden tests, runtime guardrails. |
| **FR-019** | Retirement / FIRE planner | V-04 | P2 | FR-012, FR-018 | Decumulation assumptions, nominal vs real, depletion tests. |
| **FR-020** | Financial GPS / adaptive roadmap | V-05 | P2 | FR-013, FR-016, FR-017, FR-018 | Alternative scenarios verified, explanation & safety review. |

**Acceptance untuk seluruh R3:** model numerik mendapat review tersendiri, hasil yang bersifat acak dapat direproduksi dengan seed, user memahami sifat model dan trade-off, dan tidak ada transaksi uang yang dijalankan otomatis.

## 4. Matrix R4 — Discovery backlog (tidak siap implementasi)

| ID | Ide | Pilar | Prioritas | Depends on | Status |
|---|---|---|---|---|---|
| **FR-023** | Shared financial goals | V-02 | P3 | FR-001, FR-007, FR-021 | `DISCOVERY`: akses/consent dan acceptance TBD. |
| **FR-024** | Portfolio planning | V-04 | P3 | FR-005, FR-018 | `DISCOVERY`: model data/investasi/kepatuhan TBD. |

Kedua item ini hanya hipotesis. Jangan membuat database, screen, atau API production-nya tanpa discovery, PRD revision, dan approval yang sesuai.

## 5. Engineering enabler matrix

| ID | Enabler | Release gate | Depends on | Bukti minimal |
|---|---|---|---|---|
| **EN-001** | Monorepo & CI baseline | R0 | — | Flutter dan Go dapat build/test terpisah; lint/format dijalankan. |
| **EN-002** | PostgreSQL migrations & ownership base | R0 | EN-001 | Migration up/down diuji, user-owned resource terisolasi. |
| **EN-003** | API contract, validation, middleware | R0 | EN-001, EN-002 | Kontrak terverifikasi; error standar; access checks. |
| **EN-004** | Independent financial engine + test harness | R0 sebelum FR-008 | EN-001 | Unit tests dapat dijalankan tanpa API/DB; golden cases siap dipasang. |
| **EN-005** | Background processing untuk heavy simulation | R3 sebelum FR-018 | EN-002, EN-003 | Job lifecycle, retry/idempotency, timeout, tests. |
| **EN-006** | Secrets, observability & recovery baseline | R0 | EN-001 | Tidak ada secret di repo/log; backup/restore plan. |

**Catatan:** EN-005 tidak memaksa penggunaan Redis atau queue provider tertentu; opsi PostgreSQL-backed worker masih keputusan arsitektur. EN-004 tidak mengizinkan agent mengarang formula untuk mengisi test harness.

## 6. Dependency chains yang harus dihormati

```text
CHAIN A — Secure actual data
EN-001 -> EN-002/EN-003/EN-006 -> FR-001 -> FR-002
                                      -> FR-003 -> FR-004
FR-002 -> FR-005
FR-003 + FR-004 + FR-005 -> FR-006

CHAIN B — First financial plan
EN-004 -> FR-008
FR-002 -> FR-007
FR-007 + FR-008 + FR-004 -> FR-009
FR-001 + FR-003/004/005/007 -> FR-021

CHAIN C — Life planning & prediction
FR-004 -> FR-010
FR-007 -> FR-011
FR-003/004/005/007/008/010/011 -> FR-012
FR-011 + FR-012 -> FR-013
FR-012 -> FR-014 -> FR-015 (also needs FR-013)
FR-006 + FR-009 + FR-014 + FR-015 -> FR-016
FR-007 + FR-016 -> FR-022

CHAIN D — Advanced engines
FR-007/009/012/013 -> FR-017
FR-012/013 + EN-005 -> FR-018
FR-012 + FR-018 -> FR-019
FR-013 + FR-016 + FR-017 + FR-018 -> FR-020
```

Setiap panah menunjukkan dependensi rancangan, **bukan** jaminan bahwa fitur pengikut baru boleh dipikirkan setelah semua predecessor selesai. Prototype isolated boleh untuk riset jika jelas bukan production implementation.

## 7. Suggested delivery slices untuk AI coding agent

Satu slice harus menghasilkan perilaku teruji, bukan sekadar scaffolding. Berikut **rencana awal**, bukan sprint commitment.

| Slice | Isi utama | Exit proof |
|---|---|---|
| **S-00** | Approval dokumen awal, `AGENTS.md` minimum, aturan domain & financial tests. | Scope disetujui; blocker inti tercatat; golden cases disiapkan. |
| **S-01** | EN-001, EN-002, EN-003, EN-006. | Backend/Flutter build, CI, migrasi, authz scaffolding teruji. |
| **S-02** | FR-001, FR-002. | Login, isolation test, onboarding terbatas end-to-end. |
| **S-03** | FR-003, FR-004. | Input saldo dan transaksi; transfer atomik dan benar. |
| **S-04** | FR-005, FR-006. | Net worth dashboard cocok dengan test ledger. |
| **S-05** | EN-004, FR-007, FR-008. | Goal dapat dibuat; compound golden cases lulus. |
| **S-06** | FR-009, FR-021. | Rencana pertama dapat disimpan; data export/deletion diuji. **Gate R1**. |
| **S-07** | FR-010, FR-011. | Budget/event model siap tanpa mencampur actuals. |
| **S-08** | FR-012, FR-013, FR-014. | Minimal dua versi skenario dengan cash-flow result reproducible. |
| **S-09** | FR-015, FR-016, FR-022. | Stress/insights/reminder yang aman. **Gate R2**. |
| **S-10** | FR-017 dan EN-005. | Optimizer infeasible cases + reliable worker. |
| **S-11** | FR-018, FR-019. | Seeded Monte Carlo + retirement assumptions tervalidasi. |
| **S-12** | FR-020. | Alternative planning path terjelaskan dan dapat diuji. **Gate R3**. |

S-00 adalah **blocking planning gate**, bukan alasan agar agent mengarang keputusan yang belum ada. Slice berikutnya boleh dipecah menjadi task lebih kecil menurut modul dan acceptance PRD.

## 8. Gate dokumentasi per kelompok

| Kumpulan fitur | Dokumentasi wajib sebelum implementasi production |
|---|---|
| R0 dan seluruh coding | `AGENTS.md`, `SYSTEM-ARCHITECTURE.md`, `BACKEND-ARCHITECTURE.md` / `FRONTEND-ARCHITECTURE.md` sesuai area, `DATABASE-DESIGN.md`, `SECURITY-PRIVACY.md`. |
| FR-003–FR-006 | `FINANCIAL-DOMAIN-RULES.md`, ledger/transfer/valuation test reference. |
| FR-008–FR-009 | `CALCULATION-SPECS.md`, `FINANCIAL-TEST-CASES.md`. |
| FR-011–FR-015 | `SCENARIO-ENGINE.md`, deterministic projection cases, OpenAPI. |
| FR-017 | `GOAL-OPTIMIZER.md` dengan reference infeasible/feasible cases. |
| FR-018–FR-019 | `MONTE-CARLO.md`, retirement assumptions, stochastic test protocol. |
| FR-020 | Rules for explainability and recommendation boundaries, evidence dari engine output. |
| FR-021–FR-023 | `SECURITY-PRIVACY.md`, data retention/consent/ownership tests. |

Penamaan dokumen tersebut mengikuti rencana knowledge base proyek. Jika file belum ada, statusnya `BLOCKED_BY_DOC`, bukan alasan untuk menulis spesifikasi finansial secara spontan di dalam kode.

## 9. Standard workflow untuk memperbarui matrix

Status implementasi yang **boleh dipakai setelah repo mulai diaudit**:

`UNVERIFIED → NOT_STARTED → READY → IN_PROGRESS → IN_REVIEW → DONE`  
Alternatif `BLOCKED` dapat digunakan dari status apa pun, disertai sebab; `CANCELLED` memerlukan keputusan Product Owner.

Definisi ringkas:

- **UNVERIFIED:** tidak ada pemeriksaan kode/repo yang memadai.
- **NOT_STARTED:** repo sudah diperiksa dan implementasi belum ada.
- **READY:** PRD + docs + tests spesifik ready, dependencies terpenuhi.
- **IN_PROGRESS:** satu atau lebih task sedang diimplementasikan, belum lulus gate.
- **IN_REVIEW:** acceptance dan tes tersedia; pemeriksaan akhir masih berlangsung.
- **DONE:** bukti acceptance, testing, security, dan review dicatat.
- **BLOCKED:** isu yang mencegah transisi; tulis referensi blocker/decision.

Ketika mulai coding, simpan tracking tugas operasional pada `docs/planning/TASKS.md` dan `PROGRESS.md`. Matrix ini tetap menjadi **indeks requirement**, bukan log harian agent.

Template update yang bisa ditambahkan kemudian untuk tiap feature:

```yaml
id: FR-008
spec_status: REVIEWED_INTERNAL
implementation_status: UNVERIFIED
assigned_task: null
blocked_by: ["Q-04", "CALCULATION-SPECS.md", "FINANCIAL-TEST-CASES.md"]
proof: []  # commit, test output, review reference setelah benar-benar ada
```

Jangan mengganti `proof` dengan klaim yang belum dapat diverifikasi.

## 10. Risiko dependensi dan keputusan yang masih terbuka

| Risiko | FR terdampak | Aturan tindakan |
|---|---|---|
| Auth provider belum dipilih | FR-001 dan seluruh data privat | Jangan mengunci SDK/provider tanpa keputusan. |
| Model ledger, goal funding, nominal, pembulatan belum disetujui | FR-003–FR-009, FR-012–FR-020 | Tetapkan domain rules dan reference tests terlebih dahulu (Q-03, Q-04, Q-11). |
| Definisi snapshot/version scenario belum diputuskan | FR-012–FR-020 | Jangan mencampur actual input dengan hypothetical output. |
| Model risiko statistik belum tervalidasi | FR-018, FR-019, FR-020 | Tidak ada release claim probabilitas/kesuksesan. |
| Biaya dan strategi deployment belum jelas | EN-005, EN-006, FR-018 | Batas resource dan pemrosesan berat perlu didesain. |
| Consent dan retensi belum jelas | FR-021, FR-022, FR-023 | Tidak boleh melepaskan fitur data-sensitive ke production. |

## 11. Review internal dan persetujuan

**Validasi yang perlu dijalankan setiap ada perubahan:**

- [x] Setiap FR-001–FR-024 di matrix memiliki definisi di PRD.
- [x] EN-001–EN-006 konsisten dengan PRD.
- [x] Tidak ada dependency cycle pada graph saat penyusunan awal.
- [x] Semua dependensi merujuk ID fitur/enabler yang ada.
- [x] R1/R2/R3/R4, P0/P1/P2/P3, dan pilar Vision selaras dengan PRD.
- [x] R4 secara eksplisit discovery-only.
- [x] Status implementasi `UNVERIFIED` tidak mengklaim sudah ada source code yang diperiksa.
- [ ] **Product Owner menyetujui urutan rilis dan dependency prioritas.**
- [ ] **Audit source code dilakukan sebelum status implementasi diubah.**

**Hasil review internal:** LULUS sebagai dependency plan awal, **pending owner approval**.
