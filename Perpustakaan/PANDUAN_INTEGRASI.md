# Panduan Integrasi: Gabungkan Semua Modul Jadi Satu Simulasi Perpustakaan 3D

**Untuk 1 orang integrator** yang akan menggabungkan 6 modul yang sudah dikerjakan oleh anggota kelompok.

---

## Ringkasan

Setiap anggota kelompok mengerjakan 1 modul. Sekarang saatnya menggabungkan semuanya menjadi **1 file `.alp`** yang bisa di-run sebagai simulasi perpustakaan lengkap.

### Alur final setelah integrasi

```
srcParkirMobil ──┐
srcParkirMotor ──┤          ┌────────────────────────────────────────┐
                 ├──→ srvScanKTM → selectTujuan                     │
srcJalanKaki ────┘          │ (6 output)                            │
                            │                                        │
  out1 (25%) ─── srvPinjam ─┐                                       │
  out2 (15%) ─── srvKembali ─┤                                       │
  out3 (25%) ─── srvCariDuduk ┤ → wJalanKeluar → selectPulang       │
  out4 (10%) ─── srvToilet ──┤         ├─ isParkir? → wJalanBalikParkir
  out5 (10%) ─── srvFotokopi ┤         └─ tidak → snkSelesai        │
  out6 (15%) ─── langsung ───┘                                       │
                            └────────────────────────────────────────┘
```

**Konsep cepat untuk integrator:**
- Anda akan memulai dari **Modul 2 (Skeleton)** sebagai file utama
- **Copy-paste** komponen dari modul lain ke file ini
- **Sambungkan** semua blok sesuai alur di atas
- **Merge** semua variabel, fungsi, 3D objects, dan dashboard
- **Test** untuk memastikan semuanya jalan

---

## Prasyarat

**Yang harus sudah selesai dan siap diintegrasikan:**

| Modul | Isi | Status |
|---|---|---|
| Modul 1 | `srcParkirMobil`, `srcParkirMotor`, `srvParkirMobil`, `srvParkirMotor`, `jalanKePerpus`, `jalanKeParkiran` | Harus selesai |
| Modul 2 | `srcJalanKaki`, `srvScanKTM`, `selectTujuan`, `selectPulang`, `wJalanKeluar`, `snkSelesai` | **BASE FILE** |
| Modul 3 | `selectAktivitas`, `srvToiletSinggah`, `srvCariBuku`, `srvLoker`, `selectZonaDuduk`, `srvDudukSepi`, `srvDudukDiskusi` | Harus selesai |
| Modul 4 | `selectGenderToilet`, `srvToiletPria`, `srvToiletWanita` | Harus selesai |
| Modul 5 | `srvFotokopi` | Harus selesai |
| Modul 6 | `srvPinjam`, `srvKembali` | Harus selesai |

---

## Langkah 1: Mulai dari Skeleton (Modul 2)

1. Buka file `Perpustakaan3D_Skeleton.alp` (project Modul 2).
2. **File → Save As →** `Perpustakaan3D_Full.alp`.
3. Sekarang kita akan menambahkan semua komponen dari modul lain ke file ini.

> **Peringatan:** Jangan pernah mengedit file Asli skeleton. Selalu kerja di file `_Full`.

---

## Langkah 2: Merge Semua Variabel ke PengunjungPed

Buka diagram **PengunjungPed** (klik 2x di panel Projects). Tambahkan **semua variabel** dari tabel di bawah. Jika ada variabel yang sudah ada, jangan dibuat duplikat.

**Cara menambah variable:**
1. Dari palette **Agent**, drag **Variable** ke canvas.
2. Isi nama persis seperti di tabel (case-sensitive!).
3. Pilih type yang sesuai.
4. Isi initial value.

### Semua variabel PengunjungPed (30 variabel)

| Nama | Type | Initial | Modul Asal | Keterangan |
|---|---|---|---|---|
| `idPed` | String | `""` | Semua | ID unik pedestrian |
| `tMasuk` | double | `0` | Semua | Waktu masuk sistem |
| `isDosen` | boolean | `false` | Semua | `true`=dosen, `false`=mahasiswa |
| `isParkir` | boolean | `false` | Modul 1 | `true`=naik kendaraan |
| `jenisKendaraan` | String | `""` | Modul 1 | "MOBIL" atau "MOTOR" |
| `waktuParkir` | double | `0` | Modul 1 | Waktu mulai parkir |
| `noktp` | String | `""` | Modul 2 | Nomor KTM/KTP |
| `isValidKTM` | boolean | `false` | Modul 2 | `true`=KTM valid |
| `tujuanLayanan` | String | `""` | Modul 2 | Catatan tujuan: "PINJAM", dll |
| `jenisKelamin` | String | `""` | Modul 3, 4 | "PRIA" atau "WANITA" |
| `waktuMulaiAktivitas` | double | `0` | Modul 3 | Waktu mulai aktivitas sebelum duduk |
| `waktuMulaiDuduk` | double | `0` | Modul 3 | Waktu mulai duduk belajar |
| `durasiBelajar` | double | `0` | Modul 3 | Durasi belajar (menit) |
| `zonaDuduk` | String | `""` | Modul 3 | "SEPI" atau "DISKUSI" |
| `aktivitasSebelumDuduk` | String | `""` | Modul 3 | "TOILET", "BUKU", "LOKER", "LANGSUNG" |
| `waktuMulaiToilet` | double | `0` | Modul 4 | Waktu masuk bilik toilet |
| `jumlahHalaman` | int | `0` | Modul 5 | Jumlah halaman fotokopi/scan |
| `tipeLayananFotokopi` | String | `""` | Modul 5 | "FOTOKOPI" atau "SCAN" |
| `mesinRusak` | boolean | `false` | Modul 5 | `true`=mesin rusak |
| `waktuMulaiFotokopi` | double | `0` | Modul 5 | Waktu mulai fotokopi |
| `isPeminjaman` | boolean | `true` | Modul 6 | `true`=pinjam, `false`=kembali |
| `jumlahBuku` | int | `1` | Modul 6 | Jumlah buku |
| `tipePinjaman` | String | `"REGULER"` | Modul 6 | "REGULER"/"REFERENSI"/"RESERVE" |
| `deadlineHari` | int | `7` | Modul 6 | Deadline pengembalian (hari) |
| `nomorResi` | String | `""` | Modul 6 | Nomor bukti transaksi |
| `scannerError` | boolean | `false` | Modul 6 | `true`=scanner error |
| `waktuPinjam` | double | `0` | Modul 6 | Waktu transaksi pinjam |
| `waktuKembali` | double | `0` | Modul 6 | Waktu transaksi kembali |
| `jumlahHariTerlambat` | int | `0` | Modul 6 | Hari keterlambatan |
| `denda` | double | `0` | Modul 6 | Total denda |

> **Tips:** Buka semua diagram PengunjungPed dari modul 1-6 secara bersamaan. Bandingkan daftar variabel. Copy yang belum ada di file `_Full`.

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

Buka **Main**, dari palette **Agent** drag **Function** ke canvas untuk setiap fungsi berikut. Copy kode dari modul masing-masing.

### Semua fungsi (14 fungsi)

| Nama Fungsi | Return type | Parameter | Modul |
|---|---|---|---|
| `getInterarrivalTime()` | double | — | Modul 2 |
| `hitungWaktuScanKTM(ped)` | double | PengunjungPed ped | Modul 2 |
| `hitungWaktuCariParkirMobil()` | double | — | Modul 1 |
| `hitungWaktuCariParkirMotor()` | double | — | Modul 1 |
| `hitungDurasiBelajar(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuToiletSinggah(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuCariBuku(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuLoker(ped)` | double | PengunjungPed ped | Modul 3 |
| `hitungWaktuToilet()` | double | — | Modul 4 |
| `hitungWaktuFotokopi(ped)` | double | PengunjungPed ped | Modul 5 |
| `hitungWaktuServicePeminjaman(ped)` | double | PengunjungPed ped | Modul 6 |
| `hitungWaktuServicePengembalian(ped)` | double | PengunjungPed ped | Modul 6 |
| `avgWaktuSistem()` | double | — | Semua |
| `rataRataDenda()` | double | — | Modul 6 |

> **Tips penting:** Pastikan semua fungsi menggunakan parameter `PengunjungPed ped`, **bukan** `Agent agent`. Kalau pakai `Agent`, akan error saat runtime.

---

## Langkah 5: Tambah Markup dari Semua Modul

Di **Main**, buka area markup. Tambahkan **Service with Lines** berikut:

**Cara:**
1. Dari **Space Markup** → drag **Service with Lines** ke area yang sesuai.
2. Rename sesuai tabel.
3. Atur Number of services dan N of queues.

| Markup | Modul | Services | Queues | Posisi (X, Y) |
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

### Layout final 3D (ilustrasi koordinat)

```
Y=24  [ToiletWanita]                       [AreaDiskusi]
Y=20  [ToiletPria]    [Loker]   [AreaSepi]
      [ToiletSinggah] [RakBuku]
Y=16  [CariBuku]
      [Fotokopi]
Y=10  [entryLine]     [Peminjaman] [Pengembalian]
Y=8   [ScanKTM]
Y=4
Y=2
      X=2    X=6     X=10       X=14      X=18      X=22     X=26
```

---

## Langkah 6: Tambah dan Hubungkan Blok dari Modul Lain

### A. Dari Modul 1 (Parkir) — 2 blok sumber + 2 service + 2 PedGoTo

**Blok yang perlu ditambahkan dari Modul 1 ke file `_Full`:**

Copy dari project Modul 1 atau buat ulang:

| Blok | Colok dari | Colok ke |
|---|---|---|
| `srcParkirMobil` | — | `srvParkirMobil.in` |
| `srvParkirMobil` | `srcParkirMobil.out` | `jalanKePerpus.in` |
| `srcParkirMotor` | — | `srvParkirMotor.in` |
| `srvParkirMotor` | `srcParkirMotor.out` | `jalanKePerpus.in` |
| `jalanKePerpus` | `srvParkirMobil.out` + `srvParkirMotor.out` | **`srvScanKTM.in`** |
| `jalanKeParkiran` | **`wJalanBalikParkir.out`** (dari skeleton) | `snkParkir.in` (jika ada) |

> **Hub penting:** `jalanKePerpus.out` colok ke `srvScanKTM.in` (bukan ke sink — karena setelah parkir, orang harus scan KTM dulu).

### B. Dari Modul 2 (Skeleton) — sudah ada di file

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
| (langsung) | `selectAktivitas.out4` langsung | `selectZonaDuduk.in` |
| `selectZonaDuduk` | dari 4 sumber | ke srvDudukSepi/srvDudukDiskusi |
| `srvDudukSepi` | `selectZonaDuduk.out1` | **`wJalanKeluar.in`** |
| `srvDudukDiskusi` | `selectZonaDuduk.out2` | **`wJalanKeluar.in`** |

> **HAPUS** `srcMasuk` dan `snkSelesai` temporary dari Modul 3.

### D. Dari Modul 4 (Antrian Toilet)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `selectGenderToilet` | `selectTujuan.out4` | ke srvToiletPria/Wanita |
| `srvToiletPria` | `selectGenderToilet.out1` | **`wJalanKeluar.in`** |
| `srvToiletWanita` | `selectGenderToilet.out2` | **`wJalanKeluar.in`** |

> **HAPUS** `srcMasuk` dan `snkSelesai` temporary dari Modul 4.

### E. Dari Modul 5 (Fotokopi/Scan)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `srvFotokopi` | `selectTujuan.out5` | **`wJalanKeluar.in`** |

> **HAPUS** `srcMasuk` dan `snkSelesai` temporary dari Modul 5.

### F. Dari Modul 6 (Peminjaman & Pengembalian)

| Blok | Colok dari | Colok ke |
|---|---|---|
| `srvPinjam` | `selectTujuan.out1` | **`wJalanKeluar.in`** |
| `srvKembali` | `selectTujuan.out2` | **`wJalanKeluar.in`** |

> **HAPUS** `srcMasuk`, `snkSelesai`, dan `selectJenisLayanan` temporary dari Modul 6. Routing pinjam/kembali langsung dari `selectTujuan`.

---

## Langkah 7: Konfigurasi Routing Final

### selectTujuan (6 outputs — probabilitas)

Pastikan **Use probabilities** di-centang.

| Output | Tujuan | Probabilitas |
|---|---|---|
| out1 | `srvPinjam.in` (Modul 6) | 0.25 |
| out2 | `srvKembali.in` (Modul 6) | 0.15 |
| out3 | `selectAktivitas.in` (Modul 3) | 0.25 |
| out4 | `selectGenderToilet.in` (Modul 4) | 0.10 |
| out5 | `srvFotokopi.in` (Modul 5) | 0.10 |
| out6 | `wJalanKeluar.in` (langsung) | 0.15 |

**Verifikasi:** 0.25 + 0.15 + 0.25 + 0.10 + 0.10 + 0.15 = **1.0** ✅

### selectPulang (2 outputs — KONDISI, bukan probabilitas)

Pastikan **Use conditions** di-centang, **Use probabilities** di-uncentang.

| Output | Kondisi | Tujuan |
|---|---|---|
| out1 | `ped.isParkir == true` | `wJalanBalikParkir.in` → ke parkir |
| out2 | `ped.isParkir == false` | `snkSelesai.in` → keluar |

---

## Langkah 8: Merge 3D Objects

Gabungkan semua objek 3D di Main dengan mengatur **posisi** masing-masing agar tidak bertabrakan.

| Objek 3D | Modul | Posisi X | Posisi Y | Ukuran/Skala |
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

**Cara:**
1. Buka diagram Main.
2. Copy objek 3D dari project modul asal (Ctrl+C, Ctrl+V) atau buat ulang.
3. Atur posisi X, Y sesuai tabel.
4. Jika ada objek yang bertumpuk, geser sedikit.

---

## Langkah 9: Merge Dashboard Final

Tambahkan **Text** dinamis di Main untuk dashboard lengkap:

**Cara:** Palette **Presentation** → drag **Text** → set **Type** = `Dynamic`.

```java
// Judul
"SIMULASI PERPUSTAKAAN 3D — Terintegrasi"
```

```java
// Total pengunjung
"Total: " + totalSelesai + " | Mhs: " + totalMahasiswa + " | Dosen: " + totalDosen
```

```java
// PARKIR
"PARKIR — Mobil: " + totalMobil + " | Motor: " + totalMotor
```

```java
// PEMINJAMAN
"PEMINJAMAN — Pinjam: " + totalPeminjaman + " | Kembali: " + totalPengembalian
+ "  Antrian: " + srvPinjam.queueSize() + "/" + srvKembali.queueSize()
+ " | Buku: " + totalBukuDipinjam
+ "  Error: " + totalErrorScanner + " | Denda: Rp " + String.format("%,.0f", totalDenda)
```

```java
// BELAJAR
"BELAJAR — Total: " + (totalDudukSepi + totalDudukDiskusi)
+ "  Sepi: " + totalDudukSepi + " | Diskusi: " + totalDudukDiskusi
+ "  Antrian sepi: " + srvDudukSepi.queueSize() + " | diskusi: " + srvDudukDiskusi.queueSize()
+ "  Toilet: " + totalToiletSinggah + " | Buku: " + totalCariBuku + " | Loker: " + totalLoker
```

```java
// TOILET
"TOILET — Pria: " + totalToiletPria + " | Wanita: " + totalToiletWanita
+ "  Antrian pria: " + srvToiletPria.queueSize() + " | wanita: " + srvToiletWanita.queueSize()
```

```java
// FOTOKOPI
"FOTOKOPI — Fotokopi: " + totalFotokopi + " | Scan: " + totalScan
+ "  Antrian: " + srvFotokopi.queueSize() + " | Mesin rusak: " + totalMesinRusak
```

```java
// Rata-rata waktu sistem
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## Langkah 10: Testing Integrasi

### Test 1: Run 30 menit (semua route)

1. **Stop time** = `30` menit.
2. Klik **Run**.
3. Cek console: apakah semua jenis layanan muncul?
   - `PINJAM`, `KEMBALI`, `BELAJAR`, `TOILET`, `FOTOKOPI`, `LANGSUNG`
4. Cek visual 3D: pedestrian berjalan natural, antri di tempat yang benar.

### Test 2: Run 60 menit (peak hour)

1. **Stop time** = `60` menit.
2. Pantau:
   - Peak hour di menit 20-50 → antrian mengular di layanan populer
   - Saat menit 50+ → antrian mulai mengecil
   - Tidak ada error stack trace di console

### Test 3: Uji parkir (round-trip)

1. Pastikan ada pedestrian dengan `isParkir=true` (dari `srcParkirMobil` / `srcParkirMotor`).
2. Mereka harus melalui: parkir → jalan ke perpus → scan KTM → pilih tujuan → service → selectPulang → balik ke parkir.
3. Yang `isParkir=false` (dari `srcJalanKaki`) → langsung keluar via `snkSelesai`.

---

## Troubleshooting Integrasi

### Error: "Duplicate name 'xxx'"

Ada dua blok dengan nama sama di model.

**Solusi:** Cari nama yang duplikat (buka panel Projects, cari nama yang muncul 2x). Rename salah satunya.

### Error: "Agent cannot be cast to PengunjungPed"

Ada blok PedService yang masih memakai parameter `agent` bukan `ped`.

**Solusi:** Cek semua Delay time di semua PedService. Pastikan formatnya:
- ✅ `hitungWaktuServicePeminjaman(ped)`
- ❌ `hitungWaktuServicePeminjaman(agent)`

### Error: "Function xxx not found"

Fungsi yang dipanggil di Delay time belum dibuat di Main.

**Solusi:** Cek daftar fungsi di Langkah 4. Buat fungsi yang kurang.

### Error: "Service xxx not found"

Blok PedService menggunakan markup yang belum ada di model.

**Solusi:** Cek semua PedService → pastikan field `Services` pointing ke markup Service with Lines yang sudah dibuat.

### Antrian tidak terlihat / kosong melulu

1. Cek probabilitas `selectTujuan` — total harus **1.0**.
2. Cek koneksi — pastikan semua output terhubung ke blok yang benar.
3. Jika ada output yang tidak terkoneksi, AnyLogic bisa warning atau error.

### Semua orang balik ke parkir (tidak ada yang keluar)

**Penyebab:** `selectPulang` menggunakan **probabilities** bukan **conditions**.

**Solusi:** Cek `selectPulang`:
- **Use conditions** = centang
- **Use probabilities** = jangan centang
- Output 1 condition: `ped.isParkir == true`
- Output 2 condition: `ped.isParkir == false`

### Service point ketutup objek 3D

**Solusi:** Pindahkan Box 3D atau kecilkan ukurannya. Service point lingkaran hijau harus terlihat.

### Console tidak menampilkan log

**Solusi:** Pastikan `traceln(...)` ada di action tiap blok. Jika terlalu banyak log, bisa memperlambat simulasi.

---

## Checklist Final Integrasi

### Koneksi blok
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
- [ ] Run 60 menit: peak hour terlihat di menit 20-50
- [ ] Console tidak ada error stack trace
- [ ] Parkir → perpus → layanan → balik parkir berfungsi
- [ ] Jalan kaki → perpus → layanan → keluar berfungsi
