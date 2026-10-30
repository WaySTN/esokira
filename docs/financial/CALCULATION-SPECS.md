# Esokira — Financial Calculation Specifications

> **Path canonical:** `docs/financial/CALCULATION-SPECS.md`  
> **Versi:** `0.1.0`  
> **Disusun:** 29 Oktober 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Rilis:** R1 (MVP), R2 (Planning V1), R3 (Advanced); R4 tidak menjadi komitmen implementasi  
> **Dokumen induk:** [`PRODUCT-VISION.md`](../product/PRODUCT-VISION.md), [`PRD.md`](../product/PRD.md), [`FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md), [`FINANCIAL-DOMAIN-RULES.md`](./FINANCIAL-DOMAIN-RULES.md), semuanya v0.1.0  
> **Dokumen berikutnya:** `FINANCIAL-TEST-CASES.md`  
> **Target kode:** Go financial engine (pure domain calculation), diakses oleh Go API/worker; Flutter hanya mengirim parameter dan menyajikan output.

---

## 0. Maksud, batas, dan cara membaca

Dokumen ini adalah **kontrak perhitungan** untuk Esokira: definisi parameter, satuan, persamaan, status input, urutan perhitungan, perilaku edge case, serta contoh numerik yang dapat dipakai saat menyiapkan *golden tests*. Ini **bukan** skema database, API OpenAPI final, atau implementasi yang sudah disetujui. Aturan pemaknaan data ada di `FINANCIAL-DOMAIN-RULES.md` dan tidak boleh ditimpa oleh rumus dalam dokumen ini.

### 0.1 Aturan untuk AI coding agent

1. Baca Product Vision -> PRD -> Feature Matrix -> Financial Domain Rules -> dokumen ini sebelum coding perhitungan.
2. Pisahkan tiga hal: **identitas matematis** (benar jika premis dipenuhi), **kebijakan produk** (perlu persetujuan), dan **model probabilistik** (perlu spesifikasi/validasi statistik).
3. Label **REQUIRED** adalah invariant yang sudah diturunkan dari dokumen induk; **PROPOSED / EXAMPLE PROFILE** hanyalah opsi untuk diuji; **TBD / BLOCKED** menghalangi klaim production-ready.
4. Jangan menganggap angka ilustrasi sebagai default return, inflasi, limit tabungan, toleransi, atau pengaturan production.
5. Jika kebijakan waktu pembayaran, jenis bunga, precision, pengakuan biaya/pajak, atau definisi pendanaan belum ditetapkan, **jangan memilih diam-diam**. Catat blocker `CALC-Qxx` dan kaitkan ke `Q-xx`/`FDR-Qxx`.
6. Engine perhitungan tidak boleh menulis ke data aktual. Hasil forecast atau simulasi tidak otomatis menjadi transaksi aktual (`DR-001`, `DR-002`, `DR-011`, `DR-012`).
7. Semua rumus yang dipakai harus memiliki reference input/output yang terverifikasi dalam `FINANCIAL-TEST-CASES.md` sebelum rilis terkait (`DR-024`, `NFR-002`).

### 0.2 Pemetaan tingkat kesiapan

| Area kalkulasi | Rilis | Status kontrak di dokumen ini | Gate sebelum production |
|---|---|---|---|
| Saldo aktual, transfer, net worth | R1 | **REQUIRED** invariants; rincian lifecycle **TBD** | Q-03, FDR-Q01–Q03; test transaksi SQL |
| Compound/future value | R1 / FR-008 | Identitas rumus **DEFINED**; convention/timing/rounding **PROPOSED/TBD** | Q-04, Q-05, golden cases |
| Required contribution dan goal feasibility | R1 / FR-009 | Rumus kasus sederhana **DEFINED**; funded model/capacity **TBD** | Q-11, Q-04/Q-05, golden cases |
| Deterministic projection & cash-flow | R2 / FR-012/014 | Model state dan urutan **PROPOSED** | Q-05/Q-06, SCENARIO-ENGINE.md |
| Life event, stress, financial insights | R2 / FR-011/015/016 | Semantik perbedaan skenario **DEFINED**; desain detail **PROPOSED** | Q-05/Q-06, test pembanding |
| Goal allocation optimizer | R3 / FR-017 | Formulasi minimum **PROPOSED**, objective **BLOCKED** | Q-07, GOAL-OPTIMIZER.md |
| Monte Carlo risk | R3 / FR-018 | Statistik output **DEFINED**; model sampling **BLOCKED** | Q-08, MONTE-CARLO.md |
| Retirement/FIRE | R3 / FR-019 | Accounting penarikan **DEFINED**; policy/model **BLOCKED** | Q-05/Q-08, review khusus |
| Financial GPS | R3 / FR-020 | Insight berbasis hasil, **bukan formula baru** | FR-016, engine tervalidasi, review penjelasan |
| Shared goals, portfolio | R4 / FR-023/024 | **OUT OF SCOPE** perhitungan operasional | Discovery + revisi PRD |

### 0.3 Urutan otoritas ketika konflik

1. Product requirement, release scope, dan acceptance -> `PRD.md`.
2. Makna actual, aset, utang, goal funding, ownership, dan larangan double count -> `FINANCIAL-DOMAIN-RULES.md`.
3. Kontrak matematika, unit rate, output, dan contoh hitungan -> dokumen ini setelah disetujui.
4. `FINANCIAL-TEST-CASES.md` -> nilai expected dari rumus yang disetujui, bukan sumber kebijakan baru.
5. Implementasi Go/DB/API -> harus mengikuti kontrak di atas, **tidak** menjadi alasan untuk mengubah rumus tanpa ADR/decision log.

Jika bertentangan, hentikan area terdampak, catat konflik, dan minta review; jangan memutuskan secara tersembunyi.

---

## 1. Notasi dan kosakata kuantitatif

| Simbol | Definisi | Satuan |
|---|---|---|
| `P` | Saldo/modal awal pada `t=0` | Money dalam currency tunggal |
| `B_t` | Saldo hasil perhitungan setelah akhir periode `t` | Money |
| `C_t` | Kontribusi pada periode `t` ke posisi tujuan/investasi yang dimodelkan | Money/periode |
| `W_t` | Penarikan dari posisi yang dimodelkan pada periode `t` | Money/periode |
| `E_t` | Pengeluaran eksternal yang benar-benar diasumsikan terjadi dalam periode | Money/periode |
| `Y_t` | Pendapatan eksternal yang diasumsikan dalam periode | Money/periode |
| `F_t` | Biaya/pajak **yang telah didefinisikan secara eksplisit** dalam model | Money/periode |
| `L_t` | Pokok kewajiban outstanding setelah periode `t` | Money |
| `r_y` | Return/inflasi **efektif tahunan** (bukan persentase antarmuka) | Desimal/tahun |
| `j_y` | Rate nominal tahunan yang dikompaun `m` kali/tahun | Desimal/tahun |
| `i_t` | Return/rate **efektif per periode** `t` | Desimal/periode |
| `n` | Banyaknya interval kalkulasi utuh (bukan otomatis selisih tanggal/30) | integer periode |
| `T_n` | Target nilai nominal pada tanggal akhir | Money |
| `pi_t` | Inflasi efektif per periode | Desimal/periode |
| `A(i,n)` | Faktor anuitas untuk setoran akhir periode | dimensionless |
| `as_of` | Tanggal kondisi awal actual/baseline | calendar date |
| `engine_version` | Identitas versi implementasi/model numerik | string/version |

Semua persentase menggunakan **decimal fraction** dalam engine: `8% -> 0.08`, `-5% -> -0.05`. Tidak boleh mencampur nilai input UI `8` dengan engine `0.08` tanpa field unit eksplisit. `monthly_rate=0.01` berarti **1% per bulan**, bukan 1% per tahun.

### 1.1 Profil contoh, bukan aturan final

Bagian contoh yang membutuhkan angka memakai profil **EXAMPLE-ONLY** berikut, kecuali bagian menyatakan lain:

- Currency `IDR`, dengan nominal uang pada contoh dinyatakan dalam rupiah.
- Horizon `n` bulan penuh, mulai `t=0`; setoran terjadi di **akhir bulan** untuk contoh future value biasa.
- Return konstan diketahui **hanya sebagai asumsi matematis**, tanpa biaya/pajak, dan tanpa inflasi kecuali ditentukan eksplisit.
- Rate efektif tahunan diubah menjadi rate efektif bulanan dengan formula di §3.1, atau **rate bulanan disebut langsung**.
- Hasil rumus dijaga presisinya pada tahap kalkulasi dan angka untuk UI dapat dibulatkan hanya setelah aturan pembulatan disetujui.

Ini **tidak menetapkan** pilihan default aplikasi. Production harus menyimpan dan menampilkan asumsi pengguna secara eksplisit.

---

## 2. Invariants numerik dan kontrol presisi (REQUIRED)

### 2.1 Money dan currency

- Semua penjumlahan/pengurangan Money mewajibkan **mata uang serta satuan yang kompatibel**. Data currency berbeda -> `CURRENCY_MISMATCH`, bukan auto-konversi FX.
- Saldo actual dan transaksi aktual **tidak** memakai binary floating point (`float32`/`float64`) sebagai sumber kebenaran (`DR-004`). Representasi final PostgreSQL/Go/API tetap **TBD Q-04 / FDR-Q04**.
- Angka rate, exponent, volatility, dan model numerik dapat memakai teknik numerik yang berbeda **setelah** toleransi dan validasinya disetujui. Jangan memindahkan angka Monte Carlo secara otomatis ke ledger.
- Jangan melakukan pembulatan setiap intermediate operation kecuali algoritma memang mensyaratkan demikian; jangan menyimpan format presentasi seperti `Rp1.000.000` sebagai nilai mesin.
- Saat memakai `NUMERIC` atau desimal Go, tentukan precision, scale, rounding, dan overflow melalui keputusan pemilik produk (`FDR-Q05`).
- Hasil yang tak terhingga, `NaN`, error overflow, atau precision loss yang tidak terkontrol adalah **error**, bukan angka yang boleh ditampilkan sebagai prediksi.

### 2.2 Kelas nilai (source of truth)

| Jenis nilai | Makna | Boleh menulis actual ledger? |
|---|---|---|
| `ACTUAL` | Posisi/kejadian dicatat pengguna sebagai nyata; bukan otomatis tervalidasi bank | Hanya lewat operasi transaksi aktual terotorisasi |
| `PLANNED` | Kontribusi/biaya yang belum terjadi | Tidak |
| `FORECAST` | Hasil deterministik berdasarkan parameter | Tidak |
| `STOCHASTIC_RESULT` | Ringkasan beberapa lintasan berdasarkan model risiko | Tidak |
| `INSIGHT` | Penjelasan/alternatif dari data dan engine | Tidak |

`actual != planned != forecast`. Kontribusi yang direncanakan tidak boleh diubah menjadi bukti dana goal sudah tersedia (`DR-011`).

### 2.3 Ketelitian dan ketidakpastian

- Untuk formula eksak sederhana, hasil acuan dihitung dengan integer/decimal/rational independen.
- Untuk perpangkatan rate pecahan dan fungsi statistik, tetapkan toleransi **absolut/relatif per fungsi** di golden tests; jangan mengandalkan perbandingan `float` exact secara universal.
- `epsilon`, banyak digit rate, dan rounding Money **TBD**; jangan menyelundupkan `1e-6` atau `round to nearest rupiah` sebagai kebijakan global.
- Model prediksi wajib menampilkan jenis rate, periode, fees/taxes jika dimasukkan, serta label **estimasi, bukan jaminan** (`DR-014`, `DR-015`).

---

## 3. Normalisasi rate dan waktu

### 3.1 Annual effective -> periodic effective (identitas matematis)

Untuk `k` interval kompaun yang **seragam** per tahun (contoh 12 bulan) dan return efektif tahunan `r_y > -1`:

```text
i = (1 + r_y)^(1/k) - 1
r_y = (1 + i)^k - 1
```

Untuk numerik dekat nol, bentuk yang stabil secara floating-point ialah `expm1(log1p(r_y) / k)` jika library numerik yang dipilih mendukungnya dan tervalidasi. `r_y = 0 -> i = 0`. `r_y = -1` merupakan kehilangan 100% satu tahun dan membutuhkan kebijakan pemodelan khusus; jangan pakai `log1p(-1)` pada jalur umum. `r_y < -1` tidak valid untuk model long-only return efektif tahunan yang sederhana.

**Catatan:** `annual_effective_rate = 12%` **tidak** sama dengan `monthly_rate = 1%`. Rate bulanan efektif dari 12% tahunan adalah sekitar `0.948879%` per bulan.

### 3.2 Annual nominal -> periodic rate (kontrak terpisah)

Jika diberikan rate nominal tahunan `j_y` dengan frekuensi kompaun `m` kali/tahun:

```text
i_m = j_y / m
annual_effective = (1 + j_y/m)^m - 1
```

Formula ini **hanya berlaku** untuk definisi *nominal annual rate convertible m times*. Jangan menerapkannya pada efektif tahunan, suku bunga flat, APR biaya efektif, atau aturan bunga harian tanpa konversi/model yang benar. `m > 0`; `1 + j_y/m` harus sesuai domain produk.

### 3.3 Kalender dan rentang waktu

- `n=12` berarti **12 periode bulanan** yang didefinisikan, bukan otomatis `365/30`.
- Tanggal, akhir bulan, hari kabisat, partial-month, dan timezone adalah kebijakan **TBD FDR-Q06**.
- `occurred_on`, `recorded_at`, `as_of`, dan tanggal valuasi tidak dapat saling menggantikan (`DR-018`).
- Jika horizon berasal dari dua tanggal yang tidak sejalan dengan konvensi periode, hasil harus mengungkap apakah diprorata, ditolak, atau dibulatkan berdasarkan policy yang disetujui. Jangan otomatis `int(days/30)`.

### 3.4 Penanganan rate negatif

- Return investasi negatif adalah valid **jika berada pada domain model yang disetujui**; jangan menggantinya dengan nol.
- Model long-only dengan gross return satu periode mensyaratkan `1+i_t >= 0`, tetapi perilaku saat tepat `-100%` perlu ditetapkan eksplisit.
- Rate bunga pinjaman negatif, biaya negatif, dan berbagai batas ekstrem adalah kasus domain terpisah; **TBD** dan harus dites apabila dibuka oleh produk.

---

## 4. Kontrak input-output minimum financial engine

### 4.1 Input yang harus dapat diaudit

| Kelompok | Parameter wajib/opsional secara konseptual | Perhatian |
|---|---|---|
| Identity & baseline | `owner_reference`, `baseline_id`, `as_of`, `baseline_fingerprint` | Input engine tidak otomatis mengotorisasi akses; otorisasi tetap di service Go |
| Unit | `currency`, `amount_unit`, `rate_kind`, `rate_frequency` | Hindari rate ambigu: tahunan? efektif? nominal? |
| Time | `start_date`, `end_date`/`n`, `period_unit`, `contribution_timing` | Konvensi kalender dan timing harus eksplisit |
| Cash & account | Opening position, income/expense, transfers, fees, liabilities | Actual dan planned berbeda |
| Assumptions | Return series, inflation model, charges/taxes, life events | Input tidak lengkap -> error/insufficient data, bukan default tersembunyi |
| Reproducibility | `engine_version`, `model_version`, `scenario_version`, `seed` (jika acak) | Re-run deterministik harus teridentifikasi |

Ini **bukan JSON API yang sudah disahkan**. Penamaan field final akan diatur OpenAPI dan rancangan backend.

### 4.2 Bentuk hasil konseptual

```text
CalculationResult {
  result_class: ACTUAL | PLANNED | FORECAST | STOCHASTIC_RESULT,
  amount_currency,
  as_of_or_start, end, period_unit,
  model_name, model_version, engine_version,
  assumptions_used, source_baseline,
  values_by_period, aggregate_metrics,
  warnings[], errors[],
  is_stale_or_incomplete,
}
```

Data hasil simulasi harus **read-only terhadap actual**. Input yang sama dan versi engine yang sama menghasilkan output deterministik yang sama; seeded random reproducibility membutuhkan kesamaan RNG, model statistik, dan strategi komputasi (`DR-016`, `NFR-003`).

---

## 5. Saldo aktual, transfer, dan net worth (R1; FR-003–FR-006)

### 5.1 Saldo per account (invariant, bukan skema ledger final)

```text
closing_balance(account, as_of)
  = opening_balance(account, opening_as_of)
  + sum(posted inflows in allowed interval)
  - sum(posted outflows in allowed interval)
  + sum(approved adjustments in allowed interval)
```

- Opening balance merupakan **stock**, bukan income.
- Interval sejak tanggal pembukaan sampai `as_of` harus mengikuti aturan cutover/backdated yang masih **BLOCKED Q-03**.
- Hanya kejadian **posted** yang berhak memengaruhi saldo actual; draft/recurring/planned bukan actual.
- Perubahan transaksi tidak boleh membuat hasil aggregasi salah dan retry tidak boleh menggandakan posting (`DR-005`, `DR-008`, `DR-022`).

### 5.2 Internal transfer (REQUIRED)

Jika pengguna memindahkan `x` dari account A ke account B, tanpa biaya/FX:

```text
A_after = A_before - x
B_after = B_before + x
(A_after + B_after) = (A_before + B_before)
external_income_delta = 0
external_expense_delta = 0
net_worth_delta = 0
```

Jika ada biaya `f` yang dibebankan ke A (tanpa biaya lain):

```text
A_after = A_before - x - f
B_after = B_before + x
sum_cash_delta = -f
recognized_expense_delta = +f
net_worth_delta = -f
```

Keduanya harus diposting atomik; formula bukan pengganti transaksi database yang aman. Perubahan currency/FX = model terpisah dan **bukan cakupan R1**.

### 5.3 Aset, kewajiban, dan net worth

```text
total_assets(as_of)
  = sum(eligible account values as_of)
  + sum(other nonduplicated asset valuations as_of)

net_worth(as_of)
  = total_assets(as_of) - total_outstanding_liabilities(as_of)
```

- Jangan menambahkan nilai investasi/account yang sama sekali lagi sebagai aset (`DR-009`).
- Jangan menjepit net worth negatif menjadi nol (`DR-010`).
- Pada penarikan pinjaman `x` yang sekaligus menambah kas `x` dan outstanding debt `x`, perubahan NW awal = 0 jika tanpa fee.
- Pada pembayaran pokok pinjaman `x`, kas dan outstanding turun `x`; perubahan NW dari pokok = 0. Bunga/fee yang diakui adalah beban terpisah (NW turun).
- Nilai aset yang usang/tidak lengkap -> label *as-of* dan *stale*, bukan klaim valuasi pasar real-time.

**Edge cases yang memblokir keputusan:** overdraft dan klasifikasinya, aset berdenominasi lain, saldo historis yang diedit, bunga akrual, pembayaran sebagian/lebih, fee FX (`FDR-Q01–Q03`, `FDR-Q08`).

---

## 6. Compound interest / future value (R1; FR-008)

### 6.1 Fixed periodic return, setoran akhir periode (mathematical reference)

Misalkan `P >= 0`, setoran konstan `C >= 0` pada **akhir** setiap periode, return efektif per periode `i`, dan `n` periode utuh:

```text
A(i,n) = ((1+i)^n - 1)/i   for i != 0
A(0,n) = n

FV_end = P*(1+i)^n + C*A(i,n)
```

Dengan `1+i > 0` untuk kasus umum. Ini kontrak matematis **kasus fixed rate**, bukan keputusan bahwa semua fitur harus memakai setoran akhir bulan.

**Recurrence yang ekuivalen** dan lebih mudah diperluas:

```text
B_0 = P
for t = 1..n:
    B_t = B_(t-1)*(1+i) + C
```

Di sini `B_t` adalah posisi **setelah setoran akhir periode t**. Saat `i=0`, `FV_end = P + n*C`; saat `n=0`, `FV_end=P` dan belum ada setoran periode. Bila `P=0`, `C=0`, hasil nol. Jangan melakukan pembagian `i` jika `i=0`; penanganan numerik mendekati nol harus stabil.

### 6.2 Setoran awal periode (alternative reference; choice TBD)

Jika setoran terjadi **sebelum** return periode:

```text
B_0 = P
for t = 1..n:
    B_t = (B_(t-1) + C)*(1+i)

FV_begin = P*(1+i)^n + C*A(i,n)*(1+i)
```

Saat `i=0`, `FV_begin=P+n*C`. Timing awal/akhir bulan **TBD Q-04 / FDR-Q09** dan harus dipilih atau diekspos jelas pada product UI/API. Perhitungan awal/akhir **tidak dapat dibandingkan seolah sama**.

### 6.3 Setoran variabel, return variabel, dan penarikan (extension, tidak otomatis scope R1)

Untuk return periodik `i_t` dan setoran/penarikan akhir periode, formulasi kandidat:

```text
B_0 = P
B_t = B_(t-1)*(1+i_t) + C_t - W_t - F_t
```

Jika setoran terjadi di awal, operasi dan biaya harus diurutkan eksplisit, misalnya `(B_(t-1) + C_t)*(1+i_t) - W_t - F_t`. Ini **PROPOSED** hanya untuk diskusi R2: model waktu fee, pajak, withdrawal, dan batas saldo di bawah nol masih **TBD Q-05**. Tidak boleh menafsirkan `F_t=0` diam-diam saat UI mengklaim sudah memperhitungkan biaya.

### 6.4 Perincian output

Untuk skenario sederhana tanpa withdrawal/fee:

```text
total_contribution = P + n*C
estimated_gain = FV - total_contribution
```

Untuk skenario dengan withdrawal yang eksplisit:

```text
net_external_funding = P + sum(C_t) - sum(W_t)
net_growth_after_modeled_fees = B_n - net_external_funding
```

Penamaan kedua metrik harus mencerminkan definisi biaya/pajak. `estimated_gain` boleh negatif; jangan paksa positif. Komponen `P` adalah modal awal dan **bukan** kontribusi bulanan yang baru dibuat.

### 6.5 Validasi dan kasus tepi

| Kondisi | Perilaku |
|---|---|
| `n = 0` | Return `P` untuk formula fixed FV; tidak menciptakan satu setoran fiktif. |
| `i = 0` | Gunakan limit `A(0,n)=n`, tidak membagi nol. |
| `i` mendekati nol | Gunakan numerik stabil atau recurrence tervalidasi, bukan formula yang kehilangan digit penting. |
| `i < 0` | Nilai investasi bisa turun; tampilkan penurunan, jangan ubah ke nol. |
| `i <= -1` | **BLOCKED/model-dependent**; tidak boleh masuk formula umum tanpa kebijakan eksplisit. |
| `P`/`C` negatif pada R1 | **PROPOSED invalid** untuk kalkulator modal dan tabungan sederhana; withdrawal terpisah. |
| `n < 0`, bukan integer, input non-finite | Tolak sebagai input invalid. |
| Biaya/pajak tidak dispesifikasikan | Jangan mengklaim hasil 'net setelah biaya/pajak'; ungkap bahwa komponen itu tidak dimodelkan. |

### 6.6 Contoh numerik referensi (tanpa pembulatan)

- `P=1.000.000`, `C=100.000`, `n=12`, `i=0`, setoran akhir -> `FV=2.200.000`.
- `P=0`, `C=100.000`, `n=2`, `i=0.01 per bulan`, setoran akhir -> `FV=201.000`.
- Input sama, setoran awal -> `FV=203.010`.
- `P=1.000.000`, `C=0`, `n=2`, `i=0.01 per bulan` -> `FV=1.020.100`.

Contoh ini menguji identitas formula. Akan diturunkan menjadi reference/golden cases lengkap dan independen pada `FINANCIAL-TEST-CASES.md`.

---

## 7. Inflasi, nilai nominal, dan nilai riil (R1–R3, policy TBD)

### 7.1 Future nominal target dari purchasing power hari ini

Untuk biaya setara `T_0` pada periode dasar dan inflasi konstan **efektif per periode** `pi`:

```text
T_n_nominal = T_0_real_at_base * (1+pi)^n
```

Untuk nominal masa depan `V_n_nominal` yang ingin dinyatakan dalam daya beli periode dasar:

```text
V_n_real_at_base = V_n_nominal / (1+pi)^n
```

Syarat umum `pi > -1`. `T_0` harus diberi label **harga sekarang**; jika input goal **sudah harga nominal pada tanggal target**, jangan kalikan inflasi sekali lagi (double-inflation bug).

### 7.2 Rate riil secara konsisten

Jika return efektif nominal `r` dan inflasi efektif `pi` pada **periode yang sama**:

```text
real_return = (1+r)/(1+pi) - 1
```

Pendekatan `r-pi` hanyalah aproksimasi yang tidak eksak. Laporan harus membedakan *nominal value* dan *real value* serta periode dasar yang dipakai (`DR-014`).

### 7.3 Contoh ilustratif

`T_0=1.000.000`, inflasi efektif tahunan `10%`, `n=2 tahun` -> `T_2=1.210.000`. Nilai riil dari `1.210.000` pada akhir tahun 2 kembali `1.000.000` dalam daya beli tahun dasar.

**BLOCKED:** memilih asumsi inflasi default, memperkirakan inflasi setiap tahun, mengintegrasikan data historis, dan mendefinisikan apakah target kenaikan harga mengikuti inflasi umum atau indeks biaya spesifik (`Q-05`, `FDR-Q08`).

---

## 8. Required contribution & goal feasibility (R1; FR-007/009)

### 8.1 Setoran tetap minimal untuk satu goal, akhir periode

Jika target nominal pada akhir `n` periode adalah `T_n`, saldo eligible dan **diatribusi ke goal** saat ini adalah `P_g`, return tetap `i`, dan kontribusi konstan terjadi di akhir periode:

```text
required_C_raw = (T_n - P_g*(1+i)^n) / A(i,n)
required_C = max(0, required_C_raw)
```

Formula hanya berlaku bila `n > 0`, `A(i,n) > 0`, unit/return/timing kompatibel dan funding `P_g` tidak bertentangan dengan aturan atribusi goal. Untuk setoran awal periode, gantikan denominator dengan `A(i,n)*(1+i)` bila > 0. Semua angka ini adalah *planned funding*, bukan aktual.

Jika produk membolehkan pembulatan setoran minimum ke satuan uang (`round_up_to_money_step`), nilai dibulatkan **ke atas** pada skala yang disetujui agar target tidak gagal hanya karena rounding ke bawah. Aturan skala belum final (`FDR-Q05`).

### 8.2 Khusus rate nol dan horizon nol

```text
if n > 0 and i == 0:
    required_C = max(0, (T_n - P_g)/n)

if n == 0:
    goal_met_now = (P_g >= T_0)
    required_C is NOT applicable: no remaining contribution periods
```

Saat `n=0` dan belum funded, keluarkan status `DEADLINE_REACHED_UNFUNDED`/shortfall, **bukan** pembagian dengan nol atau angka kontribusi 0 yang terkesan feasible.

### 8.3 Feasibility dan kekurangan kapasitas

Definisikan secara eksplisit `available_capacity_t`, bukan mengambil `income-expense` secara otomatis saat ada kewajiban pokok utang, biaya periodik, kebutuhan buffer, dan atribusi uang ke goal lain. Nilai kapasitas mungkin unknown jika data tidak lengkap.

Untuk *single goal* dengan kontribusi konstan yang disetujui:

```text
capacity_gap = required_C - available_capacity
mathematical_feasible = (available_capacity >= required_C)
```

**Jangan** menyebut "pasti tercapai". Label yang tepat: "feasible **menurut model dan asumsi**". Jika kapasitas tidak diketahui -> `INSUFFICIENT_DATA`, bukan `true`/`false` otomatis. Untuk target banyak dan deadline berbeda, gunakan horizon bersyarat dan *optimizer* R3; tidak cukup hanya membagi satu jumlah total dengan seluruh bulan.

### 8.4 Funded progress masih BLOCKED

`P_g` dalam §8.1 **bukan** otomatis saldo seluruh rekening atau penjumlahan kontribusi yang direncanakan. Product Owner harus menetapkan satu model atribusi dari alternatif pada `FINANCIAL-DOMAIN-RULES.md` (§7.2; **FDR-Q07 / Q-11**). Uang actual tidak boleh dianggap dibelanjakan/dipakai dua kali untuk dua goals ketika diklaim eksklusif.

### 8.5 Contoh numerik

- `T_n=210.000`, `P_g=0`, `n=2`, `i=0.10 per periode`, setoran akhir -> `A=2.1`, `required_C=100.000`.
- `T_n=1.210.000`, `P_g=1.000.000`, `n=2`, `i=0.10 per periode` -> `required_C=0` pada model return konstan tersebut.
- `T_n=1.200.000`, `P_g=0`, `n=12`, `i=0` -> `required_C=100.000`.

**Catatan:** angka return 10% **per periode** pada contoh pertama dan kedua adalah contoh matematika untuk hasil eksak, bukan asumsi tingkat return investasi yang realistis.

---

## 9. Budget & recurring planning (R2; FR-010)

Untuk kategori `c` dan periode kalender yang jelas:

```text
actual_category_spend(c,period)
  = sum(eligible posted expenses classified as c in period)

remaining_budget = planned_budget(c,period) - actual_category_spend(c,period)
```

- Transfer internal tidak masuk `actual_category_spend`, kecuali fee yang dicatat sebagai expense kategori sesuai rule.
- Budget bukan account baru; `remaining_budget` **tidak sama** dengan cash saldo. Nilai negatif boleh berarti over-budget.
- Rule recurring menambah **planned event** hingga diposting mengikuti aturan yang disetujui; tidak boleh menggandakan actual akibat scheduler retry (`DR-013`).
- Split kategori, refund lintas periode, rollover, rekonsiliasi recurring vs posted, dan hari libur **TBD**; tidak ada pembulatan periodik implisit.

---
## 10. Deterministic projection engine (R2; FR-011–FR-014)

### 10.1 State per periode

**PROPOSED model state** (konsep; bukan tabel SQL):

```text
FinancialState(t) {
  cash_accounts,              // actual hanya di t=0; berikutnya hypothetical
  investment_positions,       // jangan duplikasi sebagai other asset
  other_asset_values,
  debt_outstanding,
  earmarked_goal_allocations, // label atas funding existing, bukan uang baru
  assumptions_reference,
}
```

Saldo awal dan valuasi berasal dari **snapshot baseline pada `as_of`** yang jelas (`DR-016`/`DR-020`). Output `t >= 1` adalah forecast. Kehadiran actuals baru setelah snapshot tidak mengubah simulation run lama; tampilkan *stale* atau buat rebase eksplisit dengan `scenario_version` baru.

### 10.2 Kandidat urutan kejadian (belum menjadi policy produksi)

Dalam satu periode waktu yang dimodelkan, contoh urutan **PROPOSED**:

1. Ambil posisi awal periode (hasil akhir periode sebelumnya).
2. Terapkan pendapatan masuk/penurunan gaji dan biaya eksternal pada tanggal efektif yang disepakati.
3. Hitung bunga pinjaman dan pembayaran cicilan menurut kalender kontrak pinjaman.
4. Hitung return/valuasi investasi berdasarkan *timing* investasi, termasuk biaya yang memang dipilih.
5. Proses pembelian/penjualan aset dan life event dengan dampak cash **serta** asset/liability lawannya.
6. Terapkan transfer antar-posisi dan kontribusi goal (hanya *allocation mark* jika tidak ada transfer nyata).
7. Hitung cash shortfall, asset/liability/net worth, dan output akhir periode.

Urutan di atas **tidak netral secara matematika** bila return/perhitungan bunga bergantung kapan arus terjadi. Karena itu, sebelum FR-012 diimplementasikan, `SCENARIO-ENGINE.md` harus menetapkan event ordering final, *partial periods*, rate per tanggal, dan interaksi beberapa event pada hari yang sama. Jangan langsung menggunakan urutan ini sebagai standar tersembunyi.

### 10.3 Konservasi aliran uang dan pemisahan posisi

Misalkan `K_t` adalah total cash-like account yang eligible dalam skenario, `I_t` adalah posisi investasi yang berbeda dari `K_t`, dan `D_t` saldo utang. Salah satu model akhir-periode yang **PROPOSED** untuk mendiskusikan double counting:

```text
K_t = K_(t-1)
      + external_income_t
      - external_consumption_t
      - paid_interest_fees_t
      - principal_repayment_t
      + new_borrowing_received_t
      - net_asset_purchase_cash_t
      + net_asset_sale_cash_t
      - transfer_cash_to_investment_t
      + transfer_investment_to_cash_t
      + other_explicit_cash_events_t

I_t = I_(t-1)*(1+i_t)
      + transfer_cash_to_investment_t
      - transfer_investment_to_cash_t
      - modeled_investment_fees_t

D_t = D_(t-1)
      + new_principal_borrowed_t
      - principal_repayment_t
      + modeled_capitalized_interest_t
```

Untuk menjumlahkan nilai aset, `I_t` tidak boleh sekaligus dihitung lagi di `K_t`; aset nonkas yang dibeli harus muncul hanya satu kali. `new_borrowing_received_t` di cash harus berpasangan dengan `new_principal_borrowed_t` pada liabilities. Pembayaran pokok mengurangi cash dan utang, **bukan** expense konsumsi. Transfer ke investasi mengurangi cash dan menambah investment, bukan mengurangi total net worth. Fee/tax harus dibukukan sekali, tidak diam-diam dimasukkan lagi di return net-of-fees.

**Penting:** contoh persamaan di atas belum lengkap untuk accrual accounting, multi-currency, cash dividend, capital gains realized/unrealized, dan pembelian properti dengan DP+KPR. Modul tambahan harus memperluas event model tanpa melanggar `DR-006`, `DR-009`, `DR-010`, `DR-023`.

### 10.4 Output per periode yang direkomendasikan

| Kolom/output | Makna |
|---|---|
| `period_start`, `period_end` | Batas periode yang konsisten dan auditable |
| `cash_open`, `cash_close` | Posisi kas; dapat negatif bila forecast menunjukkan shortfall, bukan pinjaman nyata |
| `external_income`, `external_expense` | Arus eksternal, tidak menyelundupkan transfer internal |
| `mandatory_debt_payment` | Pokok dan bunga diurai, tidak seluruhnya menjadi expense |
| `investments_value`, `other_assets_value` | Posisi aset yang tidak diduplikasi |
| `outstanding_liability` | Posisi utang, bukan akumulasi pembayaran historis |
| `net_worth` | Aset nonduplicated minus liability pada tanggal tersebut |
| `goal_funding_planned` | Alokasi yang direncanakan; bukan bukti funded actual |
| `cash_shortfall`, `warnings` | Kegagalan asumsi/kekurangan kas dan sebabnya |

Jika hasil kas negatif, jangan menyisipkan *implicit loan* untuk menghilangkan kekurangan dan jangan memotong otomatis menjadi nol. Tampilkan kekurangan menurut model; aturan tindakan alternatif bisa dibahas pada stress test dan Financial GPS.

### 10.5 Peristiwa hidup (FR-011)

Setiap event minimal mengungkap tipe (`one_off`, `recurring`, `rate_change`, atau tipe yang disahkan), tanggal efektif, jumlah, currency, arus atau perubahan posisi yang dipicu, dan scenario owner. Contoh pembelian rumah hipotetis dapat memiliki kas DP, biaya transaksi, aset properti yang muncul, pokok KPR baru, dan angsuran berikutnya; tidak boleh hanya mengurangi cash tanpa merepresentasikan asset/liability bila output mengklaim net worth sesudah transaksi.

Event yang belum mempunyai parameter cukup -> `INSUFFICIENT_DATA` atau *blocked partial calculation*. Tidak menciptakan transaksi actual.

---

## 11. Future cash-flow forecasting (R2; FR-014)

### 11.1 Dua jenis cash flow yang tidak boleh dicampur

1. **External operating cash flow:** pendapatan eksternal dikurangi konsumsi/biaya eksternal yang benar-benar dimodelkan; tidak memasukkan transfer internal.
2. **Cash account movement:** perubahan cash-like accounts, dapat dipengaruhi transfer ke investasi, pembelian aset, pencairan pinjaman, dan repayment pokok meskipun peristiwa tersebut tidak sama dengan income/expense konsumsi.

**PROPOSED metrics**:

```text
external_net_flow_t = external_income_t - external_expense_t
cash_account_delta_t = cash_close_t - cash_open_t

# delta cash tidak harus sama dengan external_net_flow
# sebab ada transfer internal antarkelas asset, pembelian asset, pembiayaan, dll.
```

Jika produk ingin menampilkan *free cash flow available for goals*, harus ada definisi tersendiri untuk pajak, kebutuhan hidup minimum, cicilan pokok, fixed buffer, pendapatan tidak berulang, dan saldo account yang tidak dapat dipakai. **Tidak boleh otomatis `income-expense`** lalu mengklaim angka itu selalu aman dialokasikan.

### 11.2 Definisi shortfall

**PROPOSED**: `cash_shortfall_t = max(0, -cash_close_t)` untuk model yang membolehkan kas forecast negatif sebagai sinyal kebutuhan pendanaan. Dalam model dengan saldo minimum `K_min`, definisi alternatif `max(0, K_min-cash_close_t)`. Pemilihan `K_min` dan apakah shortfall dideteksi sebelum/akhir pembayaran adalah **TBD Q-05/Q-06**. Output harus menyebut threshold yang dipakai.

### 11.3 Contoh arus sederhana

Cash awal Rp4 juta; pendapatan eksternal Rp6 juta; konsumsi Rp3 juta; bunga pinjaman Rp100 ribu; pokok cicilan Rp900 ribu; transfer ke account investasi Rp1 juta -> cash akhir **Rp5 juta**:

```text
4.000.000 + 6.000.000 - 3.000.000
          -   100.000 - 900.000 - 1.000.000
          = 5.000.000
```

Transfer Rp1 juta menambah posisi investasi Rp1 juta **jika tidak terkena return/biaya pada peristiwa yang sama**; principal Rp900 ribu mengurangi utang sebesar Rp900 ribu. Total expense konsumsi/biaya dalam contoh adalah Rp3,1 juta (belanja Rp3 juta + bunga Rp100 ribu), **bukan** Rp5 juta. Ini ilustrasi conservation rule; dalam test production perlu menambahkan semua kategori dan timing yang disetujui.

---

## 12. Loan amortization / KPR ilustratif (R2/R3; Q-05 masih terbuka)

### 12.1 Anuitas cicilan tetap (fixed-rate, pembayaran akhir periode)

Untuk pokok awal `L > 0`, bunga periodik `i >= 0` sesuai **jenis rate yang jelas**, `n > 0` kali cicilan, dengan pembayaran tiap akhir periode:

```text
PMT = L * i / (1 - (1+i)^(-n))     if i != 0
PMT = L / n                        if i == 0

interest_t = outstanding_(t-1) * i
principal_t = PMT - interest_t
outstanding_t = outstanding_(t-1) - principal_t
```

Dalam implementasi jadwal terkuantisasi, pembayaran terakhir dapat disesuaikan agar outstanding menjadi nol **berdasarkan policy rounding yang disetujui**, bukan menimbulkan pokok negatif kecil atau overpayment diam-diam.

**Jangan samakan** simulasi ini dengan semua KPR nyata. Fixed-rate lalu floating-rate, bunga efektif vs flat, grace period, provisi, asuransi, pajak, denda, pelunasan dini, penjadwalan ulang dan perubahan tenor memerlukan model terpisah serta data kebijakan/kredit aktual. Simulasi ini **tidak** menyatakan besaran angsuran resmi suatu bank.

### 12.2 Loan flow dan net worth

- Pencairan `L` -> cash naik `L`, liability naik `L`, NW tidak berubah tanpa fee.
- Pembayaran cicilan -> kas turun sejumlah `PMT`; liability turun `principal_t`; bunga `interest_t` adalah beban yang menurunkan NW.
- Jumlah pokok yang telah dilunasi tidak boleh dikategorikan sebagai expense konsumsi agar tidak terjadi penurunan NW ganda.
- Jika angsuran terdiri atas pokok+bunga+asuransi+biaya lain, pisahkan setiap komponen dan jangan menambahnya lagi ke `PMT` yang ternyata sudah memasukkan biaya.

### 12.3 Reference arithmetic

`L=210.000`, `i=0.10 per periode`, `n=2`, fixed-rate annuity -> `PMT=121.000`:

| Periode | Saldo awal | Bunga 10% | Pokok dilunasi | Pembayaran | Sisa pokok |
|---|---:|---:|---:|---:|---:|
| 1 | 210.000 | 21.000 | 100.000 | 121.000 | 110.000 |
| 2 | 110.000 | 11.000 | 110.000 | 121.000 | 0 |

Rate 10% **per periode** dipilih untuk contoh hitung eksak, **bukan** estimasi bunga KPR tahunan di Indonesia. Jika tidak ada rincian suku bunga dan pengenaan biaya, tampilkan keterbatasan sebagai placeholder, bukan hasil "angsuran KPR pasti".

### 12.4 Edge cases

- `n=0`, `L<0`, `i` tidak kompatibel, extra payment tanpa policy, dan jadwal tidak konsisten -> invalid/blocked.
- `i=0` -> bagi rata secara aritmetis dan jadwal pembayaran terakhir mengikuti presisi.
- `PMT <= interest_t` pada jadwal amortisasi positif -> tidak melunasi pokok; harus disinyalkan, bukan mengklaim lunas.
- Negative-rate loan, *flat interest*, floating reset, penalty, capitalization -> **BLOCKED** sampai model disetujui.

---

## 13. Financial stress test (R2; FR-015)

### 13.1 Perbandingan baseline dengan shock yang eksplisit

Skenario stress adalah baseline/skenario yang disalin lalu **di-override** oleh shock terparameterisasi (misalnya `income_reduction=100%` selama `k` periode, biaya darurat, atau penundaan pemasukan). Bukan perubahan saldo aktual.

```text
stress_result = Project(baseline_snapshot, baseline_assumptions + explicit_shocks)
base_result   = Project(baseline_snapshot, baseline_assumptions)
delta_t       = stress_result.metric_t - base_result.metric_t
```

Untuk perbandingan yang adil, gunakan versi baseline yang sama; hanya shock dan parameter yang dijelaskan yang boleh berbeda. Hasil harus menyebut shock duration, start date, currency, serta asumsi biaya/penghasilan lain.

### 13.2 Runway kas dan survival of obligations

**PROPOSED** definisi metrik utama:

- `first_shortfall_period`: periode pertama saldo kas forecast turun di bawah threshold kas minimum yang ditentukan.
- `fully_funded_periods`: banyaknya periode penuh **sebelum** periode shortfall pertama.
- `shortfall_amount`: kekurangan pada periode tersebut terhadap threshold.
- `goal_delay`: perubahan tanggal tercapainya target jika model goal mempunyai pendanaan yang disetujui.

Threshold (nol vs dana darurat minimum), waktu perhitungan (awal/tengah/akhir bulan), dan perlakuan aset likuid lain **TBD**. Jangan menganggap *cash runway 6 bulan* jika cash sudah habis pada tengah bulan keenam tanpa model intrabulanan yang menyatakannya.

### 13.3 Guardrails

Tidak boleh menampilkan label "aman" atau "pasti gagal" secara absolut. Jelaskan bahwa stress test adalah hasil dari shock dan data yang dipilih. Jika data debt obligations/living costs tidak tersedia, status `INSUFFICIENT_DATA` dapat lebih tepat daripada menghitung runway yang tampak presisi tetapi salah.

---

## 14. Scenario comparison, versioning & staleness (R2; FR-012/013)

### 14.1 Aturan reproduksi

Setiap simulation run perlu dapat ditelusuri ke:

```text
run identity = baseline snapshot/fingerprint
             + scenario_version
             + normalized input parameter set
             + model_version
             + engine_version
             + RNG details (only for stochastic runs)
```

Snapshot ini **bukan** data actual yang selalu berubah mengikuti dashboard. Jika actual diperbaiki, run lama tetap menjadi hasil terhadap baseline lama dan diberi `stale=true` relatif terhadap kondisi sekarang jika relevan (`DR-020`). Mengubah skenario menghasilkan versi baru, tidak overwrite asumsi run lama tanpa jejak.

### 14.2 Perbandingan yang sah

- Bandingkan skenario dengan metrik dan satuan yang sama: horizon, currency, nominal/real, tanggal dasar, dan konvensi return harus kompatibel.
- Perbedaan `delta = scenario_B - scenario_A` hanya bermakna jika field/parameter yang relevan disamakan atau perbedaannya dijelaskan eksplisit.
- Simulasi stochastic tidak boleh dibandingkan hanya berdasarkan satu lintasan acak. Metode statistik pembanding dan interval ketidakpastian masih memerlukan spesifikasi Monte Carlo.

### 14.3 Guard no mutation

`CreateScenario`, `RunSimulation`, `CompareSimulation` **tidak** membuat actual transaction, actual account balance, atau actual liability posting. Menerapkan suatu rencana ke actual **tidak** termasuk dalam operasi ini dan membutuhkan workflow domain eksplisit yang belum didefinisikan.

---

## 15. Multi-goal allocation optimizer (R3; FR-017; BLOCKED untuk implementasi)

### 15.1 Formulasi kendala dasar (referensi model, belum final)

Untuk goals `g = 1..G`, periode `t = 1..H`, kapasitas kontribusi `S_t` yang **telah didefinisikan dengan benar**, dan alokasi `x_(g,t)`:

```text
x_(g,t) >= 0
sum_over_goals(x_(g,t)) <= S_t        for each t
```

Setiap goal `g` memiliki target, deadline `d_g`, saldo awal eligible `P_g`, dan aturan pertumbuhan masing-masing. Jika model menetapkan return `i_(g,t)` dan kontribusi akhir-periode, kandidat recurrence:

```text
G_(g,0) = P_g
G_(g,t) = G_(g,t-1)*(1+i_(g,t)) + x_(g,t)
```

Kendala tujuan dapat menyatakan `G_(g,d_g) >= target_g`, **jika** seluruh goal wajib dipenuhi. Jika tidak feasible, jangan membuang goal yang gagal dari hasil: laporkan shortfall, goal yang terdampak, serta opsi alternatif. `P_g` harus mengikuti model funded yang disetujui (`FDR-Q07`), bukan saldo rekening yang dipakai untuk banyak goal sekaligus.

### 15.2 Objective function dan trade-off masih TBD

Tidak ada satu objective yang selalu benar untuk semua pengguna. Opsi *PROPOSED* untuk dibahas:

1. Meminimalkan total kekurangan target yang dibobot prioritas.
2. Memaksimalkan jumlah goal yang memenuhi deadline, dengan aturan prioritas saat konflik.
3. Meminimalkan keterlambatan, dengan prioritas minimum bagi dana darurat.
4. Mengutamakan kebutuhan wajib sebelum goal opsional, dengan constraint keras pada biaya hidup.

Tie-breaking, prioritas yang berubah, positive/negative returns, penggunaan surplus, saldo terikat, constraint minimum likuiditas, dan perilaku infeasible adalah **BLOCKED Q-07** sampai `GOAL-OPTIMIZER.md` disetujui.

### 15.3 Syarat sanity check tanpa return

Jika tidak ada return dan seluruh target harus didanai sampai deadline, kontribusi kumulatif hingga setiap deadline harus cukup untuk target yang telah jatuh tempo (setelah memperhitungkan saldo awal eligible dan funding yang tidak diduplikasi). Contoh: target A Rp12 juta 12 bulan, B Rp24 juta 24 bulan, C Rp120 juta 60 bulan, semua P=0 dan return nol, maka dengan kontribusi tetap `S` perlu:

```text
S >= max(12 juta/12, (12+24) juta/24, (12+24+120) juta/60)
S >= max(1 juta, 1.5 juta, 2.6 juta)
S >= 2.6 juta per bulan
```

Ini **hanya lower bound untuk contoh** pada model cash contribution konstan dan seluruh uang dialokasikan tanpa investasi/perubahan kapasitas. Jika `S=2 juta`, seluruh target tidak feasible sesuai deadline. Hasil ini belum merupakan algoritma optimizer R3 yang sudah disetujui.

---

## 16. Monte Carlo risk simulation (R3; FR-018; BLOCKED untuk model probabilistik)

### 16.1 Apa yang boleh disimpulkan dan tidak

Monte Carlo menghasilkan sekumpulan lintasan **berdasarkan asumsi sampling yang dipilih**. Proporsi lintasan yang memenuhi kondisi sukses disebut `model_success_rate`, **bukan** probabilitas pasti di dunia nyata.

```text
model_success_rate = count(successful_runs) / count(valid_runs)
```

Untuk `N` simulasi valid, misalnya 720 dari 1.000 memenuhi target, keluaran **72% dalam model ini**, bukan "72% peluang pasti". Jika ada lintasan invalid, harus dijelaskan apakah dikeluarkan, dihitung gagal, atau run keseluruhan ditolak. Kebijakan itu **TBD Q-08**.

### 16.2 Pipeline konseptual

```text
validated baseline + assumptions
    -> approved return/inflation/dependency model
    -> reproducible random generator + seed
    -> generate paths
    -> apply same cash-flow / life-event engine per path
    -> evaluate success condition on each path
    -> aggregate metrics and percentiles
    -> explain assumptions, limitations, failures
```

**BLOCKED:** tipe distribusi return, volatilitas dan horizon kalibrasi, korelasi antar-aset, model inflasi stochastic, regime shifts, fat tails, bentuk distribusi penarikan, *sequence-of-returns risk*, jumlah lintasan, RNG algorithm/version, sampling strategy, stopping criteria, dan toleransi lintas platform. Semua ini harus ditetapkan di `MONTE-CARLO.md` serta direview sebelum production (`Q-08`, `FDR-Q12`).

### 16.3 Metrik output dan validitas

| Metric | Kontrak konseptual |
|---|---|
| `n_requested`, `n_valid`, `n_failed` | Transparansi banyak lintasan yang diproses |
| `model_success_rate` | Proporsi lintasan valid yang memenuhi definisi sukses yang didokumentasikan |
| `percentile_p10/p50/p90` | Kuantil saldo/outcome akhir menurut metode percentile yang nantinya dipilih |
| `first_depletion_period` | Waktu pertama dana tidak sanggup menutup kewajiban, **bukan** sekadar saldo investasi nol jika terdapat cash lain |
| `confidence/uncertainty note` | Akui ketidakpastian Monte Carlo dan risiko salah spesifikasi model |

Metode perhitungan kuantil (nearest-rank, interpolasi, dsb.) **TBD**. Jangan mengklaim satu angka P90 tanpa menyebut apa yang diukur (*nominal wealth*, *real wealth*, *minimum wealth*, dsb.) dan kapan.

### 16.4 Validasi minimal sebelum rilis

- Input valid/invalid termasuk variasi volatilitas, batas return, inflation rate, dan matriks korelasi (symmetric dan positive semidefinite bila model mengharuskannya).
- RNG reproducibility dengan seed **dan algoritma/versi yang sama**; parallel workers harus menghindari nondeterministic path assignment jika hasil identik dijanjikan.
- Path outcomes tidak menciptakan uang dari goal planned; kerugian dapat terjadi dan output negatif ditangani menurut model.
- Unit/statistical tests untuk sampler, aggregate percentiles, dan failure semantics.
- Model review: perhatikan data historis tidak selalu mewakili masa depan; distribusi normal return sederhana tidak otomatis benar.
- Resource guard: CPU/memory/time limit, cancel/retry status, hasil parsial tidak dipromosikan menjadi sukses, dan tidak ada penulisan actual (`EN-005`).

---

## 17. Retirement / FIRE planner (R3; FR-019; policy BLOCKED)

### 17.1 Dua fase yang dibedakan

1. **Accumulation:** pendapatan, pengeluaran, kontribusi, return, kenaikan biaya, kewajiban, dan peristiwa hidup hingga rencana berhenti bekerja.
2. **Decumulation:** pendapatan non-kerja (jika ada), pengeluaran, biaya, inflasi, pajak, dan penarikan dana selama horizon pensiun.

`retirement_age` tidak sama dengan `financial_independence_age`. Model wajib membedakan kebutuhan kas dari investasi vs total kekayaan bersih; aset rumah tidak otomatis menjadi kas yang dapat digunakan membayar biaya hidup.

### 17.2 Kontrak penarikan dana

**PROPOSED recurrence** untuk portofolio yang mendanai pengeluaran dengan penarikan akhir periode dan tanpa fee:

```text
withdrawal_needed_t = max(0, spending_t - other_income_t)
portfolio_t = portfolio_(t-1)*(1+i_t) - withdrawal_needed_t
```

Jika kebijakan mengharuskan penarikan di **awal** periode, recurrence berbeda: `(portfolio_(t-1) - withdrawal_needed_t)*(1+i_t)`. Jika saldo sebelum penarikan tidak cukup, tandai `DEPLETION/SHORTFALL`, **jangan** otomatis membuka utang, menjepit ke nol tanpa menyimpan besar gap, atau menjamin keberhasilan.

Pengeluaran yang meningkat akibat inflasi harus mengikuti basis nominal/riil yang jelas. `spending_t` bisa termasuk pajak, perlindungan, dan pengeluaran tak rutin sesuai asumsi pengguna; bila tidak ada data, tampilkan *incomplete*.

### 17.3 Guardrails khusus pensiun

- Jangan mengklaim "aturan 4%" sebagai persentase penarikan yang aman secara universal, apalagi untuk semua portofolio di Indonesia.
- Tidak boleh menafsirkan `net_worth` yang mengandung rumah/asset tak likuid sebagai saldo dana pensiun yang langsung dapat ditarik.
- Keberhasilan pensiun dipandang sebagai *model outcome* di bawah definisi kebutuhan dan horizon yang diberikan, bukan janji.
- Model return, biaya, umur rencana, sumber pendapatan pensiun, withdrawal policy, dan pemilihan angka default adalah **BLOCKED Q-05/Q-08** sampai desain yang disetujui tersedia.

---

## 18. Financial insight & GPS (R2/R3; FR-016/020)

Insight **tidak membuat angka baru melalui heuristik tersembunyi**. Insight menginterpretasikan nilai yang sudah dihitung oleh engine tervalidasi.

**PROPOSED** kontrak sebuah insight:

```text
Insight {
  rule_id, rule_version,
  source_run_ids, source_metric_keys,
  observed_values, units, threshold_or_constraint,
  explanation, limitations,
  candidate_actions[],
  simulation_links[],
}
```

Contoh: saat `required_monthly_contribution > available_capacity`, sistem boleh menjelaskan gap dan membuat *candidate scenario* untuk menunda target atau meningkatkan kontribusi, tetapi **tidak** boleh menyunting target actual/goal tanpa tindakan pengguna yang eksplisit dan prosedur bisnis terpisah.

- Setiap saran harus merujuk sumber angka, parameter, dan trade-off.
- Data tidak lengkap -> `INSUFFICIENT_DATA` dan daftar input yang diperlukan.
- Penjelasan `model_success_rate` harus menegaskan angka tersebut bergantung pada model.
- AI generatif opsional (bukan requirement) tidak boleh menjadi sumber formula, mem-posting transaksi, atau memberi rekomendasi produk investasi tertentu seolah tervalidasi.

---

## 19. Error taxonomy dan perilaku yang tidak boleh ditebak

Kontrak error di bawah **konseptual**; nama final API code dan HTTP status ditetapkan pada `API-GUIDELINES.md`/OpenAPI.

| Code konseptual | Pemicu | Tindakan aman |
|---|---|---|
| `INVALID_AMOUNT` | Nilai/arah nominal melanggar domain | Tolak, sebut field yang perlu diperbaiki |
| `CURRENCY_MISMATCH` | Dua Money tak punya currency yang kompatibel | Tolak tanpa FX implisit |
| `RATE_KIND_AMBIGUOUS` | Rate tanpa effective/nominal/frequency | Meminta penjelasan rate |
| `INVALID_RATE` | Rate di luar domain formula | Tolak atau tandai model membutuhkan approval |
| `INVALID_HORIZON` | `n < 0`, horizon rusak, tanggal ambigu | Tolak, jangan mengubah tanggal diam-diam |
| `UNAPPROVED_CONVENTION` | Timing/rounding/model belum disepakati | `BLOCKED`, bukan default sukses |
| `INSUFFICIENT_DATA` | Kapasitas goal/retirement tidak bisa ditentukan | Keluarkan missing-fields, jangan buat angka |
| `GOAL_DEADLINE_REACHED` | Tidak ada periode kontribusi tersisa | Tampilkan shortfall dan status khusus |
| `INFEASIBLE_PLAN` | Constraints tidak memungkinkan seluruh goal | Tampilkan goal terdampak dan shortfall |
| `MODEL_NOT_APPROVED` | MC/optimizer/retirement model belum tervalidasi | Tidak mempublikasikan hasil sebagai fitur final |
| `NUMERIC_OVERFLOW` | Nilai di luar representasi/toleransi yang disetujui | Tolak dengan error yang tidak membocorkan data |
| `STALE_BASELINE` | Actual berubah sejak baseline yang dipakai | Tawarkan refresh/rebase eksplisit |
| `ENGINE_FAILURE` | Kesalahan runtime/timeouts/worker | Status gagal/cancel, bukan hasil parsial sukses |
| `UNAUTHORIZED_RESOURCE` | Ownership bukan milik pengguna terverifikasi | Tolak akses tanpa membocorkan isi resource |

Semua penjelasan error untuk pengguna **tidak** boleh mengandung detail keuangan milik user lain, token, query mentah yang sensitif, atau log verbose. Error akibat data tidak lengkap tidak boleh otomatis dikonversi menjadi nominal nol.

---

## 20. Contoh angka lintas fungsi yang harus masuk golden test berikutnya

Nilai ini adalah **reference arithmetic yang dicek secara independen**; kebijakan pembulatan/timing production tetap TBD. `FINANCIAL-TEST-CASES.md` akan menambahkan schema input, expected status, parameter, toleransi, sumber penghitungan, dan test harness Go.

| Ref | Kasus input ringkas | Expected yang tepat | Rilis/fitur |
|---|---|---|---|
| `CALC-REF-01` | Opening Rp10 juta + income Rp2 juta | Saldo Rp12 juta; income periode Rp2 juta, bukan Rp12 juta | R1 / FR-003–006 |
| `CALC-REF-02` | A Rp5 juta; B Rp2 juta; transfer Rp1 juta tanpa fee | A=Rp4 juta, B=Rp3 juta; total tetap Rp7 juta; expense=0 | R1 / FR-004 |
| `CALC-REF-03` | Account Rp6 juta; asset lain Rp5 juta; liability Rp3 juta | Net worth Rp8 juta | R1 / FR-006 |
| `CALC-REF-04` | `P=1.000.000,C=100.000,n=12,i=0`, akhir | FV Rp2.200.000 | R1 / FR-008 |
| `CALC-REF-05` | `P=0,C=100.000,n=2,i=.01`, akhir | FV Rp201.000 | R1 / FR-008 |
| `CALC-REF-06` | Sama, setoran awal | FV Rp203.010 | R1 / FR-008 |
| `CALC-REF-07` | `P=1.000.000,C=0,n=2,i=.01` | FV Rp1.020.100 | R1 / FR-008 |
| `CALC-REF-08` | `T=210.000,P_g=0,n=2,i=.10`, akhir | Kontribusi perlu Rp100.000/periode | R1 / FR-009 |
| `CALC-REF-09` | `T=1.200.000,P_g=0,n=12,i=0`, akhir | Kontribusi perlu Rp100.000/periode | R1 / FR-009 |
| `CALC-REF-10` | Inflasi efektif tahunan 10%, harga awal Rp1 juta, 2 tahun | Nominal target Rp1.210.000 | R2 / FR-012 |
| `CALC-REF-11` | Kas 4m + gaji 6m - konsumsi 3m - bunga .1m - pokok .9m - transfer investasi 1m | Kas akhir Rp5 juta; interest expense Rp100 ribu; pokok bukan expense | R2 / FR-014 |
| `CALC-REF-12` | Loan `L=210.000,i=.10,n=2`, cicilan akhir tetap | PMT Rp121.000; pokok sisa 110.000 lalu 0 | R2/R3 / loan |
| `CALC-REF-13` | Goals A12m/12, B24m/24, C120m/60; P=0, return nol | Batas kapasitas konstan minimal Rp2,6 juta/bulan | R3 / FR-017 |
| `CALC-REF-14` | Simulasi risiko 720 sukses dari 1.000 valid | Model success rate 0,72 (72%), bukan probabilitas dunia nyata | R3 / FR-018 |
| `CALC-REF-15` | Loan principal diterima Rp10 juta, cash +10m, debt +10m | Net worth delta 0 (tanpa fee) | R1/R2 / FR-005 |
| `CALC-REF-16` | `n=0`, goal belum funded | Status deadline reached + shortfall; bukan pembagian nol | R1 / FR-009 |

**Catatan penting:** pada `CALC-REF-13`, kebutuhan cumulatif mengasumsikan uang dapat didistribusikan sesuai deadline dan kemampuan menabung konstan, **tanpa return maupun biaya**; jangan copy hasil Rp2,6 juta sebagai general optimizer output.

### 20.1 Property/invariant tests di luar contoh angka

1. Internal transfer tanpa fee menjaga total aset dan net worth.
2. Pada `i=0`, setoran awal = setoran akhir untuk horizon yang sama.
3. Dengan return non-negatif dan setoran non-negatif, FV tidak lebih kecil dari modal plus setoran pada model fixed rate/no-fee.
4. Dengan return negatif, estimated growth bisa negatif tanpa mengubah ledger aktual.
5. Net worth menghitung setiap asset satu kali, membolehkan net worth negatif.
6. Pembayaran pokok saja tidak mengubah NW; bunga/fee yang benar-benar dibayar menurunkannya.
7. Baseline yang sama + input/engine version yang sama memberikan deterministic projection sama.
8. Menambah scenario/version tidak mengubah actual; perubahan actual setelah run dapat menyebabkan hasil lama stale.
9. Required contribution pada horizon nol tidak memakai pembagian nol.
10. Multi-goal planned contributions tidak melebihi kapasitas yang disetujui dan tidak membelanjakan funded asset yang sama dua kali.
11. Cross-user dan resource ownership gagal sebelum menerima hasil private.
12. RNG yang sama (seed, versi algoritma, path assignment) menghasilkan statistik yang sama **setelah** MC model disetujui.

**Batasan property #3:** hanya untuk `i>=0`, fixed-rate, no-fee/no-withdrawal dan tidak ada batas/revaluasi lain. Jangan mengaplikasikan property ini pada semua jenis investasi.

---

## 21. Decision register khusus perhitungan — PENDING OWNER APPROVAL

**Tidak ada baris berikut yang otomatis approved.** ID berikut adalah penajaman blocker induk yang sudah ada; jangan memperbanyak keputusan yang saling bertentangan di file berbeda.

| ID | Blocker induk | Pertanyaan/keputusan | Akibat jika belum disetujui |
|---|---|---|---|
| `CALC-Q01` | Q-04; FDR-Q04 | Money storage/API: IDR-only integer unit atau fixed decimal? | Saldo/kontrak nilai actual belum final |
| `CALC-Q02` | Q-04; FDR-Q05 | Skala intermediate dan aturan rounding financial Money/PMT/setoran minimum? | Hasil yang dibulatkan belum production-ready |
| `CALC-Q03` | Q-04; FDR-Q06 | Tanggal awal, end-of-month, tahun kabisat, partial-period, timezone? | Horizon berbasis tanggal belum final |
| `CALC-Q04` | Q-04; FDR-Q09 | Setoran awal/akhir dan frekuensi return yang tampil di UI? | FR-008/009 blocked untuk release |
| `CALC-Q05` | Q-04/Q-05; FDR-Q09 | Annual effective vs nominal as UI input; negative return domain dan batas? | Rate normalization belum production-ready |
| `CALC-Q06` | Q-05; FDR-Q08 | Inflasi nominal vs real, target harga sekarang/masa depan, default model? | Inflation-adjusted goals blocked |
| `CALC-Q07` | Q-05; FDR-Q08 | Bagaimana biaya, pajak, bunga, loan amortization, dan life events ditiming? | Debt/projection tidak boleh klaim sudah menyertakan komponen itu |
| `CALC-Q08` | Q-11; FDR-Q07 | Funded goal berdasarkan envelope/account/contribution event? | Feasibility yang bergantung saldo goal blocked |
| `CALC-Q09` | Q-03; FDR-Q01–Q03 | Opening posting, overdraft, reversal/backdated, kategori utang? | Actual balance calculation blocked |
| `CALC-Q10` | Q-06; FDR-Q10 | Period ordering, baseline snapshot/rebase/version/stale policy? | FR-012/013 blocked |
| `CALC-Q11` | Q-05/Q-06 | Available capacity dan cash buffer, shortfall threshold? | Klaim affordability/stress-runway blocked |
| `CALC-Q12` | Q-07; FDR-Q11 | Goal optimizer objective, constraints, priority/tie-break? | FR-017 blocked |
| `CALC-Q13` | Q-08; FDR-Q12 | MC distributions, parameters, correlations, RNG, percentiles, success semantics? | FR-018 blocked |
| `CALC-Q14` | Q-05/Q-08; FDR-Q08/Q12 | FIRE spending, retirement horizon, withdrawal policy, liquidity rules? | FR-019 blocked |
| `CALC-Q15` | Q-10; NFR-002/003 | Numeric tolerances, performance limits, reproducibility across architectures? | Advanced model release blocked |

### 21.1 Urutan approval yang disarankan

1. Putuskan **CALC-Q01–Q05** sebelum memfinalkan Compound Calculator (FR-008).
2. Putuskan **CALC-Q08–Q09** untuk menghitung funded goal yang benar dan account balances (FR-003–FR-009).
3. Putuskan **CALC-Q06–Q07, Q10–Q11** sebelum deterministic scenario projection dan stress test (R2).
4. Putuskan **CALC-Q12–Q15** bersama spesifikasi R3 yang khusus, jangan diasumsikan tuntas sekarang.

Jika belum ada approval: agent boleh menulis proof-of-concept terisolasi di test/sandbox dengan parameter **eksplisit** dan label `EXPERIMENTAL`, tetapi tidak menyatakan fitur siap production.

---

## 22. Traceability requirement -> formula -> test

| ID kontrak | Rumus/invariant | FR/NFR/DR terkait | Referensi test awal | Status gate |
|---|---|---|---|---|
| `CS-001` | Money precision, currency matching, no NaN | NFR-002; DR-004/014 | precision/currency cases TBD | **TBD implementation policy** |
| `CS-002` | Rate kind + conversion | FR-008; NFR-002 | annual effective vs nominal cases | **PROPOSED** |
| `CS-003` | Saldo opening/posting dan transfer | FR-003/004; DR-005–008 | CALC-REF-01/02 | **TBD posting lifecycle** |
| `CS-004` | NW dan nonduplication | FR-005/006; DR-009/010/023 | CALC-REF-03/15 | **REQUIRED invariant** |
| `CS-005` | Future value fixed rate | FR-008; EN-004 | CALC-REF-04–07 | **Math defined, timing TBD** |
| `CS-006` | Inflation nominal vs real | FR-009/012/019; DR-014 | CALC-REF-10 | **Math defined, policy TBD** |
| `CS-007` | Goal required contribution | FR-007/009; DR-011/017 | CALC-REF-08/09/16 | **Funded model TBD** |
| `CS-008` | Budget/recurring categorization | FR-010; DR-013 | period/refund tests TBD | **R2** |
| `CS-009` | Deterministic state projection | FR-011/012/014; DR-001/016/020 | CALC-REF-11; invariants | **R2 scenario policy TBD** |
| `CS-010` | Loan amortization | FR-012/015/019; DR-023 | CALC-REF-12 | **Model policy TBD** |
| `CS-011` | Stress test and runway | FR-015; DR-015/017 | shock comparison TBD | **Threshold TBD** |
| `CS-012` | Scenario reproducibility | FR-013; NFR-003/010 | same-snapshot replay | **R2 snapshot design TBD** |
| `CS-013` | Multi-goal constrained allocations | FR-017; DR-011/024 | CALC-REF-13, infeasible | **R3 BLOCKED** |
| `CS-014` | MC success/quantiles | FR-018; DR-015/016/024 | CALC-REF-14, seeded tests | **R3 BLOCKED** |
| `CS-015` | Retirement accumulation/withdrawal | FR-019; DR-015/024 | depletion tests TBD | **R3 BLOCKED** |
| `CS-016` | Financial GPS explanations | FR-016/020; DR-019 | explanation/insufficient tests | **R2/R3** |

**Persyaratan lintas modul:** ownership FR-001 dan NFR-001, logging sensitif DR-021, serta export/delete FR-021 tidak digantikan oleh golden numeric tests. Semua operasi privat masih memerlukan security/integration tests.

---

## 23. Quality gates sebelum engine Go boleh dinyatakan selesai

### 23.1 Checklist algoritma dan implementasi

- [ ] Setiap fungsi numerik yang diimplementasikan memiliki FR/`CS-xxx`, assumption list, unit input/output, dan error conditions.
- [ ] Tidak ada formula production yang mengandalkan policy `TBD` tanpa keputusan tertulis.
- [ ] Input dan return model diekspresikan secara eksplisit: *effective vs nominal*, frekuensi, dan setoran awal/akhir.
- [ ] Validasi domain untuk rate, `n=0`, `i=0`, nilai negatif yang relevan, overflow, currency mismatch, dan precision dilakukan.
- [ ] Money actual/forecast tidak tercampur pada model dan storage.
- [ ] Test vectors dari `FINANCIAL-TEST-CASES.md` telah diperiksa independen, diimplementasikan dalam Go tests, dan benar-benar dieksekusi.
- [ ] Penghitungan actual opening/transfer/debt serta net worth menjaga conservation invariants.
- [ ] UI/API menampilkan label `actual` vs `forecast`, periode, currency, dan ringkasan asumsi.
- [ ] Tidak ada log/snapshot model yang mengekspos data sensitif atau membuat ownership bypass.
- [ ] Source baseline, scenario version, model version, engine version (dan seed bila perlu) dapat diperiksa ulang.
- [ ] Kegagalan worker/job tidak mengubah actual dan tidak menampilkan hasil parsial sebagai sukses.
- [ ] Hasil benchmark/performance dan toleransi numerik didokumentasikan ketika fitur tersebut dirilis.

### 23.2 Definisi selesai yang jujur

Menyelesaikan penulisan `CALCULATION-SPECS.md` **bukan** berarti rumus produksi, SQL, API, Flutter UI, Monte Carlo model, atau test suite sudah dibuat. Status dokumen tetap `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` sampai Product Owner memutuskan blocker relevan. `FINANCIAL-TEST-CASES.md` berikutnya akan menyediakan contoh input-output lengkap dan bukti cek untuk fungsi yang memang sudah dispesifikasikan.

---

## 24. Review internal dokumen (v0.1.0)

| Pemeriksaan | Hasil |
|---|---|
| FR-003–FR-009/R1 terhubung dengan saldo, net worth, compound, dan goal feasibility | Checked |
| FR-010–FR-016/R2 tidak disamakan dengan fitur R1 | Checked |
| FR-017–FR-020/R3 ditandai membutuhkan model lanjutan | Checked |
| R4 portfolio/shared goals tidak diam-diam masuk scope | Checked |
| Actual vs forecast, planned contributions, transfer, dan nonduplication dijaga | Checked |
| Konversi rate tahunan efektif vs nominal, i=0, n=0, setoran awal/akhir dijelaskan | Checked |
| Contoh numerik dan loan payment divalidasi secara terpisah dari penyusunan teks | Checked — lihat skrip QA lokal pembuatan artefak; bukan pengganti golden test Go |
| Seluruh keputusan yang masih TBD ditandai dengan ID yang dapat ditelusuri | Checked |
| Model Monte Carlo dan objective optimizer **tidak** diklaim approved | Checked |
| Product Owner menyetujui konvensi R1 dan scope dokumen | **PENDING** |

### 24.1 Referensi pembanding teknis

Referensi di bawah mendukung prinsip matematika/keuangan yang digunakan, **tidak** mengesahkan default produk Esokira:

- Microsoft Support — [FV function](https://support.microsoft.com/en-us/excel/functions/fv-function): perilaku anuitas, rate per periode, setoran awal/akhir.
- Microsoft Support — [PV function](https://support.microsoft.com/en-us/excel/functions/pv-function): pembayaran berulang dan konsistensi unit waktu.
- Consumer Financial Protection Bureau — [How do mortgage lenders calculate monthly payments?](https://www.consumerfinance.gov/ask-cfpb/how-do-mortgage-lenders-calculate-monthly-payments-en-1965/): pembayaran pokok+bunga fixed rate tidak selalu mencakup biaya pajak/asuransi.
- PostgreSQL — [Numeric Types](https://www.postgresql.org/docs/current/datatype-numeric.html): `numeric` untuk ketelitian representasi nilai desimal; detail desain belum final.
- Dokumen internal Esokira v0.1.0 yang tercantum pada header.

## 25. Handoff ke dokumen selanjutnya

**Dokumen berikutnya: `docs/financial/FINANCIAL-TEST-CASES.md`.** Susun test vector independen untuk `CALC-REF-01` s.d. `CALC-REF-16`, lengkap dengan unit, input format, expected values, status, precision/tolerance, provenance hitungan, serta property-based/integration cases. Untuk behavior yang masih `TBD`, tulis *conditional reference tests* terhadap **EXAMPLE PROFILE**; jangan menerimanya sebagai golden production cases sebelum owner approval.

Sesudah numerik dan aturan domain matang, desain `DATABASE-DESIGN.md`, `SYSTEM-ARCHITECTURE.md`, `SCENARIO-ENGINE.md`, serta implementasi pure Go engine dapat diturunkan dengan konflik yang lebih sedikit.

**Ingat:** *Financial formulas are conditional on explicit assumptions. Actual balances are not forecasts. A transfer is not spending. A planned goal is not funded cash. A model success rate is not a real-world guarantee.*
