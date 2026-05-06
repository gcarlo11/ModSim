# Panduan Integrasi: Gabungkan Semua Modul Jadi Satu Simulasi Perpustakaan 3D

## Flowchart Final Setelah Integrasi

```
srcParkirMobil (Modul 1) ──┐
srcParkirMotor (Modul 1) ──┤          ┌────────────────────────────────────────┐
                            ├──→ srvScanKTM (Modul 2) ──→ selectTujuan       │
srcJalanKaki (Modul 2) ─────┘          │ (6 output)                           │
                                       │                                      │
  out1 (25%) ───── srvPinjam ─────┐    │  Modul 6         │                  │
  out2 (15%) ───── srvKembali ────┤    │    │                                │
  out3 (25%) ───── srvCariDuduk ──┤ → wJalanKeluar → selectPulang            │
  out4 (10%) ───── srvToilet ─────┤    │         ├─ isParkir? → wJalanBalikParkir
  out5 (10%) ───── srvFotokopi ───┤    │  (Modul 3-5)│                       │
  out6 (15%) ───── langsung ──────┘    └───────────────┘  └─ tidak → snkSelesai
```

---

## Dokumentasi Ini Untuk Siapa

Untuk **1 orang integrator** yang akan menggabungkan 6 modul yang sudah dikerjakan oleh anggota kelompok.

**Yang harus sudah selesai:**
- [ ] Modul 1 (Parkir) — `srcParkirMobil`, `srcParkirMotor`, `srvParkirMobil`, `srvParkirMotor`
- [ ] Modul 2 (Kedatangan & Kepergian) — skeleton dengan `srcJalanKaki`, `srvScanKTM`, `selectTujuan`, `selectPulang`, `wJalanKeluar`, `snkSelesai`
- [ ] Modul 3 (Cari Tempat Duduk) — `selectAktivitas`, `srvToiletSinggah`, `srvCariBuku`, `srvLoker`, `selectZonaDuduk`, `srvDudukSepi`, `srvDudukDiskusi`
- [ ] Modul 4 (Antrian Toilet) — `selectGenderToilet`, `srvToiletPria`, `srvToiletWanita`
- [ ] Modul 5 (Fotokopi/Scan) — `srvFotokopi`
- [ ] Modul 6 (Pinjam/Kembali) — `selectJenisLayanan`, `srvPinjam`, `srvKembali`

---

## Langkah 1: Mulai dari Skeleton (Modul 2)

1. Buka file `Perpustakaan3D_Skeleton.alp` (project Modul 2).
2. **File → Save As →** `Perpustakaan3D_Full.alp`.
3. Sekarang kita akan menambahkan semua komponen dari modul lain ke file ini.

---

## Langkah 2: Merge Semua Variabel ke PengunjungPed

Buka diagram **PengunjungPed**. Pastikan **semua variabel dari semua modul** ada:

| Nama | Type | Initial | Modul Asal |
|---|---|---|---|
| `idPed` | String | `""` | Semua |
| `tMasuk` | double | `0` | Semua |
| `isDosen` | boolean | `false` | Semua |
| `isParkir` | boolean | `false` | Modul 1 |
| `jenisKendaraan` | String | `""` | Modul 1 |
| `waktuParkir` | double | `0` | Modul 1 |
| `noktp` | String | `""` | Modul 2 |
| `isValidKTM` | boolean | `false` | Modul 2 |
| `tujuanLayanan` | String | `""` | Modul 2 |
| `jenisKelamin` | String | `""` | Modul 3, 4 |
| `waktuMulaiAktivitas` | double | `0` | Modul 3 |
| `waktuMulaiDuduk` | double | `0` | Modul 3 |
| `durasiBelajar` | double | `0` | Modul 3 |
| `zonaDuduk` | String | `""` | Modul 3 |
| `aktivitasSebelumDuduk` | String | `""` | Modul 3 |
| `waktuMulaiToilet` | double | `0` | Modul 4 |
| `jumlahHalaman` | int | `0` | Modul 5 |
| `tipeLayananFotokopi` | String | `""` | Modul 5 |
| `mesinRusak` | boolean | `false` | Modul 5 |
| `waktuMulaiFotokopi` | double | `0` | Modul 5 |
| `isPeminjaman` | boolean | `true` | Modul 6 |
| `jumlahBuku` | int | `1` | Modul 6 |
| `tipePinjaman` | String | `"REGULER"` | Modul 6 |
| `deadlineHari` | int | `7` | Modul 6 |
| `nomorResi` | String | `""` | Modul 6 |
| `scannerError` | boolean | `false` | Modul 6 |
| `waktuPinjam` | double | `0` | Modul 6 |
| `waktuKembali` | double | `0` | Modul 6 |
| `jumlahHariTerlambat` | int | `0` | Modul 6 |
| `denda` | double | `0` | Modul 6 |

---

## Langkah 3: Merge Semua Variabel di Main

Buka **Main**. Tambahkan semua variabel dari tabel di bawah. Total ~35 variabel.

| Nama | Type | Initial | Modul |
|---|---|---|---|
| `seqPed` | int | 0 | Modul 2 |
| `totalSelesai` | int | 0 | Modul 2 |
| `totalWaktuSistem` | double | 0 | Modul 2 |
| `totalMahasiswa` | int | 0 | Modul 2 |
| `totalDosen` | int | 0 | Modul 2 |
| `totalMobil` | int | 0 | Modul 1 |
| `totalMotor` | int | 0 | Modul 1 |
| `totalToiletSinggah` | int | 0 | Modul 3 |
| `totalCariBuku` | int | 0 | Modul 3 |
| `totalLoker` | int | 0 | Modul 3 |
| `totalDudukSepi` | int | 0 | Modul 3 |
| `totalDudukDiskusi` | int | 0 | Modul 3 |
| `maxAntrianDudukSepi` | int | 0 | Modul 3 |
| `maxAntrianDudukDiskusi` | int | 0 | Modul 3 |
| `totalDurasiBelajar` | double | 0 | Modul 3 |
| `totalToiletPria` | int | 0 | Modul 4 |
| `totalToiletWanita` | int | 0 | Modul 4 |
| `maxAntrianToiletPria` | int | 0 | Modul 4 |
| `maxAntrianToiletWanita` | int | 0 | Modul 4 |
| `totalFotokopi` | int | 0 | Modul 5 |
| `totalScan` | int | 0 | Modul 5 |
| `maxAntrianFotokopi` | int | 0 | Modul 5 |
| `totalMesinRusak` | int | 0 | Modul 5 |
| `totalWaktuRepair` | double | 0 | Modul 5 |
| `totalHalamanDiproses` | int | 0 | Modul 5 |
| `totalPeminjaman` | int | 0 | Modul 6 |
| `totalPengembalian` | int | 0 | Modul 6 |
| `totalDenda` | double | 0 | Modul 6 |
| `totalBukuRusak` | int | 0 | Modul 6 |
| `maxQueuePeminjaman` | int | 0 | Modul 6 |
| `maxQueuePengembalian` | int | 0 | Modul 6 |
| `totalBukuDipinjam` | int | 0 | Modul 6 |
| `totalErrorScanner` | int | 0 | Modul 6 |

---

## Langkah 4: Merge Semua Fungsi di Main

Buka **Main**, tambah **Function** untuk setiap fungsi berikut:

| Nama Fungsi | Return type | Parameter | Modul |
|---|---|---|---|
| `getInterarrivalTime()` | double | — | Modul 2 |
| `hitungWaktuScanKTM(ped)` | double | PengunjungPed ped | Modul 2 |
| `avgWaktuSistem()` | double | — | Modul 2 |
| `hitungDurasiBelajar(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuToiletSinggah(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuCariBuku(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuLoker(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuToilet()` | double | — | Modul 4 |
| `hitungWaktuFotokopi(ped)` | double | PengunjungPed ped | Modul 5 |
| `hitungWaktuServicePeminjaman(ped)` | double | PengunjungPed ped | Modul 6 |
| `hitungWaktuServicePengembalian(ped)` | double | PengunjungPed ped | Modul 6 |
| `avgBukuPerTransaksi()` | double | — | Modul 6 |
| `errorScannerRate()` | double | — | Modul 6 |
| `rataRataDenda()` | double | — | Modul 6 |

**Salin kode fungsi dari masing-masing modul tutorial.**

---

## Langkah 5: Tambah Markup dari Modul Lain

Di **Main**, buka area markup. Tambahkan **Service with Lines** berikut:

| Markup | Modul | Services | Queues | Posisi (contoh) |
|---|---|---|---|---|
| `svcScanKTM` | Modul 2 | 2 | 2 | (10, 3) — dekat entry |
| `svcToiletPria` | Modul 4 | 4 | 1 | (2, 16) — pojok kiri bawah |
| `svcToiletWanita` | Modul 4 | 4 | 1 | (2, 22) — di atas toilet pria |
| `svcToiletSinggah` | Modul 3 | 2 | 1 | (6, 16) — samping toilet |
| `svcCariBuku` | Modul 3 | 2 | 2 | (6, 20) — area tengah |
| `svcLoker` | Modul 3 | 3 | 1 | (10, 20) — dekat rak buku |
| `svcAreaSepi` | Modul 3 | 10 | 2 | (16, 16) — area kanan bawah |
| `svcAreaDiskusi` | Modul 3 | 4 | 2 | (16, 22) — area kanan atas |
| `svcFotokopi` | Modul 5 | 2 | 1 | (10, 8) — tengah |
| `svcPeminjaman` | Modul 6 | 2 | 2 | (12, 4) — kiri bawah |
| `svcPengembalian` | Modul 6 | 1 | 1 | (18, 4) — kanan bawah |

### Layout final 3D (contoh)

```
  Y
  ^
  24  [ToiletWanita]  [Diskusi Zone]           exitLine
  20  [ToiletPria]    [Loker] [AreaSpi]
      [ToiletSg]      [RakBuku]
  16  [CariBuku]      [Fotokopi]
  10  [entryLine]     [Peminjaman] [Pengembalian]
   8  [ScanKTM]
   4
   2
   └─────────────────────────────────────────→ X
      2    6    10    14    18    22    26    30
```

---

## Langkah 6: Tambah dan Hubungkan Blok

### A. Dari Modul 1 (Parkir) — 2 blok

Drag dari Pedestrian Library:

| Blok | Rename | Colok dari | Colok ke |
|---|---|---|---|
| PedSource | `srcParkirMobil` | — | `srvParkirMobil.in` |
| PedService | `srvParkirMobil` | `srcParkirMobil.out` | `jalanKePerpus.in` |
| PedSource | `srcParkirMotor` | — | `srvParkirMotor.in` |
| PedService | `srvParkirMotor` | `srcParkirMotor.out` | `jalanKePerpus.in` |
| PedGoTo | `jalanKePerpus` | `srvParkirMobil.out` + `srvParkirMotor.out` | **`srvScanKTM.in`** |
| PedGoTo | `jalanKeParkiran` | **`selectPulang.out1`** | `snkParkir.in` |

> **Hub penting:** `jalanKePerpus.out` colok ke `srvScanKTM.in` (bukan ke sink — karena setelah parkir, orang harus scan KTM dulu).

### B. Dari Modul 2 (Kedatangan & Kepergian) — sudah ada

```
srcJalanKaki.out → srvScanKTM.in
srvScanKTM.out → selectTujuan.in
wJalanKeluar.out → selectPulang.in
selectPulang.out2 (isParkir=false) → snkSelesai.in
selectPulang.out1 (isParkir=true) → wJalanBalikParkir.in → jalanKeParkiran.in
```

### C. Dari Modul 3 (Cari Tempat Duduk)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `selectAktivitas` | `selectTujuan.out3` | ke 4 aktivitas |
| `srvToiletSinggah` | `selectAktivitas.out1` | `selectZonaDuduk.in` |
| `srvCariBuku` | `selectAktivitas.out2` | `selectZonaDuduk.in` |
| `srvLoker` | `selectAktivitas.out3` | `selectZonaDuduk.in` |
| (langsung) | `selectAktivitas.out4` | `selectZonaDuduk.in` |
| `selectZonaDuduk` | sebelumnya | ke srvDudukSepi/srvDudukDiskusi |
| `srvDudukSepi` | `selectZonaDuduk.out1` | **`wJalanKeluar.in`** |
| `srvDudukDiskusi` | `selectZonaDuduk.out2` | **`wJalanKeluar.in`** |

> **HAPUS** srcMasuk dan snkSelesai temporary dari Modul 3.

### D. Dari Modul 4 (Antrian Toilet)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `selectGenderToilet` | `selectTujuan.out4` | ke srvToiletPria/Wanita |
| `srvToiletPria` | `selectGenderToilet.out1` | **`wJalanKeluar.in`** |
| `srvToiletWanita` | `selectGenderToilet.out2` | **`wJalanKeluar.in`** |

> **HAPUS** srcMasuk dan snkSelesai temporary dari Modul 4.

### E. Dari Modul 5 (Fotokopi/Scan)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `srvFotokopi` | `selectTujuan.out5` | **`wJalanKeluar.in`** |

> **HAPUS** srcMasuk dan snkSelesai temporary dari Modul 5.

### F. Dari Modul 6 (Peminjaman & Pengembalian)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `selectJenisLayanan` | `selectTujuan.out1` (pinjam) + `out2` (kembali) | — |
| `srvPinjam` | `selectJenisLayanan.out1` | **`wJalanKeluar.in`** |
| `srvKembali` | `selectJenisLayanan.out2` | **`wJalanKeluar.in`** |

> Atau jika ingin routing langsung: `selectTujuan.out1 → srvPinjam.in`, `selectTujuan.out2 → srvKembali.in`. Hapus `selectJenisLayanan`.

---

## Langkah 7: Konfigurasi routing final

### selectTujuan (6 outputs)

| Output | Tujuan | Probabilitas |
|---|---|---|
| out1 | `srvPinjam` | 0.25 |
| out2 | `srvKembali` | 0.15 |
| out3 | `srvCariDuduk` (via selectAktivitas) | 0.25 |
| out4 | `selectGenderToilet` | 0.10 |
| out5 | `srvFotokopi` | 0.10 |
| out6 | `wJalanKeluar` (langsung) | 0.15 |

### selectPulang (2 outputs — kondisi)

| Output | Kondisi | Tujuan |
|---|---|---|
| out1 | `ped.isParkir == true` | `jalanKeParkiran.in` |
| out2 | `ped.isParkir == false` | `snkSelesai.in` |

---

## Langkah 8: Merge 3D Objects

Gabungkan semua objek 3D di Main dengan mengatur **posisi** masing-masing agar tidak bertabrakan.

| Objek 3D | Modul | Posisi X | Posisi Y | Skala |
|---|---|---|---|---|
| `floor3D` | 2 | 15 | 12 | 30x24x0.2 |
| `mejaResepsionis3D` | 2 | 10 | 3 | 3x1x1.2 |
| `lantaiParkir3D` | 1 | 15 | 0 | 20x6x0.2 |
| `mejaPinjam3D` | 6 | 13 | 4 | 3x1x1.2 |
| `mejaKembali3D` | 6 | 19 | 4 | 3x1x1.2 |
| `rakBuku3D_1` | 3 | 8 | 18 | 2x4x2 |
| `rakBuku3D_2` | 3 | 8 | 21 | 2x4x2 |
| `loker3D` | 3 | 12 | 20 | 2x0.5x2 |
| `mejaSepi3D_1` s/d `_10` | 3 | 16-24 | 17-20 | 1.5x1x1.2 |
| `mejaDiskusi3D_1` s/d `_4` | 3 | 18-24 | 21-23 | 2x2x1.2 |
| `bilikToiletPria_1` s/d `_4` | 4 | 3-6 | 16-19 | 1x1x2 |
| `bilikToiletWanita_1` s/d `_4` | 4 | 3-6 | 20-23 | 1x1x2 |
| `mesinFotokopi1_3D` | 5 | 10 | 10 | 2x1x1.5 |
| `mesinFotokopi2_3D` | 5 | 13 | 10 | 2x1x1.5 |

---

## Langkah 9: Merge Dashboard Final

Tambahkan Text dinamis di Main untuk **dashboard lengkap**:

```
"SIMULASI PERPUSTAKAAN 3D — Terintegrasi"
```

```
"Total pengunjung: " + totalSelesai + " | Mhs: " + totalMahasiswa + " | Dosen: " + totalDosen
```

```
"PARKIR — Mobil: " + totalMobil + " | Motor: " + totalMotor
```

```
"PEMINJAMAN — Pinjam: " + totalPeminjaman + " | Kembali: " + totalPengembalian
"  Antrian: " + srvPinjam.queueSize() + "/" + srvKembali.queueSize() + " | Buku: " + totalBukuDipinjam
"  Error: " + totalErrorScanner + " | Denda: Rp " + String.format("%,.0f", totalDenda)
```

```
"BELAJAR — Total: " + (totalDudukSepi + totalDudukDiskusi)
"  Sepi: " + totalDudukSepi + " | Diskusi: " + totalDudukDiskusi
"  Antrian sepi: " + srvDudukSepi.queueSize() + " | diskusi: " + srvDudukDiskusi.queueSize()
"  Toilet(aktivitas): " + totalToiletSinggah + " | Cari buku: " + totalCariBuku + " | Loker: " + totalLoker
```

```
"TOILET — Pria: " + totalToiletPria + " | Wanita: " + totalToiletWanita
"  Antrian pria: " + srvToiletPria.queueSize() + " | wanita: " + srvToiletWanita.queueSize()
```

```
"FOTOKOPI — Fotokopi: " + totalFotokopi + " | Scan: " + totalScan
"  Antrian: " + srvFotokopi.queueSize() + " | Mesin rusak: " + totalMesinRusak
```

```
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## Langkah 10: Testing Integrasi

### Test 1: Run 30 menit (semua route)

1. Stop time = 30 menit.
2. Run.
3. Cek console: semua jenis layanan muncul (PINJAM, KEMBALI, BELAJAR, TOILET, FOTOKOPI, LANGSUNG).
4. Cek visual: pedestrian berjalan natural, antri di tempat yang benar.

### Test 2: Run 60 menit (peak hour)

1. Stop time = 60 menit.
2. Pantau:
   - Peak hour di menit 20-50 → antrian mengular di layanan populer.
   - Saat menit 50+ → antrian mulai mengecil.
   - Tidak ada error di console.

### Test 3: Uji parkir

1. Pastikan ada pedestrian dengan `isParkir=true` (dari srcParkirMobil/srcParkirMotor).
2. Mereka harus melalui: parkir → jalan ke perpus → scan KTM → pilih tujuan → service → selectPulang → balik ke parkir.
3. Yang `isParkir=false` → langsung keluar via snkSelesai.

---

## Checklist Final Integrasi

### Koneksi
- [ ] `srcParkirMobil.out → srvParkirMobil.in → jalanKePerpus.in`
- [ ] `srcParkirMotor.out → srvParkirMotor.in → jalanKePerpus.in`
- [ ] `jalanKePerpus.out → srvScanKTM.in`
- [ ] `srcJalanKaki.out → srvScanKTM.in`
- [ ] `srvScanKTM.out → selectTujuan.in`
- [ ] `selectTujuan.out1 → srvPinjam.in`
- [ ] `selectTujuan.out2 → srvKembali.in`
- [ ] `selectTujuan.out3 → selectAktivitas.in`
- [ ] `selectTujuan.out4 → selectGenderToilet.in`
- [ ] `selectTujuan.out5 → srvFotokopi.in`
- [ ] `selectTujuan.out6 → wJalanKeluar.in`
- [ ] Semua service output → `wJalanKeluar.in`
- [ ] `wJalanKeluar.out → selectPulang.in`
- [ ] `selectPulang.out1 → wJalanBalikParkir.in → jalanKeParkiran.in`
- [ ] `selectPulang.out2 → snkSelesai.in`

### Markup & Layout
- [ ] Semua 11 Service with Lines sudah ada di Main
- [ ] Service point tidak ketutup objek 3D
- [ ] Tidak ada overlap layout (semua posisi unik)

### Variabel & Fungsi
- [ ] Semua 30 variabel sudah ada di PengunjungPed
- [ ] Semua ~35 variabel sudah ada di Main
- [ ] Semua 14 fungsi sudah ada di Main
- [ ] Tidak ada duplikasi nama

### Dashboard
- [ ] Semua statistik dari semua modul muncul
- [ ] Tidak ada error "variable not found"
- [ ] Angka berubah saat run

### Testing
- [ ] Run 30 menit: semua jenis layanan terlihat
- [ ] Run 60 menit: peak hour terlihat
- [ ] Console tidak ada error stack trace
- [ ] Parkir → perpus → layanan → balik parkir berfungsi
- [ ] Jalan kaki → perpus → layanan → keluar berfungsi

---

## Troubleshooting Integrasi

### Error: Duplicate name

Ada dua blok dengan nama sama. Cari nama yang duplikat dan rename.

### Error: Agent cannot be cast to PengunjungPed

Ada blok PedService yang masih memakai parameter `agent` bukan `ped`. Cek semua Delay time:
- `hitungWaktuServicePeminjaman(ped)`
- `hitungWaktuServicePengembalian(ped)`
- dll.

### Error: Function not found

Fungsi yang dipanggil di Delay time belum dibuat di Main. Cek daftar fungsi di Langkah 4.

### Antrian tidak terlihat/kosong melulu

1. Cek probabilitas `selectTujuan` — total harus 1.0.
2. Cek koneksi — pastikan semua output terhubung ke blok yang benar.

### Semua orang balik ke parkir

Cek `selectPulang`: harus **conditions** bukan probabilities. Output 1: `ped.isParkir == true`, Output 2: `ped.isParkir == false`.

### Report selesai

Setelah semua langkah selesai dan testing lolos, model terintegrasi siap untuk demo dan laporan akhir!
