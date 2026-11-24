# FitLog — Rancangan Desain & Mockup

Dokumen rancangan UI untuk FitLog. Belum diimplementasikan. Sumber requirement: [PRD.md](PRD.md). Aturan kode: [AGENTS.md](../AGENTS.md).

## Daftar Isi

1. [Prinsip Desain](#1-prinsip-desain)
2. [Design Token](#2-design-token)
3. [Komponen Bersama](#3-komponen-bersama)
4. [Peta Navigasi](#4-peta-navigasi)
5. [Mockup Layar](#5-mockup-layar)
6. [State: Kosong, Error, Loading](#6-state-kosong-error-loading)
7. [Copy UI](#7-copy-ui)
8. [Aksesibilitas](#8-aksesibilitas)
9. [Checklist Review Desain](#9-checklist-review-desain)

---

## 1. Prinsip Desain

- **Catat dalam ≤ 3 ketukan.** Dari dashboard, setiap jenis catatan bisa dibuka dengan satu tombol aksi cepat.
- **Angka besar, label kecil.** Pengguna melihat kalori, durasi, dan berat sekilas.
- **Satu warna per domain.** Makanan = oranye, Latihan = biru, Berat = ungu. Warna utama (hijau) untuk aksi.
- **Netral, bukan menghakimi.** Tidak ada kata "gagal", "berlebihan", atau saran diet. Melewati target cukup ditandai dengan warna peringatan dan teks faktual.
- **Native terlebih dahulu.** Pakai tab, header, dan dialog bawaan platform (NativeTabs, `Alert`, modal stack).
- **Mendukung mode terang dan gelap** mengikuti sistem.

## 2. Design Token

Token akan ditulis di `src/constants/theme.ts`, memperluas `Colors` dan `Spacing` yang sudah ada.

### 2.1 Warna

| Token | Terang | Gelap | Pemakaian |
|---|---|---|---|
| `background` | `#FFFFFF` | `#000000` | Latar layar |
| `backgroundElement` | `#F0F0F3` | `#212225` | Card, input |
| `backgroundSelected` | `#E0E1E6` | `#2E3135` | Segmen terpilih, pressed |
| `text` | `#000000` | `#FFFFFF` | Teks utama |
| `textSecondary` | `#60646C` | `#B0B4BA` | Label, keterangan |
| `primary` | `#15803D` | `#22C55E` | Tombol utama, progress normal |
| `onPrimary` | `#FFFFFF` | `#052E16` | Teks di atas tombol utama |
| `food` | `#C2410C` | `#FB923C` | Aksen makanan |
| `workout` | `#1D4ED8` | `#60A5FA` | Aksen latihan |
| `weight` | `#7E22CE` | `#C084FC` | Aksen berat, garis grafik |
| `warning` | `#B45309` | `#FBBF24` | Kalori melewati target |
| `danger` | `#B91C1C` | `#F87171` | Tombol hapus, error form |
| `border` | `#D9D9E0` | `#3A3D42` | Garis input, pemisah |

Semua pasangan teks/latar di atas ditargetkan kontras ≥ 4.5:1 (cek ulang saat implementasi).

### 2.2 Tipografi

Memakai font sistem (`Fonts.sans`, angka besar pakai `Fonts.rounded`).

| Token | Ukuran / Line height | Berat | Pemakaian |
|---|---|---|---|
| `display` | 40 / 48 | 700 | Angka utama (sisa kalori, stopwatch) |
| `title` | 24 / 32 | 700 | Judul layar |
| `heading` | 18 / 24 | 600 | Judul card & section |
| `body` | 16 / 24 | 400 | Teks umum, input |
| `label` | 14 / 20 | 500 | Label field, keterangan |
| `caption` | 12 / 16 | 400 | Satuan, timestamp |

Angka memakai `fontVariant: ['tabular-nums']` agar stopwatch dan total tidak bergeser.

### 2.3 Spasi, Radius, Ukuran

- Spasi memakai `Spacing` yang sudah ada: 2, 4, 8, **16** (padding layar), **24** (jarak antar section), 32, 64.
- Radius: `sm 8` (input, chip), `md 12` (tombol), `lg 16` (card).
- Tinggi minimum area sentuh: **48 dp**. Tombol utama tinggi 52 dp.
- Lebar konten maksimum `MaxContentWidth = 800` (tablet/web).

### 2.4 Ikon

Pakai `expo-symbols` (SF Symbols di iOS, Material Symbols di Android) yang sudah terpasang.

| Konteks | Ikon |
|---|---|
| Beranda | house |
| Makanan | fork.knife / restaurant |
| Latihan | figure.run / directions_run |
| Berat | scalemass / monitor_weight |
| Tambah | plus |
| Edit | pencil |
| Hapus | trash |
| Pengaturan | gearshape / settings |
| Scan | barcode.viewfinder / barcode_scanner |

## 3. Komponen Bersama

Dimiliki Anggota 1, dipakai semua anggota. Letak: `src/components/`.

| Komponen | Props utama | Keterangan |
|---|---|---|
| `Screen` | `children`, `scroll?` | Safe area + padding 16 + latar `background` |
| `Card` | `title?`, `accent?`, `onPress?` | Latar `backgroundElement`, radius 16, garis aksen kiri 4 dp opsional |
| `Button` | `variant: primary \| secondary \| danger`, `label`, `loading?` | Tinggi 52, disabled saat loading |
| `TextField` | `label`, `value`, `error?`, `unit?`, `keyboardType` | Label di atas, satuan di kanan, error merah di bawah |
| `SegmentedControl` | `options`, `value`, `onChange` | Untuk waktu makan & rentang 7/30 hari |
| `ProgressBar` | `value`, `max`, `color` | Berubah ke `warning` bila `value > max` |
| `StatTile` | `label`, `value`, `unit`, `accent` | Angka besar + label kecil |
| `ListRow` | `title`, `subtitle?`, `trailing?`, `onPress` | Baris daftar entri |
| `EmptyState` | `icon`, `title`, `message`, `actionLabel?`, `onAction?` | Lihat §6 |
| `confirmDelete(name)` | fungsi | Membungkus `Alert.alert` dengan tombol Batal / Hapus (destructive) |

### Anatomi `TextField`

```
Kalori per porsi
┌──────────────────────────────┐
│ 450                     kkal │
└──────────────────────────────┘
⚠ Kalori tidak boleh negatif
```

### Anatomi `Card` dengan `ProgressBar`

```
┌──────────────────────────────────┐
▌ Kalori hari ini                  │
▌ 1.250 / 2.000 kkal               │
▌ ███████████████░░░░░░░░░  62%    │
▌ Sisa 750 kkal                    │
└──────────────────────────────────┘
```

## 4. Peta Navigasi

```
             ┌──────────────┐
  pertama ──▶│  Onboarding  │── Simpan ──┐
  kali       └──────────────┘            ▼
                    ┌────────── Tab Bar ──────────────────────────┐
                    │ Beranda │ Makanan │ Latihan │ Berat          │
                    └────┬────────┬─────────┬─────────┬───────────┘
                         │        │         │         │
         ⚙ Pengaturan ◀──┤        ├─▶ Detail Makanan  ├─▶ Form Berat (modal)
                         │        ├─▶ Form Makanan    │
   aksi cepat ───────────┤        │    (modal) ─▶ Scan│(bonus)
                         │        │                   │
                         │        └ Latihan ─▶ Form Latihan (modal)
                         │                  └─▶ Stopwatch
```

| Route | Layar | Presentasi |
|---|---|---|
| `/onboarding` | Onboarding | Stack, tanpa tombol kembali |
| `/(tabs)` | Beranda | Tab |
| `/(tabs)/food` | Daftar Makanan | Tab |
| `/(tabs)/workout` | Daftar Latihan | Tab |
| `/(tabs)/weight` | Berat & Tren | Tab |
| `/food/form` · `/food/form?id=` | Tambah / Edit Makanan | Modal |
| `/food/[id]` | Detail Makanan | Push |
| `/workout/form` · `?id=` | Tambah / Edit Latihan | Modal |
| `/workout/stopwatch` | Stopwatch | Push, fullscreen |
| `/weight/form` · `?id=` | Tambah / Edit Berat | Modal |
| `/settings` | Pengaturan Target | Push |
| `/scan` | Scan Barcode (bonus) | Modal |

## 5. Mockup Layar

Lebar acuan 360 dp. Simbol: `[ Tombol ]`, `( ○ ● )` segmen, `▾` dropdown, `›` navigasi ke detail.

### 5.1 Onboarding — `onboarding.tsx` (Anggota 1)

Satu layar, scroll. Tombol "Mulai" di layar sambutan lalu form.

```
┌────────────────────────────────┐
│                                │
│          🥗  FitLog            │
│  Catat makan, latihan, dan     │
│  berat badan dalam satu tempat │
│                                │
│  Target Harian                 │
│  Target Kalori *               │
│  ┌──────────────────────────┐  │
│  │ 2000                kkal │  │
│  └──────────────────────────┘  │
│                                │
│  Target Makro (opsional)       │
│  ┌────────┐┌────────┐┌───────┐ │
│  │Protein ││Karbo   ││Lemak  │ │
│  │ 120  g ││ 250  g ││ 60  g │ │
│  └────────┘└────────┘└───────┘ │
│                                │
│  Berat Awal *                  │
│  ┌──────────────────────────┐  │
│  │ 72,5                  kg │  │
│  └──────────────────────────┘  │
│                                │
│  FitLog hanya alat pencatatan, │
│  bukan pengganti saran dokter  │
│  atau ahli gizi.               │
│                                │
│  [          Simpan          ]  │
└────────────────────────────────┘
```

- Simpan → membuat `daily_target` dan `weight_entry` hari ini dengan berat awal → `router.replace('/(tabs)')`.
- Tombol aktif walau field kosong; validasi muncul saat ditekan.

### 5.2 Beranda (Dashboard) — `(tabs)/index.tsx` (Anggota 1)

```
┌────────────────────────────────┐
│ Hari ini                    ⚙  │
│ Senin, 24 November 2026        │
│                                │
│ ┌────────────────────────────┐ │
│ ▌ Sisa Kalori                │ │
│ ▌        750                 │ │
│ ▌ 1.250 / 2.000 kkal         │ │
│ ▌ ██████████████░░░░░░  62%  │ │
│ └────────────────────────────┘ │
│                                │
│ ┌────────┐┌────────┐┌────────┐ │
│ │Protein ││Karbo   ││Lemak   │ │
│ │ 80/120g││150/250g││ 40/60g │ │
│ │ ████░░ ││ ███░░░ ││ ████░░ │ │
│ └────────┘└────────┘└────────┘ │
│                                │
│ ┌─────────────┐┌─────────────┐ │
│ ▌ Latihan     ││▌ Berat       │ │
│ ▌ 45 mnt      ││▌ 72,1 kg     │ │
│ ▌ hari ini    ││▌ −0,4 dari   │ │
│ ▌ 3 sesi ·    ││▌   awal      │ │
│ ▌ 2j 10m/mgg  ││▌ 23 Nov      │ │
│ └─────────────┘└─────────────┘ │
│                                │
│ Catat cepat                    │
│ [+ Makanan][+ Latihan][+ Berat]│
├────────────────────────────────┤
│ Beranda  Makanan  Latihan Berat│
└────────────────────────────────┘
```

- Card makro hanya tampil untuk target yang diisi; bila tidak ada target makro, tampilkan total gram tanpa progress bar.
- Kalori melewati target → angka utama berubah jadi "Lebih 150 kkal" dengan warna `warning`.
- Card Latihan/Berat → ketuk membuka tab terkait.
- testID: `dashboard-calorie-summary`, `dashboard-quick-food`, `dashboard-quick-workout`, `dashboard-quick-weight`.

### 5.3 Daftar Makanan — `(tabs)/food.tsx` (Anggota 2)

```
┌────────────────────────────────┐
│ Makanan                     +  │
│ ‹  Senin, 24 Nov 2026  ›       │
│ Total 1.250 kkal               │
│                                │
│ Sarapan                 350    │
│ ┌────────────────────────────┐ │
│ │ Roti gandum telur      ›   │ │
│ │ 1 porsi · P 18 K 30 L 12   │ │
│ │                  350 kkal  │ │
│ └────────────────────────────┘ │
│                                │
│ Makan Siang             900    │
│ ┌────────────────────────────┐ │
│ │ Nasi goreng telur      ›   │ │
│ │ 2 porsi · P 24 K 110 L 30  │ │
│ │                  900 kkal  │ │
│ └────────────────────────────┘ │
│                                │
│ Makan Malam               0    │
│   Belum ada catatan  [+ Tambah]│
│                                │
│ Camilan                   0    │
│   Belum ada catatan  [+ Tambah]│
├────────────────────────────────┤
│ Beranda  Makanan  Latihan Berat│
└────────────────────────────────┘
```

- Panah `‹ ›` mengganti tanggal; tidak bisa maju melewati hari ini.
- "+ Tambah" per grup membuka form dengan `meal_type` terisi.
- Nilai yang tampil adalah total (per porsi × porsi).

### 5.4 Form Makanan — `food/form.tsx` (Anggota 2)

```
┌────────────────────────────────┐
│ Batal    Tambah Makanan        │
│                                │
│ [▣ Scan Barcode]   (bonus)     │
│                                │
│ Waktu Makan                    │
│ (Sarapan|●Siang|Malam|Camilan) │
│                                │
│ Nama makanan *                 │
│ ┌──────────────────────────┐   │
│ │ Nasi goreng telur        │   │
│ └──────────────────────────┘   │
│ Kalori per porsi *             │
│ ┌──────────────────────────┐   │
│ │ 450                 kkal │   │
│ └──────────────────────────┘   │
│ ┌────────┐┌────────┐┌───────┐  │
│ │Protein ││Karbo   ││Lemak  │  │
│ │ 12   g ││ 55   g ││ 15  g │  │
│ └────────┘└────────┘└───────┘  │
│ Jumlah porsi *                 │
│ ┌──────────────────────────┐   │
│ │  [−]       2       [+]   │   │
│ └──────────────────────────┘   │
│ Tanggal & jam   24 Nov, 12:30 ▾│
│                                │
│ Total: 900 kkal · P 24 K 110   │
│        L 30                    │
│                                │
│ [       Simpan Makanan       ] │
└────────────────────────────────┘
```

- Mode edit: judul "Edit Makanan", tombol "Simpan Perubahan".
- Stepper porsi ±0,25; tetap bisa diketik.
- Ringkasan total diperbarui langsung saat mengetik.
- testID: `food-form-name`, `food-form-calories`, `food-form-servings`, `food-form-save`.

### 5.5 Detail Makanan — `food/[id].tsx` (Anggota 2)

```
┌────────────────────────────────┐
│ ‹ Makanan                 Edit │
│                                │
│ Nasi goreng telur              │
│ Makan Siang · 24 Nov, 12:30    │
│                                │
│ ┌────────────────────────────┐ │
│ │        900 kkal            │ │
│ │   2 porsi × 450 kkal       │ │
│ └────────────────────────────┘ │
│ ┌────────┐┌────────┐┌────────┐ │
│ │Protein ││Karbo   ││Lemak   │ │
│ │  24 g  ││ 110 g  ││  30 g  │ │
│ └────────┘└────────┘└────────┘ │
│                                │
│ Dibuat   24 Nov 2026, 12:31    │
│ Diubah   24 Nov 2026, 12:45    │
│                                │
│ [        Hapus Makanan       ] │  ← variant danger
└────────────────────────────────┘
```

### 5.6 Dialog Konfirmasi Hapus (semua entitas)

```
┌──────────────────────────────┐
│ Hapus "Nasi goreng telur"?   │
│                              │
│ Data yang dihapus tidak bisa │
│ dikembalikan.                │
│                              │
│        Batal      Hapus      │
└──────────────────────────────┘
```

Pakai `Alert.alert` native; tombol Hapus `style: 'destructive'`. Setelah hapus → kembali ke daftar.

### 5.7 Daftar Latihan — `(tabs)/workout.tsx` (Anggota 3)

```
┌────────────────────────────────┐
│ Latihan                     +  │
│                                │
│ ┌────────────────────────────┐ │
│ ▌ Minggu ini (24–30 Nov)     │ │
│ ▌ 2 j 10 mnt  ·  3 sesi      │ │
│ ▌ S  S  R  K  J  S  M        │ │
│ ▌ █  ▁  █  ▁  ▁  ▁  ▁        │ │
│ └────────────────────────────┘ │
│                                │
│ [ ▶ Mulai Stopwatch ]          │
│                                │
│ Hari ini                       │
│ ┌────────────────────────────┐ │
│ │ Lari pagi              ›   │ │
│ │ 06:10 · 45 mnt             │ │
│ └────────────────────────────┘ │
│ Sabtu, 22 Nov                  │
│ ┌────────────────────────────┐ │
│ │ Angkat beban           ›   │ │
│ │ 17:00 · 1 j 05 mnt         │ │
│ └────────────────────────────┘ │
├────────────────────────────────┤
│ Beranda  Makanan  Latihan Berat│
└────────────────────────────────┘
```

- Bar mini Senin–Minggu menunjukkan hari dengan latihan (tinggi relatif durasi).
- Ketuk baris → form edit (tidak perlu layar detail terpisah; tombol hapus ada di form edit).
- `+` → form manual.

### 5.8 Form Latihan — `workout/form.tsx` (Anggota 3)

```
┌────────────────────────────────┐
│ Batal    Tambah Latihan        │
│                                │
│ Jenis latihan *                │
│ ┌──────────────────────────┐   │
│ │ Lari pagi                │   │
│ └──────────────────────────┘   │
│ [Lari][Jalan][Sepeda][Beban]   │  ← chip isi cepat
│                                │
│ Durasi *                       │
│ ┌──────────┐ ┌──────────┐      │
│ │ 0     jam│ │ 45   mnt │      │
│ └──────────┘ └──────────┘      │
│ Tanggal & jam   24 Nov, 06:10 ▾│
│                                │
│ Catatan                        │
│ ┌──────────────────────────┐   │
│ │ 5 km, santai             │   │
│ └──────────────────────────┘   │
│                                │
│ [        Simpan Latihan      ] │
│ [         Hapus Latihan      ] │  ← hanya mode edit
└────────────────────────────────┘
```

Jika dibuka dari stopwatch: durasi terisi otomatis (jam/menit/detik) dan `started_at` disimpan.

### 5.9 Stopwatch — `workout/stopwatch.tsx` (Anggota 3)

```
 Siap                Berjalan              Dijeda
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ ‹ Latihan    │   │ ‹ Latihan    │   │ ‹ Latihan    │
│              │   │              │   │              │
│  00:00:00    │   │  00:12:37    │   │  00:12:37    │
│              │   │   ● berjalan │   │   ‖ dijeda   │
│              │   │              │   │              │
│              │   │              │   │              │
│ [  ▶ Mulai ] │   │ [  ‖ Jeda  ] │   │ [▶ Lanjut]   │
│              │   │              │   │ [■ Selesai]  │
└──────────────┘   └──────────────┘   └──────────────┘
```

- Alur: Mulai → Jeda ⇄ Lanjut → Selesai → buka form latihan dengan durasi terisi.
- Kembali saat berjalan/dijeda → konfirmasi "Buang sesi ini?".
- Angka pakai token `display` + tabular-nums; layar tidak mati selama berjalan (opsional, `expo-keep-awake`).

### 5.10 Berat & Tren — `(tabs)/weight.tsx` (Anggota 3)

```
┌────────────────────────────────┐
│ Berat Badan                 +  │
│                                │
│ ┌────────────────────────────┐ │
│ ▌ 72,1 kg                    │ │
│ ▌ −0,4 kg dalam 7 hari       │ │
│ └────────────────────────────┘ │
│                                │
│ (  ● 7 Hari  |  30 Hari  )     │
│ ┌────────────────────────────┐ │
│ │73 ┤●                       │ │
│ │   │  ●──●                  │ │
│ │72 ┤       ●──●   ●──●      │ │
│ │   │            ●           │ │
│ │71 ┼──┬──┬──┬──┬──┬──┬──    │ │
│ │   18 19 20 21 22 23 24     │ │
│ └────────────────────────────┘ │
│                                │
│ Riwayat                        │
│ │ Sen, 24 Nov   72,1 kg  −0,2 ›│
│ │ Min, 23 Nov   72,3 kg  +0,1 ›│
│ │ Sab, 22 Nov   72,2 kg  −0,3 ›│
├────────────────────────────────┤
│ Beranda  Makanan  Latihan Berat│
└────────────────────────────────┘
```

- Selisih = entri terakhir − entri pertama dalam rentang. Tidak ada warna "baik/buruk"; selisih ditampilkan netral.
- Grafik: titik untuk tiap entri, garis menghubungkan; hari tanpa data tidak diisi (tidak diinterpolasi sebagai titik).
- Data < 2 titik → tampilkan titik tunggal + teks "Tambah minimal 2 catatan untuk melihat tren."
- Ketuk riwayat → form edit berat.

### 5.11 Form Berat — `weight/form.tsx` (Anggota 3)

```
┌────────────────────────────────┐
│ Batal    Catat Berat           │
│                                │
│ Tanggal          24 Nov 2026 ▾ │
│ Berat badan *                  │
│ ┌──────────────────────────┐   │
│ │ 72,1                  kg │   │
│ └──────────────────────────┘   │
│                                │
│ [          Simpan            ] │
│ [         Hapus Berat        ] │  ← hanya mode edit
└────────────────────────────────┘
```

Dialog timpa (tanggal sudah punya entri):

```
┌──────────────────────────────┐
│ Berat 24 Nov sudah tercatat  │
│                              │
│ Data lama 72,3 kg akan       │
│ diganti menjadi 72,1 kg.     │
│                              │
│        Batal      Ganti      │
└──────────────────────────────┘
```

### 5.12 Pengaturan Target — `settings.tsx` (Anggota 1)

Sama dengan form onboarding tanpa teks sambutan; judul "Target Harian", tombol "Simpan Perubahan". Bagian bawah (bonus) berisi pengaturan pengingat:

```
│ Pengingat (bonus)              │
│ Catat makan        12:00  [●○] │
│ Catat latihan      18:00  [○●] │
│ Catat berat        07:00  [●○] │
```

### 5.13 Scan Barcode — `scan.tsx` (bonus)

```
┌────────────────────────────────┐
│ Batal     Scan Barcode         │
│ ┌────────────────────────────┐ │
│ │                            │ │
│ │     ┌──────────────┐       │ │
│ │     │   kamera     │       │ │
│ │     └──────────────┘       │ │
│ │                            │ │
│ └────────────────────────────┘ │
│ Arahkan kamera ke barcode      │
│ [      Isi Manual Saja       ] │
└────────────────────────────────┘
```

Produk tidak ditemukan / tanpa internet / izin kamera ditolak → kembali ke form makanan dengan pesan dan field barcode terisi.

## 6. State: Kosong, Error, Loading

| Layar | Empty state |
|---|---|
| Beranda (belum ada data hari ini) | Card tetap tampil dengan 0; teks "Belum ada catatan hari ini. Mulai dari tombol di bawah." |
| Makanan | Ikon garpu · "Belum ada makanan" · "Catat makanan pertamamu hari ini." · [+ Tambah Makanan] |
| Latihan | Ikon lari · "Belum ada latihan" · "Catat manual atau pakai stopwatch." · [+ Tambah Latihan] |
| Berat | Ikon timbangan · "Belum ada data berat" · "Catat beratmu untuk melihat tren." · [+ Catat Berat] |
| Grafik < 2 data | "Tambah minimal 2 catatan untuk melihat tren." |

**Error validasi:** teks `danger` 14 px di bawah field, border field berubah `danger`, fokus pindah ke field error pertama. Tidak memakai toast untuk error form.

**Error database:** `Alert` "Gagal menyimpan data. Coba lagi." — form tidak ditutup sehingga input tidak hilang.

**Loading:** query lokal cepat; tidak perlu skeleton. Tombol simpan menampilkan spinner dan dinonaktifkan selama proses untuk mencegah data ganda.

**Sukses:** modal ditutup dan daftar langsung terbarui; tidak perlu toast.

## 7. Copy UI

Label ini dipakai juga oleh flow Maestro, jadi jangan diubah tanpa memperbarui test.

| Kunci | Teks |
|---|---|
| Tab | Beranda · Makanan · Latihan · Berat |
| Waktu makan | Sarapan · Makan Siang · Makan Malam · Camilan |
| Tombol onboarding | Mulai · Simpan |
| Simpan makanan | Simpan Makanan |
| Simpan latihan | Simpan Latihan |
| Simpan berat | Simpan |
| Simpan edit | Simpan Perubahan |
| Hapus | Hapus Makanan · Hapus Latihan · Hapus Berat |
| Stopwatch | Mulai · Jeda · Lanjut · Selesai |
| Rentang | 7 Hari · 30 Hari |
| Format kalori dashboard | `1.250 / 2.000 kkal` |

### Pesan Validasi

| Kondisi | Pesan |
|---|---|
| Wajib kosong | `<Label> wajib diisi` |
| Negatif | `<Label> tidak boleh negatif` |
| Di luar batas | `<Label> harus antara <min> dan <max> <satuan>` |
| Bukan angka | `<Label> harus berupa angka` |
| Porsi nol | `Jumlah porsi harus lebih dari 0` |
| Durasi nol | `Durasi minimal 1 menit` |

### Format

- Angka: `Intl.NumberFormat('id-ID')` → `2.000`, `72,5`.
- Tanggal panjang: `Senin, 24 November 2026`; pendek: `24 Nov`; jam 24-jam `12:30`.
- Durasi: `< 1 jam` → `45 mnt`; `≥ 1 jam` → `1 j 05 mnt`; stopwatch `HH:MM:SS`.

## 8. Aksesibilitas

- Area sentuh ≥ 48 dp; ikon tanpa teks wajib `accessibilityLabel` (mis. "Tambah makanan", "Pengaturan").
- Progress bar memakai `accessibilityRole="progressbar"` dan `accessibilityValue`.
- Jangan menyampaikan status hanya lewat warna — kelebihan kalori juga ditulis "Lebih 150 kkal".
- Teks mendukung Dynamic Type/font scale; layout card boleh membungkus ke baris baru.
- Grafik punya ringkasan teks (`accessibilityLabel` berisi berat terakhir dan selisih).

## 9. Checklist Review Desain

- [ ] Semua layar P0 di §5 disetujui tim.
- [ ] Token warna lolos cek kontras di mode terang & gelap.
- [ ] Setiap daftar punya empty state.
- [ ] Setiap hapus memakai dialog §5.6.
- [ ] Copy §7 sama dengan yang dipakai di flow Maestro.
- [ ] Ikon aplikasi & splash diganti dari template Expo (warna `primary`).
- [ ] Mockup high-fidelity (Figma, opsional) mengikuti wireframe ini.
