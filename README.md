# Esokira

> **Rencanakan hari esok dengan memahami kondisi keuangan hari ini.**
>
> Esokira adalah proyek aplikasi mobile untuk mencatat kondisi finansial, menyusun tujuan, membandingkan pilihan hidup, dan mengeksplorasi simulasi keuangan dengan asumsi yang dapat dijelaskan.

**Status proyek:** `DOCUMENTATION BASELINE / IMPLEMENTATION UNVERIFIED`  
**Versi dokumen:** `0.1.0`  
**Tanggal:** `2026-10-30` (Asia/Jakarta)  
**Status dokumen:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
**Bahasa utama dokumentasi:** Bahasa Indonesia, dengan istilah engineering dalam bahasa Inggris bila lebih tepat.  
**Panduan dokumentasi:** [docs/INDEX.md](docs/INDEX.md)

> [!IMPORTANT]
> README ini menjelaskan **rancangan dan rencana pengembangan**, bukan bukti bahwa aplikasi, API, migrasi, atau pipeline CI telah tersedia. Seluruh status implementasi harus diverifikasi terhadap repository yang benar-benar digunakan. Penyiapan artefak dokumentasi terpisah juga tidak membuktikan file-file tersebut telah di-commit ke monorepo.

## 1. Apa itu Esokira?

Esokira membantu pengguna menjawab pertanyaan seperti:

- Bagaimana posisi uang, aset, kewajiban, dan kekayaan bersih saya saat ini?
- Berapa kontribusi yang mungkin diperlukan untuk mencapai target finansial tertentu?
- Apa dampak biaya pendidikan, kendaraan, rumah, atau perubahan pekerjaan terhadap rencana?
- Jika asumsi berubah, apa perbedaan antar-skenario dan seberapa besar ketidakpastiannya?
- Ketika beberapa tujuan saling bersaing, pilihan serta *trade-off* apa yang perlu dipahami?

Sasarannya adalah pengalaman **mobile-first**, mudah dipahami, transparan tentang asumsi, dan berorientasi pada keputusan, bukan sekadar kalkulator angka. Segmen awal yang dibahas mencakup mahasiswa tingkat akhir, lulusan baru, dan pekerja muda; ini masih **hipotesis produk yang memerlukan validasi pengguna**, bukan hasil riset pasar yang sudah terbukti.

### Prinsip produk

1. **Realitas dan simulasi harus terpisah.** Skenario, proyeksi, dan rekomendasi tidak boleh mengubah saldo atau transaksi aktual.
2. **Ketepatan finansial lebih penting daripada tampilan.** Angka otoritatif mengikuti aturan domain, spesifikasi matematika, dan pengujian yang telah direview.
3. **Asumsi harus terlihat.** Return, inflasi, horizon waktu, kontribusi, biaya, dan keterbatasan model perlu dijelaskan sesuai relevansinya.
4. **Privasi adalah bagian dari desain.** Data keuangan privat hanya dapat diakses melalui autentikasi dan otorisasi yang valid.
5. **Tidak menjanjikan masa depan.** Forecast, probabilitas, dan saran bukan jaminan hasil keuangan pribadi.

Baca [Product Vision](docs/product/PRODUCT-VISION.md) dan [PRD](docs/product/PRD.md) untuk batasan produk dan acceptance criteria yang lengkap.

## 2. Ruang lingkup dan tahapan rilis

Rilis berikut adalah **urutan kerja yang direncanakan**, bukan jadwal yang disetujui atau klaim modul telah selesai.

| Fase | Sasaran | Contoh kemampuan | Status pada baseline dokumentasi |
|---|---|---|---|
| **S-00 - Planning gate** | Audit repository, ruang lingkup dan keputusan | Verifikasi kondisi kode, kontrak, prioritas, risiko dan blocker | Direncanakan; audit belum dibuktikan |
| **R0 - Foundation** | Fondasi engineering | Struktur Flutter/Go, PostgreSQL, security baseline, API foundation, test harness, CI | Direncanakan; implementasi `UNVERIFIED` |
| **R1 - MVP** | Financial reality dan goals | Onboarding, akun, transaksi, transfer, aset/utang, net worth, goals, compound calculator, data rights | Direncanakan; implementasi `UNVERIFIED` |
| **R2 - Planning V1** | Perencanaan dan skenario | Budget/recurring, life events, deterministic projection, scenario comparison, cash-flow forecast, stress tests, insights | Backlog bertahap; perlu keputusan lanjutan |
| **R3 - Advanced** | Analisis risiko dan optimasi | Goal optimizer, Monte Carlo, retirement/FIRE, Financial GPS, job execution | Backlog bertahap; model perlu validasi |
| **R4 - Discovery** | Penelitian fitur masa depan | Shared goals dan portfolio planning | **Bukan** komitmen build |

Roadmap rinci mengacu pada **13 delivery slices `S-00` sampai `S-12`**. Dua requirement `FR-023` dan `FR-024` ditempatkan pada discovery R4; jangan membuat implementasinya hanya karena disebut dalam dokumen.

- [Feature Matrix](docs/product/FEATURE-MATRIX.md) - cakupan, dependensi, dan fase `FR-*`.
- [Master Roadmap](docs/planning/MASTER-ROADMAP.md) - urutan pelaksanaan dan quality gates.
- [Task Backlog](docs/planning/TASKS.md) - ID serta penerimaan kerja `TK-*`/`EP-*`.
- [Progress Register](docs/planning/PROGRESS.md) - status aktual yang wajib memiliki bukti.

### Yang bukan tujuan MVP

Esokira **tidak dirancang pada fase awal** untuk memindahkan dana, menjalankan pembayaran, memperdagangkan investasi, menjadi bank, atau memberikan jaminan return. Integrasi bank, e-wallet, broker, dan market data bukan prasyarat MVP. Input data manual adalah baseline rancangan yang masih memerlukan konfirmasi lebih lanjut. Simulasi tidak boleh ditampilkan sebagai saran investasi profesional yang pasti benar.

## 3. Technology stack dan status keputusan

| Area | Teknologi/pendekatan | Status | Catatan |
|---|---|---|---|
| Mobile client | **Flutter / Dart** | **Dipilih pemilik proyek** | Target `frontend/`; detail state management/router belum final. |
| Backend | **Go** | **Dipilih pemilik proyek** | Target `backend/`; modular monolith adalah rancangan arsitektur yang perlu disejajarkan dengan keputusan final. |
| Primary database | **PostgreSQL** | **Dipilih pemilik proyek** | Schema, ledger, migration tools, tenant isolation policy perlu review. |
| Repository | **Monorepo** | **Dipilih pemilik proyek** | Area utama `frontend/` dan `backend/`. |
| API | REST + dokumentasi/kontrak OpenAPI | **Pendekatan rancangan** | Endpoint dan payload hanya final setelah kontrak disetujui. |
| Authentication | IdP, token validation, session/revocation | **TBD** | Jangan memilih provider otomatis. |
| Monetary representation | Exact money + rounding/time conventions | **Wajib akurat; detail TBD** | Penyimpanan nominal tidak boleh bergantung pada floating point. |
| Flutter libraries | State management, navigation, HTTP/storage | **TBD/PROPOSED** | Jangan menambah dependensi hanya karena disebut sebagai kandidat. |
| Go persistence/jobs | Driver, query tooling, queues, background workers | **TBD/PROPOSED** | Sesuaikan fase R0/R3 dan approval. |
| Deployment | Hosting, secrets, observability, backup, retention | **TBD** | Tidak ada cloud/region atau instruksi deploy yang disahkan oleh README ini. |

**Penting:** `DECIDED` untuk bahasa/framework/database tidak berarti keputusan implementasi turunannya sudah disetujui. Lihat [Decision Log](docs/planning/DECISION-LOG.md) dan bukti keputusan pemilik sebelum mengubah kontrak atau konfigurasi berisiko.

## 4. Struktur repository yang dituju

Berikut **peta lokasi kanonis**, bukan hasil inspeksi filesystem monorepo pengguna. File sumber aplikasi, build tools, kontrak mesin, dan migrasi hanya boleh dianggap ada setelah diperiksa.

```text
<esokira-repository>/
|-- README.md                         # Pintu masuk proyek ini
|-- AGENTS.md                         # Instruksi AI coding agent global
|-- frontend/
|   |-- AGENTS.md                     # Instruksi Flutter
|   `-- ...                           # Source, assets, tests: audit dahulu
|-- backend/
|   |-- AGENTS.md                     # Instruksi Go/PostgreSQL
|   `-- ...                           # Source, migrations, tests: audit dahulu
`-- docs/
    |-- INDEX.md                      # Peta dokumentasi
    |-- product/                      # Vision, PRD, feature matrix
    |-- financial/                    # Aturan domain, formula, golden cases
    |-- architecture/                 # Sistem, backend, frontend, DB, API
    |-- design/                       # Design system, user flows, screens
    |-- security/                     # Security dan privacy
    |-- development/                  # Workflow, testing, Definition of Done
    `-- planning/                     # Roadmap, tasks, progress, decisions
```

`contracts/openapi.yaml`, berkas migrasi, Docker/Compose, Makefile, CI workflow, dan manifest aplikasi adalah **kemungkinan artefak implementation-time**, bukan file yang dipastikan tersedia. Hindari membuat placeholder yang tampak seperti kontrak production tanpa tugas dan approval yang jelas.

Gunakan path yang tepat sesuai [Documentation Index](docs/INDEX.md). Paket ZIP satu dokumen tidak otomatis menyertakan dokumen lain.

## 5. Mulai membaca berdasarkan peran

| Jika Anda... | Mulai dari | Lalu baca |
|---|---|---|
| Baru memahami Esokira | [Product Vision](docs/product/PRODUCT-VISION.md) | [PRD](docs/product/PRD.md), [Feature Matrix](docs/product/FEATURE-MATRIX.md) |
| Menentukan pekerjaan berikutnya | [Master Roadmap](docs/planning/MASTER-ROADMAP.md) | [Tasks](docs/planning/TASKS.md), [Progress](docs/planning/PROGRESS.md), [Decision Log](docs/planning/DECISION-LOG.md) |
| Menggunakan AI coding agent | [Root AGENTS](AGENTS.md) | [Documentation Index](docs/INDEX.md), instruksi folder, [Development Workflow](docs/development/DEVELOPMENT-WORKFLOW.md) |
| Mengerjakan Flutter | [Frontend AGENTS](frontend/AGENTS.md) | [Frontend Architecture](docs/architecture/FRONTEND-ARCHITECTURE.md), [Screen Specs](docs/design/SCREEN-SPECS.md), [Design System](docs/design/DESIGN-SYSTEM.md) |
| Mengerjakan Go/database | [Backend AGENTS](backend/AGENTS.md) | [Backend Architecture](docs/architecture/BACKEND-ARCHITECTURE.md), [Database Design](docs/architecture/DATABASE-DESIGN.md), [API Guidelines](docs/architecture/API-GUIDELINES.md) |
| Mengerjakan angka finansial | [Financial Domain Rules](docs/financial/FINANCIAL-DOMAIN-RULES.md) | [Calculation Specs](docs/financial/CALCULATION-SPECS.md), [Financial Test Cases](docs/financial/FINANCIAL-TEST-CASES.md) |
| Memvalidasi keamanan | [Security & Privacy](docs/security/SECURITY-PRIVACY.md) | [Testing Strategy](docs/development/TESTING-STRATEGY.md), [Definition of Done](docs/development/DEFINITION-OF-DONE.md) |

**Hirarki otoritas berdasarkan topik:** PRD memiliki requirement; Domain Rules memiliki makna finansial; Calculation Specs memiliki formula; Financial Test Cases memiliki expected test oracle; Security & Privacy memiliki kontrol keamanan; implementasi API dan schema yang berlaku berasal dari **kontrak OpenAPI yang disetujui** dan **migrasi yang benar-benar diterapkan**, ketika keduanya ada. README dan INDEX berfungsi sebagai navigasi, bukan tempat mengubah kontrak domain.

## 6. Setup lokal: status dan cara menemukan perintah yang benar

**Petunjuk setup final belum tersedia.** File `docs/development/LOCAL-SETUP.md` masih direncanakan. Jangan mengasumsikan emulator Flutter, server Go, PostgreSQL, Docker/Compose, variabel environment, migrasi, atau command runner tertentu sudah dikonfigurasi.

### 6.1 Inspeksi awal sebelum menjalankan aplikasi

Dari **root repository yang sebenarnya**, periksa terlebih dahulu:

```sh
# Contoh inspeksi aman untuk shell Unix/Git Bash; bukan perintah build aplikasi.
pwd
git status --short
ls -la
ls -la frontend backend docs
```

Jika folder belum tersedia atau repository belum berupa Git checkout, laporkan keadaan itu, jangan mengarang output atau mengklaim error sebagai kelulusan. Pada Windows PowerShell gunakan padanan perintah yang sesuai. Periksa file nyata berikut **jika ada**:

| Yang perlu ditemukan | Mengapa diperlukan |
|---|---|
| `frontend/pubspec.yaml` | Memastikan proyek/dependensi Flutter yang aktual. |
| `backend/go.mod` | Memastikan modul Go dan lokasi menjalankan test. |
| File konfigurasi database/migrasi yang benar-benar ada | Mengidentifikasi schema dan cara setup DB yang sah. |
| Kontrak OpenAPI yang benar-benar ada | Memeriksa request/response yang disetujui. |
| `.env.example` atau template setara **jika tersedia** | Mengetahui konfigurasi tanpa membagikan secret. |
| Test runner/CI config yang aktual | Menentukan command dan coverage yang bisa diverifikasi. |

### 6.2 Pemeriksaan toolchain (jika sudah terinstal)

Perintah berikut hanya mengecek versi tool, **bukan** menjalankan Esokira:

```sh
flutter --version
go version
psql --version
```

Jika tidak terpasang, catat sebagai prasyarat setup. Jangan mengasumsikan versi, OS, package manager, database credentials, port, atau layanan cloud tertentu. Setelah inspeksi dan keputusan dependensi selesai, dokumentasikan **perintah yang sungguh berhasil dijalankan** di `docs/development/LOCAL-SETUP.md` beserta prasyarat, cara migrasi, test, dan troubleshooting.

## 7. Alur kerja untuk AI coding agent dan kontributor

**Default: satu task yang teridentifikasi dengan jelas, bukan membangun seluruh roadmap sekaligus.**

1. **Inspect:** cek folder, branch, working tree, konfigurasi, kode yang sudah ada, dan hasil tes yang dapat direproduksi; ubah `UNVERIFIED` hanya berdasarkan bukti.
2. **Select:** ambil task dengan ID tepat dari [Tasks](docs/planning/TASKS.md); untuk memulai gunakan jalur S-00 dan dependensi aktualnya.
3. **Route:** baca root/folder `AGENTS.md`, requirement pemilik, dokumen domain, [Decision Log](docs/planning/DECISION-LOG.md), dan pengujian yang relevan; tidak perlu memuat semua dokumen sekaligus.
4. **Gate:** pastikan acceptance criteria dan keputusan yang memengaruhi task sudah siap; jangan mengubah `PROPOSED`/`TBD` menjadi keputusan diam-diam.
5. **Plan:** batasi scope, file yang disentuh, dependensi, risiko, rollback, serta verifikasi yang akan dijalankan.
6. **Implement:** tulis perubahan sekecil mungkin dan pertahankan batas modul, keamanan, serta invarian finansial.
7. **Verify:** jalankan lint/build/unit/integration/financial/security tests sesuai dampak; catat perintah dan hasil sebenarnya.
8. **Review & handoff:** cek perubahan, dokumentasi, DoD, keputusan, dan update [Progress](docs/planning/PROGRESS.md) dengan evidence dan next step.

Referensi operasional: [Development Workflow](docs/development/DEVELOPMENT-WORKFLOW.md), [Testing Strategy](docs/development/TESTING-STRATEGY.md), dan [Definition of Done](docs/development/DEFINITION-OF-DONE.md).

### Status jangan disamakan

- **`UNVERIFIED`:** belum ada inspeksi/bukti yang cukup untuk menyatakan pekerjaan mulai atau selesai.
- **`NOT_STARTED`:** inspeksi membuktikan pekerjaan belum dimulai.
- **`IN_PROGRESS` / `PARTIAL`:** pekerjaan atau hasil parsial terverifikasi, belum memenuhi semua acceptance criteria.
- **`BLOCKED`:** prasyarat atau keputusan tertentu menghalangi scope yang dimaksud.
- **`DONE`:** syarat DoD relevan benar-benar lulus, direview, dan didukung bukti.
- **`NOT_RUN` (test):** bukan hasil `PASS`; tes belum dieksekusi.

Paket dokumentasi boleh lulus QA Markdown/ZIP **tanpa** membuktikan test aplikasi. Jangan menggabungkan dua jenis bukti ini.

## 8. Invarian finansial dan keamanan yang wajib dijaga

Perubahan kode tidak boleh melanggar aturan berikut, termasuk pada dummy/demo mode:

- **Actual != planned != simulated.** Asumsi, hasil skenario, dan kontribusi yang baru direncanakan tidak dapat mem-post saldo/transaksi aktual.
- **Planned goal contribution != funded progress.** Pendanaan tujuan nyata mengikuti kebijakan atribusi yang disetujui dan tidak boleh menghitung dana ganda.
- **Transfer antar-rekening sendiri bukan pendapatan/pengeluaran.** Operasi posting harus atomik, mempertahankan jejak audit dan berperilaku benar di bawah retry/concurrency.
- **Mutasi aktual wajib idempotent.** Timeout bisa menghasilkan outcome `unknown`; UI tidak boleh mengulang transaksi secara membabi buta atau menampilkan sukses tanpa bukti.
- **Uang aktual memerlukan representasi eksak.** Jangan memakai `double` sebagai sumber kebenaran saldo atau membuat aturan pembulatan secara sepihak.
- **Backend memegang otoritas.** Flutter tidak boleh mengakses PostgreSQL langsung; setiap resource privat diperiksa terhadap identitas yang diverifikasi server.
- **Simulasi punya provenance.** Baseline, asumsi, versi engine, dan seed bila relevan harus dapat ditelusuri; hasil basi tidak boleh diberi label masih terkini.
- **Data privat tidak boleh bocor.** Jangan meng-commit credentials, token, PII, rincian keuangan nyata, atau raw user data ke repo, screenshot, log, issue, prompt, dan fixture publik.
- **Uncertainty harus jujur.** Keluaran Monte Carlo, optimasi, dan Financial GPS harus menjelaskan keterbatasan serta tidak menjanjikan hasil.

Aturan normatif ada di [Financial Domain Rules](docs/financial/FINANCIAL-DOMAIN-RULES.md), [Security & Privacy](docs/security/SECURITY-PRIVACY.md), dan [PRD](docs/product/PRD.md).

## 9. API, database, dan batas perubahan kontrak

- REST API adalah batas komunikasi mobile/server yang direncanakan. Jangan membuat endpoint, body request, status code atau error schema production hanya dari contoh di Markdown.
- Jika `contracts/openapi.yaml` sudah tersedia dan disetujui, lakukan contract-first review untuk perubahan wire format; jika belum ada, proses approval/authoring adalah pekerjaan tersendiri.
- Skema aplikasi hidup di **migrasi PostgreSQL yang benar-benar diterapkan**, bukan di diagram konseptual saja. Jangan menjalankan migrasi destruktif ke DB berisi data pengguna tanpa otorisasi, backup, rollback, dan pengujian.
- Pastikan tenancy/ownership, transaksi database, idempotency dan integritas finansial diverifikasi dengan PostgreSQL sungguhan untuk perubahan kritis; mock saja tidak cukup.
- Untuk kalkulasi, backend Go harus dapat diuji secara deterministik terhadap `CS-*` dan `FTC-*`; jangan membuat hasil JavaScript/Dart yang berbeda sebagai sumber angka kedua.

## 10. Pengujian dan quality gates

Bukti minimum tergantung jenis perubahan, tetapi selalu gunakan kriteria yang relevan dari `DOD-G01`-`DOD-G10`:

| Jenis perubahan | Verifikasi yang biasanya relevan |
|---|---|
| Dokumen saja | Review substansi, tautan/path, ID, struktur Markdown, konsistensi status dan keputusan. |
| Flutter UI/state | Dart analysis, unit/widget/integration tests, error/empty/loading states, accessibility bila terkait. |
| Go business logic/API | Unit tests, input validation, API contract tests, authorization dan error-path tests. |
| PostgreSQL actual ledger/transfer | Migration/constraint tests, transaksi DB, concurrency, rollback, idempotency dan user isolation. |
| Financial calculations | Approved numeric golden vectors, boundary/property tests, precision/rounding dan reproducibility. |
| Simulations/async jobs | Seed/version replay, timeouts/cancellation, deduplication, stale-result handling, load limits yang disetujui. |
| Deployment | Secret management, observability, restore/rollback drills, privacy/retention review dan release gate. |

**Jangan mengarang:** coverage percentage, performance SLA, successful `go test`, successful `flutter test`, security clearance, atau keputusan owner. Simpan hasil konkret dan blocker di [Progress](docs/planning/PROGRESS.md).

## 11. Keputusan yang masih terbuka

[Decision Log](docs/planning/DECISION-LOG.md) mencatat **11 pertanyaan induk `Q-01`-`Q-11`** serta pertanyaan lokal tiap dokumen. Di antaranya:

| Kelompok | Keputusan yang perlu dipastikan sebelum task terkait |
|---|---|
| Identitas | IdP, login, verifikasi token, session/revocation (`Q-02`). |
| Keuangan aktual | Ledger, opening balance, koreksi, transfer, presisi serta timing (`Q-03`, `Q-04`). |
| Goals | Atribusi actual-funded vs planned contributions (`Q-11`). |
| Scenario | Returns/inflation, snapshot baseline, versioning, staleness (`Q-05`, `Q-06`). |
| Advanced | Optimizer objectives, stochastic assumptions, reproducibility (`Q-07`, `Q-08`). |
| Operasional | Consent, notifications, hosting, backups, retention dan data rights (`Q-09`, `Q-10`). |

Keputusan tertunda **tidak membuat seluruh proyek terblokir**. Lanjutkan task independen yang jelas; tahan hanya perubahan yang bergantung pada kebijakan belum disetujui. Persetujuan harus tercatat eksplisit, bukan ditafsirkan dari perintah umum seperti "gas lanjut".

## 12. Dokumentasi, kontribusi, dan pemeliharaan

- Mulai dari [docs/INDEX.md](docs/INDEX.md) agar tidak membuat duplikasi source of truth.
- Perubahan fitur menelusuri `FR-*` -> `DR/CS/FTC` bila finansial -> UI/API/DB/SEC bila relevan -> test -> `TK/EP` -> evidence `PROGRESS`.
- Ubah dokumen pemilik spesifikasi lebih dahulu jika kontrak yang **telah disetujui** berubah, kemudian sesuaikan test dan dependents. Jangan memakai README untuk mengganti PRD atau formula.
- Satu pull request/task sebaiknya memiliki scope terukur, deskripsi perubahan, risk assessment, hasil perintah pengujian dan referensi keputusan.
- Jangan menghapus histori audit atau menandai test yang tidak dieksekusi sebagai sukses. Perubahan kebijakan perlu approval dan catatan keputusan.
- Jangan masukkan secret atau data keuangan pengguna ke template environment, issue tracker, fixture atau example output.

### Snapshot inventaris dokumentasi

Saat README ini disusun terdapat **26 dokumen Markdown sebelumnya**, termasuk `docs/INDEX.md`. README adalah **dokumen ke-27** dalam target awal **37 dokumen**; **10 dokumen masih direncanakan**. Ini menghitung artefak yang telah disusun, **bukan** file yang terbukti tersedia dalam checkout pengguna. `docs/INDEX.md` masih mencerminkan baseline sebelum README (26/37) dan perlu diperbarui pada siklus pemeliharaan berikutnya agar snapshot tetap sinkron.

Dokumen yang masih direncanakan (jangan menganggap file sudah tersedia):

1. `docs/architecture/ADR/0001-TECH-STACK.md` - keputusan stack, trade-offs, dan status approval.
2. `docs/development/LOCAL-SETUP.md` - setup dan command aktual setelah audit repository.
3. `docs/features/TRANSACTIONS.md` - posting, transfer, koreksi, idempotency.
4. `docs/features/FINANCIAL-GOALS.md` - goals dan definisi actual funding.
5. `docs/features/LIFE-PLANNING.md` - event dan asumsi rencana R2.
6. `docs/features/SIMULATIONS.md` - experience, outputs, disclosure simulasi.
7. `docs/financial/SCENARIO-ENGINE.md` - deterministic projection dan snapshot/replay.
8. `docs/financial/MONTE-CARLO.md` - model stokastik dan validasi statistik.
9. `docs/financial/GOAL-OPTIMIZER.md` - objective, constraints, algoritma dan tests.
10. `docs/operations/DEPLOYMENT.md` - operational safety dan prosedur release.

Urutan yang masuk akal untuk pekerjaan berikutnya adalah **ADR stack -> LOCAL-SETUP**, dilanjutkan feature specs sesuai fase. Seluruh 10 dokumen **tidak harus selesai** sebelum memulai audit repository `S-00`; approval keputusan yang benar-benar memblokir task tetap wajib.

## 13. Informasi penting untuk pengguna

Esokira masih berada dalam tahap perencanaan dan dokumentasi. Model dan simulasi keuangan, ketika kelak diimplementasikan, adalah **alat bantu memahami asumsi dan konsekuensi**, bukan jaminan finansial atau nasihat investasi profesional yang dipersonalisasi. Hasil dapat berubah ketika data atau asumsi diperbarui; setiap output harus menjelaskan asal, versi, dan keterbatasannya.

**Tempat memulai:** [Documentation Index](docs/INDEX.md) untuk sumber dokumen; [Tasks](docs/planning/TASKS.md) untuk rencana kerja; [Progress](docs/planning/PROGRESS.md) untuk bukti pekerjaan; dan [Decision Log](docs/planning/DECISION-LOG.md) untuk keputusan yang masih membutuhkan persetujuan.

---

**Changelog**

| Versi | Tanggal | Perubahan | Verifikasi |
|---|---|---|---|
| `0.1.0` | `2026-10-30` | README awal: orientasi produk, stack, roadmap, dokumentasi, setup berbasis inspeksi, agent workflow, quality gates, status keputusan. | Dokumentasi/arsip saja; implementasi `UNVERIFIED`, app tests `NOT_RUN`. |
