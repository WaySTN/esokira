# Esokira — Financial Test Cases & Golden Reference Vectors

> **Canonical path:** `docs/financial/FINANCIAL-TEST-CASES.md`  
> **Versi:** `0.1.0`  
> **Tanggal:** 29 Oktober 2026  
> **Status dokumen:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Rilis cakupan:** R0 Foundation, R1 MVP, R2 Planning V1, R3 Advanced; R4 **tidak** menjadi komitmen pengembangan  
> **Referensi induk:** [`PRODUCT-VISION.md`](../product/PRODUCT-VISION.md), [`PRD.md`](../product/PRD.md), [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md), [`FINANCIAL-DOMAIN-RULES.md`](./FINANCIAL-DOMAIN-RULES.md), [`CALCULATION-SPECS.md`](./CALCULATION-SPECS.md), seluruhnya v0.1.0  
> **Target implementasi kelak:** unit tests pure Go financial engine, PostgreSQL integration tests, API contract tests, Flutter presentation tests  
> **Dokumen selanjutnya:** `SYSTEM-ARCHITECTURE.md` setelah review tahap financial; detail spesifik R2/R3 tetap memerlukan `SCENARIO-ENGINE.md`, `GOAL-OPTIMIZER.md`, dan `MONTE-CARLO.md` sebelum production.

---

## 0. Tujuan dan batas dokumen

Dokumen ini memberikan **nilai referensi independen, skenario pengujian, invariants, edge cases, dan kriteria bukti** agar coding agent tidak mengarang expected output. Tujuan utamanya adalah mencegah kalkulasi yang secara sintaks valid tetapi secara finansial salah. Setiap test mengacu pada `CS-001` s.d. `CS-016` dalam Calculation Specs serta aturan `DR-001` s.d. `DR-024` dalam Domain Rules.

**Bukan** dokumen persetujuan algoritma atau policy production. Nilai yang tepat di bawah asumsi tertentu **tidak** otomatis menyetujui asumsi produk, definisi currency, rounding, timing, accrual, maupun distribusi probabilistik. Tidak ada klaim bahwa kode Go, SQL, API, Flutter, atau CI untuk Esokira telah dibuat atau diuji.

### 0.1 Urutan otoritas

1. `PRD.md` → scope, user outcome, FR/NFR, acceptance, release.
2. `FINANCIAL-DOMAIN-RULES.md` → definisi actual/planned/forecast, ownership, accounting invariants.
3. `CALCULATION-SPECS.md` → rumus, notasi, unit, asumsi, batas model, blocker `CALC-Qxx`.
4. **Dokumen ini** → reference vectors, test descriptions, expected behavior **jika premis eksplisit dipenuhi**.
5. Kode Go, migrasi SQL, API, UI → implementasi yang diuji terhadap dokumen; bukan otoritas untuk menimpa kebijakan domain.

Jika ada konflik antara dokumen induk dan test: **tahan pengujian yang terdampak, catat konflik, minta review**, dan jangan diam-diam mengubah expected agar tes hijau.

### 0.2 Label kesiapan setiap kasus

| Label | Arti | Penggunaan oleh agent |
|---|---|---|
| `REF-EXACT` | Hasil eksak pada asumsi eksplisit; dihitung memakai bilangan bulat/desimal/rational sebagai referensi. | Boleh diterjemahkan ke test matematis; kontrak production tetap perlu approval. |
| `REF-APPROX` | Hasil tidak dinyatakan sebagai pecahan desimal hingga, atau membutuhkan akar/pangkat. | Gunakan ekspresi matematis dan aturan toleransi yang disetujui; digit tampilan bukan string equality. |
| `DOMAIN-INVARIANT` | Sifat yang harus selalu dijaga dalam ruang lingkup aturan terkait. | Uji pada service, DB, dan API sesuai batas modul; tidak hanya pada angka. |
| `CONDITIONAL` | Reference test hanya valid jika konvensi contoh dinyatakan eksplisit. | Jangan menaikkannya menjadi *production golden test* sebelum keputusan terkait disetujui. |
| `BLOCKED` | Test model belum punya definisi input/algoritma/hasil final. | Boleh uji guard dan error; **dilarang mengisi distribusi, objective, atau default sendiri**. |
| `INTEGRATION-SPEC` | Perilaku sistem yang membutuhkan transaksi SQL, concurrency, auth, worker, atau API. | Eksekusi setelah modul/infrastruktur tersedia; dokumen ini tidak mengklaim test sudah dijalankan. |

Semua golden references dalam dokumen ini berstatus review internal; **belum di-approve sebagai kebijakan rilis oleh Product Owner**.

### 0.3 Prinsip yang tidak dapat ditawar

- `ACTUAL`, `PLANNED`, `FORECAST`, dan `STOCHASTIC_RESULT` tidak boleh tertukar (DR-001/002/011/012).
- Transfer sesama account tanpa fee/FX bukan income atau expense (DR-006), dan posting kedua sisinya atomik serta idempotent (DR-007/008).
- Satu aset hanya dihitung satu kali; pokok utang tidak sama dengan bunga/expense (DR-009/010/023).
- Nominal aktual mempunyai representasi exact; jangan gunakan binary floating-point sebagai sumber saldo (DR-004).
- Setiap angka memiliki mata uang, satuan, basis waktu, parameter, serta label sumber; input yang belum cukup menghasilkan error/status, **bukan nilai tebakan** (DR-014/017).
- Semua perhitungan skenario bersifat read-only terhadap actual (DR-002), dengan baseline, versi model, serta seed jika relevan (DR-016/020).
- Tidak ada validasi algoritma stochastic/optimizer tanpa spesifikasi khusus dan persetujuan model (DR-024).

---

## 1. Format kontrak test dan data sintetis

### 1.1 Field minimum

| Field | Keterangan |
|---|---|
| `case_id` | ID stabil `FTC-<AREA>-NN`; jangan pakai ulang ID yang sudah dipublikasikan. |
| `title`, `release` | Nama kasus, R0/R1/R2/R3. |
| `references` | `CS-xxx`, `CALC-REF-xx` jika ada, `FR-xxx`, `DR-xxx`, dan blocker terkait. |
| `readiness` | `REF-EXACT`, `REF-APPROX`, `DOMAIN-INVARIANT`, `CONDITIONAL`, `BLOCKED`, atau `INTEGRATION-SPEC`. |
| `given` | Seluruh input dan state awal, currency, unit, horizon, `rate_kind`, timing, rounding jika relevan. |
| `when` | Operasi yang dilakukan; untuk integration, failure injection atau parallel requests jika perlu. |
| `then` | Expected yang bisa diamati: nilai, status, error, invariants, dan larangan side effects. |
| `comparison` | `exact`, `symbolic`, atau `approx` dengan policy toleransi yang nantinya disetujui. |
| `evidence` | Referensi arithmetic/SQL log/assertion; **hasil tes Go hanya sah jika benar-benar dijalankan**. |

Jangan mencantumkan data pengguna sebenarnya dalam fixtures. Seluruh account ID, user ID, dan nominal di dokumen ini sintetis.

### 1.2 Contoh fixture konseptual (BUKAN API JSON final)

```json
{
  "case_id": "FTC-FV-02",
  "readiness": "REF-EXACT",
  "release": "R1",
  "references": ["CS-005", "CALC-REF-05", "FR-008"],
  "given": {
    "currency": "IDR",
    "amount_unit": "rupiah",
    "opening_principal": "0",
    "contribution_per_period": "100000",
    "period_count": 2,
    "period_unit": "month",
    "effective_rate_per_period": "0.01",
    "contribution_timing": "end",
    "fees": "0",
    "taxes": "0"
  },
  "when": "calculate_fixed_rate_future_value",
  "then": {"future_value_unrounded": "201000"},
  "comparison": "exact_decimal",
  "policy_gate": ["CALC-Q01", "CALC-Q02", "CALC-Q04"]
}
```

Angka di JSON adalah **string desimal untuk ilustrasi representasi yang tidak kehilangan presisi**, bukan keputusan bahwa semua endpoint harus memakai string. Kontrak Go struct, JSON API, PostgreSQL storage, dan formatter final masih menunggu keputusan arsitektur.

### 1.3 Profil asumsi referensi `REF-PROFILE-A`

Berlaku **hanya** untuk contoh yang secara eksplisit menyebut profil ini:

- `currency = IDR`, unit input nominal rupiah; nominal contoh tidak memiliki pecahan rupiah kecuali disebut sebagai nilai *unrounded*.
- Periode `month` berarti interval penuh dan diskret, `t=0` tepat sebelum periode pertama; `n` bilangan bulat nonnegatif.
- Bunga/return **efektif per periode** dikirim langsung sebagai desimal (`0.01` = 1% per bulan); bila annual, tipe dan konversinya disebut terpisah.
- Setoran di **akhir periode** (`end`) kecuali test secara eksplisit meminta `begin`.
- Tidak ada fee, pajak, withdrawal, FX, dan perubahan nilai pasar di luar rate yang dinyatakan.
- Nilai numerik diharapkan sebelum pembulatan tampilan; **tidak menetapkan production rounding mode atau default yield**.
- Semua test proses transaksi harus memperlakukan perubahan sebagai state sintetis, bukan akses data pengguna riil.

**Larangan:** jangan menyalin `REF-PROFILE-A` sebagai setting default aplikasi atau menghilangkan field asumsi hanya karena kasus ilustrasi memakai angka sederhana.

---

## 2. Kebijakan perbandingan dan akurasi

| Kelas nilai | Aturan pemeriksaan | Catatan |
|---|---|---|
| Money actual integer, balance, transfer, net worth | **Exact equality** dalam unit yang disetujui. | Contoh memakai integer rupiah; desain unit asli masih `CALC-Q01`. |
| Future value dengan rate desimal berhingga dan n kecil | **Exact decimal reference** pada angka *unrounded* bila memungkinkan. | Jika produksi memakai algoritma aproksimasi, butuh toleransi yang disetujui, bukan `==` float acak. |
| Rate annual-to-monthly, `(1+r)^(1/12)` | **Symbolic equation + high-precision reference**, bukan exact string UI. | Batas galat absolut/relatif dan precision belum final (`CALC-Q02`, `CALC-Q15`). |
| Pembulatan setoran minimum, jadwal cicilan | Uji expected *unrounded* terlebih dahulu. | Expected angka yang sudah dibulatkan menjadi `BLOCKED` sampai rounding policy ditetapkan. |
| Percentile/Monte Carlo | Uji agregasi aljabar yang tak tergantung distribusi. | Percentile algorithm, RNG, distribusi, seed policy masih `CALC-Q13`. |
| Semua hasil | Cek unit, klasifikasi actual/forecast, status, warnings, asumsi dan side effects. | **Lulus angka saja tidak cukup**. |

**Dilarang** mendefinisikan toleransi numerik global, misalnya `abs(actual-expected)<1e-6`, sebagai standar produksi tanpa persetujuan `CALC-Q15`. Selain toleransi, perlu eksplisit batas maksimum jumlah periode, besar nilai, dan cara menangani overflow/NaN/Inf. Untuk angka pembanding high precision di bawah, digit panjang disediakan guna audit, bukan untuk dibandingkan sebagai string hasil pemformatan.

---

## 3. Traceability dari 16 reference arithmetic induk

Semua `CALC-REF-01` s.d. `CALC-REF-16` dari Calculation Specs §20 diwakili di sini; tanda `CONDITIONAL` berarti **contoh valid** tanpa menjadikannya policy rilis final.

| Reference induk | Case utama | Referensi kontrak | Release | Kesiapan |
|---|---|---|---|---|
| `CALC-REF-01` | `FTC-ACT-01` | `CS-003` | R1 | `CONDITIONAL` posting |
| `CALC-REF-02` | `FTC-ACT-02` | `CS-003` | R1 | `DOMAIN-INVARIANT` |
| `CALC-REF-03` | `FTC-NW-01` | `CS-004` | R1 | `REF-EXACT` |
| `CALC-REF-04` | `FTC-FV-01` | `CS-005` | R1 | `REF-EXACT` |
| `CALC-REF-05` | `FTC-FV-02` | `CS-005` | R1 | `REF-EXACT` |
| `CALC-REF-06` | `FTC-FV-03` | `CS-005` | R1 | `REF-EXACT` |
| `CALC-REF-07` | `FTC-FV-04` | `CS-005` | R1 | `REF-EXACT` |
| `CALC-REF-08` | `FTC-GOAL-01` | `CS-007` | R1 | `CONDITIONAL` goal attribution |
| `CALC-REF-09` | `FTC-GOAL-02` | `CS-007` | R1 | `CONDITIONAL` goal attribution |
| `CALC-REF-10` | `FTC-INF-01` | `CS-006` | R1–R3 | `REF-EXACT` math |
| `CALC-REF-11` | `FTC-CASH-01` | `CS-009` | R2 | `CONDITIONAL` event ordering |
| `CALC-REF-12` | `FTC-LOAN-01` | `CS-010` | R2/R3 | `CONDITIONAL` loan model |
| `CALC-REF-13` | `FTC-ALLOC-01` | `CS-013` | R3 | `CONDITIONAL` mathematical sanity |
| `CALC-REF-14` | `FTC-MC-01` | `CS-014` | R3 | `REF-EXACT` aggregation only |
| `CALC-REF-15` | `FTC-NW-03` | `CS-004` | R1/R2 | `DOMAIN-INVARIANT` |
| `CALC-REF-16` | `FTC-GOAL-05` | `CS-007` | R1 | `CONDITIONAL` status semantics |

---

## 4. Uang, currency, dan rate normalization (`CS-001`, `CS-002`)

| Case ID | Given / action | Then (expected) | Readiness / gate |
|---|---|---|---|
| `FTC-MONEY-01` | Money IDR Rp1.500.000 + IDR Rp250.000 | IDR Rp1.750.000, **exact**; tanpa loss of cent/rupiah. | `REF-EXACT`; `CALC-Q01` |
| `FTC-MONEY-02` | Money IDR Rp1.000.000 + USD 100 | `CURRENCY_MISMATCH`; **tidak** auto-convert USD ke IDR. | `DOMAIN-INVARIANT` |
| `FTC-MONEY-03` | Input actual Money `"Rp1.000.000"` sebagai field mesin numerik | Ditolak atau ditangani di presentation/parser yang terdokumentasi; bukan nilai numeric domain langsung. | `DOMAIN-INVARIANT`; API contract TBD |
| `FTC-MONEY-04` | Input `NaN`, `Infinity`, overflow, atau numeric yang tidak dapat direpresentasikan | Error yang terkendali (`INVALID_AMOUNT` / `NUMERIC_OVERFLOW`), bukan hasil sukses. | `DOMAIN-INVARIANT`; limit TBD |
| `FTC-RATE-01` | Annual effective `r_y=0.12`, `k=12` periode identik | Rate efektif bulanan = `(1.12)^(1/12)-1` ≈ `0.00948879293458297412635506919349`. | `REF-APPROX`; `CALC-Q05/Q15` |
| `FTC-RATE-02` | Annual nominal `j_y=0.12` compounded 12 kali | Rate bulanan `0.01` tepat; annual effective `(1.01)^12-1 = 0.126825030131969720661201`. | `REF-EXACT` untuk formula desimal |
| `FTC-RATE-03` | Input `12% per tahun` **tanpa** menyebut effective/nominal | `RATE_KIND_AMBIGUOUS` / validation error; dilarang menganggap 1%/bulan. | `DOMAIN-INVARIANT` |
| `FTC-RATE-04` | `r_y=0` annual effective dengan `k=12` | Rate bulanan `0`. | `REF-EXACT` |
| `FTC-RATE-05` | `r_y<-1` pada model annual effective long-only sederhana | `INVALID_RATE`; tidak menghitung akar dari gross return negatif. | `CONDITIONAL`; model domain |
| `FTC-RATE-06` | Annual effective `r_y=-1` | Model `-100%` harus diperlakukan khusus; jangan memanggil `log1p(-1)` pada jalur umum. | `BLOCKED`; `CALC-Q05` |

**Assertion tambahan:** aplikasi harus membedakan field `annual_effective_rate`, `annual_nominal_rate` dan `effective_rate_per_period` secara semantik. Nilai annual 12% **tidak** berarti monthly 1% kecuali input secara eksplisit menyatakan nominal compounded monthly.

---

## 5. Opening balance, transaksi, transfer, dan posting (`CS-003`)

### 5.1 Reference angka dan invariants

| Case ID | Given → action | Then (expected) | Kesiapan |
|---|---|---|---|
| `FTC-ACT-01` | Opening A Rp10.000.000 lalu posted income Rp2.000.000 pada periode yang eligible | Closing A **Rp12.000.000**; income periode **Rp2.000.000**, bukan Rp12.000.000. | `CONDITIONAL`; `CALC-REF-01`, cutover TBD `CALC-Q09` |
| `FTC-ACT-02` | A Rp5.000.000; B Rp2.000.000; internal transfer Rp1.000.000; fee 0 | A **Rp4.000.000**; B **Rp3.000.000**; total **Rp7.000.000**; income=expense=0; NW delta=0. | `DOMAIN-INVARIANT`; `CALC-REF-02` |
| `FTC-ACT-03` | Kondisi `FTC-ACT-02`, tetapi fee transfer Rp10.000 dibayar A | A **Rp3.990.000**, B **Rp3.000.000**; total **Rp6.990.000**; fee expense **Rp10.000**, NW turun **Rp10.000**. | `CONDITIONAL`; model fee eksplisit |
| `FTC-ACT-04` | Income Rp800.000 dan expense Rp300.000, tanpa transfer, saldo awal Rp2.000.000 | Closing **Rp2.500.000**; net external flow Rp500.000. | `CONDITIONAL`; posted & cutover eligible |
| `FTC-ACT-05` | Transfer A→B Rp100.000 lalu balik B→A Rp100.000, no fees | Saldo masing-masing kembali ke state awal; income=expense=0. | `DOMAIN-INVARIANT` |
| `FTC-ACT-06` | Planned/recurring expense Rp200.000 belum posted | Saldo actual **tidak berubah**; pengeluaran aktual tidak bertambah. | `DOMAIN-INVARIANT` (`DR-013`) |
| `FTC-ACT-07` | Draft transaction gagal commit (rollback) | Tidak ada saldo/aggregate actual yang berubah. | `INTEGRATION-SPEC` |
| `FTC-ACT-08` | Transfer dua sisi; debit berhasil di memori tetapi insert sisi kredit gagal dalam SQL transaction | Seluruh SQL transaction rollback; A dan B tetap sebelum transfer. | `INTEGRATION-SPEC` (`DR-007`) |
| `FTC-ACT-09` | Submit operasi yang sama dua kali dengan **idempotency identity yang sama** | Hanya **satu** posting efektif; respons retry konsisten. | `INTEGRATION-SPEC` (`DR-008`), key lifecycle TBD |
| `FTC-ACT-10` | Transaksi `occurred_on` dan `recorded_at` berbeda | Laporan periode menggunakan field waktu yang disetujui; tidak otomatis memakai waktu insert. | `BLOCKED` policy tanggal `CALC-Q03/Q09` |
| `FTC-ACT-11` | Transfer ke account penerima milik user lain yang tidak diberi izin | Ditolak sebelum mutasi; tidak ada disclosure saldo atau ownership pihak lain. | `INTEGRATION-SPEC` (`DR-003`) |
| `FTC-ACT-12` | Reverse/void transaksi posted dengan audit | Saldo agregat kembali konsisten dengan efek reversal yang disetujui; tidak ada posting hilang tanpa jejak. | `BLOCKED` lifecycle `FDR-Q02` |

**Catatan:** contoh transfer di sini menguji **makna ekonomi** dan invariants, bukan memaksakan satu jenis tabel ledger. Untuk `FTC-ACT-09`, definisi key/request identity harus ditetapkan di API dan database; jangan menganggap setiap POST yang nominalnya sama sebagai duplikat bisnis.

### 5.2 Assertion tingkat storage/API

- Setelah commit sukses, saldo hasil rekonstruksi tidak berbeda dari total posting eligible.
- Transaksi dengan status selain posted tidak memengaruhi saldo actual; state final TBD.
- Simultaneous request tidak mengakibatkan debit ganda, lost update, atau saldo berbeda dengan ledger.
- Duplicate retry dengan idempotency identity sama tidak menambah transaksi; error yang terjadi sebelum commit tidak dipublikasikan sebagai sukses.
- Data pengguna B tidak muncul di aggregate, export, search, maupun error response user A.

---

## 6. Net worth, aset, utang, dan kewajiban (`CS-004`)

| Case ID | Given / action | Then (expected) | Kesiapan |
|---|---|---|---|
| `FTC-NW-01` | Account Rp6.000.000 + **aset lain yang tidak duplikat** Rp5.000.000 − outstanding Rp3.000.000 | **Net worth Rp8.000.000**. | `REF-EXACT`; `CALC-REF-03` |
| `FTC-NW-02` | Total aset Rp2.000.000, outstanding Rp3.000.000 | **Net worth -Rp1.000.000**, jangan dipaksa ke nol. | `REF-EXACT` |
| `FTC-NW-03` | Meminjam Rp10.000.000: kas +Rp10.000.000, utang +Rp10.000.000, tanpa fee | **NW delta 0**, bukan income Rp10.000.000. | `DOMAIN-INVARIANT`; `CALC-REF-15` |
| `FTC-NW-04` | Dari state `FTC-NW-03`, bayar pokok Rp1.000.000 dari kas | Kas **Rp9.000.000**, outstanding **Rp9.000.000**, **NW tetap 0**. | `REF-EXACT` |
| `FTC-NW-05` | Dari state `FTC-NW-04`, bayar bunga Rp100.000 dari kas | Kas **Rp8.900.000**, utang **Rp9.000.000**, NW **-Rp100.000**. | `REF-EXACT`; bunga expense, principal bukan expense |
| `FTC-NW-06` | Satu rekening Rp4.000.000 sekaligus dicatat sebagai baris `other_asset` Rp4.000.000 dengan referensi akun sama | Angka **hanya dihitung sekali**; total eligible = Rp4.000.000, bukan Rp8.000.000, atau duplikasi ditolak. | `DOMAIN-INVARIANT`; DB link TBD |
| `FTC-NW-07` | Pembelian aset nonkas Rp3.000.000 dengan kas Rp3.000.000; recorded asset Rp3.000.000, fee=0 | Kas turun Rp3.000.000, aset naik Rp3.000.000; **NW delta 0**. | `CONDITIONAL` matching valuation/event |
| `FTC-NW-08` | Nilai aset terakhir ditinjau bulan lalu, diminta dashboard sekarang | Nilai tetap jelas `valuation_as_of`; jangan mengklaim harga real time. | `DOMAIN-INVARIANT`; freshness policy TBD |
| `FTC-NW-09` | Liability tanpa saldo pokok outstanding valid, hanya total cicilan historis | `INSUFFICIENT_DATA` untuk NW atau status incomplete yang terjelaskan; jangan menganggap liability=0. | `CONDITIONAL` data quality |

**NW invariant:** jumlah perubahan kas dari transfer/pelunasan pokok **tidak sama** dengan perubahan NW; *positions* dan *flows* tidak boleh disatukan (`DR-023`). Semua komponen dipotong pada `as_of` konsisten; jika tidak, keluaran diberi incomplete/stale sesuai kebijakan.

---
## 7. Compound interest dan future value (`CS-005`, FR-008)

**Model referensi wajib disebut lengkap:** `P` opening principal; `C` setoran konstan; `n` periode; `i` return efektif **per periode**; fee/tax/withdrawal=0; setoran `end` atau `begin` eksplisit.

```text
FV_end(P,C,i,n) = P*(1+i)^n + C*(((1+i)^n - 1)/i), jika i != 0
FV_end(P,C,0,n) = P + n*C
FV_begin(P,C,i,n) = P*(1+i)^n + C*(((1+i)^n - 1)/i)*(1+i)
FV_begin(P,C,0,n) = P + n*C
```

| Case ID | Input (`REF-PROFILE-A` kecuali disebut) | Expected **unrounded** | Kesiapan |
|---|---|---|---|
| `FTC-FV-01` | `P=1000000,C=100000,n=12,i=0,timing=end` | `FV=2200000`; contributed = `2200000`; gain=`0`. | `REF-EXACT`; `CALC-REF-04` |
| `FTC-FV-02` | `P=0,C=100000,n=2,i=.01,timing=end` | `FV=201000` (`100000*1.01+100000`). | `REF-EXACT`; `CALC-REF-05` |
| `FTC-FV-03` | `P=0,C=100000,n=2,i=.01,timing=begin` | `FV=203010` (`100000*1.01²+100000*1.01`). | `REF-EXACT`; `CALC-REF-06` |
| `FTC-FV-04` | `P=1000000,C=0,n=2,i=.01,timing=end` | `FV=1020100`. | `REF-EXACT`; `CALC-REF-07` |
| `FTC-FV-05` | `P=1000000,C=100000,n=0,i=.01,timing=end` | `FV=1000000`; `total_contribution=1000000` (belum ada setoran). | `REF-EXACT` |
| `FTC-FV-06` | `P=0,C=0,n=24,i=.02,timing=end` | `FV=0`. | `REF-EXACT` |
| `FTC-FV-07` | `P=1000000,C=100000,n=2,i=-.10,timing=end` | `FV=1000000`; contributed=`1200000`; gain=`-200000`. | `REF-EXACT`, negative return allowed in this explicit toy model |
| `FTC-FV-08` | `P=1000000,C=100000,n=12,i=0,timing=begin` | `FV=2200000` = end timing pada rate nol. | `REF-EXACT` |
| `FTC-FV-09` | `P=0,C=100000,n=2,i=.01,timing unspecified` | `UNAPPROVED_CONVENTION` atau kebutuhan input timing eksplisit; jangan memilih salah satu nilai. | `CONDITIONAL`; `CALC-Q04` |
| `FTC-FV-10` | `P=1000000,C=100000,n=-1,i=.01` | `INVALID_HORIZON`. | `DOMAIN-INVARIANT` |
| `FTC-FV-11` | `P=1000000,C=100000,n=2.5,i=.01` | `INVALID_HORIZON` untuk formula periode diskret penuh. | `DOMAIN-INVARIANT`; partial period TBD |
| `FTC-FV-12` | `P=1000000,C=100000,n=2,i=-1.2` | `INVALID_RATE` atau `MODEL_NOT_APPROVED`, tidak menghasilkan angka. | `CONDITIONAL`; valid domain policy |
| `FTC-FV-13` | `P=1000000,C=100000,n=12,i=.000000001,end` | `FV = 2200000.0186000000880000002695...` tanpa pembulatan, dihitung via recurrence high precision. | `REF-APPROX` untuk implementasi floating; `CALC-Q15` |
| `FTC-FV-14` | `P=1000000,C=100000,n=2,i=0, end` dan `begin` | Kedua metode memberi `FV=1200000`. | `REF-EXACT` |

**Jangan salah:** pada `FTC-FV-03`, kontribusi yang dilakukan di awal periode memperoleh return **dua kali** untuk setoran pertama, sekali untuk setoran kedua. Membandingkannya dengan `FTC-FV-02` tanpa menampilkan timing adalah bug. `i=-0.10` pada `FTC-FV-07` adalah **-10% per periode**, bukan -10% per tahun. Semua contoh adalah kalkulasi ilustratif, bukan proyeksi instrumen investasi tertentu.

### 7.1 Pemeriksaan implementasi recurrence vs rumus tertutup

Untuk setiap input valid pada domain `1+i>0`, hitung versi independent via loop:

```text
balance = P
repeat t in 1..n:
    if timing == end:
        balance = balance*(1+i) + C
    else if timing == begin:
        balance = (balance+C)*(1+i)
```

Hasil recurrence harus konsisten dengan persamaan tertutup untuk `i!=0`, dengan kebijakan toleransi numerik disetujui. **Jangan** membuat kedua expected melalui helper production yang sama: itu *tautological test* dan tidak membuktikan kebenaran rumus.

---

## 8. Inflasi, harga masa depan, dan real return (`CS-006`)

| Case ID | Given / tindakan | Expected | Kesiapan |
|---|---|---|---|
| `FTC-INF-01` | Harga *hari ini* IDR 1.000.000, inflasi efektif tahunan 10%, `n=2 tahun` | Target nominal masa depan **Rp1.210.000**. | `REF-EXACT`; `CALC-REF-10` |
| `FTC-INF-02` | Nilai nominal masa depan Rp1.210.000, inflasi efektif tahunan 10%, 2 tahun | Nilai dalam daya beli tahun dasar **Rp1.000.000**. | `REF-EXACT` |
| `FTC-INF-03` | Return nominal efektif 8%, inflasi efektif 3%, periode sama | Real return = `(1.08/1.03)-1` ≈ `0.0485436893203883495` atau sekitar `4.8543689%`; **bukan** `5%` eksak. | `REF-APPROX` |
| `FTC-INF-04` | Inflasi = 0; price now Rp1.000.000; `n=3` | Harga nominal masa depan Rp1.000.000. | `REF-EXACT` |
| `FTC-INF-05` | User sudah memberi *future nominal target* Rp1.210.000 untuk dua tahun mendatang | Sistem **tidak boleh** inflasi lagi menjadi Rp1.464.100 tanpa perubahan basis target yang eksplisit. | `DOMAIN-INVARIANT` |
| `FTC-INF-06` | Basis target `nominal`/`real` tidak dinyatakan, inflasi perlu diterapkan | `INSUFFICIENT_DATA` atau `UNAPPROVED_CONVENTION`, bukan aplikasi inflasi dua kali. | `CONDITIONAL`; `CALC-Q06` |
| `FTC-INF-07` | `pi=-1` pada rumus `V_real = V_nominal/(1+pi)^n`, `n>0` | `INVALID_RATE` untuk rumus normal; tidak boleh division by zero. | `DOMAIN-INVARIANT` |
| `FTC-INF-08` | Return 8% efektif bulanan tetapi inflasi 3% efektif tahunan | Normalisasi periode dahulu; jangan memakai `(1.08/1.03)-1` langsung. | `DOMAIN-INVARIANT` |

**Policy gate:** asumsi inflasi default, model inflasi berbeda untuk biaya pendidikan/rumah, dan validasi kalender masih `CALC-Q03/Q06`; test ini **tidak mengizinkan default inflasi otomatis**.

---

## 9. Goal contribution, horizon, feasibility, dan funded status (`CS-007`)

**Referensi matematis untuk target nominal akhir periode `T`** dengan saldo yang benar-benar *eligible/attributed* `P_g`, return tetap `i`, timing setoran akhir, `n>0`:

```text
A(i,n) = ((1+i)^n-1)/i  ;  A(0,n)=n
required_C_unrounded = max(0, (T - P_g*(1+i)^n)/A(i,n))
```

Rumus ini **tidak menyatakan** seluruh saldo account boleh dipakai untuk goal. `P_g` harus memenuhi model atribusi goal yang belum disetujui (`FDR-Q07`, `CALC-Q08`).

| Case ID | Given | Expected | Readiness/gate |
|---|---|---|---|
| `FTC-GOAL-01` | `T=210000, P_g=0, n=2, i=.10/period, timing=end` | `A=2.1`; kontribusi diperlukan **Rp100.000 per periode**. | `CONDITIONAL`; `CALC-REF-08` |
| `FTC-GOAL-02` | `T=1200000, P_g=0, n=12, i=0, timing=end` | Required **Rp100.000 per periode**. | `CONDITIONAL`; `CALC-REF-09` |
| `FTC-GOAL-03` | `T=1210000, P_g=1000000, n=2, i=.10/period` | `P_g*(1+i)^2=1210000`; required **0**. | `CONDITIONAL`; fixed return assumption |
| `FTC-GOAL-04` | `T=1000000, P_g=1500000, n=12, i=0` | Required **0**, bukan kontribusi negatif. | `CONDITIONAL` |
| `FTC-GOAL-05` | `T=500000, P_g=200000, n=0` | `GOAL_DEADLINE_REACHED` atau status ekuivalen; shortfall **Rp300.000**; required recurring contribution **N/A**, bukan `0`. | `CONDITIONAL`; `CALC-REF-16` |
| `FTC-GOAL-06` | `T=500000, P_g=500000, n=0` | `goal_met_now=true`, shortfall `0`, required recurring contribution **N/A**. | `CONDITIONAL` |
| `FTC-GOAL-07` | `required_C=100000`, explicitly available capacity=`80000`/period | Gap **Rp20.000**/period; `mathematical_feasible=false` *menurut asumsi kapasitas yang diberikan*. | `CONDITIONAL`; `CALC-Q11` |
| `FTC-GOAL-08` | Data kapasitas tabungan tidak tersedia | Status `INSUFFICIENT_DATA`; jangan tandai feasible `true` atau `false` dengan angka buatan. | `DOMAIN-INVARIANT` |
| `FTC-GOAL-09` | Planned contribution Rp1.000.000; real account belum berubah | Funded actual **tetap** sesuai bukti pendanaan eligible sebelumnya; tidak bertambah karena planned. | `DOMAIN-INVARIANT`; `DR-011` |
| `FTC-GOAL-10` | Satu account berisi Rp1.000.000, dua goals menandai masing-masing Rp1.000.000 sebagai *exclusive funded* | Model atribusi harus mencegah total exclusive funded Rp2.000.000 atas uang Rp1.000.000, atau hasil dinyatakan invalid. | `DOMAIN-INVARIANT`; model mekanisme `BLOCKED CALC-Q08` |
| `FTC-GOAL-11` | `n=0`, `P_g<T` lalu engine mencoba pembagian anuitas nol | Error/status deadline tercapai, **tidak** `NaN/Inf` atau angka kontribusi bulanan 0. | `DOMAIN-INVARIANT` |
| `FTC-GOAL-12` | `T=210000,P_g=0,n=2,i=.10`, tetapi timing=`begin` eksplisit | `A_begin=2.31`; required unrounded = `210000/2.31 ≈ 90909.090909...`. | `REF-APPROX`; `CALC-Q02/Q04` |

`FTC-GOAL-12` **bukan** angka rupiah bulat yang boleh disimpan langsung sebagai cicilan final: pembulatan minimum ke satuan uang serta efek penjumlahan cicilan perlu diputuskan. Rate 10% **per periode** dipilih agar struktur persamaan mudah diuji, bukan asumsi instrumen investasi.

---

## 10. Budgets dan recurring (`CS-008`, R2)

| Case ID | Given | Expected | Readiness |
|---|---|---|---|
| `FTC-BUD-01` | Budget kategori makan Rp2.000.000; eligible posted expense Rp2.200.000 pada periode sama | Remaining budget **-Rp200.000** (over-budget). | `CONDITIONAL`; period/category policy TBD |
| `FTC-BUD-02` | Budget Rp2.000.000; expense kategori makan Rp1.000.000; transfer antar rekening Rp500.000 tanpa fee | Remaining budget makan **Rp1.000.000**, bukan Rp500.000. | `DOMAIN-INVARIANT` |
| `FTC-BUD-03` | Budget Rp2.000.000; posted expenses Rp2.200.000; transfer fee kategori budget Rp10.000 diakui sebagai expense pada periode yang sama | Remaining **-Rp210.000** menurut klasifikasi eksplisit tersebut. | `CONDITIONAL` |
| `FTC-BUD-04` | Recurring draft pengeluaran Rp500.000 untuk bulan depan, belum posted | Forecast boleh memuat planned event; expense actual bulan ini **0** dari event tersebut. | `DOMAIN-INVARIANT` |
| `FTC-BUD-05` | Scheduler retry memproses recurring event yang sama | Tidak ada duplicate actual posting untuk occurrence yang diidentifikasi sama. | `INTEGRATION-SPEC`; generation/posting policy TBD |
| `FTC-BUD-06` | Refund dicatat di periode berbeda tanpa policy refund yang disetujui | Jangan mengoreksi budget historis diam-diam; perlu definisi reversal/refund. | `BLOCKED` (`FDR-Q02/Q03`) |

---

## 11. Deterministic projection dan cash-flow (`CS-009`, R2)

**Projection selalu hypothetical** dari baseline `as_of`. Contoh berikut mengasumsikan transfer ke investasi terjadi di akhir periode **setelah** return periode tersebut, sehingga transfer tidak memperoleh return instan. Ini **bukan** event ordering yang telah disetujui (`CALC-Q10`).

| Case ID | Given | Expected | Readiness |
|---|---|---|---|
| `FTC-CASH-01` | Cash awal Rp4m; income eksternal Rp6m; konsumsi Rp3m; bunga dibayar Rp0,1m; pokok utang dibayar Rp0,9m; transfer dari cash ke investasi Rp1m | Cash akhir **Rp5m**; expense konsumsi+biaya **Rp3,1m**; repayment principal Rp0,9m bukan expense; investasi bertambah Rp1m; liability turun Rp0,9m. | `CONDITIONAL`; `CALC-REF-11` |
| `FTC-CASH-02` | External income Rp6m, konsumsi Rp3m, bunga Rp0,1m pada `FTC-CASH-01` | `external_net_flow = 6m - 3,1m = Rp2,9m`; `cash_account_delta=5m-4m=Rp1m`; **dua metrik ini berbeda**. | `CONDITIONAL`; classification explicated |
| `FTC-CASH-03` | Cash Rp2m; transfer internal cash A→cash B Rp500k, tanpa fee | Total cash Rp2m; total NW tidak berubah; external expense=0. | `DOMAIN-INVARIANT` |
| `FTC-CASH-04` | Projection tanpa new borrowing, awal cash Rp1m, consumption Rp2m, income0 | Akhir forecast cash **-Rp1m** sebagai sinyal shortfall; **tidak** auto-pinjam / auto-clamp nol. | `CONDITIONAL`; `CALC-Q11` |
| `FTC-CASH-05` | Goal contribution planned Rp500k tanpa perpindahan asset sesungguhnya | Total assets forecast tidak bertambah Rp500k hanya karena label alokasi. | `DOMAIN-INVARIANT` |
| `FTC-CASH-06` | Simulated income event Rp2m satu kali; forecast dihitung dua kali dari baseline sama | Setiap fresh calculation menghasilkan total income yang sama, bukan Rp4m karena *accumulated state mutation*. | `DOMAIN-INVARIANT` |
| `FTC-CASH-07` | Membeli aset hipotetis seharga Rp3m dari cash, aset tercatat Rp3m, fee0 | Cash turun Rp3m, other asset naik Rp3m; NW delta0 pada transaksi itu. | `CONDITIONAL`; valuation model |
| `FTC-CASH-08` | Skenario future cash tidak punya data cicilan wajib, tetapi UI ingin menampilkan safe-to-spend amount | Status incomplete/`INSUFFICIENT_DATA`; jangan menganggap cicilan=0. | `DOMAIN-INVARIANT` |
| `FTC-CASH-09` | Event terakhir terjadi bulan ke-4; scenario diproyeksikan 3 bulan | Event bulan 4 **tidak** boleh masuk output 3 bulan. | `CONDITIONAL`; calendar policy TBD |

**Pembuktian `FTC-CASH-01`:** `4m+6m−3m−0,1m−0,9m−1m=5m`. Ini menunjukkan cash balance, net external operating cash flow, debt principal repayment, dan perubahan investasi tidak boleh dipadatkan menjadi satu angka expense.

---

## 12. Fixed-rate loan amortization (`CS-010`, R2/R3)

**Scope contoh:** pinjaman anuitas dengan tingkat bunga efektif tetap per periode, pembayaran di akhir periode, tanpa fee/pajak/penalty/grace period, dan tanpa rounding intermediate. **Bukan** simulasi ketentuan bank/KPR nyata; kebijakan produk `CALC-Q07` belum approved.

| Case ID | Given | Expected unrounded | Kesiapan |
|---|---|---|---|
| `FTC-LOAN-01` | `L=210000,i=.10 per period,n=2` | `PMT=121000`; periode 1 interest=21000, principal=100000, remaining=110000; periode 2 interest=11000, principal=110000, remaining=0. | `CONDITIONAL`; `CALC-REF-12` |
| `FTC-LOAN-02` | `L=120000,i=0,n=3` | `PMT=40000`; interest=0 setiap periode; remaining=80000, 40000, 0. | `REF-EXACT` pada model fixed schedule |
| `FTC-LOAN-03` | `L=210000,i=.10,n=2` | Total paid=`242000`; total interest=`32000`; total principal=`210000`. | `REF-EXACT`; fixed model |
| `FTC-LOAN-04` | Borrow Rp10m menambah cash dan liability sama besar, fee0 | NW delta 0. | `DOMAIN-INVARIANT`; `FTC-NW-03` |
| `FTC-LOAN-05` | Pembayaran Rp1.100.000 = pokok Rp1.000.000 + bunga Rp100.000 | Cash turun Rp1.100.000; outstanding turun Rp1.000.000; NW delta **-Rp100.000**. | `DOMAIN-INVARIANT` |
| `FTC-LOAN-06` | `L=210000,n=0,i=.10` | `INVALID_HORIZON`/invalid loan; tidak division by zero. | `DOMAIN-INVARIANT` |
| `FTC-LOAN-07` | `L=-210000,n=2,i=.10` | `INVALID_AMOUNT`. | `DOMAIN-INVARIANT` |
| `FTC-LOAN-08` | Fixed rate berubah ke floating setelah periode 1; reset policy tidak didefinisikan | `MODEL_NOT_APPROVED`; tidak melanjutkan cicilan fixed seolah KPR nyata. | `BLOCKED` |
| `FTC-LOAN-09` | Jadwal menghasilkan residual pecahan setelah pembulatan tiap bulan | Expected final payment **belum ditentukan**, harus uji kebijakan rounding yang disetujui. | `BLOCKED CALC-Q02` |
| `FTC-LOAN-10` | Masukan nominal *flat* rate ke fungsi anuitas efektif tanpa konversi | `RATE_KIND_AMBIGUOUS` atau rate-kind mismatch; jangan kalkulasi dengan interpretasi keliru. | `DOMAIN-INVARIANT` |

**Rekonstruksi jadwal untuk `FTC-LOAN-01`:**

| Periode | Opening debt | Interest | Principal | Payment | Closing debt |
|---|---:|---:|---:|---:|---:|
| 1 | 210.000 | 21.000 | 100.000 | 121.000 | 110.000 |
| 2 | 110.000 | 11.000 | 110.000 | 121.000 | 0 |

Model amortisasi lain tidak boleh diuji menggunakan tabel ini tanpa menyamakan jenis bunga dan konvensi pembayaran.

---
## 13. Financial stress test dan runway (`CS-011`, FR-015)

Untuk contoh di bawah, **pilih secara eksplisit** threshold cash minimum `K_min=0`, perhitungan di akhir bulan, tanpa aset likuid tambahan atau utang otomatis. **Pilihan ini ilustratif, tidak menetapkan default Esokira** (`CALC-Q11`).

| Case ID | Given | Then | Kesiapan |
|---|---|---|---|
| `FTC-STRESS-01` | Cash awal Rp4m; 5 bulan tanpa income; pengeluaran wajib Rp1m setiap akhir bulan; threshold 0 | Closing cash per bulan: **3m, 2m, 1m, 0, -1m**. First shortfall **bulan ke-5**; fully-funded periods sebelum shortfall = **4**. | `CONDITIONAL` threshold/timing |
| `FTC-STRESS-02` | Baseline alternatif cash awal Rp4m; income Rp1m tiap bulan, biaya Rp1m tiap bulan selama 5 bulan | Cash akhir setiap bulan **Rp4m**; tidak ada shortfall. | `CONDITIONAL` |
| `FTC-STRESS-03` | Bandingkan stress `FTC-STRESS-01` terhadap baseline `FTC-STRESS-02`, dari baseline awal sama | Delta closing cash pada akhir bulan ke-5 = `-1m - 4m = -Rp5m`. | `REF-EXACT` untuk premis tersebut |
| `FTC-STRESS-04` | Kondisi `FTC-STRESS-01` tetapi model mendeteksi shortfall menggunakan saldo **awal bulan** | Waktu shortfall bisa berbeda; harus diseragamkan sebelum membandingkan hasil. | `BLOCKED` jika tidak ada timing policy |
| `FTC-STRESS-05` | Pengeluaran wajib lain (misalnya cicilan) tidak diketahui | `INSUFFICIENT_DATA` untuk label “aman X bulan”; boleh memberi perhitungan parsial yang jujur. | `DOMAIN-INVARIANT` |
| `FTC-STRESS-06` | Stress run dibuat berdasarkan baseline X, kemudian pengguna mengubah actual cash | Hasil run stress X lama tetap sama; bila relevan diberi stale / ajakan rebase, tidak diperbarui diam-diam. | `INTEGRATION-SPEC`; `CALC-Q10` |
| `FTC-STRESS-07` | Stress menghasilkan cash negatif pada akhir bulan 5 | UI menampilkan deficit menurut model; **tidak** menambahkan pinjaman implisit atau menyatakan pengguna pasti bangkrut. | `DOMAIN-INVARIANT` |

**Kenapa periode ke-5, bukan ke-4?** Pada akhir periode 4 saldo adalah tepat nol, masih memenuhi threshold `>=0` dalam definisi contoh. Pada akhir periode 5 baru menjadi negatif. Bila produk memilih minimum buffer `K_min>0`, hitung ulang; jangan menyalin hasil “4 bulan” ke metrik lain tanpa definisi.

---

## 14. Scenario versioning, comparison, dan reproducibility (`CS-012`, FR-013)

| Case ID | Given / when | Then | Kesiapan |
|---|---|---|---|
| `FTC-SCEN-01` | Snapshot baseline `B1`; parameter `S1`; engine version `E1`; jalankan deterministik dua kali tanpa perubahan | Keluaran numerik dan assumptions fingerprint **identik** sesuai representasi/precision yang dipilih. | `DOMAIN-INVARIANT`; algoritma deterministik |
| `FTC-SCEN-02` | Baseline saldo A Rp2m; buat life event hipotetis “biaya Rp500k” dan hitung skenario | Actual A **tetap Rp2m**; hanya forecast skenario yang boleh menampilkan efek event. | `DOMAIN-INVARIANT` (`DR-002`) |
| `FTC-SCEN-03` | Skenario versi 1 telah disimulasikan; versi 2 mengubah asumsi gaji | Version 1 tidak ditimpa diam-diam; perbandingan menyebut version ID masing-masing. | `INTEGRATION-SPEC`; `CALC-Q10` |
| `FTC-SCEN-04` | Actual berubah sejak snapshot `B1`; run historis memakai `B1` | Run lama tidak otomatis berubah; status `STALE_BASELINE`/stale indicator atau tindakan rebase eksplisit, sesuai kebijakan final. | `CONDITIONAL`; `CALC-Q10` |
| `FTC-SCEN-05` | Bandingkan skenario IDR dengan nominal future value dan skenario USD tanpa FX yang disetujui | Perbandingan numerik ditolak sebagai tidak kompatibel; bukan ranking “lebih besar”. | `DOMAIN-INVARIANT` |
| `FTC-SCEN-06` | Skenario A membandingkan nilai real tahun dasar 2026 dengan skenario B nilai nominal 2030 | Harus normalisasi basis dan horizon secara eksplisit, atau beri status incompatible. | `DOMAIN-INVARIANT` |
| `FTC-SCEN-07` | B1+S1+E1 diperbarui menjadi B2+S1+E1 setelah actual posting | Ini adalah **input baru**; jangan klaim wajib menghasilkan output sama seperti B1. | `DOMAIN-INVARIANT` |
| `FTC-SCEN-08` | API simulation memperoleh valid `scenario_id` milik user lain | `UNAUTHORIZED_RESOURCE` atau respons aman setara; tidak boleh membuka input, asumsi, ataupun hasil private. | `INTEGRATION-SPEC` (`DR-003/021`) |

Agar reproducibility berguna, simpan/rujuk `baseline_snapshot` atau fingerprint yang stabil, `scenario_version`, normalisasi parameter, `model_version`, `engine_version`, dan, untuk stochastic model, konfigurasi RNG dan seed. Konten snapshot sensitif harus tetap terlindungi aksesnya.

---

## 15. Multi-goal allocation sanity checks (`CS-013`, FR-017, R3)

**BLOCKED untuk optimizer production.** Referensi di bawah memeriksa **kendala matematika sederhana**, bukan menyetujui objective function, tie-break, prioritas, liquidity buffer, atau funded-goal attribution (`CALC-Q08`, `CALC-Q11`, `CALC-Q12`).

### 15.1 Batas kapasitas minimum dan constructive witness

Tiga goal tanpa opening funds, tanpa return, tanpa fee, dengan kontribusi tetap per bulan dan target pada akhir bulan masing-masing:

| Goal | Target IDR | Deadline |
|---|---:|---:|
| A | 12.000.000 | 12 |
| B | 24.000.000 | 24 |
| C | 120.000.000 | 60 |

Dari syarat dana cumulatif yang diperlukan pada deadline:

```text
S_min >= max(
  12_000_000 / 12,
  (12_000_000+24_000_000) / 24,
  (12_000_000+24_000_000+120_000_000) / 60
)
S_min >= max(1_000_000, 1_500_000, 2_600_000)
S_min >= 2_600_000 IDR/bulan
```

**Saksi kelayakan (*feasibility witness*) untuk `S=2.600.000/bulan`:**

| Bulan | Alokasi bulanan A | B | C | Total/bulan |
|---|---:|---:|---:|---:|
| 1–12 | 1.000.000 | 1.000.000 | 600.000 | 2.600.000 |
| 13–24 | 0 | 1.000.000 | 1.600.000 | 2.600.000 |
| 25–60 | 0 | 0 | 2.600.000 | 2.600.000 |

Pada deadline: A=`12m`, B=`12m+12m=24m`, C=`7.2m+19.2m+93.6m=120m`. Tidak ada periode yang mengalokasikan lebih dari 2.6m dan tidak ada dana yang dihitung dua kali. Ini **membuktikan lower bound tadi dapat dicapai untuk contoh khusus tersebut**; tidak berarti strategi di atas optimal di model finansial dunia nyata.

### 15.2 Kasus

| Case ID | Given / action | Expected | Readiness |
|---|---|---|---|
| `FTC-ALLOC-01` | Tiga goals A/B/C di atas, saldo awal0, return0, capacity tetap | Minimum constant monthly capacity **Rp2.600.000**; witness schedule tersedia. | `CONDITIONAL`; `CALC-REF-13` |
| `FTC-ALLOC-02` | Goals sama, capacity Rp2.000.000/bulan | Semua goal **tidak feasible** dipenuhi tepat deadline. Pada bulan60 total kapasitas Rp120m < total target Rp156m. | `REF-EXACT` sanity only |
| `FTC-ALLOC-03` | Goals sama, capacity Rp2.600.000/bulan dengan witness di §15.1 | Alokasi setiap bulan `<= capacity`, tidak negatif, semua target dipenuhi pada deadline. | `REF-EXACT` constructive example |
| `FTC-ALLOC-04` | Any proposed allocation vector mengalokasikan Rp3m ketika capacity Rp2m | **Constraint violation**; jangan memberi status feasible. | `DOMAIN-INVARIANT` |
| `FTC-ALLOC-05` | Goal A dan B diberi prioritas sama, dua solusi feasible dengan objective belum dipilih | Ranking/tie-break **BLOCKED**; tidak boleh agent mengarang urutan prioritas. | `BLOCKED`; `CALC-Q12` |
| `FTC-ALLOC-06` | Goal tidak dapat terpenuhi tetapi solver menghasilkan alokasi parsial | Status `INFEASIBLE_PLAN`/hasil keterbatasan, daftar goal/shortfall; tidak boleh diam-diam menghilangkan goal. | `DOMAIN-INVARIANT` |
| `FTC-ALLOC-07` | Saldo cash aktual sama dinyatakan full-funded untuk dua goal eksklusif | Harus mengikuti aturan atribusi tunggal; tidak boleh digandakan. | `DOMAIN-INVARIANT`; `CALC-Q08` |
| `FTC-ALLOC-08` | Seluruh tujuan punya return/timing yang belum ditetapkan | Objective/solution **BLOCKED**, meski kendala `sum allocations <= capacity` tetap dapat diuji. | `BLOCKED`; `CALC-Q12` |

---

## 16. Monte Carlo risk simulation (`CS-014`, FR-018, R3)

**Model stochastic belum disetujui** (`CALC-Q13`). Oleh karena itu, yang siap diuji saat ini adalah kebenaran **agregasi deterministik atas hasil lintasan yang disuplai**, integritas jumlah lintasan, penanganan error, dan keamanan. **Jangan** menciptakan expected percentile atau return distribution sebagai “golden production” sebelum `MONTE-CARLO.md` disetujui.

| Case ID | Given / action | Expected | Readiness |
|---|---|---|---|
| `FTC-MC-01` | 1.000 path valid, 720 sukses, 280 gagal menurut definisi sukses yang disebut | `model_success_rate=720/1000 = 0.72` atau 72%; bukan 72% kepastian masa depan. | `REF-EXACT` agregasi; `CALC-REF-14` |
| `FTC-MC-02` | `n_requested=1000, n_valid=1000, n_failed=0` | Transparansi count: `n_valid+n_failed=1000`. | `REF-EXACT` untuk contoh lengkap |
| `FTC-MC-03` | `n_valid=0`, successful=0 | Jangan membagi nol atau menampilkan 0% sukses; `INSUFFICIENT_DATA`/`ENGINE_FAILURE` menurut taxonomy yang disetujui. | `DOMAIN-INVARIANT` |
| `FTC-MC-04` | `n_requested=1000,n_valid=800,n_failed=200,n_success=720` | Pecahan **di antara 800 valid** ialah `0.90`; kelayakan mempublikasikan angka dan perlakuan 200 failed **BLOCKED**. Jangan memakai 0.72 tanpa kebijakan denominator. | `CONDITIONAL / BLOCKED`; `CALC-Q13` |
| `FTC-MC-05` | Input sama, seed sama, tetapi model/RNG algorithm version berbeda | Tidak ada jaminan path-identical. Wajib menampilkan versi yang berbeda. | `DOMAIN-INVARIANT` |
| `FTC-MC-06` | Input, seed, RNG algorithm/version, path-index assignment, engine version identik | Reproducibility yang dijanjikan baru diuji setelah model final. | `BLOCKED` statistical implementation |
| `FTC-MC-07` | Matrik korelasi input asimetris dalam model yang mewajibkan simetris/PSD | Invalid parameter/model configuration; jangan silent-fix. | `BLOCKED` model validation specifics |
| `FTC-MC-08` | Daftar saldo akhir hasil run valid dan ingin dihitung P10/P50/P90 | **Metode kuantil dan basis nilai** harus disetujui terlebih dahulu; expected numeric percentile TBD. | `BLOCKED` |
| `FTC-MC-09` | Worker dibatalkan setelah sebagian lintasan | Status cancelled/failed; hasil parsial tidak dinyatakan run berhasil; actual tetap utuh. | `INTEGRATION-SPEC` |
| `FTC-MC-10` | Result card bertuliskan “720 dari 1.000 skenario berhasil” | Wajib ada keterangan **bergantung pada model/asumsi** serta horizon/success definition. | `DOMAIN-INVARIANT` (`DR-015`) |

**Jangan memakai Monte Carlo untuk menjamin hasil investasi.** Simulasi sesuai asumsi sampling tidak membuktikan probabilitas nyata. Review model risiko harus mencakup korelasi, nonnormalitas, risiko sequence-of-returns, sumber input, RNG reproducibility, performance budget, cancellation, dan error path.

---

## 17. Retirement / FIRE illustrative guards (`CS-015`, FR-019, R3)

Untuk kasus numerik, **hanya** gunakan portfolio likuid Rp300.000, return 0, biaya hidup bersih Rp100.000 per periode, other income 0, no fees/tax, penarikan **akhir** periode, dan periode penuh. Pilihan ini **bukan kebijakan pensiun yang disetujui** (`CALC-Q14`).

| Case ID | Given | Expected | Readiness |
|---|---|---|---|
| `FTC-RET-01` | Portfolio 300000, withdraw100000 pada akhir periode1 dengan rate0 | Akhir periode1 = **200000**. | `REF-EXACT` toy model |
| `FTC-RET-02` | Lanjut identik 3 periode | Akhir periode 1..3 = **200000, 100000, 0**. | `REF-EXACT` toy model |
| `FTC-RET-03` | Pada awal periode4 saldo=0, kebutuhan withdrawal100000, return0 | **Shortfall Rp100.000**; jangan mengklaim terpenuhi atau membuat pinjaman implisit. | `CONDITIONAL`; depletion policy TBD |
| `FTC-RET-04` | Net worth Rp1 miliar termasuk rumah tidak likuid Rp900 juta, liquid portfolio Rp100 juta | Jangan menghitung semua Rp1 miliar dapat langsung membayar belanja pensiun; metrik likuiditas terpisah. | `DOMAIN-INVARIANT` |
| `FTC-RET-05` | Pengguna memilih umur berhenti kerja 50 dan target financial independence age 45 | Dua tanggal/konsep terpisah; jangan memaksa keduanya identik. | `DOMAIN-INVARIANT` |
| `FTC-RET-06` | UI/insight menjanjikan “4% withdrawal selalu aman” | Test presentation **gagal**: output tidak boleh mengklaim aturan universal. | `DOMAIN-INVARIANT` |
| `FTC-RET-07` | Asumsi return/biaya/horizon masa pensiun tidak dinyatakan | `INSUFFICIENT_DATA`/`MODEL_NOT_APPROVED`; tidak boleh memublikasikan angka usia pensiun pasti. | `BLOCKED`; `CALC-Q14` |

---

## 18. Financial insights dan GPS (`CS-016`, FR-016/020)

| Case ID | Given | Expected | Readiness |
|---|---|---|---|
| `FTC-INS-01` | Contribution required100000, verified available capacity80000 dari same baseline/period | Insight gap **Rp20.000**, menyebut metric source, horizon, assumptions, dan opsi yang perlu diuji. | `CONDITIONAL`; `CALC-Q11` |
| `FTC-INS-02` | User meminta Financial GPS “undur goal satu tahun” | Hanya membuat **candidate scenario** bila dapat dimodelkan; tidak otomatis mengedit goal actual atau posting transfer. | `DOMAIN-INVARIANT` |
| `FTC-INS-03` | Engine belum dapat menetapkan kapasitas tabungan karena debt inputs kosong | Insight menjelaskan `INSUFFICIENT_DATA`; jangan mengatakan pengguna aman secara finansial. | `DOMAIN-INVARIANT` |
| `FTC-INS-04` | Rekomendasi dari Monte Carlo `0.72` tanpa model version/assumption | Tidak boleh mengklaim insight tervalidasi; minta data provenance yang diperlukan. | `DOMAIN-INVARIANT` |
| `FTC-INS-05` | Dua alternatif keputusan memiliki trade-off berbeda | Penjelasan mengacu ke source metrics masing-masing; ranking tanpa rule yang disetujui tidak boleh dikarang. | `CONDITIONAL / BLOCKED` ranking |

---
## 19. Error taxonomy, boundary tests, dan failure semantics

Error code di Calculation Specs adalah **konseptual**. Status HTTP dan nama final DTO ditetapkan kemudian di `API-GUIDELINES.md`/OpenAPI. Yang diuji sekarang adalah **perilaku aman**, bukan string error yang kebetulan dipilih agent.

| Case ID | Input / kejadian | Expected behavior | Referensi |
|---|---|---|---|
| `FTC-ERR-01` | Negative `n` pada fixed FV atau invalid date range | Reject `INVALID_HORIZON`; tidak return sukses. | `CS-005/007` |
| `FTC-ERR-02` | `n=0` pada required recurring goal contribution | Status deadline reached/met; tidak division by zero. | `CS-007` |
| `FTC-ERR-03` | Rate annual tanpa effective/nominal kind | `RATE_KIND_AMBIGUOUS`; tidak auto-convert. | `CS-002` |
| `FTC-ERR-04` | Currency mismatch pada penjumlahan Money | Reject; tidak silent-FX. | `CS-001` |
| `FTC-ERR-05` | Non-finite/overflow numerik | Numeric error yang terkendali, tidak NaN/Inf ke UI atau DB. | `CS-001` |
| `FTC-ERR-06` | Missing required fee/loan parameter pada calculation yang mengklaim “after-fees” | Incomplete/model unsupported; jangan diam-diam set biaya 0. | `CS-009/010` |
| `FTC-ERR-07` | Goal capacity data missing | `INSUFFICIENT_DATA`; tidak feasible=true otomatis. | `CS-007/016` |
| `FTC-ERR-08` | Optimizer objective tidak disetujui | `MODEL_NOT_APPROVED`/feature gated; tidak jalankan ranking sembarang. | `CS-013` |
| `FTC-ERR-09` | MC distribution/percentile method belum disahkan | `MODEL_NOT_APPROVED`; jangan menghasilkan kuantil yang tampak valid. | `CS-014` |
| `FTC-ERR-10` | User A meminta simulation run user B | Deny tanpa data leakage, audit aman; tidak perlu konfirmasi apakah ID ada. | `FR-001`, `DR-003` |
| `FTC-ERR-11` | Worker timeout saat Monte Carlo, sebelum final persistence | `ENGINE_FAILURE`/failed; tidak mark complete; actual unchanged. | `EN-005`, `DR-002` |
| `FTC-ERR-12` | Simulasi memakai baseline usang setelah actual berubah | Tampilkan stale/rebase sesuai policy; tidak silently overwrite baseline. | `CS-012`, `CALC-Q10` |
| `FTC-ERR-13` | Error menampilkan payload account / token / raw financial values milik user lain | Test security **gagal**; response/log harus disanitasi. | `DR-021`, `NFR-007` |
| `FTC-ERR-14` | Request menghitung `P+sum(C)` lalu mengklaim itu actual cash saldo | Test classification **gagal**; output harus `FORECAST`/`PLANNED`. | `DR-001/011/014` |
| `FTC-ERR-15` | Perhitungan manual menggunakan horizon 365/30 untuk `n` bulanan tanpa policy | Test time normalization gagal; calendar policy dibutuhkan. | `CS-002/009`, `CALC-Q03` |

**Hal-hal yang bukan error otomatis:** net worth negatif, anggaran overspent, return investasi negatif dalam domain yang disetujui, ataupun cash forecast negatif sebagai sinyal shortfall. Klasifikasikan hasil secara benar daripada membuang data yang tidak menyenangkan.

---

## 20. Property-based dan metamorphic tests

Golden vectors menguji angka tertentu. Property tests memeriksa invariants untuk banyak input valid agar kesalahan tidak lolos hanya karena cocok di satu contoh. Semua generator harus dibatasi pada domain kasus; jangan menghasilkan NaN/overflow dan memaksa hasil numerik lalu menyebutnya bug.

| Property ID | Klaim yang diuji | Domain dan pengecualian |
|---|---|---|
| `PROP-01` | Transfer fee0 A→B menjaga `A+B` dan NW. | Mata uang sama, ownership sah, kedua account tidak diduplikasi. |
| `PROP-02` | Transfer A→B→A nominal sama tanpa fee mengembalikan saldo awal. | Tidak ada transaksi lain, return, atau pending state. |
| `PROP-03` | FV recurrence=end dan closed form menghasilkan nilai setara. | `n>=0`, `P,C>=0`, `i>-1`, no fee, timing end; toleransi numerik disetujui. |
| `PROP-04` | Pada `i=0`, FV begin = FV end = `P+n*C`. | `n>=0`, kontribusi sama di kedua timing. |
| `PROP-05` | Untuk `i>=0` dan `C>=0`, FV begin `>=` FV end. | Return constant, no withdrawal/fee, n integer. |
| `PROP-06` | Untuk `-1<i<0`, `C>=0`, FV begin `<=` FV end. | Same simple fixed-return model; bukan semua produk investasi. |
| `PROP-07` | Untuk `i>=0`, `P,C>=0`, FV `>= P+n*C`. | Fixed nonnegative effective return, no fee/tax/withdrawal. |
| `PROP-08` | Untuk `n>0` dan `A(i,n)>0`, required goal contribution tidak menurun jika target naik. | Same P, i, timing, currency, fully defined model. |
| `PROP-09` | Required goal contribution `>=0`, nol bila opening attributed future value `>=T`. | Target nominal sama, valid attributed P, no hidden constraints. |
| `PROP-10` | Annual effective↔periodic effective roundtrip menjaga rate secara numerik. | `r_y>-1`, `k>0`, dan precision/tolerance sesuai policy. |
| `PROP-11` | Deflate(inflate(T,pi,n),pi,n) = T secara matematis. | `pi>-1`, periode sama, perhitungan unrounded. |
| `PROP-12` | Penambahan liability dan cash dengan nilai sama tanpa fee tidak mengubah NW. | Tidak ada revaluation/tax di event yang sama. |
| `PROP-13` | Pembayaran pokok mengurangi kas dan utang sama besar; delta NW0. | Repayment tanpa bunga/fee dan recorded cash/debt konsisten. |
| `PROP-14` | `interest_t + principal_t = PMT` dan `opening_debt - principal_t = closing_debt`. | Fixed-rate amortization unrounded, no fees/extra payments. |
| `PROP-15` | Penjumlahan posisi asset yang berbeda tidak menghitung ulang asset yang sudah direpresentasikan sebagai account. | Link account↔asset sah; valuation date sama. |
| `PROP-16` | Allocation `x_(g,t)>=0` dan `sum_g x_(g,t)<=capacity_t`. | Multi-goal math dasar, tanpa klaim solver optimal. |
| `PROP-17` | Scenario execution/replay tidak mengubah actual record count/balance. | Snapshot isolated dan auth valid. |
| `PROP-18` | Same deterministic normalized inputs+engine version memberikan hasil yang sama. | Tidak ada time-dependent/hidden random global state. |
| `PROP-19` | `0<=success_count<=valid_count<=requested_count`; success_rate=`success_count/valid_count` jika `valid_count>0`. | Definisi valid dan failure path disepakati; untuk aggregator saja. |
| `PROP-20` | Perubahan UI formatter IDR tidak mengubah value object/aggregate Money. | Formatter roundtrip sesuai locale/precision policy. |

### 20.1 Strategi generator

- Generate nominal pada range aman representasi yang disetujui; variasikan `0`, `1`, angka besar dekat batas, dan nominal non-bulat **jika diizinkan**.
- Generate `n` termasuk `0`, `1`, `2`, `12`, `60`, tenor panjang; gunakan test overflow terpisah.
- Generate `i` di sekitar nol, termasuk positif kecil dan return negatif yang masih `>-1`; `i=-1` dan `<-1` masuk error/model-specific tests.
- Generate beberapa account dan transfer dua arah, dengan invariants conserved total Money.
- Pisahkan **test generator** dari helper production; jangan memakai formula production yang sama untuk expected.
- Simpan deterministic seed untuk generator test agar failure bisa direproduksi di CI, tetapi seed test property **bukan** RNG model Monte Carlo production.

**Property equality:** untuk Money exact gunakan equality pada representasi exact; untuk operasi pangkat/statistik tetapkan toleransi terpisah sesuai `CALC-Q15` sebelum CI gate diberlakukan.

---

## 21. Integration, concurrency, security, dan persyaratan operasional

Bagian ini merupakan *test specification*, bukan bukti test yang sudah dijalankan. Dibutuhkan setelah backend Go, PostgreSQL, auth, dan background jobs benar-benar ada.

| ID | Test flow | Expected invariant | Depends on |
|---|---|---|---|
| `INT-01` | Posted transfer SQL transaction; injeksikan kegagalan sisi kredit | Seluruh unit of work rollback; saldo lama terbaca konsisten. | FR-004, `DR-007` |
| `INT-02` | Dua request paralel mengklaim idempotency key sama | Tepat satu event finansial efektif, tidak double credit/debit. | FR-004, NFR-006, `DR-008` |
| `INT-03` | Transfer dan expense paralel pada account yang sama | Tidak ada lost update, total posting merekonsiliasi saldo. | FR-004, NFR-006 |
| `INT-04` | User A mencoba read/update/delete account, goal, scenario, snapshot user B | Semua ditolak; side effect zero; response tidak mengungkap isi resource B. | FR-001, NFR-001 |
| `INT-05` | User A coba export atau delete data user B | Ditolak sama seperti endpoint privat lainnya. | FR-021, `DR-003` |
| `INT-06` | Simulation run membaca snapshot B1 dan user mem-posting transaksi baru | Run lama tetap B1; tidak memperbarui actual; stale/rebase sesuai policy. | FR-012/013, `DR-016/020` |
| `INT-07` | Retry/cancel worker sesudah partial result persisted | Partial tidak dapat diklaim `SUCCEEDED`; results final atomik; no actual mutation. | FR-018, EN-005 |
| `INT-08` | Urutan dua event berbeda pada tanggal sama belum disepakati | Tidak meloloskan production projection; status blocked/policy decision. | `CALC-Q10` |
| `INT-09` | SQL migration up pada DB bersih lalu rerun sesuai strategy | Skema valid dan constraints berfungsi; rollback/drop diuji **hanya di test DB**, bukan production. | EN-002 |
| `INT-10` | API mengirim amount di luar representasi type final | Validation error, tidak terjadi integer overflow/silent truncation. | NFR-002, `CALC-Q01/Q02` |
| `INT-11` | Login auth berhasil tetapi request memakai owner ID dari body milik user lain | Resource owner ditentukan server, tidak mengadopsi user ID dari body. | FR-001, `DR-003` |
| `INT-12` | Logging error pada operasi gagal dengan nilai akun dan token sintetis | Log structured tanpa token, raw financial payload, dan data privat antar user. | NFR-007, `DR-021` |
| `INT-13` | UI membedakan actual balance dan forecast projection pada halaman sama | Label, periode, as-of, source, disclaimers tidak ambigu. | NFR-003/005/010 |
| `INT-14` | Delete/export user dengan job simulation aktif | Request tidak bocor data; prosedur retention/cancellation konsisten sesuai policy disetujui. | FR-021, NFR-008; privacy policy TBD |

**Kebutuhan test database:** jalankan integration tests pada PostgreSQL terisolasi; gunakan rollback/cleanup yang tidak bisa menyentuh production; uji concurrency dengan synchronization eksplisit agar race bisa direproduksi. **Jangan** mengganti Postgres dengan mock lalu menyatakan transaksi atomik SQL telah diuji.

**Kebutuhan test Flutter:** widget/golden UI test harus memeriksa format angka negatif, perubahan angka akibat input, label nominal vs real, loading/failed/empty states, teks risiko, serta tidak menampilkan “sukses” untuk run yang failed. Ini tidak menggantikan numerical tests pure Go.

---

## 22. Matrik penelusuran `CS-001` s.d. `CS-016`

| Calculation contract | ID test pokok | Requirement terkait | Gate / catatan |
|---|---|---|---|
| `CS-001` Money/currency/precision | `FTC-MONEY-01..04`, `FTC-ERR-04/05` | NFR-002, DR-004/014 | `CALC-Q01/Q02/Q15` |
| `CS-002` Rate normalization | `FTC-RATE-01..06`, `FTC-ERR-03` | FR-008, NFR-002 | `CALC-Q05/Q15` |
| `CS-003` Actual balance and transfer | `FTC-ACT-01..12`, `INT-01..03` | FR-003/004, DR-005–008 | `CALC-Q09` |
| `CS-004` Net worth | `FTC-NW-01..09`, `PROP-12..15` | FR-005/006, DR-009/010/023 | Asset classification gate |
| `CS-005` Future value | `FTC-FV-01..14`, `PROP-03..07` | FR-008, EN-004 | `CALC-Q04/Q05/Q15` |
| `CS-006` Inflation | `FTC-INF-01..08`, `PROP-11` | FR-009/012/019 | `CALC-Q06` |
| `CS-007` Goal contribution | `FTC-GOAL-01..12`, `PROP-08/09` | FR-007/009, DR-011/017 | `CALC-Q08/Q11` |
| `CS-008` Budget/recurring | `FTC-BUD-01..06`, `INT-02` | FR-010, DR-013 | R2 policy TBD |
| `CS-009` Projection | `FTC-CASH-01..09`, `PROP-17/18` | FR-011/012/014 | `CALC-Q07/Q10/Q11` |
| `CS-010` Loan | `FTC-LOAN-01..10`, `PROP-13/14` | FR-012/015/019 | `CALC-Q02/Q07` |
| `CS-011` Stress test | `FTC-STRESS-01..07` | FR-015 | `CALC-Q11` |
| `CS-012` Scenario replay | `FTC-SCEN-01..08`, `INT-06` | FR-013, NFR-003/010 | `CALC-Q10` |
| `CS-013` Goal allocation | `FTC-ALLOC-01..08`, `PROP-16` | FR-017, DR-024 | **BLOCKED** `CALC-Q12` |
| `CS-014` Monte Carlo | `FTC-MC-01..10`, `PROP-19` | FR-018, DR-024 | **BLOCKED** `CALC-Q13/Q15` |
| `CS-015` Retirement | `FTC-RET-01..07` | FR-019, DR-024 | **BLOCKED** `CALC-Q14` |
| `CS-016` Insights/GPS | `FTC-INS-01..05`, `INT-13` | FR-016/020 | `CALC-Q11`, validated engine |

`FR-001` security dan `FR-021` export/deletion juga dilacak lewat `INT-04/05/11/14`; tidak dipaksakan menjadi rumus numerik. Engineering enablers R0 tetap memerlukan test CI, migrasi, konfigurasi, dan observability tersendiri sebagaimana PRD.

---

## 23. Panduan mengimplementasikan test di Go (belum ada kode nyata)

### 23.1 Organisasi yang disarankan

```text
backend/
  internal/
    finance/
      compound/              # fixed FV and required rate-dependent calculations
        ..._test.go
      projection/            # future cash positions
        ..._test.go
      allocation/            # future R3; blocked until model approved
        ..._test.go
      montecarlo/            # future R3; blocked until model approved
        ..._test.go
  internal/modules/
    accounts/                # service/integration tests
    transactions/            # atomicity/idempotency/ownership tests
    scenarios/               # immutability/version tests
  tests/
    integration/             # optional: based on final repo conventions
```

Path di atas **rekomendasi**, bukan instruksi untuk membuat semua folder kosong sekarang. Ketika arsitektur disetujui, tempatkan test sedekat mungkin dengan package yang diuji dan hindari duplikasi test harness.

### 23.2 Pola *table-driven test* yang perlu diterapkan

```text
for each approved test fixture:
    assert preconditions and explicit assumptions
    call domain function through its public domain interface
    assert expected amount with approved comparison policy
    assert expected result class, currency, rate-kind, time basis
    assert expected errors/warnings/status if present
    assert NO modifications to actual storage for any simulation
```

Saat mengimplementasikan dengan `testing.T`, beri nama subtest berdasarkan `case_id`, agar laporan CI dan perubahan rumus dapat ditelusuri ke satu referensi. Fixture test **jangan** dijadikan source formula: expected values disimpan sebagai konstanta referensi yang dihitung independen.

### 23.3 Test execution evidence template

```text
Case ID:
Implementation commit SHA:
Engine version:
Go version:
PostgreSQL version (if integration):
Command executed:
Timestamp/timezone:
Result: PASS / FAIL / SKIPPED / BLOCKED
Actual observed result:
Expected reference:
Difference / tolerance and why valid:
Failure logs (sanitized):
Reviewer:
Decision/policy approvals referenced:
```

**Jangan mengisi `PASS` bila command tidak dijalankan.** Sampai ada repository Go, status `TEST_NOT_IMPLEMENTED`; pemeriksaan pembuatan Markdown/ZIP hanya QA dokumen dan independen dari automated engine tests.

### 23.4 Test commands saat repository tersedia (ilustrasi)

```bash
cd backend
# Setelah modul benar-benar dibuat:
go test ./internal/finance/...
# Test integrasi Postgres perlu environment test dan skema yang sudah disiapkan:
go test ./internal/modules/...
```

Command tersebut **belum dijalankan untuk proyek Esokira**. Jangan mengira `go test` lulus hanya dari keberadaan spesifikasi di Markdown.

---

## 24. Register keputusan yang memblokir production

Kita **tidak membuat daftar policy baru yang bertabrakan**. Gunakan `CALC-Qxx` dari Calculation Specs §21 dan `FDR-Qxx` dari Financial Domain Rules §16; keputusan final harus tercatat sebelum test terkait menjadi release gate.

| Blocker | Efek terhadap golden tests | Aksi sebelum coding production |
|---|---|---|
| `CALC-Q01` | Unit/encoding Money dan currency parsing belum tetap. | Putuskan integer rupiah vs decimal; mapping Go/DB/API. |
| `CALC-Q02` | Rounding FV/goal/PMT dan close-out tidak bisa difinalkan. | Tetapkan rounding level, precision, final payment dan toleransi. |
| `CALC-Q03` | Tanggal, calendar boundary, EOM/leap year belum tetap. | Pilih waktu efektif dan period conversion policy. |
| `CALC-Q04` | `begin` vs `end` contribution belum dipilih untuk UI/default. | Jadikan timing input eksplisit atau kebijakan yang disetujui. |
| `CALC-Q05` | Rate kind, compounding, negative rate domain. | Tetapkan accepted input dan konversi rate. |
| `CALC-Q06` | Inflation target basis dan asumsi default. | Tentukan nominal/real contract; double inflation guard. |
| `CALC-Q07` | Loan, fees, tax, event ordering memengaruhi output. | Tentukan scope dan kebijakan tiap jenis charge. |
| `CALC-Q08` | Goal funded amount tidak bisa dipercaya tanpa atribusi. | Setujui model allocated/eligible funds. |
| `CALC-Q09` | Posting, cutover, overdraft, reversal masih TBD. | Finalisasi actual accounting lifecycle. |
| `CALC-Q10` | Scenario ordering/snapshot/version/stale TBD. | Finalisasi `SCENARIO-ENGINE.md` dan version semantics. |
| `CALC-Q11` | Capacity, liquidity buffer, runway threshold TBD. | Setujui metrik dan safeguards affordability. |
| `CALC-Q12` | Objective/tie-break optimizer tidak ditentukan. | Finalisasi `GOAL-OPTIMIZER.md` dan acceptance khusus. |
| `CALC-Q13` | RNG/distribution, quantiles, failed-path rules TBD. | Finalisasi `MONTE-CARLO.md` dan statistical validation. |
| `CALC-Q14` | Retirement spending/withdrawal/liquidity TBD. | Setujui retirement model dan timing. |
| `CALC-Q15` | Accuracy budget, toleransi dan runtime budget TBD. | Putuskan test tolerance per jenis fungsi serta compute limits. |

**Urutan approval paling penting:** uang dan posting (`CALC-Q01–Q05`, `CALC-Q09`) → goal funding (`CALC-Q08/Q11`) → projection (`CALC-Q06/Q07/Q10/Q11`) → stochastic/optimizer/retirement (`CALC-Q12–Q15`). Tidak perlu menyelesaikan seluruh R3 untuk mengerjakan R1, tetapi juga tidak boleh menonaktifkan guard R1 karena fitur advanced masih jauh.

---

## 25. Definition of Ready, Definition of Done, dan QA dokumen

### 25.1 Definition of Ready untuk sebuah test production

- [ ] Referensi FR/CS/DR sesuai fitur dan release di PRD/Feature Matrix.
- [ ] Asumsi lengkap: currency, rate kind, periode, timing, fees, basis inflasi, state source.
- [ ] Semua `CALC-Qxx` terkait telah diputuskan dan direkam oleh Product Owner.
- [ ] Expected output dihitung dari sumber independen, bukan fungsi produksi yang sama.
- [ ] Perbandingan `exact` vs `approx` dan toleransi angka telah disetujui.
- [ ] Error, status, side effects, privacy, dan negative cases terdokumentasi.

### 25.2 Definition of Done untuk implementasi numerik

- [ ] Pure Go test untuk formula utama benar-benar dijalankan dan pass.
- [ ] Edge/error cases dan property tests untuk domain tersebut pass.
- [ ] Integration tests dijalankan pada database PostgreSQL terisolasi jika ada posting/ownership/idempotency.
- [ ] Semua output menampilkan currency, jenis nilai, as-of, rate kind, horizon, dan asumsi utama.
- [ ] Tidak ada mutasi actual dari calculator atau simulation endpoint.
- [ ] Kegagalan tidak dipresentasikan sebagai success; warning/error tidak membocorkan data keuangan pribadi.
- [ ] Test evidence dan perubahan formula didokumentasikan; regresi terdeteksi oleh CI.
- [ ] Pemilik produk menerima keputusan dan perilaku yang berdampak pada interpretasi hasil finansial.

### 25.3 Review internal dokumen v0.1.0

| Pemeriksaan | Status QA dokumen |
|---|---|
| Mengacu pada lima dokumen induk dan menjaga otoritas masing-masing | CHECKED |
| Seluruh `CALC-REF-01` s.d. `CALC-REF-16` mempunyai pasangan case | CHECKED |
| Setiap `CS-001` s.d. `CS-016` mempunyai test coverage | CHECKED |
| Opening balance, transfer, NW, FV, goal, inflation, loan diperiksa secara aritmetika terpisah | CHECKED melalui skrip QA artefak; **bukan** Go tests |
| Actual tidak dicampur planned/simulated; conservation invariants tersedia | CHECKED secara review spesifikasi |
| Monte Carlo dan optimizer tetap `BLOCKED` untuk model policy | CHECKED |
| Seluruh `CALC-Q01` s.d. `CALC-Q15` tetap open | CHECKED |
| Test integration/concurrency, CI, Flutter belum tersedia | **NOT EXECUTED** — butuh implementasi |
| Product Owner menyetujui asumsi/toleransi/finance release gate | **PENDING** |

### 25.4 Ringkasan risiko yang sengaja belum diselesaikan

1. **Representasi uang** (BIGINT unit vs NUMERIC dan API encoding) menentukan exactness dan rounding test production.
2. **Konvensi waktu dan rate** dapat menyebabkan FV/angsuran berbeda sekalipun formula yang dipakai sama-sama benar.
3. **Funded goal** tidak boleh disimpulkan dari saldo seluruh akun tanpa model atribusi yang dapat diuji.
4. **Amortisasi KPR** tidak dapat diasumsikan identik dengan penawaran bank mana pun.
5. **Monte Carlo** hanya menghasilkan metrik kondisional terhadap model, bukan kepastian probabilitas dunia nyata.
6. **Optimization objective** belum ditentukan sehingga tidak ada satu solusi optimal final untuk dipakai sebagai expected.

---

## 26. Handoff ke tahapan berikutnya

Setelah `FINANCIAL-DOMAIN-RULES.md`, `CALCULATION-SPECS.md`, dan dokumen ini direview, lanjutkan secara berurutan:

1. **Review keputusan R1 yang memblokir:** `CALC-Q01–Q05`, `CALC-Q08–Q09`, serta konsistensinya dengan `FDR-Q01–Q09`.
2. `SYSTEM-ARCHITECTURE.md` dan `BACKEND-ARCHITECTURE.md`: tentukan kontrak pure Go engine, boundary API/repository, worker, dan keamanan.
3. `DATABASE-DESIGN.md` dan `API-GUIDELINES.md`: turunkan Money types, ownership, posting atomicity/idempotency, error taxonomy, source baseline/version.
4. Hanya setelah readiness gate terselesaikan: implementasikan test Go bertahap untuk R1, lalu R2 setelah `SCENARIO-ENGINE.md`, kemudian R3 setelah model spesifik disetujui.

**Aturan akhir untuk agent:** *Jangan mengarang expected output, jangan mengubah formula agar cocok dengan angka yang salah, jangan menganggap return pasti, jangan mengubah actual lewat simulasi, dan jangan mengklaim test Go pass sebelum command benar-benar dijalankan.*
