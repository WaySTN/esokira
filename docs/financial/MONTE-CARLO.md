# Esokira — Monte Carlo Risk Simulation Specification

> **Dokumen kanonis:** `docs/financial/MONTE-CARLO.md`  
> **Versi:** `0.1.0`  
> **Tanggal draft:** `2026-10-30`  
> **Status dokumen:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Status fitur:** `SPECIFICATION ONLY / UNVERIFIED / R3 PRODUCTION MODEL BLOCKED`  
> **Produk / stack dasar:** Esokira; Flutter/Dart, Go, PostgreSQL, monorepo  
> **Release & epic:** R3, `S-11`, `FR-018`, `CS-014`, `EP-S11-01`; integrasi bersyarat ke `FR-019`, `FR-020`, `EN-005`  
> **Pemilik persetujuan:** Product Owner dengan reviewer model/statistik/keuangan dan keamanan yang ditunjuk. Dokumen ini sendiri **bukan** persetujuan model.

> **Instruksi untuk AI coding agent:** Baca aturan root [`AGENTS.md`](../../AGENTS.md), backend [`AGENTS.md`](../../backend/AGENTS.md), sumber matematis dan keputusan terkait sebelum mengimplementasikan fitur ini. Jangan memilih distribusi, parameter return, korelasi, PRNG, jumlah lintasan, confidence interval, percentile, kebijakan penarikan, atau target performa produksi hanya karena ada contoh di dokumen ini. Keputusan yang mengubah hasil harus dicatat/di-approve di [`DECISION-LOG.md`](../planning/DECISION-LOG.md). Selama `Q-08`/`CALC-Q13` belum resolved, **publikasi angka probabilistik production adalah BLOCKED**. Penulisan dokumen atau demo sintetis yang berlabel jelas **bukan** implementasi yang sudah selesai.

---

## 0. Tujuan, batas, dan prinsip utama

Monte Carlo membantu pengguna memahami **rentang hasil keuangan yang mungkin menurut suatu model**, bukan memprediksi nilai masa depan secara pasti. Untuk Esokira, simulasi stokastik mengambil sebuah skenario teridentifikasi, menjalankan banyak lintasan dengan perubahan kondisi yang disampel, lalu mengagregasi hasil seperti distribusi nilai terminal, shortfall, dan frekuensi pencapaian target.

Prinsip yang tidak boleh ditawar:

1. **Tidak ada transaksi aktual.** Menjalankan simulasi, membatalkan job, membuka hasil, atau mengubah parameter hanya beroperasi pada input/hasil simulasi yang terpisah; tidak membuat posting ledger, aset, utang, atau funded goal.
2. **Probability conditioned on assumptions.** `model_success_rate` adalah frekuensi dalam lintasan model yang valid, bukan peluang objektif bahwa hasil dunia nyata akan terjadi dan bukan rekomendasi investasi yang pasti.
3. **Kebenaran sebelum visualisasi.** Hasil tanpa model disetujui, input kritis, lineage, denominator, horizon, atau definisi sukses yang eksplisit tidak boleh ditampilkan sebagai angka sah.
4. **Replay terdefinisi.** Klaim hasil yang dapat direproduksi harus mencantumkan versi snapshot, scenario, model, engine, RNG/seed, mapping random draws, dan kebijakan numerik.
5. **Prioritas konsistensi domain.** Kas, investasi, aset nonlikuid, liabilitas, income, expense, transfer, kontribusi goal terencana, dan actual-funded adalah besaran yang berbeda.
6. **Batas komputasi dan privasi.** Setiap job memiliki owner, quota, timeout, cancellation, retry safety, dan retention; sample path tidak boleh bocor ke log atau pengguna lain.
7. **Statistical review merupakan gate tersendiri.** Test agregasi `720/1000` tidak memvalidasi distribusi return atau kemampuan prediktif model.

### 0.1 Satu dokumen, satu sumber kebenaran untuk tiap lapisan

- **Produk/acceptance:** [`PRD.md`](../product/PRD.md) dan [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md).
- **Aturan uang & perilaku domain:** [`FINANCIAL-DOMAIN-RULES.md`](FINANCIAL-DOMAIN-RULES.md), `DR-001`–`DR-024`.
- **Kontrak kalkulasi & referensi numerik:** [`CALCULATION-SPECS.md`](CALCULATION-SPECS.md), khususnya `CS-014`; [`FINANCIAL-TEST-CASES.md`](FINANCIAL-TEST-CASES.md), `FTC-MC-01`–`FTC-MC-10`.
- **Perubahan keadaan deterministik:** [`SCENARIO-ENGINE.md`](SCENARIO-ENGINE.md), termasuk timing/event ordering yang belum disetujui.
- **Pengalaman skenario pengguna:** [`SIMULATIONS.md`](../features/SIMULATIONS.md) dan [`LIFE-PLANNING.md`](../features/LIFE-PLANNING.md).
- **Goal dan funded progress:** [`FINANCIAL-GOALS.md`](../features/FINANCIAL-GOALS.md); transaksi aktual: [`TRANSACTIONS.md`](../features/TRANSACTIONS.md).
- **Arsitektur storage, API, worker:** [`BACKEND-ARCHITECTURE.md`](../architecture/BACKEND-ARCHITECTURE.md), [`DATABASE-DESIGN.md`](../architecture/DATABASE-DESIGN.md), [`API-GUIDELINES.md`](../architecture/API-GUIDELINES.md), [`SYSTEM-ARCHITECTURE.md`](../architecture/SYSTEM-ARCHITECTURE.md).
- **UX dan security:** [`USER-FLOWS.md`](../design/USER-FLOWS.md), [`SCREEN-SPECS.md`](../design/SCREEN-SPECS.md), [`DESIGN-SYSTEM.md`](../design/DESIGN-SYSTEM.md), [`SECURITY-PRIVACY.md`](../security/SECURITY-PRIVACY.md).
- **Delivery/evidence:** [`TASKS.md`](../planning/TASKS.md), [`MASTER-ROADMAP.md`](../planning/MASTER-ROADMAP.md), [`PROGRESS.md`](../planning/PROGRESS.md), [`TESTING-STRATEGY.md`](../development/TESTING-STRATEGY.md), [`DEFINITION-OF-DONE.md`](../development/DEFINITION-OF-DONE.md), [`DEVELOPMENT-WORKFLOW.md`](../development/DEVELOPMENT-WORKFLOW.md).
- **Keputusan:** [`DECISION-LOG.md`](../planning/DECISION-LOG.md); ADR stack: [`0001-TECH-STACK.md`](../architecture/ADR/0001-TECH-STACK.md); navigasi: [`docs/INDEX.md`](../INDEX.md).

**Prioritas saat konflik:** keputusan pemilik yang disetujui dan dicatat → product/domain rules yang disetujui → calculation/golden test sesuai status → spesifikasi ini untuk subdomain stokastik → contoh teknis nonfinal. Jangan mengangkat proposal menjadi kontrak final. Perubahan hasil atau kegagalan menelusuri keputusan harus ditandai `BLOCKED` pada irisan tugas yang terkena.

### 0.2 Label normatif

| Status | Makna dalam dokumen ini | Agen boleh melakukan apa |
|---|---|---|
| **REQUIRED** | Batas domain, keselamatan, provenance, privacy yang harus dipertahankan | Menjadikan acceptance dan test negatif |
| **REFERENCE-EXACT** | Aljabar untuk input buatan yang memang telah memiliki oracle | Menguji implementasi agregasi independen dari sampler |
| **PROPOSED** | Pilihan model, desain data, arsitektur, atau algoritma yang masuk akal tetapi belum disetujui | Riset/prototipe sintetis terisolasi, dokumentasi keputusan |
| **TBD** | Input/model/policy belum ditentukan | Mengumpulkan pilihan, trade-off, reviewer, bukti |
| **BLOCKED** | Tidak boleh menjadi fitur production atau laporan lulus | Memisahkan kerja independen; eskalasi keputusan |
| **UNVERIFIED / NOT_RUN** | Kode atau test repo belum benar-benar diperiksa/dijalankan | Tidak membuat klaim implementasi selesai |

## 1. Scope dan dependensi rilis

| Cakupan | ID | Peran dan syarat | Status |
|---|---|---|---|
| R0 | `EN-004` | Pure financial engine + test harness, **bukan** Monte Carlo selesai | Foundation gate terpisah |
| R2 | `FR-012/013/014/015` | Deterministic projection, scenario versioning, forecast, stress tests dari `SCENARIO-ENGINE.md` | Prasyarat sesuai approval R2 |
| R3 / S-10 | `EN-005`, `EP-S10-02` | Bounded heavy/background job foundation | Worker queue/limit/cancel TBD |
| **R3 / S-11** | **`FR-018`, `CS-014`, `EP-S11-01`** | Seeded paths, agregasi, validasi statistik, run metadata, UI keterbatasan model | **BLOCKED `Q-08`** |
| R3 / S-11 | `FR-019`, `CS-015`, `EP-S11-02` | Retirement accumulation/decumulation, withdrawal sequence/depletion | Memerlukan approval model pensiun tersendiri |
| R3 / S-12 | `FR-020`, `CS-016`, `EP-S12-01` | Financial GPS boleh merujuk hasil model yang sah | Tidak boleh auto-post atau mengarang saran |
| R4 | `FR-023/024` | Shared goals/portfolio planning | **Discovery only**, bukan implementasi |

**Dependensi task:** `EP-S11-01` bergantung pada `EP-S08-02` (scenario versions) dan `EP-S10-02` (bounded worker foundation), sesuai [`TASKS.md`](../planning/TASKS.md). Jangan melompati ke R3 hanya karena kode R2 dapat memanggil `simulate()`.

### 1.1 In scope

- Menentukan makna input dan output stochastic simulation; fingerprint baseline dan scenario version.
- Menyusun _sampling model registry_ **sebagai proposal**, bukan memilih return distribution default.
- Memisahkan sample path generation, deterministic state transition, success evaluator, aggregate/percentile, dan disclosure.
- Menangani reproducibility di bawah eksekusi paralel, retry, cancellation, partial failure, perubahan model, dan stale baseline.
- Mendefinisikan metode validasi sampler, distribution diagnostics, aggregation oracles, performance/operability, ownership, dan UX.
- Merumuskan keputusan terbuka beserta bukti yang diperlukan untuk approval owner/statistical reviewer.

### 1.2 Out of scope

- Menetapkan return historis dunia nyata, volatilitas nominal/riil, atau korelasi aktual tanpa sumber data dan model review.
- Menyatakan normal/lognormal, bootstrap, regime, correlated GBM, atau distribusi tertentu sebagai **pilihan final**.
- Menjamin hasil investasi, keamanan dana pensiun, rasio penarikan universal (misalnya aturan 4%), atau probabilitas sukses dunia nyata.
- Membuat optimizer goal (spesifikasi berikutnya: `docs/financial/GOAL-OPTIMIZER.md`) atau memberi saran pembelian instrumen tertentu.
- Menetapkan OpenAPI route, nama kolom SQL, kode error HTTP, vendor worker, package Go, atau library statistik final tanpa review kontrak.
- Menghubungkan ke rekening bank, mengeksekusi investasi, membeli produk finansial, atau mengubah posting keuangan aktual.

## 2. Istilah dan dimensi ketidakpastian

| Istilah | Makna operasional dan guardrail |
|---|---|
| Baseline snapshot | Keadaan finansial pada as-of tertentu dengan revision/fingerprint, bukan query saldo live di dalam loop sampling |
| Scenario version | Kumpulan asumsi dan life events yang dibekukan; bukan draft yang dapat berubah saat job berjalan |
| Deterministic transition | Perhitungan saldo/utang/aliran per periode menggunakan `SCENARIO-ENGINE.md` dengan input lengkap |
| Stochastic factor | Besaran yang disampel dari model tervalidasi (misalnya return), dengan unit dan periode jelas |
| Random seed | Input generator pseudo-acak; bukan rahasia finansial, tetapi bagian provenance yang harus disimpan secara aman |
| RNG/PRNG algorithm | Definisi generator, versi, pembagian stream, dan transformasi draw; **belum dipilih** |
| Path / lintasan | Satu realisasi terurut faktor sepanjang horizon; tidak sama dengan satu nilai terminal acak |
| Sample size / `n_requested` | Banyak lintasan yang diminta dan telah melewati quota gate sebelum eksekusi |
| `n_valid` | Banyak lintasan dengan state dan outcome yang berhasil dinilai menurut model/policy |
| `n_failed` | Banyak lintasan yang gagal secara teknis/numerik/validasi; kategori harus tersimpan |
| Model success | Predikat eksplisit berdasarkan target, deadline, liquidity atau depletion policy yang disetujui |
| Model success rate | `n_success / n_valid` **hanya jika** kebijakan failed-path/denominator mengizinkan publikasi |
| Percentile | Kuantil *variabel yang disebut* (misalnya terminal liquid assets), waktu dan metode kuantil yang didokumentasikan |
| Confidence interval / Monte Carlo sampling error | Ketidakpastian akibat sampel lintasan terbatas, **bukan** cakupan kesalahan asumsi/model atau jaminan masa depan |
| Parameter uncertainty | Ketidakpastian estimasi mean/volatility/correlation dari data; berbeda dari path variability |
| Model risk | Risiko struktur model, perubahan rezim, tails, data bias, dan variabel yang tidak dimodelkan |
| Sequence-of-returns risk | Perbedaan hasil karena urutan return, khususnya bila ada penarikan dana, bukan hanya rata-ratanya |
| Stale result | Output yang baseline/model/parameter acuannya tidak lagi cocok dengan konteks yang sedang dilihat |
| `STOCHASTIC_RESULT` | Kelas data hasil simulasi; **bukan** `ACTUAL`, `PLANNED`, atau `FORECAST` deterministik |

## 3. Required input contract dan validation gates

**Skema di bawah konseptual**; bentuk DTO, JSON, enum, precision dan nama endpoint final harus mengikuti OpenAPI dan keputusan yang disetujui. Agen tidak boleh menyalin struktur ini sebagai migration/API production.

| Kelompok | Informasi wajib sebelum run | Validasi/penolakan |
|---|---|---|
| Identity | Authenticated `owner_id`, run access scope | Owner dari token server; tidak percaya owner ID yang diberikan klien |
| Baseline | Immutable reference, as-of date, revision/fingerprint, currency context | Tidak menerima snapshot milik user lain atau baseline yang hilang |
| Scenario | Immutable `scenario_version`, assumptions fingerprint | Draft yang berubah tidak boleh dibaca sambil sampling |
| Horizon | Start/as-of, period unit, end horizon, timezone/date basis | Periode tidak valid, urutan waktu salah, horizon di atas limit ditolak |
| Model | `model_id`, `model_version`, approved policy, parameter set | Hanya versi allowlisted yang disetujui dapat menghasilkan production result |
| Return | Factors, frequency/units, distributions/parameters, asset mapping | Parameter negatif/tidak finite/tidak konsisten ditolak sesuai model |
| Inflation | Mode nominal/real, data dasar, method of deflation | Jangan deflate dua kali atau mencampur basis unit |
| Dependence | Correlation/covariance structure and factor order bila dipakai | Dimensi salah, asimetri, non-PSD (bila diperlukan) ditolak, tidak silent-fix |
| Goal/retirement | Success predicate, target, due date, liquidity basis, withdrawal policy | Tidak menganggap planned contribution sebagai funded; policy TBD diblokir |
| RNG | Explicit seed policy, algorithm/version, draw mapping, path count | Missing/unsupported version ditolak; seed saja tidak cukup untuk replay |
| Resource | Quotas on paths/horizon/factors/bytes/concurrency/time | Too large = bounded rejection; jangan fallback diam-diam ke run lebih kecil |
| Results | Requested metric IDs, quantile definition, output real/nominal units | Metrik/quantile tidak disetujui = unavailable, bukan guessed value |

### 3.1 Input classification

Setiap field harus ditandai `USER_ENTERED`, `ACTUAL_REFERENCE`, `APPROVED_MODEL_PARAMETER`, `DERIVED`, atau `SYSTEM_METADATA`. Parameter yang hilang tidak boleh diam-diam menggunakan asumsi imbal hasil, inflasi, life expectancy, atau toleransi risiko. Model yang menggunakan dataset eksternal harus menyimpan `dataset_id`, tanggal cakupan, izin penggunaan, sumber, transformasi, dan versi. Menyimpan sumber boleh berupa referensi akses terbatas; jangan menyalin seluruh data sensitif ke respons publik.

### 3.2 Mandatory preflight

Urutan validasi **logis** sebelum menerima pekerjaan berat:

1. Authenticate, owner-scope setiap resource termasuk nested scenario/source.
2. Periksa release dan approval `FR-018`, `Q-08`, `CALC-Q13`, model registry, policies.
3. Bekukan immutable references/snapshot dan identifikasi revision; validasi as-of dan currency.
4. Validasi parameter, units, dependencies, finite values, matrix, horizon, limits.
5. Validasi seed/RNG/version mapping dan evaluator semantics.
6. Rekonsiliasi logical run identity/idempotency berdasarkan kebijakan approved.
7. Simpan metadata run dan enqueue intent secara konsisten **jika** worker architecture telah disetujui.
8. Kembalikan status `ACCEPTED/QUEUED` yang tidak disalahartikan sebagai `COMPLETED`.

Jika salah satu prasyarat model atau persetujuan gagal, hasilnya **`BLOCKED/REJECTED` dengan alasan aman**, bukan memanggil distribusi contoh demi mengisi chart.

## 4. Representasi financial state dan unit

Monte Carlo **tidak menggantikan** state model `SCENARIO-ENGINE.md`. Untuk setiap path, stokastik hanya memasok faktor yang telah disampel; aturan aliran uang, hutang, life events, amortization, dan klasifikasi tetap mengikuti kontrak deterministic engine yang disetujui.

**State concept (bukan Go struct final):**

```text
SimulationState_t:
  cash_by_account                 # jika relevan dan eligible
  invested_liquid_positions      # posisi terpapar return, tanpa double count
  other_assets_and_valuations    # rumah/nonlikuid tidak otomatis cash
  liabilities_and_debt_schedules
  actual_baseline_fingerprint    # read-only reference
  goals_and_plans                # allocations planned != actual-funded
  period_income_expense_flows
  shortfall_flags_and_amounts
  provenance: model/assumptions/time/unit
```

Syarat finansial:

- Total net worth tidak boleh menghitung account sebagai asset dua kali.
- Transfer internal bukan pemasukan/pengeluaran, kecuali fee/FX dinyatakan.
- Perubahan return investasi bukan arus kas dari luar, dan unrealized gains tidak otomatis menjadi kas untuk biaya hidup.
- Jika model rugi, hasil negatif atau hilangnya nilai harus diperlakukan menurut domain yang disetujui; **jangan clamp** saldo secara diam-diam.
- Jika saldo kas tidak cukup, catat `SHORTFALL/DEPLETION` beserta besarnya. Tidak ada utang otomatis atau pengambilan aset nonlikuid tanpa kebijakan eksplisit.
- Perubahan nilai rumah hipotetis tidak mem-post asset actual.
- Untuk nilai uang aktual, representasi harus exact; floating-point untuk model statistik internal (jika disetujui) **tidak** memberi izin untuk menyimpan saldo aktual sebagai binary float.
- Timing bunga, kontribusi, inflasi, biaya, FX, pajak dan rounding mengikuti keputusan `Q-04/Q-05` dan versi deterministic engine; jangan mengganti semantik antarpath.

## 5. Model stochastic: opsi yang harus dibandingkan, bukan default

Sebelum production, `Q-08`, `FDR-Q12`, `CALC-Q13`, `ARCH-Q07`, `BE-Q12`, `TS-Q10` membutuhkan dokumen keputusan dan persetujuan. Penjelasan di bawah **PROPOSED RESEARCH OPTIONS**, bukan fitur yang telah dipilih.

| Keluarga kandidat | Apa yang disampel | Keuntungan potensial | Batas/risiko yang harus diuji |
|---|---|---|---|
| Independent parametric returns | Return per factor/period dengan distribusi univariat | Mudah diuji sebagai baseline sederhana | Mengabaikan korelasi, tail risk, dan regime |
| Correlated parametric factors | Return multi-aset melalui matriks kovarians dan transformasi | Memodelkan dependence linear | Struktur covariance berubah; fat tails tidak otomatis tertangkap |
| Log-return model | Pertumbuhan melalui log-return untuk harga eksposur eligible | Menjaga harga positif pada asumsi tertentu | Salah guna untuk cashflows, liabilities dan aset nonpasar; tail/model risk |
| Historical/bootstrap | Resampling historical blocks/periods terdefinisi | Mempertahankan beberapa pola empiris | Bias periode data, horizon terbatas, data tidak representative |
| Regime/tail-aware | Kondisi ekonomi beralih, shock dan tail modeling | Sensitivity pada skenario buruk | Kalibrasi kompleks, parameter instability, risiko overfitting |
| Joint stochastic inflation/return | Korelasi inflasi dan return dalam factor model | Dapat menangkap risiko daya beli | Korelasi data dan model inflasi perlu bukti kuat |
| Deterministic stress overlays | Shock eksplisit tanpa sampling | Interpretasi jelas untuk R2 | **Bukan Monte Carlo** dan tidak boleh dihitung sebagai probabilitas |

Pemilihan model harus menjawab: domain sasaran (portfolio/goal/retirement), cakupan instrumen eligible, horizon, frekuensi, regional & mata uang, ketersediaan data, baseline risk-free assumptions, biaya, pajak, inflasi, korelasi, nilai negatif yang sah, dan keterbatasan yang jelas.

### 5.1 Distribusi return dan sampling constraints

- Harus jelas apakah input rate adalah **arithmetic return**, **log return**, **nominal rate**, **real rate**, annualized, atau per-period.
- Jika faktor dikonversi antarfrekuesi, rumus, timing dan treatment volatility harus di-review; jangan menggunakan akar waktu atau compound sederhana secara otomatis untuk semua model.
- Jika menggunakan harga investasi, harga tidak boleh jatuh di bawah nol pada model yang melarangnya; return dapat negatif, dan negative wealth/shortfall tidak boleh disembunyikan.
- Jika data menghasilkan rare tail events, samplers tidak boleh membuang outlier untuk memperhalus chart tanpa kebijakan.
- Keuntungan investasi tidak boleh dihitung pada account kas yang tidak ditetapkan sebagai investasi.
- Fixed/random contributions/withdrawals harus berinteraksi dengan event time secara eksplisit, tidak memakai pertumbuhan terminal sekali di akhir untuk menghitung kebutuhan liquidity sepanjang horizon.

### 5.2 Correlation/dependence validation

Bila model memakai matriks korelasi `C`, validasi setidaknya meliputi: jumlah dimensi sesuai urutan faktor, simetri `C_ij=C_ji`, diagonal unit, seluruh elemen finite dan di dalam `[-1,1]`, serta positive semidefinite sesuai model dan toleransi numerik yang disetujui. Jika tidak valid, **reject**; jangan memperbaiki dengan mendiam-diam mengganti matrix menjadi identity. Cholesky tidak selalu berlaku pada singular PSD; strategi decomposition dan toleransi harus tercatat sebagai keputusan tersendiri. Keputusan penggunaan covariance atau correlation, treatment missing data, dan interpolation tetap TBD.

### 5.3 Inflation dan nominal/real consistency

Jika output nominal `V_nominal,t` dan indeks harga `I_t` sama-sama valid dan berbasiskan satuan yang sama, hubungan konseptual daya beli adalah:

```text
V_real,t = V_nominal,t / (I_t / I_reference)
```

Rumus ini bukan izin memilih indeks inflasi tertentu. Kebijakan apakah inflasi deterministic, sampled, correlated, atau constrained masih `Q-05/Q-08`. Angka real tidak boleh diberi label nominal atau ditambah lagi inflasinya. Periode referensi harus terlihat di UI.

## 6. Seed, RNG, dan reproducibility

Seed saja **tidak cukup** untuk menjanjikan replay. Reproduksi tergantung seluruh transformasi draw, urutan pengambilan angka, versi generator, floating/decimal policy, path assignment, model parameters, dan deterministic engine version.

**Replay identity (conceptual):**

```text
canonical_run_identity = hash(
  owner_scope | baseline_fingerprint | scenario_version |
  model_id | model_version | engine_version |
  normalized_parameters | numerical_policy_version |
  rng_algorithm_id | rng_version | seed |
  path_index_mapping | factor_order | period_calendar |
  n_requested | output_metric_definitions
)
```

Hash di atas adalah *deskripsi atribut*, bukan canonical serialization atau algoritma hash final. Jangan memakai `hash()` bahasa pemrograman yang tidak stabil sebagai fingerprint lintas proses. Privacy: jangan membuat fingerprint public dari PII mentah atau menjadikannya alat enumerasi resource.

### 6.1 Parallel streams: deterministic mapping, not scheduling

**PROPOSED strategy:** setiap lintasan diberi identitas deterministik `(run_id, path_index)` yang mengarah ke stream/draws secara stabil; path 0, 1, ... tetap sama meski worker menyelesaikan lintasan dalam urutan berbeda. Faktor dan periode dipetakan secara eksplisit. Random global state yang dibagi thread/goroutine **dilarang bila** klaim replay identik bergantung pada urutan goroutine yang nondeterministik. Jangan memilih generator/library, counter-based approach, atau jump-ahead secara final sebelum `MC-Q04` diputuskan.

### 6.2 Reproducibility tiers

| Tier | Klaim yang diizinkan setelah terbukti | Harus diuji |
|---|---|---|
| **R0: aggregation reference** | Output agregasi diketahui dari daftar hasil lintasan yang diberikan | `FTC-MC-01`–`04` hanya jika kondisinya sesuai |
| **R1: same binary replay** | Input+seed+model/engine binary sama → canonical output sama | Lintasan, metrics, error class, parallel scheduling |
| **R2: same model/version cross-machine** | Hasil ekuivalen menurut toleransi approved | Architecture, IEEE/decimal/numeric policy, serialisasi |
| **R3: statistical reproducibility** | Karakteristik agregat berada pada acceptance bands terjustifikasi | Multiple seeds, sample sizes, distribution diagnostics |

**R0–R3 pada tabel di atas adalah tier verifikasi RNG lokal**, **bukan** fase produk Esokira R0–R3. Istilah tersebut tidak boleh menyebabkan kebingungan saat dicatat di Task/Progress; gunakan label `REPLAY-TIER-*` jika diimplementasikan.

### 6.3 Model versioning

- Semver tidak cukup tanpa definisi apa yang dianggap model-breaking. Perubahan distribusi, parameterisasi, basis inflasi, RNG transform, failure policy, precision, success predicate, dan percentile harus memicu peninjauan model version.
- Hasil historis **immutable**. Re-run di versi baru menghasilkan run baru dengan lineage dan tampilan perbedaan.
- Perubahan parameter dapat membuat perbandingan tidak apple-to-apple; UI harus menyebutkan perbedaan, bukan hanya menunjukkan dua garis.

## 7. Pure engine pipeline (logis; bukan nama fungsi final)

```text
[Authenticated & owner-scoped request]
  -> Validate approved model/release + resource quotas
  -> Resolve immutable baseline and scenario version
  -> Normalize typed parameters, units, timing and RNG plan
  -> Create run identity (or return existing logical run)
  -> Enqueue bounded work (approved orchestration)
      -> For path_index = 0..n_requested-1:
           obtain versioned reproducible random stream
           sample approved stochastic factors by period
           execute approved deterministic financial transitions
           evaluate per-period liquidity/depletion and goal predicate
           emit bounded path summary or typed path failure
      -> Validate counters and failure policy
      -> Aggregate model-conditioned statistics and distribution
      -> Attach full provenance, error/bias/disclosure labels
      -> Commit immutable result only if publishable under policy
  -> Owner-scoped retrieval / current baseline stale assessment
  -> Flutter display with unit, sample size, caveats and statuses
```

**STOP conditions:** cancellation, deadline exceeded, owner/account deletion, model revocation, unrecoverable numerical input violation, resource exhaustion, excess failed paths, or invalid output validation. Tidak ada status `COMPLETED` jika publishability gate gagal.

### 7.1 Path independence dan legitimate shared factors

Path berbeda tidak boleh secara tidak sengaja menggunakan return draws sama akibat reuse seed atau race. **Shared risk factors dalam satu lintasan** justru dapat dibutuhkan sesuai model korelasi yang disetujui. Faktor yang berbeda tidak otomatis independent; menyingkirkan korelasi hanya agar implementasi sederhana adalah perubahan model yang memerlukan approval.

### 7.2 Bounded path outcome

Untuk menghindari penyimpanan PII dan biaya besar, **PROPOSED**: simpan metadata dan agregat setelah statistik tervalidasi; simpan path detail hanya bila kebijakan debug, audit, user experience, retention, consent, dan kebutuhan percentile mengharuskannya. Exact quantiles membutuhkan kemampuan menyusun atau menghitung order statistics yang sesuai; approximation/sketch memerlukan algoritma dan error bound approved. Jangan berjanji exact percentile lalu memakai approximation terselubung.

## 8. Success evaluator: apa sebenarnya arti "berhasil"?

**Aturan REQUIRED:** setiap rate sukses harus memiliki `success_definition_id`, `success_definition_version`, target variable, horizon, threshold, liquidity policy, dan treatment shortfall yang eksplisit. Menampilkan angka sukses tanpa menyatakan apa yang diukur adalah salah secara semantik, sekalipun pecahannya benar.

| Kandidat predikat | Apa yang diperiksa | Risiko penafsiran |
|---|---|---|
| Goal amount by deadline | Dana eligible untuk goal memenuhi target pada due date | Planned deposits atau aset nonlikuid tidak otomatis funded |
| No cash shortfall | Kas tersedia menutup kewajiban tiap periode | Portfolio kaya tetapi kas habis tetap bisa gagal |
| Retirement non-depletion | Dana eligible memenuhi withdrawal policy selama horizon | Konvensi penarikan dan horizon mengubah hasil |
| Terminal liquid balance floor | Saldo likuid akhir di atas ambang tertentu | Bisa menyembunyikan shortfall di tengah jalan |
| Multiple-goal constraints | Semua hard goals terpenuhi sesuai prioritas/batas | Keberhasilan per-goal != all-goals success |

**Tidak ada predikat di atas yang otomatis menjadi default Esokira.** Produk dan reviewer model harus menentukan apakah keberhasilan berarti memenuhi seluruh hard constraints, apa status gap satu hari vs satu bulan, apakah partial goals dihitung, bagaimana pajak/fee, dan data apa yang dianggap eligible cash.

### 8.1 Formal contract (untuk input yang sudah tervalidasi)

Misalkan `Y_i` adalah outcome lengkap lintasan valid ke-`i`, dan `g(Y_i)` adalah predikat biner terdefinisi dan versioned. Maka:

```text
success_i          = 1 if g(Y_i) == true else 0
n_success          = sum(success_i over valid paths)
model_success_rate = n_success / n_valid     [only if n_valid > 0
                                            and failed-path publication policy approves]
```

`n_success <= n_valid <= n_requested`; untuk kebijakan satu path satu outcome, `n_valid + n_failed == n_requested` **hanya setelah semua lintasan requested telah diproses atau diklasifikasikan**. Saat run canceled/partial, jumlah processed dapat kurang dari requested; jangan memaksa counter menjadi lengkap atau mempublikasikan aggregate sebagai sukses.

### 8.2 Failed path ≠ failed financial plan

Bedakan:

- **Valid path, finansial gagal:** model memprediksi shortfall/depletion sesuai aturan. Ini **valid**, dihitung sebagai `success=false`.
- **Invalid numerical path:** overflow, NaN, sampler gagal, state tidak dapat dinilai, atau parameter error. Ini **technical failure**, bukan otomatis rencana gagal.
- **Run failure:** banyak path invalid, bug generator, timeout, atau policy invalid membuat hasil keseluruhan tidak layak ditampilkan.
- **Canceled:** pekerjaan dihentikan sebelum cukup bukti; jangan menganggap sisanya gagal atau 0%.

Persentase valid paths yang boleh ditoleransi, treatment invalid, dan syarat publikasi masih `MC-Q08`/`Q-08`. Jangan memilih denominator sendiri.

## 9. Aggregation contract, quantile, dan interpretasi statistik

### 9.1 Metrics yang berpotensi dihasilkan

| Metric conceptual | Definisi yang harus disertakan | Catatan approval |
|---|---|---|
| `n_requested / n_processed / n_valid / n_failed / n_success` | Counter run dan denominators, disertai failure category | Counting REQUIRED; publishability policy TBD |
| `model_success_rate` | Fraksi path sukses **di antara path yang policy izinkan** | Model conditioned, bukan guarantee |
| `terminal_value_p10/p50/p90` | Kuantil variabel terminal, asset eligibility, nominal/real, currency, date | Quantile algorithm & meaning **TBD** |
| `trajectory_quantile_by_period` | Kuantil cross-sectional pada setiap periode | Bukan satu sample path; jangan dijadikan lintasan tunggal yang mungkin |
| `first_depletion_time_distribution` | Distribusi pertama kali cash/portfolio tidak mampu memenuhi kewajiban | Depletion semantics/policy TBD |
| `any_shortfall_rate` | Fraksi valid paths dengan shortfall pada sekurang-kurangnya satu periode | Tidak sama dengan terminal loss rate |
| `expected_shortfall_amount` | Kondisional/perpath vs unconditional expectation harus jelas | Definisi loss, zero, dan tail TBD |
| `downside_quantiles` | Outcome tail pada quantile yang didefinisikan | Risiko sample tail dan uncertainty |
| `path_level_warnings` | Missing data, unsupported features, sensitivity concerns | Tidak boleh disembunyikan |
| `disclosure` | Model ID, model limitations, as-of, sample size, horizon | Wajib untuk UI publishable |

### 9.2 Kuantil tidak boleh dipilih hanya untuk terlihat rapi

**PROPOSED evaluation alternatives**: empirical nearest-rank, linear interpolation pada order statistics, exact weighted quantiles, atau approximate streaming quantile sketch. Berbeda algoritma bisa memberi hasil numerik berbeda untuk data kecil. Sebelum `MC-Q09` disetujui:

- Tidak membuat expected numerical value P10/P50/P90 sebagai "golden produksi".
- Tidak mencampur P10 nominal/net worth dengan P50 real/liquid.
- Tidak menyebut "P90 peluang 90% mendapat sekian" tanpa menjelaskan orientasi variabel, horizon, dan model.
- Tidak menyajikan kurva kuantil cross-sectional seolah-olah satu perjalanan finansial individu yang kontinu; anggota path yang membentuk P10 dapat berbeda tiap periode.
- Saat sampel kecil dan tail tidak stabil, tampilkan status insufficient/unstable sesuai policy, bukan angka sangat presisi.

### 9.3 Monte Carlo sampling error vs model risk

Untuk `n` *independent and identically distributed* valid Bernoulli successes, **ilustrasi matematis** standard error dari proporsi adalah:

```text
p_hat  = k / n
SE_hat = sqrt(p_hat * (1 - p_hat) / n)
```

Jika sampling berkorelasi, ada path weights, resampling khusus, adaptivity, atau failed-path selection, rumus tersebut **tidak otomatis valid**. Bahkan interval statistik yang sempurna untuk proporsi model **tidak** mengukur ketidakpastian parameter pasar, distribusi salah, data historis bias, atau perubahan ekonomi. Pilihan metode confidence interval (misalnya Wilson vs lainnya), level, stopping rule, dan user-facing wording adalah `MC-Q10` **TBD**. Jangan menyebut "95% yakin rencana sukses" karena sebuah confidence interval Monte Carlo.

### 9.4 Precision dan jumlah angka UI

Jumlah digit harus mengikuti resolusi hasil dan error model. Menampilkan `72.000000%` dari 1.000 path mengesankan kepastian palsu. Pembulatan hasil statistik, presisi nominal finansial, dan indikator sampel/konvergensi adalah keputusan UI+model terpisah. Nilai yang dikonversi ke satuan uang tampilan harus diberi currency dan basis waktu yang jelas.

## 10. Oracle numerik yang sudah tersedia — jangan diubah

Sumber utama: [`FINANCIAL-TEST-CASES.md`](FINANCIAL-TEST-CASES.md) §16 dan [`CALCULATION-SPECS.md`](CALCULATION-SPECS.md) §16, `CALC-REF-14`.

### 10.1 Reference case A: agregasi sukses lengkap (`FTC-MC-01`, exact)

```text
n_requested = 1000
n_valid     = 1000
n_failed    = 0
n_success   = 720
n_failure_financial = 280  # valid path, predicate is false

model_success_rate = 720 / 1000 = 0.72 = 72%
```

Makna hasil: **72% dari 1.000 lintasan valid yang dibuat model tertentu memenuhi predikat sukses tertentu**. Bukan ramalan 72% kemungkinan finansial nyata terjadi, bukan estimasi return, dan bukan jaminan. Angka ini menguji **agregasi**, **bukan** model distribusi atau proyeksi kas.

### 10.2 Reference case B: teknik error pada lintasan (`FTC-MC-04`, conditional)

```text
n_requested = 1000
n_valid     = 800
n_failed    = 200  # technical/model failures; policy not decided
n_success   = 720
n_financial_failure_valid = 80

720 / 800 = 0.90  # arithmetic conditional on 800 valid paths
720 / 1000 = 0.72 # would be different denominator and meaning
```

**Jangan publikasikan 90% maupun 72%** sebagai hasil run production sampai kebijakan failed path, bias/penghilangan path, dan minimum completion rate disetujui. Test ini membuktikan denominator sensitivity, bukan validitas produk.

### 10.3 Reference case C: nol valid (`FTC-MC-03`)

Bila `n_valid=0`, tidak boleh membagi nol atau menampilkan 0% berhasil. Sistem mengembalikan status insufficient/engine failure yang disetujui beserta alasan aman dan tanpa raw stack trace atau angka statistik palsu.

### 10.4 Reference case D: run canceled (`FTC-MC-09`)

Jika sebagian lintasan belum selesai saat cancel, catat processed/valid/failed, tetapi run tidak `COMPLETED`. Jangan memakai agregat parsial sebagai success figure kecuali ada spesifikasi eksplisit terpisah (yang saat ini **tidak** disetujui).

### 10.5 Reference case E: input/versi RNG berubah (`FTC-MC-05/06`)

- Input sama dan seed sama **tetapi** model/RNG version berbeda → **tidak** menjanjikan path-identical.
- Replay yang mengklaim identik membutuhkan input, snapshot, model/engine/RNG versions, path mapping, seed, numeric settings yang benar-benar sama; test eksekusi masih `NOT_RUN` dan policy terbuka.

## 11. Sequential returns, cashflow, dan retirement integration

Peningkatan nilai portofolio tidak boleh diperlakukan sebagai satu CAGR terminal yang kemudian dipakai seolah-olah tidak ada kebutuhan kas di antaranya. Untuk goal dan retirement, *urutan* return memengaruhi kemampuan memenuhi biaya dan kewajiban. Bahkan dua deret dengan produk pertumbuhan sama dapat menghasilkan hasil terminal yang berbeda setelah ada contributions/withdrawals sepanjang periode.

**Ilustrasi matematis saja (bukan withdrawal policy Esokira):**

```text
Start invested liquid funds: 100 (unit abstrak)
Withdraw 20 after each period; no fees, taxes, other income.

Path A returns: +50%, then -50%
  Period 1 before withdrawal: 100 * 1.5 = 150
  Period 1 after withdrawal:  150 - 20 = 130
  Period 2 before withdrawal: 130 * 0.5 = 65
  Period 2 after withdrawal:   65 - 20 = 45

Path B returns: -50%, then +50%
  Period 1 before withdrawal: 100 * 0.5 = 50
  Period 1 after withdrawal:   50 - 20 = 30
  Period 2 before withdrawal: 30 * 1.5 = 45
  Period 2 after withdrawal:   45 - 20 = 25
```

Walau multiset return kedua path sama, hasil setelah penarikan berbeda (`45` vs `25`). Contoh ini menyatakan **urutan dan timing penting**, tidak menetapkan bahwa penarikan akhir periode, angka return ini, atau usia/horizon tertentu sudah menjadi kebijakan produk. Pada kasus dengan saldo tak cukup, engine harus memberi shortfall/depletion, bukan saldo negatif tanpa label atau utang otomatis.

### 11.1 Retirement/FIRE bukan sekadar success metric Monte Carlo

`FR-019/CS-015` membutuhkan model terpisah mengenai tahap accumulation/decumulation, spending trajectory, pension/other income, withdrawal ordering, liquid vs illiquid assets, pajak/fee, inflasi, horizon, dan failure predicate. Monte Carlo menyediakan path uncertainty **setelah** model pensiun tersebut disetujui. Jangan menampilkan label "aman pensiun" hanya dari percentile saldo akhir atau memakai nilai properti sebagai cash yang dapat ditarik.

### 11.2 Goal optimizer bukan sampler

`FR-017/CS-013` menentukan alokasi berdasarkan objective, constraints, dan feasibility. Menjalankan random simulations tidak membuktikan suatu alokasi optimal. Integrasi risk-aware optimization kelak memerlukan keputusan bersama Q-07/Q-08, kontrak risiko, dan evaluasi out-of-sample; tanpa itu kedua engine diperlakukan **berbeda** dan tidak saling mengesahkan.

## 12. Performance, quota, dan anti-resource-exhaustion

Keputusan jumlah lintasan, tingkat concurrency, horizon, timeout, memory cap dan latency target masih `MC-Q11`, `EN-005`, `NFR-004`, bukan angka standar yang boleh dikarang.

**Required guardrails sebelum worker menerima run:**

1. Maksimum horizon/periode dan factor dimensions disetujui per model.
2. Maksimum `n_requested`, request size, output size, concurrent jobs/user, dan global quotas disetujui.
3. Per-run memory/CPU/time budget ditegakkan oleh orchestrator atau in-process safeguards yang terukur.
4. Job intake dan queue melindungi single user agar tidak dapat memonopoli resource pengguna lain.
5. Cancellation signal memeriksa checkpoint berkala, termasuk loop per path/periode, tanpa mengubah hasil path yang sudah valid.
6. Run timeout bukan izin untuk infinite retries; failed attempts harus bounded dan diaudit.
7. Jika pressure tinggi, terapkan backpressure/rejection yang eksplisit; tidak menurunkan jumlah path secara diam-diam sambil mempertahankan label yang sama.
8. Tidak ada memory growth linear tak terbatas per path; retained details dan sort strategy punya batas teruji.

### 12.1 Complexity & output strategy (proposed, not targets)

Jika `N` path, `T` periode, `F` faktor, kompleksitas perhitungan dasar paling sedikit bergantung pada `N*T` state transitions dan faktor model. Satu rumus Big-O generik tidak cukup sebagai SLA. Instrumentasi minimal: queue wait, path throughput, execution CPU time, peak memory, valid/failed path counts, retries, cancellation latency, database write volume, dan response payload size. Semua metrik ini harus aman dari PII.

### 12.2 Statistical convergence bukan alasan untuk silent early stop

Adaptive stopping/early-exit dapat menghemat resource tetapi mengubah sample-size distribution, bias, confidence, dan replay semantics. Sebelum diperbolehkan perlu aturan berhenti, minimum/max N, alpha/precision policy, independent validation, dan UI disclosure. Sampai disetujui, jangan mengadopsi stopping criterion otomatis atau menyebutkan sampling convergence yang tidak diuji.

## 13. Background jobs, reliability, dan state machine

Asynchronous processing (`EN-005`) merupakan **planned R3 dependency**. Vendor seperti River/PostgreSQL-backed queue tetap kandidat, bukan pilihan final. Engine pure Go tidak memanggil DB sendiri; application service menyelesaikan auth, snapshots, persistence, scheduling dan result publication. Worker membaca hanya scoped model input yang dibekukan.

**Contoh state machine PROPOSED (belum menjadi enum API atau SQL):**

```text
SUBMITTED -> VALIDATING -> ACCEPTED -> QUEUED -> RUNNING
                  |          |          |        |
                  v          v          v        +--> CANCEL_REQUESTED -> CANCELLED
               REJECTED    BLOCKED    CANCELLED |
                                             +--> FAILED
                                             +--> TIMED_OUT
                                             +--> VALIDATING_RESULT
                                                     |
                                                     +--> FAILED / BLOCKED
                                                     +--> COMPLETED

COMPLETED can later be displayed as STALE (view-level assessment),
without mutating the historical result record.
```

**Invariants lifecycle:**

- `ACCEPTED/QUEUED/RUNNING` bukan hasil simulasi selesai.
- Status terminal (`COMPLETED`, `FAILED`, `CANCELLED`, `TIMED_OUT`) tidak boleh ditimpa oleh retry yang terlambat tanpa kebijakan version/attempt.
- Worker dapat menerima pesan berulang (at-least-once); logical run identity, attempt ID, ownership, CAS/lease, dan idempotent result publication diperlukan.
- Failure setelah perhitungan sebelum commit hasil tidak boleh membuat dua result final dengan versi/identity sama.
- Cancellation saat running harus punya terminal status jujur; partial aggregate disimpan hanya bila audit/retention mengizinkan dan tidak disajikan sebagai sukses.
- Clock time bukan sumber random seed tanpa dokumentasi dan policy. `time.Now()` tersembunyi di engine melanggar replay contract.
- Mutasi aktual oleh worker adalah pelanggaran berat (`DR-001/002`), termasuk implicit goal funding dan recurring postings.

### 13.1 Retry, timeout, dan logical identity

Setelah network timeout pada request pembuatan run, klien tidak boleh membuat logical run baru dengan identity berbeda tanpa status reconciliation. API idempotency semantics dan run dedup key mengikuti keputusan `API-Q*/DB-Q*`, bukan dijadikan route fiktif. Retry worker harus dapat memproses ulang pekerjaan dengan seed/path mapping sama **jika** mode replay identik telah disetujui; jika terjadi perubahan model/version, itu **run baru**, bukan replay silent.

### 13.2 Crash consistency

- Transactional enqueue intent/outbox atau pola konsistensi lain **harus dipilih** jika enqueue + DB metadata berisiko partial commit.
- Worker tidak boleh mengekspos hasil separuh commit.
- Result validation gate memeriksa counts, approved metrics, finite numeric values, provenance completeness, dan owner scope sebelum atomic publication.
- Job purge/expiry harus mengikuti data retention dan deletion policy; response harus aman bila hasil sudah kedaluwarsa.
- Benchmark crash after enqueue, after compute, before commit, after commit + before ACK, dan duplicate delivery.

## 14. Storage, provenance, dan retention (conceptual)

**Contoh konseptual** field metadata run; **bukan** schema PostgreSQL, final API contract, atau izin membuat migrasi:

```yaml
simulation_run:
  run_id: opaque-id
  owner_scope_ref: server-derived
  source_baseline_ref: immutable-snapshot-id
  baseline_fingerprint: canonical-fingerprint
  scenario_version_ref: immutable-version-id
  scenario_fingerprint: canonical-fingerprint
  model_id: approved-model-id
  model_version: approved-model-version
  engine_version: exact-engine-build-or-semantics-version
  rng_algorithm_id: approved-rng-id
  rng_version: approved-rng-version
  seed_reference: recorded-according-to-policy
  path_mapping_version: approved-mapping-version
  factor_order_version: approved-factor-map-version
  numerical_policy_version: approved-numeric-policy
  success_definition_id: approved-evaluator-id
  success_definition_version: approved-evaluator-version
  quantile_method_version: approved-quantile-policy
  sample_counts: {requested: 0, processed: 0, valid: 0, failed: 0, success: 0}
  time_basis: explicit-period-calendar
  currency_and_real_basis: explicit
  snapshot_as_of: explicit
  data_source_versions: refs-if-relevant
  state: approved-lifecycle-state
  warnings_and_limitations: bounded-codes
  result_artifact_ref: owner-scoped-if-published
  created_at_recorded_at_finished_at: server-timestamps
```

**Note:** placeholder `0` dalam metadata contoh adalah bentuk nilai, **bukan** hasil run sungguhan. Secret, token, full account history, raw personal finance data, dan RNG implementation state yang tidak dibutuhkan **tidak** dicantumkan dalam logs/metric tags. Keputusan apakah seed dipublikasi di UI, disimpan encrypted, dibatasi eksport, atau disalin dalam reproducibility evidence masih memerlukan privacy review.

### 14.1 Immutability, staleness, and deletion

- Run yang selesai mereferensikan snapshot/versions as-of saat diterima; revisi account/scenario setelahnya tidak boleh menulis ulang hasil historis.
- Saat baseline versi lama, UI menunjukkan `stale`/rebase availability tanpa melakukan rebase otomatis atau menghapus jejak versi lama.
- Owner berhak atas data miliknya sesuai fitur export/deletion dan approved retention policy; workers/cache/artifact storage harus termasuk dalam penghapusan.
- After deletion, foreign/previous session tidak dapat melihat metadata, existence, raw artifacts, metrics, atau path summary.
- Backups dan legal retention exceptions jika ada harus mengikuti `SECURITY-PRIVACY.md` dan keputusan data lifecycle, bukan asumsi agent.

## 15. API/application boundary: contract-first, not path-first

Tidak ada endpoint produksi yang diotorisasi melalui dokumen ini. Perubahan API harus dilacak ke `contracts/openapi.yaml` **jika benar-benar sudah ada dan disetujui**, serta `API-GUIDELINES.md`. Fungsionalitas yang kelak diperlukan biasanya berupa:

| Logical operation | Request required | Response semantic | Safety |
|---|---|---|---|
| Describe approved model | Model capabilities, version and supported inputs | Model policy and limitations | Tidak mengekspos private model data atau secret |
| Validate preflight | Source refs + model choices | Invalid/blocked/ready with safe warnings | Tidak memulai kerja berat bila belum ready |
| Submit logical run | Scenario version + model params + desired metrics + run identity | Accepted/queued/status ref only | Owner-scoped, bounded, idempotent |
| Read run state | Opaque run ref | Queued/running/terminal status, safe progress | Tidak mengungkap keberadaan data user lain |
| Cancel run | Opaque run ref + authorized intent | Cancel requested/terminal semantics | Tidak mengubah actual; race-safe |
| Read result | Owner-scoped completed run | Aggregate, assumptions, provenance, warnings | No partial or stale disguised as current |
| Compare/rebase | Two eligible run references / explicit source revision | Compatibility decision; new run if necessary | Tidak memanipulasi historical output |
| Delete/export | User data rights scope | Report as per approved data lifecycle | Safe artifact/cache deletion |

**Disallowed:** mobile app memutuskan hasil sukses, menerima `user_id` klien sebagai otorisasi, mem-post transaksi saat get result, menganggap cancel segera menjamin no compute, atau menyatakan `HTTP 202` berarti simulasi lulus.

### 15.1 Error taxonomy (proposed codes)

Konsep error untuk test/UX tanpa mengunci nama wire/API:

- `MODEL_NOT_APPROVED`, `MISSING_CRITICAL_ASSUMPTION`, `INVALID_HORIZON`, `UNSUPPORTED_FACTOR`, `INVALID_CORRELATION_MATRIX`.
- `INSUFFICIENT_BASELINE`, `STALE_SOURCE_VERSION`, `UNAUTHORIZED_RESOURCE`, `RESOURCE_LIMIT_EXCEEDED`.
- `RNG_VERSION_MISMATCH`, `NUMERICAL_FAILURE`, `TOO_MANY_FAILED_PATHS`, `RESULT_NOT_PUBLISHABLE`.
- `RUN_CANCELLED`, `RUN_TIMEOUT`, `WORKER_UNAVAILABLE`, `RESULT_EXPIRED`, `MODEL_VERSION_RETIRED`.

Wire codes, status HTTP, retryability, user text, telemetry signals, dan boundary error versus warning **TBD** dalam kontrak API.

## 16. Flutter UX contract: risk communication without false certainty

R3 conceptual screens: `SCR-19` (setup/queue/result distribusi) via `UF-27`; `SCR-20` retirement via `UF-28`; downstream `SCR-21` Financial GPS via `UF-29`. Error/fallback sesuai `SCR-22`. UI memanggil backend dan **tidak menjalankan model stochastic independen** yang dapat berbeda dari Go.

### 16.1 Required states

| State | Yang harus terlihat | Yang tidak boleh dilakukan |
|---|---|---|
| Not configured | Input yang diperlukan, capability, risiko dan keterbatasan | Default return tersembunyi atau tombol run aktif tanpa model approved |
| Validating/blocked | Prasyarat yang hilang atau keputusan belum approved | Placeholder statistik agar chart terlihat penuh |
| Queued/running | Progress wajar jika didukung worker, pembatalan jika tersedia, asumsi yang dibekukan | Menyebut sukses sebelum result dipublikasi |
| Completed/validated | Model version, as-of, `n_valid`, horizon, unit, outcome variable, success condition, distribusi dan caveat | Menyajikan frequency model sebagai kepastian |
| Partial failure/not publishable | Status gagal dan alasan aman, tindakan yang dapat diambil | Menggunakan 800 dari 1.000 path diam-diam |
| Canceled/timed out | Status terminal jujur, memungkinkan retry/reconciliation sesuai policy | Memunculkan residual partial chart sebagai result final |
| Stale/replaced | Versi baseline/model lama, tindakan membuat run baru | Mengganti model historis atau menandai old result sebagai current |
| Offline/expired/unauthorized | Fallback netral dan akses terbatas | Membocorkan metadata/saldo dari cache lama |

### 16.2 Wording yang aman (contoh, bukan copy final)

- **Dapat digunakan setelah validasi:** "Dalam model ini, 720 dari 1.000 lintasan simulasi memenuhi target yang dipilih. Hasil bergantung pada asumsi, bukan jaminan hasil nyata." 
- **P10/P50/P90 jika approved:** "Rentang hasil nilai *[variabel yang spesifik]* per *[tanggal/horizon]*, dengan satuan *[nominal atau riil]*; hasil dihitung dari sampel model *[versi]*." 
- **Blocked:** "Simulasi risiko belum tersedia karena asumsi model perlu ditetapkan atau divalidasi." 
- **Tidak boleh:** "Anda punya 72% peluang pasti sukses investasi", "100% aman pensiun", "Esokira menjamin return", atau probabilitas tanpa model/denominator.

### 16.3 Chart dan accessibility

- Distribusi wajib ada label variabel, mata uang, as-of/terminal date, nominal/real, legend, dan sample size.
- Band kuantil harus dibedakan dari confidence interval atas parameter/proporsi; tooltip menjelaskan definisinya.
- Charts menyediakan alternatif data tabular/teks yang dapat diakses dengan screen reader dan text scaling.
- Outcome tail buruk dan shortfall tidak boleh disembunyikan hanya untuk membuat dashboard tampak positif.
- Rounding/palette/visual encodings mengikuti Design System; tidak memakai warna hijau sebagai satu-satunya indikator.
- Empty/error/blocked views tidak menampilkan mock angka production.

## 17. Security, privacy dan threat boundaries

Stochastic simulation memproses informasi sensitif: pendapatan, pengeluaran, aset, utang, scenario usia/timeline, dan model input pribadi. Seed bukan password, tetapi job metadata dan output tetap dapat mengungkap kondisi pribadi.

**Required controls:**

1. Authentikasi server-side untuk seluruh run CRUD, status, cancellation, result, export, dan nested baseline/scenario.
2. Resource ownership checks per item, termasuk job yang dipicu worker; `run_id` opaque bukan mekanisme authorization.
3. Jangan log raw baseline, seluruh lintasan, nomor rekening, token, user financial profile, model result sensitif dalam trace atau exception.
4. Batasi hasil yang disimpan dan retensi sesuai user consent/policy; tidak membuat artefak distribusi publik secara default.
5. Hindari IDOR melalui job status, error timing, object-storage link, model cache, dan polling.
6. Pastikan canceled/deleted user job tidak menjadi konten yang dapat diretrieval dari worker retry lama.
7. Secret management, encryption, backup/restore, incident response, deletion tombstones mengikuti `SECURITY-PRIVACY.md`.
8. Rate limits dan quotas mencegah resource exhaustion, cost abuse, dan kerugian ketersediaan pengguna lain.
9. Telemetry cardinality dibatasi, tidak menggunakan seed/user ID/plaintext input sebagai label metrik publik.
10. Sampling run baru setelah baseline berubah harus melalui owner action/approval yang jelas; worker tidak membuat keputusan finansial otomatis.

**Privacy review diperlukan** bila menggunakan historical market datasets dari pihak ketiga, provider analytics, cloud worker, eksternal AI, atau menampilkan/share raw results. Jangan menambahkan integrasi eksternal tanpa approval.

## 18. Observability, operations, dan incident response

Observability harus dapat menjawab "apakah perhitungan berhasil, versi mana, dan mengapa gagal" tanpa menyalin data finansial ke log.

| Signal | Metrik/log aman | Pertanyaan yang dijawab |
|---|---|---|
| Intake | Approved model label, result status, reject reason code | Mengapa run ditolak sebelum enqueue? |
| Worker | Queue wait, retries/attempts, lease lag, cancel latency | Apakah worker sehat dan crash/replay aman? |
| Compute | Duration, path throughput, peak memory bucket, failure categories | Apakah batas resource ditaati? |
| Statistical | `n_valid`, `n_failed`, validation gate result | Apakah hasil layak dipublikasikan? |
| Replay | Version mismatch classification, deterministic replay check | Apakah reproducibility rusak setelah deploy? |
| Privacy | Unauthorized access rejection counters, deletion completion | Apakah isolasi dan data lifecycle berjalan? |
| User experience | Completion/cancel/error ratios, stale display signal | Apakah pengalaman async jujur dan stabil? |

**Model incident examples:** generator/version berubah tanpa bump; return distribution ternyata salah unit; failed paths dikeluarkan diam-diam; success predicate bergeser; asset double counting; unauthorized cross-user results; quota bypass menyebabkan outage; historical result berubah diam-diam. Tindakan minimum: hentikan publikasi model terdampak, catat impacted versions/run IDs (private), beri status tidak valid pada hasil yang relevan, root-cause, perbaiki model + test oracle, lakukan migration/reprocess **hanya dengan approval** dan komunikasikan disclosure yang benar.

### 18.1 Data export/deletion and retention

Hasil advanced adalah bagian dari data keuangan pribadi; hak pengguna mengikuti `FR-021` dan security specification. Harus ada kebijakan apakah raw paths, aggregates, seeds, parameters, snapshots, worker logs/metadata, queued job, dan backup artifacts dipertahankan. Jangan memilih batas hari/ukuran arbitrer. Jika status legal retention belum diputuskan, buat decision record, bukan mengklaim deletion complete.

## 19. Validation protocol dan independent review

Model probabilistik memerlukan **lapisan verifikasi terpisah**, bukan hanya HTTP 200 atau chart:

1. **Pure aggregation oracle:** known synthetic arrays untuk counts, success rate, quantiles setelah metode approved, failure semantics.
2. **Random-generator validation:** known test vectors, seed replay, stream independence sesuai generator approved, parallel scheduling invariance.
3. **Distribution diagnostics:** sampling means/variances/tails/dependence sesuai model yang dipilih, dengan acceptance bands statistik terjustifikasi.
4. **Financial transitions:** per-path cashflow, debt, transfers, dates, returns dan shortfall mengikuti deterministic engine yang telah diuji.
5. **Scenario invariants:** snapshot/version integrity, actual isolation, no phantom funded progress, no double counting.
6. **Robustness and edge cases:** zero volatility bila model mendukung, extreme losses, long horizon, invalid covariance, negative cash, no valid paths, overflow, cancellation, transient failures.
7. **Independent numerical evaluation:** reviewer menghitung ulang sample fixtures dan menguji sensitivity terhadap asumsi return/inflation/correlation/withdrawal.
8. **Statistical model validation:** band selection, number of seeds, tail and regime stress, out-of-sample/backtest limitations (jika relevan), parameter uncertainty.
9. **Security and load:** owner isolation, duplicate enqueue, request storms, job quotas, cancel race, storage and deletion.
10. **UX interpretability:** user memahami bahwa hasil conditioned on assumptions, bukan financial guarantee.

### 19.1 Approval sequence, no numerical shortcuts

```text
Model research notes
  -> explicit model candidate + alternatives
  -> source data/provenance and unit review
  -> independent statistical model review
  -> success/failed-path/percentile/error policies
  -> approved seed/RNG + version plan
  -> approved resource/worker/privacy policy
  -> frozen reference tests + benchmark thresholds
  -> owner signoff in Decision Log
  -> implementation task readiness
  -> code + actual tests + evidence
  -> R3 release gate
```

**Sebelum approval** agent boleh mengerjakan ekstraksi agregasi pure-function pada dataset sintetis, merancang test harness, benchmark scripts non-production, studi parameter, dan dokumentasi trade-off. **Tidak boleh** mengekspos hasil sebagai keputusan finansial production.

### 19.2 Statistical thresholds are decisions, not folklore

Tidak ada toleransi baku yang tepat untuk semua model/sampel. Acceptance ranges untuk mean/variance/correlation, sensitivity, replay/tolerance, confidence level, failure rates, path counts, runtime, dan precision harus dipilih menurut error budget dan multiple-testing considerations. Reviewer harus membuktikan bahwa test tidak terlalu longgar (melewatkan bug) atau terlalu sempit (flaky atas seed berbeda). Test suite yang selalu memakai satu seed hanya menguji satu lintasan generator dan tidak cukup untuk validasi distribusi.

## 20. Invariant register — `MC-I01`–`MC-I24`

Semua invariant di bawah berlaku sebagai acceptance guard kecuali item yang eksplisit menyatakan prasyarat keputusan. Ini **ID lokal** dan tidak menggantikan `DR-001`–`DR-024`.

| ID | Invariant | Source / proof route |
|---|---|---|
| `MC-I01` | Semua run/read/cancel/export owner-scoped server-side. | `DR-003`, security integration tests |
| `MC-I02` | Run tidak memodifikasi actual ledger, account, asset, liability, atau goal-funded. | `DR-001/002/011/012`, read-only diff tests |
| `MC-I03` | `STOCHASTIC_RESULT` tidak pernah diberi label actual atau deterministic forecast. | `DR-014/015`, UI/e2e |
| `MC-I04` | Hasil mengikat immutable baseline revision dan as-of. | `DR-016`, snapshot replay |
| `MC-I05` | Hasil mengikat immutable scenario version dan canonical parameter revision. | `DR-016`, stale tests |
| `MC-I06` | Model dan engine version tercatat; perubahan model tidak menulis ulang hasil lama. | `DR-016`, history tests |
| `MC-I07` | Replay claim mensyaratkan seed, RNG/version, draw mapping, dan numeric policy. | `FTC-MC-05/06` |
| `MC-I08` | Semua units, frequency, nominal/real basis dinyatakan dan divalidasi. | `Q-04/05`, parameter tests |
| `MC-I09` | Correlation/covariance inputs yang invalid ditolak, bukan silent fix. | `FTC-MC-07`, validation tests |
| `MC-I10` | Goal planned contribution tidak otomatis funded/cash. | `DR-011`, R1/R3 integration |
| `MC-I11` | Transfer internal tidak menghasilkan phantom income/expense. | `DR-006`, per-path tests |
| `MC-I12` | Double-count asset/net worth tidak diperbolehkan. | `DR-009/010`, state tests |
| `MC-I13` | Depletion/shortfall nyata dalam model ditandai, tidak silently clamped/borrowed. | `CS-015`, `FTC-RET-*` |
| `MC-I14` | `n_success <= n_valid <= n_processed <= n_requested` untuk run yang tidak melewati quota. | Counter tests |
| `MC-I15` | `n_valid==0` tidak menghasilkan success percentage. | `FTC-MC-03` |
| `MC-I16` | Failed technical paths bukan financial failure paths. | `FTC-MC-04`, failure policy gate |
| `MC-I17` | Canceled/partial run tidak dipublikasikan sebagai completed. | `FTC-MC-09`, worker tests |
| `MC-I18` | Percentile selalu menyebut outcome variable, horizon, nominal/real dan approved algorithm. | `FTC-MC-08`, UX tests |
| `MC-I19` | Sampling uncertainty tidak disamakan dengan uncertainty model/dunia nyata. | `DR-015`, disclosure review |
| `MC-I20` | Status queued/running tidak diinterpretasi sebagai sukses. | `TS-034`, UX states |
| `MC-I21` | Bounded path/horizon/concurrency/time/memory; tidak ada unlimited job. | `EN-005`, load/resource tests |
| `MC-I22` | Duplicate worker delivery tidak melahirkan dua logical final results. | Worker crash/replay tests |
| `MC-I23` | Deletion/retention menjaga semua artifact, job, cache dan backup sesuai policy. | Security lifecycle tests |
| `MC-I24` | Model unapproved/stale/invalid tidak menghasilkan angka probabilitas production. | `DR-024`, R3 gate |

## 21. Test matrix — `MCT-001`–`MCT-060`

`MCT-*` adalah **skenario tambahan yang didefinisikan dokumen ini**, bukan hasil eksekusi atau pengganti oracle `FTC-MC-*`. Status awal keseluruhan adalah `NOT_RUN`. Label `READY-REFERENCE` berarti expected **dapat dihitung tanpa memilih stochastic model**; `CONDITIONAL` mensyaratkan policy approved; `BLOCKED` tidak boleh dinyatakan lulus sebelum keputusan.

### 21.1 Counters, algebra, dan publication semantics

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-001` | `n_valid=1000,n_success=720,n_failed=0` | `model_success_rate=0.72`, source `FTC-MC-01`; wording conditional | `READY-REFERENCE` |
| `MCT-002` | `n_requested=1000,n_valid=1000,n_failed=0` | Count identity holds after terminal processing | `READY-REFERENCE` |
| `MCT-003` | `n_valid=0,n_success=0` | No 0/0, no 0% success card | `READY-REFERENCE` |
| `MCT-004` | `n_requested=1000,n_valid=800,n_failed=200,n_success=720` | Conditional `720/800=0.90`; **not publishable** without failure policy | `READY-REFERENCE / BLOCKED publish` |
| `MCT-005` | `n_success > n_valid` | Aggregate rejected; diagnostic counter invariant fails | `READY-REFERENCE` |
| `MCT-006` | `n_processed > n_requested` | Reject corrupted metadata, no result publish | `READY-REFERENCE` |
| `MCT-007` | Valid path ends below target without technical error | Financial failure counted within valid, not `n_failed` | `CONDITIONAL` success predicate |
| `MCT-008` | Numeric overflow/NaN in path | Technical failed path; do not silently treat as no-goal-success | `CONDITIONAL` numeric policy |
| `MCT-009` | Cancel before all paths classified | Terminal canceled/partial; success card unavailable | `READY-REFERENCE` |
| `MCT-010` | One synthetic success label without model/horizon | UI/server blocks or marks incomplete; no unsupported promise | `READY-REFERENCE` |

### 21.2 Deterministic replay, RNG, dan versioning

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-011` | Same canonical inputs, seed, versions, mapping | Same complete result under approved replay tier | `BLOCKED` RNG/model |
| `MCT-012` | Same seed, different RNG/version | No path identity promised; version discrepancy visible | `READY-REFERENCE` invariant |
| `MCT-013` | Same seed, different factor order | Fingerprint/replay identity changes; cannot silently compare | `CONDITIONAL` mapping |
| `MCT-014` | Same run across workers with different completion order | Path-index outputs and aggregate equal if replay tier promises | `BLOCKED` RNG mapping |
| `MCT-015` | Retries after worker crash, same run identity | Single logical result, no duplicate effect | `CONDITIONAL` queue policy |
| `MCT-016` | Same scenario, baseline changed afterward | Old result immutable and displayed stale, re-run is new identity | `CONDITIONAL` stale policy |
| `MCT-017` | Same baseline, model parameters changed | New model fingerprint/result lineage | `READY-REFERENCE` |
| `MCT-018` | Retired/unsupported RNG version | Block replay with clear status; no substitute PRNG | `CONDITIONAL` registry |
| `MCT-019` | Unstable serialization order of JSON/map | Canonical fingerprint stable by approved serialization | `BLOCKED` canonicalization |
| `MCT-020` | Hidden wall-clock reading by pure engine | Static/dynamic test detects forbidden nondeterministic dependency | `READY-REFERENCE` architecture |

### 21.3 Parameter/model validation dan distribution tests

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-021` | Negative horizon, zero period, out-of-policy date | Reject as invalid before scheduling worker | `READY-REFERENCE` |
| `MCT-022` | Path request above quota | Reject bounded request, no silent shrink | `BLOCKED` quota value |
| `MCT-023` | NaN/infinite input rate | Reject and prevent nonfinite output | `READY-REFERENCE` |
| `MCT-024` | Factor dimensions mismatch correlation matrix | Reject; do not auto-pad, truncate or assume independence | `READY-REFERENCE` |
| `MCT-025` | Asymmetric correlation matrix | Reject under correlation-based model | `CONDITIONAL` correlation model |
| `MCT-026` | Not PSD matrix for PSD-required model | Reject with safe code; no silent nearest-PSD projection | `BLOCKED` tolerance/factorization |
| `MCT-027` | Zero volatility for approved supported model | Degenerate factor follows approved deterministic expectation | `BLOCKED` model choice |
| `MCT-028` | A fixed known RNG test vector | Sample draw transforms match independent golden vectors | `BLOCKED` RNG algorithm |
| `MCT-029` | Multi-seed distribution diagnostic set | Accepted bands/false-positive rate documented and passed | `BLOCKED` statistical protocol |
| `MCT-030` | Heavy-tailed stress sample or unusual regime | Model limitation/sensitivity correctly disclosed | `BLOCKED` data/model review |

### 21.4 Financial state, goals, cashflow dan pensions

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-031` | Goal planned contribution, no actual funding mapping | No funded progress inflation, success criterion blocked if requires it | `READY-REFERENCE` |
| `MCT-032` | Transfer own cash account A→B, no fees/FX | Net worth and external income/expense unchanged in each path | `CONDITIONAL` ledger model |
| `MCT-033` | Investment return positive/negative | Investment value changes without phantom external income | `CONDITIONAL` gain accounting |
| `MCT-034` | Portfolio appears liquid but cash shortfall occurs earlier | Shortfall preserved; not masked by positive terminal net worth | `CONDITIONAL` eligibility |
| `MCT-035` | Same returns reordered with withdrawals | Path outcomes differ as illustrated in §11 | `READY-REFERENCE` toy arithmetic |
| `MCT-036` | House purchase event with mortgage | Hypothetical asset/loan/cash deltas conserve state under approved model; actual unchanged | `CONDITIONAL` scenario rules |
| `MCT-037` | Inflation nominal-to-real conversion | Once-only conversion, reference date/unit visible | `CONDITIONAL` inflation policy |
| `MCT-038` | Cash insufficient for withdrawal | Model-defined shortfall/depletion, no automatic loan | `CONDITIONAL` withdrawal policy |
| `MCT-039` | Asset shown both account balance and asset position | Reject double counting | `READY-REFERENCE` domain invariant |
| `MCT-040` | Retirement success requested without approved withdrawal rule | `BLOCKED`, not a fabricated FIRE success probability | `READY-REFERENCE` |

### 21.5 Async reliability, safety, dan privacy

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-041` | Unauthorized user guesses `run_id` | No resource/status/result leakage, error neutral | `CONDITIONAL` API/DB |
| `MCT-042` | Same logical run submitted twice during timeout | One logical work item/result after approved reconciliation | `CONDITIONAL` idempotency |
| `MCT-043` | Queue worker crashes after result commit, before ACK | Replay cannot publish duplicate result | `CONDITIONAL` worker |
| `MCT-044` | User cancels while near result commit | Race resolves to one approved terminal state, no false partial completed | `CONDITIONAL` state machine |
| `MCT-045` | Deadline/resource quota exceeded | Controlled error state, no unbounded process | `BLOCKED` resource thresholds |
| `MCT-046` | User account/result deleted during processing | Result cannot remain retrievable; policy-compliant cancellation/purge | `CONDITIONAL` retention |
| `MCT-047` | Log sampled from worker error | No account balance/raw personal data/token/PII in log | `READY-REFERENCE` |
| `MCT-048` | Owner changes scenario version during queued work | Run uses frozen old version; stale/new version explicit | `CONDITIONAL` snapshot policy |
| `MCT-049` | Partial path artifacts left in temp storage | Not accessible as completed output; cleaned by approved lifecycle | `CONDITIONAL` retention |
| `MCT-050` | User submits massive run batch | Rate/concurrency quota enforces fair bounded admission | `BLOCKED` quota policy |

### 21.6 UX/interpretation, result compatibility, dan acceptance

| ID | Given / action | Expected acceptance | Readiness |
|---|---|---|---|
| `MCT-051` | Result card shows 720/1000 | Copy states conditional frequency, assumptions and no guarantee | `READY-REFERENCE` |
| `MCT-052` | Only P90 displayed, no outcome variable/date/units | Block or request metric context; no naked percentile | `READY-REFERENCE` |
| `MCT-053` | Quantile method not approved | No numerical P10/P50/P90 labeled production | `READY-REFERENCE` |
| `MCT-054` | Percentile band across periods | Chart correctly describes cross-sectional quantiles, not single path | `CONDITIONAL` metric policy |
| `MCT-055` | Success rate accompanied by modeled sampling CI | UI distinguishes sample error vs structural/parameter model risk | `BLOCKED` CI policy |
| `MCT-056` | Old model version compared to updated model | Warning prevents direct comparability without explanation | `READY-REFERENCE` |
| `MCT-057` | Missing approved model or data | Blocked/insufficient state, no fake chart | `READY-REFERENCE` |
| `MCT-058` | Chart with user text scaling/screen reader | Accessible summary includes units, interval, model, caveats | `CONDITIONAL` Flutter implementation |
| `MCT-059` | UI request returns QUEUED only | No result/success badge until publishable completed | `READY-REFERENCE` |
| `MCT-060` | Full R3 acceptance walk-through | Approvals, statistical tests, worker security, performance, UX evidence and owner signoff all present | `BLOCKED` release gate |

**Trace to primary tests:** `FTC-MC-01`–`FTC-MC-10` remain the authoritative numeric references where approved. `MCT-*` provides implementation-oriented coverage, not replacement. `TS-034` covers worker/run ownership; `TS-Q10` still blocks unconditional R3 statistical acceptance. Any added numeric golden output must be independently reviewed in `FINANCIAL-TEST-CASES.md` or an approved test artifact, not guessed in this file.

## 22. Open decisions register — `MC-Q01`–`MC-Q18`

Seluruh item di bawah berstatus **`TBD / PENDING_OWNER_APPROVAL`** kecuali ada keputusan eksplisit terbaru di [`DECISION-LOG.md`](../planning/DECISION-LOG.md). ID lokal ini *tidak* menggantikan `Q-08` dan keputusan asal. Jumlah/jenis keputusan perlu tetap sinkron dengan register pusat.

| ID | Parent/source | Keputusan yang diperlukan | Dampak jika belum dipilih |
|---|---|---|---|
| `MC-Q01` | `Q-08`, `FDR-Q12`, `CALC-Q13` | Model family, eligible asset classes, region/currency | **BLOCKED** production sampler |
| `MC-Q02` | `Q-08`, `Q-05` | Parameter return, volatility, unit/time frequency, calibration window | **BLOCKED** path values |
| `MC-Q03` | `Q-08`, `CALC-Q13` | Joint correlation/dependence model, matrix validation/tolerance | **BLOCKED** correlated sampling |
| `MC-Q04` | `Q-08`, `BE-Q12` | PRNG algorithm/version, seed policy, stream/path mapping | **BLOCKED** replay guarantees |
| `MC-Q05` | `Q-05`, `Q-08` | Inflation stochastic/deterministic and nominal/real treatment | **BLOCKED** real-valued output |
| `MC-Q06` | `Q-08` | Tail risk, regime shifts, extreme losses, sensitivity policy | **BLOCKED** statistical risk review |
| `MC-Q07` | `Q-08`, `Q-11` | Model success predicates, eligible liquidity and goal funding | **BLOCKED** success claim |
| `MC-Q08` | `Q-08`, `FTC-MC-04` | Technical failed-path denominator, publishability threshold, partial paths | **BLOCKED** rate publication |
| `MC-Q09` | `Q-08`, `CALC-Q13` | Quantile method, terminal and time-series metrics, approximate tolerance | **BLOCKED** P10/P50/P90 production |
| `MC-Q10` | `Q-08`, `TS-Q10` | Sampling error/CI method, confidence wording, convergence evidence | **BLOCKED** confidence UI |
| `MC-Q11` | `EN-005`, `NFR-004` | Path/horizon factor quota, CPU/memory/time/concurrency SLO | **BLOCKED** heavy production runs |
| `MC-Q12` | `ARCH-Q10`, `BE-Q12` | Worker/queue tech, dedup, retry, cancellation, crash recovery | **BLOCKED** async lifecycle |
| `MC-Q13` | `Q-06`, `DR-016` | Snapshot canonicalization, run identity, stale and rebase semantics | **BLOCKED** reproducible lineage |
| `MC-Q14` | `Q-10`, `SEC-Q15/16/20` | Raw path/aggregate/seed retention, deletion/export, PII logging | **BLOCKED** data lifecycle |
| `MC-Q15` | `Q-08`, `CALC-Q14` | Retirement withdrawals, depletion, horizon, liquidity, taxes/fees | **BLOCKED** probabilistic FIRE |
| `MC-Q16` | `API-Q16`, `DB-Q16` | Result schema, metrics, error taxonomy, version compatibility | **BLOCKED** final DB/API contract |
| `MC-Q17` | `DS-Q12`, `SS-Q17` | Risk disclosure/copy, percentile chart and accessibility acceptance | **BLOCKED** user-facing risk claims |
| `MC-Q18` | `Q-08`, `TS-Q10`, `DR-024` | Independent reviewer, validation datasets, sign-off/rollback criteria | **BLOCKED** production release |

### 22.1 Required decision record fields

Setiap keputusan yang benar-benar disetujui harus memuat: `decision_id`, owner/reviewer, dated approval, candidate options, rationale/trade-off, affected `FR/CS/DR/SEC/TS`, chosen model & version, approved parameter/unit ranges, test oracle, statistical acceptance bands, implementation consequences, migration/replay compatibility, observability, privacy, rollback, dan next review trigger. `REVIEWED_INTERNAL` **tidak** sama dengan `DECIDED`. Setelah keputusan tercatat, perbarui sumber utama dan tambahkan regression coverage sebelum menandai affected task READY.

### 22.2 Decision priority / sequencing

1. **First:** `MC-Q01`–`MC-Q08` (model, financial meaning, RNG, success/failed path) menentukan arti hasil.
2. **Next:** `MC-Q09/10/15` menentukan outcome/uncertainty/retirement interpretation.
3. **In parallel before production:** `MC-Q11`–`MC-Q14`, `MC-Q16/17` menentukan operasi, keamanan, API/UX.
4. **Last release gate:** `MC-Q18` memerlukan evidence dari seluruh jalur di atas. Gagal satu hard gate → `BLOCKED`, bukan waiver diam-diam.

## 23. AI coding agent implementation slices (planning, not authorization)

Jangan menafsirkan langkah di bawah sebagai task baru yang otomatis siap/approved. Semua langkah harus direkonsiliasi dengan `EP-S11-01`, `MASTER-ROADMAP.md`, dan `PROGRESS.md`, lalu dipecah menjadi bounded tasks setelah keputusan terkait disetujui.

| Candidate step | Prasyarat | Deliverable yang dapat direview | Test/evidence minimum |
|---|---|---|---|
| MC-PLAN-01 | `FR-018/CS-014` dipahami | Model requirements + source trace matrix | Reviewer confirm scope and blockers |
| MC-PLAN-02 | `FTC-MC-*` available | Pure aggregation function over synthetic path statuses | `MCT-001`–`010`, actual executed results |
| MC-PLAN-03 | Model family/params/units approved | Versioned factor sampler module | `MCT-021`–`030`, independent vectors |
| MC-PLAN-04 | RNG algorithm & policy approved | Deterministic random streams/path assignment | `MCT-011`–`020`, parallel replay |
| MC-PLAN-05 | Scenario Engine R2 verified | Pure stochastic path adapter with immutable snapshot input | `MCT-031`–`040`, financial invariants |
| MC-PLAN-06 | Failure/quantile/success semantics approved | Aggregator, result publication validator, metrics | `FTC-MC-*`, `MCT-051`–`055` |
| MC-PLAN-07 | Worker/DB/API decision approved | Run lifecycle, authz, dedup/cancel/retry, retention | `MCT-041`–`050`, `TS-034`, security |
| MC-PLAN-08 | Design/UX choices approved | Flutter risk setup/results and accessible disclosure | `UF-27`, `SCR-19`, UX/security tests |
| MC-PLAN-09 | Retirement policy approved | Optional conditional FR-019 adapter | Depletion, withdrawal, `FTC-RET-*` |
| MC-PLAN-10 | All model tests and owner approvals recorded | R3 risk simulation acceptance package | DoD, perf, statistical review, rollback plan |

**Recommended architecture dependency rule:** `Go pure model functions` depend on approved numeric/state inputs only; application service wraps them for auth/version/DB/workers; Flutter consumes typed result and model metadata; PostgreSQL persists immutable metadata/artifacts per approved lifecycle. A package suggestion is not a Go directory layout that must be created blindly.

### 23.1 Example bounded work request for agent

```text
Task: Implement synthetic aggregation reference (MC-PLAN-02 only)
Scope: Go pure function that accepts pre-evaluated valid/failed/success path statuses.
Required docs: AGENTS.md, backend/AGENTS.md, CS-014, FTC-MC-01..04,
               this MONTE-CARLO.md, TESTING-STRATEGY.md, DEFINITION-OF-DONE.md.
Explicit exclusions: no distribution, no RNG, no production success card,
                     no endpoint, no SQL migration, no real user data.
Acceptance: same input counts => exact reference ratios; zero-valid is nonpublishable;
            invalid count state rejected; no actual mutation.
Evidence: test command, output, changed files, untested areas, status.
Blockers: choose API/DB model semantics only through Decision Log if needed.
```

## 24. Definition of Ready and Definition of Done — R3 risk model

### 24.1 Definition of Ready for actual FR-018 implementation

- [ ] `EP-S08-02` scenario versioning, baseline fingerprints and deterministic engine are verified for required paths.
- [ ] `EP-S10-02` EN-005 worker lifecycle, deduplication, quota, and cancellation accepted, or owner-approved alternative documented.
- [ ] `Q-08`, `CALC-Q13`, `FDR-Q12` and all affected `MC-Q*` decisions have dated owner/statistical reviewer approval.
- [ ] Input classifications, eligible assets, currency, timing, inflation, return and correlation models are explicit and sourceable.
- [ ] RNG/version/seed/draw mapping and reproducibility tier approved.
- [ ] Success evaluator, invalid path policy, outcome metrics, quantiles, uncertainty/disclosure approved.
- [ ] Numeric fixtures, sampler statistical validation protocol, test bands and load thresholds independently reviewed.
- [ ] API/DB/worker/retention policies approved and reconciled; no fabricated route/migration.
- [ ] Cross-user isolation, sensitive-data storage, deletion and observability design reviewed.

### 24.2 Definition of Done for delivered FR-018 (future, not achieved)

- [ ] Requirements and policy decisions trace to code, runtime config, migrations, and OpenAPI where applicable.
- [ ] All relevant `FTC-MC-*`, `MCT-*`, `TS-034`, `ST-*`, `UXT-*` cases **actually executed** with evidence and explicit statuses.
- [ ] Independent statistical reviewer approves distributions, correlation, model calibration/limitations and acceptance report.
- [ ] Seeded replay and non-deterministic execution tests pass within approved exactness/tolerance.
- [ ] Path aggregation/count/failed-path thresholds and percentile formulas pass independent tests.
- [ ] Assets, cash, debt, goals, transfer and actual/forecast boundaries hold for per-path results.
- [ ] Worker timeout, cancellation, duplicate delivery, crash/retry, deletion and ownership tests pass.
- [ ] Resource and performance budgets measured under representative workload; release limits enforced.
- [ ] Flutter disclosure, accessibility and stale/blocked/partial states pass UX validation.
- [ ] Privacy review, retention/export/deletion evidence, incident handling and rollback plan are accepted.
- [ ] Product Owner signs off R3 release gate with no unresolved critical blockers.

**Tidak ada item pada dua checklist di atas yang otomatis dianggap checked** hanya karena dokumen ini dibuat. `NOT_RUN` harus tetap `NOT_RUN`; gagal/blocked bukan "passed with caveat".

## 25. Handoff, evidence dan review protocol

Agent harus mengirim laporan setiap task dengan struktur berikut (contoh template, bukan bukti pekerjaan telah terjadi):

```text
TASK / EPIC: <actual-id-from-TASKS.md>
SCOPE: <what changed / what explicitly did not change>
SOURCE DOCS: <path + version / decision IDs>
OWNER APPROVAL: <decision ID + date, or BLOCKED>
BASELINE & MODEL: <fingerprints/versions, redact sensitive data>
RNG & SAMPLING: <version/seed handling, if applicable>
TEST MATRIX: <FTC/MCT/TS/ST/UXT IDs with PASS/FAIL/NOT_RUN/SKIPPED/BLOCKED>
EXECUTED COMMANDS: <real commands and exit codes; not hypothetical>
STATISTICAL REVIEW: <reviewer/evidence link or NOT_RUN>
SECURITY/PERFORMANCE: <measured evidence or NOT_RUN>
FILES CHANGED: <actual repository paths>
RISKS / BLOCKERS: <including model uncertainty and open decisions>
STATUS: DONE | PARTIAL | BLOCKED | UNVERIFIED
NEXT SAFE TASK: <bounded action and its prerequisites>
```

Tidak boleh menyimpan raw PII, secrets, atau personal financial histories dalam progress logs. Jika agen tidak memiliki repository atau environment test, ia dapat menyatakan **dokumentasi tersusun**, tetapi **bukan** `DONE` untuk fitur `FR-018`.

## 26. Cross-document traceability checklist

| Item | Authoritative origin | Monte Carlo usage | Status at document draft |
|---|---|---|---|
| `FR-018` | `PRD.md` | Risk simulation product behavior | `PENDING_OWNER_APPROVAL` / R3 |
| `CS-014` | `CALCULATION-SPECS.md` | Model success, quantiles, seeded path contract | **BLOCKED** distribution/percentile policy |
| `FTC-MC-01..10` | `FINANCIAL-TEST-CASES.md` | Independent aggregation references & blocked tests | Mixed `REF-EXACT`/conditional/blocked |
| `DR-001/002` | `FINANCIAL-DOMAIN-RULES.md` | Actual isolation | REQUIRED invariant |
| `DR-011/012` | `FINANCIAL-DOMAIN-RULES.md` | Planned contribution ≠ actual-funded | REQUIRED invariant |
| `DR-015/016` | `FINANCIAL-DOMAIN-RULES.md` | Not guaranteed; replay lineage | REQUIRED invariant |
| `DR-024` | `FINANCIAL-DOMAIN-RULES.md` | Advanced model approval | HARD R3 GATE |
| `NFR-003/004` | `PRD.md` | Reproducibility and performance budgets | Values/policies TBD |
| `EN-005` | `FEATURE-MATRIX.md` / `PRD.md` | Bounded background processing | R3 prerequisite |
| `EP-S11-01` | `TASKS.md` | Seeded Monte Carlo implementation epic | Planning, not done |
| `EP-S11-02` | `TASKS.md` | Optional retirement integration | Requires separate approval |
| `UF-27`, `SCR-19` | `USER-FLOWS.md`, `SCREEN-SPECS.md` | Risk setup, queue, results UX | R3 spec only |
| `TS-034`, `TS-Q10` | `TESTING-STRATEGY.md` | Run lifecycle, validation bands | Gate open |
| `Q-08` | `PRD.md`, `DECISION-LOG.md` | Model, RNG, statistical claims | `NOT_RECORDED` |
| `MC-Q01..18` | This file | Local decision routing | `TBD` |

## 27. Document QA checklist (not application tests)

- [x] Required financial isolation, no auto-post and no false success claims specified.
- [x] Source of truth for `CS-014` and `FTC-MC-01..10` linked without redefining expected production percentiles.
- [x] Model family / sample size / RNG / correlation / quantile / CI / failed-path policies remain `PROPOSED`/`TBD`.
- [x] Decision parents for R3 and worker release gates listed explicitly.
- [x] 24 local invariants and 60 local test definitions included as **not-yet-executed** artifacts.
- [x] Reproducibility identity records baseline, scenario, model/engine/RNG, seed, mapping, numeric policy.
- [x] Financial and retirement outputs clearly distinguish liquid cash, nonliquid wealth, funded progress and hypothetical forecast.
- [x] Worker quotas, duplicate delivery, cancel/timeout and deletion retention represented.
- [x] R3 `FR-018` implementation explicitly remains `UNVERIFIED / BLOCKED` until approvals and executed tests.

**QA note:** checklist ini adalah pemeriksaan isi naskah, bukan laporan berhasil menjalankan kode Go/PostgreSQL/Flutter. Pemeriksaan otomatis tautan, ID, serta integritas ZIP dilakukan terpisah ketika artefak didistribusikan.

## 28. Penutup dan urutan kerja berikutnya

1. Review model bersama pemilik keputusan, statistik/keuangan, backend, security, dan UX; rekam `Q-08` dan `MC-Q*` dalam Decision Log.
2. Setelah approval, pecah `EP-S11-01` menjadi bounded tasks dengan acceptance serta test oracle independen; jangan memulai sampler production dari proposal ini saja.
3. Dokumentasikan perbandingan dengan goal optimization (dokumen selanjutnya: `docs/financial/GOAL-OPTIMIZER.md`) dan pipeline deployment/retention (`docs/operations/DEPLOYMENT.md`), keduanya **masih rencana** pada tahap dokumen ini.
4. Sebelum implementasi, audit monorepo, OpenAPI, migration, tests, dan keputusan owner yang benar-benar ada; perbarui [`docs/INDEX.md`](../INDEX.md), [`TASKS.md`](../planning/TASKS.md), [`PROGRESS.md`](../planning/PROGRESS.md), dan [`DECISION-LOG.md`](../planning/DECISION-LOG.md) ketika artefak sudah ditempatkan dan disetujui.

**Kesimpulan:** Monte Carlo Esokira dapat menghasilkan *estimasi risiko bersyarat atas model yang disetujui dan tervalidasi*. Tidak boleh mengubah kondisi keuangan aktual, mengarang kepastian, atau dianggap siap hanya karena formula dan pseudo-algoritmanya sudah dituliskan.

---

### Lampiran A — Contoh headline hasil yang aman

> *Dalam model simulasi dengan parameter dan horizon yang tertera, 720 dari 1.000 lintasan valid memenuhi definisi target ini (72% keberhasilan model). Angka tersebut tergantung pada asumsi dan tidak menjamin hasil nyata. Perhatikan kondisi ketika saldo kas tidak cukup dan sensitivitas terhadap perubahan return, inflasi, serta horizon.*

Tidak boleh memakai contoh tersebut sebagai hasil pengguna tertentu atau menunjukkan 72% bila tidak berasal dari run yang benar-benar memenuhi metadata/quality gate.

### Lampiran B — Non-production analysis checklist untuk reviewer

- Apakah target pencapaian diukur pada deadline dan dengan **funding eligible**, bukan hanya planned contributions?
- Apakah cash shortfalls dihitung setiap periode dan tidak ditutupi positive terminal wealth?
- Apakah correlations masuk akal dan disertai sumber, validasi matrix, serta model uncertainty?
- Apakah random streams tetap stabil di worker concurrency berbeda?
- Apakah success/failed-path counts dan denominator dapat direproduksi?
- Apakah quantile outcomes, nominal/real basis, dan confidence claims dibedakan secara jelas?
- Apakah insufficient data/cancel/invalid run tidak menghasilkan angka publishable?
- Apakah tests statistik dapat gagal ketika sengaja diberi biased generator atau missing tails?
- Apakah privacy/retention dan owner-level authz mencakup metadata, artifacts, dan background jobs?
- Apakah owner sudah menyetujui `Q-08`, bukan hanya menyetujui penulisan dokumen?

### Lampiran C — Referensi regresi lintas fitur

| Integration | Regression guard | Rujukan |
|---|---|---|
| Actual transactions → baseline | No double posting, opening balance not income, transfers neutral | [`TRANSACTIONS.md`](../features/TRANSACTIONS.md) |
| Goals → success metric | Planned contribution not funded; deadline and eligible assets explicit | [`FINANCIAL-GOALS.md`](../features/FINANCIAL-GOALS.md) |
| Life events → stochastic path | Hypothetical effects only, consistent cash/assets/debt | [`LIFE-PLANNING.md`](../features/LIFE-PLANNING.md) |
| Scenario engine → path transition | Identical approved event ordering, shortfall and amortization | [`SCENARIO-ENGINE.md`](SCENARIO-ENGINE.md) |
| Run → UI | Source, units, assumptions, states, uncertainty and accessible text | [`SCREEN-SPECS.md`](../design/SCREEN-SPECS.md) |
| Worker → DB/API | Owner scope, idempotent outcomes, state machine, retention | [`BACKEND-ARCHITECTURE.md`](../architecture/BACKEND-ARCHITECTURE.md) |
| Privacy → raw paths/result | No sensitive logging, export/deletion and proper storage boundaries | [`SECURITY-PRIVACY.md`](../security/SECURITY-PRIVACY.md) |

