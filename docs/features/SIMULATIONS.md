# Esokira — SIMULATIONS

**Dokumen:** Spesifikasi fitur simulasi, scenario builder, scenario comparison, future cash-flow forecast, dan stress-test experience  
**Path kanonis:** `docs/features/SIMULATIONS.md`  
**Versi:** `0.1.0`  
**Tanggal draft:** `2026-10-30`  
**Status review:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
**Kesiapan implementasi:** `SPECIFICATION ONLY; R2 BLOCKED until relevant decisions, engine specification, API contract, and tests are approved`  
**Penanggung jawab persetujuan:** Product Owner / pihak yang ditunjuk, belum ada approval tercatat  
**Scope utama:** R2, `S-08` (`FR-012/013/014`) dan `S-09` (`FR-015/016`); integrasi `S-07` (`FR-010/011`). R3 dijelaskan hanya sebagai batas integrasi.

> **Untuk AI coding agent:** Jangan menulis implementasi produksi fitur simulasi hanya karena dokumen ini sudah ada. Kontrak ini menjelaskan *apa yang harus dialami pengguna* dan *apa yang harus benar*. Detail matematika, penyimpanan, endpoint, retensi, concurrency, dan aturan waktunya belum semuanya disetujui. Baca `../planning/DECISION-LOG.md` dan `../financial/CALCULATION-SPECS.md` sebelum merencanakan pekerjaan; gunakan `../financial/SCENARIO-ENGINE.md` setelah file tersebut benar-benar tersedia dan telah memperoleh approval yang sesuai. **Hasil simulasi tidak boleh mengubah uang aktual.**

---

## 0. Tujuan, otoritas, dan batas pemakaian

Esokira membantu pengguna mengeksplorasi pertanyaan seperti **"bagaimana jika saya membeli rumah tahun depan?"**, **"bagaimana jika penghasilan saya turun selama beberapa bulan?"**, dan **"apa perbedaan dua jalur hidup dalam kondisi yang saya asumsikan?"** Hasil merupakan kalkulasi bersyarat berdasarkan data dan asumsi, **bukan** ramalan, kepastian, rekomendasi investasi personal, atau perintah memindahkan uang.

Kontrak ini mengatur *experience layer*: pembuatan dan pemilihan input skenario, provenance, visualisasi, komparabilitas, error/stale states, serta kebutuhan acceptance testing. Hal-hal yang **tidak** diotorisasi oleh dokumen ini:

1. Menetapkan rumus projection dan event ordering yang belum disetujui.
2. Memilih jenis uang, precision, rounding, rate, calendar, atau interpretasi negative cash tanpa kontrak final.
3. Memilih skema database, nama endpoint, DTO, library chart, router, auth provider, queue, atau worker.
4. Menghasilkan probabilitas kesuksesan, percentile, rekomendasi optimasi, atau klaim FIRE pada R2; itu milik R3 dengan validasi khusus.
5. Mengubah transaksi, opening balance, asset valuation, liability, atau funded goal aktual sebagai efek simulation run.
6. Memasukkan R4 ke implementasi: fitur R4 tetap discovery sampai disetujui.

**Urutan otoritas:** approved owner decisions / accepted ADR dan kontrak implementasi yang benar-benar telah disetujui → PRD/Domain Rules/Calculation Specs → Feature Matrix & User Flows/Screen Specs → dokumen fitur ini. Jika terdapat benturan, *jangan* pilih sendiri: catat konflik, tentukan blast radius, dan laporkan `BLOCKED`. Konten bertanda `PROPOSED`, `TBD`, `CONDITIONAL`, atau `EXAMPLE` tidak berubah status hanya karena dirujuk dari sini.

**Terminologi status dokumen:** `REVIEWED_INTERNAL` berarti konsistensi dokumen diperiksa secara internal; **bukan** izin membangun atau bukti pengujian aplikasi. Persetujuan hanya sah bila dicatat eksplisit dalam Decision Log.

## 1. Peta sumber kebenaran dan hubungan dokumen

Agent membaca **dokumen yang relevan terhadap task**, bukan membabi buta mengulang seluruh spesifikasi. Tautan berikut mengarah ke lokasi kanonis di monorepo; validasi harus diulang ketika seluruh arsip digabung ke repository nyata.

| Kebutuhan | Sumber utama | Alasan membuka |
|---|---|---|
| Scope R2/R3 dan acceptance fitur | [PRD](../product/PRD.md), [Feature Matrix](../product/FEATURE-MATRIX.md) | Tidak menambah fitur atau release diam-diam. |
| Rencana implementasi dan dependensi | [Master Roadmap](../planning/MASTER-ROADMAP.md), [Tasks](../planning/TASKS.md), [Progress](../planning/PROGRESS.md) | `S-07`–`S-09` masih epics, belum bounded executable tasks. |
| Keputusan belum final dan approval | [Decision Log](../planning/DECISION-LOG.md), [ADR-0001](../architecture/ADR/0001-TECH-STACK.md) | `Q-04`, `Q-05`, `Q-06`, `Q-11`, keputusan API/DB/FE. |
| Larangan actual mutation dan semantic financial | [Financial Domain Rules](../financial/FINANCIAL-DOMAIN-RULES.md) | `DR-001/002/003/011/012/014/015/016/017/019/020/021/024`. |
| Formula, units, numeric references | [Calculation Specs](../financial/CALCULATION-SPECS.md) | `CS-008/009/010/011/012/016`; referensi **bukan seluruhnya approved policy**. |
| Reference numerical cases | [Financial Test Cases](../financial/FINANCIAL-TEST-CASES.md) | `FTC-CASH-*`, `FTC-SCEN-*`, `FTC-STRESS-*`, `FTC-BUD-*`. |
| Transaksi dan saldo aktual | [Transactions](TRANSACTIONS.md) | Posted actual vs hypothetical, fee, transfer neutrality. |
| Goal dan funded status | [Financial Goals](FINANCIAL-GOALS.md) | `Q-11`; planned contribution bukan uang aktual tersedia. |
| Life event inputs | [Life Planning](LIFE-PLANNING.md) | `FR-011`, `LPT-*`, dated event hypothetical. |
| User journeys dan state visual | [User Flows](../design/USER-FLOWS.md), [Screen Specs](../design/SCREEN-SPECS.md) | `UF-19..24`, `SCR-14..17`; aksesibilitas. |
| Design system dan UX content | [Design System](../design/DESIGN-SYSTEM.md) | Chart semantics, layout, loading, empty/error states. |
| Boundary Go/Flutter & modules | [System Architecture](../architecture/SYSTEM-ARCHITECTURE.md), [Backend Architecture](../architecture/BACKEND-ARCHITECTURE.md), [Frontend Architecture](../architecture/FRONTEND-ARCHITECTURE.md) | Ownership, financial calculation source, typed results. |
| API, versioning, job | [API Guidelines](../architecture/API-GUIDELINES.md) | Shape dan endpoint **usulan**, bukan final OpenAPI. |
| Persistence, stale, deletion | [Database Design](../architecture/DATABASE-DESIGN.md) | Baseline/version/run/result candidates, bukan migration final. |
| Sensitive data and controls | [Security & Privacy](../security/SECURITY-PRIVACY.md) | Owner-scoped access, session lifecycle, retention, nonleaky logs. |
| Execution and sign-off | [Development Workflow](../development/DEVELOPMENT-WORKFLOW.md), [Testing Strategy](../development/TESTING-STRATEGY.md), [Definition of Done](../development/DEFINITION-OF-DONE.md) | Evidence-based `DONE`; no fictional PASS. |
| Frontend/backend agent constraints | [Root AGENTS](../../AGENTS.md), [Frontend AGENTS](../../frontend/AGENTS.md), [Backend AGENTS](../../backend/AGENTS.md) | Subtree rules extend root rules. |
| Index navigasi | [Docs Index](../INDEX.md) | Membedakan file yang benar-benar ada dari rencana. |

**Dokumen turunan berikutnya yang belum dibuat saat draft ini:** `../financial/SCENARIO-ENGINE.md`, `../financial/MONTE-CARLO.md`, `../financial/GOAL-OPTIMIZER.md`, dan `../operations/DEPLOYMENT.md`. Mereka dicantumkan sebagai **rencana, bukan tautan ke file tersedia**, kecuali sudah dibuat pada saat dokumen ini digunakan kembali.

## 2. Pemetaan release, requirement, dan delivery slice

| Fase | Kebutuhan | Pengalaman pengguna | Entry gate | Batas |
|---|---|---|---|---|
| `R2/S-07` | `FR-010` | Budget/recurring sebagai input rencana yang berbeda dari posted actual | Recurring timing/idempotency & budget policies approved | Event terjadwal bukan actual otomatis. |
| `R2/S-07` | `FR-011` | Life-event editor pada planning draft | Event model, baseline parent dan date semantics reviewed | Belum harus ada comparison UI. |
| `R2/S-08` | `FR-012` | Jalankan projection deterministik, tampilkan timeline hasil | Verified engine, approved model/time policy | Tidak menulis actual. |
| `R2/S-08` | `FR-013` | Fork skenario, versi immutable, compare A/B | Baseline snapshot/revision/rebase policies | Tidak overwrite versi lama. |
| `R2/S-08` | `FR-014` | Future cash-flow chart/table dan shortfall | Projection output yang tervalidasi | Forecast tidak disamakan history. |
| `R2/S-09` | `FR-015` | Stress shock income/expense dan perbandingan | Approved shock/shortfall semantics, paired results | Tidak ada "jaminan aman". |
| `R2/S-09` | `FR-016` | Explainable rule-based insights | Approved rules dengan provenance | No generative guess or investment action. |
| `R2/S-09` | `FR-022` | Reminder opt-in terkait goal/rencana | `Q-09` privacy, consent, provider | **Bukan** bagian run model. |
| `R3/S-10..12` | `FR-017/018/019/020` | Optimizer, Monte Carlo, Retirement/FIRE, Financial GPS | Standalone approved specs, statistical validation, worker readiness | **OUT OF IMPLEMENTATION SCOPE** dokumen ini. |

**Urutan dependensi:** S-07 → `EP-S08-01` (projection) → `EP-S08-02` (versions/compare) + `EP-S08-03` (forecast) → `EP-S09-01` (stress) → `EP-S09-02` (insights) → `EP-S09-03` (consent/reminders & R2 release gate). Urutan ini merupakan dependencies, bukan izin otomatis untuk memulai epics R2. Sebelum coding, pecah setiap epic menjadi task sempit sesuai `TASKS.md` dan dokumentasikan statusnya di `PROGRESS.md`.

## 3. Definisi istilah yang tidak boleh bergeser

| Istilah | Makna spesifikasi | Bukan |
|---|---|---|
| `Actual` | Transaksi/posisi yang dicatat pengguna sebagai nyata pada backend | Data bank yang sudah diverifikasi pihak ketiga. |
| `Planned` | Niat/schedule/goal/life event yang belum otomatis terjadi | Posted expense, new asset atau funded balance. |
| `Baseline` | Posisi awal yang teridentifikasi, as-of, actual revision dan input scope tertentu | Angka terbaru yang boleh mengganti snapshot lama tanpa jejak. |
| `Scenario` | Wadah pilihan masa depan berbasis baseline dan asumsi | Duplicate account pengguna. |
| `Scenario draft` | Model perencanaan yang masih diedit sesuai approval | Hasil run final yang tidak dapat dibedakan dari versi immutable. |
| `Scenario version` | Versi input yang stabil/immutable untuk kalkulasi ulang | Label nama saja yang dapat digeser semaunya. |
| `Run` | Satu eksekusi engine dengan input dan engine version tertentu | Transaksi debit/kredit. |
| `Result` | Output berbasis input/assumptions dan run yang diketahui | Mutasi saldo aktual ataupun janji hasil. |
| `Stale` | Hasil dengan baseline/input revision sudah bukan versi terkini untuk konteks yang dibandingkan | Bukti output lama salah dihitung. |
| `Rebase` | Aksi eksplisit membentuk versi baru atas basis aktual terbaru | Refresh otomatis yang menimpa versi sebelumnya. |
| `Cash-flow forecast` | Estimasi perubahan kas masa depan per periode | Laporan pemasukan/pengeluaran aktual. |
| `Projected balance` | Perkiraan posisi kas akhir periode dalam model | Uang yang tersedia secara faktual untuk dibelanjakan. |
| `Shortfall` | Kesenjangan kas terhadap threshold model dan periode tertentu | Pernyataan pengguna dipastikan gagal bayar. |
| `Stress test` | Hasil skenario dengan shock yang dinyatakan eksplisit | Model probabilitas/Monte Carlo. |
| `Insight` | Penjelasan rule-based yang bisa ditelusuri ke input/hasil | Saran investasi pasti atau keputusan otonom. |
| `Comparable` | Dua run memiliki konvensi/metric dan dimensi yang selaras atau telah dinormalisasi menurut kebijakan approved | Dua angka yang kebetulan sama-sama rupiah. |

## 4. Invariants tidak bisa ditawar (`SIM-I01`–`SIM-I20`)

| ID | Aturan wajib | Sumber / risiko yang dicegah |
|---|---|---|
| `SIM-I01` | Save, edit, delete, run, compare, rebase, cancel scenario **tidak mem-post** actual transaction, asset valuation, liability atau opening balance. | DR-001/002; NFR-010. |
| `SIM-I02` | Snapshot dan hasil lama tidak diubah diam-diam saat actual atau draft terbaru berubah. | DR-016/020; CS-012. |
| `SIM-I03` | Setiap hasil menampilkan provenance: source, baseline-as-of, scenario/version, horizon, input assumptions dan engine version. | DR-014/016. |
| `SIM-I04` | Uang tetap exact sesuai kontrak backend; UI tidak memakai `double` untuk menghasilkan saldo resmi. | DR-004; Q-04. |
| `SIM-I05` | Planned contribution ≠ actual funded progress; alokasi goal tidak menggandakan kekayaan. | DR-011/012; Q-11. |
| `SIM-I06` | Transfer internal tanpa fee/FX tidak menjadi income/expense baru dan tidak mengubah net worth. | DR-006/023. |
| `SIM-I07` | Skenario tanpa data kritis melaporkan `INSUFFICIENT_DATA` atau blocker spesifik; tidak mengisi dengan angka tersembunyi. | DR-017. |
| `SIM-I08` | Forecast memakai label historis vs projected yang berbeda, beserta unit, currency, dan tanggal. | DR-014/018. |
| `SIM-I09` | Nilai kas forecast negatif ditampilkan sebagai shortfall; tidak otomatis di-clamp ke nol atau ditambal dengan pinjaman fiktif. | FTC-CASH-04; CS-009/011. |
| `SIM-I10` | Comparison A/B tidak meranking hasil beda currency, horizon, nominal/real basis, model atau metric tanpa normalisasi approved. | FTC-SCEN-05/06. |
| `SIM-I11` | Rerun deterministik dengan input/fingerprint/engine sama menghasilkan output yang ekuivalen sesuai numerical policy. | FTC-SCEN-01; NFR-003. |
| `SIM-I12` | Shock adalah input eksplisit, tidak menyisipkan asumsi survival atau probabilitas keberhasilan. | DR-015/017; FR-015. |
| `SIM-I13` | Insight menyebut aturan pemicu, sumber dan keterbatasan, serta tidak melakukan tindakan eksternal. | DR-019; FR-016. |
| `SIM-I14` | Authz object-level, termasuk nested scenario/version/run/result dan link berbagi, diperiksa server-side per user. | DR-003; SEC. |
| `SIM-I15` | Data privat, assumed salary/asset/debt, dan detail skenario tidak muncul di general logs, telemetry, errors atau analytics bebas. | DR-021. |
| `SIM-I16` | Response kuno tidak boleh mengganti hasil pemilihan skenario yang lebih baru pada Flutter. | UF-19/21; async UI. |
| `SIM-I17` | Retry setelah timeout tidak otomatis membuat run/versi ganda; aksi tulis memakai identitas logis yang aman setelah policy disetujui. | API/DB idempotency. |
| `SIM-I18` | Cancel/timeout/failure berbeda dari projected negative outcome yang sah; keduanya tidak dapat disamakan. | UX error taxonomy. |
| `SIM-I19` | Event one-time/recurring dihitung paling banyak sesuai recurrence dan period policy yang disetujui; tidak ada silent double count. | DR-013/018; LP-Q07. |
| `SIM-I20` | R3 stochastic/optimization/retirement hanya berjalan setelah model, expected tests, dan safety copy disetujui; no fake probability. | DR-024; FR-017..020. |

Semua invariants merupakan **perilaku yang wajib dijaga**, bukan keputusan implementasi library. Jika suatu invariant tidak dapat dibuktikan karena keputusan belum final, status test yang tepat adalah `BLOCKED`/`NOT_RUN`, bukan `PASS`.

## 5. Peta user flows dan screen ownership

| Flow | Layar | Hasil yang harus dipahami pengguna | Varian penting |
|---|---|---|---|
| `UF-18` | `SCR-13` | Menambahkan life event ke planning draft | Data tidak masuk actual; terhubung ke [Life Planning](LIFE-PLANNING.md). |
| `UF-19` | `SCR-16` | Memilih versi dan menjalankan deterministic projection | Input kurang; run gagal; hasil lama; stale; hasil negatif sah. |
| `UF-20` | `SCR-14` | Membuat/fork scenario version immutable | Draft conflict; save error; original tidak berubah. |
| `UF-21` | `SCR-15` | Membandingkan dua versi sah dan memahami perbedaan | Currency/horizon incompatible; stale A/B; old async response. |
| `UF-22` | `SCR-16.V03` | Memahami forecast kas per periode | Shortfall; sumber mismatch; historical vs future label. |
| `UF-23` | `SCR-16.V04` | Menjalankan shock dan meninjau delta vs baseline | Shock incomplete; baseline berbeda; no absolute safety claim. |
| `UF-24` | `SCR-17.V01/V02` | Memahami insight + sumber + cara baca | Evidence missing; outdated insight; not investment advice. |
| `UF-25` | `SCR-17.V03` | Memilih notifikasi opt-in | OS permission ditolak; private preview. Bukan kalkulasi. |
| `UF-30` | `SCR-22` | Resource privat tidak dapat diakses | Deep link milik user lain tidak boleh bocor. |
| `UF-31` | `SCR-22` | Recovery setelah offline/timeout/pindah akun | Draft lokal bukan server truth. |

**Requirement desain:** gunakan prinsip Design System yang disetujui untuk warna, keterbacaan, accessibility, dan empty states. Nama subview dalam Screen Specs adalah referensi desain, bukan instruksi membuat route tanpa router decision.

## 6. Entry flow: membuat simulasi pertama

**Precondition konseptual:** pengguna sudah login, owner identity resolved di backend, memiliki data aktual/plan yang cukup untuk model terpilih atau dapat mengisi missing inputs, serta kebijakan baseline dan periode terdefinisi.

1. Pengguna membuka Rencana/Simulasi melalui jalur navigasi yang disetujui, lalu memilih `Buat skenario`.
2. Tampilkan dua sumber input yang jelas: posisi aktual `per tanggal ...` dan variabel hipotetis yang dapat diedit (`rencana`). Jangan menampilkan data aktual sebagai editable via scenario control.
3. Pengguna memilih horizon, scenario name, dan perubahan dibanding baseline; setiap asumsi penting punya *unit* dan *basis waktunya*.
4. Form memvalidasi input sebelum run; beri daftar yang spesifik bila tidak lengkap atau ambigu.
5. Save draft dan/atau publish immutable version mengikuti state model approved `Q-06`. Tidak ada endpoint yang boleh diasumsikan dari wireframe.
6. Pengguna menekan `Hitung proyeksi` sebagai aksi eksplisit; server menghitung lewat engine Go.
7. Flutter menampilkan hasil dengan provenance, grafik/table, batas aktual-proyeksi, data yang tidak diketahui, dan tombol untuk meninjau asumsi.
8. Pengguna dapat membuat versi alternatif dari skenario atau membandingkannya dengan versi lain yang kompatibel.

**Alternatif:** tidak ada actual baseline → tampilkan requirement data; deadline/horizon invalid → jangan kirim perhitungan; offline → draft lokal hanya jika aturan storage lokal disetujui; event menabrak recurrence → tandai kemungkinan double count dan minta pemilihan prioritas; user logout → hilangkan cached user-sensitive state.

**Postcondition:** hanya draft/version/run/result milik pengguna yang dapat berubah. `accounts`, posted ledger, liabilities, dan actual valuations tidak berubah. Jika input belum layak, hasil angka tidak boleh diciptakan untuk mengisi layar.

## 7. Kontrak input & validasi asumsi

| Kelompok input | Semantik minimal | Validasi / policy gate |
|---|---|---|
| Principal/owner | User internal yang telah diautentikasi | Server menentukan owner, bukan client field yang dipercaya. |
| Baseline | `as_of`, actual revision, currency set, included positions | `Q-06`; full snapshot vs immutable references belum disetujui. |
| Horizon | Start, end, time step dan calendar convention | `Q-05`, `CALC-Q10/11`; leap year/partial month. |
| Starting cash | Eligible cash positions dari baseline | Jangan menghitung aset noncash sebagai kas otomatis. |
| Income model | One-time, recurring, delta/replacement yang eksplisit | Tidak menggandakan salary event dengan recurring salary. |
| Expense model | Mandatory/optional, one-time/recurring, classification | Fee/interest/principal dipisahkan. |
| Debt model | Opening outstanding, payment schedule, accrual basis | `CS-010`; loan policy belum disetujui. |
| Goal contributions | Planned vs actual-attributed funding | `Q-11`; no double funding. |
| Life event | Typed effect, effective date, event source, multiplicity | `LP-Q02..08`; tidak mem-post actual. |
| Return & inflation | Rate kind, effective period, nominal/real base | `Q-05`; no hidden growth/"average return". |
| Shock | Income disruption magnitude/duration/start, expense shock | `CS-011`; baseline comparability. |
| Presentation | Chosen metrics, currency, time step, rounding for display | API money encoding and display precision `Q-04`. |

**Validation responses (konseptual):** `INVALID_INPUT`, `INVALID_HORIZON`, `AMBIGUOUS_RATE`, `INSUFFICIENT_DATA`, `CONFLICTING_EVENTS`, `MODEL_NOT_APPROVED`, `INCOMPATIBLE_SCENARIOS`, `RESOURCE_UNAVAILABLE`, `STALE_BASELINE`, `CALCULATION_FAILED`. Ini bukan enumerasi final JSON/API dan tidak boleh disalin menjadi enum implementasi sebelum OpenAPI approved.

**No silent defaults:** parameter yang bisa mengubah kesimpulan finansial secara bermakna harus diminta, diturunkan dari sumber yang sah/approved, atau dinyatakan sebagai **asumsi ilustratif eksplisit**. Tidak boleh menggunakan `0%` return, `0` inflation, no debt payment, atau eternal salary hanya agar UI bisa menghasilkan chart.

## 8. Planning draft, immutable scenario version, dan branching

### 8.1 Ownership model

`Scenario` adalah wadah milik user; `draft` berisi perubahan calon input; `version` mewakili snapshot input yang immutable menurut kebijakan yang disetujui; `run` menunjuk versi dan engine yang mengeksekusinya; `result` menunjuk run dan metadata. Setiap link harus diverifikasi owner secara transitif, bukan cuma level scenario.

### 8.2 Perilaku edit

- Edit label/deskripsi scenario tidak otomatis mengubah published version atau calculated result.
- Edit parameter pada published version harus menghasilkan versi baru/fork, bukan `UPDATE` payload immutable sebelumnya.
- Atribut yang disalin dari baseline harus bisa dibedakan dari override pengguna.
- Jika actual berubah dan user memilih *rebase*, tampilkan perubahan sumber/data dan akibatnya; buat versi/run baru sesuai `Q-06`.
- Delete/archive punya policy retention, export, in-flight runs, dan linked results yang belum final; tidak boleh memotong bukti audit diam-diam.
- Version numbering, generation, optimistic lock, dan conflict retry belum ditentukan (`DB-Q08`, `API-Q10`, `LP-Q13`).

### 8.3 Comparison of alternatives

Dua skenario contoh: A = "tetap dengan kondisi asumsi saat ini", B = "menambah biaya rumah 12 bulan lagi". **Nama bukan baseline identity**. Setiap versi harus menyatakan baseline dan perubahan konkret; user tidak boleh disesatkan bahwa B memakai data aktual terkini bila dibangun di revision lama.

Comparison hanya dihitung dari output engine valid dan berlabel sama, bukan mengambil angka kartu ringkasan secara bebas. Hindari edit-in-place saat compare; tombol `Ubah skenario B` membuka fork/editor yang jelas.

## 9. Deterministic projection output (R2, `FR-012`, `CS-009`)

**Output minimum konseptual:** periode/batas waktu; beginning dan ending cash; modeled external inflows/outflows; principal vs interest jika relevan; internal transfers; asset/liability positions bila eligible; net worth jika ada definisi sumber yang cukup; event contributors; per-period warning; status kelengkapan; provenance/assumptions; engine version.

Tidak semua metrik harus tersedia dalam semua run: jika utang dan valuasi tidak ada, tampilkan *limited to cash projection* atau status data kurang, bukan net-worth projection yang terkesan lengkap. Pastikan setiap agregasi sesuai `DR-023`.

### 9.1 Dekomposisi posisi dan flow

`Cash at end` tidak identik dengan `income minus expenses` karena ada transfer ke investasi, repayment principal, dan financing. Net worth bukan penjumlahan income saja; perubahan harga aset dan bunga dapat berkontribusi. Referensi `FTC-CASH-01` menunjukkan posisi kas Rp5 juta meski net external flow Rp2,9 juta—mencampur metrik tersebut menghasilkan dashboard salah.

### 9.2 Chart vs table

Chart menunjukkan tren; **tabel data numerik dengan unit, periode, sumber dan metode konversi** wajib tersedia untuk accessibility dan verifikasi. Grafik tidak boleh interpolate titik yang tidak dihitung menjadi data finansial "faktual". Warna untuk historical vs forecast harus dibedakan juga dengan pola/label, bukan warna semata. Tooltip berisi label tanggal dan sumber; bila nominal/real toggle belum disetujui, jangan buat toggle yang diam-diam mengonversi.

### 9.3 Result variants

- Valid result dan stable provenance.
- Valid result dengan saldo negatif: tampilkan shortfall, bukan generic server error.
- Partial/incomplete: tampilkan hanya metrik yang sah dengan penjelasan apa yang tidak bisa dihitung.
- Stale result: tetap bisa dibuka bila berizin, diberi label dan aksi eksplisit rebase/rerun.
- Error/no data: tidak membuat placeholder angka yang terlihat seperti hasil riil.

## 10. Scenario comparison (`FR-013`, `CS-012`)

Sebelum menampilkan delta A/B, periksa compatibility vector minimum:

| Dimensi | Contoh pertanyaan | Tindakan bila berbeda |
|---|---|---|
| Owner & authorization | Apakah kedua version/run milik pengguna aktif? | Deny safely, tanpa mengungkap existence pihak lain. |
| Baseline scope/revision | Apakah posisi awal dan as-of sebanding? | Tolak atau tampilkan comparison caveat yang telah disetujui. |
| Currency/FX | Apakah metrik dalam unit uang yang sama? | Block bila FX/normalization policy tidak disetujui. |
| Horizon/time step | Apakah bulan/tanggal akhir sebanding? | Block atau approved resampling, tidak diam-diam memotong. |
| Nominal vs real | Apakah basis harga dan inflation base sebanding? | Block atau explicit approved conversion. |
| Metric definition | Cash vs net worth; before vs after contribution | Jangan kurangi dua metrik yang berbeda. |
| Engine/model | Versi engine dan policy formula kompatibel? | Flag incompatibility; jangan asumsikan delta valid. |
| Completeness | Apakah hasil sukses vs parsial? | Beri status dan cakupan, tidak melakukan ranking definitif. |
| Staleness | Apakah salah satu baseline obsolete? | Label `Perlu diperbarui`, pertahankan hasil lama. |

Perbandingan yang kompatibel menampilkan setidaknya: scenario names + immutable version IDs; baseline date, horizon, currency, source, engine; assumption deltas; chosen metric A/B; numeric delta dengan satuan dan tanda; key events yang menyebabkan perbedaan; disclaimer. Pengguna dapat menukar sisi A/B tetapi **tidak** mengubah identitas versi.

**Tidak boleh:** "B lebih baik" berdasarkan final cash jika B menyembunyikan outstanding debt; menyamakan nominal 2030 dengan real 2026; menormalisasi secara otomatis tanpa kebijakan; menghapus stale warning karena chart tampak lebih rapi.

### 10.1 Contoh kompatibilitas

A: Rp dengan horizon 12 bulan, actual revision `R1`, result nominal. B: USD dengan horizon 24 bulan, actual revision `R2`, result real. **Expected UI:** `Belum bisa dibandingkan secara langsung` dan rincian dimensi yang tidak cocok. Jangan konversi USD, menginterpolasi 24→12 bulan, atau menimpa baseline tanpa persetujuan dan rekalkulasi eksplisit.

## 11. Future cash-flow forecasting (`FR-014`)

Tabel forecast per period secara konseptual harus membedakan:

| Kolom | Makna | Guard |
|---|---|---|
| `Starting cash` | Posisi cash model di awal periode | Bukan total net worth. |
| `External inflow` | Income/receipt aktual yang diproyeksikan secara eksplisit | Jangan masukkan transfer dari account sendiri sebagai income. |
| `External operating outflow` | Pengeluaran operasional + biaya yang sah | Jangan masukkan repayment principal sebagai consumption. |
| `Debt principal movement` | Penurunan kas dan liability jika dibayar | Tidak otomatis menjadi expense. |
| `Internal allocation/transfer` | Pergeseran antara posisi milik user | Konservasi total assets, jika tanpa fee/FX. |
| `Closing cash` | Posisi cash perkiraan | Bisa negatif dalam model; tandai. |
| `Threshold / shortfall` | Kekurangan relatif pada ambang yang dinyatakan | Ambang dan pemeriksaan period harus approved. |
| `Confidence` | Pada R2: **tidak ada statistical probability** | Jangan buat "90% chance" dari deterministic run. |

Sumber net inflow harus konsisten dengan time conventions `CS-009`. Forecast dapat menampilkan `income expense`, `cash position` dan `liquidity risk` **sebagai metrik berbeda**. Bila tidak ada data cicilan wajib, output `safe-to-spend` harus menjadi `INSUFFICIENT_DATA`, bukan angka berdasarkan cicilan nol (`FTC-CASH-08`).

**Contoh numerik referensi:** input `FTC-CASH-01`: cash opening Rp4m, income Rp6m, konsumsi Rp3m, interest Rp0,1m, principal repayment Rp0,9m, transfer internal ke investasi Rp1m. Closing cash = **Rp5m**, net external operating flow = **Rp2,9m**. Ini contoh spesifikasi dari golden tests, **bukan** formula umum cash-flow tanpa definisi model.

## 12. Parameterized financial stress test (`FR-015`, `CS-011`)

**Purpose:** tunjukkan perbedaan dua hasil berdasarkan *shock* yang dimasukkan pengguna, bukan "probability of bankruptcy".

### 12.1 Shock input

Tipe awal yang diusulkan untuk review: penurunan/berhentinya income, biaya darurat one-time, kenaikan biaya rutin, atau perubahan tingkat bunga **hanya jika model debt disetujui**. Semua shock memerlukan nominal/percentage yang jelas, effective date, duration, target cash flow, dan apakah berupa replacement atau delta terhadap baseline. Jika shock menyatakan income=0 selama lima bulan, hitungan baseline salary pada bulan yang sama **tidak boleh tetap masuk** tanpa penjelasan.

### 12.2 Pairing dan provenance

Bandingkan shock dan no-shock menggunakan baseline, horizon, source revisions, dan engine policy yang kompatibel. Variabel lain harus dikontrol/dilaporkan sehingga delta bisa ditelusuri. Perubahan baseline secara diam-diam membuat stress comparison tidak sah.

### 12.3 Contoh numerik ilustratif (sudah tersedia sebagai upstream tests)

Gunakan premis eksplisit `FTC-STRESS-01/02/03`: starting cash **Rp4.000.000**, horizon **5 bulan**, outflow wajib **Rp1.000.000 di akhir tiap bulan**, threshold **0 pada akhir bulan**, tanpa utang otomatis; baseline memiliki income **Rp1.000.000/bulan**; shock income menjadi **Rp0**.

| Bulan | Baseline closing cash | Shock closing cash | Delta shock vs baseline |
|---|---:|---:|---:|
| 1 | Rp4.000.000 | Rp3.000.000 | -Rp1.000.000 |
| 2 | Rp4.000.000 | Rp2.000.000 | -Rp2.000.000 |
| 3 | Rp4.000.000 | Rp1.000.000 | -Rp3.000.000 |
| 4 | Rp4.000.000 | Rp0 | -Rp4.000.000 |
| 5 | Rp4.000.000 | -Rp1.000.000 | -Rp5.000.000 |

**Interpretasi yang tepat:** dalam premis model ini, **shortfall pertama di akhir bulan kelima** dan empat periode sebelumnya sepenuhnya didanai. Ini **bukan** pernyataan aman empat bulan untuk kondisi keuangan nyata yang mungkin memiliki kewajiban belum tercatat. Jika period cutoff, liquidity threshold, dan debt policy berubah, hasil yang relevan harus dihitung ulang; no hardcoded defaults.

### 12.4 Safe copy

- Layak: `Pada asumsi ini, proyeksi kas turun di bawah Rp0 pada akhir bulan ke-5.`
- Layak: `Hasil dapat berubah jika kewajiban, pendapatan, atau biaya aktual berbeda.`
- Tidak layak: `Anda pasti bangkrut bulan depan.`
- Tidak layak: `Peluang gagal bayar 87%` bila tidak ada approved probabilistic model.

## 13. Explainable rule-based insights (`FR-016`, `CS-016`)

Setiap insight, bila ditampilkan, harus memiliki paket penjelasan:

1. **Trigger rule ID/version** yang telah disetujui, bukan teks generatif tanpa aturan.
2. **Source metrics** beserta `as_of`, input revision, dan perhitungan yang mendasari.
3. **Observation** bersifat bersyarat (misalnya cash shortfall dalam suatu skenario).
4. **Uncertainty and missing data** (misalnya cicilan tidak diketahui, return tidak dimodelkan).
5. **User option** yang informasional seperti `Tinjau asumsi` / `Buat skenario alternatif`, bukan transfer otomatis atau produk investasi tertentu.
6. **Stale state:** jika evidence berubah, insight lama diberi label usang/withdrawn, tidak terus tampil seolah current.

`FR-022` reminder dan consent/notification tidak diimplementasikan sebagai bagian dari insight engine tanpa approved `Q-09`. Tidak ada data sensitif pada push preview atau lock screen secara default. **AI generatif bukan dependensi yang diwajibkan oleh FR-016.**

## 14. Metadata hasil dan reproducibility

Setiap run yang dapat diinspeksi kembali harus memungkinkan reviewer menjawab pertanyaan berikut tanpa membuka data pengguna lain:

- **Siapa owner** dan apakah principal aktif berhak membaca run ini? (Owner internal saja di server; jangan bocor PII.)
- **Dari versi mana?** Baseline identity atau revision, scenario version, input fingerprint.
- **Kapan kondisi awal?** Effective `as_of`, calendar/timezone convention, data revision.
- **Bagaimana dihitung?** Engine version, model configuration, rate/rounding/time-step policy versions.
- **Apa outputnya?** Unit, currency, horizon, results completeness, warning/status, `calculated_at`.
- **Apakah ini masih current?** Stale relation ke revision aktual terkini + alasan.
- **Apakah ada stochastic model?** `seed`, sample count, model version **hanya R3** dan hanya setelah statistical approval.

**Reproducibility invariant:** `same baseline snapshot + same immutable scenario input + same engine version + same approved policy` menghasilkan output yang ekuivalen di batas precision resmi. Perubahan library/engine/rounding adalah potensi perubahan meaning dan dapat membutuhkan versi baru serta regression tests, bukan silent rewrite historical results.

**Data minimization:** tidak perlu menduplikasi seluruh ledger privat pada setiap result record jika cukup memakai snapshot ref yang aman. Namun ref harus benar-benar immutable/tersedia untuk replay menurut approved retention policy. Pilihan full snapshot vs reference masih `Q-06/DB-Q08`.

## 15. Staleness, rebase, dan version conflicts

### 15.1 Sumber stale

- Ada actual transaction baru/koreksi, account change, asset valuation update atau liability update setelah baseline diambil.
- Input plan, goal, life event, recurring rule, rate assumptions, atau scenario draft berubah dibanding versi yang dipakai run.
- Engine version atau policy berubah sehingga hasil tidak sebanding.

**Tidak semua perubahan otomatis membuat semua run stale:** policy perlu menentukan dependency-aware invalidation. Yang wajib adalah **tidak menyajikan hasil yang tak lagi representatif sebagai current tanpa label**. Jika exact revision semantics belum disetujui, gunakan blocker, bukan heuristik parsial tanpa review.

### 15.2 Rebase adalah aksi pengguna yang eksplisit

Arah UX: `Data keuangan Anda berubah setelah simulasi ini. Lihat perubahan dan buat versi perhitungan baru.` Meskipun rebase tersedia, versi/run lama tetap dapat diinspeksi sesuai retention policy; tidak boleh memodifikasi recorded result secara retroaktif.

### 15.3 Konflik multi-device

Save draft dari perangkat A vs B harus menggunakan optimistic concurrency token atau mekanisme setara setelah API design disetujui. Konflik harus diberi penjelasan dan pilihan meninjau perubahan; tidak boleh otomatis last-write-wins bila itu menghapus perubahan pihak pengguna tanpa tahu.

### 15.4 Offline state

Bila Flutter menampilkan cache offline, ia wajib menandai *cached at*, owner/session, versi dan **bukan current source of truth**. Pending local edits tidak boleh mengubah server-side actual. Clear sensitive state saat sign out/account switch. Offline edit queue baru boleh dibangun setelah conflict, encryption, session, dan replay policies disetujui.

## 16. State machine pengalaman dan error taxonomy

State berikut **konseptual**, bukan enum final yang harus disalin ke source code:

```text
DRAFT -> VALIDATING -> READY_TO_RUN -> RUN_REQUESTED
             |             |              |
             +-> NEEDS_INPUT             +-> RUNNING / QUEUED (only if async approved)
                                               |
                              +----------------+------------------+
                              |                |                  |
                           SUCCEEDED         FAILED          CANCELLED
                              |
                    RESULT_CURRENT / RESULT_STALE
```

Untuk deterministic R2, **run synchronous mungkin cukup**; async queue state dan `202 Accepted` **hanya** dipakai jika backend benar-benar memiliki durable run/job contract approved. Jangan memperkenalkan background worker hanya untuk menyalin mock API R3. `CANCELLED` tidak sama dengan request yang timeout sementara server tetap menghitung; client perlu memeriksa status sebelum mengklaim pembatalan berhasil.

| State/event | UX pesan / tindakan | Tidak boleh |
|---|---|---|
| `NEEDS_INPUT` | Tampilkan input spesifik yang kurang, tidak jalankan run | Isi asumsi diam-diam. |
| `VALIDATION_ERROR` | Tandai field beserta unit | Tampilkan grafik dummy. |
| `MODEL_NOT_APPROVED` | Jelaskan fitur masih belum tersedia/menunggu keputusan | Anggap parameter alternatif otomatis sah. |
| `PROCESSING` | Loading dengan context scenario/version | Mengklaim sudah tersimpan atau selesai tanpa server evidence. |
| `RESULT_CURRENT` | Chart, table, assumptions, limitations | Menyatakan outcome terjamin. |
| `RESULT_STALE` | Badge `Perlu diperbarui`; rebase/rerun explicit | Menimpa historical result. |
| `INCOMPATIBLE_PAIR` | Jelaskan beda currency/horizon/basis/engine | Delta angka asal. |
| `SHORTFALL` | Tampilkan negatif pada periode relevan | Tandai sebagai server error atau real debit. |
| `RUN_FAILED` | Error yang aman, retry bila semantik jelas | Membuka raw server stack trace. |
| `RUN_STATUS_UNKNOWN` | Reconcile run identity/status sebelum retry | Membuat run ganda atau menyatakan gagal padahal commit terjadi. |
| `AUTH_DENIED` | Pesan privat aman + session recovery | Membocorkan nama/saldo skenario user lain. |
| `OFFLINE` | Cached draft/result labeled, no pretend freshness | Menyimpan PII sensitif tanpa kontrol. |

## 17. API boundary: kontrak semantik, bukan endpoint final

**Tidak ada endpoint final** sampai `contracts/openapi.yaml` atau sumber kontrak setara diperiksa dan disetujui. Tabel ini menjelaskan *operasi yang dibutuhkan*:

| Operasi konseptual | Input valid | Server output/guard yang wajib |
|---|---|---|
| List own scenarios | Authenticated principal, pagination | Hanya own scenarios, no cross-owner metadata. |
| Save draft | Owner + expected revision + typed deltas | Validation, conflict state, no actual mutation. |
| Publish scenario version | Valid draft + baseline ref + version control | Immutable version identity, no overwrite. |
| Request deterministic run | Authorized immutable version + approved model inputs | Run identity/result/status with provenance. |
| Read own run & result | Owner-verified run + parent version | Result and stale metadata, no leaking inputs. |
| Compare two runs | Two authorized versions, compatibility dimensions | Typed comparable result or explicit incompatible reasons. |
| Rebase | Authorized scenario + explicit fresh baseline selection | New version, historical version preserved. |
| Archive/delete | Strong owner auth, lifecycle/references handled | No orphaned private data and no unexpected actual writes. |

**API rules:** consistent error semantics; typed money strings/units after `Q-04`; body and status versioning; safe retry identity; bounded input/query limits; authorization on nested resource IDs; no user-provided `owner_id` as security basis. Mock examples in API Guidelines are illustrations, **not** guarantee that the route actually exists.

**Timeout ambiguity:** saat save/run request mendapat timeout, server mungkin sudah berhasil. Client harus memakai approved idempotency/status lookup sebelum retry write. For `202 Accepted`, acknowledge only after work durably recorded; `Retry-After` sample in API Guidelines bukan cadence final.

## 18. Go orchestration & pure financial engine boundary

Rancangan yang dituju: Flutter mengurus UI/request dan meng-render typed result; **Go** memverifikasi identity/authorization, mengambil baseline berizin, memvalidasi assumptions, memanggil **pure calculation engine**, dan menyimpan version/run/result sesuai persistence policy. Tidak boleh ada formula resmi duplikat di widget Flutter atau handler HTTP.

`ScenarioService` atau nama serupa adalah **konsep**, bukan instruksi membuat package tertentu. Dependency direction yang disarankan untuk review:

```text
Flutter UI (inputs + result display)
   -> REST boundary (auth + shape validation)
   -> Go orchestration/use case (owner authz, baseline, versions, run lifecycle)
   -> pure financial engine (deterministic numeric state; no I/O / actual writes)
   -> persistence adapter (scenario & result only)

Actual account/ledger reads -> approved baseline assembly -> engine input
Actual account/ledger WRITES <- never from simulation use case
```

**Hard boundary:** engine accepts immutable value inputs and returns deterministic values; it must not know logged-in user ID, HTTP request, DB transaction, Flutter state, or storage path. Penanganan budget/recurring event, life-event ordering, loan and cash policies **didefinisikan di `SCENARIO-ENGINE.md` nanti**, mengikuti `CS-009/010/011/012` dan keputusan owner; jangan putuskan algoritma akhir di file UI ini.

**Execution cost:** bounded horizon, input event count, memory/time budget, cancellation handling, and result size; detail thresholds `TBD`. Long-running R3 worker tidak otomatis perlu untuk deterministic R2. Update engine version and regenerate golden tests on any meaning-affecting model changes.

## 19. PostgreSQL persistence & data lifecycle boundary

`scenario_baselines`, `scenarios`, `scenario_versions`, `simulation_runs`, `simulation_results`, dan `actual_revisions` adalah **candidate entities**, bukan migration final. Keputusan storage dan encoding menunggu DB approval. Garis pemisah akses dan lifecycle:

- Scenario dan version harus dimiliki user, nested foreign keys tidak boleh lintas owner; repository queries owner-scoped.
- Published version/inputs dan stored results immutable sesuai policy; fields semacam `stale` boleh dihitung atau disimpan melalui separate metadata, tetapi historical output tidak diganti.
- Actual data revisions tercatat secara konsisten dan terhubung ke baseline yang digunakan; strategy full snapshot vs reference masih TBD.
- Simulations tidak boleh membuka transaksi database yang melakukan actual balance write; write-set harus dapat diuji.
- Permintaan deletion, logout, user deletion dan active job/race memerlukan policy yang menutup akses setelah revoke; retention backup dan export tidak boleh dikarang.
- DB constraints, index, transaction isolation, lock, optimistic versions serta idempotency keys memerlukan migration review dan integration tests terpisah.
- Jangan menyimpan raw salary/debt/transactions dalam umum log/audit akibat exception atau debug mode.

### 19.1 Status run dan asset isolation

Jika asinkron pada tahap yang disetujui, satu run harus punya immutable input fingerprint, owner, lifecycle status, updated timestamps, dan output integrity. Worker retry boleh terjadi secara teknis; **logically** tidak boleh menulis dua hasil yang saling bertentangan untuk run ID sama. `Cancel` setelah commit harus mengakui status sebenarnya, bukan menutupi result yang sudah final.

## 20. Flutter UI architecture & interaction rules

- Setiap halaman memakai state yang eksplisit: initial, input editing, validation, loading, loaded, stale, empty, offline, error, permission denied, status unknown.
- Gunakan komponen reusable untuk money/units, as-of badge, scenario version header, assumption list, warning copy, chart/table, and safe retry; nama komponen **kandidat** dan library choice TBD.
- Navigation deep links memeriksa session dan owner server-side; session switch harus invalidate state sensitif/response lama.
- Debounce input hanya untuk preview yang aman; run resmi dimulai lewat aksi eksplisit jika required. No chart re-render with phantom success while request pending.
- Async search/sim requests harus dipasangkan dengan input revision dan request generation agar respons lama tidak mengalahkan pilihan baru.
- Loading chart skeleton tidak menggunakan angka seolah data aktual; empty states memberi langkah perbaikan yang konkret.
- Tombol publish/rebase/archive/destructive action menunjukkan confirmation dan konflik yang jelas; tidak melakukan side effect tersembunyi.
- Format angka dan rounding hanya pada presentation layer setelah policy; sumber nilai resmi tetap backend Go, dan jangan memaksa parse nominal dengan `double`.
- Table fallback, screen reader labels, zoom/dynamic type, chart contrast, locale dan long-number handling wajib di-review terhadap [Design System](../design/DESIGN-SYSTEM.md).
- Chart unit date axis harus jelas; jangan sekadar line chart warna dua dengan legenda tak terbaca.
- Tidak ada upsell, false urgency, klaim `Guaranteed`, atau metric pseudo-confidence untuk membuat UI tampak canggih.

## 21. Security, privacy, dan cross-user test obligations

| Threat surface | Kebutuhan dan bukti |
|---|---|
| IDOR scenario/version/run | Backend owner authorization untuk setiap nested fetch/edit/comparison; test token user A terhadap resource B. |
| Leaky 404/403 metadata | Response aman, tidak mengungkap nama scenario, angka, atau keberadaan secara beda yang bisa dienumerasi. |
| Client impersonation | Never trust `owner_id` dari Flutter; server maps validated principal. |
| Replay / duplicate create | Stable idempotency/concurrency scheme saat approved; test timeout-after-commit. |
| Parameter bombs | Bounds untuk event count, horizon, return, recursion, sample counts, arrays dan response size. |
| Logs/analytics | Redact salary, debt, asset balances, free-form scenario titles; controlled identifiers only. |
| Offline cache | Encrypted or minimized following approved local storage policy; purge on logout/account switch. |
| Share/export/deletion | Explicit permissions, expiry, scope, retention, deletion of linked runs and queue race controls. |
| Notification/insights | Consent if notifications, no sensitive lockscreen payload; results never push automatically. |
| Retention & audit | No unlimited private result history default until policy review; audit appropriate to risk. |

Gunakan security controls `SEC-001`–`SEC-026` dan security tests `ST-001`–`ST-075` dari [Security & Privacy](../security/SECURITY-PRIVACY.md); local tests `SIMT-*` menambah, bukan menggantikan, suite tersebut.

## 22. Observability, performance, availability, dan reliability

**Telemetry yang boleh dipertimbangkan setelah privacy review:** jumlah run per status, durasi per horizon bucket, rasio `INSUFFICIENT_DATA`, error category, stale detection count, comparison-incompatibility reasons, size/performance budget, retry rates. Hindari raw scenario values, free-form labels, user target/salary/debt, token, exact cash atau private engine inputs dalam telemetry umum.

**Performance** `NFR-*`: latensi target, batas horizon/events, caching, pagination, mobile frame quality, rate limits, timeout, async switching, dan availability **harus diambil dari NFR resmi dan dikonfirmasi**, tidak boleh menebak 200 ms, 99.9%, max 100 event, atau polling tiap 3 detik sebagai kebijakan. R3 worker capacity berbeda dari synchronous R2.

**Resilience:** request failed bisa diulang hanya jika aman; server failure tidak menghapus draft yang memang tersimpan; no two-result confusion; session expiry dilayani tanpa reusing prior user's cache; backend restart/DB reconnect menjaga version/run status sesuai policy; execution error muncul sebagai pesan aman, bukan angka yang dikarang.

**Monitoring & incident handling:** jika penyimpangan angka/reference test terjadi, hentikan rollout terkait; dokumentasikan engine/config/baseline/version dan reproduction minimal sintetis; jangan meminta pengguna memposting data keuangan sensitif pada log umum atau tiket publik.

## 23. Local acceptance test matrix (`SIMT-001`–`SIMT-048`)

**Aturan ID:** `SIMT-*` berlaku lokal untuk dokumen ini dan hanya menetapkan *test obligations*. Jangan memakai ID tersebut untuk mengganti `FTC-*`, `TS-*`, `ST-*`, `LPT-*`, atau `UXT-*`; implementasi harus menautkan bukti aktual ke ID yang relevan. `EXPECTED` di bawah adalah test oracle konseptual, bukan hasil pengujian yang sudah dijalankan. Status awal semua executable tests adalah **`NOT_RUN`**; item yang membutuhkan policy/model mendapat readiness **`BLOCKED/CONDITIONAL`** sebelum pelaksanaan.

### 23.1 Actual/plan separation dan authorization

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-001` | Go + DB integration | Save draft yang memuat future expense Rp500.000 | Posted actual ledger/account balance tak berubah | `FTC-SCEN-02`, `DR-002`. |
| `SIMT-002` | Go + DB integration | Publish new version untuk event berulang | Tidak menambahkan actual future transactions | `DR-012/013`, `LPT-001`. |
| `SIMT-003` | Go + DB integration | Run deterministic projection 2x | Actual revisions dan balances sama seperti sebelum kedua run | `FTC-CASH-06`, `DR-002`. |
| `SIMT-004` | Go + DB integration | Stress shock mengurangi salary model | Tidak mengubah posted income di backend | `FTC-STRESS-01`, `DR-001/002`. |
| `SIMT-005` | Engine | Planned goal contribution Rp500k; tanpa actual movement | Total assets tidak bertambah hanya karena label alokasi | `FTC-CASH-05`, `DR-011`. |
| `SIMT-006` | DB/API security | User B meminta run milik user A | Tidak mendapat input/result/nama/provenance privat | `FTC-SCEN-08`, `DR-003`. |
| `SIMT-007` | DB/API security | User B menebak version ID milik user A di nested scenario | Unauthorized resource response yang aman, tanpa cross-owner read | `SEC-*`, `DR-003`. |
| `SIMT-008` | Go + DB integration | Archive scenario dan lakukan logout | Data aktual tetap; hasil user lama tak tampil dari cache user baru | `DR-002/021`, `UF-30/31`. |

### 23.2 Input, versions, staleness, and orchestration

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-009` | Go domain | Valid scenario, income wajib tidak diketahui | `INSUFFICIENT_DATA`; tidak menyisipkan income=0 | `FTC-STRESS-05`, `DR-017`. |
| `SIMT-010` | Go domain | Rate tahunan tanpa type/frequency jelas | Reject/needs input; tidak mengasumsikan effective period otomatis | `CS-001/002`, `Q-05`. |
| `SIMT-011` | Go domain | Life event di luar horizon | Event tidak masuk period result | `FTC-CASH-09`, `LP-Q03/04`. |
| `SIMT-012` | Go domain | Recurring event overlap dengan budget/expense yang sama | Identifikasi kebutuhan deduplication/relationship; no silent double posting/model count | `LP-Q07`, `DR-013`. |
| `SIMT-013` | DB integration | User mengedit published version 1 dengan income baru | Version 1/result lama immutable; version 2 baru | `FTC-SCEN-03`, `Q-06`. |
| `SIMT-014` | DB integration | Actual baseline B1 berubah ke B2 setelah run | Old run masih merujuk B1; stale/rebase menurut approved policy | `FTC-SCEN-04/07`, `DR-020`. |
| `SIMT-015` | API integration | Duplicate publish request setelah response timeout | Tidak menciptakan versi ganda di bawah idempotency policy yang disetujui | API-Q / DB-Q, `SIM-I17`. |
| `SIMT-016` | Flutter integration | User mengubah parameter lalu response lama tiba terakhir | Output lama diabaikan; state terikat ke latest request/version | `UF-19/21`, `SIM-I16`. |

### 23.3 Deterministic projection & cash flow

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-017` | Engine golden | B1+S1+E1 dirun dua kali | Output matematis dan fingerprint ekuivalen sesuai precision resmi | `FTC-SCEN-01`, `CS-012`. |
| `SIMT-018` | Engine golden | `FTC-CASH-01` | Closing cash Rp5m menurut premis, bukan net external flow Rp2,9m | `FTC-CASH-01/02`. |
| `SIMT-019` | Engine invariant | Transfer internal Rp500k antar cash accounts tanpa fee | Total cash dan NW sama; income/expense eksternal 0 | `FTC-CASH-03`, `DR-006`. |
| `SIMT-020` | Engine invariant | Cash Rp1m, consumption Rp2m, income0, no borrowing | Forecast -Rp1m / shortfall, bukan dipaksa 0 | `FTC-CASH-04`. |
| `SIMT-021` | Engine invariant | Pembelian asset hipotetis Rp3m dari cash, fee0, valuation Rp3m | Cash -3m, asset +3m; NW delta0 pada event | `FTC-CASH-07`, policy conditional. |
| `SIMT-022` | Engine/UX | Tidak ada data cicilan wajib, diminta safe-to-spend | `INSUFFICIENT_DATA`, bukan klaim safe spending positif | `FTC-CASH-08`, `DR-017`. |
| `SIMT-023` | Flutter widget | Tampilkan satu actual history dan satu future projection | Keduanya punya label source dan boundary waktu/legend | `SCR-16`, `DR-014`. |
| `SIMT-024` | Flutter accessibility | Chart disajikan ke screen reader dan user butuh angka per periode | Tabel tersedia, currency, date, metric definition terbaca | `SCR-16`, Design System. |

### 23.4 Comparison, compatibility, and stale UX

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-025` | Engine/API | A=IDR dan B=USD tanpa FX policy approved | `INCOMPATIBLE_SCENARIOS`; no numeric ranking | `FTC-SCEN-05`. |
| `SIMT-026` | Engine/API | A=real basis 2026 vs B=nominal 2030 | Tolak atau require explicit approved normalization | `FTC-SCEN-06`. |
| `SIMT-027` | Engine/API | A horizon 12 months, B horizon 24 months | Jelaskan mismatch, jangan silently slice/resample | `UF-21`, `SCR-15`. |
| `SIMT-028` | Engine/API | A cash balance vs B net worth | Tidak menghasilkan numeric delta as if same metric | `DR-023`, `SCR-15`. |
| `SIMT-029` | Engine/API | A dan B different immutable engine-policy versions | Tampilkan compatibility assessment, no hidden assumption | `FTC-SCEN-03`, `CS-012`. |
| `SIMT-030` | Flutter/widget | Run A stale dan B current | Badge stale hanya pada yang tepat; old result masih inspectable | `DR-020`, `SCR-15`. |
| `SIMT-031` | Flutter/widget | Tukar posisi scenario A/B | Nilai delta sign/basis konsisten, version IDs tak berubah | `UF-21`, `SIM-I10`. |
| `SIMT-032` | DB integration | User rebase B1 ke B2 | Membuat hasil/version baru; historical B1 result tidak diubah | `Q-06`, `FTC-SCEN-04`. |

### 23.5 Stress test, insights, and information integrity

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-033` | Engine golden | Baseline cash 4m, income 1m/mo, expenses 1m/mo, 5 months | Closing cash tetap 4m tiap bulan | `FTC-STRESS-02`. |
| `SIMT-034` | Engine golden | Shock income 0, expenses 1m/mo, starting 4m, threshold 0 end-month | Closing 3m,2m,1m,0,-1m; shortfall pertama bulan 5 | `FTC-STRESS-01` (conditional timing). |
| `SIMT-035` | Engine golden | Dua run di `SIMT-033/034` dibandingkan akhir bulan 5 | Delta shock vs baseline = -Rp5m | `FTC-STRESS-03`. |
| `SIMT-036` | Engine/model | Shortfall detector diberi timing awal vs akhir bulan berbeda | Tolak mixing definitions tanpa approved policy | `FTC-STRESS-04`. |
| `SIMT-037` | Flutter UX | Forecast cash -Rp1m pada bulan 5 | Tampilkan asumsi/shortfall; tidak menyebut "pasti bangkrut" | `FTC-STRESS-07`, `FR-015`. |
| `SIMT-038` | Flutter UX | Stress result deterministic, tidak ada distribution model | Tidak ada "87% chance of failure" atau percentile sintetis | `DR-015`, `SIM-I12`. |
| `SIMT-039` | Insights | Rule approved dipicu oleh shortfall berdasarkan run current | Penjelasan punya rule version, input evidence, as-of, limitations | `FR-016`, `DR-019`. |
| `SIMT-040` | Insights | Data minimum rule hilang atau run stale | Insight withheld/marked insufficient/stale; no specific advice | `DR-017/019/020`. |

### 23.6 Failure, privacy, and operational evidence

| ID | Level | Given / When | Expected / invariant | Basis |
|---|---|---|---|---|
| `SIMT-041` | API security | Invalid nested run IDs / ownership denied | No leaks in status/body/errors/timing-sensitive metadata where mitigated | `SEC-*`, `DR-003/021`. |
| `SIMT-042` | Observability | Run with synthetic salary/debt, then inspect logs/telemetry | No raw sensitive values, access token, or freeform title | `DR-021`, Security & Privacy. |
| `SIMT-043` | Flutter integration | Sign out A, sign in B while A run response arrives | A response/cache ignored and purged; B isolated | `UF-30/31`, `SIM-I16`. |
| `SIMT-044` | API integration | Save/run times out after server commit | Client reconciles existing logical action; no duplicate version/run | `SIM-I17`, backend retry policy. |
| `SIMT-045` | API integration | Server returns `failed` vs valid negative forecast | Distinct states; negative cash does not become calculation failure | `SIM-I09/18`. |
| `SIMT-046` | Flutter accessibility | Large fonts, long IDR values, color-blind comparison | Labels/tables readable, focus order correct, no color-only encoding | Design System, `UXT-*`. |
| `SIMT-047` | API/worker | Cancel vs success race (if async exists) | Accurate final durable status; no phantom cancelled result | API Guidelines §15; conditional R3. |
| `SIMT-048` | Release E2E | Run scenario A/B + stress with synthetic owned data | Traceable input→result; actual unchanged; security/QA proof attached | `FR-012..016`, `TS-*`, `DOD-G*`. |

**Readiness nuance:** `SIMT-001..048` are all **specified**, not executed. Items requiring `Q-04/05/06/11`, approved `SCENARIO-ENGINE.md`, DB migrations, real API contract, or real Flutter widget implementations are `BLOCKED` or `CONDITIONAL` until those prerequisites exist. Run tests only against synthetic fixtures; no production account/transaction contents in artifacts.

## 24. Local decision register (`SIM-Q01`–`SIM-Q18`)

`SIM-Q*` bukan 18 keputusan baru yang sudah disetujui dan bukan pengganti **11 parent decisions** dalam [Decision Log](../planning/DECISION-LOG.md). Register ini menguraikan *questions needed* untuk pengalaman simulasi. Ketika owner menyetujui pilihan, catat keputusan induk, evidence, tanggal/aktor approval, lalu propagasikan ke `CS-*`, API, DB, Screen Specs, Test Strategy, dan task terkait.

| ID | Parent | Keputusan yang dibutuhkan | Dampak bila belum disetujui | Awal |
|---|---|---|---|---|
| `SIM-Q01` | `Q-06` | Baseline materialization: snapshot penuh, immutable refs atau hybrid? | Block production baseline persistence/replay | `BLOCKED` |
| `SIM-Q02` | `Q-06` | Revision/stale tracking: apa saja yang membuat invalidation dan bagaimana dependency-aware? | Block final staleness semantics | `BLOCKED` |
| `SIM-Q03` | `Q-06` | Draft → publish version, immutability, fork, rename/archive, retention? | Block durable versioning policy | `BLOCKED` |
| `SIM-Q04` | `Q-05/Q-06` | Engine event ordering and period boundaries (start/end, transfers, salary, event)? | Block exact projection result policy | `BLOCKED` |
| `SIM-Q05` | `Q-04` | Money encoding/rounding and cross-currency comparison strategy? | Block exact API schema and numeric comparisons | `BLOCKED` |
| `SIM-Q06` | `Q-05` | Effective annual/period rate, inflation and nominal/real basis rules? | Block unsupported growth/real-price claims | `BLOCKED` |
| `SIM-Q07` | `Q-05` | Horizon, timezone, month-end, leap day, partial-period conversion? | Block reproducible calendar model | `BLOCKED` |
| `SIM-Q08` | `Q-05/Q-06` | Income replacement vs delta and overlapping recurrence deduplication? | Block potentially double-counted projections | `BLOCKED` |
| `SIM-Q09` | `Q-11` | Goal actual-funded attribution and simulated contribution semantics? | Block funded-percentage/safe allocation claims | `BLOCKED` |
| `SIM-Q10` | `Q-05` | Supported debt/loan, KPR interest/amortization, fees, projected valuation? | Block full asset+debt house scenarios | `BLOCKED` |
| `SIM-Q11` | `Q-05` | Stress model shock catalog, threshold/check timing, runway definition? | Block production stress labels | `BLOCKED` |
| `SIM-Q12` | `Q-06` | A/B compatible metrics, approved normalization and stale comparison policy? | Block misleading scenario ranking | `BLOCKED` |
| `SIM-Q13` | `Q-06` | Run lifecycle synchronous vs async, idempotent retry, timeout reconciliation? | Block durable run API orchestration | `TBD` |
| `SIM-Q14` | `Q-06/Q-10` | Scenario/run result retention, export/deletion and active-run races? | Block persistent lifecycle release | `BLOCKED` |
| `SIM-Q15` | `Q-02/Q-10` | Session/cache/offline encryption and cross-user isolation on device? | Block offline sensitive drafts & cache policies | `TBD` |
| `SIM-Q16` | `Q-10` | Bounded horizon/event count, latency/budget, result quotas and abuse limits? | Block unbounded computation and rollout | `TBD` |
| `SIM-Q17` | `Q-01/Q-06` | Approved explanatory insight rules, data eligibility and UI wording? | Block derived personalized insight cards | `BLOCKED` |
| `SIM-Q18` | `Q-08/Q-10` | R3 stochastic results/optimizer integration, worker and risk disclosures? | Block R3 integration; does **not** block isolated R2 prototypes | `TBD` |

**Stop rule:** Jika suatu task membutuhkan pilihan di atas, boleh membuat **isolated prototype dengan fixture sintetis** apabila disetujui sebagai eksplorasi dan tanpa label production; jangan menambah dependency, endpoint, DB migration, atau legal/security commitments seolah keputusan sudah final. Dokumentasikan rekomendasi, alternatif, risiko, dan tradeoffs dalam Decision Log, kemudian tunggu approval owner.

## 25. Implementasi bertahap per epic (proposal, bukan permission)

| Tahap | Epic | Output kecil yang dapat diverifikasi | Prerequisite review | Acceptance exit evidence |
|---|---|---|---|---|
| S-07 baseline inputs | `EP-S07-01` | Approved budget/recurring semantics dan future occurrence preview | `Q-03/04/05`, recurring policy | `FTC-BUD-*` dan posting safety. |
| S-07 life event | `EP-S07-02` | Draft hypotheticals with clear source and date | `Q-05/06`, [Life Planning](LIFE-PLANNING.md) | `LPT-*`; actual unaffected. |
| S-08 A | `EP-S08-01` | Pure deterministic engine + period-level reference fixtures | `SCENARIO-ENGINE.md` approved, `Q-04/05/06` | `FTC-CASH-*`, `SIMT-017..024`, no actual writes. |
| S-08 B | `EP-S08-02` | Immutable scenario versions, two-result comparable UX | Approved baseline/revision/compatibility | `FTC-SCEN-*`, `SIMT-013..016/025..032`. |
| S-08 C | `EP-S08-03` | Forecast cash chart + table + shortfall labels | Validated period output | `SIMT-018..024`, `UF-22`, `SCR-16`. |
| S-09 A | `EP-S09-01` | Stress shock A/B and safe explanations | Approved threshold/shock, pair semantics | `FTC-STRESS-*`, `SIMT-033..038`. |
| S-09 B | `EP-S09-02` | Explainable rules and evidence links | Approved rule catalog and insufficient-data policy | `SIMT-039/040`, `UF-24`, `SCR-17`. |
| S-09 gate | `EP-S09-03` | Consent-first reminders if approved; full R2 regression/release review | `Q-09/10`, security/privacy release gates | Security `ST-*`, E2E, R1 regressions. |

**Implementation sequencing for AI agent:** inspect actual code/repo first; identify approved decisions; break epic into a task with a bounded file list; implement backend contract + tests before spending time on charts; validate integration with real auth and DB; then UI states; finally update `TASKS.md`/`PROGRESS.md` with executable evidence. No production R2 coding by simply reading a roadmap `EP-*` row.

## 26. Definition of Ready dan Definition of Done

### 26.1 DoR: before coding a simulation task

- [ ] Epic decomposed into task with owner, acceptance criteria and expected file changes.
- [ ] Relevant release `R2/S-08` or `R2/S-09` authorized; no unrelated R3 scope creep.
- [ ] `Q-04/05/06`, optional `Q-11`, and targeted local `SIM-Q*` resolved or strictly scoped away.
- [ ] Approved deterministic engine spec/golden oracles available; model `CS-009/011/012` policy known.
- [ ] Approved API contract and reviewed DB design/migrations for touched persistence.
- [ ] Auth/owner scopes, money representation, calendar semantics, data retention and failure modes known.
- [ ] Design flow/screen states traceable to `UF-*`, `SCR-*`, `UXT-*`, `ST-*`.
- [ ] Test harness for Go + PostgreSQL + Flutter available or a blocker documented.

### 26.2 DoD: only after real implementation and verification

- [ ] All selected task acceptance criteria met with exact code/commit reference.
- [ ] Pure calculation engine has golden and invariant test results; no duplication in Flutter.
- [ ] PostgreSQL integration tests demonstrate actual non-mutation, ownership, version immutability and retry safety.
- [ ] REST/OpenAPI contract tests match real source, including errors/staleness.
- [ ] Flutter unit/widget/integration tests cover loading, shortfall, stale, comparison incompatibility, offline, and error.
- [ ] UI consistently labels historical vs projected, time horizon, units and assumptions.
- [ ] E2E scenario create→run→compare→stress uses synthetic data and passes; no user PII in logs.
- [ ] Security `ST-*`, financial `FTC-*`, local `SIMT-*`, and regression scopes have test evidence.
- [ ] Relevant performance/resource/retention/backup gates validated against approved NFR and security policies.
- [ ] Results documented with command, environment, revision, actual PASS/FAIL/NOT_RUN/SKIPPED/BLOCKED and reproducible artifacts.
- [ ] `PROGRESS.md` status and `DECISION-LOG.md` updated without fabricated approvals.
- [ ] Release gate R2 reviewed separately; one task passing never makes all of R2 `DONE`.

### 26.3 Handoff template (copy for actual task execution)

```markdown
#### Task: <canonical TASK-ID>
- Scope/release: R2 / S-08 or S-09
- Baseline: git revision <verified> / DB migration revision <verified>
- Approvals: PRD Q-.. / SIM-Q.., links to actual Decision Log records
- Changed files: <exact paths>
- Traceability: FR-... / DR-... / CS-... / UF-... / SCR-...
- Tests executed:
  - ID: SIMT-... / FTC-... / ST-...
  - Command: <exact, genuinely executed command>
  - Environment: <actual details, sanitized>
  - Result: PASS | FAIL | NOT_RUN | SKIPPED | BLOCKED
  - Evidence: <log/artifact/commit reference>
- Known gaps: <unresolved decisions, flaky tests, incomplete functionality>
- Actual mutation guard: verified | not verified, with test reference
- New status: IN_PROGRESS | PARTIAL | BLOCKED | IN_REVIEW | DONE
- Next safe action: <one narrow next step>
```

Do not publish illustrative example outputs or tool commands as executed evidence. **Documentation-only quality check can be `DONE` as an artifact, while application implementation remains `UNVERIFIED` or `NOT_RUN`.**

## 27. Internal review, traceability, dan dokumentasi yang masih harus dibuat

| Check | Review target | State for this draft |
|---|---|---|
| R2 ownership | `FR-012/013/014`, `FR-015/016` mapped to S-08/S-09 | REVIEWED_INTERNAL |
| Planning integration | `FR-010/011` S-07 and prior feature docs referenced | REVIEWED_INTERNAL |
| UX source | `UF-19..24`, `SCR-14..17` and fallback `SCR-22` | REVIEWED_INTERNAL |
| Engine boundary | Formulas delegated to `CS-009/011/012` and future engine spec | REVIEWED_INTERNAL |
| Financial invariants | `DR-001..024` relevant rules, no actual writes | REVIEWED_INTERNAL |
| Local tests | `SIMT-001..048` are **test specifications**, not actual PASS | REVIEWED_INTERNAL |
| Local decisions | `SIM-Q01..18` map to unresolved parent Q decisions | REVIEWED_INTERNAL |
| App/CI test execution | Code repository not audited; no tests executed by writing this document | NOT_RUN |
| Owner approval | None documented in this draft | PENDING_OWNER_APPROVAL |

**Required next financial document:** `docs/financial/SCENARIO-ENGINE.md` to define numeric state transitions, event order, baseline/state versioning decisions, replay/golden cases, and error policies after approval. `MONTE-CARLO.md` and `GOAL-OPTIMIZER.md` remain dedicated R3 specs and must not be silently folded into deterministic UI implementation.

**Updating related documents:** Setelah owner menyetujui SIM-Q decisions, perbarui `../planning/DECISION-LOG.md`, `../planning/TASKS.md`, `../planning/PROGRESS.md`, `../development/TESTING-STRATEGY.md`, `../architecture/API-GUIDELINES.md` serta `../architecture/DATABASE-DESIGN.md` as needed, dan review perubahan pada `../design/SCREEN-SPECS.md`. `../INDEX.md` perlu diperbarui agar merujuk file ini setelah integrasi artefak ke repo. Ini tidak otomatis dilakukan hanya karena file `.md` dibuat di sesi chat.

---

## Appendix A — Contoh copy UI yang aman (bukan final localized strings)

| Kondisi | Copy bahasa Indonesia yang bisa direview | Jangan gunakan |
|---|---|---|
| Versi asal jelas | `Simulasi berdasarkan data per 30 Okt 2026 dan asumsi yang Anda masukkan.` | `Inilah saldo Anda di masa depan.` |
| Hasil stale | `Data aktual berubah sejak simulasi ini dibuat. Buat perhitungan baru untuk memakai data terbaru.` | `Hasil lama salah.` |
| Input tidak lengkap | `Kami belum bisa menghitung karena data cicilan bulanan belum tersedia.` | `Kami menganggap cicilan Anda Rp0.` |
| Forecast minus | `Pada asumsi ini, proyeksi kas akhir bulan ke-5 adalah -Rp1.000.000.` | `Anda pasti kehabisan uang bulan kelima.` |
| Beda model | `Skenario ini belum bisa dibandingkan karena mata uang dan horizon berbeda.` | `Skenario B memberi hasil lebih baik.` |
| Tidak ada statistical model | `Ini proyeksi berdasarkan asumsi tetap, bukan peluang kejadian.` | `Kesuksesan Anda 95% pasti.` |
| Protected data | `Rencana tidak tersedia atau Anda tidak memiliki akses.` | `Skenario 'Rumah Pak A' milik user 123 ditemukan.` |
| Retry uncertain | `Status penyimpanan belum dapat dipastikan. Periksa kembali sebelum mengulangi.` | `Gagal total, tekan simpan berkali-kali.` |

Contoh tanggal di atas **bukan** tanggal baseline nyata user; semua UI akan memperoleh `as_of` dari data yang benar-benar dimiliki response approved.

## Appendix B — QA fixture example, NOT an OpenAPI contract

```text
Fixture name       = synthetic_stress_income_disruption
Source             = hypothetical/deterministic (R2)
Baseline cash      = IDR 4,000,000 at t0
Monthly income     = IDR 1,000,000 end of month, baseline only
Monthly expense    = IDR 1,000,000 end of month, baseline + shock
Shock              = replace income with IDR 0 for months 1..5
Horizon            = 5 whole months
Liquidity threshold= IDR 0, checked end of each month
No extra debt      = explicit fixture assumption (not product default)
Baseline close     = [4m,4m,4m,4m,4m]
Shock close        = [3m,2m,1m,0,-1m]
First shock breach = month 5 (for threshold and timing above)
Delta month 5      = -5m relative to baseline
Run state          = illustrative expected data, NOT an executed test
Upstream sources   = FTC-STRESS-01, FTC-STRESS-02, FTC-STRESS-03
```

## Appendix C — Perubahan yang wajib memicu review silang

Perubahan berikut tidak boleh dianggap "UI-only" tanpa analisis impact: arti `shortfall`; nominal/real representation; horizon normalization; `stale` detection; actual baseline revisions; goal funded percentage; income replacement logic; event ordering; refund/transfer categorization; chart metric definition; source labels; error statuses; engine/model versions; authz for scenario nested resources; retention/export/delete linked runs; offline caching; async retry/cancel semantics; approved shock catalog. Review minimal `PRD`, domain/calculation specs, fixture `FTC-*`, backend/API/DB specs, `UF/SCR` dan DoD, lalu update task/test evidence.

**Dokumen ini menutup spesifikasi pengalaman simulasi R2, bukan mengesahkan algoritma produksinya.**
