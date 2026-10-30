# Esokira — Product Requirements Document (PRD)

> **Dokumen:** `docs/product/PRD.md`  
> **Versi:** `0.1.0`  
> **Tanggal penyusunan:** 26 Oktober 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Dokumen induk:** [`PRODUCT-VISION.md`](./PRODUCT-VISION.md) v0.1.0  
> **Turunan:** [`FEATURE-MATRIX.md`](./FEATURE-MATRIX.md)  
> **Peruntukan:** Product Owner, product/design engineer, backend/frontend developer, AI coding agent

## 0. Status dan kebijakan implementasi

PRD ini adalah **spesifikasi kebutuhan produk awal**, diturunkan dari Product Vision yang telah direview secara internal. Keduanya **belum disetujui secara eksplisit oleh Product Owner**. Jangan menganggap roadmap atau acceptance criteria sementara sebagai izin untuk memulai seluruh pengembangan.

- **Decided:** Flutter, Go, PostgreSQL, monorepo dengan `/frontend` dan `/backend`.
- **Proposed:** nama kerja Esokira, pengguna Indonesia/IDR, modular monolith, REST/OpenAPI, input keuangan manual untuk MVP, fase R0–R4.
- **TBD:** pilihan autentikasi, kebijakan presisi dan pembulatan uang, metodologi model investasi, deployment, serta hal teknis lain pada bagian `Open Decisions`.
- **Blocking principle:** fitur finansial **tidak boleh dinyatakan siap rilis** tanpa spesifikasi perhitungan yang telah direview dan contoh hasil uji terverifikasi.

**Urutan sumber keputusan:** Product Vision mengikat tujuan/batas produk; PRD mendefinisikan behavior dan acceptance; `FEATURE-MATRIX.md` hanya mengatur dependensi/prioritas/status; spesifikasi financial domain (disusun berikutnya) mengikat aturan angka; kontrak OpenAPI/migrasi SQL (setelah ada) mengikat bentuk endpoint dan skema yang diterapkan. Jika bertentangan, buat issue/ADR dan minta keputusan, bukan memilih sendiri.

## 1. Ringkasan produk dan outcome

**Esokira** adalah aplikasi mobile perencanaan keuangan pribadi berbasis keadaan aktual, financial goals, simulasi skenario hidup, analisis risiko, serta rekomendasi alternatif tindakan yang transparan. Esokira **bukan aplikasi untuk mengirim uang atau melakukan trading**.

### Outcome pengguna

| ID | Outcome terukur dari perspektif pengguna | Pilar visi |
|---|---|---|
| O-01 | Bisa mengetahui gambaran aset, utang, kas, dan keterbatasan data yang disimpan. | V-01 |
| O-02 | Bisa membuat tujuan, menghitung kebutuhan kontribusi, dan memantau progres. | V-02 |
| O-03 | Bisa membandingkan skenario masa depan tanpa mengubah riwayat aktual. | V-03 |
| O-04 | Bisa memahami risiko dan konflik antar tujuan dari asumsi yang dapat dilihat. | V-04 |
| O-05 | Bisa meninjau alternatif tindakan serta alasan perhitungannya. | V-05 |

Outcome ini adalah arah produk; target adopsi/retensi belum ditetapkan melalui riset pengguna.

## 2. Target pengguna, platform, dan batasan

- **Pengguna awal (Proposed):** mahasiswa tingkat akhir, fresh graduate, dan pekerja muda di Indonesia yang mengelola keuangan pribadi.
- **Platform awal (Decided):** mobile Flutter; backend Go; database PostgreSQL; monorepo `/frontend`, `/backend`.
- **Locale awal (Proposed):** Bahasa Indonesia, IDR sebagai mata uang pertama, format tanggal dan angka lokal. Dukungan multi-currency belum dijanjikan.
- **Sumber data R1 (Proposed):** input manual. Integrasi rekening/broker/e-wallet tidak wajib untuk MVP.
- **Hak pengguna:** melihat, memperbarui, mengekspor, dan menghapus data sesuai kebijakan retensi yang harus ditetapkan.
- **Out of scope R1–R3:** pembayaran, trading, pemberian kredit, rekomendasi produk investasi spesifik sebagai jasa profesional, serta akuntansi bisnis/enterprise.

## 3. Definisi fase rilis dan prioritas

| Release | Fokus | Exit gate minimum |
|---|---|---|
| **R0 — Foundation** | Monorepo, API/service skeleton, migrasi DB, identitas, testing, keamanan dasar. | Build/test dasar berjalan, isolasi data diuji, kerangka finansial dapat diuji. |
| **R1 — MVP** | Profil, akun, transaksi, aset/utang, net worth, financial goals, compound, goal forecast, data control. | Pengguna bisa menuntaskan alur actual → goal → calculation → saved progress tanpa dummy permanen; pengujian finansial wajib lulus. |
| **R2 — Planning V1** | Budget, life events, deterministic projection, scenario compare, cash-flow forecast, stress test, insights, reminders. | Dua skenario dapat dibandingkan; tidak memutasi aktual; input dan asumsi dapat ditinjau. |
| **R3 — Advanced** | Goal optimizer, Monte Carlo, retirement/FIRE, Financial GPS; worker bila diperlukan. | Model tervalidasi, hasil reproducible, ketidakpastian dijelaskan, perhitungan berat terkendali. |
| **R4 — Discovery backlog** | Shared goals dan portfolio planning. | Tidak mempunyai komitmen build hingga discovery dan privacy review selesai. |

**Prioritas:** `P0` wajib untuk MVP; `P1` setelah core stabil; `P2` advanced; `P3` discovery / belum disetujui. Label release adalah **rencana dependensi**, bukan tenggat waktu.

## 4. Core user journeys

### UJ-01 — Memulai rencana pertama (R1)

1. Pengguna masuk atau mendaftar, mengetahui cara data disimpan.
2. Mengisi profil minimum, membuat akun finansial, dan memasukkan saldo awal atau informasi finansial dasar.
3. Membuat goal (jumlah, target waktu, prioritas).
4. Menjalankan perhitungan kontribusi dan melihat asumsi.
5. Menyimpan goal dan melihat ringkasan; tidak ada transaksi uang yang dieksekusi.

**Kondisi gagal:** input tidak valid, tidak ada data cukup, koneksi gagal, atau akses tidak sah harus diberi error yang dapat dipahami dan tidak mengubah saldo.

### UJ-02 — Memperbarui keadaan aktual (R1)

1. Pengguna menambahkan pemasukan, pengeluaran, atau transfer.
2. Sistem memperbarui riwayat dan indikator keuangan berdasarkan aturan posting yang konsisten.
3. Dashboard dan progress tujuan mencerminkan data aktual, **tidak** memakai hasil proyeksi sebagai fakta.

### UJ-03 — Membandingkan dua masa depan (R2)

1. Pengguna membuat skenario dari baseline tertentu.
2. Menambahkan life event atau mengubah asumsi gaji/pengeluaran/waktu goal.
3. Sistem menghitung kedua skenario; menampilkan perbedaan, asumsi, dan risiko.
4. Pengguna dapat menyimpan versi skenario tanpa mengedit transaksi aktual.

### UJ-04 — Mengevaluasi rencana kompleks (R3)

1. Pengguna memasukkan beberapa goal dan constraint alokasi.
2. Sistem mendeteksi apakah seluruh target feasible.
3. Pengguna menjalankan simulasi risiko pada skenario yang dipilih.
4. Financial GPS menawarkan opsi perubahan setoran, target, tenggat, atau prioritas, dengan trade-off yang dapat diuji kembali.

## 5. Katalog functional requirements

Setiap **FR-ID** wajib unik dan tidak berubah ketika fitur dipindah rilis. Deskripsi di bawah adalah behavior produk; detail rumus/DB/API dirujuk ke dokumen teknis/domain yang akan dibuat kemudian.

### R1 — MVP (P0)

#### FR-001 — Identity, session, dan isolasi data

**Tujuan:** pengguna dapat mengakses hanya data finansial yang berhak ia kelola.

**Acceptance criteria:**

- Pengguna dapat masuk, mempertahankan sesi secara aman, dan keluar; penyedia autentikasi final belum ditetapkan.
- Backend memvalidasi identitas **pada setiap akses data pribadi**; manipulasi `user_id` atau ID resource orang lain tidak memberikan akses.
- Data keuangan tidak dikirim ke log biasa, analytics, atau error message secara tidak sengaja.

#### FR-002 — Financial profile dan onboarding

**Tujuan:** mengumpulkan informasi minimum untuk membuat rencana personal.

**Acceptance criteria:**

- Pengguna dapat mengisi dan mengubah profil keuangan dasar yang diperlukan; field wajib/opsional didokumentasikan sebelum implementasi.
- Input sensitif mempunyai penjelasan tujuan, validasi format, dan fallback bila profil belum lengkap.
- Tidak ada simulasi yang mengisi angka finansial pribadi dengan default tersembunyi tanpa label asumsi.

#### FR-003 — Akun finansial dan saldo awal

**Tujuan:** mencatat sumber saldo (misalnya uang tunai/rekening) yang dimiliki pengguna.

**Acceptance criteria:**

- Pengguna dapat membuat, melihat, dan memperbarui akun miliknya; kepemilikan divalidasi backend.
- Saldo awal dan transaksi tidak menyebabkan perhitungan ganda; definisi posting dijelaskan di domain spec.
- Perubahan data memiliki validasi dan menghasilkan ringkasan saldo yang konsisten.

#### FR-004 — Pemasukan, pengeluaran, dan transfer

**Tujuan:** mencatat arus uang aktual secara manual.

**Acceptance criteria:**

- Pengguna dapat membuat, melihat, mengubah, dan menghapus/membatalkan catatan sesuai kebijakan audit yang akan diputuskan.
- `income`, `expense`, dan `transfer` dibedakan; transfer antar-akun sendiri tidak dihitung sebagai biaya atau pendapatan eksternal.
- Aplikasi menolak nominal/tanggal/akun yang tidak valid, mencegah posting sebagian pada transfer, serta membatasi data berdasarkan user.

#### FR-005 — Aset dan kewajiban

**Tujuan:** mencatat aset dan utang untuk gambaran kekayaan bersih.

**Acceptance criteria:**

- Pengguna dapat mencatat penilaian aset, saldo kewajiban, dan tanggal penilaiannya.
- Saldo akun yang telah termasuk aset finansial tidak dihitung dua kali sebagai aset terpisah.
- Data yang belum diperbarui diberi penanda/tanggal penilaian, bukan disajikan sebagai nilai real-time.

#### FR-006 — Financial dashboard dan net worth

**Tujuan:** memperlihatkan kondisi aktual yang dapat diperiksa pengguna.

**Acceptance criteria:**

- Tersedia total aset, total kewajiban, dan `net worth = aset − kewajiban`; net worth negatif juga ditampilkan dengan benar.
- Pendapatan/pengeluaran ditampilkan berdasarkan periode yang jelas; transfer tidak menambah pengeluaran.
- Widget menunjukkan sumber data dan status *last updated*; proyeksi ditandai terpisah dari saldo aktual.

#### FR-007 — Financial goals

**Tujuan:** menyimpan tujuan beserta target dan progresnya.

**Acceptance criteria:**

- Goal memiliki nama, target nominal, target tanggal, prioritas, dan kondisi awal/progres yang tidak ambigu.
- Pengguna dapat membuat, mengubah, menandai tercapai, dan mengarsipkan goal.
- Angka kontribusi yang *direncanakan* berbeda dari kontribusi yang *benar-benar tercatat*.

#### FR-008 — Compound interest calculator

**Tujuan:** menghitung ilustrasi pertumbuhan dana yang transparan.

**Acceptance criteria:**

- Mendukung modal awal, kontribusi berkala, horizon, serta rate sesuai definisi di `CALCULATION-SPECS.md` yang belum disusun.
- Menangani rate 0, durasi 0, nilai input tidak valid, serta perbedaan satuan waktu tanpa crash atau hasil menyesatkan.
- Menampilkan asumsi, komponen total setoran vs estimasi pertumbuhan, dan label **simulasi—bukan jaminan**.
- Implementasi **diblokir** untuk rilis sampai konvensi setoran/rate/pembulatan dan golden cases disetujui.

#### FR-009 — Goal contribution & feasibility forecast

**Tujuan:** memperkirakan kontribusi yang diperlukan untuk suatu target.

**Acceptance criteria:**

- Berdasarkan goal, saldo yang dialokasikan, tanggal, dan asumsi, sistem menampilkan estimasi setoran diperlukan serta apakah kapasitas pengguna mencukupi.
- Untuk kondisi tidak feasible, UI mengungkap shortfall tanpa secara diam-diam mengubah target.
- Rumus, kebijakan pembulatan, dan kondisi tepi merujuk domain/calculation specs yang disetujui.

#### FR-021 — Data export dan deletion controls

**Tujuan:** memberi kontrol dasar atas data pribadi.

**Acceptance criteria:**

- Pengguna dapat meminta ekspor data aktual dan goals dalam format terdokumentasi; skenario ditambahkan ketika fitur tersebut ada.
- Pengguna dapat mengajukan penghapusan akun/data melalui alur yang aman dan mengonfirmasi tindakan destruktif.
- Retensi backup, kegagalan proses, dan batasan penghapusan harus dinyatakan secara transparan sebelum rilis; tidak ada janji hapus-seketika dari seluruh sistem tanpa implementasi.

### R2 — Planning V1 (P1)

#### FR-010 — Budgets dan recurring transactions

- Pengguna dapat mengatur anggaran per periode/kategori serta mencatat transaksi berulang.
- Proses transaksi berulang bersifat idempotent; tidak menduplikasi posting bila dijalankan ulang.
- Perbandingan budget vs actual mengecualikan transfer internal dan menyebutkan periode data.

#### FR-011 — Life event planning

- Pengguna dapat menambahkan event finansial hipotetis dengan tanggal dan dampak sekali bayar/berulang, jika sesuai spesifikasi.
- Event dikaitkan dengan **planning draft/default scenario** (model dasar yang dapat dibuat sebelum UI scenario comparison tersedia), bukan otomatis membuat transaksi aktual.
- FR-013 nanti menambahkan versioning dan comparison terhadap planning draft tersebut; event tetap tidak memutasi actuals.
- Konflik atau kekurangan parameter ditandai, tidak diisi sembarangan.

#### FR-012 — Deterministic financial projection

- Engine menghitung keadaan per periode berdasarkan snapshot awal, pemasukan, pengeluaran, kewajiban, kontribusi, dan life events.
- Hasil dapat direproduksi dari input/asumsi/versi engine yang sama.
- Output menjelaskan horizon, rate, inflasi/biaya jika digunakan, serta keterbatasan model; tidak memutasi actuals.

#### FR-013 — Scenario builder, versioning, dan comparison

- Pengguna dapat membuat salinan skenario dari baseline/versi yang jelas, mengubah asumsi, dan membandingkan sedikitnya dua hasil.
- Identitas baseline, asumsi berbeda, versi, dan waktu penghitungan ditampilkan atau dapat diperiksa.
- Mengedit/menghapus skenario tidak mengedit akun/transaksi asli atau skenario lain.

#### FR-014 — Future cash-flow forecasting

- Sistem menampilkan estimasi arus masuk, arus keluar, dan posisi kas per periode dari projection engine.
- Periode dengan proyeksi kas negatif/shortfall diberi penanda yang dapat ditelusuri ke asumsi penyebabnya.
- Nilai historis aktual dan nilai forecast menggunakan label visual berbeda.

#### FR-015 — Financial stress test

- Pengguna dapat mensimulasikan gangguan finansial terparameterisasi, seperti penghasilan turun atau biaya mendadak.
- Sistem menampilkan perubahan ketahanan kas dan dampak terhadap goal, berdasarkan input yang diketahui.
- Hasil tidak menyebut "aman" secara absolut dan tidak mengubah transaksi aktual.

#### FR-016 — Explainable rule-based insights

- Insight berasal dari aturan yang terdokumentasi dan data/hasil engine; setiap insight menyertakan alasan serta asumsi relevan.
- Jika data kurang, sistem menyatakan *insufficient data* alih-alih membuat penilaian personal tanpa dasar.
- Tidak memberikan rekomendasi produk investasi spesifik atau keputusan wajib; AI generatif tidak diperlukan.

#### FR-022 — Pengingat dan notification preferences

- Pengingat target/peninjauan rencana hanya aktif melalui pilihan pengguna dan dapat dimatikan.
- Notifikasi lock screen tidak membocorkan angka saldo/utang sensitif secara default.
- Kegagalan pengiriman tidak mengubah data finansial; frekuensi dan delivery provider masih TBD.

### R3 — Advanced (P2)

#### FR-017 — Multi-goal allocation optimizer

- Pengguna memasukkan kapasitas kontribusi, tujuan, prioritas, tenggat, dan constraint yang terdokumentasi.
- Optimizer mengembalikan alokasi per periode atau status infeasible beserta trade-off/shortfall; **tidak** menyembunyikan tujuan yang gagal.
- Objective function, perlakuan prioritas, dan batasan matematis harus direview sebelum implementasi; output reproducible.

#### FR-018 — Monte Carlo risk simulation

- Pengguna dapat menjalankan simulasi variasi hasil untuk skenario tertentu dengan input statistik yang disebutkan secara eksplisit.
- Output meliputi distribusi/rentang yang relevan, definisi kondisi berhasil/gagal, jumlah lintasan, serta *model success rate*—bukan peluang pasti dunia nyata.
- Simulasi dengan seed + parameter + versi algoritma yang sama menghasilkan ringkasan yang dapat direproduksi sesuai toleransi numerik yang ditentukan.
- Distribusi return, korelasi, inflation modeling, *sequence risk*, jumlah iterasi, dan validasi model **TBD sampai spesifikasi risiko disetujui**.

#### FR-019 — Retirement / FIRE planner

- Pengguna dapat menyatakan usia rencana pensiun, kebutuhan pengeluaran, sumber pendapatan, serta horizon penarikan.
- Hasil membedakan nominal/riil, menunjukkan risiko dana tidak mencukupi, dan memperlihatkan asumsi penarikan.
- Tidak menganggap aturan penarikan tetap (misalnya 4%) sebagai jaminan aman bagi semua pengguna.

#### FR-020 — Financial GPS / adaptive roadmap

- Untuk rencana yang tidak feasible atau berisiko, sistem dapat menyajikan alternatif yang dapat dibandingkan (tenggat, target, kontribusi, atau prioritas).
- Setiap alternatif memakai projection/risk engine yang sama, mencantumkan trade-off serta asumsi, dan membutuhkan keputusan pengguna.
- Tidak menjalankan transfer uang atau mengubah keadaan aktual secara otomatis.

### R4 — Discovery backlog (P3; **bukan komitmen build**)

#### FR-023 — Shared financial goals

Hipotesis: beberapa pengguna dapat mendanai tujuan bersama dengan izin yang berbeda. **Acceptance criteria, akses data, consent, invitation, dan pencabutan akses belum ditetapkan; tidak boleh diimplementasikan sebelum discovery dan privacy review.**

#### FR-024 — Portfolio planning

Hipotesis: pengguna dapat memodelkan beberapa kelas aset dan alokasi investasi. **Sumber harga, definisi portofolio, atribusi hasil, risiko, dan batas rekomendasi belum ditetapkan; tidak boleh diimplementasikan sebelum spesifikasi dan penilaian kepatuhan.**

## 6. Engineering enablers (bukan fitur pengguna)

| ID | Kebutuhan | Gate |
|---|---|---|
| EN-001 | Monorepo `/frontend`, `/backend`; build, lint, format, dan CI dasar masing-masing. | R0 |
| EN-002 | Migrasi PostgreSQL, identitas owner resource, constraint, transaction safety, serta tes isolasi data. | R0 |
| EN-003 | Kontrak API ter-versioning, error standar, validasi request, authorization middleware, dan log tanpa data sensitif. | R0 |
| EN-004 | Financial engine terpisah dari transport/storage dan test harness untuk golden cases; belum boleh mengarang spesifikasi numerik. | R0; sebelum FR-008/FR-009 |
| EN-005 | Infrastruktur background jobs dengan retry, idempotency, batas resource, cancel/timeout jika beban menuntut. | Sebelum FR-018 atau pemrosesan berat lain; solusi **TBD** |
| EN-006 | Config/secrets, observability, backup/restore test plan, dan prosedur deployment yang aman. | R0 baseline, diperluas sebelum publikasi |

Paket dan library spesifik (chi, pgx, sqlc, River, Riverpod, provider auth) **belum dianggap disetujui** hanya karena pernah direkomendasikan.

## 7. Non-functional requirements dan release guards

| ID | Aspek | Requirement yang dapat diuji |
|---|---|---|
| NFR-001 | Security & authorization | Semua API data pribadi memverifikasi identitas dan ownership; IDOR/cross-user tests wajib lulus. |
| NFR-002 | Financial correctness | Semua golden cases yang relevan lulus; urutan posting, presisi, rate, dan pembulatan terdokumentasi. |
| NFR-003 | Reproducibility | Proyeksi menyimpan/menampilkan input, versi engine, baseline, dan seed bila acak; model output dapat direproduksi. |
| NFR-004 | Performance | Ukur p50/p95 response dan memori pada beban uji yang disepakati; **SLA angka masih TBD**, tidak dikarang. |
| NFR-005 | UX & accessibility | Nilai uang terbaca dalam format lokal; label actual vs forecast jelas; error/empty/loading state tersedia; aksesibilitas diperiksa. |
| NFR-006 | Reliability | Pencatatan/transfer atomik, request/job aman terhadap retry, backup dan restore diuji sebelum production. |
| NFR-007 | Observability & privacy | Correlation ID dan error monitoring tanpa mengekspos nominal sensitif, token, password, atau PII berlebihan. |
| NFR-008 | Consent & lifecycle | Pemberitahuan penggunaan data, kebijakan retensi, ekspor dan penghapusan data diuji; kebutuhan hukum ditinjau sebelum publikasi. |
| NFR-009 | Quality gate | Backend unit/integration tests, frontend widget tests, API contract checks, migrasi dan keamanan sesuai fitur yang dirilis. |
| NFR-010 | Data boundary | Skenario, insight, dan prediksi tidak pernah menulis ke saldo atau transaksi aktual tanpa aksi eksplisit terpisah yang telah dispesifikasikan. |

## 8. Konsep data dan aturan pembatas

Konsep entitas: **User → Accounts/Transactions/Assets/Liabilities** untuk keadaan aktual; **Goals → Scenarios/Scenario Versions → Simulation Runs/Results** untuk perencanaan dan pengujian masa depan. Hubungan, constraints, dan tabel final belum diputuskan dalam dokumen database.

Aturan yang **wajib dipertahankan**:

1. Actual transaction, opening balance, transfer, asset valuation, liability, dan net worth memiliki definisi yang tidak boleh saling bertentangan.
2. Semua data pengguna mempunyai ownership dan diperiksa pada server.
3. Simulasi tidak mengubah actuals; baseline dan asumsi pada saat penghitungan dapat ditelusuri.
4. Nominal aktual memakai representasi presisi yang aman; **BIGINT rupiah vs NUMERIC dan aturan pembulatan masih harus diputuskan pada financial domain**.
5. Perbedaan `estimate`, `actual`, `nominal`, `real`, dan `model success rate` diwajibkan pada UI dan/atau output API yang relevan.
6. Catatan finansial sensitif tidak boleh menjadi telemetry umum atau data pelatihan AI secara implisit.

## 9. UX content, penjelasan hasil, dan error states

- Secara default gunakan bahasa yang dapat dipahami pengguna non-keuangan; istilah kompleks tampil sebagai detail lanjutan.
- Nilai aktual dan hipotetis dibedakan lewat label teks, bukan hanya warna.
- Simulasi harus menyediakan *assumption summary* yang dapat dibuka; nilai default wajib eksplisit.
- Jika kemampuan menabung kurang dari kebutuhan goal, tampilkan shortfall dan opsi, **bukan** memaksa status sukses.
- Jika tidak ada cukup data untuk sebuah rekomendasi/angka, tampilkan alasan dan cara melengkapi data.
- Koneksi gagal, authorization gagal, job pending/failed, dan hasil kedaluwarsa harus ditangani tanpa memalsukan kesuksesan.

## 10. Analytics dan pengukuran (usulan; perlu privacy review)

| Event konseptual | Untuk mengukur | Batasan |
|---|---|---|
| `first_goal_created` | Aktivasi awal / M-01 | Jangan kirim nama goal atau nominal mentah. |
| `first_projection_viewed` | Aktivasi rencana / M-01 | Jangan kirim input finansial pribadi. |
| `scenario_compared` | Penggunaan perencanaan / M-02, M-03 | Hanya metadata non-sensitif bila sah untuk dikumpulkan. |
| `plan_reviewed` | Retensi perencanaan / M-02 | Perlu definisi sesi dan izin yang jelas. |
| `calculation_test_result` | Kualitas engine / M-04 | Telemetry engineering, bukan data keuangan pengguna. |

Metrik M-01–M-05 didefinisikan pada Product Vision. **Tidak** menetapkan target numerik akuisisi/retensi tanpa baseline.

## 11. Dependensi dokumentasi sebelum implementasi

| Area | Dokumen yang harus tersedia sebelum scope tersebut dibangun |
|---|---|
| Semua kode | `AGENTS.md` minimal, arsitektur terkait, tugas dan acceptance yang disetujui. |
| Transaksi & net worth (FR-003–FR-006) | `FINANCIAL-DOMAIN-RULES.md`, `DATABASE-DESIGN.md`, test posting/transfer. |
| Compound & goal forecast (FR-008–FR-009) | `CALCULATION-SPECS.md`, `FINANCIAL-TEST-CASES.md`. |
| Scenario & projection (FR-011–FR-015) | `SCENARIO-ENGINE.md`, calculation specs, contract API, reproducibility cases. |
| Optimizer (FR-017) | `GOAL-OPTIMIZER.md` dan reference solutions, termasuk infeasible cases. |
| Monte Carlo (FR-018) | `MONTE-CARLO.md`, parameterisasi statistik, seeded tests, review keterbatasan model. |
| Financial GPS (FR-020) | Penjelasan aturan insight, hasil engine yang tervalidasi, dan review bahasa rekomendasi. |
| Data export/deletion (FR-021) | `SECURITY-PRIVACY.md`, retention policy, integration tests. |

## 12. Open decisions & blockers

| ID | Keputusan terbuka | Menghambat |
|---|---|---|
| Q-01 | Nama kerja, target Indonesia/IDR, dan prioritas MVP disetujui Product Owner? | Approval PRD; bukan penulisan draf. |
| Q-02 | Penyedia identitas: Firebase Authentication, OIDC, atau lainnya? | FR-001 dan detail security architecture. |
| Q-03 | Model saldo awal, posting transaksi/transfer, reversal, dan audit? | FR-003–FR-006. |
| Q-04 | Presisi uang, mata uang, satuan rate, tanggal efektif, pembulatan, setoran awal/akhir periode? | FR-008–FR-009 dan projection engine. |
| Q-05 | Kebijakan inflasi, biaya, return, utang, dan asumsi yang kosong? | FR-012–FR-019. |
| Q-06 | Definisi baseline skenario dan kebijakan snapshot/versioning? | FR-012–FR-013 dan reproducibility. |
| Q-07 | Objektif dan constraints goal optimizer? | FR-017. |
| Q-08 | Distribusi return, korelasi, Monte Carlo seed, batas runtime dan confidence communication? | FR-018. |
| Q-09 | Provider notifikasi, preferensi dan frekuensi? | FR-022. |
| Q-10 | Biaya hosting, backup/retensi data, offline/sync, dan kebutuhan legal review? | Production release. |
| Q-11 | Bagaimana kontribusi actual dikaitkan ke goal tanpa double counting akun/transaksi, dan bagaimana progress ditentukan? | FR-007, FR-009, FR-012. |

Jika blocker belum terjawab, agent boleh **membuat rencana atau prototipe terisolasi yang diberi label**, tetapi **tidak boleh mengklaim fitur production-ready**.

## 13. Definition of Ready / Definition of Done (minimum)

**Ready untuk implementasi:** FR-ID dan acceptance jelas; dependensi ready; desain data/API disepakati; spesifikasi perhitungan dan test reference tersedia bila numerik; risiko privasi ditinjau; task terukur dibuat.

**Done untuk fitur:** sesuai acceptance, unit/integration/UI tests relevan lulus dan dicatat, cross-user access diuji, tidak merusak modul lain, error/loading/empty state diuji, dokumentasi di-update, tidak ada blocker kritis tersembunyi. **UI selesai ≠ fitur selesai**.

**Rilis:** seluruh acceptance untuk FR yang discope ke rilis terpenuhi; known limitations dicantumkan; Product Owner menyetujui rilis. Status `Planned` di feature matrix tidak boleh otomatis diubah ke `Done` tanpa bukti tes dan review.

## 14. Traceability ke Product Vision

| Pilar Vision | FR yang terutama mendukung |
|---|---|
| V-01 — Financial Reality | FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, FR-021 |
| V-02 — Goal-Based Planning | FR-007, FR-008, FR-009, FR-010, FR-022, FR-023 |
| V-03 — Life & Scenario Simulation | FR-011, FR-012, FR-013, FR-014 |
| V-04 — Risk & Optimization | FR-015, FR-017, FR-018, FR-019, FR-024 |
| V-05 — Explainable Guidance | FR-016, FR-020 |

## 15. Review dan persetujuan

- [x] PRD mengacu pada seluruh lima pilar Vision (V-01–V-05).
- [x] Scope R1, R2, R3, dan backlog discovery dibedakan.
- [x] Requirement unik memiliki FR-ID stabil dan acceptance atau status discovery yang eksplisit.
- [x] Finance/security blockers ditandai, tidak diasumsikan selesai.
- [x] `actual ≠ simulated` dinyatakan dalam alur, data, dan acceptance.
- [x] Tidak ada target bisnis numerik yang diklaim sudah tervalidasi.
- [ ] **Product Owner menyetujui scope dan prioritas.**
- [ ] **Financial rules, auth, dan desain data yang dibutuhkan tiap modul telah disetujui sebelum coding.**

**Hasil review internal:** LULUS untuk menjadi sumber *draf Feature Matrix*. **Bukan izin otomatis untuk mengeksekusi seluruh backlog.**
