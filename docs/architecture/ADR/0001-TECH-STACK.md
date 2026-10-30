# ADR-0001 — Pilihan Technology Stack dan Arsitektur Dasar Esokira

> **Artifact**: `docs/architecture/ADR/0001-TECH-STACK.md`  
> **Versi**: `v0.1.0`  
> **Tanggal penyusunan**: `2026-10-30`  
> **Status dokumen**: `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Status keputusan formal ADR**: `PROPOSED / NOT FORMALLY ACCEPTED`  
> **Decision owner**: Product Owner Esokira (persetujuan formal belum tercatat)  
> **Tahap pertama yang terdampak**: `S-00` dan `S-01 / R0`  
> **Supersedes**: tidak ada  
> **Dibaca bersama**: [Decision Log](../../planning/DECISION-LOG.md), [System Architecture](../SYSTEM-ARCHITECTURE.md), [Master Roadmap](../../planning/MASTER-ROADMAP.md).

**Aturan paling penting:** ada perbedaan antara **teknologi dasar yang sudah dipilih dalam perencanaan proyek** dan **ADR yang sudah disahkan secara formal**. Flutter, Go, PostgreSQL, dan monorepo telah tercatat sebagai `DECIDED` dalam dokumen sumber. Status ADR ini masih menunggu persetujuan pemilik; isinya **tidak** mengesahkan pilihan library, provider, desain ledger, kebijakan nominal uang, platform deploy, atau konfigurasi yang belum diputuskan.

---

## 0. Cara memakai ADR ini

ADR (*Architecture Decision Record*) menjelaskan **apa yang dipilih, alasan dan trade-off-nya, apa yang masih belum dipilih, serta bagaimana keputusan akan dibuktikan dan diubah**. ADR ini bukan instruksi untuk menghasilkan seluruh kode Esokira sekaligus.

1. Baca [instruksi agent global](../../../AGENTS.md) dan instruksi area yang relevan: [`frontend/AGENTS.md`](../../../frontend/AGENTS.md) atau [`backend/AGENTS.md`](../../../backend/AGENTS.md).
2. Periksa **repository aktual**, bukan hanya diagram atau target struktur pada dokumentasi. Belum ada audit repository yang dapat dijadikan bukti implementasi.
3. Identifikasi slice (`S-00`–`S-12`), requirement (`FR-*`, `EN-*`, `NFR-*`), dan task `TK-*`/`EP-*` yang sedang dikerjakan.
4. Tinjau status pada [Decision Log](../../planning/DECISION-LOG.md) dan dokumen domain yang paling relevan.
5. Pilihan yang `DECIDED` dapat dipakai sebagai **baseline perencanaan**. Pilihan `PROPOSED` hanya kandidat; pilihan `TBD` tidak boleh diam-diam diubah menjadi dependency atau kontrak produksi.
6. Jika implementasi memerlukan keputusan yang belum disetujui, lanjutkan hanya pekerjaan independen yang aman; tandai bagian tersisa `BLOCKED`, sertakan alternatif dan bukti yang diperlukan.
7. Laporan `PASS`, `DONE`, atau `DEPLOYED` hanya boleh muncul setelah bukti nyata tersedia sesuai [Testing Strategy](../../development/TESTING-STRATEGY.md) dan [Definition of Done](../../development/DEFINITION-OF-DONE.md).

### 0.1 Arti status

| Status | Arti operasional | Tindakan agent |
|---|---|---|
| `DECIDED` | Pilihan dasar telah dinyatakan sebagai keputusan proyek oleh sumber sebelumnya. | Konsisten pada rancangan; verifikasi dengan source code sebelum mengubah repository. |
| `PROPOSED` | Rekomendasi/rancangan yang belum mendapat persetujuan formal. | Boleh membuat perbandingan, prototipe terisolasi, atau rencana; jangan mengunci dependency production. |
| `TBD` | Opsi/semantik masih terbuka atau belum cukup dibahas. | Tanyakan/eskalasi tepat waktu; jangan mengarang default tersembunyi. |
| `REQUIRED` | Sifat keselamatan/kebenaran yang harus dipenuhi oleh desain apa pun. | Terapkan invariant dan uji; jangan samakan dengan pilihan vendor/tool. |
| `BLOCKED` | Tidak ada izin/bukti yang cukup untuk langkah spesifik. | Hentikan langkah spesifik; catat blocker dan jalur keputusan. |
| `ACCEPTED` (ADR) | Persetujuan eksplisit dengan pemilik, tanggal, scope, dan bukti yang tertaut. | **Belum berlaku untuk ADR-0001**. |

**Catatan**: `REVIEWED_INTERNAL` adalah status kualitas dokumen, **bukan** status bahwa keputusan baru sudah disetujui atau fitur sudah dibuat.

## 1. Konteks, masalah, dan pendorong keputusan

Esokira adalah aplikasi **perencanaan keuangan pribadi dan simulasi masa depan**, bukan aplikasi trading, pemroses pembayaran, ataupun sistem agregasi bank yang wajib tersedia untuk MVP. Aplikasi perlu menangani data finansial aktual dan simulasi hipotetis yang **tidak boleh saling mengubah**, lalu berkembang dari fondasi (`R0`) menuju MVP (`R1`), planning (`R2`), dan simulasi lanjutan (`R3`). Item `R4` tetap *discovery only*.

**Masalah arsitektur:** memilih stack inti yang cukup sederhana untuk dikerjakan secara bertahap—termasuk dengan bantuan AI coding agent—namun mampu menjaga ketepatan numerik, kepemilikan data, transaksi atomik, pengujian independen, dan jalur pengembangan ke simulasi yang lebih berat tanpa memaksakan microservices sejak awal.

### 1.1 Driver dan konsekuensinya

| Driver | Konsekuensi untuk stack | Kaitan |
|---|---|---|
| **Kebenaran finansial** | Nilai uang aktual harus exact; domain engine dapat diuji tanpa UI/API/DB. | `NFR-002`, `EN-004`, aturan `DR-*`, kontrak `CS-*` |
| **Keamanan dan isolasi pengguna** | Klien tidak dipercaya; principal diverifikasi server; setiap resource diperiksa owner-nya. | `EN-002`, `EN-006`, `SEC-*` |
| **Mutasi aktual atomik** | Transfer internal dan koreksi memerlukan transaksi, kontrol concurrency, serta idempotency. | `FR-003`, `FR-004`, `Q-03`, `Q-04` |
| **Pemisahan actual dan hypothetical** | Preview/skenario tidak melakukan posting ledger; hasil simulasi harus traceable. | `NFR-010`, `Q-06` |
| **Maintainability untuk incremental delivery** | Batas modul jelas, satu backlog, kontrak antarkomponen teruji, hindari scaffolding berlebihan. | `EN-001`, `EN-003`, `S-00`–`S-12` |
| **Kemampuan kerja offline/privasi** | Kebijakan cache/sinkronisasi harus dipilih, bukan diasumsikan dari kemampuan framework. | `Q-10`, `SEC-Q*` |
| **Kinerja dan simulasi berat** | Pekerjaan asinkron dapat ditambah kemudian bila benar-benar diperlukan (`R3`). | `EN-005`, `FR-018` |
| **Keterlacakan AI agent** | Semua dependency, migration, version, dan status hasil harus punya bukti; kontrak tidak diciptakan dari contoh. | `AGENTS.md`, `DW-*`, `DOD-*` |

### 1.2 Constraint yang telah tersedia

- `frontend/`: aplikasi mobile **Flutter/Dart**.
- `backend/`: server **Go**, dengan **PostgreSQL** sebagai database utama.
- **Monorepo** untuk koordinasi kode, dokumen, dan perubahan kontrak.
- Kebutuhan kontrak komunikasi mobile/server dan integritas data melintasi boundary.
- Belum ada hasil audit monorepo, file build, `pubspec.lock`, `go.mod`, toolchain/SDK version, CI config, atau migration yang bisa dijadikan bukti dalam ADR ini.

Batasan deployment, lisensi, hosting, biaya, jumlah pengguna, kebutuhan throughput, platform iOS/Android minimum, dan lokasi penyimpanan data belum disimpulkan tanpa sumber keputusan.

## 2. Ringkasan keputusan dan status sebenarnya

| ID area | Pilihan / pendekatan | Status per sumber | Lingkup yang tercakup | Yang **belum** ikut diputuskan |
|---|---|---|---|---|
| `STACK-01` | Flutter + Dart untuk mobile client | **DECIDED** | Bahasa/framework aplikasi klien pada `frontend/` | Riverpod/BLoC, router, HTTP client, chart, cache, min SDK |
| `STACK-02` | Go untuk backend | **DECIDED** | Bahasa server pada `backend/` | Versi Go, router, DI, query codegen, job framework |
| `STACK-03` | PostgreSQL sebagai primary database | **DECIDED** | Penyimpanan sistem backend | Versi, schema, data type uang, RLS, migrasi, provider hosting |
| `STACK-04` | Monorepo `frontend/` dan `backend/` | **DECIDED** | Pola organisasi repository | Build orchestration, CI vendor, repo tooling, path file eksisting |
| `STACK-05` | Modular monolith untuk Go | **PROPOSED** | Batas modul dan deploy sederhana | Package final, service boundaries, jumlah binaries |
| `STACK-06` | HTTPS REST/JSON + OpenAPI | **PROPOSED** | Cara merancang interface Flutter ↔ Go | Endpoint, payload, versioning, auth, idempotency headers |
| `STACK-07` | Pure Go financial engine | **PROPOSED** untuk mekanisme; correctness **REQUIRED** | Formula dapat diuji independen | Rate/calendar/rounding/decimal libraries, model spesifik |
| `STACK-08` | API/server sebagai otoritas data aktual | **REQUIRED** | Trust boundary, mutations, audit | Skema kepemilikan, provider identitas, offline write policy |
| `STACK-09` | Exact persisted money & precise rates | **REQUIRED**, detail **TBD** | Hindari binary float sebagai sumber nominal aktual | IDR vs multi-currency, `BIGINT` vs `NUMERIC`, API encoding |
| `STACK-10` | Background jobs untuk komputasi berat `R3` | **PROPOSED** bila dibutuhkan | Eksekusi simulations/optimizer yang lama | Queue, retry semantics, resource limits, topology |
| `STACK-11` | Identity provider dan session strategy | **TBD** | Authentication/authorization | Firebase/OIDC/provider lain, token lifecycle |
| `STACK-12` | Hosting, backup, retention, observability vendor | **TBD** | Operasional dan recovery | Cloud, region, metrics vendor, biaya, retention period |

`STACK-*` pada tabel di atas **hanya label pembacaan dalam ADR ini**, bukan requirement/approval ID baru. Rekaman keputusan induk yang harus diperbarui tetap [`Q-01`–`Q-11`](../../planning/DECISION-LOG.md), sementara pertanyaan teknis lokal tetap milik dokumen sumber masing-masing.

## 3. Baseline stack yang telah dipilih

### 3.1 Flutter/Dart — mobile client (`STACK-01`, DECIDED)

**Alasan kecocokan**:

- Satu codebase UI mobile dapat memfasilitasi pembangunan bertahap untuk fitur input transaksi, grafik, tujuan finansial, dan tampilan skenario.
- Pemisahan widget/presentation, application state, domain models, dan repository dapat diuji pada setiap lapisan.
- Cocok dengan desain *feature-first* yang telah ditulis pada [Frontend Architecture](../FRONTEND-ARCHITECTURE.md), tanpa menjadikannya keputusan framework state management otomatis.
- UI dapat mengomunikasikan *uncertainty*, asumsi, stale simulation, empty/error states, dan provenance yang penting untuk aplikasi perencanaan keuangan.

**Trade-off**:

- Widget kompleks, grafik, serta perbedaan perilaku perangkat memerlukan pengujian UI dan aksesibilitas terpisah.
- Keputusan plugin untuk token storage, network, charting, background lifecycle, dan offline data tetap harus ditinjau keamanan/maintenance-nya.
- Perhitungan pada Dart boleh dipakai untuk tampilan sementara yang diberi label jelas, **bukan** menggantikan hasil authoritative financial engine backend.

**Keputusan yang sengaja tidak disertakan**: Riverpod vs BLoC (`FE-Q01`), router (`FE-Q02`), transport (`FE-Q03`), dan auth secure storage (`FE-Q04`) tetap terbuka. Jangan menganggap `go_router`, Dio, Firebase SDK, atau paket chart tertentu telah disahkan.

### 3.2 Go — backend (`STACK-02`, DECIDED)

**Alasan kecocokan**:

- Bahasa statically typed dengan fitur standard library yang memadai untuk HTTP, cancellation, testing, dan concurrency.
- Implementasi modul domain, API, DB access, serta pure calculation engine dapat berbagi tipe domain dan kebijakan validasi secara eksplisit.
- Deployment process yang relatif sederhana masuk akal sebagai baseline aplikasi skala awal, selama observability, error handling, dan operasi database dibangun dengan benar.
- Background workers dapat ditambahkan pada `R3` tanpa harus memecah seluruh sistem menjadi microservices.

**Trade-off**:

- Banyak detail harus didesain eksplisit: error propagation, uang exact, transaksi, locking, audit, cancellation, serialization, data ownership.
- Goroutine bukan pengganti job system yang tahan gagal; worker yang *replayable* memerlukan state, retry, timeouts, idempotency, dan bukti pengujian.
- Handler HTTP dan domain finance harus dipisah agar formula dapat diuji tanpa membuka koneksi database.

**Belum diputuskan**: `net/http` vs `chi`, `pgx`/`sqlc`, pilihan migration tool, dependency injection style rinci, engine decimal library, worker system (lihat [Backend Architecture](../BACKEND-ARCHITECTURE.md)).

### 3.3 PostgreSQL — primary database (`STACK-03`, DECIDED)

**Alasan kecocokan**:

- Mendukung transaksi ACID dan kontrol concurrency yang relevan untuk saldo dan pemindahan nilai antarakun.
- Constraints, keys, dan transaksi dapat mendukung isolasi kepemilikan resource serta integritas relational.
- Tipe numerik exact tersedia, meskipun **pemilihan tipe, scale, dan rounding policy belum disetujui**.
- Satu primary relational store membantu menjaga relasi actual events, goals, scenarios, result metadata, audit, dan idempotency.

**Trade-off**:

- DB tidak otomatis menjamin kebenaran ledger; desain model posting dan transaction boundary harus diuji.
- Locking, isolation, migration rollback/forward, test fixtures, backup/restore dan recovery membutuhkan prosedur yang terbukti.
- Multiuser authorization harus diterapkan oleh backend dan—jika disetujui—diperkuat RLS; sekadar menyimpan `user_id` tidak cukup.

**Belum diputuskan**: `BIGINT` minor/base units atau `NUMERIC`, struktur ledger, PostgreSQL version, migration runner, RLS, timezone/data retention, hosting/backup (lihat [Database Design](../DATABASE-DESIGN.md)).

### 3.4 Monorepo (`STACK-04`, DECIDED)

**Alasan kecocokan**:

- Perubahan kontrak mobile/server, migrasi database, test, dan dokumentasi dapat direview dalam satu rangkaian perubahan.
- AI coding agent dapat mengakses [instruksi root](../../../AGENTS.md) dan petunjuk khusus frontend/backend tanpa kehilangan konteks dependency.
- Memudahkan traceability requirement → task → diff → test evidence → release gate.

**Trade-off**:

- Perlu aturan pemisahan build, kontrol dependency, dan CI agar perubahan backend tidak tanpa alasan memblokir pekerjaan UI dan sebaliknya.
- Merge dan koordinasi multi-agent tetap membutuhkan scope ownership dan review, bukan sekadar memiliki satu repository.
- Menaruh source dalam satu repo tidak berarti Flutter boleh langsung mengakses database atau modul internal Go.

## 4. Alternatif yang dipertimbangkan (rasional, bukan benchmark)

> Tabel ini adalah **penilaian kualitatif berdasarkan kebutuhan proyek**, bukan hasil pengujian performa, biaya, atau benchmark aktual. Alternatif tidak dinyatakan tidak layak secara universal; hanya **tidak dipilih sebagai baseline Esokira**.

| Keputusan | Baseline saat ini | Alternatif masuk akal | Pertimbangan / trade-off |
|---|---|---|---|
| Mobile UI | Flutter/Dart | React Native/TypeScript; native Kotlin + Swift | Native memberi kontrol platform mendalam tetapi butuh lebih banyak pengembangan terpisah; React Native tetap valid tetapi bukan baseline yang sudah dipilih. |
| Backend | Go | Node.js/TypeScript; Python/FastAPI; JVM/Kotlin | Alternatif dapat produktif, namun berpindah sekarang menambah perubahan desain dan pekerjaan integrasi tanpa bukti kebutuhan. |
| Primary data store | PostgreSQL | SQLite; document-oriented store (contoh Firestore/MongoDB) | SQLite dapat efektif untuk klien/local use; document store punya pola data berbeda, sedangkan transaksi relasional lintas entitas dominan pada kebutuhan ledger server. |
| Repository model | Monorepo | Multi-repo terpisah per layanan/aplikasi | Multi-repo dapat memisahkan lifecycle, tetapi menaikkan biaya koordinasi kontrak pada tim/proyek kecil. |
| Backend deployment architecture | Modular monolith **(PROPOSED)** | Microservices; serverless functions; *layered single service* | Monolith bermodul mengurangi kompleksitas distribusi di R0–R1; risiko modul terlalu saling terikat perlu dijaga. |
| API style | REST + OpenAPI **(PROPOSED)** | GraphQL; gRPC | REST sederhana untuk mobile/client-server dan kontrak pengujian; teknologi lain bisa berguna di kebutuhan spesifik yang belum terbukti. |
| Heavy jobs (`R3`) | Conditional Go worker **(PROPOSED)** | Synchronous API; external managed job service | Perlu bukti kapasitas & reliability sebelum menambah proses/queue; operasi simpan hasil harus konsisten. |

**Mengubah keputusan `DECIDED`** memerlukan proposal perubahan formal, analisis migrasi dan dampak biaya/risiko, serta persetujuan owner; bukan pilihan sepihak agent karena preferensi pribadi.

## 5. Desain arsitektur yang direkomendasikan — belum menjadi paket final

### 5.1 Modular monolith pada backend (`STACK-05`, PROPOSED)

**Rekomendasi:** mulai dengan satu domain-oriented Go backend yang memisahkan HTTP adapters, application use cases, persistence adapters, dan pure finance engine. Setiap modul bisnis harus memiliki ownership jelas; dependency lintas modul melalui interface atau application boundary yang disepakati.

**Bukan keputusan final:** nama package, single-Go-module vs multi-module detail, worker executable, cara dependency injection, dan bentuk deploy harus mengikuti audit dan approval. Jangan membuat microservices, event bus, Redis, atau broker hanya untuk mencocokkan diagram.

**Syarat bila usulan diterima:**

- Domain finance tidak mengimpor HTTP transport, PostgreSQL driver, atau Flutter DTO.
- Handler tidak memuat rumus finansial atau logika posting transfer yang harus atomik.
- Database transaction diatur di boundary use case; satu transfer aktual tidak boleh menjadi dua transaksi terpisah tanpa desain komit yang aman.
- Modul yang memiliki data harus mengontrol akses owner, audit, dan validasi input terkait.
- Semua jalur aktual dan simulasi memiliki pemisahan eksplisit pada model data, hak operasi, dan pengujian.

### 5.2 API interface (`STACK-06`, PROPOSED)

**Rekomendasi:** gunakan HTTPS REST/JSON untuk komunikasi mobile dengan Go dan pertahankan **satu kontrak OpenAPI authoritative** setelah disetujui. Path `contracts/openapi.yaml` adalah lokasi **target yang diusulkan**, **bukan bukti file telah tersedia**.

Rincian yang **belum final**: endpoint catalog, serialized money/rates/time, request/response envelopes, pagination, error schema, auth header/session, rate limits, idempotency key dan conflict behavior. Sampel pada [API Guidelines](../API-GUIDELINES.md) bukan izin langsung membuat endpoint produksi.

**Peraturan interoperabilitas**:

- Perubahan API harus memperbarui kontrak, validasi Go, pemetaan Flutter, tes contract, dan dokumentasi yang terkena dalam satu task reviewable.
- Klien tidak boleh menentukan owner melalui `user_id` yang diterima mentah oleh server.
- Request yang mempunyai akibat aktual memerlukan perilaku retry/idempotency yang sudah ditentukan dan diuji.
- Jangan menjanjikan *offline write sync* sebelum kebijakan konflik/duplikasi dan auth tersedia.

### 5.3 Financial engine (`STACK-07`, PROPOSED + REQUIRED correctness)

**Rekomendasi:** buat calculation core di Go yang dapat dipanggil sebagai fungsi yang jelas input/output-nya, dengan golden cases independen dari HTTP, DB, dan Flutter. Pada `S-01`, `EN-004` menuntut **boundary serta test harness**; implementasi formula compound yang sebenarnya mengikuti `S-05` / `FR-008` setelah keputusan numeric conventions disetujui.

**Tidak boleh terjadi**:

- `float64`/`double` menjadi satu-satunya sumber nominal uang persisten aktual, tanpa representasi exact di backend/DB.
- Tampilan angka hasil simulasi diberi label seolah saldo aktual, instruksi transaksi, atau prediksi pasti.
- Agent memilih default suku bunga, inflasi, sampling distribution, seed, optimizer objective, atau rounding dengan asumsi diam-diam.
- Preview atau skenario mengubah posted transactions, account balances, assets, liabilities atau funded progress.

**Jalur keputusan**: `Q-04` dan `Q-05` untuk konvensi numerik awal; `Q-06` untuk scenario baseline/revisions; `Q-07` optimizer; `Q-08` Monte Carlo. Lihat [Financial Domain Rules](../../financial/FINANCIAL-DOMAIN-RULES.md), [Calculation Specs](../../financial/CALCULATION-SPECS.md), dan [Financial Test Cases](../../financial/FINANCIAL-TEST-CASES.md).

### 5.4 Heavy simulations dan worker (`STACK-10`, R3 conditional)

Background execution relevan pada `EN-005` sebelum `FR-018` jika waktu eksekusi dan skala benar-benar memerlukannya. **Bukan komitmen memasang River, Redis, Kafka, Kubernetes, scheduler, atau cloud queue pada R0/R1.**

Jika proposal worker di kemudian hari disetujui, test wajib mencakup: job lifecycle/status, duplicate enqueue, retry, dead-letter atau terminal failure policy, cancellation, resource limits, stale input, seed/version reproducibility, authorization hasil, dan rollback/retention.

## 6. Keputusan teknologi turunan yang belum final

| Area | Kandidat/contoh dalam dokumen terdahulu | Status | Kriteria sebelum memilih | Titik implementasi yang terpengaruh |
|---|---|---|---|---|
| Flutter state | Riverpod atau BLoC | `TBD` (`FE-Q01`) | Maintainability, state lifecycle, testability, skill set, cache invalidation | `TK-S01-04` |
| Flutter router | `go_router` atau alternatif | `PROPOSED/TBD` (`FE-Q02`) | Deep links, guarded routes, restoration, auth transitions | `TK-S01-04` |
| HTTP/serialization | `Dio`, `http`, manual/generated DTO | `TBD` (`FE-Q03`) | OpenAPI conformity, error/retry model, maintenance | `TK-S01-07`, client integrations |
| Auth/session | Firebase Auth, OIDC provider, atau lainnya | `TBD` (`Q-02`, `ARCH-Q01`) | Token verification, server principal, revocation, recovery, privacy, cost | `TK-S00-03`, `TK-S01-09`, `S-02` |
| Go HTTP router | modern `net/http`, `chi`, atau lain | `TBD` | Standardization, middleware, testability, routing semantics | `TK-S00-07`, `TK-S01-03` |
| Go PostgreSQL access | `pgx`, `sqlc`, explicit SQL atau alternatif | `PROPOSED/TBD` | Transaction control, generated code reproducibility, type mapping, security | `TK-S00-07`, `S-01/S-03` |
| DB migration | `goose`, `golang-migrate`, atau alternatif | `TBD` | Repeatability, review, integration test, rollback policy | `TK-S01-05` |
| Money type/API | fixed-precision `NUMERIC`, integer units, decimal strings | Exact **REQUIRED**, details `TBD` (`Q-04`) | Currency, negative values, rounding, overflow, temporal semantics | `TK-S00-05`, `S-03`, `S-05` |
| Ledger | event/movement ledger atau alternatif dengan audit kuat | `TBD` (`Q-03`) | Atomic transfer, reversals, ownership, duplicates, corrections | `TK-S00-04`, `S-03` |
| Isolation | backend scoped queries + possible PostgreSQL RLS | Owner checks **REQUIRED**, RLS `TBD` | Defense in depth, migration correctness, DB principal model | `TK-S01-06` |
| Local caching/offline | minimal cache, online-first atau offline support | `TBD` (`Q-10`) | Sensitive storage, invalidation, logout cleanup, conflict semantics | Mobile data layer |
| Worker queue | PostgreSQL-backed queue, River, managed service | `TBD`, R3 | Durability, retries, cancellation, resources, deployment burden | `EN-005`, `S-10/S-11` |
| Hosting/CI/observability | Provider/tools belum ditetapkan | `TBD` (`Q-10`) | Cost, operational skill, secrets, backup/restore, availability | `TK-S01-12/13`, public release |
| SDK/tool versions | Flutter/Dart, Go, PostgreSQL supported versions | `TBD` setelah audit | Target devices, supported updates, dependency compatibility | `TK-S00-01`, `TK-S00-07` |

**Tidak boleh** menyimpulkan keputusan final hanya karena satu kandidat muncul paling sering, tercantum dalam diagram, atau menjadi opsi yang familiar bagi agent.

## 7. Struktur target dan boundary repository

Berikut **layout konseptual**. Hanya `README.md`, `AGENTS.md`, tiga instruksi agent area, dan dokumen yang telah dibuat sebagai artefak dapat dianggap sebagai **dokumen yang tersedia** di rangkaian penyusunan ini; keberadaannya di *repository pengguna* tetap harus diverifikasi. Source dan konfigurasi di bawah bukan klaim file sudah dibuat.

```text
esokira/                             # Target monorepo, belum diaudit
├── README.md                        # Dokumentasi pintu masuk
├── AGENTS.md                        # Instruksi root
├── frontend/
│   ├── AGENTS.md                    # Instruksi agent Flutter
│   ├── lib/                         # TARGET; periksa sebelum mengubah
│   ├── test/                        # TARGET; periksa sebelum mengubah
│   └── pubspec.yaml                 # TARGET; jangan berasumsi sudah ada
├── backend/
│   ├── AGENTS.md                    # Instruksi agent Go
│   ├── cmd/                         # TARGET
│   ├── internal/                    # TARGET; modul platform/domain
│   ├── db/migrations/               # TARGET
│   └── go.mod                       # TARGET; tidak ada versi yang di-lock di sini
├── contracts/
│   └── openapi.yaml                 # TARGET dan PROPOSED; belum authoritative
└── docs/
    ├── INDEX.md
    ├── product/
    ├── financial/
    ├── architecture/
    │   └── ADR/0001-TECH-STACK.md  # Dokumen ini
    ├── design/
    ├── security/
    ├── development/
    └── planning/
```

**Boundary minimum**, terlepas dari struktur direktori akhirnya:

- Flutter → komunikasi dengan Go melalui boundary API yang disepakati; **tidak** memiliki kredensial database, mengakses PostgreSQL langsung, atau menerima private resource tanpa autorisasi server.
- Go HTTP → hanya adapter; validasi autentikasi dan ownership pada seluruh private operations; transaksi DB dikendalikan use case.
- Go calculation engine → semantik finansial dapat diuji tanpa request, storage, network, atau job queue.
- PostgreSQL → primary authoritative persistensi, dengan constraints/ownership, audit/idempotency dan transaksi konsisten.
- Simulated data → terpisah dari actual data; baseline/result mempunyai identitas dan versioning yang disepakati sebelum implementasi R2–R3.
- Configuration/secrets → dikelola dari lingkungan yang sesuai; tidak dicantumkan di source, contoh docs, log, screenshot, atau keluaran agent.

## 8. Pemilihan versi, dependency, dan lisensi

ADR ini **tidak menentukan nomor versi** Flutter, Go, PostgreSQL, SDK Android/iOS, paket pub.dev, atau modul Go karena toolchain nyata belum diaudit. Nomor versi akan menjadi bagian *evidence* setelah `TK-S00-01` dan review `TK-S00-07`.

Sebelum menambahkan dependency:

1. **Buktikan kebutuhan tugas** dan alternatif tanpa dependency tambahan (termasuk standard library).
2. Periksa status pilihan pada [Decision Log](../../planning/DECISION-LOG.md); jika `TBD`, rancang evaluasi kecil dan minta approval sebelum menetapkan library sebagai standard.
3. Verifikasi maintenance, compatibility, security advisories, lisensi, ukuran/footprint, dan kemudahan pengujian **berdasarkan bukti aktual yang dikumpulkan saat task berjalan**.
4. Catat versi yang benar-benar terpasang di manifest/lockfile/repository yang diaudit. Jangan menyalin angka versi dari template dokumentasi.
5. Ubah lockfile, build config, tes, dan dokumentasi pada change yang sama; jangan me-*upgrade* seluruh stack bersamaan tanpa kebutuhan jelas.
6. Jika suatu dependency dibatalkan, catat dampak refactor dan cara migrasi pada ADR susulan atau catatan keputusan.

**Penggunaan Docker** dapat menjadi solusi lokal/CI/deploy yang dinilai, tetapi tidak wajib dipasang hanya karena berguna secara umum; artefak compose, image, dan version pin belum dijamin ada. Panduan setup aktual akan diturunkan ke `docs/development/LOCAL-SETUP.md` setelah audit repository.

## 9. Security dan financial invariant — kriteria yang tidak bisa ditawar

Keamanan dan integritas berikut berlaku untuk semua stack yang dipilih dan **tidak menunggu pilihan vendor**:

| Invariant | Penerapan sistem | Minimum bukti sebelum dinyatakan DONE |
|---|---|---|
| Principal verified server-side | Auth adapter Go memverifikasi token dan memetakan user internal yang sah | Tes token invalid/missing/revoked sesuai provider terpilih |
| Owner-scoped access | Setiap read/write private memverifikasi owner, termasuk nested resources | Tes dua pengguna, akses silang ditolak tanpa kebocoran |
| Transfer atomicity | Debit/credit posting dalam satu boundary transaksi yang disepakati | DB integration tests untuk commit, rollback, concurrency |
| Idempotency | Retry mutasi aktual tidak menciptakan transaksi atau saldo ganda | Duplicate requests, timeout/unknown outcome, conflict cases |
| Exact stored actual money | Representasi DB/API/arithmetic tidak kehilangan nilai karena binary float | Golden & property tests, boundary/rounding/overflow cases |
| Actual/hypothetical isolation | Scenario/forecast/insight tidak memodifikasi ledger dan funded goal actual | Tests pada DB fixture sebelum/sesudah preview/simulation |
| Privacy-safe logging | Secret/token/PII/nominal sensitif tidak ditulis sembarang ke log | Pemeriksaan structured logs, redaction, negative test |
| Reproducibility R2/R3 | Input, baseline, engine/model version, seed/assumption terekam saat dibutuhkan | Replay tests, stale detection dan provenance disclosure |
| Recovery dan retention | Backup/restore dan data rights mengikuti kebijakan yang disetujui | Restore drill berbasis data sintetis; review kebijakan |

Bukti lain tetap mengikuti [`SEC-001`–`SEC-026`](../../security/SECURITY-PRIVACY.md), [`TS-001`–`TS-040`](../../development/TESTING-STRATEGY.md) serta financial tests. **Tidak satu pun baris tabel ini berarti test sudah dijalankan.**

## 10. Adopsi bertahap dan kaitan dengan task

### 10.1 S-00 — keputusan dan audit sebelum implementasi berisiko

| Task | Keluaran yang diperbolehkan saat ini | Gerbang sebelum pekerjaan terkait lanjut |
|---|---|---|
| `TK-S00-01` | Audit tree, manifests, toolchain, infra, tes, konfigurasi dan riwayat yang tersedia | Repository evidence tercatat; gap tidak dianggap file hilang definitif |
| `TK-S00-02` | Konfirmasi MVP, scope dan target pengguna | `Q-01` siap dan bukti tercatat |
| `TK-S00-03` | Matriks evaluasi IdP/token/session | `Q-02` untuk implementasi auth produksi |
| `TK-S00-04` | Alternatif ledger/transfer/correction/funding | `Q-03` dan `Q-11` sebelum actual posting schema |
| `TK-S00-05` | Kebijakan uang, rates, calendar, rounding | `Q-04`/`Q-05` sebelum nominal aktual dan formula tersambung |
| `TK-S00-06` | Threat/privacy/retention/backup baseline review | `Q-10` dan security controls terkait |
| `TK-S00-07` | Evaluasi router, DB tooling, Flutter libraries, migration & CI | Approval dependensi/kontrak yang akan dikunci |
| `TK-S00-08` | Klasifikasi financial oracle/golden cases yang siap vs blocked | Tidak mengarang nilai expected/formula |
| `TK-S00-09` | Register persetujuan dan evidence log | Cross-reference tidak menggantikan tanda tangan owner |

### 10.2 S-01 / R0 — foundation yang bisa dibuktikan

| Enabler | Task relevan | Ekspektasi hasil terverifikasi |
|---|---|---|
| `EN-001` | `TK-S01-01`, `TK-S01-03`, `TK-S01-04`, `TK-S01-12` | Build/check Flutter dan Go **yang benar-benar dijalankan**, CI minimum, ownership code |
| `EN-002` | `TK-S01-02`, `TK-S01-05`, `TK-S01-06` | Test DB disposable, migrations teruji, owner-scoped access |
| `EN-003` | `TK-S01-07`, `TK-S01-08`, `TK-S01-09` | Contract yang disetujui, input validation, deny-by-default |
| `EN-004` | `TK-S01-10` | Pure Go financial engine boundary dan harness; tidak mengklaim compound formula FR-008 selesai |
| `EN-006` | `TK-S01-11`, `TK-S01-13` | Secret hygiene, safe logs, recovery plan dan restore evidence |
| Gate `R0` | `TK-S01-14` | Review berdasarkan [DoD](../../development/DEFINITION-OF-DONE.md) & [Progress](../../planning/PROGRESS.md) |

`EN-005` **bukan** bagian wajib `R0`; worker untuk simulasi berat datang di `R3` sesuai [Feature Matrix](../../product/FEATURE-MATRIX.md). Ini menghindari pemborosan infrastruktur sebelum ada kebutuhan terukur.

### 10.3 R1–R3 — cegah perluasan scope otomatis

- **R1 / S-02–S-06:** identitas, actual accounts/transactions, assets/liabilities, goals, compound calculator, feasibility, dan data rights. Detail kontrak ditentukan sesuai keputusan yang terkena, bukan sekaligus.
- **R2 / S-07–S-09:** life events, recurring plans, deterministic scenario, forecasting dan insight; baseline/version/staleness perlu disetujui lebih dahulu.
- **R3 / S-10–S-12:** optimizer, durable simulation processing jika relevan, Monte Carlo, retirement/FIRE dan Financial GPS; statistical assumptions dan objective belum disahkan.
- **R4:** eksplorasi, bukan izin memasukkan fitur tambahan seperti shared goals atau portfolio planning ke rilis aktif.

Tidak ada tanggal rilis, cost ceiling, latency SLA, coverage threshold, atau target MAU yang disepakati oleh ADR ini.

## 11. Trade-off, risiko, dan mitigasi

| Risiko | Dampak jika diabaikan | Mitigasi / trigger review |
|---|---|---|
| Backend monolith menjadi tight coupling | Regression dan sulit dipisah di masa depan | Review dependency antar-modul, golden domain tests, batas paket jelas |
| Perhitungan keuangan memakai tipe kurang tepat | Saldo/goal salah tanpa error eksplisit | `Q-04` approved, exact type, golden cases, property tests |
| Pergantian library sepihak | Lockfile drift, API incompatibility, security regression | `TK-S00-07`, review dependency, ADR follow-up |
| Banyak proses/queue dipasang terlalu dini | Operasional dan debugging lebih kompleks daripada manfaat | R3 worker bersyarat berdasarkan profiling dan job reliability needs |
| Auth provider dipilih berdasarkan tutorial | Token validation/revocation salah, private data bocor | `Q-02` + security threat analysis + cross-user tests |
| Ledger dan transfer dirancang tanpa integration tests | Double posting, partial write, salah net worth | `Q-03` + PostgreSQL concurrency/idempotency tests |
| UI menganggap cache authoritative | Data stale, mismatch dengan server, simulasi disalahpahami | Server-authoritative reads, stale provenance, cache policy review |
| PostgreSQL menjadi bottleneck atau kehilangan data | Availability/recovery risiko | Index/transaction profiling ketika ada bukti, backups + restore drill |
| Docs dan source tidak sinkron | Agent menjalankan instruksi salah atau melaporkan hasil fiktif | Satu root index, contract reviews, evidence-based progress |
| Vendor lock-in belakangan | Sulit pindah hosting/auth/jobs | Adapter boundaries, export/data rights, scenario migration plan |

## 12. Ketergantungan keputusan dan kapan harus berhenti

Lihat [Decision Log](../../planning/DECISION-LOG.md) untuk status authoritative. Ringkasan berikut **bukan approval baru**.

| Keputusan induk | Topik yang terkait stack | Kapan menghambat | Hal yang aman dilakukan sebelumnya |
|---|---|---|---|
| `Q-01` | Produk/locale/MVP scope | Perubahan prioritas/publikasi produk | Audit dokumentasi, membuat daftar asumsi |
| `Q-02` | IdP dan session | Auth dan private mutations `S-01/S-02` | Interface provider-agnostic, contract review |
| `Q-03` | Ledger dan transfer | DB posting actual `S-03` | Model alternatif, integration test plan |
| `Q-04` | Exact money/time/rounding | DB/API money kontrak dan S-05 formula | Pure harness, test classification |
| `Q-05` | Return/inflation assumptions | Default compound dan R2 forecasting | Unit tests tanpa hardcoded asumsi yang belum dipilih |
| `Q-06` | Baseline/provenance/staleness | `S-08` scenario persistence | Conceptual data flows |
| `Q-07` | Optimizer objective | `S-10` | Constraints comparison dan research |
| `Q-08` | Monte Carlo model | `S-11` | Methodology review, deterministic RNG experiments terisolasi |
| `Q-09` | Notifications/consent | `S-09` reminders | UX concept, privacy review |
| `Q-10` | Hosting, data rights, offline, operations | Public release / relevant foundation operations | Local disposable environment dan backup plan tanpa produksi |
| `Q-11` | Funded goal actual semantics | `S-05` progress | Goal CRUD scaffolding tanpa mengarang accounting |

**Contoh perilaku agent:** bila task memerlukan `sqlc` tapi belum ada approval, agent boleh mengusulkan evaluasi `sqlc` vs SQL manual, **tidak** memasang dan mengubah seluruh repository secara permanen. Bila task hanya merapikan README, keputusan IdP yang masih terbuka tidak memblokir tugas dokumentasi tersebut.

## 13. Konsekuensi pilihan ini

### 13.1 Konsekuensi positif

- Satu stack inti yang terdefinisi untuk mobile, backend dan database, sehingga onboarding/agent instructions lebih fokus.
- Integrasi dan data modeling dapat diuji pada batas yang jelas.
- Peluang memakai toolchain umum tanpa memaksakan vendor cloud atau queue tambahan.
- R0–R1 dapat dibangun secara bertahap, sedangkan kebutuhan advanced baru dipenuhi saat didukung requirement dan evidence.

### 13.2 Biaya dan konsekuensi negatif

- Pengembangan lintas Flutter + Go membutuhkan konsistensi DTO/API, timezone, formatting, validation dan auth state.
- PostgreSQL memberi fleksibilitas dan integritas namun mewajibkan operasi DB yang benar.
- Aplikasi finansial tetap berisiko walaupun teknologi modern; keamanan dan oracle numerik harus didefinisikan, diuji, dan dikaji terpisah.
- Monorepo memerlukan disiplin CI/ownership pada tiap area dan pembaruan dokumen yang sejalan.

### 13.3 Hal-hal yang **tidak** disimpulkan dari ADR

- Tidak ada klaim Flutter/Go/PostgreSQL "paling cepat", "termurah", "100% aman", atau unggul dalam benchmark yang belum dijalankan.
- Tidak ada asumsi pengguna sudah menyediakan source code, Firebase project, cloud database, `Dockerfile`, container image, atau akun CI.
- Tidak ada penentuan provider, architecture diagram final, skema SQL final, public launch date, atau approval privasi/legal.
- Tidak ada perintah implementasi R2/R3/R4 otomatis hanya karena desainnya tercantum.

## 14. Trigger untuk merevisi atau mengganti ADR

Buat ADR baru/versi baru atau pembaruan keputusan eksplisit bila terjadi salah satu hal berikut:

1. Pemilik ingin mengganti bahasa atau primary database karena constraint baru atau hasil pengujian yang bermakna.
2. Modul monolith tidak lagi dapat memenuhi batas resource/isolasi yang terukur; opsi pemisahan service dievaluasi melalui profiling dan failure analysis.
3. Persyaratan offline sync, data residency, biaya/hosting, autentikasi, atau regulatory review mengubah desain fundamental.
4. Pilihan exact money, ledger, rounding, atau scenario provenance memerlukan perubahan kontrak publik/DB.
5. Pilihan worker/queue/telemetry dari `TBD` menjadi final dan berdampak pada operasi/deploy.
6. Versi dependency/toolchain yang digunakan tidak lagi supported atau ditemukan risiko keamanan substansial.

Proses perubahan: tulis masalah → kumpulkan bukti/alternatif → identifikasi diff/rollback/data migration → tinjau financial/security impact → peroleh persetujuan owner → catat tanggal dan keputusan pada [Decision Log](../../planning/DECISION-LOG.md) → perbarui dokumen turunan → buat/ubah task dan test gate → implementasikan secara terbatas.

## 15. Format persetujuan ADR (template, **bukan approval**)

```yaml
# TEMPLATE ONLY — do not populate with fictitious approval
adr_id: ADR-0001
adr_version: v0.1.0
formal_status: PROPOSED
reviewer: NOT_RECORDED
owner_approval: NOT_RECORDED
approval_date: NOT_RECORDED
approved_scope: NOT_RECORDED
approved_decisions: []
approved_tool_versions: []
conditions: []
evidence_links: []
supersedes: null
```

**Ketika benar-benar approved**, cantumkan siapa yang memberi persetujuan (tanpa informasi personal sensitif yang tidak perlu), kapan, untuk apa, dan bukti keputusan eksplisit. Tanda `DECIDED` pada baseline stack **tidak otomatis mengisi field** `owner_approval` untuk ADR maupun pilihan library di bawahnya.

## 16. Checklist review ADR-0001

- [x] Flutter, Go, PostgreSQL, dan monorepo diperlakukan sebagai baseline yang sudah dipilih dalam dokumentasi.
- [x] Modular monolith serta REST/OpenAPI diperlakukan sebagai pendekatan `PROPOSED`, bukan pilihan vendor/final contract.
- [x] Auth provider, state library, Go router, data access tooling, money encoding, worker queue, hosting dinyatakan `TBD/PROPOSED`.
- [x] Kebenaran numerik, kepemilikan data, atomic transfers, idempotency, dan pemisahan actual/simulated dinyatakan invariant.
- [x] `S-00`, `S-01 / R0`, `S-02`–`S-12`, dan scope `R4` tidak dipertukarkan.
- [x] `EN-004` sebagai boundary/test harness R0 dibedakan dari implementasi `FR-008` pada S-05.
- [x] Tautan dokumen ditujukan ke path kanonis dan tidak menyatakan file target source code sudah tersedia.
- [x] Approval formal belum dibuat dan contoh toolchain tidak dianggap telah dijalankan.
- [ ] Product Owner mereview dan secara eksplisit menyetujui/menolak ruang lingkup ADR.
- [ ] Dependency/versions final dicatat berdasarkan audit repository dan keputusan yang benar-benar diambil.
- [ ] Tugas implementasi nyata dan hasil test ditambahkan dalam [Progress](../../planning/PROGRESS.md) setelah dieksekusi.

**Status checklist bukan bukti pengujian software.** Ini hanya review statis terhadap isi dokumen.

## 17. Sumber kebenaran dan tautan navigasi

| Jenis pertanyaan | Dokumen sumber |
|---|---|
| Navigasi seluruh dokumen | [Documentation Index](../../INDEX.md) |
| Gambaran produk, scope dan requirements | [Product Vision](../../product/PRODUCT-VISION.md), [PRD](../../product/PRD.md), [Feature Matrix](../../product/FEATURE-MATRIX.md) |
| Pertanyaan/approval yang masih terbuka | [Decision Log](../../planning/DECISION-LOG.md) |
| Arsitektur konteks dan batas modul | [System Architecture](../SYSTEM-ARCHITECTURE.md) |
| Detail backend, frontend dan database | [Backend Architecture](../BACKEND-ARCHITECTURE.md), [Frontend Architecture](../FRONTEND-ARCHITECTURE.md), [Database Design](../DATABASE-DESIGN.md) |
| Kontrak API yang direncanakan | [API Guidelines](../API-GUIDELINES.md) |
| Domain keuangan dan test oracle | [Financial Domain Rules](../../financial/FINANCIAL-DOMAIN-RULES.md), [Calculation Specs](../../financial/CALCULATION-SPECS.md), [Financial Test Cases](../../financial/FINANCIAL-TEST-CASES.md) |
| Keamanan dan privasi | [Security & Privacy](../../security/SECURITY-PRIVACY.md) |
| Standar implementasi dan pengujian | [Development Workflow](../../development/DEVELOPMENT-WORKFLOW.md), [Testing Strategy](../../development/TESTING-STRATEGY.md), [Definition of Done](../../development/DEFINITION-OF-DONE.md) |
| Prioritas dan bukti kerja | [Master Roadmap](../../planning/MASTER-ROADMAP.md), [Tasks](../../planning/TASKS.md), [Progress](../../planning/PROGRESS.md) |
| Instruksi permanen agent | [Root AGENTS](../../../AGENTS.md), [Frontend AGENTS](../../../frontend/AGENTS.md), [Backend AGENTS](../../../backend/AGENTS.md) |
| Pintu masuk repository | [README](../../../README.md) |

## 18. Riwayat perubahan

| Versi | Tanggal | Perubahan | Verifikasi |
|---|---|---|---|
| `v0.1.0` | `2026-10-30` | ADR stack baseline, alternatif, konsekuensi, aturan keputusan, R0 gates dan referensi dokumen | Review statis; implementasi tidak diperiksa |

---

**Ringkasan keputusan**: Esokira mempertahankan **Flutter/Dart + Go + PostgreSQL + monorepo** sebagai baseline yang telah dipilih dalam dokumentasi proyek. **Modular monolith, REST/OpenAPI dan pure calculation engine** adalah rancangan dengan invariant finansial/keamanan yang wajib dipenuhi, **bukan** tanda semua detail operasionalnya sudah approved. ADR-0001 sendiri masih **menunggu persetujuan formal Product Owner**. Semua keputusan lebih detail kembali ke [Decision Log](../../planning/DECISION-LOG.md) dan bukti implementasi ke [Progress](../../planning/PROGRESS.md).
