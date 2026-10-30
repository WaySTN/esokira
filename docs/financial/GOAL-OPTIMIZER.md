# Esokira — Multi-Goal Financial Optimizer Specification

> **Dokumen kanonis:** `docs/financial/GOAL-OPTIMIZER.md`  
> **Versi:** `0.1.0`  
> **Tanggal draft:** `2026-10-30`  
> **Status dokumen:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Status fitur:** `SPECIFICATION ONLY / UNVERIFIED / R3 PRODUCTION MODEL BLOCKED`  
> **Produk / stack terpilih:** Esokira; Flutter/Dart, Go, PostgreSQL, monorepo  
> **Release / requirement / kontrak / epic:** R3, `S-10`, `FR-017`, `CS-013`, `EP-S10-01`  
> **Keputusan utama:** `Q-07` (objective, constraints, priority, feasibility), bersama `Q-11` (goal funding), `Q-06` (snapshot) dan `Q-04` (presisi uang) sesuai dampak  
> **Approval:** Product Owner + reviewer domain keuangan/matematika dan keamanan. Draft ini bukan keputusan untuk mengaktifkan optimizer produksi.

> **Untuk AI coding agent:** Ikuti [`AGENTS.md`](../../AGENTS.md) dan [`backend/AGENTS.md`](../../backend/AGENTS.md) sebelum menyentuh engine, serta [`frontend/AGENTS.md`](../../frontend/AGENTS.md) untuk UI. Jangan memilih *objective*, solver, batas likuiditas, kebijakan prioritas, return, tie-break, funding attribution, parameter retry, atau batas waktu produksi hanya karena ada contoh di sini. Catat persetujuan di [`DECISION-LOG.md`](../planning/DECISION-LOG.md). **Selama `Q-07` tidak disetujui, keluaran optimizer yang diberi label `optimal` untuk pengguna adalah BLOCKED.** Hasil feasibility pada fixture sintetis boleh diteliti dan diuji secara terisolasi, bukan diklaim sebagai fitur produk selesai.

---

## 0. Tujuan, kontrak, dan batas interpretasi

Multi-goal optimizer membantu pengguna mengevaluasi **bagaimana kapasitas kontribusi yang terbatas dapat dibagi secara terencana** kepada beberapa tujuan dengan nominal, tanggal, prioritas, dan kendala berbeda. Engine harus dapat menjelaskan tujuan yang terpenuhi, tidak terpenuhi, serta *trade-off* yang dibuat; tidak boleh menghasilkan angka alokasi seolah tersedia dana tunai tambahan atau menjanjikan hasil investasi.

Prinsip wajib:

1. **Plan ≠ actual.** Hasil optimasi hanyalah rencana (`proposed allocation`), tidak membuat transfer, posting ledger, pendanaan goal aktual, atau penambahan saldo.
2. **Feasible ≠ optimal.** Jadwal yang memenuhi constraint hanyalah bukti kelayakan. Label *optimal* baru boleh digunakan jika fungsi tujuan, algoritma, status solver, dan pembuktian/toleransinya disetujui.
3. **Budget is not free cash.** Kapasitas harus berasal dari model cash-flow/komitmen yang disetujui, setelah kewajiban dan cadangan, bukan sekadar angka saldo akun yang tersedia.
4. **No double count.** `P_g` (dana awal yang benar-benar eligible untuk goal) tidak boleh diperhitungkan ulang pada dua goal eksklusif, cash flow, atau posisi aset.
5. **No hidden omission.** Semua goal dipilih yang diproses harus muncul pada hasil, termasuk shortfall, deadline miss, excluded dengan alasan, dan hasil tidak diketahui.
6. **Reproducible and explainable.** Simpan referensi snapshot, versi input, tujuan, kebijakan, engine/solver, toleransi numerik, dan fingerprint. Edit atau data aktual yang berubah tidak diam-diam mengganti run lama.
7. **Safe & bounded.** Seluruh data, job, hasil dan unduhan owner-scoped; tidak ada loop optimasi tanpa batas, klaim kepastian finansial, atau data privat pada log.

Dokumen ini adalah spesifikasi **algoritma dan kontrak engine** untuk `CS-013`. UX mengikuti [`SCREEN-SPECS.md`](../design/SCREEN-SPECS.md) `SCR-18` dan [`USER-FLOWS.md`](../design/USER-FLOWS.md) `UF-26`. Aturan finansial referensi tetap berasal dari [`FINANCIAL-DOMAIN-RULES.md`](FINANCIAL-DOMAIN-RULES.md) dan [`CALCULATION-SPECS.md`](CALCULATION-SPECS.md). Contoh numerik di bawah menguji suatu model sempit dan **bukan** menetapkan kebijakan produk universal.

### 0.1 Status normatif

| Label | Artinya | Perilaku agent |
|---|---|---|
| `REQUIRED` | Domain/security invariant atau perlakuan kejujuran model | Jadikan acceptance dan negative test |
| `REFERENCE-EXACT` | Oracle numerik untuk data sintetis dan asumsi yang disebut eksplisit | Uji hasil persis pada fixture tersebut |
| `PROPOSED` | Opsi desain/algoritma yang belum dipilih | Dokumentasikan pilihan, prototipe terisolasi |
| `CONDITIONAL` | Berlaku setelah prasyarat input/model disetujui | Jangan gunakan sebagai default produksi |
| `TBD` | Keputusan atau kontrak belum disepakati | Catat decision dan evidence needed |
| `BLOCKED` | Penerapan atau klaim production tidak diizinkan | Hindari implementasi yang bergantung pada keputusan |
| `UNVERIFIED` / `NOT_RUN` | Kode atau pengujian repository belum diperiksa/dijalankan | Jangan laporkan sebagai PASS/DONE |

## 1. Source of truth dan peta dokumen

| Pertanyaan yang hendak dijawab | Sumber yang harus diperiksa | Batas kewenangan |
|---|---|---|
| Fitur, scope, prioritas rilis | [`PRD.md`](../product/PRD.md), [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md) | `FR-017` R3; R4 tetap discovery |
| Arti planned/actual, net worth, non-mutation | [`FINANCIAL-DOMAIN-RULES.md`](FINANCIAL-DOMAIN-RULES.md) | `DR-011/012/014/016/017/020/024` |
| Formula dan contoh nilai | [`CALCULATION-SPECS.md`](CALCULATION-SPECS.md) §15 | `CS-013`, `CALC-REF-13`; formulasi awal bukan objective final |
| Oracle independent & kasus negatif | [`FINANCIAL-TEST-CASES.md`](FINANCIAL-TEST-CASES.md) §15 | `FTC-ALLOC-01`–`FTC-ALLOC-08` |
| Goal dan pendanaan aktual | [`FINANCIAL-GOALS.md`](../features/FINANCIAL-GOALS.md) | `FR-007/009`, `Q-11`, funded attribution masih pending |
| Sumber transaksi aktual | [`TRANSACTIONS.md`](../features/TRANSACTIONS.md) | Tidak ada auto-transfer dari optimizer |
| Sumber kapasitas/cash forecast dan snapshot | [`SCENARIO-ENGINE.md`](SCENARIO-ENGINE.md), [`SIMULATIONS.md`](../features/SIMULATIONS.md) | Timing, baseline, stale, assumptions mengikuti keputusan R2 |
| Ketidakpastian dan risiko | [`MONTE-CARLO.md`](MONTE-CARLO.md) | `FR-018/CS-014`, Q-08; bukan bukti optimalitas |
| Domain Go, DB dan wire contract | [`BACKEND-ARCHITECTURE.md`](../architecture/BACKEND-ARCHITECTURE.md), [`DATABASE-DESIGN.md`](../architecture/DATABASE-DESIGN.md), [`API-GUIDELINES.md`](../architecture/API-GUIDELINES.md) | Contoh package/tabel/endpoint tidak otomatis final |
| UI, alur, privasi | [`USER-FLOWS.md`](../design/USER-FLOWS.md), [`SCREEN-SPECS.md`](../design/SCREEN-SPECS.md), [`DESIGN-SYSTEM.md`](../design/DESIGN-SYSTEM.md), [`SECURITY-PRIVACY.md`](../security/SECURITY-PRIVACY.md) | `UF-26`, `SCR-18`, `SEC-*` sesuai fase |
| Scope task dan evidence | [`TASKS.md`](../planning/TASKS.md), [`MASTER-ROADMAP.md`](../planning/MASTER-ROADMAP.md), [`TESTING-STRATEGY.md`](../development/TESTING-STRATEGY.md), [`DEFINITION-OF-DONE.md`](../development/DEFINITION-OF-DONE.md), [`DEVELOPMENT-WORKFLOW.md`](../development/DEVELOPMENT-WORKFLOW.md) | `EP-S10-01`, R3 gate; tidak ada status PASS tanpa run |
| Owner decisions dan teknologi | [`DECISION-LOG.md`](../planning/DECISION-LOG.md), [`0001-TECH-STACK.md`](../architecture/ADR/0001-TECH-STACK.md), [`docs/INDEX.md`](../INDEX.md) | Keputusan disetujui dicatat sebelum mengubah outcome |

Jika contoh dalam dokumen ini bertentangan dengan sumber domain/calc atau keputusan owner yang benar-benar tercatat, **jangan memilih salah satu diam-diam**. Laporkan `BLOCKED` untuk irisan implementasi yang terdampak, jelaskan konflik, dan minta keputusan atau update sumber yang sah. Semua referensi adalah dokumen desain; keberadaannya **tidak membuktikan** file implementasi, OpenAPI final, migrations, atau CI benar-benar ada.

## 2. Release, dependensi, dan batas scope

| Release/slice | ID | Kontribusi ke optimizer | Status/ketergantungan |
|---|---|---|---|
| R0 | `EN-004` | Pure engine boundary dan harness untuk contoh finansial | Fondasi, bukan `FR-017` selesai |
| R1 / S-05–06 | `FR-007/009`, `TK-S06-03` | Goal metadata dan affordability satu goal | Belum menetapkan allocation optimizer |
| R2 / S-08 | `FR-012/013`, `EP-S08-02` | Baseline, scenario versions, cash-flow forecast | Baseline/source capacity harus jelas |
| **R3 / S-10** | **`FR-017`, `CS-013`, `EP-S10-01`** | **Multi-goal optimizer, feasible/infeasible + evidence** | **BLOCKED `Q-07` sampai objective/constraints disetujui** |
| R3 / S-10 | `EN-005`, `EP-S10-02` | Worker bila beban membutuhkan asynchronous run | Kebijakan queue, quota, retry, cancel masih TBD |
| R3 / S-11 | `FR-018`, `CS-014` | Risiko stokastik, bukan optimizer default | `Q-08`, terpisah |
| R3 / S-12 | `FR-020`, `CS-016` | Financial GPS boleh membaca hasil yang telah lolos gate | Tidak boleh auto-post actual |
| R4 | `FR-023/024` | Shared goals, portfolio planning | **Discovery only** |

`EP-S10-01` bergantung pada `EP-S08-02` dan `TK-S06-03` sesuai [`TASKS.md`](../planning/TASKS.md). Implementasi engine R3 hanya setelah owner menyetujui scope rilis; keberhasilan test R1 tidak secara otomatis memberi izin untuk R3.

### 2.1 Termasuk scope

- Alokasi kontribusi terencana beberapa goal menurut periode, kapasitas dan constraint yang disetujui.
- Validasi konsistensi model dan pengecekan kelayakan; keluaran yang menyebut deadline, sumber pendanaan, shortfall dan alasan.
- Alternatif *candidate* untuk membantu diskusi trade-off, **bukan** rekomendasi instrumen investasi spesifik.
- Auditability, perbandingan solusi yang kompatibel, numerical testing dan integrasi UI R3.

### 2.2 Dikecualikan dari scope

- Auto-debit, auto-invest, transfer otomatis, pembayaran nyata, perubahan data aktual, kredit baru, broker trading.
- Pemilihan reksa dana, saham, cryptocurrency atau produk investasi spesifik atas nama pengguna.
- Mengubah hasil Monte Carlo menjadi klaim optimum tanpa definisi optimasi stokastik yang disetujui.
- Membuat shared-goal multiuser (`FR-023`) atau portfolio-planning (`FR-024`) R4 tanpa discovery.
- Memastikan target keuangan tercapai; optimizer menunjukkan pilihan **di bawah asumsi dan kendala**, bukan jaminan hasil.

## 3. Terminologi dan tipe data konseptual

| Istilah | Makna | Guardrail |
|---|---|---|
| `goal_g` | Tujuan dengan target, tanggal, prioritas, metadata/currency | Owner-scoped, unik, tidak diam-diam diabaikan |
| `target_g` (`T_g`) | Nilai nominal target pada basis yang disebut | Tidak menyamakan nominal dan real |
| `deadline_g` (`d_g`) | Periode terakhir untuk mencapai target | Kalender, zona waktu, periode inklusif TBD |
| `eligible_initial_funds` (`P_g`) | Dana awal yang sah diatribusikan ke goal | **Bukan** total cash copy; `Q-11` |
| `capacity_t` (`S_t`) | Batas dana tambahan yang boleh direncanakan di periode `t` | Sesudah biaya hidup/kewajiban/buffer yang disetujui |
| `planned_allocation` (`x_{g,t}`) | Kontribusi ke goal `g` pada periode `t` | Rencana, **tidak** mengubah ledger atau funded amount |
| `projected_goal_value` (`G_{g,t}`) | Hasil model goal jika rencana/asumsi berlaku | *Forecast*, bukan saldo aktual |
| `shortfall_g` | Selisih target dan nilai yang memenuhi syarat pada deadline | Tidak disembunyikan, satuan/tanggal jelas |
| `hard_constraint` | Syarat yang tidak boleh dilanggar jika hasil diberi label feasible | Pelanggaran → infeasible/invalid |
| `soft_preference` | Preferensi yang trade-off-nya ditentukan objective | Bobot/tie-break belum disetujui |
| `candidate_solution` | Jadwal alokasi yang diusulkan solver | Baru `feasible` setelah diverifikasi |
| `optimal_solution` | Feasible solution dengan pembuktian optimum menurut objective dan toleransi yang disetujui | **BLOCKED** tanpa `Q-07` |
| `baseline_snapshot` | Keadaan finansial actual pada referensi/as-of tertentu | Immutable/traceable; stale detection |
| `scenario_version` | Versi asumsi & cash forecast yang dipakai | Tidak diam-diam mutate historical runs |

**Tanggal dan unit adalah bagian dari data.** Nilai `Rp2.000.000/bulan` tidak sama dengan Rp2 juta pada tanggal tidak terdefinisi; kapasitas tidak identik dengan saldo rekening. Definisi `capacity_t` dan `P_g` harus dapat dipertanggungjawabkan dari model yang disetujui, bukan hasil tebakan UI.

## 4. Input contract dan validasi awal

Sebuah request konseptual membutuhkan:

1. **Identity & provenance:** authenticated principal, owner-scoped goal IDs, `baseline_ref`, `scenario_version`, `as_of`, versi kontrak/model engine, fingerprint aturan keuangan.
2. **Goals:** daftar nonempty dari goal eligible, identifier unik, nominal target dan currency, deadline, status lifecycle, kemungkinan komitmen existing, `P_g` yang telah dibuktikan jika model funding disetujui.
3. **Capacity:** `S_t` per periode dari sumber cash forecast valid/approved, tingkat likuiditas minimum, pengeluaran wajib, kewajiban lain, serta pelarangan double spending.
4. **Model:** discretization/calendar, contribution timing, target inflation/nominal conversion, return assumption, apakah goal wajib sepenuhnya dipenuhi atau best-effort, objective dan solver policy **setelah approval**.
5. **Safety controls:** batas jumlah goal, horizon, ukuran model, timeout, deadline server, quota pemilik, cancel/retry policy, audit metadata.

Validasi harus memisahkan `INVALID_INPUT` (mis. deadline hilang), `MODEL_NOT_APPROVED`, `INCOMPATIBLE_CURRENCY`, `STALE_INPUT`, `CAPACITY_UNAVAILABLE`, dan kasus matematis **`INFEASIBLE`**. *Infeasible* bukan bug; tidak ada model yang valid untuk diuji saat input kritis belum disetujui juga **bukan** infeasible matematis.

### 4.1 Incompatible inputs

- Goal A IDR vs Goal B USD **tidak** boleh dijumlahkan tanpa FX dan tanggal/basis konversi disetujui.
- Goal target real tahun dasar dan kapasitas nominal masa depan membutuhkan konversi eksplisit; tak boleh ranking berdasar angka raw.
- Akun dengan dana dialokasikan eksklusif kepada satu goal tidak boleh diasumsikan eligible untuk goal lainnya.
- Tenggat sebelum as-of atau `d_g=0` memerlukan status sesuai kontrak, bukan pembagian oleh nol.
- Tujuan sudah selesai/ditutup, duplikat ID, owner lain, atau data yang sudah stale tidak boleh diproses sebagai goal aktif tanpa aturan.
- `S_t` negatif akibat komitmen melebihi pemasukan tidak boleh diam-diam diubah menjadi nol lalu disimpulkan layak; harus ada pengakuan funding gap / invalid capacity policy.

## 5. Model matematika konseptual (`CS-013`)

Kandidat **formulasi dasar** dari [`CALCULATION-SPECS.md`](CALCULATION-SPECS.md) §15 untuk goal `g∈{1,…,G}` dan periode `t∈{1,…,H}` adalah:

```text
x[g,t] >= 0                                          # kontribusi terencana, bukan posting
sum_g x[g,t] <= S[t]                               # tidak melampaui kapasitas periode
G[g,0] = P[g]                                     # dana goal yang eligible, tanpa double count
G[g,t] = G[g,t-1] * (1 + r[g,t]) + x[g,t]          # hanya jika return & timing disetujui
G[g,d[g]] >= T[g]                                 # berlaku hanya jika goal g adalah hard goal
```

Kebijakan dana yang tidak dialokasikan (`slack_t`), pengeluaran goal pada deadline, top-up, inflasi target, loss/fee, drawdown, kontribusi awal/akhir periode dan perubahan rencana **bukan** terselesaikan dengan menuliskan persamaan di atas. Semua harus masuk kebijakan yang disetujui dan diuji. Jika return belum valid, fixture netral `r=0` untuk verifikasi matematis **tidak mengesahkan** return nol sebagai asumsi produk default.

### 5.1 Bentuk constraints

**Hard constraints, kandidat REQUIRED setelah model approved:** capacity per-period, `x>=0`, eligibility dan ownership, minimum cash reserve, syarat deadline goal wajib, batas likuiditas, non-overlap dana awal, komitmen existing, mata uang dan waktu konsisten, jumlah total dana tidak negatif, larangan alokasi ke goal noneligible.

**Soft preferences, membutuhkan Q-07:** bobot prioritas, preferensi goal selesai lebih cepat, pemerataan, mengurangi shortfall persentase, membatasi perubahan alokasi dari bulan ke bulan, menghindari volatilitas kontribusi, komitmen goal opsional.

**Model choice:** Jika semua target wajib dan tak tercapai, status `INFEASIBLE` adalah hasil sah. Jika objective mengizinkan shortfall, jadwal mungkin memenuhi hard constraints tetapi ada unmet goals. Kontrak hasil harus membedakan kedua hal itu; istilah `FEASIBLE_WITH_SHORTFALL` hanya boleh dipakai bila objective/semantics disetujui.

### 5.2 Jangan mengasumsikan linear selalu

Constraint yang memuat fixed activation cost, ketentuan minimum per goal, return yang bergantung pada tindakan, atau status all-or-nothing dapat menjadi integer/mixed-integer/nonlinear. Tidak boleh memilih LP solver atau greedy heuristic sambil menyembunyikan perbedaan kelas masalah. Setiap reformulasi perlu bukti ekuivalensi terhadap kontrak domain dan validasi di batas integer money.

## 6. Kapasitas kontribusi per periode dan sumber dana

Sumber `S_t` harus merupakan **capacity envelope yang disetujui**, bukan rata-rata pendapatan tanpa komitmen. Sumber dapat berasal dari R2 deterministic cash-flow forecast (`FR-014`) yang mencakup pendapatan, pengeluaran prioritas, kewajiban utang, life events, dan buffer; namun definisi cadangan dan hubungan dengan goal contributions masih keputusan kebijakan.

Contoh *pemeriksaan prinsip*, bukan kebijakan formula produksi:

```text
cash_projection_before_goal_t
  = opening_eligible_cash_t + forecast_income_t
  - mandatory_consumption_t - mandatory_debt_cash_t
  - approved_reserves_or_earmarked_commitments_t

candidate_S_t <= verified_liquid_surplus_t
# Do not silently assume it equals savings account current balance.
```

**Anti circularity:** Jika proyeksi cash-flow sudah mengurangi kontribusi goal, sumber `S_t` harus menghindari pemotongan kedua kali. Jika proyeksi memasukkan rencana goal masa depan, optimizer harus memakai baseline konsisten/versi yang ditentukan, bukan feed-back tanpa iterasi dan konvergensi yang dibuktikan. Transfer antar-account actual netral terhadap total wealth dan tidak boleh diperlakukan sebagai pemasukan.

### 6.1 Reserve and emergency fund

Model dapat melindungi dana darurat/kebutuhan wajib melalui batas likuiditas, tetapi besaran, eligible accounts, prioritas kebutuhan, urutan pembayaran dan kebijakan funding **TBD** (`Q-07`, `Q-11`, kebijakan finance lain yang terkait). Jangan memilih sendiri angka 'cadangan tiga bulan' atau default yang terkesan final. Setiap batas yang tampil pada hasil harus menyebut asal kebijakannya.

### 6.2 Constraints ketika actual dan forecast berubah

Run sebelumnya tetap merekam snapshot yang dipakai; actual posting baru/goal edit harus menyebabkan status stale atau rebase eksplisit sesuai `Q-06`. Tidak boleh membatalkan rekomendasi lama sambil menampilkan label terbaru tanpa provenance.

## 7. Funded attribution dan pemisahan actual vs proposed

`P_g` bukan angka yang boleh didapat dari `account_balance` lalu dipakai untuk seluruh goal. Opsi funded model dalam [`FINANCIAL-GOALS.md`](../features/FINANCIAL-GOALS.md)—virtual envelope, sumber dedicated balance, atau mekanisme lain—masih bergantung pada `Q-11`. Persyaratan invariant:

1. Jumlah dana yang diatribusikan secara eksklusif ke goal tidak melebihi dana eligible yang tersedia menurut snapshot.
2. Dana milik goal A tidak otomatis menjadi dana awal goal B. Reallocation harus jelas dan tidak mengeksekusi transfer actual.
3. Nilai target dan progress yang diinput sendiri bukan bukti deposito/investasi actual.
4. Perubahan planned `x[g,t]` tidak otomatis mengubah actual `funded_amount`, account balance, transaction history, liability, atau net worth.
5. `P_g` harus memakai effective date/funding policy, bukan menyalin future returns sebagai dana yang sudah diterima.
6. Jika funded model belum dipilih, output yang bergantung padanya adalah **BLOCKED**, walaupun contoh fixture `P=0` bisa diuji secara terisolasi.

Cek `FTC-ALLOC-07` dan [`TRANSACTIONS.md`](../features/TRANSACTIONS.md) untuk invariants double-count dan idempotence. Jika actual-funded tidak dapat diverifikasi, UI harus membedakan *target*, *rencana*, *simulasi*, dan *dana aktual* tanpa menyimpulkan progress% palsu.

## 8. Kalender, periode, dan target deadlines

Kebijakan tanggal per bulan menentukan kapan suatu kontribusi tersedia dan apakah goal tercapai sebelum batas target. Hal-hal yang **harus dispesifikasikan**:

- `as_of`, timezone, akhir bulan pendek, leap year, horizon `H`, deadline `d_g` inklusif/eksklusif.
- Timing pembayaran gaji, pengeluaran wajib, kontribusi goal dan pembelian/biaya goal dalam siklus periode.
- Cara proyeksi nominal/real dan inflasi target dinormalisasi pada tanggal yang sama.
- Perubahan deadline dari draft ke versi baru; tidak mengubah run historis.
- Perilaku untuk goal yang sudah melewati deadline: deadline reached + shortfall atau status lifecycle yang tepat, bukan divide by zero.

Contoh `CALC-REF-13` menggunakan **bulan diskret dan kontribusi akhir-bulan**; pemilihan timing untuk semua goal lain tetap perlu approval. Semua hasil UI dan API wajib menyebut horizon dan basis waktu yang dipakai.

## 9. Klasifikasi constraints dan status solusi

| Status konseptual | Definisi ketat | Apa yang boleh ditampilkan |
|---|---|---|
| `VALIDATED_FEASIBLE` | Jadwal diketahui memenuhi **semua hard constraints** yang disetujui pada model versi tersebut | Alokasi kandidat + bukti/diagnostik; bukan klaim optimal |
| `PROVEN_INFEASIBLE` | Ada bukti bahwa **tidak ada** solusi memenuhi seluruh hard constraints model yang disetujui | Deadline/constraint konflik dan shortfall diagnostik |
| `FEASIBLE_WITH_UNMET_PREFERENCES` | Semua hard constraints dipenuhi tetapi target lunak belum terpenuhi | Trade-off lengkap, hanya jika soft policy approved |
| `OPTIMAL_PROVEN` | Objective disetujui dan solver menyajikan proof/certificate atau bound/gap sesuai kebijakan | Boleh menyebut optimal **menurut model**, bukan rekomendasi pasti |
| `FEASIBLE_NOT_PROVEN_OPTIMAL` | Solver punya feasible incumbent namun belum bisa membuktikan optimum | Boleh tampilkan kandidat, **tanpa** label optimal |
| `UNKNOWN_OR_TIMED_OUT` | Tidak ditemukan jawaban terbukti sebelum resource cap / solver gagal | Tidak boleh klaim infeasible atau optimal |
| `INVALID_INPUT` | Currency/time/owner/amount/assumption salah atau tak lengkap | Validasi aman; tanpa rekomendasi numerik |
| `MODEL_NOT_APPROVED` | Objective/constraints/funding policy belum final | Blocker jelas, tanpa hasil produksi |
| `STALE_RESULT` | Snapshot atau scenario revision lebih baru daripada yang dipakai | Label run historis, perintah rebase eksplisit |

Kode status pada tabel adalah **semantik kandidat**, bukan enum API/DB yang telah disetujui. Mapping ke status HTTP dan schema akhir menunggu [`API-GUIDELINES.md`](../architecture/API-GUIDELINES.md), review QA, dan kontrak formal. Jangan menggabungkan *infeasible matematis* dengan *timeout sistem*.

## 10. Objective function dan prioritas — **BLOCKED Q-07**

Tidak ada objective yang universal. Opsi berikut adalah **PROPOSED OPTIONS, bukan keputusan final**:

| Opsi | Contoh objective | Potensi konsekuensi |
|---|---|---|
| A. Hard all-goals first | Feasibility untuk semua goal wajib, kemudian minimisasi slack atau perubahan plan | Banyak instance `INFEASIBLE`; butuh alternatif yang jujur |
| B. Weighted absolute shortfall | `min Σ_g w_g · max(0,T_g−G_g,d_g)` | Goal nominal besar bisa dominan; unit dan bobot harus bermakna |
| C. Weighted relative shortfall | `min Σ_g w_g · shortfall_g / T_g` untuk `T_g>0` | Lebih seimbang proporsional, tetapi sensitif goal kecil dan target nol |
| D. Lexicographic priority | Penuhi kelas prioritas tertinggi dulu, lalu berikutnya | Dapat menyingkirkan goal prioritas rendah; perlu transparansi |
| E. Max goals achieved | Maksimalkan jumlah goal yang memenuhi deadline, dengan constraint lain | Dapat mengorbankan goal besar; memerlukan binary variables |
| F. Minimize lateness | Minimalkan keterlambatan berbobot setelah deadline | Butuh definisi keterlambatan dan horizon tambahan |
| G. Preserve liquidity | Minimalkan risiko shortfall cash/buffer sebagai hard/soft objective | Tidak boleh disamakan dengan return pasti |

Penggunaan bobot prioritas memerlukan skala, satuan, tie-break, pilihan goal wajib/opsional dan default yang disetujui. Jangan menyebut hasil 'optimal' jika angka objective belum ditetapkan. Alternatif yang sah bagi UX sementara adalah label **'Alternatif pembagian dana'** atau **'Contoh rencana berdasarkan asumsi'**.

### 10.1 Trade-off yang wajib dijelaskan

Sebuah hasil hanya dapat dipercaya jika menampilkan `goal_id`, target, deadline, planned per-period, total projected/shortfall, constraints yang aktif, asumsi return/capacity, alasan goal tidak terpenuhi, dan status bukti kelayakan. Jika prioritas goal memengaruhi pengorbanan goal lain, dampaknya tidak boleh disembunyikan atau disamarkan menjadi 'semua tercapai'.

### 10.2 Tie-break dan stabilitas

Untuk input yang setara, objective yang sama mungkin menghasilkan banyak solusi. Tanpa tie-break disetujui, urutan tidak boleh dipaksakan berdasarkan sort `goal_id` lalu diklaim sebagai aturan produk. **Deterministic tie-break** harus ditetapkan sebelum produksi agar perubahan urutan input/versi solver tidak membuat penjelasan bertentangan tanpa sebab.

## 11. Solver, optimality gap, dan batas kepercayaan

Pemilihan algoritma baru dilakukan setelah objective/constraint disetujui dan kelas masalah jelas:

- **Deterministic constructive feasibility checker** untuk subkasus tanpa return/fee/FX dapat menjadi oracle awal; *bukan* pengganti optimizer global.
- **Linear programming** mungkin cocok untuk constraints linier kontinu; butuh money-scale policy, feasibility tolerance dan dual diagnostics.
- **Mixed-integer optimization** dapat diperlukan untuk all-or-nothing goals atau fixed event costs, dengan time cap dan bound/gap.
- **Dynamic programming** mungkin masuk akal untuk struktur diskret terbatas; ukuran state dan precision tetap harus diukur.
- **Greedy/heuristic** bisa membuat kandidat feasible yang berguna, tetapi **tidak** boleh diberi label optimal tanpa bukti terpisah.
- **Risk-aware/stochastic optimization** berada di luar optimizer deterministik ini sampai Q-07 dan Q-08 serta validation protocol disetujui bersama.

### 11.1 Solver result verification terpisah

Jangan percaya *solver status* tanpa memverifikasi jadwal hasil terhadap constraints dalam representasi uang yang dipilih. Untuk solusi *optimal*, evaluasi perlu mencatat model/objective version, bound, gap, tolerance, tie-break, proof/certificate jika disyaratkan, dan pengujian pembanding terhadap solusi orakel untuk instance kecil. Timeout dengan *incumbent* feasible tidak identik dengan optimum dan **tidak** membuktikan infeasibility.

### 11.2 Numerical precision dan integer boundary

Keuangan aktual disimpan/diproses menggunakan representasi presisi aman; penggunaan `float64` sebagai satu-satunya sumber nilai uang bukan solusi. Representasi Go, skala minor unit, rounding, tolerance dan validasi solver terhadap exact-money constraints bergantung pada `Q-04`/`CALC-Q*`. Solver internal yang menggunakan floating-point dapat dipertimbangkan hanya dengan **reconciliation verifier exact** dan rounding policy approved. Jangan memperbaiki pelanggaran kapasitas dengan menyembunyikan rupiah selisih.

## 12. Golden reference `CALC-REF-13` dan `FTC-ALLOC-*`

**Fixture sintetis, bukan kebijakan produk:** mata uang IDR; semua goal mulai `P=0`; tidak ada return, fee, inflasi, FX, dan perubahan kapasitas; kontribusi akhir bulan; seluruh goal wajib terpenuhi tepat di deadline; dana tidak digunakan ulang.

| Goal | Target | Deadline |
|---|---:|---:|
| A | Rp12.000.000 | Akhir bulan ke-12 |
| B | Rp24.000.000 | Akhir bulan ke-24 |
| C | Rp120.000.000 | Akhir bulan ke-60 |

### 12.1 Necessary lower bound untuk kapasitas konstan `S`

```text
S >= 12_000_000 / 12                            = 1_000_000
S >= (12_000_000 + 24_000_000) / 24            = 1_500_000
S >= (12_000_000 + 24_000_000 + 120_000_000) / 60
  = 156_000_000 / 60                          = 2_600_000
S_min = max(1_000_000, 1_500_000, 2_600_000) = 2_600_000 IDR/bulan
```

Syarat perlu ini adalah syarat cukup **untuk fixture ini** karena ada *constructive witness* berikut; jangan memakai rumus sebagai aturan semua model dengan return, liquidity lock, atau biaya.

### 12.2 Constructive feasibility witness `S=Rp2.600.000/bulan`

| Periode | A per bulan | B per bulan | C per bulan | Total |
|---|---:|---:|---:|---:|
| 1–12 (12 bulan) | 1.000.000 | 1.000.000 | 600.000 | 2.600.000 |
| 13–24 (12 bulan) | 0 | 1.000.000 | 1.600.000 | 2.600.000 |
| 25–60 (36 bulan) | 0 | 0 | 2.600.000 | 2.600.000 |

- Pada bulan 12, A=`12×1.000.000 = Rp12.000.000`.
- Pada bulan 24, B=`24×1.000.000 = Rp24.000.000`.
- Pada bulan 60, C=`12×600.000 + 12×1.600.000 + 36×2.600.000 = 7.200.000 + 19.200.000 + 93.600.000 = Rp120.000.000`.
- Tiap bulan total alokasi Rp2.600.000 dan tidak ada target yang diabaikan.
- **Kesimpulan sah:** fixture tersebut feasible dengan kapasitas konstan Rp2.600.000/bulan, dan itu kapasitas minimum untuk seluruh target hard pada asumsi contoh. **Tidak mengesahkan objective atau optimalitas solusi solver generik.**

### 12.3 Infeasible instance `S=Rp2.000.000/bulan`

Pada 60 bulan, total kapasitas `60 × Rp2.000.000 = Rp120.000.000`, sedangkan total target `Rp156.000.000`. Defisit agregat **setidaknya Rp36.000.000** walaupun semua kontribusi dapat dialihkan sesuai kebutuhan; tidak ada jadwal yang bisa memenuhi ketiga hard goals sesuai batas waktu dan asumsi fixture. Ini **bukti infeasible** yang terlepas dari pemilihan objective. Jangan memilih goal yang gagal sendiri lalu menampilkan hasil 'semua tujuan tercapai'.

### 12.4 Oracle trace

| Sumber | Apa yang diperiksa | Batas |
|---|---|---|
| `CALC-REF-13` | `S_min` Rp2,6 juta/bulan untuk fixture | Tidak membuktikan general optimizer |
| `FTC-ALLOC-01/03` | Nilai batas dan witness feasible | Conditional/reference fixture |
| `FTC-ALLOC-02` | S Rp2 juta/bulan infeasible, gap agregat | `REF-EXACT` fixture |
| `FTC-ALLOC-04` | Jumlah alokasi > capacity harus ditolak | Domain invariant |
| `FTC-ALLOC-05` | Tie-break belum approved | `BLOCKED` |
| `FTC-ALLOC-06` | Infeasible/shortfall tidak tersembunyi | Domain invariant |
| `FTC-ALLOC-07` | Dana initial tidak diatribusikan ganda | Domain invariant, funding Q-11 |
| `FTC-ALLOC-08` | Return/timing belum disetujui → rank blocked | `BLOCKED` |

### 12.5 Contoh tambahan yang **bukan** oracle keputusan

Jika hanya satu goal `A` dengan target Rp12 juta pada akhir bulan 12 dan kapasitas Rp1 juta/bulan, jadwal Rp1 juta per bulan **memenuhi** hard constraint pada model tanpa return dan biaya. Itu tidak menjawab apakah menjadwalkan setoran lebih awal, lebih lambat, atau dengan kombinasi investment yang lain lebih optimal menurut preferensi pengguna.

## 13. Alternative plans, sensitivity, dan explanation

Fitur boleh membantu meninjau **trade-off** hanya untuk plan/model yang kompatibel. Jika kapasitas, deadline, prioritas, atau goal berbeda, perubahan harus dinyatakan sebagai perubahan asumsi, bukan bukti bahwa solver tertentu 'lebih cerdas'. Untuk perubahan satu parameter, catat input yang berubah, delta alokasi, delta shortfall, kendala aktif, dan hasil status.

Contoh *counterfactual draft* yang sah: “Pada skenario sintetis ini, Rp2 juta/bulan tidak cukup untuk memenuhi tiga target. Kebutuhan minimum pada asumsi tanpa return adalah Rp2,6 juta/bulan. Anda bisa mengevaluasi kapasitas yang berbeda, deadline yang lebih panjang, atau target yang diubah.” Ini **tidak** menjadi saran menambah penghasilan, menunda kebutuhan penting, atau membeli produk investasi.

**Batas sensitivity:** perubahan satuan/horizon dan baseline memerlukan run versi baru. Jika beberapa constraint berubah, tidak boleh mengatribusi seluruh delta ke satu penyebab tanpa decomposisi yang tervalidasi. Keterangan “lebih baik” harus menyebut metrik objektif yang approved.

## 14. Scenario snapshots, lineage, replay, stale

Minimal lineage konseptual setiap run:

```text
run_id (owner-scoped)
owner_id (internal; never trust client-supplied owner)
baseline_snapshot_ref + baseline_version/as_of
scenario_id + immutable_scenario_version / scenario_fingerprint
ordered_goal_ids + goals_version + funding_attribution_policy_version
capacity_curve_version + currency/time_basis/rounding_policy
objective_version + constraints_policy_version + priority_policy_version
engine_version + solver_version + solver_configuration_hash
status + validation_evidence + generated_at
result_hash + input_hash + approval_refs
```

Tidak semua field wajib berupa kolom sendiri; desain database menunggu approval dan migration nyata. *Input fingerprint* harus mencakup urutan stabil, normalisasi tanggal dan satuan, serta versi aturan; jangan menganggap dua JSON dengan urutan key berbeda sebagai data finansial berbeda jika semantiknya sama. Setiap run *immutable*; perubahan actual atau scenario terbaru memicu label stale sesuai kebijakan Q-06 dan proses rebase eksplisit, bukan retroactive rewrite.

### 14.1 Determinism dan perbandingan solver

Ulangan input + policy + solver version yang sama harus memenuhi klaim determinisme yang disetujui. Namun multiple optimal solutions mungkin ada; exact vector equality hanya boleh diwajibkan jika tie-break deterministik sudah disahkan. Perbandingan lintas versi bisa menghasilkan alokasi berbeda walaupun objective sama; tampilkan versi, status, dan kendala yang dibandingkan.

## 15. Go domain boundary dan pseudocode yang **tidak mengikat API**

Financial engine `CS-013` adalah bagian pure computation yang tidak bergantung pada HTTP, widget Flutter, atau PostgreSQL. *Candidate* pemisahan:

```text
backend/internal/finance/allocation/    # conceptual, verify actual layout
  model.go       # typed goal/capacity inputs, units, policy IDs
  validate.go    # provenance, numeric & policy validation
  feasibility.go # constraint verification and diagnostics
  solver.go      # approved method only
  explain.go     # reason codes, shortfalls, trade-offs
  tests/...      # fixture oracles/property tests
```

Kode pseudocode berikut menegaskan **batas tanggung jawab**, tidak mendefinisikan nama struct Go yang final atau memilih dependency solver:

```go
// PSEUDOCODE ONLY — illustrative architecture, not compile-ready API.
func EvaluateGoalAllocation(ctx Context, input ApprovedInput) (Result, error) {
    checkOwnershipAndInputLineage(ctx, input)
    if !approved(input.ModelPolicy) { return Result{}, ErrModelNotApproved }
    normalized := normalizeWithApprovedMoneyTimeRules(input)
    if err := validateCapacityAndFunding(normalized); err != nil { return Result{}, err }

    solution := selectedSolver(ctx, normalized) // Q-07: not approved yet
    verified := verifyAllHardConstraintsExact(normalized, solution)
    if !verified { return Result{}, ErrInvalidSolverOutput }
    return explainWithoutPostingActual(normalized, solution), nil
}
```

**Trust boundary:** verifikasi owner dilakukan di orchestration/use-case sebelum membaca sumber private; pure engine menerima input lengkap yang sudah diotorisasi, dan tidak menjalankan SQL, HTTP, side effects, atau logging data sensitif. Core calculation boleh dipakai unit test tanpa database/Flutter.

### 15.1 Cross-module responsibilities

- `goal` module mengelola goal definition/lifecycle, bukan alokasi otomatis.
- `transactions` module memegang actual ledger, tidak boleh dipanggil dari optimizer untuk posting.
- `scenario` module menyediakan versi baseline dan projection yang tervalidasi, tidak menebak funded goal.
- `allocation` engine memvalidasi mathematical problem dan menghasilkan kandidat + evidence.
- Orchestration mengatur owner, approval status, run lifecycle, persistence, caching jika disetujui.

## 16. PostgreSQL, API, dan contract governance

`DATABASE-DESIGN.md` serta `API-GUIDELINES.md` memuat rancangan R3, **bukan migrations/OpenAPI final**. Kandidat resource `goal_optimization_runs` atau tabel input/output job harus menjalani review schema dan retention sebelum dibuat. Request/result yang disimpan hanya data yang diperlukan, owner-scoped dengan FK dan checks yang relevan, immutable run result atau append-only versioning sesuai kebijakan, tidak menggandakan current balance sebagai fakta baru.

**Jangan menetapkan** `/api/v1/optimizations/goal-allocations` sebagai endpoint yang sudah tersedia hanya karena disebut pada API Guidelines. Kode hanya boleh mengikuti kontrak yang benar-benar telah di-approve di `contracts/openapi.yaml` (jika sudah ada dan terverifikasi) dan migrasi yang relevan. Endpoint potensial harus mencakup: validasi mode, idempotent creation/retry policy, `run_id` owner scope, status/failed reasons, result provenance, stale flag, dan larangan menulis ledger.

### 16.1 Result DTO konseptual

```text
OptimizationResult (SCHEMA CANDIDATE, NOT API CONTRACT):
  status                   # distinction: valid feasible / proven infeasible / unknown
  model_and_solver_versions
  baseline_and_scenario_refs
  goal_outcomes[]: goal_id, target, deadline, planned_by_period, projected_at_deadline,
                   shortfall, priority_policy_ref, met/unmet reason
  capacity_by_period[], unused_capacity_by_period[]
  constraint_evidence[]    # constraint IDs, bounds, violation diagnostics
  objective_metadata?      # only when Q-07 approved
  optimality_evidence?     # proof/gap/tolerance only if validated
  warnings[], assumptions[], stale_indicator, timestamps
```

Satuan uang, enum, nama field, pagination, retention, bentuk error, request identity dan serialisasi **semuanya mengikuti kontrak final**, bukan contoh ini. Jangan menyebarkan owner ID, secret, detail goal, atau nominal privat dalam exception atau telemetry yang tidak aman.

## 17. Worker / asynchronous execution (`EN-005`) — kondisional

Optimizer ringan **tidak harus** menggunakan worker hanya karena modul Monte Carlo membutuhkannya. Bila benchmark nyata dan persetujuan desain menyatakan perlu background job, gunakan kebijakan `EN-005` yang disetujui:

- Per-owner quota dan fair use, hard caps goal/horizon/iterations, bounded concurrency, deadline/timeout.
- State job konseptual `accepted/queued/running/succeeded/failed/cancelled` hanya jika backend benar-benar menyediakan; Flutter tidak membuat progress palsu.
- Idempotency/dedupe terhadap request replay, crash recovery, dan hasil yang disimpan secara atomik sesuai kontrak.
- Cancellation dan timeout tidak memproduksi hasil `OPTIMAL_PROVEN` tanpa validation. Feasible incumbent dapat disimpan sesuai kebijakan tetapi tidak boleh disamakan dengan complete run.
- Race pada perubahan goal actual/snapshot harus menghasilkan run versi lama dengan stale indicator atau penolakan input, bukan silently mixed-version optimization.
- Tidak ada hasil cross-tenant; worker dan storage menjalankan identity/authorization yang setara dengan API.

Worker library, queue provider, retry intervals dan SLA masih TBD. Jangan membuat dependencies/service permanen sebelum approval dan kebutuhan terukur.

## 18. Flutter UX (`UF-26` / `SCR-18`)

Alur minimal **ketika model sudah approved**:

1. Pengguna memilih goal yang eligible dan melihat target, deadline, prioritas serta pendanaan yang **benar-benar** tersedia menurut policy.
2. UI menjelaskan kapasitas per periode, cadangan, asumsi return/nominal-real, baseline dan batas rekomendasi.
3. Pengguna mengubah *draft constraint* dan melakukan validasi sebelum menjalankan solver; request berbasis versi input yang eksplisit.
4. UI menampilkan state yang sesuai: `input missing`, `model not approved`, `running`, `feasible candidate`, `infeasible with reason`, `unknown/timed out`, `stale`, atau `error`.
5. Per-goal result menampilkan allocation schedule dan shortfall; semua goal yang dipilih tetap disebut, termasuk yang gagal atau dikecualikan beserta alasannya.
6. Penyimpanan *proposed draft plan* hanya setelah konfirmasi, bila kontrak mendukung. **Tidak ada automatic transaction**, perubahan funded progress atau actual balance.

Untuk mockup R3 sebelum model disetujui, gunakan **data sintetis yang terlihat jelas sebagai contoh**, bukan angka rekomendasi produksi. Copy `Alternatif pembagian dana` lebih aman daripada `Alokasi optimal`. Tabel yang accessible dan pembaca layar harus menyajikan nominal, periode, hasil dan sumber data sama jelasnya dengan grafik. Hindari warna hijau tunggal sebagai bukti sukses; gunakan teks dan label status.

### 18.1 Result explanation & transparency

Hasil perlu membedakan:

- **Target tidak tercapai** karena kapasitas yang diberikan tidak cukup menurut model.
- **Goal excluded** karena eligibility/input, bukan karena solver menghilangkannya tanpa catatan.
- **Belum bisa dihitung** karena keputusan model/timing/funding belum ada.
- **Solusi sementara** karena solver belum membuktikan optimum atau timeout.
- **Run historis** dengan baseline stale, bukan rekomendasi mutakhir.

Tombol `Simpan rencana` tidak menyiratkan uang sudah dipindah. Untuk perubahan actual, pengguna harus melalui flow transaksi yang eksplisit dan otorisasi berbeda.

## 19. Security, privacy, dan tenant isolation

Aturan [`SECURITY-PRIVACY.md`](../security/SECURITY-PRIVACY.md) berlaku penuh:

- Authn/authz server-side untuk goal IDs, source accounts, snapshot, scenario, job, dan stored results, termasuk nested resource dan file export.
- Jangan percaya `owner_id` dari Flutter atau request; query harus owner-scoped dan menolak foreign IDs walaupun UUID valid.
- Tidak boleh menyertakan financial amounts, goal titles, atau serialized solver input di log standar, analytics, distributed tracing, exception publik, atau notification payload.
- Rate limit/quotas mencegah denial-of-service memakai goal count/horizon tinggi; cancellation wajib bounded bila tersedia.
- Retention, user export/delete, audit access, hasil lama dan worker cleanup sesuai kebijakan yang telah disetujui; jangan mengarang jangka waktu.
- Caching/dedupe harus menggunakan user scope dan input fingerprint. Hash bukan pengganti authorization; hindari cross-user cache-key collisions/leaks.
- Test request owner A terhadap run milik B pada endpoint get/status/cancel/download harus gagal tanpa bocor isi.

## 20. Failure modes, retries, dan reason codes

| Kondisi | Penanganan engine/orchestration | Perilaku UI |
|---|---|---|
| Objective/solver policy belum disetujui | `MODEL_NOT_APPROVED`; jangan menjalankan production allocation | Jelaskan blocker, tanpa angka optimal palsu |
| Tidak ada kapasitas yang dapat dipercaya | `CAPACITY_UNAVAILABLE` atau validasi sesuai API contract | Minta sumber asumsi yang lengkap |
| Hard constraints terbukti tidak mungkin | `INFEASIBLE` dengan bukti/shortfall | Tampilkan konflik dan alternatif manual |
| Solver timeout tanpa proof | `UNKNOWN/TIMED_OUT`, mungkin incumbent feasible jika terverifikasi | Tidak bilang optimal atau infeasible |
| Goal dari akun lain | `UNAUTHORIZED_RESOURCE` atau respons aman setara | Tidak memunculkan data privat |
| Snapshot berubah setelah submit | Historical result + stale policy atau explicit invalidation | Baca ulang/rebase setelah persetujuan |
| Queue retries after uncertain submission | Dedupe by approved semantics, don't duplicate run or actual posting | Status lookup sebelum retry jika identitas tersedia |
| Jumlah uang tidak kompatibel currency/nominal basis | `INCOMPATIBLE_INPUT` | Jangan ranking raw values |
| Negative capacity dari perhitungan kewajiban | Funding gap/invalid policy, bukan silent clamp | Jelaskan kebutuhan data/perubahan asumsi |
| Solver menghasilkan alokasi > capacity akibat rounding | Verifier menolak, jangan terbitkan feasible | Error terukur dan trace yang aman |

Reason names adalah placeholder semantik, bukan error enum produksi. Pemilihan HTTP status, retryable vs terminal, dan response details menunggu API policy disetujui.

## 21. Performance, reliability, dan acceptance benchmarking

Target p50/p95, jumlah goal maksimum, bulan maksimum, solver gap, runtime deadline, memori, concurrency dan quota **belum boleh dikunci tanpa keputusan**. Rencana benchmark minimum:

- **Small exact-oracle:** 1–3 goals, 12–60 periode, no return, witness dan brute-force/independent checker yang dapat diverifikasi.
- **Medium structured:** 5–20 goals, tenggat tersebar, capacity bervariasi, priority conflicts; observasi memori dan validasi constraints (bukan klaim angka SLA).
- **Large/adversarial:** horizon panjang, goal count tinggi, target nol, duplicate IDs, overflow input, empty capacity, conflict keras; timeout aman.
- **Concurrency/multiuser:** banyak request, retry, owner-scope, cancel/isolation, stale input, GC pressure, worker crash jika async.
- **Model change:** objective/solver version upgrade; old results remain traceable; tidak ada perubahan diam-diam pada rekomendasi yang sudah pernah ditampilkan.

Performa bukan alasan untuk mengembalikan solusi yang melanggar hard constraint, menandai result acak sebagai *optimal*, atau menghapus shortfall. Simpan evidence benchmark nyata dan environment-nya di progress/handoff saat implementasi.

## 22. Strategi testing dan oracle independen

Baca [`TESTING-STRATEGY.md`](../development/TESTING-STRATEGY.md) dan [`FINANCIAL-TEST-CASES.md`](FINANCIAL-TEST-CASES.md). Test perlu berlapis:

1. **Pure math tests:** exact checks `CALC-REF-13`, capacity, deadline, `P_g` unique, zero return, no mutation.
2. **Independent witness/verifier:** lakukan rekonstruksi jumlah alokasi, deadlines, funded attribution; bedakan solver candidate dari checker.
3. **Property/metamorphic:** `Σx<=S`; menambahkan capacity tidak boleh *mengurangi himpunan feasible* pada model konstan tanpa constraint tambahan; renaming `goal_id` tidak semestinya mengubah kemampuan target tercapai; reproducibility dengan policy versi tetap.
4. **Solver objective tests setelah approval:** bench terhadap oracle independen untuk instance kecil, tie-break, optimality gap, condition yang menjadikan hasil unknown.
5. **Integration tests:** owner authorization, database transaction semantics, job retry, queue failure/cancel, stale run, API contract yang telah disetujui.
6. **Flutter tests:** empty/invalid/infeasible/stale/error, aksesibilitas nominal, display assumptions dan *not actual funded*, one-click retry behavior.
7. **Security/privacy and release gates:** per-user isolation, logging hygiene, denial of service, data lifecycle, R3 approvals dan DoD.

Status test harus dicatat sebagai `PASS/FAIL/NOT_RUN/SKIPPED/BLOCKED` **hanya berdasarkan eksekusi yang benar-benar dilakukan**. Review dokumen ini tidak berarti test Go, Flutter, PostgreSQL, worker atau model telah dijalankan.

### 22.1 Golden-to-engine traceability

| Reference | Fokus | Test lokal |
|---|---|---|
| `CALC-REF-13` | Minimum kapasitas konstan + witness Rp2,6 juta | `GOT-001/002/003/007/008` |
| `FTC-ALLOC-01` | Threshold minimum pada fixture 3 goal | `GOT-001` |
| `FTC-ALLOC-02` | Kapasitas Rp2m infeasible | `GOT-002` |
| `FTC-ALLOC-03` | Alokasi witness semua goal | `GOT-003` |
| `FTC-ALLOC-04` | Melampaui capacity bukan feasible | `GOT-004` |
| `FTC-ALLOC-05` | Equal priority tie-break belum disetujui | `GOT-033` |
| `FTC-ALLOC-06` | Infeasible/shortfall tidak disembunyikan | `GOT-006/032` |
| `FTC-ALLOC-07` | Double-funded initial amount dicegah | `GOT-014` |
| `FTC-ALLOC-08` | Return/timing belum disetujui → block | `GOT-038` |

## 23. Domain invariants (`GO-I01`–`GO-I24`)

Seluruh invariant di bawah wajib dijadikan test/acceptance. Ada yang dapat diuji pada fixture sintetis sekarang, ada yang membutuhkan pemilihan model sebelum produksi; status tidak boleh disamarkan.

| ID | Invariant | Syarat verifikasi | Referensi |
|---|---|---|---|
| `GO-I01` | Actual finance unchanged | Run, view, save draft, retry/cancel tidak memutasi account, transaction, asset, liability maupun net worth. | `DR-002/012; FR-017` |
| `GO-I02` | Planned vs funded | `x[g,t]` hanyalah planned; tak mengubah actual-funded progress. | `DR-011; Q-11` |
| `GO-I03` | Owner authorization | Goal/snapshot/scenario/job/result owner-scoped end-to-end. | `DR-003; SEC-005` |
| `GO-I04` | Money exactness | Semua amount output yang disajikan sebagai uang lolos exact-money verifier. | `DR-004; Q-04` |
| `GO-I05` | Capacity conservation | Untuk setiap periode, `sum_g x[g,t] <= S_t`. | `FTC-ALLOC-04` |
| `GO-I06` | Nonnegative allocations | Tidak ada kontribusi negatif yang disembunyikan oleh model. | `CS-013` |
| `GO-I07` | No duplicated initial funding | `P_g` exclusive allocation tidak diduplikasi pada dua goal. | `FTC-ALLOC-07; Q-11` |
| `GO-I08` | Goal completeness | Semua goal terpilih muncul: met/unmet/excluded dengan alasan. | `FR-017; FTC-ALLOC-06` |
| `GO-I09` | Deadline correctness | Goal hard dinilai pada deadline/basis waktu yang disetujui. | `CS-013; Q-07` |
| `GO-I10` | No secret objective | Tidak ada ranking tanpa objective dan priority policy approved. | `Q-07` |
| `GO-I11` | Feasible verified | `FEASIBLE` harus lolos independent hard-constraint checker. | `FTC-ALLOC-03/04` |
| `GO-I12` | Infeasible proved | `INFEASIBLE` hanya dengan bukti, bukan timeout atau error. | `FTC-ALLOC-02` |
| `GO-I13` | Optimal only if proven | `OPTIMAL` mensyaratkan objective, proof/gap/tolerance approved. | `Q-07; DR-024` |
| `GO-I14` | No hidden goal deletion | Goal yang tak terpenuhi tak disembunyikan atau otomatis dibuang. | `FTC-ALLOC-06` |
| `GO-I15` | No undocumented cash creation | S_t harus berasal dari kapasitas terverifikasi, bukan current balance saja. | `DR-017` |
| `GO-I16` | No currency/real mismatch | Tidak menjumlahkan unit/basis nilai tidak kompatibel. | `DR-014/018` |
| `GO-I17` | Reproducibility lineage | Input, policy, solver/engine dan baseline versions terlacak. | `DR-016; NFR-003` |
| `GO-I18` | Stale handled | Perubahan input actual/goal/scenario tidak diam-diam memperbarui run lama. | `DR-020` |
| `GO-I19` | Timeout honesty | Timeout/unknown bukan bukti infeasible atau optimal. | `NFR-006` |
| `GO-I20` | Result provenance | Setiap angka/shortfall mempunyai referensi asumsi, periode, model. | `DR-014/019` |
| `GO-I21` | No sensitive telemetry | Nominal, goal title, token, atau raw input privat tak bocor di log umum. | `DR-021; NFR-007` |
| `GO-I22` | Bounded resources | Run dijaga dalam limit dan tidak dibuat berganda oleh retry. | `EN-005; NFR-006` |
| `GO-I23` | Accessibility and honesty | UI menampilkan shortfall/status/limitation dalam teks, bukan warna saja. | `NFR-005; SCR-18` |
| `GO-I24` | Release gate | Tidak menerbitkan hasil production tanpa Q-07, Q-11 relevan, model proof dan tests. | `DR-024; EP-S10-01` |

## 24. Matrix of 60 locally numbered test scenarios (`GOT-001`–`GOT-060`)

**Status awal semua test:** `NOT_RUN` pada repository aplikasi. `REF-EXACT` berlaku hanya untuk data sintetis dan asumsi yang tertulis; `BLOCKED` berarti test production memerlukan approval. ID `GOT-*` adalah ID lokal untuk dokumen ini, **bukan** pengganti `FTC-ALLOC-*` dan bukan bukti pengujian sudah ada.

### 24.1 Test group 1 — Numeric reference & feasibility (`GOT-001`–`GOT-010`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-001` — Three-goal lower bound | A12m/12,B24m/24,C120m/60; P=0,r=0,S konst. | Minimum constant S Rp2.600.000/bulan. | `REF-EXACT` | `FTC-ALLOC-01` |
| `GOT-002` — Three-goal insufficient budget | S=Rp2m/bulan 60 bulan, target total 156m | PROVEN_INFEASIBLE; total shortfall minimum agregat Rp36m. | `REF-EXACT` | `FTC-ALLOC-02` |
| `GOT-003` — Three-goal witness | Jadwal 1–12 1m/1m/.6m;13–24 0/1m/1.6m;25–60 0/0/2.6m | Constraint tiap bulan dan semua deadline valid. | `REF-EXACT` | `FTC-ALLOC-03` |
| `GOT-004` — Budget overflow | Vector total 3m dengan S=2m. | Reject; tidak boleh dinilai feasible. | `REQUIRED` | `FTC-ALLOC-04` |
| `GOT-005` — Zero capacity | S_t=0 untuk satu periode, goal aktif. | Hanya alokasi nol pada periode itu; status global menurut constraints. | `REQUIRED` | `CS-013` |
| `GOT-006` — Goal omission | Semua 3 goal dipilih, satu tak didanai. | Goal gagal tetap tercantum dengan shortfall/constraint. | `REQUIRED` | `FTC-ALLOC-06` |
| `GOT-007` — Witness A deadline | 12×1m untuk A bulan1–12. | A target 12m tepat pada d=12. | `REF-EXACT` | `CALC-REF-13` |
| `GOT-008` — Witness B deadline | 24×1m untuk B bulan1–24. | B target 24m tepat pada d=24. | `REF-EXACT` | `CALC-REF-13` |
| `GOT-009` — Witness C deadline | C=.6m×12+1.6m×12+2.6m×36. | C=120m tepat d=60. | `REF-EXACT` | `CALC-REF-13` |
| `GOT-010` — One-goal sanity | Goal12m/d12, S1m×12, return0. | Feasible candidate, tidak otomatis optimal. | `REFERENCE` | `CS-013` |

### 24.2 Test group 2 — Capacity, reserve & funding (`GOT-011`–`GOT-020`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-011` — Capacity varies | S=[1m,0,2m], model simple no returns. | Setiap t mematuhi kapasitas masing-masing. | `REQUIRED` | `CS-013` |
| `GOT-012` — Negative allocation | Input x<0 pada goal kapanpun. | Invalid candidate tanpa silent fix. | `REQUIRED` | `CS-013` |
| `GOT-013` — Negative financial surplus | Forecast cash negatif setelah kewajiban. | Tampilkan funding gap/validation; jangan silent clamp. | `CONDITIONAL` | `Q-07` |
| `GOT-014` — Duplicate funded cash | Saldo yang sama diklaim penuh oleh A+B secara exclusive. | Tolak double attribution. | `REQUIRED` | `FTC-ALLOC-07` |
| `GOT-015` — No funded evidence | P_g diisi dari asumsi manual yang tidak approved. | Block actual-funded claim. | `BLOCKED` | `Q-11` |
| `GOT-016` — Reserve cannot be spent | Plan akan memakai restricted emergency reserve. | Constraint reserve melarang jika approved. | `CONDITIONAL` | `Q-07` |
| `GOT-017` — Commitment counted twice | Forecast sudah mengurangi goal savings dan optimizer mengurangi lagi. | Double deduction terdeteksi atau model invalid. | `REQUIRED` | `DR-017` |
| `GOT-018` — Actual balance not capacity | Cash balance besar tapi surplus aktual nonpositive. | Tak boleh menyimpulkan S_t dari balance saja. | `REQUIRED` | `DR-011` |
| `GOT-019` — Transfer not income | Transfer internal dari A ke B di source forecast. | Tak menambah S_t/net worth hanya karena transfer. | `REQUIRED` | `DR-006` |
| `GOT-020` — No ledger mutation | Jalankan/simpan hasil beberapa kali. | Transaction count, balance, net worth actual tidak berubah. | `INTEGRATION` | `DR-012` |

### 24.3 Test group 3 — Time, FX, money correctness (`GOT-021`–`GOT-030`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-021` — Deadline reached | Goal d=0 dan tidak funded. | Deadline reached/shortfall; no divide by zero. | `REQUIRED` | `CALC-REF-16` |
| `GOT-022` — Past deadline | Goal deadline sebelum as_of. | Invalid/lifecycle status, tidak ubah deadline diam-diam. | `CONDITIONAL` | `Q-07` |
| `GOT-023` — Month end date | Goal jatuh 31 dan Februari/April. | Date convention approved konsisten. | `BLOCKED` | `Q-07` |
| `GOT-024` — Contribution timing | Awal/akhir periode dengan return tidak nol. | Hasil hanya menurut timing policy approved. | `BLOCKED` | `Q-07` |
| `GOT-025` — Target inflation | Target nominal vs real basis tidak sama. | Normalisasi eksplisit atau reject. | `REQUIRED` | `DR-014` |
| `GOT-026` — Currency mismatch | Goal IDR dan USD tanpa FX approved. | Tidak dijumlahkan atau diranking raw values. | `REQUIRED` | `DR-014` |
| `GOT-027` — Rounding boundary | Solver float menghasilkan Σx melebihi S sebesar minor unit. | Exact verifier menolak hasil. | `REQUIRED` | `DR-004` |
| `GOT-028` — Large amount overflow | Nominal melampaui batas tipe money approved. | Invalid input/overflow safe, bukan wrap. | `REQUIRED` | `NFR-002` |
| `GOT-029` — Zero target | T_g=0 pada objective relative shortfall. | Tidak membagi nol, policy input/achieved TBD. | `BLOCKED` | `Q-07` |
| `GOT-030` — Negative return | r_t<0 pada model investasi. | Jangan auto-clamp dana; policy kerugian approved. | `BLOCKED` | `Q-07` |

### 24.4 Test group 4 — Objective, priority & proof (`GOT-031`–`GOT-040`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-031` — Contradictory hard constraints | Dana minimum wajib > kapasitas tiap periode. | Infeasible dengan bukti jika model fixed. | `CONDITIONAL` | `CS-013` |
| `GOT-032` — Unmet goal displayed | Solver memiliki partial plan yang gagal goal wajib. | Tidak mencap plan hard-feasible atau menghapus goal. | `REQUIRED` | `FTC-ALLOC-06` |
| `GOT-033` — Equal priorities tie | Dua goal prioritas sama, banyak solusi. | No production ranking tanpa tie-break policy. | `BLOCKED` | `FTC-ALLOC-05` |
| `GOT-034` — Priority tradeoff | Goal wajib vs opsional berebut budget. | Hasil hanya setelah hard/soft semantics approved. | `BLOCKED` | `Q-07` |
| `GOT-035` — Objective unit consistency | Absolute dan relative shortfall memberi ranking berbeda. | Tidak mengklaim superior tanpa objective approved. | `BLOCKED` | `Q-07` |
| `GOT-036` — Optimality proof | Solver menyatakan optimal tanpa bound/proof policy. | Tolak label optimal. | `REQUIRED` | `DR-024` |
| `GOT-037` — Timeout incumbent | Solver timeout dengan feasible schedule. | Feasible but not proven optimal; no false claim. | `REQUIRED` | `Q-07` |
| `GOT-038` — Unapproved timing | Return/contribution timing belum ditetapkan. | Block production objective/solution. | `BLOCKED` | `FTC-ALLOC-08` |
| `GOT-039` — Solver infeasible vs timeout | Solver gagal menemukan solusi sebelum limit. | UNKNOWN bukan PROVEN_INFEASIBLE. | `REQUIRED` | `NFR-006` |
| `GOT-040` — Input order independence | Goal input permutation tetapi same IDs/model. | Feasibility sama, allocation equality only tie-break approved. | `CONDITIONAL` | `Q-07` |

### 24.5 Test group 5 — Ownership, storage & reliability (`GOT-041`–`GOT-050`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-041` — Owner foreign goal | User A memasukkan goal ID user B. | Server denies and no detail leaked. | `INTEGRATION` | `DR-003` |
| `GOT-042` — Owner foreign run | User A membaca result B. | Denied incl nested view/download/status. | `INTEGRATION` | `SEC-005` |
| `GOT-043` — Owner foreign cancel | User A mencoba cancel job B. | Denied, job B unchanged. | `INTEGRATION` | `SEC-005` |
| `GOT-044` — Stale baseline | Actual changed after run snapshot. | Historical result marked stale/rebase explicit. | `CONDITIONAL` | `Q-06` |
| `GOT-045` — Scenario version immutable | Assumptions edited after run. | Old run remains tied to old version. | `REQUIRED` | `DR-016` |
| `GOT-046` — Retry identical request | Network timeout after run accepted. | No duplicate actual posting; dedupe semantics approved. | `CONDITIONAL` | `EN-005` |
| `GOT-047` — Worker crash recovery | Job crashes mid-solve. | No false terminal success; retry bounded. | `CONDITIONAL` | `EN-005` |
| `GOT-048` — Cancel midway | Cancel before solver complete. | No optimal claim or bogus valid results. | `CONDITIONAL` | `EN-005` |
| `GOT-049` — Privacy of logs | Solver errors include raw goal amounts/titles. | Telemetry sanitized, secrets absent. | `INTEGRATION` | `DR-021` |
| `GOT-050` — Missing approved contract | No approved OpenAPI schema or DB migration. | Task slice blocked; no invented endpoints. | `REQUIRED` | `API-Q16` |

### 24.6 Test group 6 — Flutter UX and release (`GOT-051`–`GOT-060`)

| ID | Setup / aksi | Expected | Readiness | Sumber |
|---|---|---|---|---|
| `GOT-051` — Feasible UI copy | Screen displays valid schedule. | Label plan/proposed; never actual transfer. | `WIDGET` | `SCR-18` |
| `GOT-052` — Infeasible UI copy | All hard goals not feasible. | Show goal shortfalls and reasons, not blank error. | `WIDGET` | `UF-26` |
| `GOT-053` — Unknown UI copy | Solver timed out. | Show unknown, not infeasible or success. | `WIDGET` | `SCR-18` |
| `GOT-054` — Accessibility result | Result has currency/time and status info. | Screen-reader text/table conveys info without color. | `WIDGET` | `NFR-005` |
| `GOT-055` — Save proposed plan | Explicit user confirms save draft. | Persist draft only, no account mutation/funded mark. | `INTEGRATION` | `DR-012` |
| `GOT-056` — Stale UX | Old result after account correction. | Stale badge and explicit rebase path per policy. | `CONDITIONAL` | `Q-06` |
| `GOT-057` — Alternative explanations | Capacity/deadline changed in new draft. | Shows changed assumptions and constraint deltas. | `CONDITIONAL` | `FR-017` |
| `GOT-058` — Resource exhaustion | Adversarial large goals/horizon. | Bounded response/timeout, no cross-user side effect. | `INTEGRATION` | `NFR-004` |
| `GOT-059` — DoD evidence | No engine/test execution yet. | Task remains UNVERIFIED/NOT_RUN; never DONE. | `REQUIRED` | `DOD-G*` |
| `GOT-060` — R3 gate | Q-07 or funding model Q-11 missing. | Do not release numerical optimizer to production. | `REQUIRED` | `DR-024` |

## 25. Requirement traceability dan source mapping

| ID | Tujuan / kontrak | Evidence lokal minimal | Approval gate |
|---|---|---|---|
| `FR-017` | Multi-goal optimizer R3 | `GOT-001`–`GOT-060`, UI `UF-26` / `SCR-18` | `Q-07` objective/constraints |
| `CS-013` | Constraint formulation, feasibility dan solver | `FTC-ALLOC-01`–`FTC-ALLOC-08`, witness, independent proof | `CALC-Q12`, `FDR-Q11` |
| `FR-007/009` | Goal lifecycle/feasibility dan funded source | `GOT-014/015/020`, financial goal tests | `Q-11` |
| `FR-012/013` | Deterministic model, immutable scenario versions | `GOT-044/045/056`, scenario tests | `Q-06` & scenario policies |
| `FR-018` | Monte Carlo risk side, if integrated later | Separate model/statistical test suite | `Q-08`, not implied by FR-017 |
| `EN-005` | Bounded worker when justified | `GOT-046`–`GOT-048`, owner/timeout tests | Queue provider/resource policies TBD |
| `NFR-001` | Security & authz | `GOT-041`–`GOT-043`, security cases | Actual integration tests |
| `NFR-002` | Financial correctness | `GOT-001`–`GOT-030`, independent numeric oracle | Exact-money/rounding policy |
| `NFR-003` | Reproducible scenarios | `GOT-044/045`, provenance snapshot | Approved versioning |
| `NFR-004` | Performance / bounded compute | `GOT-058`, benchmark evidence | SLA caps not decided |
| `NFR-005` | Accessibility and honest results | `GOT-051`–`GOT-054` | UX review |
| `NFR-006` | Reliability and retry | `GOT-046`–`GOT-048`, state machine | Worker policies |
| `NFR-007/008` | Observability/privacy and data lifecycle | `GOT-049`, deletion/export/retention tests | Security review |
| `NFR-009` | Quality gate | `GOT-059/060`, R3 acceptance | `DEFINITION-OF-DONE.md` |
| `NFR-010` | No actual mutation from simulations | `GOT-020/055` | Backend isolation tests |
| `DR-011/012/014/016/017/020/024` | Funding, nonmutation, assumptions, reproducibility, stale, model gates | `GO-I01`–`GO-I24` | Applicable owner approvals |
| `EP-S10-01` | Implementation epic | Go + PostgreSQL + API + Flutter test evidence | Depends `EP-S08-02`, `TK-S06-03` |

*Beberapa dokumen sebelumnya belum mencatat keberadaan file baru ini karena dibuat lebih dulu.* Saat audit final, perbarui `docs/INDEX.md`, `README.md`, `TASKS.md`, dan `DECISION-LOG.md` **hanya setelah file benar-benar ada dan owner menyetujui perubahan yang relevan**. Jangan mengubah task status menjadi DONE hanya karena spesifikasi ditulis.

## 26. Open decisions (`GO-Q01`–`GO-Q18`)

Semua keputusan berikut **`NOT_RECORDED / PENDING_OWNER_APPROVAL`** sampai bukti approval yang sah ditemukan. ID lokal adalah rincian dari `Q-07`/keputusan lain, bukan pengganti keputusan induk di [`DECISION-LOG.md`](../planning/DECISION-LOG.md).

| ID | Keputusan belum final | Parent/terkait | Dampak bila belum diputuskan |
|---|---|---|---|
| `GO-Q01` | Objective utama (hard targets, weighted shortfall, lexicographic, etc.) | `Q-07; CALC-Q12` | BLOCKED optimizer production ranking |
| `GO-Q02` | Hard vs soft goals, all-or-nothing semantics | `Q-07` | BLOCKED feasibility result interpretation |
| `GO-Q03` | Sumber `S_t` dan akurasi cash-flow capacity, reserves | `Q-07; Q-06` | BLOCKED budget authority |
| `GO-Q04` | Funding attribution `P_g`, dedicated/virtual/fund sources | `Q-11; FDR-Q07` | BLOCKED funded-goal optimization |
| `GO-Q05` | Priority levels/weights, normalization absolute/relative | `Q-07; FDR-Q11` | BLOCKED trade-off priority |
| `GO-Q06` | Tie-break and stability across permutations, solver versions | `Q-07` | BLOCKED optimal allocation disclosure |
| `GO-Q07` | Time discretization, calendar/deadline inclusivity, contribution timing | `Q-07; CALC-Q12` | BLOCKED production date exactness |
| `GO-Q08` | Return, losses, inflation, FX, nominal/real policy | `Q-05; Q-07` | BLOCKED generalized projections |
| `GO-Q09` | Money scale/rounding and post-solver exact verifier | `Q-04; CALC-Q12` | BLOCKED exact-money publication |
| `GO-Q10` | Solver family and independent optimality proof, gap/tolerance | `Q-07; BE-Q11` | BLOCKED `OPTIMAL_PROVEN` |
| `GO-Q11` | Infeasible diagnostics, partial plan and unmet goal UX | `Q-07; SS-Q16` | BLOCKED user promise semantics |
| `GO-Q12` | Goal lifecycle/eligibility; duplicates, past deadlines, changes | `Q-07; FG-Q*` | BLOCKED incomplete input semantics |
| `GO-Q13` | Snapshot version, stale detection, rebase/result retention | `Q-06; DB-Q*` | BLOCKED saved run behavior |
| `GO-Q14` | Async/worker necessity and quotas/cancel/retries | `EN-005; BE-Q10` | BLOCKED job design if asynchronous |
| `GO-Q15` | API schemas/reason codes/endpoint/authz, source contract | `API-Q16; SEC-Q16` | BLOCKED API production |
| `GO-Q16` | UI explanation, uncertainty, non-automatic action copy | `DS-Q12; SS-Q16` | BLOCKED `SCR-18` release |
| `GO-Q17` | Risk-aware optimization/Monte Carlo coupling | `Q-07; Q-08` | BLOCKED stochastic optimizer |
| `GO-Q18` | Independent validation, owner/reviewer sign-off and release rollback | `TS-Q10; DR-024` | BLOCKED R3 production approval |

## 27. Definition of Ready dan Definition of Done — `EP-S10-01`

### 27.1 Definition of Ready (production implementation)

- [ ] `FR-017` / `EP-S10-01` benar-benar disetujui untuk rilis R3; dependensi `EP-S08-02` dan `TK-S06-03` telah terverifikasi sesuai [`PROGRESS.md`](../planning/PROGRESS.md).
- [ ] `Q-07`, `FDR-Q11`, `CALC-Q12`, `BE-Q11` diselesaikan, tujuan/constraints/priority, tie-break dan shortfall policy memiliki versi serta owner approval.
- [ ] `Q-11` funded attribution dan cash-flow capacity yang dipakai dapat ditelusuri, diaudit, dan diuji; `Q-04` exact money; `Q-06` snapshot; `Q-05` inflation/real jika relevan.
- [ ] Contoh feasible + infeasible independent oracle, termasuk `FTC-ALLOC-01`–`FTC-ALLOC-08`, sign-off dan tolerance/solver proofs tersedia.
- [ ] Cek repository sebenarnya: Go package, migration, OpenAPI, CI, Flutter modules yang relevan; jangan mengarang file/command.
- [ ] Model threats/authz/retention, accessibility dan bounded compute sudah disetujui untuk cakupan yang dipilih.
- [ ] Task dipecah berdasarkan dependency dan pembuktian, bukan mengejar satu generator solusi besar sekaligus.

### 27.2 Definition of Done (hanya berdasarkan evidence)

- [ ] Pure Go engine yang mengonsumsi model typed approved; tidak bergantung pada UI, HTTP/DB, atau actual posting.
- [ ] Model validator, solver, independent verifier, explanation/provenance sudah diimplementasikan; failure handling memisahkan infeasible vs unknown vs invalid.
- [ ] Golden `CALC-REF-13`, `FTC-ALLOC-*` dan tests lokal yang relevan **dijalankan** dan hasilnya dicatat bersama command, commit, environment, expected/actual; `BLOCKED` tidak dilaporkan PASS.
- [ ] Property tests, solver optimum oracle (untuk objective approved), concurrency, performance benchmark dan resource caps lulus sesuai kriteria yang ditetapkan.
- [ ] PostgreSQL transactional storage, owner-scoped API, retry/cancel bila dipakai, no cross-user leak, retensi/cleanup teruji.
- [ ] Flutter `SCR-18` mencakup validasi input, accessible feasible/infeasible/unknown/stale states, goal shortfall dan planned-not-funded copy.
- [ ] Tidak ada perubahan actual balance, transaksi, net worth atau actual-funded akibat optimasi/simpan draft; negative tests membuktikan.
- [ ] API contract dan migrations yang disetujui sinkron; QA, security/privacy, financial reviewer, Product Owner menandatangani release gate.
- [ ] Status `TASKS.md`/`PROGRESS.md` diubah hanya setelah evidensi repo tersedia; dokumentasi ini sendiri tidak membuktikan `DONE`.

Jika salah satu gate wajib gagal/terblokir, laporkan `PARTIAL`, `BLOCKED`, atau `UNVERIFIED` sesuai workflow. **Jangan mengganti status test yang tidak dijalankan menjadi PASS.**

## 28. Agent implementation sequence / staged safe work

| Step | Tujuan | Apa yang boleh dilakukan sekarang | Gate sebelum langkah berikutnya |
|---|---|---|---|
| 1 | Trace requirement | Baca `FR-017`, `CS-013`, `Q-07/Q-11`, `FTC-ALLOC-*`, task dependencies | Ketidaksesuaian dicatat |
| 2 | Check reality | Inspeksi struktur repo/versi, actual tests, approval status | Jangan mengarang command/schema |
| 3 | Independent fixture math | Implement isolated checker/witness pada fixture sintetis tanpa data user | Pass fixture + property, no production claim |
| 4 | Approve model | Review objective, funding policy, capacity, solver, tolerance dan trade-off | Approval dengan tanggal/owner masuk Decision Log |
| 5 | Implement pure Go contract | Model typed, solver, exact verifier, bounded execution | Unit/property/oracle checks |
| 6 | Integrate backend | Owner scopes, safe persistence, contract/schema actual | Integration/security/race/retry tests |
| 7 | Integrate Flutter | `UF-26`, `SCR-18` states, accessible explanation | Widget/integration UX tests |
| 8 | R3 release review | Model audit, negative tests, benchmark, privacy, provenance, rollback | Evidence sign-off, no silent blocker |

**Jangan mulai membuat background queue hanya demi memenuhi roadmap apabila solver belum terbukti berat.** `EN-005` adalah kemampuan kondisional yang memerlukan alasan dan review. Tugas R0/R1/R2 harus tetap aman walau optimizer belum dibuat.

## 29. Anti-patterns dan red flags (hard stop)

- Mengembalikan pembagian `25%/25%/50%` atau ranking greedy sebagai **'optimal'** tanpa objective dan bukti.
- Mengabaikan salah satu goal yang tidak tercapai, menambahkan return atau inflasi default tanpa persetujuan.
- Menggunakan monthly income gross sebagai capacity tanpa biaya wajib/reserve/utang.
- Menggunakan saldo cash penuh sebagai `P_g` untuk setiap goal (double funding), atau menyamakan plan dengan saldo aktual.
- Menambahkan saldo positif dalam proyeksi tanpa sumber, berutang implisit untuk menutup cash gap, atau menganggap saldo aset nonlikuid sebagai cash.
- Mengubah goal actual/transfer saat user mengeklik `Cari alternatif`/`Simpan rencana`.
- Memberi status `INFEASIBLE` hanya karena solver timeout, atau mengklaim optimum pada feasible incumbent tanpa proof.
- Memilih solver package/worker queue dan mengunci API endpoint/migration tanpa keputusan terverifikasi.
- Mencampur nominal dan real, USD dan IDR, awal dan akhir bulan, atau mengganti rounding diam-diam.
- Menampilkan chart trend, success percentage, atau klaim keamanan dana dari model yang belum tervalidasi.
- Menempatkan data sensitif di logs dan membiarkan pengguna lain menebak `run_id` untuk membaca hasil.
- Melaporkan `DONE` walau belum ada kode yang diuji, atau menandai demo sintetis sebagai rekomendasi finansial pengguna.

## 30. Reference glossary dan next review

| Reference | Apa yang relevan | Kapan dibaca |
|---|---|---|
| `DR-011/012/024` | Planned ≠ funded, no actual posting, advanced model approval | Setiap perubahan alokasi |
| `CS-013` | Domain constraints and basic recurrence | Sebelum solver design |
| `CALC-REF-13`, `FTC-ALLOC-01..08` | Exact reference numbers and negative test | Sebelum implementasi reference oracle |
| `Q-07` | Objective/priority/constraints | Sebelum production solver |
| `Q-11` | Goal funding exclusivity | Sebelum `P_g` nyata |
| `Q-06` | Baseline and stale/rebase | Sebelum saved results |
| `Q-08` | Monte Carlo model, risk integration | Hanya jika integrasi stokastik disetujui |
| `EN-005` | Worker when needed | Sesudah benchmark/approval |
| `UF-26` / `SCR-18` | UX, goal explanations and save draft | Saat Flutter feature work |
| `EP-S10-01` | R3 optimizer epic | Perencanaan implementation tasks |

**Unresolved:** Pilihan fungsi tujuan, capacity, funded-source dan reserve, optimizer algorithm/proof, money precision, timing, cache/run lifecycle, API schema, performance limits dan user disclosures tetap memerlukan keputusan yang dicatat. Tabel test dan *proposed* field merupakan standar review, **bukan bukti bahwa aplikasi atau hasil produksi sudah ada**.

---

## Lampiran A. Pseudocode independent constraint verifier

Di bawah ini bukan kode kompilasi dan tidak menggantikan actual `CS-013` setelah approval:

```text
verify(plan, problem):
  require plan.goal_set == problem.eligible_goal_set
  for each t in 1..H:
    require all x[g,t] >= 0 and exact_money(x[g,t])
    require sum_over_goals(x[g,t]) <= S[t] in exact units
    require reserve_constraints_satisfied(t)
  for each g:
    require initial_funding_eligible_and_exclusive(P[g])
    compute projected_G[g,d[g]] using APPROVED timing/return/rounding
    if hard_goal(g): require projected_G[g,d[g]] >= T[g]
    record projected_G, shortfall and constraints
  require actual_ledger_unchanged
  return validated_feasible_evidence  # NOT optimality proof
```

**Verifier** dan **solver** tidak boleh hanya menggunakan perhitungan yang sama persis bila tujuannya memvalidasi implementasi secara independen; setidaknya fixture dan agregasi harus dapat dihitung ulang dari model yang terpisah. Untuk optimality, bandingkan dengan oracle independent pada problem kecil dan catat proof/gap sesuai solver yang disetujui.

## Lampiran B. Sample handoff record untuk AI agent

```yaml
# EXAMPLE ONLY; do not mark as executed.
feature: FR-017
task: EP-S10-01
release: R3
phase: SPECIFICATION_REVIEW
status: BLOCKED
blockers:
  - Q-07: objective/constraints not approved
  - Q-11: funded attribution for real user data unresolved
implementation_evidence: NONE
test_evidence:
  FTC-ALLOC-01: NOT_RUN
  FTC-ALLOC-02: NOT_RUN
  FTC-ALLOC-03: NOT_RUN
  GOT-001: NOT_RUN
notes:
  - Synthetic reference proves only the described narrow feasibility instance.
  - No claim about optimality or production readiness.
next_action: Obtain approved model choices, create implementation tasks, run independent tests.
```

## Lampiran C. Documentation cross-review and updates required later

- Dokumen ini merinci R3 `FR-017` dan tidak mengubah requirement R0–R2. Semua id primary tetap dari sumber sebelumnya.
- Dokumen yang dibuat sebelumnya dapat menyebut `GOAL-OPTIMIZER.md` sebagai **planned**; setelah file ini benar-benar ditambahkan ke repository, revisi status **keberadaan file** pada indeks/roadmap tanpa menyetujui objective maupun menandai feature DONE.
- Dokumen berikutnya dalam target 37 adalah `docs/operations/DEPLOYMENT.md`. Sumber operasi dan environment masih memerlukan audit repository sebelum perintah deployment dinyatakan berjalan.
- Invarians domain dan security berlaku tanpa menunggu optimization model selesai; kepatuhan model tidak menggantikan pengujian keamanan atau correctness aktual.
- Setiap perubahan output numerik, optimization criteria, atau data lineage setelah approval memerlukan version bump, independent regression tests, dan pembaruan Decision Log.

