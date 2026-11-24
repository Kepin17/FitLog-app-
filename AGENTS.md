This is an Expo/React Native mobile application. Prioritize mobile-first patterns, performance, and cross-platform compatibility.

## Expo has changed — do not trust your training data

Expo ships breaking changes every SDK release. APIs you remember are likely renamed, moved, or removed. Before writing any code that touches an Expo, EAS, or React Native API:

1. Read the major version of the `expo` package in `package.json`.
2. Fetch the matching versioned docs: `https://docs.expo.dev/versions/v<major>.0.0/`
3. For anything else, fetch https://docs.expo.dev/llms.txt — an index of all Expo docs with corrections to common LLM misconceptions. Follow its links to the specific page you need; never answer from memory.

## Commands

Use `bunx` instead of `npx` if the project uses bun (`bun.lock` present).

```bash
npx expo install <package>  # ALWAYS use instead of npm/yarn/pnpm/bun add — resolves SDK-compatible versions
npx expo start              # start the dev server
npx expo lint               # lint
npx tsc --noEmit            # typecheck
npx expo-doctor             # diagnose dependency and config issues
npx expo install --fix      # fix incompatible package versions
```

Run lint and typecheck before declaring any task done.

## Navigation & Routing

- Use **Expo Router** for all navigation. Routes live in `src/app/` — every file there is a screen, `_layout.tsx` files define navigators. Keep non-route code (components, hooks, utils) outside `src/app/`.
- Import `Link`, `router`, and `useLocalSearchParams` from `expo-router`.
- Docs: https://docs.expo.dev/router/introduction.md

## Building with EAS

Use EAS to build, sign, and submit the app in the cloud (`eas build`, `eas submit`) and to ship over-the-air updates (`eas update`) — no local Xcode or Android Studio required. Run EAS CLI as `bunx eas-cli <command>` in Bun projects, or `npx eas-cli@latest <command>` otherwise; substitute that for bare `eas` in docs examples.
Docs: https://docs.expo.dev/eas/index.md

## Rules

- If `ios/` and `android/` directories do not exist, they are generated (Continuous Native Generation). Never create or edit them by hand — configure native behavior in `app.json` and config plugins.
- Expo Go only includes its bundled native modules. After adding a library with native code, the app needs a development build: `npx expo run:ios|android` locally, or `eas build --profile development`.
- Prefer recommended Expo modules over third-party libraries, and check your available skills before adding dependencies. Docs: https://docs.expo.dev/versions/latest/index.md

---

# FitLog — Project Context

FitLog adalah aplikasi pencatat makanan, latihan, dan berat badan harian (UAS Pemrograman Mobile). Penilaian berfokus pada **CRUD lengkap, validasi form, dashboard dari data asli, dan penyimpanan lokal**.

Dokumen sumber (baca sebelum mengerjakan fitur):

- [`docs/PRD.md`](docs/PRD.md) — requirement, skema data, pembagian tugas, rencana testing.
- [`docs/DESIGN.md`](docs/DESIGN.md) — design token, komponen, wireframe tiap layar, copy UI.

Jika kode dan dokumen bertentangan, ikuti dokumen lalu laporkan perbedaannya. Jangan menambah fitur di luar PRD.

> Status: tahap dokumentasi. Kode di `src/` masih template Expo (`index.tsx`, `explore.tsx`) dan akan diganti sesuai struktur di bawah.

## Stack (rencana)

| Kebutuhan | Pilihan | Catatan |
|---|---|---|
| Framework | Expo SDK 58, React Native 0.88, TypeScript | Sudah terpasang |
| Navigasi | Expo Router (`src/app/`), `NativeTabs` untuk tab utama | Sudah terpasang |
| Database lokal | `expo-sqlite` | Pasang via `npx expo install expo-sqlite` |
| Grafik berat | `react-native-svg` + garis sederhana buatan sendiri | Hindari library chart besar |
| Scan barcode (bonus) | `expo-camera` | Butuh development build |
| Pengingat (bonus) | `expo-notifications` | Butuh development build |
| Unit test | `jest-expo` | Fokus `src/lib/` dan `src/db/` |
| E2E | Maestro (`maestro/*.yaml`) | Pakai `testID` stabil |

Verifikasi API tiap paket ke docs versi SDK 58 sebelum dipakai (lihat aturan di atas).

## Struktur Folder Target

```
src/
  app/                        # HANYA route
    _layout.tsx               # Root Stack; redirect ke /onboarding bila daily_target kosong
    onboarding.tsx
    settings.tsx              # Edit target & berat awal
    (tabs)/
      _layout.tsx             # NativeTabs: Beranda, Makanan, Latihan, Berat
      index.tsx               # Dashboard harian
      food.tsx                # Daftar makanan per waktu makan
      workout.tsx             # Daftar sesi + ringkasan mingguan
      weight.tsx              # Riwayat + grafik 7/30 hari
    food/
      form.tsx                # Tambah/edit (?id= untuk edit), presentation: modal
      [id].tsx                # Detail + hapus
    workout/
      form.tsx                # Tambah/edit manual
      stopwatch.tsx
    weight/
      form.tsx
    scan.tsx                  # Bonus
  db/
    client.ts                 # Buka DB + migrasi (PRAGMA user_version)
    target.ts  food.ts  workout.ts  weight.ts   # Query per entitas
  lib/
    calc.ts                   # Total kalori/makro, sisa kalori, total mingguan, filter tren
    validate.ts               # Aturan validasi form (pure function)
    date.ts                   # Helper tanggal lokal, awal minggu Senin
    format.ts                 # Format angka id-ID ("2.000 kkal")
  components/                 # UI bersama (lihat DESIGN.md §Komponen)
  constants/theme.ts          # Design token
```

## Aturan Data (keputusan yang mengikat)

- **Kalori & makro di `food_entry` adalah per porsi.** Total entri = nilai × `servings`. Semua agregasi wajib lewat `src/lib/calc.ts`.
- Tanggal disimpan sebagai string ISO lokal. `weight_entry.date` berformat `YYYY-MM-DD` dengan constraint `UNIQUE` di SQLite — jangan hanya dicek di aplikasi.
- "Hari ini" dan "minggu ini" memakai zona waktu perangkat. Minggu = **Senin 00:00 – Minggu 23:59**.
- `duration_seconds` adalah durasi bersih (tanpa jeda). Stopwatch menghitung dari selisih timestamp, bukan dari jumlah tick interval.
- `daily_target` hanya satu baris (`id = 1`).
- Setiap tabel punya `created_at` dan `updated_at`; `updated_at` diperbarui di setiap UPDATE.
- Hapus selalu lewat dialog konfirmasi. Tidak ada soft delete.
- Tidak ada network kecuali lookup barcode (bonus).

## Validasi (usulan, sesuaikan bila tim memutuskan lain)

| Field | Aturan |
|---|---|
| Nama makanan / jenis latihan | Wajib, 1–60 karakter setelah trim |
| Kalori per porsi | Wajib, 0–5.000 |
| Protein/karbo/lemak | Opsional (default 0), 0–500 g |
| Porsi | Wajib, > 0 dan ≤ 20, kelipatan 0,25 |
| Target kalori | Wajib, 800–10.000 |
| Target makro | Opsional, 0–1.000 g |
| Berat badan / berat awal | Wajib, 20–300 kg, 1 desimal |
| Durasi manual | Wajib, 1 menit – 12 jam |

Input desimal menerima koma dan titik (`72,5` = `72.5`). Pesan error ditulis dalam Bahasa Indonesia di bawah field (lihat DESIGN.md).

## Kepemilikan Kode

| Anggota | File utama |
|---|---|
| Anggota 1 | `app/_layout.tsx`, `app/onboarding.tsx`, `app/settings.tsx`, `app/(tabs)/_layout.tsx`, `app/(tabs)/index.tsx`, `db/client.ts`, `db/target.ts`, `components/`, `constants/theme.ts` |
| Anggota 2 | `app/(tabs)/food.tsx`, `app/food/*`, `db/food.ts`, bagian makanan di `lib/calc.ts` & `lib/validate.ts` |
| Anggota 3 | `app/(tabs)/workout.tsx`, `app/(tabs)/weight.tsx`, `app/workout/*`, `app/weight/*`, `db/workout.ts`, `db/weight.ts`, bagian latihan/berat di `lib/calc.ts`, `maestro/` |

Saat diminta mengerjakan fitur satu anggota, jangan mengubah file anggota lain kecuali integrasi yang memang diperlukan, dan sebutkan perubahan tersebut.

## Konvensi

- Semua teks UI dalam Bahasa Indonesia; ambil copy dari DESIGN.md agar konsisten dengan flow Maestro.
- Elemen yang dipakai test diberi `testID` kebab-case: `food-form-name`, `food-form-save`, `dashboard-calorie-summary`, dll.
- Logika hitung & validasi berupa pure function di `src/lib/` dan wajib punya unit test.
- Layar membaca data ulang saat fokus (`useFocusEffect`) agar dashboard langsung berubah setelah CRUD.
- Jangan menyimpan data di state global yang tidak sinkron dengan DB; DB adalah sumber kebenaran.
- Fitur bonus (barcode, pengingat, riwayat makanan) hanya dikerjakan setelah checklist "Pengujian Minimum" di PRD lulus.

## Definition of Done

1. `npx expo lint` dan `npx tsc --noEmit` bersih.
2. Unit test untuk logika baru lulus.
3. Fitur jalan di Android (Expo Go/dev build) dan data tetap ada setelah app ditutup.
4. Empty state, error validasi, dan dialog konfirmasi hapus sesuai DESIGN.md.
