# FitLog

Aplikasi pencatatan kebugaran dan nutrisi harian untuk UAS Pemrograman Web/Mobile.

FitLog membantu pengguna mencatat makanan, latihan, dan berat badan dalam satu aplikasi. Seluruh data ditampilkan pada dashboard harian sehingga pengguna dapat melihat asupan kalori, durasi latihan, dan perubahan berat badan tanpa berpindah antar beberapa aplikasi.

> **Status:** Dokumentasi proyek dan rencana implementasi. Platform dan teknologi aplikasi belum ditentukan dalam PRD, sehingga bagian instalasi dan menjalankan aplikasi akan diperbarui setelah stack final dipilih.

## Daftar Isi

- [Tujuan Proyek](#tujuan-proyek)
- [Fitur Utama](#fitur-utama)
- [Fokus UAS: CRUD](#fokus-uas-crud)
- [Struktur Aplikasi](#struktur-aplikasi)
- [Rancangan Database](#rancangan-database)
- [Pembagian Tugas Tim](#pembagian-tugas-tim)
- [Rencana Pengerjaan](#rencana-pengerjaan)
- [Instalasi dan Menjalankan Aplikasi](#instalasi-dan-menjalankan-aplikasi)
- [Rencana Testing](#rencana-testing)
- [Kriteria Penerimaan](#kriteria-penerimaan)
- [Pengujian Minimum](#pengujian-minimum)
- [Catatan Penting](#catatan-penting)

## Tujuan Proyek

FitLog dibuat untuk:

1. Mencatat asupan makanan beserta kalori dan makronutrien.
2. Mencatat latihan dan durasinya.
3. Mencatat berat badan harian.
4. Menampilkan progres harian dan ringkasan latihan mingguan.
5. Menampilkan tren berat badan untuk rentang 7 dan 30 hari.
6. Menyimpan seluruh data secara lokal di perangkat pengguna.

FitLog hanya berfungsi sebagai alat pencatatan. Aplikasi ini tidak memberikan rekomendasi diet dan bukan pengganti saran dokter atau ahli gizi.

## Fitur Utama

### Fitur Inti

- Onboarding berisi target kalori harian, target makro opsional, dan berat badan awal.
- Dashboard yang menampilkan ringkasan data makanan, latihan, dan berat badan.
- Pencatatan makanan manual beserta waktu makan, kalori, makronutrien, dan jumlah porsi.
- Pencatatan latihan dengan stopwatch atau input durasi manual.
- Pencatatan berat badan dengan satu entri untuk setiap tanggal.
- Grafik tren berat badan 7 hari dan 30 hari.
- Penyimpanan lokal sehingga data tetap tersedia setelah aplikasi ditutup.

### Fitur Tambahan

Fitur berikut hanya dikerjakan jika fitur inti dan CRUD telah stabil:

- Scan barcode makanan dengan fallback ke form manual.
- Riwayat makanan untuk input ulang cepat.
- Pengingat catat makan, latihan, dan berat badan.
- Menyembunyikan pengingat jika pengguna sudah mencatat pada hari tersebut.

## Fokus UAS: CRUD

Setiap entitas utama wajib memiliki operasi berikut:

| Entitas | Create | Read | Update | Delete |
|---|---|---|---|---|
| Makanan | Tambah entri makanan | Lihat daftar dan detail makanan | Edit data makanan | Hapus dengan konfirmasi |
| Latihan | Tambah sesi latihan | Lihat daftar dan ringkasan mingguan | Edit sesi latihan | Hapus dengan konfirmasi |
| Berat badan | Tambah berat harian | Lihat riwayat dan grafik | Edit data berat | Hapus dengan konfirmasi |
| Target harian | Buat target saat onboarding | Lihat target pada pengaturan | Edit target dan berat awal | Tidak diperlukan |

Dashboard bukan entitas tersendiri. Dashboard hanya menampilkan data yang sudah tersimpan pada entitas di atas.

## Struktur Aplikasi

| Section | Nama | Fungsi Utama | Prioritas |
|---|---|---|---|
| 1 | Dashboard Harian | Menampilkan ringkasan hari ini dan tombol aksi cepat | P0 |
| 2 | Catat Makanan | Mengelola entri makanan dan target harian | P0 |
| 3 | Scan Barcode | Mengisi data produk secara otomatis | P0/bonus |
| 4 | Catat Latihan | Mengelola sesi latihan dan stopwatch | P0 |
| 5 | Berat Badan dan Tren | Mengelola berat badan dan menampilkan grafik | P0 |
| 6 | Notifikasi Pengingat | Mengingatkan pengguna untuk mencatat | P0/bonus |

Untuk UAS dengan penilaian fokus CRUD, scanner barcode, notifikasi, dan stopwatch lanjutan sebaiknya diperlakukan sebagai fitur bonus. Fitur yang harus diprioritaskan adalah CRUD lengkap, validasi form, dashboard berbasis data asli, dan penyimpanan lokal.

## Rancangan Database

### `daily_target`

Menyimpan target nutrisi dan berat awal pengguna.

| Field | Tipe | Keterangan |
|---|---|---|
| `id` | String/Integer | Primary key |
| `calorie_target` | Number | Target kalori harian, wajib |
| `protein_target_g` | Number | Target protein, opsional |
| `carbs_target_g` | Number | Target karbohidrat, opsional |
| `fat_target_g` | Number | Target lemak, opsional |
| `start_weight_kg` | Number | Berat awal pengguna |
| `created_at` | DateTime | Waktu data dibuat |
| `updated_at` | DateTime | Waktu data terakhir diperbarui |

### `food_entry`

Menyimpan seluruh catatan makanan.

| Field | Tipe | Keterangan |
|---|---|---|
| `id` | String/Integer | Primary key |
| `date` | DateTime | Tanggal dan waktu makan |
| `meal_type` | Enum | Sarapan, Makan Siang, Makan Malam, atau Camilan |
| `name` | String | Nama makanan |
| `calories` | Number | Kalori dalam kkal |
| `protein_g` | Number | Protein dalam gram |
| `carbs_g` | Number | Karbohidrat dalam gram |
| `fat_g` | Number | Lemak dalam gram |
| `servings` | Number | Jumlah porsi |
| `barcode` | String/null | Diisi ketika entri berasal dari scan barcode |
| `created_at` | DateTime | Waktu data dibuat |
| `updated_at` | DateTime | Waktu data terakhir diperbarui |

### `workout_session`

Menyimpan sesi latihan.

| Field | Tipe | Keterangan |
|---|---|---|
| `id` | String/Integer | Primary key |
| `date` | DateTime | Tanggal latihan |
| `exercise_type` | String | Jenis latihan |
| `started_at` | DateTime/null | Waktu mulai stopwatch |
| `duration_seconds` | Number | Durasi bersih tanpa jeda, dalam detik |
| `notes` | String/null | Catatan opsional |
| `created_at` | DateTime | Waktu data dibuat |
| `updated_at` | DateTime | Waktu data terakhir diperbarui |

### `weight_entry`

Menyimpan catatan berat badan.

| Field | Tipe | Keterangan |
|---|---|---|
| `id` | String/Integer | Primary key |
| `date` | Date | Tanggal pencatatan, unik per hari |
| `weight_kg` | Number | Berat badan dalam kilogram |
| `created_at` | DateTime | Waktu data dibuat |
| `updated_at` | DateTime | Waktu data terakhir diperbarui |

### `reminder_setting`

Menyimpan pengaturan pengingat jika fitur tersebut dikerjakan.

| Field | Tipe | Keterangan |
|---|---|---|
| `type` | Enum | Makan, Berat, atau Latihan |
| `is_active` | Boolean | Status aktif pengingat |
| `time` | String | Jam pengingat dalam format `HH:mm` |

## Pembagian Tugas Tim

### Anggota 1 — Frontend, Dashboard, dan Desain

**Software**

- Membuat onboarding dan pengaturan target.
- Membuat dashboard harian.
- Menyediakan komponen UI bersama seperti tombol, form, card, progress bar, dan dialog konfirmasi.
- Menghitung dan menampilkan total kalori, sisa kalori, makronutrien, latihan harian, latihan mingguan, dan berat terakhir.

**Database**

- Menangani struktur dan koneksi database lokal.
- Menjadi penanggung jawab entitas `daily_target`.

**Design**

- Membuat rancangan onboarding, dashboard, navigasi, pengaturan, dan tampilan daftar data.
- Menentukan warna, tipografi, ikon, empty state, dan gaya pesan error.

### Anggota 2 — Fitur Makanan dan Database Entri

**Software**

- Membuat daftar makanan yang dikelompokkan berdasarkan waktu makan.
- Membuat form tambah dan edit makanan.
- Membuat fitur detail dan hapus makanan dengan konfirmasi.
- Menerapkan validasi nama, kalori, makronutrien, dan jumlah porsi.

**Database**

- Menjadi penanggung jawab entitas `food_entry`.
- Menyediakan query tambah, ambil, edit, hapus, dan agregasi harian.

**Design**

- Memberi masukan untuk form makanan, daftar entri, dan empty state.
- Memastikan tampilan kategori waktu makan konsisten dengan desain Anggota 1.

### Anggota 3 — Latihan, Berat Badan, dan Pengujian

**Software**

- Membuat CRUD latihan dan input durasi manual.
- Membuat stopwatch dengan mulai, jeda, lanjut, dan selesai.
- Membuat CRUD berat badan dengan konfirmasi penimpaan data pada tanggal yang sama.
- Membuat grafik tren 7 dan 30 hari serta ringkasan latihan mingguan.

**Database**

- Menjadi penanggung jawab entitas `workout_session` dan `weight_entry`.
- Menyediakan query agregasi latihan mingguan dan tren berat badan.

**Design**

- Memberi masukan untuk tampilan stopwatch, daftar latihan, dan grafik.
- Menyusun checklist pengujian dan mencatat bug yang ditemukan.

### Tanggung Jawab Bersama

- Menentukan platform, teknologi, dan struktur project.
- Melakukan review desain dan database.
- Mengintegrasikan fitur ke dashboard.
- Melakukan pengujian akhir dan perbaikan bug.
- Menyiapkan dokumentasi serta presentasi UAS.

## Rencana Pengerjaan

| Hari | Anggota 1 | Anggota 2 | Anggota 3 |
|---|---|---|---|
| Kamis | Finalisasi desain, navigasi, setup project dan database | Model database makanan dan form tambah | Model latihan/berat dan rancangan stopwatch |
| Jumat | Onboarding, target, dan dashboard dasar | CRUD makanan lengkap | CRUD berat badan lengkap |
| Sabtu | Integrasi dashboard | Validasi, edit, dan hapus makanan | CRUD latihan, input manual, dan stopwatch |
| Minggu | Penyempurnaan UI dan integrasi | Uji makanan serta bantu integrasi | Grafik, total mingguan, dan testing |
| Senin | Perbaikan UI dan persiapan demo | Perbaikan bug makanan/database | Final testing, build, dan checklist demo |

## Instalasi dan Menjalankan Aplikasi

Bagian ini akan diisi setelah platform dan teknologi final ditentukan.

### Prasyarat

- Belum ditentukan.

### Langkah Instalasi

1. Belum ditentukan.
2. Belum ditentukan.
3. Belum ditentukan.

### Menjalankan Aplikasi

```bash
# Perintah akan ditambahkan setelah teknologi dipilih.
```

### Menjalankan Pengujian

```bash
# Perintah akan ditambahkan setelah framework pengujian dipilih.
```

## Rencana Testing

Testing dilakukan pada tiga level: pengujian logika/data, pengujian integrasi penyimpanan lokal, dan pengujian alur pengguna. Prioritasnya adalah memastikan seluruh operasi CRUD tersimpan dengan benar dan dashboard menampilkan hasil perhitungan yang tepat.

### Pilihan Alat Testing

| Platform aplikasi | Unit dan integrasi | End-to-end/UI | Keputusan |
|---|---|---|---|
| Android/iOS dengan React Native atau Flutter | Gunakan test bawaan framework | Maestro | **Direkomendasikan** |
| Aplikasi web | Gunakan test bawaan framework | Playwright | **Direkomendasikan** |
| Belum ada automation | Checklist manual dan rekam hasil uji | Tidak wajib | Minimal untuk demo UAS |

**Maestro cocok jika FitLog dibuat sebagai aplikasi mobile.** Maestro dapat menjalankan alur pengguna seperti membuka aplikasi, mengisi form, menekan tombol, memeriksa teks pada layar, dan menguji navigasi tanpa perlu banyak kode automation. Maestro tidak menggantikan unit test untuk perhitungan kalori, total latihan, atau validasi data.

Untuk aplikasi web, gunakan Playwright untuk pengujian UI end-to-end karena Playwright berjalan langsung pada browser. Jangan memakai Maestro dan Playwright sekaligus untuk MVP kecuali tim memang punya waktu; pilih satu alat UI automation sesuai platform.

### Pembagian Tugas Testing

| Anggota | Tanggung jawab testing |
|---|---|
| Anggota 1 | Menguji dashboard, target harian, navigasi, dan konsistensi tampilan data setelah perubahan CRUD. |
| Anggota 2 | Menguji CRUD makanan, validasi form makanan, perhitungan total kalori, dan total makronutrien. |
| Anggota 3 | Menguji CRUD latihan/berat badan, stopwatch, grafik, data tanggal sama, lalu mengelola checklist bug dan regression test. |
| Semua anggota | Melakukan smoke test alur utama sebelum demo dan memperbaiki bug pada fitur masing-masing. |

### Test Logika dan Database

Test ini dapat dibuat segera setelah teknologi dipilih. Fokus pada fungsi yang tidak bergantung pada tampilan.

- Perhitungan total kalori, protein, karbohidrat, dan lemak berdasarkan entri pada tanggal yang sama.
- Perhitungan sisa kalori: `calorie_target - total_calories`.
- Perhitungan total durasi dan jumlah sesi latihan pada minggu Senin sampai Minggu.
- Filter data berat badan untuk rentang 7 hari dan 30 hari.
- Validasi nilai wajib, angka negatif, nilai tidak wajar, dan jumlah porsi nol.
- Aturan satu entri berat badan per hari, termasuk proses konfirmasi timpa data.
- Penyimpanan, pembacaan ulang, pembaruan, dan penghapusan data pada database lokal.

### Test Integrasi

Lakukan setelah fitur tiap anggota digabungkan.

1. Tambah makanan, kembali ke dashboard, lalu pastikan total kalori dan makro berubah tanpa membuat data ganda.
2. Edit atau hapus makanan, lalu pastikan dashboard menghitung ulang nilai yang benar.
3. Tambah sesi latihan, lalu pastikan total durasi dan jumlah sesi minggu berjalan berubah.
4. Tambah atau ubah berat badan, lalu pastikan riwayat serta grafik ikut diperbarui.
5. Tutup aplikasi, buka kembali, dan pastikan semua data lokal tetap tersedia.
6. Uji kondisi database kosong agar daftar, dashboard, dan grafik tetap menampilkan empty state yang jelas.

### Skenario End-to-End Prioritas

Skenario ini dijalankan secara manual dan, jika platform mobile dipilih, dapat diotomatisasi dengan Maestro.

| ID | Skenario | Hasil yang diharapkan |
|---|---|---|
| E2E-01 | Onboarding -> isi target -> masuk dashboard | Target tersimpan dan dashboard dapat dibuka. |
| E2E-02 | Tambah makanan -> lihat dashboard -> edit -> hapus | Daftar dan total nutrisi selalu sesuai perubahan terakhir. |
| E2E-03 | Tambah latihan manual atau stopwatch -> simpan | Sesi tampil pada daftar dan total mingguan bertambah. |
| E2E-04 | Tambah berat -> pilih tren 7/30 hari | Riwayat, selisih, dan grafik memakai data dalam rentang yang dipilih. |
| E2E-05 | Tambah berat kedua pada tanggal sama | Pengguna diminta konfirmasi sebelum data pertama ditimpa. |
| E2E-06 | Tutup paksa aplikasi -> buka ulang | Data makanan, latihan, target, dan berat tetap tersimpan. |

### Contoh Struktur Flow Maestro

Gunakan contoh ini hanya apabila aplikasi dibuat untuk Android/iOS dan elemen UI sudah memiliki teks atau `testID` yang stabil.

```yaml
appId: <application-id>
---
- launchApp:
    clearState: true
- tapOn: "Mulai"
- tapOn: "Target Kalori"
- inputText: "2000"
- tapOn: "Simpan"
- tapOn: "Catat Makanan"
- tapOn: "Nama makanan"
- inputText: "Nasi goreng telur"
- tapOn: "Kalori"
- inputText: "450"
- tapOn: "Simpan Makanan"
- assertVisible: "450 / 2.000 kkal"
```

Simpan flow Maestro pada folder `maestro/`, misalnya `maestro/onboarding-and-food-crud.yaml`. Ganti `<application-id>` dengan ID aplikasi yang sebenarnya setelah platform diputuskan. Untuk automation yang stabil, gunakan label tombol yang konsisten atau `testID` pada elemen penting.

### Jadwal Testing

| Hari | Fokus |
|---|---|
| Setelah tiap fitur selesai | Unit test atau checklist manual untuk validasi dan operasi CRUD pemilik fitur. |
| Sabtu | Integrasi makanan, berat badan, target, dan dashboard. |
| Minggu | Jalankan skenario E2E utama, perbaiki bug, lalu ulangi regression test. |
| Senin sebelum demo | Smoke test seluruh alur utama pada perangkat/lingkungan demo. |

### Bukti Pengujian untuk UAS

Simpan bukti berikut pada folder dokumentasi proyek:

- Checklist test case beserta status lulus/gagal.
- Screenshot atau rekaman alur CRUD utama.
- Screenshot data sebelum dan sesudah edit/hapus.
- Bukti data tetap ada setelah aplikasi dibuka ulang.
- Daftar bug yang ditemukan, status perbaikan, dan pengujian ulangnya.

## Kriteria Penerimaan

Aplikasi dinyatakan siap untuk demo apabila:

- Seluruh alur utama dapat dijalankan tanpa error.
- Semua fitur CRUD pada makanan, latihan, berat badan, dan target berfungsi.
- Data tetap tersedia setelah aplikasi ditutup dan dibuka kembali.
- Dashboard menampilkan data yang sesuai dengan catatan pengguna.
- Grafik berat badan menampilkan data 7 dan 30 hari dengan benar, termasuk saat jumlah data masih sedikit.
- Entri berat badan hanya dibuat satu kali untuk setiap tanggal.
- Validasi form mencegah data kosong dan angka negatif tersimpan.
- Pengingat makan dapat diaktifkan dan muncul pada jam yang diatur jika fitur tersebut dikerjakan.

## Pengujian Minimum

- [ ] Menambah, membaca, mengubah, dan menghapus makanan.
- [ ] Membuat dan mengubah target kalori serta makronutrien.
- [ ] Memastikan total kalori dan makro pada dashboard sesuai dengan data makanan.
- [ ] Menambah, membaca, mengubah, dan menghapus latihan.
- [ ] Memastikan durasi stopwatch tersimpan dengan benar.
- [ ] Memastikan total latihan mingguan sesuai dengan data sesi.
- [ ] Menambah, membaca, mengubah, dan menghapus berat badan.
- [ ] Memastikan input berat dua kali pada tanggal yang sama tidak menghasilkan duplikasi.
- [ ] Memastikan grafik 7 hari dan 30 hari menampilkan rentang yang tepat.
- [ ] Memastikan data tetap ada setelah aplikasi ditutup dan dibuka lagi.
- [ ] Memastikan dashboard langsung berubah setelah data ditambah, diubah, atau dihapus.

## Catatan Penting

- Bahasa antarmuka aplikasi adalah Bahasa Indonesia.
- Data bersifat pribadi dan tidak dibagikan ke pihak lain.
- Aplikasi dapat digunakan tanpa internet, kecuali ketika mencari data produk melalui barcode.
- Sumber data barcode dan cakupan produk Indonesia masih perlu diputuskan.
- Platform, teknologi, identitas visual, dan penerima demo masih perlu dikonfirmasi.