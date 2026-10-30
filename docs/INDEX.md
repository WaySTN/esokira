# Esokira — Documentation Index & Source-of-Truth Router

> **Canonical path:** `docs/INDEX.md`  
> **Version:** `0.1.0`  
> **Prepared:** `2026-10-30` (Asia/Jakarta)  
> **Document status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Purpose:** Navigasi dan tata kelola dokumentasi lintas produk, finansial, arsitektur, desain, keamanan, dan implementasi.  
> **Document inventory:** 25 dokumen yang disusun sebelum indeks ini, ditambah indeks ini = **26 Markdown** pada baseline dokumentasi.  
> **Planning baseline:** 37 Markdown terencana; setelah indeks ini terdapat **11 Markdown yang belum disusun** (lihat §11).  
> **Implementation audit:** `UNVERIFIED` — repository aplikasi, skema yang diterapkan, dan deployment aktual belum diperiksa dalam penyusunan indeks.  
> **Application tests:** `NOT_RUN` — pemeriksaan integritas dokumentasi **bukan** bukti test Flutter, Go, PostgreSQL, atau CI.  
> **Owner approval:** `NOT_RECORDED` untuk keputusan terbuka; indeks ini tidak mengesahkan pilihan teknis baru.

---

## 0. Mulai dari sini: kontrak penggunaan

**Esokira** adalah aplikasi mobile personal financial life planning dan simulation. Pilihan proyek yang sudah dinyatakan pemilik adalah **Flutter** (frontend), **Go** (backend), **PostgreSQL** (database utama), serta **monorepo** dengan `frontend/` dan `backend/`. Detail seperti provider identity, model ledger, format presisi uang, router dan library, kontrak API final, distribusi Monte Carlo, dan deployment **belum otomatis diputuskan**.

Indeks ini merupakan **peta**, bukan pengganti dokumen spesifikasi atau bukti bahwa folder/kode di monorepo sudah ada.

1. **Untuk AI coding agent:** baca [`../AGENTS.md`](../AGENTS.md) dahulu. Jika mengubah Flutter atau Go, baca juga [`../frontend/AGENTS.md`](../frontend/AGENTS.md) atau [`../backend/AGENTS.md`](../backend/AGENTS.md). Instruksi folder menambah, tidak melemahkan, aturan root.
2. **Untuk requirement:** cari ID `FR-*`, `EN-*`, atau `NFR-*` di [`product/PRD.md`](product/PRD.md); lihat urutan dependensinya di [`product/FEATURE-MATRIX.md`](product/FEATURE-MATRIX.md).
3. **Untuk pekerjaan nyata:** cek [`planning/TASKS.md`](planning/TASKS.md), keputusan di [`planning/DECISION-LOG.md`](planning/DECISION-LOG.md), dan bukti status di [`planning/PROGRESS.md`](planning/PROGRESS.md).
4. **Untuk perubahan finansial:** baca domain rules + calculation specs + financial test cases yang terkait sebelum coding. Tidak boleh menganggap contoh angka sebagai kebijakan final.
5. **Untuk acceptance:** ikuti [`development/DEVELOPMENT-WORKFLOW.md`](development/DEVELOPMENT-WORKFLOW.md), [`development/TESTING-STRATEGY.md`](development/TESTING-STRATEGY.md), dan [`development/DEFINITION-OF-DONE.md`](development/DEFINITION-OF-DONE.md).
6. **Untuk konflik atau keputusan kosong:** cek sumber otoritatif, catat `DOCUMENTATION_DRIFT`/`BLOCKED` untuk pekerjaan terdampak, dan jangan memilih opsi `PROPOSED`/`TBD` tanpa persetujuan pemilik. Pekerjaan independen boleh dilanjutkan jika scope-nya jelas.
7. **Untuk verifikasi keberadaan:** periksa file yang *benar-benar ada* dalam workspace pengguna. Daftar di bawah berdasarkan artefak Markdown yang telah disiapkan; **bukan** hasil audit monorepo pengguna.

**Aturan keamanan dan keakuratan:** simulasi tidak boleh memodifikasi transaksi/saldo aktual; planned contribution tidak sama dengan goal funding; transfer antar-rekening sendiri bukan pendapatan/beban; request mutasi aktual memerlukan idempotency serta penanganan outcome tidak pasti; seluruh akses privat harus diperiksa otorisasinya di backend; hasil finansial harus dapat ditelusuri ke rumus, asumsi, dan uji yang relevan. Jangan membocorkan data finansial pribadi, token, atau secret melalui log, fixture, prompt, ataupun dokumen progres.

---

## 1. Dashboard dokumentasi — baseline 2026-10-30

| Indikator | Status | Interpretasi |
|---|---|---|
| Target inventaris awal | **37** file `.md` | Target perencanaan yang telah dibahas, bukan syarat mutlak untuk mulai coding. |
| Dokumen sebelum INDEX | **25** | Disusun sebagai artefak Markdown/ZIP pada rangkaian dokumentasi. |
| Dokumen ini | **1** | `docs/INDEX.md` menjadi file ke-26. |
| Total disusun setelah dokumen ini | **26 / 37** | Sekitar 70,3% dari inventaris yang direncanakan. |
| Belum disusun | **11** | Daftar lengkap dan rekomendasi urutan di §11. |
| Dokumen yang otomatis dianggap owner-approved | **0** | `REVIEWED_INTERNAL` tidak sama dengan approval pemilik. |
| Status implementasi yang dapat disimpulkan dari dokumen | **`UNVERIFIED`** | Tidak boleh menganggap sudah terimplementasi ataupun pasti belum ada kode. |
| Status tes aplikasi yang dapat disimpulkan dari dokumen | **`NOT_RUN`** | Golden test cases terdokumentasi, tetapi belum membuktikan test telah dieksekusi. |

**Batas snapshot:** angka 26/37 menggambarkan dokumen yang sudah **disiapkan** di rangkaian artefak proyek ini. Angka tersebut tidak membuktikan bahwa 26 file telah di-commit ke monorepo. Jika file kemudian ditambahkan, diganti nama, atau dipindah, perbarui indeks dan rekonsiliasi dengan isi repository aktual.

---

## 2. Inventaris 26 dokumen — tautan kanonis

Semua tautan pada tabel ini **relatif terhadap `docs/INDEX.md`** dan akan dapat ditavigasi setelah seluruh artefak ditempatkan sesuai path kanonis di monorepo. Satu ZIP khusus indeks **tidak** berisi 25 berkas pendukungnya.

### 2.1 Instruksi agent dan router (4 dokumen termasuk indeks)

| File | Peran / kapan dibaca | Otoritas |
|---|---|---|
| [`../AGENTS.md`](../AGENTS.md) | Instruksi global untuk setiap perubahan kode atau dokumentasi | Aturan kerja agent root |
| [`../frontend/AGENTS.md`](../frontend/AGENTS.md) | Aturan Flutter, integrasi API, state, money UI, keamanan lokal | Aturan khusus `frontend/` |
| [`../backend/AGENTS.md`](../backend/AGENTS.md) | Aturan Go, PostgreSQL, transaksi, worker, financial engine | Aturan khusus `backend/` |
| **`docs/INDEX.md`** (file ini) | Peta dokumen, prioritas pembacaan, kelengkapan dan traceability | Navigator, **bukan** sumber requirement |

### 2.2 Product (3 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`product/PRODUCT-VISION.md`](product/PRODUCT-VISION.md) | Masalah produk, pengguna, tujuan, batasan, dan non-goals. |
| [`product/PRD.md`](product/PRD.md) | Perilaku fitur dan acceptance; pemilik definisi `FR-001`–`FR-024`, `EN-001`–`EN-006`, `NFR-001`–`NFR-010`, serta keputusan induk `Q-01`–`Q-11`. |
| [`product/FEATURE-MATRIX.md`](product/FEATURE-MATRIX.md) | Prioritas, fase R0–R4, dependensi fitur, dan delivery slices `S-00`–`S-12`. |

### 2.3 Financial domain & calculation (3 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`financial/FINANCIAL-DOMAIN-RULES.md`](financial/FINANCIAL-DOMAIN-RULES.md) | Definisi actual/planned/funded, ledger, transfers, goals, invariants `DR-001`–`DR-024`. |
| [`financial/CALCULATION-SPECS.md`](financial/CALCULATION-SPECS.md) | Kontrak matematika `CS-001`–`CS-016`, input, output, satuan, batas rumus dan asumsi. |
| [`financial/FINANCIAL-TEST-CASES.md`](financial/FINANCIAL-TEST-CASES.md) | Vektor dan oracle uji `FTC-*`, edge cases, prasyarat golden-test approval. |

### 2.4 Architecture & API (5 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`architecture/SYSTEM-ARCHITECTURE.md`](architecture/SYSTEM-ARCHITECTURE.md) | Batas sistem, komponen, trust boundaries, jalur data, non-functional architecture. |
| [`architecture/BACKEND-ARCHITECTURE.md`](architecture/BACKEND-ARCHITECTURE.md) | Modul Go, boundary use case/repository, financial engine, asynchronous jobs. |
| [`architecture/FRONTEND-ARCHITECTURE.md`](architecture/FRONTEND-ARCHITECTURE.md) | Struktur Flutter, layar, state, API client, penyimpanan lokal dan UX kondisi async. |
| [`architecture/DATABASE-DESIGN.md`](architecture/DATABASE-DESIGN.md) | Konsep entitas, invariants, constraints, migrasi usulan, ownership, transaksi. |
| [`architecture/API-GUIDELINES.md`](architecture/API-GUIDELINES.md) | Kebijakan REST, request/response, error codes, auth, idempotency, versi kontrak. |

> **Perhatikan:** usulan SQL/API di dokumen arsitektur adalah rancangan. Bentuk API mesin yang final kelak berasal dari `contracts/openapi.yaml` **jika sudah dibuat dan disetujui**; bentuk database yang benar-benar diterapkan berasal dari migrasi yang nyata dan telah diperiksa. Path tersebut **bukan anggota inventaris Markdown** dan belum terbukti ada hanya karena disebut di dokumentasi.

### 2.5 Design & user experience (3 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`design/DESIGN-SYSTEM.md`](design/DESIGN-SYSTEM.md) | Design tokens, komponen reusable, typography, colors, accessibility; keputusan token yang terbuka tetap proposed. |
| [`design/USER-FLOWS.md`](design/USER-FLOWS.md) | Alur pengguna dan error/recovery `UF-00`–`UF-31`. |
| [`design/SCREEN-SPECS.md`](design/SCREEN-SPECS.md) | Spesifikasi dan state layar `SCR-00`–`SCR-22`, verifikasi UX `UXT-*`. |

### 2.6 Security & privacy (1 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`security/SECURITY-PRIVACY.md`](security/SECURITY-PRIVACY.md) | Security/privacy controls `SEC-001`–`SEC-026`, threat model `TH-01`–`TH-14`, security tests `ST-001`–`ST-075`, data governance dan gate privasi. |

### 2.7 Development quality & operations of work (3 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`development/DEVELOPMENT-WORKFLOW.md`](development/DEVELOPMENT-WORKFLOW.md) | Inspect → Trace → Plan → Implement → Verify → Handoff; pengelolaan task, blocker dan bukti. |
| [`development/TESTING-STRATEGY.md`](development/TESTING-STRATEGY.md) | Strategi dan paket uji `TS-001`–`TS-040`, CI quality gates, Go/Flutter/DB/API/security/financial tests. |
| [`development/DEFINITION-OF-DONE.md`](development/DEFINITION-OF-DONE.md) | Acceptance dan 10 universal gates `DOD-G01`–`DOD-G10` untuk task, fitur, modul, dan release. |

### 2.8 Planning, keputusan, dan progres (4 dokumen)

| File | Peran / sumber kebenaran |
|---|---|
| [`planning/MASTER-ROADMAP.md`](planning/MASTER-ROADMAP.md) | Implementasi bertahap R0–R3, `S-00`–`S-12`, ketergantungan, entry/exit gates. |
| [`planning/TASKS.md`](planning/TASKS.md) | **Backlog rencana:** 58 task rinci R0/R1 + 13 epic R2/R3, ID `TK-*`/`EP-*`, scope, dependensi, acceptance. |
| [`planning/PROGRESS.md`](planning/PROGRESS.md) | **Ledger observasi:** status aktual, bukti test/review, blocker, session log, handoff; baseline semuanya `UNVERIFIED`. |
| [`planning/DECISION-LOG.md`](planning/DECISION-LOG.md) | **Register keputusan:** 11 keputusan induk, indeks pertanyaan teknis, approval trail dan dependency blockers. |

**Rekonsiliasi total:** instruksi/indeks 4 + product 3 + financial 3 + architecture 5 + design 3 + security 1 + development 3 + planning 4 = **26** file. Jumlah tersebut tidak termasuk 11 file terencana, source code, migrasi, OpenAPI, ZIP, test fixture, atau deployment artifact.

---

## 3. Urutan membaca dokumen — jangan memuat semuanya sekaligus

Gunakan **minimum sufficient context**: instruksi yang berlaku + requirement yang dituju + domain owner docs + dokumen pengujian/keputusan. Pindah ke bagian yang diperlukan melalui pencarian ID; tidak perlu selalu membuka seluruh Markdown puluhan ribu baris.

| Jenis perubahan | Wajib mulai dari | Tambahkan jika berdampak |
|---|---|---|
| Mengambil task / memulai sesi | Root AGENTS → `TASKS` → `PROGRESS` → `DECISION-LOG` | `MASTER-ROADMAP`, workflow, DoD |
| UI Flutter / widget / screen | Root + frontend AGENTS → `SCREEN-SPECS` → `USER-FLOWS` → `DESIGN-SYSTEM` | `FR-*`, `FRONTEND-ARCHITECTURE`, API, security |
| Navigation dan async/error state | Frontend AGENTS → `SCREEN-SPECS` → `USER-FLOWS` | API Guidelines, Security & Privacy, DoD |
| API request/response, auth middleware | Root + backend AGENTS → `PRD` → `API-GUIDELINES` → Security | Database, Backend Architecture, approved OpenAPI *jika ada* |
| PostgreSQL/migrations/tenant ownership | Backend AGENTS → `DATABASE-DESIGN` → `FINANCIAL-DOMAIN-RULES` | API, Security, relevant `FTC-*`, actual migrations *jika ada* |
| Actual accounts, transfer, correction | `PRD` → `FINANCIAL-DOMAIN-RULES` → `DATABASE-DESIGN` | Backend, API, Security, Financial Test Cases, DoD |
| Net worth / asset valuation | `PRD` → `FINANCIAL-DOMAIN-RULES` → `CALCULATION-SPECS` | `FINANCIAL-TEST-CASES`, Database, Screen Specs |
| Goals / compound / feasibility | `FR-007`–`FR-009` → `DR-*` → `CS-*` → `FTC-*` | Goal UI, Backend Architecture, decision `Q-04/Q-11` |
| Life event / scenario / projections | `FR-011`–`FR-016` → DR/CS/FTC → Roadmap | User Flows, Backend, API, `Q-05/Q-06`, future scenario spec |
| Monte Carlo / optimizer / FIRE / GPS | R3 `FR-017`–`FR-020` → DR/CS/FTC → R3 Roadmap | Backend workers, `Q-07/Q-08`, *future* model-specific docs; do not finalize early |
| Identity, data export/deletion, privacy | `PRD` + `SECURITY-PRIVACY` + `API-GUIDELINES` | Database, frontend lifecycle, consent/retention decisions |
| Infrastructure / CI / release | Root AGENTS → `MASTER-ROADMAP` → `TESTING-STRATEGY` → DoD | System Architecture, Security, `DECISION-LOG`/`Q-10` |
| Bug/regression | `TASKS` + `PROGRESS` evidence → module source → test oracle | Workflow, DoD, relevant controls/decisions |
| Mengubah dokumen | Dokumen owner + dependents → `DECISION-LOG` | `INDEX`, `TASKS`, `PROGRESS`, tests dan arsitektur jika terdampak |

**Jangan jadikan INDEX sebagai shortcut untuk mengabaikan instruksi AGENTS atau acceptance PRD.** Jika perubahan menyentuh beberapa domain, gabungkan jalur baca yang relevan saja.

---

## 4. Hirarki otoritas dan penyelesaian konflik

Ini bukan aturan bahwa "dokumen paling baru selalu benar". Sumber kebenaran ditentukan oleh **jenis kontrak** dan **persetujuan yang tercatat**.

| Pertanyaan | Sumber utama | Hal yang **tidak** boleh menggantikannya |
|---|---|---|
| Mengapa produk ada / apa non-goal? | Product Vision | Klaim dalam roadmap atau sebuah mockup |
| Perilaku produk dan acceptance? | PRD (`FR-*`, `EN-*`, `NFR-*`) | Task summary, UX label, rencana endpoint |
| Kapan dan dalam slice apa? | Feature Matrix + Master Roadmap | Nama folder atau urutan task yang dikarang agent |
| Apa arti jumlah uang/goal/transfer? | Financial Domain Rules (`DR-*`) | UI formatting, contoh random, SQL sample |
| Bagaimana suatu angka dihitung? | Calculation Specs (`CS-*`) + oracle Financial Test Cases (`FTC-*`) | Nilai hardcoded pada layar atau unit test buatan sendiri tanpa review |
| API contract yang dideploy? | Approved OpenAPI + verified implementation *jika ada*; policy di API Guidelines | Contoh path/body di dokumentasi sebelum approval |
| Database structure yang dideploy? | Verified applied migrations/constraints *jika ada*; rancangan di Database Design | DDL ilustratif di Markdown yang belum dijalankan |
| UI appearance/interaction? | Design System, User Flows, Screen Specs, PRD | Screenshot mockup sebagai pengganti state/error requirements |
| Security/privacy? | Security & Privacy + relevant NFR, trust boundaries | Convenience implementation atau bypass auth demi demo |
| Apa yang harus dilakukan agent? | Root AGENTS + folder AGENTS + Development Workflow | Ringkasan dalam INDEX atau jawaban AI lain |
| Task planned vs status sebenarnya? | TASKS = planned; PROGRESS = observed | Checkbox centang dari dokumen yang belum dieksekusi |
| Apakah keputusan sudah disetujui? | DECISION-LOG dengan bukti/approval owner | Banyaknya dokumen yang mengulang satu proposal |
| Boleh menyebut DONE/releasable? | Definition of Done + Testing Strategy + bukti hasil + approval yang relevan | Build hijau tunggal, checklist tanpa evidence |

Jika menemukan konflik:

1. Catat **kedua path**, heading/ID tepat, dan status persetujuan masing-masing.
2. Tetapkan **ruang lingkup yang terdampak** dan risiko; jangan menyelesaikan kebijakan secara sepihak.
3. Tulis usulan opsi/konsekuensi di `DECISION-LOG`; jika terkait kode yang sudah ada, catat `DOCUMENTATION_DRIFT` di `PROGRESS`.
4. Tandai task yang bergantung pada pilihan itu `BLOCKED` atau `PARTIAL` sesuai definisi workflow; pekerjaan lain boleh berjalan.
5. Setelah Product Owner memberi keputusan eksplisit, sinkronkan **dokumen owner → dependent docs → tests → task gates**, baru implementasikan atau refactor kode yang terkait.

`REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` adalah hasil pemeriksaan dokumentasi, **bukan** persetujuan requirement, implementasi, legal compliance, atau deployment.

---

## 5. Peta identifier dan traceability

| Prefix / rentang | Lokasi pertama untuk mencari | Makna |
|---|---|---|
| `V-*`, `O-*` | `PRODUCT-VISION.md`, `PRD.md` | Pilar visi/outcomes. |
| `FR-001`–`FR-024` | `PRD.md` | Functional requirements; `FR-023/024` adalah **R4 discovery**, bukan pekerjaan yang boleh langsung di-build. |
| `EN-001`–`EN-006` | `PRD.md`, `FEATURE-MATRIX.md` | Engineering enablers. |
| `NFR-001`–`NFR-010` | `PRD.md` | Non-functional requirements. |
| `DR-001`–`DR-024` | `FINANCIAL-DOMAIN-RULES.md` | Financial semantics dan invariants. |
| `CS-001`–`CS-016` | `CALCULATION-SPECS.md` | Calculation contracts. |
| `FTC-*` | `FINANCIAL-TEST-CASES.md` | Reference financial test cases / oracle. |
| `UF-00`–`UF-31` | `USER-FLOWS.md` | User journeys. |
| `SCR-00`–`SCR-22` | `SCREEN-SPECS.md` | Screen families / UI states. |
| `UXT-*` | `SCREEN-SPECS.md` | UX acceptance/test references. |
| `SEC-001`–`SEC-026` | `SECURITY-PRIVACY.md` | Security/privacy controls. |
| `TH-01`–`TH-14` | `SECURITY-PRIVACY.md` | Threat inventory. |
| `ST-001`–`ST-075` | `SECURITY-PRIVACY.md` | Security verification references. |
| `TS-001`–`TS-040` | `TESTING-STRATEGY.md` | Test work packages. |
| `DOD-G01`–`DOD-G10` | `DEFINITION-OF-DONE.md` | Universal quality/acceptance gates. |
| `Q-01`–`Q-11` | `PRD.md`, `DECISION-LOG.md` | Keputusan produk induk. |
| `ARCH-Q*`, `BE-Q*`, `FE-Q*`, `DB-Q*`, `API-Q*` dan prefix lokal lain | Dokumen domain masing-masing, indeks di `DECISION-LOG.md` | Pertanyaan teknis **lokal**, bukan alias otomatis `Q-*` dengan nomor sama. |
| `S-00`–`S-12` | `FEATURE-MATRIX.md`, `MASTER-ROADMAP.md` | Delivery slices. |
| `TK-*`, `EP-*` | `TASKS.md` | Identitas task rinci dan epic; gunakan ID persis dari backlog. |

**Cara melakukan trace:** `FR → DR/CS (jika finansial) → UF/SCR (jika UI) → API/DB/Security (jika data) → FTC/TS/ST/UXT → task TK/EP → evidence PROGRESS → DOD gate`. Tidak semua FR membutuhkan semua kategori; hanya tautkan yang benar-benar relevan. Jangan menciptakan ID baru untuk menutupi celah traceability.

---

## 6. Peta fase implementasi dan batas ruang lingkup

Fase/slice berikut adalah **rencana**, belum berarti sudah dimulai atau disetujui untuk produksi.

| Fase | Slice | Deliverable yang dituju | Gate pembacaan/keputusan |
|---|---|---|---|
| Planning | `S-00` | Audit repository, product/architecture readiness, decision triage | PRD, TASKS, PROGRESS, DECISION-LOG; tidak mengaku sudah ada aplikasi. |
| R0 | `S-01` | Monorepo, API/DB/auth foundation, CI, financial engine test harness | Q-02/Q-03/Q-04/Q-10 untuk bagian yang tergantung keputusan; EN-001/002/003/006 dan readiness EN-004. |
| R1 | `S-02` | Identity, session, financial profile | FR-001/002, authz, consent/privacy. |
| R1 | `S-03` | Accounts, opening balance, income, expense, transfer | FR-003/004, Q-03/Q-04, atomicity, idempotency. |
| R1 | `S-04` | Assets, liabilities, net worth/dashboard | FR-005/006, valuation dan anti-double-counting. |
| R1 | `S-05` | Goals dan compound calculator | FR-007/008; persetujuan rumus, time/rate precision, Q-11; implementasi numerik EN-004. |
| R1 | `S-06` | Feasibility, secure export/deletion, R1 exit gate | FR-009/021, hak data, tests, DoD. |
| R2 | `S-07` | Budget/recurring + life events | FR-010/011; rencana baru boleh dirinci sesudah gate relevan. |
| R2 | `S-08` | Deterministic projection, scenario compare, forecast | FR-012/013/014; Q-05/Q-06 dan spesifikasi scenario engine. |
| R2 | `S-09` | Stress tests, insights, notifications | FR-015/016/022; consent dan decision Q-09. |
| R3 | `S-10` | Multi-goal optimizer, worker foundation | FR-017, EN-005, Q-07. |
| R3 | `S-11` | Monte Carlo + Retirement/FIRE | FR-018/019; Q-08, statistical validity, model provenance. |
| R3 | `S-12` | Financial GPS + R3 exit gate | FR-020; explainability, actual/hypothetical separation. |
| R4 | *Discovery only* | Shared goals / portfolio planning concepts | FR-023/024 tidak otomatis masuk delivery plan. |

**Khusus EN-004:** readiness pure-engine dan test harness direncanakan pada **R0/S-01**, sedangkan perhitungan compound yang disetujui dan golden tests relevannya diselesaikan pada **R1/S-05**. Jangan menyamakan kesiapan infrastruktur dengan fitur finansial yang sudah lulus acceptance.

---

## 7. Keputusan yang memblokir dan urutan review

Prioritaskan keputusan berdasarkan slice yang akan dikerjakan, bukan urutan angka semata. Status keputusan induk saat baseline adalah **`NOT_RECORDED`**, bukan berarti opsinya tidak pernah didiskusikan.

| Keputusan induk | Ruang lingkup | Diperlukan paling lambat |
|---|---|---|
| `Q-01` | Brand kerja, audiens, locale, cakupan MVP | `S-00` |
| `Q-02` | Provider auth, session, principal validation | `S-01/S-02` |
| `Q-03` | Ledger/opening balance/transfer/reversal/correction | `S-01/S-03` |
| `Q-04` | Presisi uang, waktu/rate, rounding dan representasi | `S-01/S-03/S-05` |
| `Q-05` | Asumsi inflation/return/debt untuk projection | `S-07/S-08` |
| `Q-06` | Scenario baseline/version/rebase/stale behavior | `S-07/S-08` |
| `Q-07` | Objective dan constraints optimizer | `S-10` |
| `Q-08` | Probabilistic model, correlation, seed, disclosure | `S-11` |
| `Q-09` | Consent/channel/frequency notification | `S-09` |
| `Q-10` | Hosting, backup, retention, offline/sync, legal/release review | `S-00` baseline; sebelum R1 live-data rollout |
| `Q-11` | Actual goal funding/progress dan anti-double-counting | `S-05` |

Cari sub-keputusan di [`planning/DECISION-LOG.md`](planning/DECISION-LOG.md), kemudian buka **dokumen asalnya** untuk konteks lengkap. Jangan membuat salinan `Q-*` kedua dengan makna baru.

**Tidak boleh membangun sebagai production default tanpa approval:** auth/provider, detail ledger, representasi/pembulatan uang, migrasi final, kontrak request-response final, metode probabilistik, preferensi/opt-in, retensi data, atau deployment/provider cloud.

---

## 8. Model status: bedakan dokumen, keputusan, tugas, test, dan release

| Lapisan | Status awal yang benar | Bukti untuk mengubah status |
|---|---|---|
| Dokumen | `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` | Owner approval eksplisit untuk lingkup dokumen dan revisinya. |
| Keputusan teknis/produk terbuka | `NOT_RECORDED`, `PROPOSED`, atau `TBD` sesuai source | Catatan pilihan, alasan, owner, tanggal, impact, referensi di Decision Log. |
| Task/epic sebelum audit | `UNVERIFIED` | Audit file/kode/test sebenarnya lalu tetapkan `NOT_STARTED`, `IN_PROGRESS`, `PARTIAL`, `BLOCKED`, `DONE` sesuai workflow. |
| Hasil tes aplikasi | `NOT_RUN` | Perintah yang dieksekusi, versi environment, hasil, log sanitasi, artefak oracle. |
| Rilis | Belum disetujui / belum diverifikasi | Semua gate DoD, security, financial, rollback, dan owner acceptance. |

Jangan menulis `PASS`, `DONE`, `APPROVED`, atau `DEPLOYED` hanya karena terdapat contoh script, screenshot, diagram, atau laporan AI bahwa perubahan “seharusnya berhasil”. Hasil review dokumen dan hasil pengujian ZIP tetap **bukan pengujian produk**.

---

## 9. SOP satu sesi AI coding agent

1. **Inspect:** identifikasi cwd, branch, file nyata, artefak dan perubahan belum di-commit; audit sebelum mengubah `UNVERIFIED`.
2. **Select:** ambil **satu** task dari [`planning/TASKS.md`](planning/TASKS.md) dengan ID asli dan dependensi jelas.
3. **Route:** baca `AGENTS` yang berlaku, requirement PRD, dokumen domain, test oracle, dan keputusan yang memblokir task itu.
4. **Trace:** tulis daftar `FR/EN/NFR`, `DR/CS/FTC` (jika relevan), UX/API/security, files touched, rollback/risk.
5. **Decision gate:** lanjut hanya bagian dengan scope yang disetujui; minta approval untuk pilihan yang menentukan perilaku production.
6. **Plan:** buat langkah kecil, acceptance eksplisit dan tests yang dapat dieksekusi. Hindari refactor seluruh repo untuk satu task.
7. **Implement:** pertahankan batas Go sebagai otoritas finansial dan security; Flutter tidak mengakses PostgreSQL secara langsung.
8. **Verify:** jalankan tes relevan (unit, widget, database integration, contract, concurrency/security jika perlu); dokumentasikan hasil `PASS/FAIL/NOT_RUN/BLOCKED` apa adanya.
9. **Review:** cek diff, docs, migrations, UX states, privacy/logging, kriteria DoD, dan risiko yang masih terbuka.
10. **Handoff:** tambahkan catatan ke [`planning/PROGRESS.md`](planning/PROGRESS.md); update status task **hanya** berdasarkan bukti, sertakan next step dan blocker.

**Format handoff minimal:** `Task ID | change scope | files changed | relevant FR/DR/SEC | decision state | commands actually executed | result/evidence | remaining risks | PROGRESS update | next action`.

---

## 10. Contoh jalur navigasi spesifik

### 10.1 Implementasi transfer actual (`FR-004`, S-03)

`AGENTS root` → `backend/AGENTS` → `PRD/FR-004` → `Financial Domain Rules/DR` → `Database Design` → `API Guidelines` → `Security & Privacy` → `Financial Test Cases` + `Testing Strategy` → `TASKS/TK-S03-*` → `DECISION-LOG/Q-03/Q-04` → `Definition of Done` → `PROGRESS`.

Periksa **dua sisi transfer dalam satu transaksi**, kepemilikan rekening, idempotency, race/retry, kesalahan nominal dan outcome timeout tidak pasti. Jangan membuat dua operasi income/expense semu.

### 10.2 Implementasi compound calculator (`FR-008`, S-05)

`AGENTS root` → `backend/AGENTS` (untuk engine) + `frontend/AGENTS` (untuk UI) → `PRD/FR-008` → `Financial Domain Rules` → `Calculation Specs` → `Financial Test Cases` → `Design System` + `Screen Specs` → `Q-04` dan keputusan lokal terkait → tests → `PROGRESS`.

Pisahkan contoh numerik, identitas matematika, dan **kebijakan return/timing/rounding**. Backend menghasilkan nilai otoritatif; Flutter menampilkan asumsi, unit dan penjelasan.

### 10.3 Implementasi Monte Carlo (`FR-018`, S-11)

`AGENTS root` + `backend/AGENTS` → `PRD/FR-018` → `Financial Domain Rules` + `Calculation Specs` + `Financial Test Cases` → `Backend Architecture` → `Testing Strategy` → `MASTER-ROADMAP/S-11` → `DECISION-LOG/Q-08` → **`MONTE-CARLO.md` bila sudah disusun/approved**.

Jika stochastic model, correlation, seed, quota atau disclosure masih TBD, jangan memberi probabilitas produksi seolah telah tervalidasi. Simulasi **tidak** melakukan posting ke actual ledger.

---

## 11. Dokumen yang belum dibuat: **11 file Markdown tersisa**

Daftar ini adalah **rencana file**, bukan tautan ke dokumen yang sudah ada. Penempatan path di bawah adalah **path yang diusulkan** dan boleh disesuaikan melalui keputusan pemilik; jangan membuat folder/placeholder kosong untuk mengklaim kelengkapan.

| Rekomendasi urut | Path rencana | Fungsi | Prioritas sebelum coding awal |
|---:|---|---|---|
| 1 | `README.md` | Pintu masuk repository, ruang lingkup dan quickstart berbasis kenyataan repo. | Tinggi; isi setup harus berbasis file nyata. |
| 2 | `docs/architecture/ADR/0001-TECH-STACK.md` | Merekam keputusan stack yang memang approved serta opsi yang belum final. | Tinggi; jangan mengunci library/provider tanpa approval. |
| 3 | `docs/development/LOCAL-SETUP.md` | Instalasi lokal, env template, migrasi, cara menjalankan Flutter/Go/test. | Tinggi; perlu audit repo/commands asli. |
| 4 | `docs/features/TRANSACTIONS.md` | Detail actual transactions, transfer/reversal, concurrency, idempotency. | Tinggi sebelum S-03. |
| 5 | `docs/features/FINANCIAL-GOALS.md` | Goal lifecycle, funding/progress semantics, feasibility and boundaries. | Sebelum S-05/S-06. |
| 6 | `docs/features/LIFE-PLANNING.md` | Life events, budget/recurring, dan assumption management R2. | Sebelum R2. |
| 7 | `docs/features/SIMULATIONS.md` | UX/output/consent/disclosure simulasi dan scenario comparison. | Sebelum R2/R3. |
| 8 | `docs/financial/SCENARIO-ENGINE.md` | Model baseline, snapshot/versioning, deterministic projection/replay. | Gate sebelum S-08. |
| 9 | `docs/financial/MONTE-CARLO.md` | Model probabilistik, seed, distribusi, korelasi, validasi statistik. | Gate sebelum S-11. |
| 10 | `docs/financial/GOAL-OPTIMIZER.md` | Objective function, constraints, infeasible result, algorithm tests. | Gate sebelum S-10. |
| 11 | `docs/operations/DEPLOYMENT.md` | Environment, secrets, migrations, monitoring, recovery, rollback, release. | Gate sebelum deployment aktual. |

**Sisa baseline:** 37 total target − 26 disusun = **11 file `.md`**. Rekomendasi prioritas di atas tidak mewajibkan penyelesaian seluruh advanced specs sebelum coding awal. Jika kebutuhan baru muncul atau dua dokumen digabung/disetujui untuk dihapus, **perbarui target dan hitungan** alih-alih memaksakan angka 37.

---

## 12. Cara menjaga INDEX dan dokumentasi tetap sinkron

Setiap ada dokumen baru/rename/approval atau kontrak yang berubah:

1. Periksa apakah path canonical benar-benar ada di monorepo; hindari alias yang bersaing, kapitalisasi berbeda, atau file duplikat.
2. Perbarui inventaris di §2 dan status/hitungan di §1 dan §11; jangan menganggap dokumen rencana sudah ada.
3. Perbarui router §3 bila sumber baru menjadi **pemilik** keputusan yang relevan; jangan menduplikasi spesifikasi detail dalam INDEX.
4. Periksa pengaruh ke ID `FR/DR/CS/FTC`, `UF/SCR`, `SEC/ST`, dan task `TK/EP`, terutama untuk perubahan domain finansial.
5. Rekonsiliasi dengan `DECISION-LOG` untuk perubahan status `PROPOSED/TBD → DECIDED` menggunakan approval nyata, bukan asumsi.
6. Perbarui `TASKS` hanya jika scope/dependensi/rencana berubah; perbarui `PROGRESS` hanya jika **ada observasi/evidence aktual**.
7. Periksa tautan relatif dan **case sensitivity** sesuai sistem berkas target. Tautan Markdown tidak membuktikan suatu file sudah di-commit.
8. Catat versi/tanggal, perubahan dan reviewer; jangan menimpa histori keputusan/test untuk menyembunyikan masalah.
9. Jika ada perubahan API/DB yang disetujui, cek kesesuaian **approved OpenAPI / applied migrations** dan regression tests saat artefak itu benar-benar tersedia.
10. Setelah verification, dokumentasikan apa yang **benar-benar** diperiksa; jangan menulis "test produk lulus" jika yang diuji hanya keberadaan file/ZIP.

### Pemeriksaan workspace (contoh, jangan klaim sudah dijalankan pada repo pengguna)

```sh
# Jalankan dari root monorepo yang sebenarnya, setelah repository tersedia.
pwd
git status --short
find docs -type f -name '*.md' | sort
# Cari status dan referensi, jika ripgrep sudah dipasang:
rg -n 'PENDING_OWNER_APPROVAL|UNVERIFIED|BLOCKED|TBD' docs
```

Jangan mengarang script `make`, `flutter test`, `go test`, Docker Compose, migration runner, atau command CI sebelum benar-benar memastikan project memiliki file konfigurasi/alat terkait. Setelah kode tersedia, command harus dibaca dari repository aktual dan dicatat bersama hasilnya.

---

## 13. Aturan penempatan file dan kompatibilitas artefak

- **Path canonical** menggunakan root monorepo: `AGENTS.md`, `frontend/AGENTS.md`, `backend/AGENTS.md`, lalu `docs/<domain>/<DOCUMENT>.md`.
- **ZIP produk edisi lama** pernah menggunakan awalan `esokira/docs/product/...`. Jika sudah berada di root `esokira/`, ekstrak **isi `docs/product/` saja**, bukan membuat folder `esokira/esokira/` atau menduplikasi `docs/product`.
- Setiap ZIP spesifik dokumen umumnya berisi path canonical relatif root. Pastikan `docs/INDEX.md` tidak diletakkan di `docs/docs/INDEX.md`.
- Nama unduhan seperti `FRONTEND-AGENTS.md`/`BACKEND-AGENTS.md` adalah nama artefak *standalone*. **Nama di repository** tetap `frontend/AGENTS.md` dan `backend/AGENTS.md`.
- Satu file Markdown yang sama pada dua lokasi belum tentu dua dokumen independen; hitung **berdasarkan path canonical unik**, bukan jumlah ZIP yang pernah dikirim.
- File rencana di §11 **tidak boleh** dianggap sudah ada sampai fisiknya diperiksa.

### Struktur direktori target (bagian yang telah direncanakan)

```text
esokira/
├── AGENTS.md
├── frontend/
│   └── AGENTS.md
├── backend/
│   └── AGENTS.md
└── docs/
    ├── INDEX.md
    ├── product/
    ├── financial/
    ├── architecture/
    ├── design/
    ├── security/
    ├── development/
    └── planning/
```

Struktur ini **deskriptif** untuk organisasi dokumentasi; keberadaan kode aplikasi, folder tambahan, `contracts/`, `infra/`, atau konfigurasi proyek harus dibuktikan melalui audit repository aktual.

---

## 14. Checklist quality assurance untuk INDEX

- [x] Semua **25 path dokumen yang sudah disiapkan** terdaftar satu kali menurut path kanonis; indeks menjadi file ke-26.
- [x] `AGENTS.md` root dan dua instruksi khusus folder dipisahkan jelas dari PRD dan arsitektur.
- [x] Router pembacaan diarahkan ke owner docs, bukan mengganti isi spesifikasi.
- [x] Status `REVIEWED_INTERNAL`, `NOT_RECORDED`, `UNVERIFIED`, `NOT_RUN` tidak disamakan.
- [x] `S-00`–`S-12`, R0–R3 dan R4 discovery dipisahkan; `FR-023/FR-024` tidak dipromosikan menjadi scope implementasi.
- [x] Jumlah backlog **58 task rinci + 13 epic** dan 11 dokumen terencana selaras dengan baseline perencanaan.
- [x] Tidak ada keputusan teknis baru yang diklaim telah mendapat approval.
- [x] Tidak ada klaim keberhasilan unit/integration/UI/security test terhadap aplikasi.
- [ ] **Validasi link di monorepo pengguna:** `PENDING_REPOSITORY_AUDIT` — saat ini baru dapat dicek terhadap daftar artefak Markdown yang disiapkan.
- [ ] **Owner review / approval:** `PENDING_OWNER_APPROVAL`.

---

## 15. Catatan perubahan

| Versi | Tanggal | Perubahan | Persetujuan |
|---|---|---|---|
| `0.1.0` | `2026-10-30` | Indeks 25 dokumen terdahulu, router pembacaan, traceability, sumber kebenaran, status dan 11 dokumen tersisa | `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` |

**Next recommended artifact:** `README.md` pada root monorepo. Buat sebagai dokumentasi repository yang jujur dan sesuai hasil inspeksi file nyata; bagian setup yang belum bisa diverifikasi harus diberi label rencana/`TBD`, bukan diisi dengan command fiktif.
