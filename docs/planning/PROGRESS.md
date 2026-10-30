# Esokira - Project Progress & Evidence Ledger

> **Canonical path:** `docs/planning/PROGRESS.md`  
> **Version:** `0.1.0`  
> **Prepared:** 2026-10-30 (Asia/Jakarta)  
> **Document status:** `REVIEWED_INTERNAL - PENDING_OWNER_APPROVAL`  
> **Observed implementation state:** `UNVERIFIED` (repository aplikasi belum diaudit)  
> **Observed application tests:** `NOT_RUN` (tidak ada bukti eksekusi Go/Flutter/PostgreSQL/CI)  
> **Owner approval:** `NOT_RECORDED` untuk dokumen dan pekerjaan implementasi ini  
> **Scope:** S-00 sampai S-12 (58 task rinci R0/R1 + 13 epic R2/R3). R4 tetap discovery-only.

Dokumen ini adalah **ledger progres berbasis bukti**, bukan laporan bahwa aplikasi telah dibangun. Status awal semua task/epic adalah `UNVERIFIED`, karena hanya dokumen perencanaan yang tersedia untuk ditinjau saat penyusunan. Nilai `UNVERIFIED` **bukan** sinonim dari `NOT_STARTED` atau `BLOCKED`. File ini bisa langsung digunakan sebagai baseline ketika coding agent mulai mengaudit repository sebenarnya.

---

## 0. Operating contract - wajib dibaca sebelum memperbarui progres

1. `TASKS.md` adalah sumber **ID, scope, dependensi dan acceptance criteria yang direncanakan**. `PROGRESS.md` adalah sumber **observasi, bukti, status aktual, dan handoff**. Jangan menduplikasi spesifikasi task secara panjang di sini.
2. Audit repository aktual dulu sebelum mengganti `UNVERIFIED` dengan `NOT_STARTED`, `PARTIAL`, atau `DONE`. Jangan menganggap project kosong hanya karena kode tidak dilampirkan dalam percakapan.
3. Jangan menandai test `PASS` tanpa perintah atau prosedur yang benar-benar dijalankan, environment, hasil, dan referensi bukti. Dokumen test case tidak sama dengan hasil test.
4. `REVIEWED_INTERNAL`, `APPROVED`, `DONE`, `PASS`, dan `DEPLOYED` adalah status berbeda. Persetujuan dokumen tidak membuktikan implementasi, dan kelulusan CI tidak membuktikan seluruh financial domain benar.
5. Prompt seperti **"gas buat markdown selanjutnya"** hanya mengizinkan pembuatan dokumen terkait; bukan approval provider autentikasi, ledger, model uang, OpenAPI, migrasi, distribusi Monte Carlo, atau deployment.
6. Jangan mencatat data pelanggan asli, token, informasi rekening, lokasi pribadi, secret, maupun isi response finansial sensitif di Markdown progres, screenshot publik, atau log CI.
7. Perubahan yang berpotensi memodifikasi saldo aktual, menghapus data, mengubah kontrak, atau membuka akses antarpengguna wajib melalui approval dan tes relevan sebelum acceptance.
8. Setiap aktivitas implementasi harus mengacu minimal pada satu task `TK-*` atau epic `EP-*` yang sudah dipecah dan disetujui. Gunakan ID task persis dari `TASKS.md` tanpa renumbering.
9. Status pekerjaan boleh berubah hanya dengan alasan dan bukti. Catatan aktivitas tidak boleh ditimpa untuk menghapus kegagalan, blocker, atau keputusan historis.
10. Ketika source-of-truth tidak sepakat, tandai `DOCUMENTATION_DRIFT`, tuliskan kedua referensi, dan berhenti pada bagian terdampak sampai owner menyetujui penyelesaiannya.

**Perintah kerja agent:** baca root `AGENTS.md` + instruksi folder relevan, baca task dan sumber domain, lakukan inspect, buat plan, lewati decision gate, implement hanya scope yang diizinkan, jalankan test, review, lalu catat bukti di sini.

## 1. Hubungan file dan batas sumber kebenaran

| Kebutuhan | Sumber kanonis | Yang disimpan di PROGRESS.md |
|---|---|---|
| Apa yang harus dibangun | [`PRD.md`](../product/PRD.md), [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md) | Status observasi kebutuhan, bukan definisi fitur baru |
| Urutan rilis dan dependensi | [`MASTER-ROADMAP.md`](./MASTER-ROADMAP.md) | Status aktual gate R0-R3 dan kendala yang ditemukan |
| Task ID, scope, expected acceptance | [`TASKS.md`](./TASKS.md) | Pemilik kerja (jika diketahui), hasil audit, bukti aktual |
| Cara agent bekerja | [`DEVELOPMENT-WORKFLOW.md`](../development/DEVELOPMENT-WORKFLOW.md) | Catatan task lifecycle dan handoff |
| Kriteria keberterimaan | [`DEFINITION-OF-DONE.md`](../development/DEFINITION-OF-DONE.md) | PASS/FAIL/NOT_RUN/BLOCKED/N/A per gate yang relevan |
| Testing dan test oracle | [`TESTING-STRATEGY.md`](../development/TESTING-STRATEGY.md), [`FINANCIAL-TEST-CASES.md`](../financial/FINANCIAL-TEST-CASES.md) | Link test run, lingkungan, exit code, hasil dan anomali |
| Persetujuan keputusan teknis | Dokumen domain masing-masing dan `DECISION-LOG.md` bila tersedia | ID keputusan serta status approval yang diverifikasi; **bukan** keputusan baru |
| API dan DB deployed | `contracts/openapi.yaml` dan migrasi terapan **jika terbukti ada** | Ref commit, versi, drift; bukan asumsi contoh dokumentasi |

**Status referensi:** 23 dokumen sumber telah disusun dalam rangkaian dokumentasi Esokira. Keberadaannya dalam percakapan/arsip **bukan** bukti bahwa seluruh file sudah di-commit ke repository pengguna atau sudah disetujui owner. `DECISION-LOG.md`, `docs/INDEX.md`, dan beberapa spesifikasi advanced masih merupakan deliverable masa depan jika belum ditemukan saat audit.

**Bila dokumen sumber hilang dari repository saat audit**, catat sebagai item setup/audit pada task terkait; jangan membuat ulang isi dari tebakan atau menjadikan statusnya `APPROVED`.

## 2. Taksonomi status: jangan dicampur

### 2.1 Implementasi task dan epic

| Status | Makna yang sah | Bukti minimum |
|---|---|---|
| `UNVERIFIED` | Keadaan kode belum diketahui atau tidak dapat dibuktikan | Belum ada bukti audit yang memadai |
| `NOT_STARTED` | Repo telah diperiksa, implementasi untuk task spesifik tidak ada | Branch/commit, path yang diperiksa, hasil inspeksi |
| `READY` | Definition of Ready terpenuhi dan task diizinkan | Referensi keputusan/kontrak/dependensi dan approval |
| `IN_PROGRESS` | Pekerjaan terotorisasi sedang berlangsung | Work order dan perubahan yang bisa ditelusuri |
| `IN_REVIEW` | Implementasi/bukti tersedia, acceptance belum final | Diff/PR, test run, daftar risiko dan reviewer |
| `DONE` | Seluruh DoD yang berlaku lulus dan approval diterima | Evidence lengkap, DoD gates, acceptance nyata |
| `PARTIAL` | Sebagian scope selesai, acceptance masih kurang | Daftar yang sudah/belum selesai dan bukti terpisah |
| `BLOCKED` | Ada blocker yang relevan, jelas, dan menghalangi task | ID blocker/keputusan + dampak + pemilik keputusan |
| `CANCELLED` | Task dihentikan secara eksplisit | Persetujuan owner dan alasan; ID tetap historis |

`IN_REVIEW` **bukan** `DONE`. `BLOCKED` harus menyebutkan blocker task yang benar-benar terdampak, bukan otomatis memberlakukan semua `Q-*` pada seluruh project. `EP-*` tidak boleh menjadi `IN_PROGRESS` production sampai dipecah menjadi task granular dan diberi izin.

### 2.2 Status pengujian

`NOT_RUN`, `PASS`, `FAIL`, `SKIPPED`, `BLOCKED`, `N/A` adalah nilai per pemeriksaan atau test run. `SKIPPED` dan `N/A` wajib disertai alasan; tes yang seharusnya berjalan tetapi tidak tersedia adalah `NOT_RUN` atau `BLOCKED`, bukan `N/A`.

### 2.3 Status approval dan dokumentasi

- `DRAFT`: proposal belum disahkan.
- `REVIEWED_INTERNAL`: dokumen ditinjau, **bukan** approval owner.
- `PENDING_OWNER_APPROVAL` / `NOT_RECORDED`: belum ada approval yang tercatat untuk objek ini.
- `DECIDED` atau `APPROVED`: hanya sesudah tindakan eksplisit owner/reviewer berwenang dicatat dengan referensi.
- `RELEASED`: harus menunjuk environment dan versi yang benar-benar dirilis; tidak berasal dari dashboard atau checklist kosong.

**Progress summary harus selalu menyebut jenis statusnya** (dokumen, task, test, keputusan, release).

## 3. Baseline snapshot - 2026-10-30 (belum audit repo)

| Field | Nilai baseline yang dapat dibuktikan | Catatan |
|---|---|---|
| Dokumen sumber pada rangkaian ini | 23 dokumen disusun sebelum `PROGRESS.md` | Penyusunan dokumen != persetujuan implementation |
| Backlog task rinci R0/R1 | 58 task `TK-*` | Bersumber dari `TASKS.md` v0.1.0 |
| Backlog epic R2/R3 | 13 epic `EP-*` | Belum dipecah menjadi production tasks |
| Total work items yang dipetakan | 71 | Tidak sama dengan pekerjaan yang telah diselesaikan |
| Slice yang direncanakan | S-00 sampai S-12 (13 slice) | S-00 adalah planning/approval, bukan fitur R0 |
| Repository aplikasi diperiksa? | **TIDAK** | Jangan mengarang branch, commit, struktur folder aktual |
| Task implementation state | **71 `UNVERIFIED`** | Tidak diketahui mana yang sudah/belum dibuat di repo |
| Application tests yang dijalankan di sini | **Tidak ada; `NOT_RUN`** | Tidak ada bukti Go, Flutter, PostgreSQL, device atau CI |
| Current assigned task | **Tidak ada yang diverifikasi** | Assignment tidak boleh diinferensikan |
| Actual active release / deployment | **UNVERIFIED** | Jangan mengklaim sudah di staging/production |
| First suggested next work | `TK-S00-01` (audit repo/environment) | Saran task berikutnya; **belum** dijalankan/diotorisasi |
| Keputusan besar | Terdapat decision gates `Q-01`-`Q-11` | Tiap status harus dibaca dari sumber/approval aktual |
| Kesiapan rilis | **UNVERIFIED** untuk R0-R3 | Tidak ada release gate yang diklaim lulus |

Tidak ada angka persentase implementasi yang valid sebelum audit. **Jangan menampilkan 0% completion seolah-olah sudah diketahui belum ada kode**; tampilkan `UNVERIFIED` sampai baseline aktual diperoleh. Jumlah `DONE` yang *tercatat dengan bukti di ledger ini saat dibuat* adalah nol, tetapi jumlah yang *sudah dikerjakan di repo* tetap tidak diketahui.

## 4. Release dashboard (status awal berbasis bukti, bukan estimasi)

| Release | Slice | Goal utama | Status implementasi | Test/gate | Owner acceptance |
|---|---|---|---|---|---|
| Planning | `S-00` | Audit, keputusan, readiness | `UNVERIFIED` | `NOT_RUN` / approval belum tercatat | `NOT_RECORDED` |
| `R0` | `S-01` | Monorepo, API/DB/auth foundation, engine harness, CI | `UNVERIFIED` | `NOT_RUN` | `NOT_RECORDED` |
| `R1` | `S-02` - `S-06` | Keuangan aktual, goals, kalkulator, hak data | `UNVERIFIED` | `NOT_RUN` | `NOT_RECORDED` |
| `R2` | `S-07` - `S-09` | Planning dan simulasi deterministik | `UNVERIFIED` | `NOT_RUN`; spec belum final | `NOT_RECORDED` |
| `R3` | `S-10` - `S-12` | Optimizer, Monte Carlo, FIRE, Financial GPS | `UNVERIFIED` | `NOT_RUN`; model belum final | `NOT_RECORDED` |
| `R4` | Discovery only | Eksplorasi produk di luar scope | `NOT_AUTHORIZED` | `N/A` untuk release production saat ini | `NOT_RECORDED` |

**Release gate:** baca `MASTER-ROADMAP.md` dan `DEFINITION-OF-DONE.md` sesuai rilis; jangan menandai sebuah release `DONE` hanya karena semua UI tampil atau backend bisa di-build. Penyelesaian R1 tidak dapat mengabaikan transfer, privacy/data rights, idempotency, dan golden financial tests.

## 5. Slice dashboard dan cakupan backlog

| Slice | Rilis | Work item | Jenis | Implementasi | Verifikasi |
|---|---|---:|---|---|---|
| `S-00` | Planning | 9 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-01` | R0 | 14 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-02` | R1 | 6 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-03` | R1 | 8 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-04` | R1 | 6 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-05` | R1 | 8 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-06` | R1 | 7 | `TK-*` | `UNVERIFIED` | Belum audit |
| `S-07` | R2 | 2 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| `S-08` | R2 | 3 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| `S-09` | R2 | 3 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| `S-10` | R3 | 2 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| `S-11` | R3 | 2 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| `S-12` | R3 | 1 | `EP-*` | `UNVERIFIED` | Epic belum dipecah |
| **Total** | | **71** | **58 task + 13 epic** | **Belum diverifikasi** | **Tidak ada test aplikasi yang diklaim** |

**Aturan pemutakhiran dashboard:** jangan memperbarui angka progres secara manual berdasarkan jumlah file yang baru dihasilkan. Rekap harus dihitung dari status task dengan bukti yang sudah di-review; jika seluruh task suatu slice belum diaudit, tetap tampilkan `UNVERIFIED`.

## 6. Canonical task / epic status register (initial snapshot)

**Penting:** nama dan ID di bawah diturunkan langsung dari heading `TASKS.md` v0.1.0. Rincian scope, dependency, risk dan acceptance tetap berada di sana. Semua baris berstatus `UNVERIFIED / NOT_RUN` sebagai baseline yang jujur, bukan kepastian bahwa implementasinya belum pernah dikerjakan. Status di file ini dapat diperbarui satu per satu setelah audit dan evidence dicatat.

### S-00 - Review, approval and decision readiness

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S00-01` | Audit repository dan environment aktual | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-02` | Review scope R0/R1 dan prioritas MVP dengan pemilik proyek | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-03` | Putuskan authentication provider dan model sesi | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-04` | Putuskan model saldo, transfer, koreksi, dan funding goals | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-05` | Putuskan kebijakan uang, rate, waktu, dan pembulatan | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-06` | Putuskan baseline privasi, data rights dan lingkungan | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-07` | Review pilihan library, kontrak, dan strategi test awal | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-08` | Review oracle financial golden dan blokir model belum siap | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S00-09` | Buat decision register dan catatan bukti awal | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** S-00 adalah task audit/approval, bukan fitur yang sudah berjalan.

### S-01 - R0 Engineering Foundation

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S01-01` | Bangun struktur monorepo terverifikasi | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-02` | Siapkan lingkungan PostgreSQL lokal dan CI disposable | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-03` | Siapkan Go API bootstrap dan lifecycle | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-04` | Siapkan Flutter shell dan error/empty/session states dasar | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-05` | Siapkan lifecycle migration PostgreSQL | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-06` | Definisikan boundary principal dan owner di PostgreSQL | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-07` | Buat OpenAPI contract minimum yang valid | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-08` | Implement validasi request dan error middleware | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-09` | Implement auth adapter interface dan deny-by-default guard | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-10` | Buat pure Go financial engine boundary & test harness | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-11` | Siapkan config, secrets, dan privacy-safe observability | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-12` | Buat CI minimal yang benar-benar menjalankan quality checks | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-13` | Rancang dan uji backup/restore pada data sintetis | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S01-14` | Review release gate R0 berdasarkan bukti | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-02 - R1 Identity and Onboarding

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S02-01` | Integrasikan identitas dan session verification Go | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S02-02` | Buat user mapping dan tenant isolation end-to-end | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S02-03` | Buat model profil keuangan dan endpoint validasinya | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S02-04` | Buat Flutter login, restore session, dan logout | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S02-05` | Buat onboarding dan financial profile Flutter | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S02-06` | Tutup acceptance S-02 dengan dua user sintetis | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-03 - R1 Accounts and Transactions

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S03-01` | Tetapkan dan migrasikan account + ledger posting schema | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-02` | Implement account CRUD dan opening balance API | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-03` | Implement pencatatan income/expense atomik | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-04` | Implement transfer atomik, idempotency, dan unknown outcome | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-05` | Implement koreksi/pembatalan sesuai audit policy | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-06` | Buat UI accounts dan opening balance | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-07` | Buat UI income, expense, transfer dan recovery | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S03-08` | Jalankan regression ledger, cross-user, dan concurrency | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-04 - R1 Assets and Net Worth

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S04-01` | Migrate aset, kewajiban dan valuation metadata | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S04-02` | Implement API assets dan liabilities | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S04-03` | Implement net worth dan non-duplication di Go | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S04-04` | Buat API dashboard actual-only | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S04-05` | Bangun dashboard dan assets/liabilities Flutter | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S04-06` | Verify net worth dan period reporting regression | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-05 - R1 Goals and Compound Calculator

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S05-01` | Implement model goal dan aturan status funding | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-02` | Implement Go financial goals API | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-03` | Buat UI goals dan progress yang jujur | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-04` | Setujui calculator contract dan golden oracle sebelum kode | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-05` | Implement pure Go compound calculator dan golden tests | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-06` | Buat endpoint compound calculation yang tervalidasi | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-07` | Buat Flutter compound calculator dengan asumsi eksplisit | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S05-08` | Review integrasi goals dan calculator S-05 | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-06 - R1 Feasibility, Data Rights and R1 Gate

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `TK-S06-01` | Implement required contribution & goal feasibility Go | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-02` | Implement endpoint feasibility dan response provenance | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-03` | Implement UI feasibility dan opsi penyesuaian non-eksekusi | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-04` | Implement safe data export backend | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-05` | Implement secure deletion request/account lifecycle backend | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-06` | Buat Flutter export/delete settings dan confirmation states | `UNVERIFIED` | `NOT_RUN` | - |
| `TK-S06-07` | Review R1 acceptance dan release readiness | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Jangan klaim R0/R1 selesai sebelum test dan acceptance di dokumen DoD.

### S-07 - R2 Budget and Life Events (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S07-01` | Budget & recurring actuals | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S07-02` | Life events as hypothetical planning inputs | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

### S-08 - R2 Projection and Scenarios (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S08-01` | Deterministic projection engine | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S08-02` | Scenario versions & comparison | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S08-03` | Future cash-flow forecast | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

### S-09 - R2 Stress, Insights, Notifications (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S09-01` | Financial stress test | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S09-02` | Explainable rule-based insights | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S09-03` | Consent-first reminders & R2 gate | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

### S-10 - R3 Optimizer and Worker (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S10-01` | Multi-goal optimization engine | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S10-02` | Bounded background jobs foundation | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

### S-11 - R3 Monte Carlo and Retirement (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S11-01` | Seeded Monte Carlo risk simulation | `UNVERIFIED` | `NOT_RUN` | - |
| `EP-S11-02` | Retirement/FIRE accumulation & decumulation | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

### S-12 - R3 Financial GPS (Epic only)

| Work ID | Ringkasan dari TASKS.md | Implementasi | Test | Bukti |
|---|---|---|---|---|
| `EP-S12-01` | Financial GPS & R3 release gate | `UNVERIFIED` | `NOT_RUN` | - |

**Gate:** Epic ini belum menjadi task implementasi production yang diotorisasi.

## 7. Decision watchlist - pantauan, bukan catatan approval

ID di tabel ini berasal dari `MASTER-ROADMAP.md` / `TASKS.md` (owner keputusan: PRD dan dokumen domain terkait). **Tidak ada approval keputusan individual yang direkam oleh penyusunan `PROGRESS.md`**. Periksa status paling mutakhir dan bukti persetujuan sebelum task yang bergantung pada keputusan tersebut dimulai.

| Decision ID | Area | First impacted slice | Risiko bila diabaikan | Record approval di ledger ini |
|---|---|---|---|---|
| `Q-01` | Identitas/scope MVP | `S-00` | Scope creep/branding dianggap final | `NOT_RECORDED` |
| `Q-02` | Authentication & session | `S-01`/`S-02` | Akses data tanpa principal yang benar | `NOT_RECORDED` |
| `Q-03` | Ledger/opening balance/correction | `S-01`/`S-03` | Saldo/transfer/koreksi keliru | `NOT_RECORDED` |
| `Q-04` | Monetary precision, time and rounding | `S-01`/`S-03`/`S-05` | Angka finansial tidak konsisten | `NOT_RECORDED` |
| `Q-05` | Inflation/returns/debt modeling | `S-07`/`S-08` | Proyeksi mengarang asumsi | `NOT_RECORDED` |
| `Q-06` | Scenario baseline/version/staleness | `S-07`/`S-08` | Forecast terlihat aktual; data stale | `NOT_RECORDED` |
| `Q-07` | Optimizer objective/constraints | `S-10` | Klaim solusi optimal tanpa dasar | `NOT_RECORDED` |
| `Q-08` | Stochastic risk model | `S-11` | Simulasi probabilistik menyesatkan | `NOT_RECORDED` |
| `Q-09` | Notifications/consent | `S-09` | Pelanggaran privasi / spam | `NOT_RECORDED` |
| `Q-10` | Hosting/backup/retention/legal review | `S-00`, prod R1 | Kehilangan data/pelanggaran privasi | `NOT_RECORDED` |
| `Q-11` | Actual goal funding semantics | `S-05` | Planned contribution dihitung sebagai actual | `NOT_RECORDED` |

Keputusan tambahan seperti `BE-Q*`, `FE-Q*`, `DB-Q*`, `API-Q*`, `TS-Q*`, `DW-Q*`, dan `DOD-Q*` tetap dimiliki dokumen sumber masing-masing. `DECISION-LOG.md` nantinya dapat mengindeks **approval nyata**, tetapi tidak boleh menduplikasi penetapan kebijakan baru. Jika approval tersedia di luar file ini, tambahkan referensi yang dapat ditinjau; jangan mengubah `NOT_RECORDED` menjadi `DECIDED` berdasarkan dugaan.

## 8. Current blockers, risks and drift register

**Verified active blockers from application repository:** belum ada yang dapat dipastikan karena repository belum diaudit. Tabel di bawah adalah **gate yang diketahui dari dokumen**, bukan klaim bahwa semua task saat ini berstatus `BLOCKED`.

| Gate category | Mengapa perlu diperiksa | Pengaruh pada task | Status observasi |
|---|---|---|---|
| Repo & environment baseline | Komponen yang sudah ada belum diketahui | `TK-S00-01`, seluruh task berikutnya | `UNVERIFIED` |
| Product/architecture approvals | Approval task dan keputusan tidak boleh diinferensikan | `TK-S00-02` s.d. `TK-S00-09` | `NOT_RECORDED` |
| Identity and ownership | Token/principal dan user isolation harus benar | `TK-S01-06`, `TK-S01-09`, S-02/S-03 | `DECISION_GATE` |
| Financial ledger/currency | Posting, koreksi, presisi dan golden values | S-03/S-04/S-05/S-06 | `DECISION_GATE` |
| Model assumptions/advanced specs | SCENARIO / OPTIMIZER / MONTE-CARLO belum otomatis disetujui | S-07 sampai S-12 | `DECISION_GATE` |
| Release/privacy readiness | Test, backup/restore, data rights, review | Semua release termasuk R1 | `UNVERIFIED` |

### 8.1 Verified incident / blocker records

Tidak ada insiden kode, test fail, regression, atau blocker implementasi yang sudah diverifikasi dalam pekerjaan pembuatan dokumen ini. **Jangan menambahkan baris sintetik sebagai insiden nyata.** Saat ditemukan, tambahkan record seperti template di bawah.

```yaml
blocker_id: null                    # Assign when a real blocker is found
observed_at: null                   # ISO-8601 with timezone
work_id: null                       # Existing TK-/EP- ID
kind: null                          # DECISION | SECURITY | FINANCIAL | TEST | INFRA | DRIFT
severity: null                      # Determined from impact, not guessed
observed_evidence_ref: null         # Sanitized log/file/PR reference
impact: null
owner_of_resolution: null
next_safe_action: null
status: OPEN
resolved_at: null
resolution_evidence_ref: null
```

**Financial/security hard stop:** dilarang menganggap transfer sukses saat outcome tidak diketahui, menyamakan planned funds dengan aktual, menampilkan hasil proyeksi sebagai saldo riil, atau membaca data pengguna lain. Kegagalan terkait harus tetap terlihat sampai diperbaiki dan diverifikasi ulang.

## 9. Evidence schema dan verifikasi yang boleh dilaporkan

Rekam **bukti minimum per task**, bukan test summary palsu. Referensi boleh berupa commit hash, PR, path log aman, CI run URL, atau laporan review yang benar-benar ada. Jangan membuat URL atau hash berdasarkan pola. Bila artifact belum tersedia, gunakan `null` dan status yang benar.

```yaml
# TEMPLATE ONLY - NOT A LIVE TASK UPDATE
work_id: TK-S03-04
session_id: null
reported_at: null
branch: null
commit_sha: null
implementation_status: UNVERIFIED
planning_status: DRAFT
approved_task_scope_ref: null
source_requirements: [FR-004]
decision_refs: [Q-03, Q-04]
files_changed: []
verification:
  - check: disposable PostgreSQL transfer / rollback / idempotency
    command_or_procedure: null
    environment: null
    status: NOT_RUN
    exit_code: null
    evidence_ref: null
applicable_dod_gates: []
critical_findings: []
blocker_ids: []
reviewed_by: null
accepted_by: null
next_step: null
```

Minimum untuk setiap test run:

| Field | Wajib | Ketentuan |
|---|---|---|
| Work ID dan requirement/test ID | Ya | Referensi `TK-`, `FR-*`, `TS-*`, `FTC-*`, `ST-*` yang benar |
| Environment dan code identity | Ya jika code diuji | Version/runtime, DB disposable, commit/branch atau jelaskan tidak ada |
| Executed command/procedure | Ya untuk `PASS`/`FAIL` | Perintah/prosedur asli, bukan command usulan |
| Exit code / observed output | Ya jika tersedia | Jangan mengubah failure menjadi PASS |
| Evidence reference | Ya untuk klaim PASS | Log terpotong dan tersanitasi, artifact CI, hasil uji manual teridentifikasi |
| Limitations dan relevant exclusions | Ya | Device yang tidak diuji, acceptance yang belum diputuskan |
| Reviewer and approval | Jika disyaratkan DoD | Nama/identitas dari bukti asli saja; jangan diisi otomatis |

**Tidak ada live test result dalam baseline dokumen ini.** `TESTING-STRATEGY.md` dan `FINANCIAL-TEST-CASES.md` adalah perencanaan/golden references, bukan log eksekusi Go/Flutter/PostgreSQL.

## 10. Pembaruan progres saat satu task dikerjakan

1. **Identify:** ambil `TK-*` dari `TASKS.md` dan baca dependency/acceptance yang relevan. Untuk R2/R3, pecah `EP-*` menjadi task disetujui terlebih dahulu.
2. **Inspect repo:** catat branch/commit aktual, path terkait, file yang ada, keadaan Git (`dirty` atau bersih), versi tool dan test environment yang benar-benar terlihat. Bila akses gagal, tetap `UNVERIFIED`.
3. **Trace:** petakan `FR/EN/NFR`, `DR`, `CS`, `FTC`, `API`, dan `ST/UXT/TS` yang terpengaruh. Catat approval untuk keputusan yang *tepat relevan*, bukan seluruh proyek.
4. **Plan:** tulis scope kecil, file changes yang direncanakan, dampak ke API/DB/Flutter, test plan dan rollback bila operasi sensitif.
5. **Authorize:** baru ubah `READY` jika DoR benar-benar terpenuhi. Approval untuk satu task tidak otomatis mengesahkan seluruh dependensi proyek.
6. **Implement:** update `IN_PROGRESS` dengan work order. Jangan ubah status unrelated task, ubah security/financial rules diam-diam, atau melakukan destructive production operations.
7. **Verify:** jalankan test yang relevan. Catat command+environment+exit code+hasil dan review keluaran, termasuk integration test PostgreSQL untuk transaksi uang.
8. **Review:** catat `IN_REVIEW` dan checklist `DOD-G01`-`DOD-G10` yang relevan; reviewer harus bisa membuka bukti yang dirujuk.
9. **Close:** `DONE` hanya setelah acceptance benar-benar terpenuhi; jika tidak, gunakan `PARTIAL`, `BLOCKED`, `IN_REVIEW`, atau `UNVERIFIED` yang tepat.
10. **Handoff:** append log aktivitas (jangan hapus histori), update baris register + dashboard berdasarkan bukti, serahkan next bounded task; bila ada keputusan baru, catat pada source document/`DECISION-LOG.md` saat tersedia.

**Atomic update rule:** perubahan `TASKS.md` untuk scope dan `PROGRESS.md` untuk status harus konsisten berdasarkan ID yang sama. Jangan overwrite perubahan agent lain tanpa merge/reconciliation. Simpan histori perubahan status bila proses kolaboratif memerlukannya.

## 11. Session log (append-only)

Catatan session dibuat **setelah pekerjaan benar-benar dilakukan**, bukan setiap kali user meminta ide. `session_id` bisa memakai pola `SESS-YYYYMMDD-NNN` bila sudah diterapkan, tetapi bukan identitas log yang dianggap valid sebelum ada event asli. Setiap entri harus membedakan artefak dokumentasi dan pengujian aplikasi.

### 2026-10-30 - Documentation baseline: PROGRESS.md

- **Aktivitas:** menyusun baseline status dari `TASKS.md`, `MASTER-ROADMAP.md`, `DEVELOPMENT-WORKFLOW.md`, `TESTING-STRATEGY.md`, `DEFINITION-OF-DONE.md`, dan instruksi agent.
- **Output:** `docs/planning/PROGRESS.md` v0.1.0, beserta arsip ZIP tunggal sebagai delivery artifact.
- **Scope:** document-only; tidak mengedit Flutter source, Go source, PostgreSQL schema, data pengguna, atau infrastruktur production.
- **Observed work items:** 58 `TK-*` + 13 `EP-*`; semua tetap `UNVERIFIED` untuk implementasi dan `NOT_RUN` untuk application test.
- **Documentation QA:** pemeriksaan jumlah/keunikan ID, slice mapping, struktur Markdown, dan isi ZIP. Hasil teknis diverifikasi sebagai bagian dari pembuatan artefak; detail checksum dapat diperiksa pada paket yang diunduh.
- **Application QA:** `NOT_RUN` - repository aplikasi tidak diinspeksi, tidak ada build/Go test/Flutter test/PostgreSQL integration test dijalankan.
- **Owner approval:** belum tercatat untuk baseline dokumen ini.
- **Next proposed action:** audit repository aktual sebagai `TK-S00-01` setelah repo tersedia dan task tersebut diotorisasi.

### Template entry untuk session berikutnya

```markdown
### YYYY-MM-DD - [Real work summary] ([Real session ID])

- Task: `TK-Sxx-nn` (link to TASKS.md)
- Status before -> after: ... -> ...
- Owner authorization: actual reference or NOT_RECORDED
- Repo / branch / commit: actual values or UNVERIFIED
- Files changed: exact paths only
- Tests: [check / environment / result / evidence]
- Security and finance review: exact results or NOT_RUN
- Problems found and mitigation: ...
- Decision/blocker refs: ...
- Reviewer/acceptance: ...
- Next authorized work: ...
```

Jangan menyalin template sebagai progress log yang sudah terlaksana.

## 12. Handoff summary untuk owner dan agent berikutnya

Setiap akhir task, tulis ringkasan singkat yang bisa dipahami tanpa membuka seluruh percakapan:

```markdown
## Handoff: TK-Sxx-nn - [Title]

**Implementation status:** UNVERIFIED / NOT_STARTED / READY / IN_PROGRESS / IN_REVIEW / DONE / PARTIAL / BLOCKED
**Authorized scope:** [actual reference]
**Evidence inspected:** [repo revision, source files, relevant documents]
**Implemented behavior:** [facts only; say none if documentation-only]
**Files changed:** [exact paths]
**Tests executed:**
- [name] - [environment] - PASS/FAIL/NOT_RUN/BLOCKED - [evidence]
**DoD:** [relevant DOD-Gxx with evidence / N/A reason]
**Known limitations / security / finance risks:** [...] 
**Open decision IDs:** [...] 
**Reviewer / owner acceptance:** [actual approval or NOT_RECORDED]
**Next bounded task:** [exact TASKS.md ID, subject to DoR]
```

**No implicit claims:** Jika agent hanya membuat dokumentasi, katakan "documentation-only". Jika hanya unit test yang lolos, jangan mengklaim integrasi database, Flutter E2E, atau keamanan production telah lolos.

## 13. Release readiness and gate checklist

### R0 gate - engineering foundation

- [ ] Repositori diinspeksi dan struktur monorepo terkonfirmasi.
- [ ] Flutter shell, Go API bootstrap, disposable PostgreSQL, migration lifecycle, basic API contract terverifikasi jika sesuai scope yang disetujui.
- [ ] Auth boundary deny-by-default dan ownership controls diuji dalam batas R0.
- [ ] Financial engine test harness sudah ada dan dapat dijalankan (belum menyamakan ini dengan compound implementation R1).
- [ ] CI benar-benar mengeksekusi quality checks yang relevan; bukan workflow placeholder.
- [ ] Backup/restore rehearsal dengan data sintetis jika disyaratkan scope R0.
- [ ] Seluruh gate DoD R0 dipenuhi dan acceptance owner direkam.

### R1 gate - MVP keuangan aktual

- [ ] User identity, session lifecycle dan isolation antarpengguna diverifikasi.
- [ ] Income/expense, transfer, koreksi dan opening balances menjaga ledger invariant dengan DB integration tests.
- [ ] Assets, liabilities, net worth menghindari double count dan dibuktikan dengan golden oracles.
- [ ] Goals tidak mengklaim "funded" hanya dari rencana setoran.
- [ ] Compound + feasibility menggunakan kontrak perhitungan dan FTC yang relevan, termasuk rate nol, timing, rounding.
- [ ] Export/deletion controls, privacy dan retry/unknown outcome di-review.
- [ ] Regression R0/R1 serta owner acceptance direkam tanpa mengarang hasil test.

### R2 gate - deterministic planning

- [ ] Scenario dan actual data terisolasi.
- [ ] Versioning, baseline revision, staleness dan deterministic replay memenuhi spesifikasi yang disetujui.
- [ ] Cash-flow, stress, insight, notifications memiliki asumsi/provenance dan consent yang jelas.
- [ ] QA mencakup R1 regression, privacy dan explainability.

### R3 gate - advanced simulation

- [ ] Multi-goal optimizer memiliki objective/constraints dan reference optimum yang disetujui.
- [ ] Monte Carlo memiliki model statistik, seed/replay, probabilistic test bands, uncertainty caveats dan resource budgets yang disetujui.
- [ ] Worker retry/cancel/deduplication/ownership/retention aman dan teruji.
- [ ] Retirement/FIRE dan Financial GPS menjelaskan risiko; tidak menjanjikan hasil atau mengeksekusi transaksi secara otomatis.
- [ ] Regression R0-R3 dan release acceptance direkam.

Semua checkbox di atas **kosong** karena tidak ada pengujian aplikasi yang dilakukan saat membuat dokumen ini. Checklist adalah rancangan readiness gate, bukan bukti selesai.

## 14. Anti-hallucination & handover controls

1. **Never infer a working repo from docs.** Tidak ada `go.mod`, `pubspec.yaml`, migration, API server, atau CI yang diklaim ada sampai benar-benar diinspeksi.
2. **Never infer approval from a generated file.** Semua dokumen sebelumnya masih perlu review/persetujuan owner sesuai status yang tercatat.
3. **Never change expected numeric results to green a failing test.** Reconcile `FINANCIAL-TEST-CASES.md` + calculation spec + independent oracle.
4. **Never collapse actual, planned and simulated money.** Actual ledger tidak boleh bermutasi karena goal/scenario.
5. **Never display fabricated completion metrics.** Jika numerator atau denominator didasarkan pada task yang belum diaudit, tulis `UNVERIFIED`, bukan angka pasti.
6. **Never hide unknown outcome.** Saat timeout pada money posting, cek status via idempotency/reconciliation sebelum retry, jangan mengklaim sukses.
7. **Never leave unapproved infrastructure choices behind as facts.** Provider auth/queues/framework hanya menjadi kandidat sampai disetujui.
8. **Keep event history.** Failures, approvals, changes, dan rollback dicatat sebagai peristiwa baru; jangan overwrite audit trail.
9. **Report only what was observed.** Jika tool atau izin tidak tersedia, catat `NOT_RUN`/`UNVERIFIED` dan dampaknya pada penerimaan task.
10. **Prefer small verified work.** Satu task berikutnya yang terdefinisi jelas lebih aman daripada banyak task yang hanya berstatus UI done.

### Suggested minimal context for a new coding agent

1. `AGENTS.md` + `frontend/AGENTS.md` atau `backend/AGENTS.md` yang relevan.
2. `docs/planning/PROGRESS.md` bagian baseline, task register, session log, blocker, current task.
3. `docs/planning/TASKS.md` untuk task yang dipilih + dependensi.
4. `docs/planning/MASTER-ROADMAP.md` untuk gate rilis dan urutan.
5. Dokumen domain/arsitektur/API/UX/tes **yang diperlukan task tersebut saja**.
6. Inspeksi real repo sebelum membuat diagnosis, patch, atau klaim status.

## 15. Updating summary without misleading metrics

Jika seluruh task terkait sudah diinspeksi, metrik dapat dihitung dari registry yang valid, misalnya:

- `audited_items`: jumlah task yang memiliki evidence audit repository.
- `verified_done_items`: jumlah task status `DONE` yang memiliki DoD dan acceptance lengkap.
- `active_items`: `IN_PROGRESS`/`IN_REVIEW` dengan task ID dan work order sah.
- `blocked_items`: `BLOCKED` dengan referensi blocker yang spesifik.
- `test_pass_rate`: hanya untuk test yang benar-benar dijalankan pada scope/versi/env yang sama; tampilkan denominator dan exclusions.

Jangan mengubah "71 item telah direncanakan" menjadi "71 tugas yang belum dikerjakan". Persentase completion hanya boleh dihitung setelah audit cukup lengkap dan status dapat dipercaya; jika belum, tampilkan proporsi yang sudah diaudit dengan label jelas, bukan progress produk yang menyesatkan.

## 16. Maintenance rules and consistency checks

- **Jika scope TASKS berubah:** update ID/task metadata (jangan pernah reuse ID), perbarui register di sini dan hitung ulang slice dashboard. Task yang dihapus hanya karena re-plan tetap memiliki sejarah `CANCELLED`/`SUPERSEDED` jika sebelumnya sudah ada evidence.
- **Jika requirement berubah:** periksa PRD, Feature Matrix, Master Roadmap dan affected acceptance/tests terlebih dulu; perubahan status di sini tidak mengubah requirement.
- **Jika keputusan disetujui:** catat sumber approval dan perbarui owner document serta `DECISION-LOG.md` bila sudah dibuat, baru ubah status terkait di sini.
- **Jika test gagal:** tulis environment, observed failure dan follow-up; jangan memaksa `PASS`, jangan mengedit golden oracle tanpa analisis.
- **Jika terjadi security incident:** batasi bukti sensitif, ikuti `SECURITY-PRIVACY.md`, catat jejak dan stop task berisiko.
- **Jika multi-agent:** pastikan satu work item punya ownership jelas, perubahan ke file ledger diserialisasi atau digabung hati-hati; jangan membiarkan status diklaim selesai oleh dua agent tanpa review.
- **Jika deploy:** catat image/artifact identity, migration version, environment, waktu, sign-off, rollback readiness dan health checks aktual; jangan klaim deploy hanya dari Dockerfile.

**Next proposed task:** `TK-S00-01` - audit monorepo, codebase, environment, migration/API status dan test commands; setelah audit, perbarui semua status yang dapat dibuktikan dan tandai sisa `UNVERIFIED`. Keputusan yang dapat memblokir fase kemudian ditangani melalui `TK-S00-02`-`TK-S00-09` tanpa bulk approval.

---

## 17. Source inventory (documentation references)

Katalog di bawah membantu agent menentukan dokumen yang perlu dibaca. Link ditulis relatif terhadap path target `docs/planning/PROGRESS.md`; berfungsi setelah file ditempatkan ke monorepo bersama dokumen terkait. Referensi tidak menjamin dokumen sudah committed dalam repository pengguna.

| No. | Source | Canonical relative link |
|---:|---|---|
| 01 | Product Vision | [`PRODUCT-VISION.md`](../product/PRODUCT-VISION.md) |
| 02 | PRD | [`PRD.md`](../product/PRD.md) |
| 03 | Feature Matrix | [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md) |
| 04 | Financial Domain Rules | [`FINANCIAL-DOMAIN-RULES.md`](../financial/FINANCIAL-DOMAIN-RULES.md) |
| 05 | Calculation Specs | [`CALCULATION-SPECS.md`](../financial/CALCULATION-SPECS.md) |
| 06 | Financial Test Cases | [`FINANCIAL-TEST-CASES.md`](../financial/FINANCIAL-TEST-CASES.md) |
| 07 | System Architecture | [`SYSTEM-ARCHITECTURE.md`](../architecture/SYSTEM-ARCHITECTURE.md) |
| 08 | Backend Architecture | [`BACKEND-ARCHITECTURE.md`](../architecture/BACKEND-ARCHITECTURE.md) |
| 09 | Frontend Architecture | [`FRONTEND-ARCHITECTURE.md`](../architecture/FRONTEND-ARCHITECTURE.md) |
| 10 | Database Design | [`DATABASE-DESIGN.md`](../architecture/DATABASE-DESIGN.md) |
| 11 | API Guidelines | [`API-GUIDELINES.md`](../architecture/API-GUIDELINES.md) |
| 12 | Design System | [`DESIGN-SYSTEM.md`](../design/DESIGN-SYSTEM.md) |
| 13 | User Flows | [`USER-FLOWS.md`](../design/USER-FLOWS.md) |
| 14 | Screen Specs | [`SCREEN-SPECS.md`](../design/SCREEN-SPECS.md) |
| 15 | Security & Privacy | [`SECURITY-PRIVACY.md`](../security/SECURITY-PRIVACY.md) |
| 16 | Root AGENTS | [`AGENTS.md`](../../AGENTS.md) |
| 17 | Flutter AGENTS | [`AGENTS.md`](../../frontend/AGENTS.md) |
| 18 | Backend AGENTS | [`AGENTS.md`](../../backend/AGENTS.md) |
| 19 | Development Workflow | [`DEVELOPMENT-WORKFLOW.md`](../development/DEVELOPMENT-WORKFLOW.md) |
| 20 | Testing Strategy | [`TESTING-STRATEGY.md`](../development/TESTING-STRATEGY.md) |
| 21 | Definition of Done | [`DEFINITION-OF-DONE.md`](../development/DEFINITION-OF-DONE.md) |
| 22 | Master Roadmap | [`MASTER-ROADMAP.md`](./MASTER-ROADMAP.md) |
| 23 | Tasks | [`TASKS.md`](./TASKS.md) |

## 18. Internal QA checklist for this document

- [x] Task register dibangun dari 58 heading `TK-*` dan 13 heading `EP-*` dari `TASKS.md` v0.1.0.
- [x] Semua 71 ID unik, tanpa ID dummy yang dianggap task nyata.
- [x] Cakupan S-00 sampai S-12 dan pembagian R0-R3 sesuai `MASTER-ROADMAP.md`.
- [x] Semua implementasi `UNVERIFIED`, semua test aplikasi `NOT_RUN`; tidak ada progress rekayasa.
- [x] `DECIDED` tidak diinferensikan dari rekomendasi atau pembuatan Markdown.
- [x] Istilah DoD dan workflow cocok dengan sumbernya (`DOD-G01`-`DOD-G10`, decision gate, actual-vs-scenario).
- [x] Tidak ada identitas pengguna asli, token, credential, nomor rekening, atau tautan log palsu.
- [x] Source links diarahkan ke path canonical; existence dalam repository tetap harus diaudit.
- [x] Struktur Markdown, status rows, referensi task, dan integritas arsip diperiksa setelah generation.

**QA boundary:** pemeriksaan ini adalah *documentation/packaging QA*; ia tidak menjalankan test Flutter, Go, PostgreSQL atau membuktikan kecocokan implementasi dengan spesifikasi.

**Document status:** `REVIEWED_INTERNAL - PENDING_OWNER_APPROVAL`. Pengesahan Product Owner baru boleh direkam setelah diberikan secara eksplisit. Dokumen berikutnya yang disarankan adalah `DECISION-LOG.md` atau audit `TK-S00-01` ketika repository siap; keduanya bukan pekerjaan yang sudah dilakukan.

