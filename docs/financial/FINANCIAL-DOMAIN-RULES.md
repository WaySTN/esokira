# Esokira — Financial Domain Rules

> **Dokumen:** `docs/financial/FINANCIAL-DOMAIN-RULES.md`  
> **Versi:** `0.1.0`  
> **Tanggal penyusunan:** 29 Oktober 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Dokumen acuan:** [`PRODUCT-VISION.md`](../product/PRODUCT-VISION.md), [`PRD.md`](../product/PRD.md), [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md), masing-masing v0.1.0  
> **Target implementasi:** Flutter (frontend), Go (backend), PostgreSQL (primary database), monorepo `/frontend` dan `/backend`  
> **Pembaca:** Product Owner, backend engineer, frontend engineer, database designer, QA, AI coding agent  
> **Dokumen turunan:** `CALCULATION-SPECS.md`, `FINANCIAL-TEST-CASES.md`, `DATABASE-DESIGN.md`, `SCENARIO-ENGINE.md`

---

## 0. Fungsi, cakupan, dan kekuatan aturan

Dokumen ini mendefinisikan **makna data keuangan, business invariants, pemisahan actual vs hypothetical, serta batas perilaku sistem**. Dokumen ini **bukan** DDL PostgreSQL, spesifikasi API, desain UI, algoritma Monte Carlo, ataupun definisi final rumus bunga majemuk.

**Aturan penggunaan untuk AI coding agent:**

1. Baca `PRODUCT-VISION.md` -> `PRD.md` -> `FEATURE-MATRIX.md` -> dokumen ini sebelum mengubah fitur finansial.
2. Gunakan `FR-xxx` di PRD sebagai sumber definisi fitur dan acceptance criteria. Dokumen ini menjadi sumber makna domain, sedangkan rumus lengkap ada di `CALCULATION-SPECS.md`.
3. Jangan mengubah sesuatu yang berstatus **TBD / Proposed** menjadi keputusan final, migration, endpoint, atau perilaku production tanpa persetujuan yang tercatat.
4. Jika ada kontradiksi dengan dokumen induk, **hentikan bagian yang terdampak**, buat catatan konflik dan rekomendasi, lalu eskalasi. Jangan memilih aturan secara diam-diam.
5. Semua contoh angka di sini **ilustrasi verifikasi aturan domain**, bukan proyeksi return investasi, saran investasi, atau golden test komputasi kompleks.

### 0.1 Legenda status

| Label | Makna | Konsekuensi |
|---|---|---|
| **REQUIRED / MUST** | Constraint yang diturunkan dari PRD/visi atau keselamatan integritas data; wajib dipertahankan. | Implementasi yang melanggar tidak boleh lulus review. |
| **PROPOSED** | Desain kerja yang direkomendasikan agar pembahasan konkret, tetapi belum disetujui. | Bisa diprototipekan secara terisolasi; tidak boleh dianggap kebijakan final. |
| **TBD / BLOCKED** | Keputusan yang memang belum ditetapkan atau memerlukan spesifikasi lanjutan. | Agent harus meminta keputusan/menandai blocker sebelum mengklaim fitur siap rilis. |
| **OUT OF SCOPE** | Tidak termasuk rilis yang dituju. | Jangan menambahkan implementasi diam-diam. |

Status dokumen **REVIEWED_INTERNAL** berarti sudah diperiksa konsistensinya dengan dokumen sebelumnya, **bukan** sudah disetujui Product Owner. Dokumen vision/PRD/matrix induk juga masih `PENDING_OWNER_APPROVAL`.

### 0.2 Lingkup rilis

| Release | Domain rules yang dominan | Referensi |
|---|---|---|
| **R0 Foundation** | Identitas pemilik data, presisi nilai, integritas dan pengujian dasar. | EN-002, EN-004; NFR-001, NFR-002, NFR-006 |
| **R1 MVP** | Akun/saldo awal, transaksi, transfer, aset, utang, net worth, goals, compound/goal forecast. | FR-003 s.d. FR-009 |
| **R2 Planning V1** | Budget, recurring, life event, baseline, skenario, forecast, stress test, insights. | FR-010 s.d. FR-016, FR-022 |
| **R3 Advanced** | Goal optimizer, simulasi risiko, retirement, Financial GPS. | FR-017 s.d. FR-020 |
| **R4 Discovery** | Shared goals dan portfolio planning; aturan domain belum cukup untuk implementasi. | FR-023, FR-024 |

---

## 1. Glosarium domain (arti yang tidak boleh bergeser)

| Istilah | Definisi operasional |
|---|---|
| **Owner / user** | Pengguna terautentikasi yang berhak mengakses suatu resource finansial; relasi ownership ditetapkan server. |
| **Actual** | Catatan finansial yang dinyatakan pengguna sebagai kejadian/posisi nyata, bukan hasil simulasi. Actual di sini **berarti dicatat sebagai nyata**, bukan diverifikasi oleh bank. |
| **Account** | Sumber saldo finansial yang dicatat, misalnya tunai atau rekening. Konvensi saldo, tipe, dan tanggal opening harus terdokumentasi. |
| **Opening balance** | Saldo awal account pada suatu titik waktu (*as-of*); **bukan** otomatis income periode tersebut. |
| **Transaction** | Kejadian finansial aktual yang diposting untuk memengaruhi account atau laporan, sesuai tipe dan state transaksi. |
| **Transfer** | Perpindahan dana antara account yang dimiliki pengguna yang sama; bukan income/expense eksternal jika tanpa fee/FX. |
| **Income** | Arus nilai dari luar batas keuangan pribadi pengguna yang memenuhi definisi pemasukan yang disetujui. Contoh R1: gaji yang benar-benar diterima. |
| **Expense** | Penggunaan nilai untuk biaya/konsumsi yang mengurangi kekayaan bersih, berbeda dari pemindahan antar-akun atau pelunasan pokok utang. |
| **Asset** | Sumber nilai ekonomi yang tercatat dalam posisi keuangan pengguna; nilainya dapat berupa saldo account atau valuasi aset lain. |
| **Liability** | Kewajiban finansial yang masih outstanding; misalnya pokok utang/kredit. |
| **Net worth** | Total nilai aset yang tidak diduplikasi dikurangi total saldo kewajiban pada *as-of* tertentu. |
| **Goal** | Target masa depan dengan jumlah, tanggal, prioritas dan status; **bukan** rekening, transaksi, ataupun instruksi investasi otomatis. |
| **Planned contribution** | Jumlah yang *direncanakan* dialokasikan ke goal; tidak boleh menambah saldo aktual atau funded progress. |
| **Funded progress** | Bagian dana aktual yang dianggap mendukung goal berdasarkan kebijakan atribusi yang **masih TBD (Q-11)**. |
| **Budget** | Batas perencanaan pengeluaran pada periode/kategori; bukan saldo kas atau kewajiban baru. |
| **Recurring rule** | Aturan jadwal transaksi yang bisa menghasilkan rencana/kejadian; tidak setara dengan transaksi aktual yang sudah diposting. |
| **Life event** | Peristiwa hidup yang dimodelkan beserta dampak finansial hipotetis (misalnya membeli rumah di masa depan). |
| **Baseline** | Kondisi awal/as-of, beserta data dan versi yang menjadi titik mulai skenario; bukan data real-time yang berubah diam-diam. |
| **Scenario** | Himpunan perubahan hipotetis terhadap baseline untuk menjawab pertanyaan *what if*. |
| **Projection / forecast** | Hasil yang dihitung dari input dan asumsi; bukan catatan aktual maupun jaminan hasil nyata. |
| **Simulation run** | Eksekusi engine yang menggunakan snapshot input, parameter, algoritma/versi, dan seed jika acak. |
| **Nominal / real** | Nilai pada satuan uang tahun berjalan / nilai daya beli relatif terhadap periode dasar sesuai model inflasi yang disetujui. |
| **As-of date** | Tanggal efektif posisi data yang dijadikan dasar perhitungan; dapat berbeda dari waktu data terakhir disunting. |
| **Recorded-at / occurred-on** | Waktu sistem merekam perubahan / tanggal kejadian finansial yang diakui; tidak boleh disamakan. |

**Penting:** aplikasi pencatatan manual tidak berhak mengklaim saldo bank pengguna telah diverifikasi hanya karena field-nya bernama `actual_balance`.

---

## 2. Daftar invariants inti (kontrak domain)

Setiap ID `DR-xxx` adalah anchor stabil untuk desain database, handler, service, UI, dan test suite. Perubahan makna memerlukan review berdampak lintas dokumen.

| ID | Aturan REQUIRED | Asal kebutuhan |
|---|---|---|
| **DR-001** | Actual dan simulated dipisahkan pada level model, storage, API, dan penyajian. | G-01; NFR-010 |
| **DR-002** | Satu perubahan scenario tidak boleh mengubah saldo, transaksi, valuasi, atau kewajiban aktual. | FR-013; NFR-010 |
| **DR-003** | Semua resource keuangan privat selalu diperiksa kepemilikannya di backend, termasuk nested resource dan export. | FR-001; NFR-001 |
| **DR-004** | Nominal aktual harus diproses/disimpan dengan representasi presisi aman, tanpa floating-point biner sebagai sumber saldo. | NFR-002; Q-04 |
| **DR-005** | Opening balance tidak boleh dihitung ulang sebagai income dan transaksi sesudahnya tidak boleh digandakan. | FR-003 |
| **DR-006** | Transfer antar-account milik user yang sama adalah netral terhadap total kekayaan dan total income/expense, selama tidak ada fee, FX, atau selisih lain. | FR-004; FR-006 |
| **DR-007** | Transfer yang diposting bersifat atomik: debit/kredit terkait berhasil bersama atau gagal bersama. | FR-004; NFR-006 |
| **DR-008** | Satu kejadian finansial tidak boleh menghasilkan dua posting akibat retry request/job. | NFR-006 |
| **DR-009** | Aset berupa saldo account yang sudah dihitung tidak boleh dihitung ulang sebagai baris aset independen. | FR-005; FR-006 |
| **DR-010** | `net_worth = total_nonduplicated_assets - total_outstanding_liabilities`; angka dapat negatif. | FR-006 |
| **DR-011** | Setoran/penyisihan dana yang *direncanakan* tidak boleh diperlakukan sebagai kas yang benar-benar ada atau kontribusi yang sudah terealisasi. | FR-007; FR-009 |
| **DR-012** | Goal, budget, dan scenario tidak otomatis mengeksekusi perpindahan uang. | Product non-goals; FR-007; FR-020 |
| **DR-013** | Future recurring event tidak otomatis menjadi actual yang sudah terjadi tanpa mekanisme posting yang disepakati. | FR-010; FR-012 |
| **DR-014** | Output finansial harus jelas menyebut sumber (`actual`/`planned`/`forecast`), unit uang, periode/as-of, dan asumsi relevan. | NFR-003; NFR-005 |
| **DR-015** | Return, inflation, model success rate, dan prediksi tidak boleh disajikan sebagai hasil terjamin. | G-04; FR-018, FR-019 |
| **DR-016** | Angka simulasi harus bisa ditelusuri ke versi baseline, parameter, dan engine; seed disimpan bila stokastik. | NFR-003 |
| **DR-017** | Jika input kritis tidak ada atau bertentangan, jangan mengarang angka atau asumsi tersembunyi demi menghasilkan tampilan sukses. | FR-002; FR-009; NFR-002 |
| **DR-018** | Waktu efektif kejadian, waktu pencatatan, dan periode laporan tidak boleh dicampur tanpa aturan eksplisit. | FR-004; FR-006; NFR-002 |
| **DR-019** | Insight harus dapat dilacak ke input dan hasil engine; tidak mengeksekusi transfer atau tindakan eksternal sendiri. | FR-016; FR-020 |
| **DR-020** | Perubahan aktual yang memengaruhi baseline harus menyebabkan refresh yang eksplisit atau penandaan stale pada hasil simulasi sebelumnya. | FR-012, FR-013; NFR-003 |
| **DR-021** | Data sensitif tidak boleh bocor lewat log, analitik umum, stack trace, atau pesan error. | FR-001; NFR-007 |
| **DR-022** | Pembuatan, koreksi, atau pembatalan transaksi harus menjaga konsistensi saldo dan dapat ditelusuri sesuai kebijakan audit yang disetujui. | FR-004; Q-03 |
| **DR-023** | Asset, liability, income, expense, dan transfer harus memiliki klasifikasi yang konsisten sehingga arus kas tidak disamakan dengan perubahan net worth. | FR-004 s.d. FR-006 |
| **DR-024** | Simulasi finansial lanjutan tidak boleh dipublikasikan tanpa spesifikasi model dan reference test yang disetujui. | NFR-002; FR-017–FR-020 |

---

## 3. Uang, mata uang, tanda, dan presisi

### 3.1 Money value

**REQUIRED:** semua besaran uang mempunyai arti unit yang eksplisit. `100000` tanpa konteks currency, satuan, dan apakah actual/forecast bukan representasi domain yang cukup untuk interchange antarmodul.

**PROPOSED** value object konseptual:

```text
Money {
  amount: exact_decimal_or_integer_unit,  // keputusan representasi TBD (Q-04)
  currency: ISO_currency_code,              // locale awal IDR adalah PROPOSED
}
```

- Untuk pencatatan aktual, **hindari `float32`/`float64` sebagai sumber kebenaran saldo**. Tipe database final (`BIGINT` unit integer vs `NUMERIC`) dan kontrak serialisasi masih **TBD (Q-04)**.
- Jika memakai `NUMERIC`, tentukan precision/scale dan kapan rounding boleh terjadi. Jika memakai unit integer, tetapkan dulu satuan terkecil yang secara produk didukung. **Jangan berasumsi semua input IDR selalu boleh/tidak boleh memiliki desimal sebelum kebijakan disetujui.**
- Jangan menggunakan format UI seperti `"Rp1.200.000"` sebagai nilai perhitungan/penyimpanan. Format ribuan, simbol, dan pemisah desimal adalah tanggung jawab presentation layer.
- Aturan rounding (*half-up* / *half-even* / lain), akumulasi sisa pecahan, dan pembulatan hasil proyeksi **TBD (Q-04)**; ditentukan di `CALCULATION-SPECS.md`.
- Monetary comparisons harus menggunakan unit dan mata uang yang sama; penjumlahan beda currency tanpa rate dan tanggal konversi yang disetujui adalah **invalid**.

**PROPOSED untuk R1:** IDR-only dengan satuan yang distandardisasi di backend. Multi-currency, FX gain/loss dan konversi nilai belum termasuk requirement awal.

### 3.2 Sign conventions dan amount semantics

**REQUIRED:** API, DB, dan engine harus menggunakan konvensi tanda yang konsisten; jangan ada negatif ganda yang membuat suatu expense menjadi income.

**PROPOSED:** input transaction memakai besaran positif (`amount > 0`) dan `type` eksplisit (`income`, `expense`, `transfer`) serta `source_account`/`destination_account` yang sesuai. Pembalikan/koreksi tidak dilakukan dengan menyisipkan expense bertanda negatif tanpa domain action yang terdokumentasi.

- `0` pada transfer dan transaksi baru dianggap **invalid secara usulan**, kecuali produk secara eksplisit membutuhkan zero-value adjustment.
- Saldo account atau net worth **boleh** negatif jika model account mengizinkan; tidak boleh otomatis dijepit ke nol hanya agar UI terlihat baik. Aturan overdraft account tetap **TBD**.
- Negative rate atau loss investasi adalah kemungkinan yang berbeda dari negative recorded transaction amount. Jangan campurkan.

### 3.3 Kapan nilai dihitung

- **Principal/balance actual:** exact value sesuai posting.
- **Rates, volatility, inflation:** parameter model yang unitnya eksplisit (misal tahunan vs bulanan; effective vs nominal); detail konversi ada di `CALCULATION-SPECS.md`.
- **Stochastic paths:** pendekatan numerik dapat memakai floating-point untuk sampling statistik sesuai model yang nanti disetujui; tidak boleh menjadikannya sumber ledger saldo aktual.
- **Display:** aturan pembulatan tampilan tidak boleh diam-diam mengubah nilai persisted.

---

## 4. User ownership, access, dan privasi domain

### 4.1 Ownership

**REQUIRED:** setiap account, transaction, goal, asset, liability, scenario, scenario version, simulation run/result, dan insight bersifat privat terhadap user, kecuali fitur shared goals kelak secara eksplisit mendefinisikan ACL berbeda.

- Identitas owner diambil dari konteks autentikasi server yang telah diverifikasi, bukan `user_id` arbitrer dari request body/query.
- Relasi child -> parent **tidak cukup** tanpa memeriksa kepemilikan parent; ID yang valid tetap tidak membuktikan izin.
- Endpoint agregat/dashboard hanya boleh menggabungkan data milik pengguna yang berhak.
- Export dan penghapusan data melewati pemeriksaan ownership setara endpoint biasa.
- Shared goals (FR-023) dan perubahan izin lintas pengguna **OUT OF SCOPE R1–R3**, kecuali keputusan PRD kelak berubah.

### 4.2 Sensitive data boundary

- Jangan log payload lengkap transaksi, angka saldo pribadi, informasi identitas sensitif, token akses, atau random seed yang disatukan dengan data pribadi tanpa kebijakan logging yang aman.
- Analytics produk tidak menerima nominal, deskripsi transaksi, atau nama goal mentah secara implisit.
- Data finansial contoh/test harus sintetis; tidak memakai data asli pengguna tanpa otorisasi, minimisasi, dan kebijakan pemrosesan yang jelas.
- Retention, jenis export, mekanisme delete akun, backup erasure, serta consent tetap tunduk pada `SECURITY-PRIVACY.md` (**TBD Q-10**, jangan diasumsikan otomatis lengkap).

---

## 5. Accounts, opening balances, dan posting transaksi aktual

### 5.1 Account dan snapshot pembukaan

**REQUIRED:** account mempunyai owner, currency (bila relevan), klasifikasi, tanggal saldo awal, dan status yang dapat ditelusuri. Saldo awal adalah **stock/position**, bukan **flow/income**.

**PROPOSED:** akun awal sederhana seperti `cash`, `bank`, dan `savings`, dengan identitas stabil meskipun nama tampilan diubah. Investment/loan account khusus akan memerlukan pemetaan tambahan pada aset/kewajiban yang disetujui.

**Aturan saldo konseptual untuk sebuah account:**

```text
closing_balance(as_of) = opening_balance(as_of_open)
                       + sum(posted_inflows up to as_of)
                       - sum(posted_outflows up to as_of)
                       + sum(approved_adjustments up to as_of)
```

Hanya transaksi yang **tercakup setelah cutover opening balance** dapat masuk ke agregasi tersebut. Aturan transaksi *backdated* atau edit tanggal opening **TBD Q-03**, karena jika keliru akan menggandakan riwayat uang.

**PROPOSED:** `opening_balance` memiliki `effective_date` yang jelas. Jika pengguna memasukkan transaksi sebelum tanggal itu, minta tindakan rekonsiliasi/saldo awal; jangan menambahkannya diam-diam ke closing balance.

### 5.2 Income, expense, dan transfer

| Kejadian actual | Perubahan posisi | Income periode? | Expense periode? | Efek net worth (tanpa perubahan valuasi lain) |
|---|---|---|---|---|
| Gaji diterima tunai/ke rekening | Account bertambah | Ya | Tidak | Bertambah |
| Belanja konsumsi dibayar tunai | Account berkurang | Tidak | Ya | Berkurang |
| Transfer bank A -> bank B, tanpa fee | A turun, B naik sama besar | Tidak | Tidak | Tidak berubah |
| Transfer bank A -> bank B plus fee | A turun sesuai dana keluar; B naik nominal diterima | Tidak | Fee dicatat expense | Berkurang sebesar fee |
| Memindahkan uang ke account investasi milik sendiri | Saldo asal turun, saldo investasi naik | Tidak | Tidak untuk pokok transfer | Tidak berubah saat pemindahan pokok |
| Pembelian aset nonkas dari account | Kas turun, aset baru bertambah pada nilai tercatat | Tidak | Biaya terkait bila diklasifikasikan expense | Umumnya netral untuk pertukaran nilai setara; biaya menurunkan NW |
| Melunasi pokok utang dari kas | Kas dan liability turun sama besar | Tidak | Bukan expense konsumsi untuk porsi pokok | Tidak berubah karena pembayaran pokok saja |
| Membayar bunga/biaya pinjaman | Kas turun | Tidak | Ya, jika diakui sebagai biaya sesuai domain | Turun sebesar biaya |

Tabel ini menjelaskan **sifat ekonomi domain**, bukan mewajibkan model tabel debit/credit tertentu. Perincian sumber/destinasi, fee, refund, penyesuaian, dan pencatatan transaksi majemuk dirancang kemudian.

**REQUIRED:** transfer terdiri atas efek account yang terkait dalam **satu operasi atomik**. Jika salah satu sisi gagal, saldo kedua account harus tetap seperti sebelum operasi.

### 5.3 State, koreksi, dan keamanan retry

- **REQUIRED:** sistem tidak boleh mengklaim transaksi sudah diposting bila commit database gagal.
- **REQUIRED:** request yang diulang karena timeout tidak boleh menggandakan posting yang sama.
- **REQUIRED:** koreksi dan pembatalan tidak boleh menghilangkan konsistensi saldo agregat maupun jejak perubahan yang diwajibkan.
- **TBD Q-03:** apakah transaksi bisa diedit langsung sebelum posting, bagaimana reversal/void, audit history, status `draft/posted/void`, serta kebijakan penyesuaian saldo.
- **PROPOSED:** pisahkan pembuatan draft dari efek posted; gunakan idempotency key untuk operasi create/post yang dapat diulang, transaksi SQL untuk transfer, dan record audit yang aman. Pemilihan desain final memerlukan `DATABASE-DESIGN.md` dan API contract.
- **TBD:** perilaku pemindahan uang antar owner, multi-currency transfer, split transaction, refund lintas periode, overdraft, import duplikat. **Jangan diimplementasikan secara tersirat.**

### 5.4 Tidak menghitung income dua kali

Sumber income adalah event yang diakui, bukan perubahan saldo account semata. Jika seseorang menambah opening balance Rp10 juta dan kemudian mencatat gaji Rp2 juta, `income` periode hanya Rp2 juta; opening balance tetap membentuk saldo, tetapi bukan gaji.

---

## 6. Assets, liabilities, net worth, dan valuasi

### 6.1 Asset tidak boleh diduplikasi

**REQUIRED:** daftar aset harus punya aturan apakah suatu posisi diwakili oleh **saldo account** atau **valuasi aset terpisah**, bukan keduanya sekaligus.

**PROPOSED** kategori pelaporan:

```text
Total assets
  = Sum(eligible account balances as_of)
  + Sum(other nonduplicated asset valuations as_of)
```

- Rekening tabungan Rp10 juta yang telah menjadi `account` jangan dibuat lagi sebagai `asset` senilai Rp10 juta lalu dijumlahkan keduanya.
- Aset investasi dapat menjadi account/holding atau valuation object sesuai desain data, tetapi hanya **satu** representasi berkontribusi pada total aset pada satu *as-of*.
- Nilai aset yang dilaporkan pengguna harus disertai tanggal valuasi; aplikasi tidak boleh mengklaim harga pasar live tanpa feed sah.
- **TBD:** klasifikasi aset likuid/non-likuid, basis nilai (historical cost vs market estimate), depresiasi, aset yang diagunkan, dan hubungan antar holding/investment account.

### 6.2 Liability dan outstanding principal

**REQUIRED:** outstanding liability adalah stock yang berbeda dari arus pembayaran cicilan.

- Pinjaman baru Rp10 juta meningkatkan kas Rp10 juta **dan** kewajiban Rp10 juta; net worth tidak otomatis bertambah Rp10 juta.
- Pembayaran pokok Rp1 juta menurunkan kas dan outstanding liability masing-masing Rp1 juta; perubahan net worth dari komponen pokok adalah nol.
- Bunga/biaya pinjaman yang diakui sebagai expense mengurangi net worth dan tidak menurunkan pokok pinjaman kecuali ada aturan kapitalisasi yang disetujui.
- **TBD Q-05:** bunga berjalan, amortization schedule, tanggal bunga, denda, pembayaran dipercepat, refinancing, default, serta metode pengakuan biaya.
- **PROPOSED:** simpan outstanding principal dan tanggal perhitungan; tampilkan bahwa nilai dapat stale jika tidak direkonsiliasi.

### 6.3 Rumus position

```text
net_worth(as_of) = total_nonduplicated_assets(as_of)
                 - total_outstanding_liabilities(as_of)
```

- `net_worth` boleh negatif; jangan membuat `max(net_worth, 0)`.
- `net_worth` adalah posisi pada satu tanggal, bukan cash flow atau keuntungan investasi.
- Laporan harus mengungkap cakupan data: belum mencatat aset tertentu, valuasi lama, utang belum terbarui, atau currency unsupported -> **incomplete/stale**, bukan klaim total penuh.
- Jika account negatif mencerminkan overdraft, klasifikasi asset/liability untuk pelaporan belum final; **tidak boleh dipotong atau dihitung dua kali**.

---

## 7. Goals dan pendanaan: planned bukan actual

### 7.1 Definisi goal

Goal minimal memiliki `owner`, `name`, `target_amount`, `target_date`, `priority`, dan lifecycle state sesuai FR-007. Representasi target apakah nilai nominal pada tanggal tersebut atau nilai daya beli riil harus jelas, bukan disimpulkan dari nama goal.

**REQUIRED:**

1. `planned_contribution` dan `actual_funded` adalah dua fakta berbeda.
2. Menambah goal **tidak** menambah kas, utang, atau nilai aset aktual.
3. Mengubah target atau tanggal **tidak** menciptakan atau menghapus transaksi aktual.
4. Menandai goal selesai adalah state progres tujuan, bukan bukti uang telah dibelanjakan/ditransfer.
5. Satu rupiah account yang sama tidak boleh otomatis dianggap membiayai dua goals jika model mengklaim dana tersebut sudah dialokasikan secara eksklusif.
6. Angka kebutuhan setoran adalah hasil perhitungan yang memerlukan rumus, horizon, rate, dan asumsi yang disetujui (`CALCULATION-SPECS.md`).

### 7.2 Alternatif model goal funding — **Q-11 masih BLOCKED**

| Alternatif (PROPOSED) | Arti progress funded | Tantangan |
|---|---|---|
| **Virtual envelope** | Sejumlah saldo existing ditandai untuk goal; **tidak menciptakan aset baru**. | Perlu menjamin alokasi tidak melebihi saldo eligible dan tidak terpakai dua kali. |
| **Linked dedicated account** | Saldo account tertentu dikaitkan ke goal secara eksklusif/sebagian. | Butuh aturan linking, un-linking, dan transaksi yang menggunakan account tersebut. |
| **Explicit contributions** | Event kontribusi terkonfirmasi membentuk progres; pergerakan uang aktualnya tetap mengikuti model account. | Perlu menentukan apakah event kontribusi selalu membutuhkan transfer nyata atau hanya earmarking. |

**Jangan pilih salah satu secara otomatis.** Sebelum FR-007/FR-009 production-ready, Product Owner harus menyetujui definisi `funded_amount`, bagaimana bukti pendanaan dibuat, penghitungan `progress%`, dan pengaruh penarikan/realokasi. Semua alternatif harus mencegah double counting.

### 7.3 Feasibility dan kapasitas tabungan

- *Available contribution capacity* bukan selalu `income - expense`. Bisa ada cicilan pokok, pajak, kewajiban jatuh tempo, cadangan, transaksi musiman, dan saldo yang tidak boleh digunakan.
- **REQUIRED:** jika input tidak cukup menentukan kemampuan menabung, hasil ditandai *estimate/insufficient data*; jangan klaim target pasti feasible.
- **TBD Q-04/Q-05:** definisi kapasitas, urutan prioritas tujuan, jadwal kontribusi, pertumbuhan nilai target akibat inflasi, dan penanganan target terlambat.
- Multi-goal optimizer R3 memiliki objective dan constraint tersendiri (`GOAL-OPTIMIZER.md`); jangan disisipkan heuristik tersembunyi ke MVP.

---

## 8. Budgets dan recurring transactions (R2)

### 8.1 Budget adalah batas rencana

- Budget kategori/bulanan adalah referensi untuk membandingkan pengeluaran aktual, bukan saldo account baru.
- Transfer internal tidak menambah expense kategori budget, kecuali fee yang secara sah masuk kategori biaya.
- Budget dapat terlampaui tanpa otomatis menolak transaksi aktual; blocking spending memerlukan requirement produk terpisah.
- **TBD:** definisi category taxonomy, rollover budget, shared category, expense split, dan perlakuan refund.

### 8.2 Recurring rule vs posted instance

- Template gaji bulanan atau cicilan masa depan **bukan** actual posting sampai mekanisme konfirmasi/posting disetujui.
- Proyeksi boleh memperhitungkan jadwal recurring sebagai asumsi, tetapi laporan actual tidak boleh otomatis menjumlahkan rencana tersebut.
- **REQUIRED:** pembuatan satu kejadian recurring tidak boleh diduplikasi bila scheduler dijalankan ulang.
- **TBD:** frekuensi, tanggal jatuh tempo akhir bulan, skip/holiday policy, generated draft vs automatic posting, serta kebijakan notifikasi FR-022.

---

## 9. Waktu, periode, cutoff, dan stale data

**REQUIRED:** bedakan tiga dimensi waktu:

| Dimensi | Untuk apa | Contoh |
|---|---|---|
| `occurred_on` / effective date | Mengakui transaksi ke periode yang benar. | Belanja tanggal 28 Oktober. |
| `recorded_at` | Mengetahui kapan sistem menyimpan/mengubah catatan. | Baru diinput tanggal 29 Oktober. |
| `as_of` / valuation date | Mengetahui kapan posisi saldo/valuasi berlaku. | Nilai rumah per 1 Oktober. |

**PROPOSED:** simpan timestamp audit dengan `timestamptz` yang terstandar; gunakan *local date* untuk transaksi yang hanya membutuhkan tanggal kalender. Zona waktu pengguna, tanggal batas bulan, serta implementasi detail tipe DB akan diputuskan di `DATABASE-DESIGN.md`.

- Jangan memakai waktu server untuk menggeser tanggal transaksi pengguna secara diam-diam.
- Goal deadline adalah tanggal kalender; jangan mengasumsikan selalu tanggal 31 atau selalu tepat 30 hari per bulan.
- Nilai current dashboard harus memisahkan data yang baru dicatat dari data yang efektif pada tanggal lebih lama.
- Kalau transaksi backdated mengubah hasil proyeksi sebelumnya, hasil tersebut harus ditandai stale atau tetap dapat direproduksi terhadap snapshot lama; jangan diam-diam memperbarui history hasil simulasi.
- **TBD Q-04:** urutan setoran awal/akhir bulan, prorata periode parsial, konvensi kalender, leap year, perbandingan bulan, serta rounding timeline.

---

## 10. Batas actual -> projection -> scenario

### 10.1 Pemisahan data secara konseptual

```text
ACTUAL (recorded financial facts)
  Accounts, Transactions, Asset Valuations, Liabilities
                   |
                   | read-only snapshot / explicit baseline reference
                   v
BASELINE (as-of + data version / snapshot fingerprint)
                   |
             hypothetical overrides
                   v
SCENARIO VERSION (assumptions + life events + goal changes)
                   |
                   | engine version + seed (if stochastic)
                   v
SIMULATION RUN -> FORECAST / RISK METRICS / EXPLAINABLE INSIGHTS

NO PATH from simulation output directly into actual ledger.
```

### 10.2 Baseline dan reproducibility

**REQUIRED:**

- Scenario harus menyebut baseline yang dipakai, bukan mengikuti nilai current secara implisit setelah disimpan.
- Hasil run dapat diaudit dengan mengetahui **input snapshot atau immutable reference yang cukup, parameter yang digunakan, engine version, run time, dan random seed bila acak**.
- Saat pengguna mengubah actual setelah baseline disimpan, hasil lama tetap terikat input lama; run berikutnya terhadap baseline baru adalah run lain, bukan perubahan sunyi terhadap hasil historis.
- Perbandingan skenario dengan baseline berbeda harus diberi peringatan atau disamakan secara eksplisit sebelum disebut *like-for-like*.
- Scenario dan simulation result adalah data sensitif milik user yang sama; akses ke child resource tetap memverifikasi owner.

**TBD Q-06:** apakah baseline memakai snapshot penuh, versioned read model, atau gabungan keduanya; schema untuk `scenario_version`, invalidation policy, serta ukuran/retensi hasil.

### 10.3 Life events, shock, dan status rencana

- *Life event* seperti membeli rumah atau kehilangan pekerjaan adalah **asumsi hipotetis** selama pengguna belum mencatat kejadian tersebut sebagai actual melalui workflow yang disetujui.
- Event dengan cash impact berulang harus dibedakan dari *one-time* cash flow; detail timing dan urutan efek ditentukan di `SCENARIO-ENGINE.md`.
- Scenario yang gagal mencapai target harus menampilkan shortfall/tenggat/pilihan, bukan mengubah asumsi tanpa pemberitahuan agar terlihat berhasil.
- Forecast dan stress test tidak memberikan instruksi otomatis untuk menarik dana, membeli produk, atau mengubah akun aktual.

---

## 11. Batas antara domain rules dan spesifikasi algoritma

Dokumen ini **menetapkan guardrail**, bukan mengunci rumus/parameter matematis lanjutan yang belum disetujui.

| Area | Aturan domain yang sudah wajib | Yang harus ditetapkan pada dokumen turunan |
|---|---|---|
| Compound interest (FR-008) | Output = estimate; unit rate/horizon jelas; rate 0 dan kondisi tidak valid di-handle. | Formula tepat, nominal/effective conversion, compounding frequency, timing setoran, rounding. |
| Goal contribution (FR-009) | Planned != funded; insufficient input tidak boleh disamarkan. | Solusi setoran, capacity definition, target inflation, feasibility criteria. |
| Deterministic projection (FR-012) | Actual immutable; baseline dan asumsi tersedia; hasil reproducible. | Loop periode, urutan cash flow/debt/interest, opening & closing states. |
| Stress test (FR-015) | Shock hipotetis; kas tidak boleh dianggap mampu menutup kewajiban tanpa cek. | Shock types, depletion threshold, recovery assumptions. |
| Multi-goal optimizer (FR-017) | Jumlah alokasi tidak melampaui batasan dana; infeasible tidak disembunyikan. | Objective function, constraint, priority handling, feasible/infeasible reference cases. |
| Monte Carlo (FR-018) | Tidak menyamakan model success rate dengan kepastian dunia nyata. | Distribution, correlation, seed, RNG, iteration count, model validation. |
| Retirement/FIRE (FR-019) | Nominal/real dibedakan; aturan 4% bukan jaminan. | Withdrawal, depletion, income sources, inflation and horizon conventions. |
| Financial GPS (FR-020) | Semua alternatif berdasarkan hasil engine terlacak; no auto transfer. | Decision rules, ranking criteria, limitations, explanation format. |

**Batasan tegas:** jangan menulis *default return 8%/tahun*, inflasi 3%, probabilitas sukses 90%, atau safe withdrawal 4% sebagai nilai yang sudah disetujui. Jika contoh diperlukan, tandai **hypothetical example only** dan minta nilai eksplisit dalam runtime.

---

## 12. Domain APIs, state transition, dan error semantics

Ini bukan definisi endpoint OpenAPI; ini kontrak perilaku yang perlu diterjemahkan ke dalam schema/API design.

### 12.1 Klasifikasi keluaran

Setiap response atau model yang menyajikan angka finansial seharusnya cukup untuk membedakan:

```text
actual_position        // posisi finansial tercatat as_of tertentu
actual_period_flow     // income/expense periode tertentu
planned_goal           // target dan kontribusi yang direncanakan
forecast_deterministic // proyeksi dengan asumsi tertentu
risk_model_result      // estimasi statistik model, bukan kepastian
```

Jangan menggunakan label umum `balance` untuk hasil simulasi dan saldo actual tanpa `kind`, provenance, dan periode yang eksplisit.

### 12.2 Validasi dan error yang harus didukung

| Kondisi | Perilaku wajib |
|---|---|
| Account tujuan transfer tidak dimiliki user | Akses ditolak tanpa membocorkan metadata owner lain. |
| Amount negatif/format invalid/tidak dapat direpresentasikan | Validasi gagal; tidak ada posting parsial. |
| Currency kedua operand berbeda, konversi belum didukung | Operasi ditolak atau menunggu FX specification; tidak ada konversi implisit. |
| Nominal melebihi representasi/toleransi yang diperbolehkan | Operasi ditolak secara eksplisit; tidak overflow/rounding sunyi. |
| Retry create/post request | Hasil identik atau rekonsiliasi aman; tidak membuat posting kedua. |
| Proyeksi tanpa data penting | Tampilkan `insufficient_data` dengan daftar input yang perlu dilengkapi; jangan mengarang isi. |
| Angka valuasi lawas | Tampilkan tanggal/keterbatasan dan status stale, bukan `live valuation`. |
| Engine/job gagal | Status failed dengan informasi aman untuk user dan diagnostic code untuk engineer, tanpa mengubah actual. |
| Scenario baseline berubah | Tampilkan baseline masih lama/stale atau tawarkan rebase yang eksplisit. |

**PROPOSED** hasil domain menggunakan error code stabil (contoh `INVALID_MONEY`, `CURRENCY_MISMATCH`, `ACCOUNT_NOT_OWNED`, `INSUFFICIENT_DATA`, `STALE_BASELINE`, `INFEASIBLE_GOAL`). Nama final menjadi bagian API contract dan `API-GUIDELINES.md`.

### 12.3 Backend source of truth

- Backend/financial engine teruji adalah sumber aturan domain; Flutter menampilkan dan mengumpulkan input, bukan mendefinisikan ulang posting atau feasibility.
- Flutter boleh menyediakan preview input lokal yang **jelas non-authoritative** apabila rumus dan konvensinya tersedia dan diuji seragam.
- Data import dari file eksternal tidak langsung dianggap valid sampai melewati klasifikasi, validasi, deduplikasi, dan ownership checks.

---

## 13. Contoh kasus untuk review aturan domain

> Semua nominal di bawah **ilustrasi aritmetika sederhana** (IDR tanpa biaya/interest kecuali disebut). Bukan acuan model matematika lanjutan. Untuk golden cases, lihat dokumen berikutnya `FINANCIAL-TEST-CASES.md`.

### CASE-01 — Opening balance bukan income

- Akun A opening = Rp10.000.000; selama periode dicatat gaji masuk Rp3.000.000; tidak ada transaksi lain.
- Closing A = Rp13.000.000; **income periode = Rp3.000.000**, bukan Rp13.000.000.
- Menambah opening lagi sebagai income akan melanggar **DR-005**.

### CASE-02 — Internal transfer netral

- Saldo A = Rp10.000.000, saldo B = Rp3.000.000.
- Transfer Rp2.000.000 A -> B, tanpa biaya.
- Saldo baru A = Rp8.000.000, B = Rp5.000.000. Total tetap **Rp13.000.000**; income = 0; expense = 0.
- Jika B gagal dikredit, A **tidak boleh** tetap turun Rp2.000.000 (**DR-007**).

### CASE-03 — Transfer dengan biaya

- Transfer Rp2.000.000 A -> B, dan dikenakan biaya Rp5.000 yang dibayar dari A.
- A turun **Rp2.005.000**, B naik **Rp2.000.000**; expense = **Rp5.000**; total aset kas turun Rp5.000.
- Biaya harus diklasifikasi eksplisit; jangan diam-diam mengubah nominal transfer.

### CASE-04 — Membeli aset bukan expense pokok otomatis

- Akun kas Rp25.000.000. Pengguna membeli aset yang dicatat dengan nilai Rp20.000.000 dari kas, tanpa biaya.
- Kas menjadi Rp5.000.000, aset nonkas Rp20.000.000, total asset Rp25.000.000.
- Pencatatan itu **tidak** boleh sekaligus menyajikan expense Rp20.000.000 untuk laporan perubahan net worth tanpa kebijakan pengakuan yang terpisah. Jika biaya tambahan ada, perlakukan sesuai kategorinya.

### CASE-05 — Pokok pinjaman berbeda dari bunga

- Saldo kas Rp12.000.000, pokok pinjaman outstanding Rp5.000.000. Bayar pokok Rp1.000.000 dan bunga Rp100.000.
- Kas menjadi Rp10.900.000; liability menjadi Rp4.000.000.
- Net worth sebelum Rp7.000.000, sesudah Rp6.900.000; **penurunan Rp100.000 adalah biaya bunga**, bukan karena pembayaran pokok Rp1.000.000.

### CASE-06 — Net worth negatif tetap valid

- Total aset eligible Rp8.000.000, liability Rp12.000.000.
- Net worth = **-Rp4.000.000**. Jangan diubah menjadi Rp0 atau disembunyikan.

### CASE-07 — Planned goal tidak menciptakan uang

- Kas actual Rp4.000.000. User membuat goal Rp30.000.000 dan berencana menabung Rp1.000.000 per bulan.
- Kas actual **tetap Rp4.000.000**. Planned contribution bukan income/transfer yang sudah terjadi.
- Funded progress masih **unknown/bergantung Q-11**, bukan otomatis Rp1.000.000 atau Rp4.000.000.

### CASE-08 — Simulasi membeli rumah tidak mengubah actual

- Pengguna membuat scenario dengan DP Rp50.000.000 pada tahun tertentu.
- Hasil proyeksi boleh memperlihatkan kas hipotetis turun pada periode tersebut.
- Tidak ada perubahan saldo actual, liabilities actual, ataupun real transaction sampai user mengambil aksi pencatatan actual yang terpisah dan disetujui.

### CASE-09 — Duplicate request harus aman

- Client mengirim POST posting transfer, server commit, respons jaringan terputus, lalu client retry dengan identitas operasi yang sama.
- Hasil yang benar: **satu** transfer posted, **bukan dua** transfer; metode deduplikasi/idempotency ditetapkan dalam API/DB design.

### CASE-10 — Data aktual diperbarui setelah simulation run

- Simulation run S1 dibuat dari baseline tanggal 1 Oktober. User lalu menambahkan expense actual backdated tanggal 30 September.
- S1 tetap menjelaskan baseline lamanya atau ditandai stale; tidak boleh berubah diam-diam menjadi kalkulasi yang memakai data baru.
- Re-run terhadap baseline baru menghasilkan S2 dengan metadata/run ID berbeda.

### CASE-11 — Stale valuation tidak sama dengan real-time

- Nilai kendaraan Rp50.000.000 dicatat pada 1 Januari; dashboard dibuka 29 Oktober tanpa pembaruan valuasi.
- Boleh ditampilkan sebagai nilai tercatat per 1 Januari, tetapi tidak sebagai nilai pasar *hari ini* yang terverifikasi.

### CASE-12 — Simulated goal contribution tidak boleh double spend

- Dua goals masing-masing secara hipotetis mengklaim Rp4.000.000 dana yang *funded*, tetapi dana eligible eksklusif hanya Rp5.000.000.
- Sistem tidak boleh melaporkan total funded Rp8.000.000 secara eksklusif tanpa aturan alokasi yang menjaga konsistensi. Model funding final masih Q-11.

---

## 14. Edge cases dan test obligations (bukan pengganti golden tests)

| ID | Skenario wajib diperiksa | Rule | Status spesifikasi numerik |
|---|---|---|---|
| EC-01 | Saldo awal nol dan transaksi tanpa income | DR-005 | Domain clear; DB posting TBD |
| EC-02 | Transfer dari dan ke account sama | DR-006, DR-007 | **TBD:** reject atau no-op explicit; jangan implement diam-diam |
| EC-03 | Transfer saldo tidak cukup/overdraft | DR-007 | **TBD Q-03** kebijakan saldo negatif |
| EC-04 | Transfer gagal di antara dua posting | DR-007 | Harus rollback atomik |
| EC-05 | Request yang sama diulang akibat timeout | DR-008 | Harus dedupe/idempotent |
| EC-06 | Expense pada tanggal sebelum opening | DR-005, DR-018 | **TBD Q-03** backdated policy |
| EC-07 | Akun dihapus tetapi masih punya transaksi | DR-022 | **TBD** archive/delete policy |
| EC-08 | Saldo account dan asset merepresentasikan uang yang sama | DR-009 | Harus cegah duplikasi |
| EC-09 | Liability negatif/tidak valid | DR-010, DR-023 | **TBD** adjustment/refund model |
| EC-10 | Net worth negatif | DR-010 | Harus didukung |
| EC-11 | Planned goal tanpa actual funding evidence | DR-011 | Jangan klaim funded |
| EC-12 | Recurring run dua kali | DR-008, DR-013 | Jangan double post |
| EC-13 | Target goal lewat tanggal, shortfall | DR-017 | Perlu formula & status di calc specs |
| EC-14 | Rate 0 dan hasil negatif pada projection | DR-015 | Perlu reference numerical cases |
| EC-15 | Target mata uang berbeda | DR-004 | Tidak ada FX implicit |
| EC-16 | Tahun kabisat dan target akhir bulan | DR-018 | **TBD Q-04** kalender |
| EC-17 | User A mencoba akses scenario milik B | DR-003 | Harus ditolak |
| EC-18 | Actual berubah setelah scenario disimpan | DR-020 | Harus stale/explicit refresh |
| EC-19 | Monte Carlo seed dan input sama | DR-016 | Reproducibility policy di MC spec |
| EC-20 | User input tidak lengkap namun meminta insight | DR-017, DR-019 | Tampilkan insufficient-data |
| EC-21 | Perhitungan near integer/decimal limit | DR-004 | Validate overflow/precision |
| EC-22 | Simulasi heavy job gagal/timeout | DR-001, DR-016 | Tidak ada actual mutation |

---

## 15. Traceability dari domain rules ke PRD dan testing

| Domain area | Relevant FR / EN / NFR | Minimal bukti implementasi nantinya |
|---|---|---|
| Ownership/privacy | FR-001, FR-021, EN-002/003, NFR-001/007/008 | Cross-user access tests, private export/delete coverage |
| Accounts & opening | FR-003, FR-006, NFR-002 | Golden opening/closing balance cases |
| Income/expense/transfer | FR-004, FR-006, NFR-006 | Atomic posting, retry/idempotency, period aggregation tests |
| Assets & liabilities | FR-005, FR-006 | Anti-double-count, net worth negative and debt repayment tests |
| Goals & contributions | FR-007, FR-009 | Planned-vs-funded tests and contribution integrity checks |
| Compound calculator | FR-008, EN-004 | Golden mathematical tests from Calculation Specs |
| Budgets & recurring | FR-010, FR-022 | Recurring deduplication, actual-vs-plan tests |
| Life event & scenario | FR-011, FR-012, FR-013, FR-014, NFR-003/010 | No mutation, snapshot/versioning, replay tests |
| Stress & insights | FR-015, FR-016 | Safe explanation and insufficient data tests |
| Advanced simulation | FR-017, FR-018, FR-019, FR-020, EN-005 | Validated reference tests, reproducibility, model disclaimers |

**Pengujian wajib minimum:** unit test financial domain, transactional integration test PostgreSQL untuk posting/transfer, property/invariant tests pada aliran nilai dan ownership, serta regression golden cases. `FINANCIAL-TEST-CASES.md` harus memuat input, expected output, langkah pembuktian, toleransi numerik jika diperlukan, dan kaitan ke `DR-xxx`.

---

## 16. Open decisions — jangan diisi oleh agent tanpa approval

Tabel berikut adalah penguraian blocker Q-xx dari PRD; bukan keputusan baru yang otomatis berlaku.

| ID | Terkait PRD | Keputusan yang perlu disetujui | Pilihan/implikasi yang perlu dibahas |
|---|---|---|---|
| **FDR-Q01** | Q-03 | Model opening balance dan backdated transaction. | Cutover event, penyesuaian saldo, historical import, reconciliation. |
| **FDR-Q02** | Q-03 | Edit, void/reverse, audit, dan lifecycle transaksi. | Mutable draft / immutable posted + reversal / alternatif. |
| **FDR-Q03** | Q-03 | Overdraft, account negatif, fee, refund, split & transfer same account. | Validasi balance, klasifikasi dan audit. |
| **FDR-Q04** | Q-04 | Mata uang awal dan representasi uang exact. | IDR-only + integer unit vs fixed-precision decimal; output API as string/unit. |
| **FDR-Q05** | Q-04 | Rounding, scale, negative rate, format dan overflow limit. | Aturan per operasi vs final; payment schedules. |
| **FDR-Q06** | Q-04 | Zona waktu, tanggal effective, period closing, end-of-month. | Date-only vs timestamp; user timezone; leap year. |
| **FDR-Q07** | Q-11 | Funded progress dan hubungan goal-account. | Virtual envelope vs dedicated account vs explicit contributions. |
| **FDR-Q08** | Q-05 | Kebijakan liabilitas/interest/penilaian aset/inflasi. | Outstanding, valuation basis, nominal vs real, freshness. |
| **FDR-Q09** | Q-04/Q-05 | Formula, compounding timing, kapasitas kontribusi. | Menjadi bagian `CALCULATION-SPECS.md`. |
| **FDR-Q10** | Q-06 | Baseline dan rebase/version scenario. | Snapshot vs immutable reference, staleness, retention. |
| **FDR-Q11** | Q-07 | Objective function, constraints, tie-breaking untuk optimizer. | Menjadi bagian `GOAL-OPTIMIZER.md`. |
| **FDR-Q12** | Q-08 | RNG/statistical model untuk Monte Carlo. | Menjadi bagian `MONTE-CARLO.md`; belum mengunci distribusi. |
| **FDR-Q13** | Q-10 | Retensi, export, deletion, dan aturan privasi lanjutan. | Menjadi bagian `SECURITY-PRIVACY.md`. |

### 16.1 Proposal diskusi bertahap (BUKAN approved defaults)

Rekomendasi untuk sesi review Product Owner berikutnya:

1. Putuskan **model opening/posting dan koreksi** terlebih dahulu (FDR-Q01–Q03). Ini memblokir FR-003–FR-006.
2. Putuskan **money type, precision, timezone, rounding** (FDR-Q04–Q06). Ini memengaruhi DB, API, dan seluruh engine.
3. Putuskan **model funded goal** (FDR-Q07) sebelum fitur goal progress dibangun.
4. Baru tetapkan formula deterministik di `CALCULATION-SPECS.md` (FDR-Q08–Q09).
5. R3: spesifikasi optimizer/Monte Carlo dibuat menjelang implementasinya, bukan ditebak di R0.

**Jika Product Owner belum memutuskan:** hanya desain/prototipe terisolasi yang diberi label *assumption*, bukan migrasi atau behavior yang diklaim final.

---

## 17. Definition of Ready untuk domain implementation

Suatu modul finansial baru boleh masuk implementasi production bila semua yang relevan sudah terpenuhi:

- [ ] FR-ID dan acceptance criteria yang relevan disetujui Product Owner.
- [ ] Domain rules `DR-xxx` untuk modul tersebut tidak bertentangan dengan Product Vision/PRD.
- [ ] Blocker Q/FDR-Q yang memengaruhi modul sudah ditetapkan dalam decision log/ADR.
- [ ] Model schema PostgreSQL dan integritas transaksinya direview.
- [ ] Kontrak OpenAPI/API request-response, ownership, dan idempotency dijelaskan.
- [ ] Rumus, periode, rate, dan rounding (jika numerik) ada di `CALCULATION-SPECS.md` yang telah direview.
- [ ] Golden/reference test cases relevan telah divalidasi independen dalam `FINANCIAL-TEST-CASES.md`.
- [ ] Error, empty, insufficient-data, dan stale states tersedia.
- [ ] Tidak ada data simulasi yang mengubah actual secara implisit.
- [ ] Security/privacy review relevan dan cross-user access tests siap dijalankan.

**Definition of Done:** semua acceptance serta tes benar-benar dijalankan dan lulus; tidak cukup hanya tampilan selesai. Jika salah satu gate masih gagal, status yang benar adalah *blocked/in review*, bukan *done*.

---

## 18. Review internal, batasan, dan approval

### 18.1 Review konsistensi dokumen

- [x] Terminologi sesuai Product Vision (V-01 s.d. V-05) dan PRD v0.1.0.
- [x] Cakupan R0–R4 dan FR-001–FR-024 yang relevan tidak dipindahkan diam-diam ke rilis lain.
- [x] Actual vs simulated dibedakan secara eksplisit pada domain, contoh kasus, dan aturan API.
- [x] Transfer, opening balance, dan nilai aset memiliki perlindungan terhadap double counting.
- [x] Goal funding, monetary precision, rounding, timeline, dan stochastic assumptions tetap berstatus TBD sesuai PRD.
- [x] Setiap contoh aritmetika sederhana di dokumen ini telah direview; bukan model forecast produksi.
- [x] Dokumen ini **tidak** mengklaim perubahan teknis sudah diterapkan pada kode/database yang belum diaudit.
- [ ] **Product Owner menyetujui domain rules serta menyelesaikan blocker untuk modul pertama.**

### 18.2 Batasan dokumen

Dokumen ini tidak menyatakan bahwa Esokira berlisensi, berstatus financial advisor, mampu menyinkronkan bank, atau dapat memastikan keberhasilan target. Kewajiban hukum, privasi dan pilihan penyedia layanan harus dievaluasi tersendiri menjelang rilis sesuai cakupan produk dan yurisdiksi.

### 18.3 Referensi teknis pendukung (bukan pengganti keputusan produk)

- PostgreSQL — [Numeric Types](https://www.postgresql.org/docs/current/datatype-numeric.html): perbedaan exact `numeric` dan inexact floating-point untuk data uang.
- PostgreSQL — [Date/Time Types](https://www.postgresql.org/docs/current/datatype-datetime.html): tipe `date`, `timestamp`, dan `timestamptz` memiliki semantik berbeda.
- OWASP API Security Top 10 — [Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/): pengecekan ownership tidak dapat digantikan dengan asumsi ID resource sulit ditebak.

---

## 19. Urutan dokumen berikutnya

1. Review dan persetujuan domain rules ini, terutama FDR-Q01–FDR-Q09 yang berdampak pada R1.
2. Susun **`docs/financial/CALCULATION-SPECS.md`** dari aturan yang sudah jelas, sembari memberi **BLOCKED/TBD** pada formula yang belum diputuskan.
3. Susun **`docs/financial/FINANCIAL-TEST-CASES.md`** berisi test vector terverifikasi untuk setiap formula dan invariant.
4. Sesudah itu, turunkan ke `DATABASE-DESIGN.md`, `SYSTEM-ARCHITECTURE.md`, dan implementasi engine Go; jangan menebak skema atau rumus dari contoh UI.

**Ringkasan aturan terpenting:** *Actual is not forecast. Transfer is not an expense. A goal is not a bank account. Opening balance is not income. Recorded assets must not be double counted. Financial calculations must be explainable and testable.*
