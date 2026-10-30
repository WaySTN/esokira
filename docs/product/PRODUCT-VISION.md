# Esokira — Product Vision

> **Dokumen:** `docs/product/PRODUCT-VISION.md`  
> **Versi:** `0.1.0`  
> **Tanggal penyusunan:** 26 Oktober 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Pemilik keputusan produk:** Product Owner (pengguna/proyek Esokira)  
> **Jenis dokumen:** Visi dan batas produk; **bukan** spesifikasi implementasi fitur

## 0. Cara menggunakan dokumen

Dokumen ini menjawab **mengapa Esokira dibuat, siapa pengguna awalnya, masalah apa yang diselesaikan, dan apa yang tidak akan dikerjakan**. Ia menjadi landasan `PRD.md`, lalu `FEATURE-MATRIX.md`. Coding agent **tidak boleh** menganggap gagasan strategis di dokumen ini sebagai instruksi membuat fitur tanpa requirement dan acceptance criteria di PRD.

Label keputusan:

- **Decided:** secara eksplisit telah dipilih dalam pembicaraan proyek.
- **Proposed:** arah yang direkomendasikan; perlu konfirmasi Product Owner sebelum dianggap final.
- **TBD:** belum diputuskan; tidak boleh diisi diam-diam oleh agent.

Dokumen telah diperiksa untuk konsistensi internal, tetapi **belum mendapat persetujuan eksplisit Product Owner**.

## 1. Identitas produk

| Atribut | Definisi |
|---|---|
| Nama kerja | **Esokira** — *working title*, validasi merek/domain masih diperlukan (**Proposed**) |
| Kategori | Personal Financial Life Planning & Simulation |
| Bentuk utama | Aplikasi mobile (**Decided**: Flutter) |
| Pasar dan bahasa awal | Pengguna Indonesia; Bahasa Indonesia dan Rupiah/IDR sebagai pengalaman utama (**Proposed**) |
| Backend dan data utama | Go dan PostgreSQL (**Decided**) |
| Model pengembangan | Monorepo dengan direktori `/frontend` dan `/backend` (**Decided**) |
| Struktur backend | Modular monolith; financial engine terpisah dari HTTP/persistence (**Proposed**) |
| Model bisnis | Belum ditetapkan (**TBD**); tidak memengaruhi requirement MVP |

**Elevator pitch:** Esokira membantu seseorang memahami kondisi finansialnya saat ini, merencanakan tujuan masa depan, menguji konsekuensi pilihan hidup, dan menyesuaikan rencana berdasarkan hasil simulasi yang transparan—bukan sekadar melihat angka keuntungan investasi.

**Tagline sementara:** *Hitung kemungkinan. Rancang masa depan.* (**Proposed**)

## 2. Visi jangka panjang

> Menjadi pendamping perencanaan kehidupan finansial yang memungkinkan pengguna mengeksplorasi beberapa masa depan, memahami asumsi dan risikonya, lalu memilih tindakan yang sesuai kondisi serta prioritas pribadinya.

Esokira memodelkan **kondisi keuangan aktual** dan **skenario hipotetis** sebagai dua hal yang terpisah. Sistem dapat membangun proyeksi berbasis asumsi—sering disebut *financial digital twin* sebagai metafora produk—tetapi **bukan** representasi sempurna kondisi pengguna atau prediksi yang dijamin benar.

### Nilai utama yang dijanjikan

1. **Clarity:** memahami pemasukan, pengeluaran, aset, kewajiban, dan kemampuan menyisihkan dana.
2. **Planning:** menerjemahkan keinginan (rumah, pendidikan, dana darurat, pensiun) menjadi target dan kontribusi terukur.
3. **Exploration:** membandingkan hasil keputusan dengan simulasi *what-if* dan life events.
4. **Resilience:** memahami konsekuensi ketidakpastian, cash-flow stress, dan risiko tidak tercapainya tujuan.
5. **Actionability:** memperoleh alternatif tindakan yang dapat ditinjau pengguna, bukan perintah finansial yang mengklaim pasti benar.

## 3. Masalah pengguna yang ingin diselesaikan

| ID | Problem statement | Dampak bagi pengguna |
|---|---|---|
| P-01 | Data keuangan harian dan rencana masa depan sering terpisah. | Sulit mengetahui apakah tujuan realistis. |
| P-02 | Kalkulator investasi umumnya menguji satu parameter atau satu tujuan secara terisolasi. | Pengguna kurang melihat trade-off antarprioritas. |
| P-03 | Banyak rencana mengasumsikan pertumbuhan stabil dan mengabaikan risiko. | Hasil proyeksi dapat disalahartikan sebagai kepastian. |
| P-04 | Pengguna sulit memahami pengaruh keputusan hidup seperti pindah kerja atau membeli rumah. | Keputusan besar dilakukan tanpa membandingkan skenario. |
| P-05 | Hasil perhitungan tidak selalu diterjemahkan menjadi langkah praktis. | Pengguna tahu angka akhir, tetapi tidak tahu pilihan yang tersedia. |

Masalah di atas adalah **hipotesis produk** berdasarkan diskusi awal, bukan temuan riset pengguna yang telah divalidasi.

## 4. Pengguna sasaran dan jobs-to-be-done

### Segmen utama (Proposed)

**Mahasiswa tingkat akhir, fresh graduate, dan pekerja muda di Indonesia** yang mulai mengelola penghasilan dan memiliki beberapa tujuan finansial, tetapi belum membutuhkan sistem akuntansi profesional.

- **JTBD-01 — Kenali posisi:** "Saat mulai merencanakan keuangan, saya ingin mengetahui uang, aset, kewajiban, dan kapasitas menabung agar dapat membuat target yang masuk akal."
- **JTBD-02 — Rancang tujuan:** "Saat ingin mencapai milestone hidup, saya ingin mengetahui estimasi kebutuhan setoran dan waktu yang dibutuhkan."
- **JTBD-03 — Uji keputusan:** "Saat mempertimbangkan pilihan besar, saya ingin membandingkan hasil beberapa skenario tanpa mengubah catatan keuangan nyata."
- **JTBD-04 — Hadapi ketidakpastian:** "Saat kondisi berubah, saya ingin mengetahui risiko dan alternatif penyesuaian rencana."

### Segmen lanjutan (Future; bukan prioritas MVP)

Pasangan/keluarga yang berbagi tujuan, pengguna dengan portofolio kompleks, dan orang yang merancang pensiun dini. Kebutuhannya harus divalidasi sebelum modul lanjutan dianggap final.

### Hal yang perlu divalidasi melalui riset

Apakah calon pengguna bersedia memasukkan data secara manual, mengerti hasil simulasi, memahami perbedaan aktual vs proyeksi, dan kembali memperbarui rencana. Persona di atas **bukan** bukti demand pasar.

## 5. Pilar produk

| ID | Pilar | Hasil bagi pengguna | Arah produk |
|---|---|---|---|
| V-01 | **Financial Reality** | Satu gambaran kondisi aktual yang dapat ditelusuri. | Akun, transaksi, aset, utang, net worth. |
| V-02 | **Goal-Based Planning** | Tujuan memiliki target, tenggat, dan kebutuhan kontribusi. | Goals, compound calculations, goal forecast. |
| V-03 | **Life & Scenario Simulation** | Dapat membandingkan beberapa jalur masa depan. | Life events, deterministic projection, what-if scenarios. |
| V-04 | **Risk & Optimization** | Mengerti trade-off, ketahanan, dan keterbatasan rencana. | Stress test, allocation optimization, Monte Carlo, retirement. |
| V-05 | **Explainable Guidance** | Memahami alasan rekomendasi dan dapat memilih sendiri. | Rule-based insights, Financial GPS/adaptive roadmap. |

**Prinsip integrasi:** modul advanced memakai model keuangan dan asumsi yang konsisten. Perubahan skenario tidak pernah memodifikasi catatan aktual, dan hasil simulasi bukan saldo rekening.

## 6. Pengalaman ideal pengguna

1. **Onboarding singkat:** pengguna membuat profil dan menambahkan informasi finansial minimum; data sensitif diberi penjelasan tujuan penggunaan.
2. **Current picture:** aplikasi menjelaskan posisi finansial saat ini, keterbatasan data, serta komponen yang belum lengkap.
3. **Set a goal:** pengguna membuat target, tanggal, prioritas, dan melihat estimasi kebutuhan dana.
4. **Explore possibilities:** pengguna mengubah asumsi pendapatan, pengeluaran, tanggal, dan peristiwa hidup pada salinan skenario.
5. **Compare & understand:** aplikasi menampilkan hasil, alasan perbedaan, risiko, dan asumsi yang mendasarinya.
6. **Adjust over time:** pengguna memperbarui kondisi aktual dan mengevaluasi apakah rencana perlu diubah.

**North-star experience:** "Saya dapat memahami tujuan saya, melihat beberapa alternatif untuk mencapainya, dan tahu asumsi yang membuat setiap alternatif masuk akal atau berisiko."

## 7. Prinsip produk yang tidak boleh dilanggar

| ID | Prinsip | Implikasi wajib |
|---|---|---|
| G-01 | **Actual ≠ simulated** | Data aktual dan data proyeksi dipisahkan secara konseptual, penyimpanan, UI, dan API. |
| G-02 | **Asumsi transparan** | Setiap simulasi mengungkap horizon, rate, inflasi, biaya, dan ketidakpastian yang relevan. |
| G-03 | **Financial correctness before visual polish** | Formula harus memiliki spesifikasi, contoh terverifikasi, serta regression tests sebelum dipublikasikan. |
| G-04 | **No false certainty** | Jangan menyebut proyeksi, probabilitas model, atau rekomendasi sebagai jaminan hasil nyata. |
| G-05 | **Privacy by design** | Pengguna hanya mengakses data miliknya; minimisasi data, kontrol penghapusan, dan keamanan diwajibkan. |
| G-06 | **Explainable over magical** | AI generatif (bila nanti ada) tidak menjadi sumber angka atau keputusan akhir tanpa engine yang tervalidasi. |
| G-07 | **Mobile-first simplicity** | Bahasa mudah dipahami; rincian statistik tersedia melalui advanced view, bukan dipaksakan pada semua pengguna. |
| G-08 | **One source of calculation truth** | Logika keuangan inti terdefinisi pada spesifikasi domain dan satu engine yang dapat diuji. |

## 8. Diferensiasi yang dituju (hipotesis, bukan klaim pasar)

- **Terintegrasi:** menghubungkan catatan aktual, tujuan, life events, dan risiko alih-alih kalkulator terpisah.
- **Berorientasi keputusan:** menampilkan perubahan rencana dan trade-off, bukan sekadar angka akhir.
- **Ramah Indonesia:** penggunaan bahasa Indonesia, Rupiah, dan contoh milestone yang relevan; tidak mengasumsikan produk investasi tertentu.
- **Simulasi dapat diperiksa:** rumus, asumsi, versi perhitungan, serta batasan output dijelaskan secara wajar.
- **Financial GPS:** fitur vision-level yang mengusulkan rute alternatif ketika target tidak feasible; implementasinya menunggu engine dan kebijakan rekomendasi tervalidasi.

Diferensiasi ini **masih perlu diverifikasi** dengan riset kompetitor dan pengujian pengguna. Nama Esokira juga belum melalui clearance merek/domain.

## 9. Ruang lingkup berdasarkan horizon

| Horizon | Fokus pengalaman | Catatan |
|---|---|---|
| **R0 — Engineering Foundation** | Fondasi sistem, akses aman, database, pengujian, standar API. | Enabler teknis, bukan rilis produk publik. |
| **R1 — MVP** | Posisi finansial aktual, goals, compound calculator, target forecast, dasar transparansi perhitungan, kontrol data pengguna. | Alur dasar end-to-end harus bekerja tanpa data dummy permanen. |
| **R2 — Planning V1** | Budgeting, life events, deterministic projection, scenario comparison, future cash flow, stress test dasar, explainable insights. | Membentuk pengalaman perencanaan terintegrasi. |
| **R3 — Advanced** | Multi-goal optimizer, Monte Carlo, retirement/FIRE, Financial GPS. | Memerlukan model matematis, pengujian, dan evaluasi risiko tambahan. |
| **R4 — Discovery backlog** | Shared goals, portfolio planning, koneksi keuangan, AI assistant. | Bukan komitmen implementasi. |

Label R0–R4 adalah **usulan urutan dependensi, bukan tanggal atau komitmen rilis**. Daftar dan klasifikasi requirement ada di `PRD.md`; status pekerjaan ada di `FEATURE-MATRIX.md`.

## 10. Batasan dan non-goals

Untuk R1–R3, kecuali ditentukan lagi dalam requirement yang disetujui:

- **Tidak** memindahkan uang, menerima deposit, memproses pembayaran, atau mengeksekusi perdagangan/investasi.
- **Tidak** menjamin imbal hasil investasi, kelulusan kredit/KPR, maupun keberhasilan mencapai tujuan.
- **Tidak** menjadi penasihat investasi berizin atau memberikan rekomendasi produk investasi spesifik sebagai layanan profesional.
- **Tidak** mengharuskan integrasi bank, e-wallet, broker, atau market-data berbayar pada MVP; input manual adalah baseline (**Proposed**).
- **Tidak** menyimpan proyeksi sebagai saldo aktual, memodifikasi transaksi nyata karena scenario, atau mengganti audit trail transaksi dengan hasil model.
- **Tidak** menjadikan AI chatbot generatif sebagai fondasi perhitungan finansial.
- **Tidak** menyasar enterprise accounting, payroll, perpajakan profesional, atau fungsi banking inti.

Jika ruang lingkup berubah, lakukan pembaruan PRD, feature matrix, dan keputusan teknis terkait lebih dulu.

## 11. Ukuran keberhasilan dan guardrails

**Metrik produk yang diusulkan (perlu baseline dan target setelah validasi pengguna):**

| ID | Metrik | Cara menghitung | Status target |
|---|---|---|---|
| M-01 | First-plan activation | Persentase akun baru yang menyelesaikan profil minimum, membuat goal, dan melihat estimasi pertama. | **TBD**, jangan invent angka. |
| M-02 | Planning retention | Persentase pengguna aktif yang kembali meninjau/memperbarui rencana dalam 30 hari. | **TBD** |
| M-03 | Scenario usefulness | Persentase partisipan uji yang bisa menjelaskan beda dua skenario tanpa bantuan. | **TBD** |
| M-04 | Calculation validity | Proporsi golden test cases yang lulus pada CI sebelum rilis. | Target rilis: **100% untuk suite wajib**. |
| M-05 | Security/privacy readiness | Tidak ada isu kritis terbuka terkait otorisasi lintas akun dan penghapusan data sebelum rilis. | Kriteria release gate. |

Tidak mengumpulkan data finansial mentah untuk analitik produk tanpa kebutuhan, dasar pemrosesan, dan desain privasi yang jelas. Metode pengukuran M-01–M-03 perlu dikaji saat implementasi analitik.

## 12. Risiko produk dan mitigasi awal

| Risiko | Dampak | Strategi awal |
|---|---|---|
| Pengguna enggan memasukkan data | Manfaat aplikasi rendah | Onboarding progresif; minimalkan input awal; uji kebutuhan impor CSV kemudian. |
| Simulasi terlihat seperti kepastian | Keputusan pengguna bisa keliru | Label asumsi, rentang, penjelasan keterbatasan, hindari klaim pasti. |
| Salah rumus/presisi | Hasil finansial menyesatkan | Domain specs, golden cases, property tests, review algoritma. |
| Fitur advanced meluas tanpa kendali | Waktu pengembangan membengkak | Rilis berjenjang dan dependency gates. |
| Paparan informasi keuangan pribadi | Kerugian pengguna dan reputasi | Auth, authorization, logging aman, minimisasi data, security tests. |
| Kompleksitas UI | Pengguna tidak mengerti hasil | Mode sederhana sebagai default, progressive disclosure untuk detail statistik. |

## 13. Keputusan awal dan pertanyaan terbuka

| ID | Topik | Status | Keputusan / kebutuhan konfirmasi |
|---|---|---|---|
| D-001 | Flutter mobile | **Decided** | Frontend Flutter. |
| D-002 | Go backend | **Decided** | Backend Go. |
| D-003 | PostgreSQL | **Decided** | Penyimpanan data utama PostgreSQL. |
| D-004 | Monorepo | **Decided** | Struktur utama `/frontend` dan `/backend`. |
| D-005 | Nama Esokira | **Proposed** | Working name; periksa merek, domain, store listing. |
| D-006 | Target Indonesia/IDR | **Proposed** | Validasi pasar, locale, dan dukungan mata uang awal. |
| D-007 | Modular monolith + REST/OpenAPI | **Proposed** | Finalisasi pada dokumentasi arsitektur. |
| D-008 | Authentication provider | **TBD** | Firebase Authentication vs solusi OIDC lain. |
| D-009 | Detail financial modeling | **TBD** | Presisi uang, pembulatan, jadwal setoran, inflasi, utang, asumsi risiko. |
| D-010 | Distribusi return & Monte Carlo | **TBD** | Didefinisikan dan direview dalam spesifikasi finansial sebelum implementasi. |
| D-011 | Model bisnis dan monetisasi | **TBD** | Tidak memblokir penyusunan MVP. |
| D-012 | Cloud/deployment & batas biaya | **TBD** | Ditetapkan di fase arsitektur/deployment. |
| D-013 | Pengaturan offline/sync | **TBD** | Kebutuhan offline harus jelas sebelum dipilih solusinya. |
| D-014 | Data perbankan eksternal | **Proposed out-of-scope R1** | Hanya dipertimbangkan setelah validasi kebutuhan pengguna dan kepatuhan. |

## 14. Kriteria persetujuan Product Vision

- [x] Masalah pengguna, nilai, target awal, dan non-goals dijelaskan.
- [x] Pembedaan kondisi aktual vs hipotetis ditegaskan.
- [x] Horizon fitur dibedakan dari komitmen implementasi.
- [x] Keputusan teknis yang dipilih dipisahkan dari usulan/TBD.
- [x] Risiko perhitungan, privasi, dan misleading claims diidentifikasi.
- [ ] **Persetujuan eksplisit Product Owner atas visi, target pengguna, batas MVP, dan penggunaan nama kerja.**

**Hasil review internal:** LULUS untuk menjadi *input draf PRD*; **belum approved untuk memerintahkan implementasi**.
