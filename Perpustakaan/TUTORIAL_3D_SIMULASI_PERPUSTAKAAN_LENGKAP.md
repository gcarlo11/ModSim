# Tutorial AnyLogic Lengkap: Simulasi Perpustakaan 3D
## Dua Counter (Peminjaman & Pengembalian) + Peak Hour + Dua Tipe Pengunjung + Sistem Denda

**Versi tutorial:** 2.0 (pengembangan dari `TUTORIAL_3D_ANTRE_COUNTER_PEMULA_ANYLOGIC.md`)

**Status verifikasi:** Dokumentasi resmi AnyLogic dicek pada April 2026.

---

## Daftar Isi

0. [Apa yang Baru?](#0-apa-yang-baru)
1. [Hasil Akhir](#1-hasil-akhir)
2. [Perbedaan dengan Versi Sebelumnya](#2-perbedaan-dengan-versi-sebelumnya)
3. [Buat Project Baru / Migrasi](#3-buat-project-baru--migrasi)
4. [Update Pedestrian Type (MahasiswaPed)](#4-update-pedestrian-type-mahasiswaped)
5. [Siapkan Layout 3D dengan Dua Counter](#5-siapkan-layout-3d-dengan-dua-counter)
6. [Buat Markup Pedestrian (Paling Penting)](#6-buat-markup-pedestrian-paling-penting)
7. [Bangun Flowchart Pedestrian](#7-bangun-flowchart-pedestrian)
8. [Konfigurasi Detail Setiap Blok](#8-konfigurasi-detail-setiap-blok)
9. [Variabel Statistik Baru di Main](#9-variabel-statistik-baru-di-main)
10. [Fungsi-Fungsi Baru di Main](#10-fungsi-fungsi-baru-di-main)
11. [Dashboard Lengkap](#11-dashboard-lengkap)
12. [Jalankan Simulasi](#12-jalankan-simulasi)
13. [Troubleshooting](#13-troubleshooting)
14. [Checklist Final](#14-checklist-final)
15. [Referensi](#15-referensi)

---

## 0. Apa yang Baru?

Jika kamu sudah mengerjakan tutorial dasar (`TUTORIAL_3D_ANTRE_COUNTER_PEMULA_ANYLOGIC.md`), berikut fitur **baru** di versi ini:

| Fitur | Tutorial Dasar | Tutorial Lengkap Ini |
|---|---|---|
| Counter | 1 layanan (peminjaman) | 2 layanan: peminjaman + pengembalian |
| Tipe pengunjung | Mahasiswa saja | Mahasiswa (80%) + Dosen (20%) |
| Kedatangan | Constant rate (`exponential(1.8)`) | **Peak hour**: sepi -> ramai -> normal |
| Antrean | 1 antrean | 2 antrean terpisah (pinjam + kembali) |
| Pengembalian buku | Tidak ada | Ada dengan deteksi buku rusak |
| Denda keterlambatan | Tidak ada | Ada (Rp 2.000/hari/buku) |
| Error scanner | Ada | Ada (di peminjaman) |
| Statistik | 6 variabel | 12+ variabel + per-hitungan denda |

---

## 1. Hasil Akhir

Setelah selesai, model akan memiliki alur lengkap:

1. Pengunjung muncul di pintu masuk dengan **tipe acak** (mahasiswa/dosen).
2. Saat muncul, sistem menentukan tujuan: **meminjam (70%)** atau **mengembalikan (30%)**.
3. Jika **meminjam**:
   - Berjalan ke counter peminjaman.
   - Antri di queue peminjaman (2 service point).
   - Data buku diproses (jumlah, tipe, deadline).
   - Ada kemungkinan **error scanner** (5%) -> waktu service bertambah.
4. Jika **mengembalikan**:
   - Berjalan ke counter pengembalian.
   - Antri di queue pengembalian (1 service point).
   - Buku diperiksa, ada kemungkinan **buku rusak** (10%) -> inspeksi tambahan.
   - Denda dihitung otomatis jika terlambat.
5. Kedatangan lebih **padat di menit 20-50** (peak hour) -> antrian terlihat mengular.
6. Setelah selesai, semua pengunjung berjalan ke area keluar.
7. Semua statistik tercatat di dashboard.

Flowchart utama:

```
srcMasuk -> selectTujuan (70/30)
    -> srvPinjam (counter peminjaman)
    -> srvKembali (counter pengembalian)
        -> (bergabung) wJalanKeluar -> snkSelesai
```

---

## 2. Perbedaan dengan Versi Sebelumnya

Jika kamu **sudah punya model dari tutorial dasar**, berikut perubahan yang harus dilakukan:

1. **Tambah** `PedSelectOutput` baru (`selectTujuan`) di antara `srcMasuk` dan `srvCounter`.
2. **Ubah nama** `srvCounter` menjadi `srvPinjam` (atau biarkan, tapi akan membingungkan).
3. **Tambah** `srvKembali` (PedService baru) dan `svcPengembalian` (markup baru).
4. **Tambah variabel baru** di `MahasiswaPed` dan `Main`.
5. **Tambah fungsi baru** untuk service pengembalian dan interarrival time.

Jika kamu **mulai dari nol**, ikuti tutorial ini dari awal. Semua langkah lengkap.

---

## 3. Buat Project Baru / Migrasi

### 3.1 Buat project baru (jika mulai dari nol)

1. Buka AnyLogic.
2. Klik **File -> New -> Model**.
3. Nama model: `Perpustakaan3D_Lengkap`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 3.2 Migrasi dari project tutorial dasar (jika sudah punya)

1. Buka project `Perpustakaan3D_AntriCounter`.
2. Klik **File -> Save As...** `Perpustakaan3D_Lengkap`.
3. Sekarang kamu bisa menambahkan fitur baru tanpa merusak model lama.
4. Hapus koneksi dari `srvCounter.out -> wJalanKeluar.in` (akan diganti).

Catatan: Semua nilai waktu di tutorial ini mengikuti satuan `minute`.

---

## 4. Update Pedestrian Type (MahasiswaPed)

### 4.1 Buat pedestrian type (jika mulai dari nol)

1. Buka `Main`.
2. Dari Palette **Pedestrian Library**, drag elemen **Pedestrian Type** ke canvas.
3. Wizard akan muncul.
4. Isi nama: `MahasiswaPed`.
5. Pilih animasi 3D (pilih model orang dari daftar bawaan AnyLogic).
6. Klik **Next** sampai selesai.

### 4.2 Variabel di MahasiswaPed

Setelah tipe dibuat, buka diagram `MahasiswaPed` lalu tambah Variable berikut:

| Nama | Type | Initial value | Keterangan |
|---|---|---|---|
| `idPed` | String | `""` | ID unik ped |
| `isDosen` | boolean | `false` | `true` = dosen, `false` = mahasiswa |
| `isPeminjaman` | boolean | `true` | `true` = mau pinjam, `false` = mau kembali |
| `jumlahBuku` | int | `1` | Jumlah buku yg dipinjam/dikembalikan |
| `tipePinjaman` | String | `"REGULER"` | REGULER / REFERENSI / RESERVE |
| `deadlineHari` | int | `7` | Batas waktu pinjam (hari) |
| `nomorResi` | String | `""` | Nomor resi transaksi |
| `scannerError` | boolean | `false` | Apakah terjadi error scanner |
| `waktuPinjam` | double | `0` | Waktu saat buku dipinjam |
| `waktuKembali` | double | `0` | Waktu saat buku dikembalikan |
| `tMasuk` | double | `0` | Waktu masuk sistem |
| `tMulaiService` | double | `0` | Waktu mulai dilayani |
| `tSelesaiService` | double | `0` | Waktu selesai dilayani |
| `jumlahHariTerlambat` | int | `0` | Jumlah hari terlambat (untuk denda) |
| `denda` | double | `0` | Total denda dalam Rupiah |

> **Tips pemula:** Variabel `waktuPinjam` dipakai untuk dua hal:
> - Saat **peminjaman**: diisi di `srvPinjam.onEndService()` sebagai waktu mulai pinjam.
> - Saat **pengembalian**: diisi di `srcMasuk.onExit()` untuk simulasi "pinjaman sebelumnya" (1-10 hari lalu).

### 4.3 Jika sudah punya MahasiswaPed dari tutorial dasar

Kamu hanya perlu **menambah** variabel berikut ke diagram `MahasiswaPed` yang sudah ada:

`isDosen`, `isPeminjaman`, `waktuPinjam`, `waktuKembali`, `jumlahHariTerlambat`, `denda`

Biarkan variabel lama (`idPed`, `jumlahBuku`, dll) tetap ada.

---

## 5. Siapkan Layout 3D dengan Dua Counter

### 5.1 3D Window dan Kamera

1. Dari Palette **Presentation**, drag **3D Window** ke `Main`. Rename: `win3D`.
2. Drag **Camera**. Rename: `camMain`.
3. Atur posisi kamera agar kedua counter terlihat jelas.

Contoh posisi kamera (boleh disesuaikan):
- X = 22
- Y = -16
- Z = 14
- Look at X = 10 (arahkan ke area tengah)

### 5.2 Objek Ruangan

Dari **3D Objects** palette:

1. **Box** untuk lantai. Rename: `floor3D`. Skala: 30 x 20 x 0.2.
2. **Box** untuk meja counter peminjaman. Rename: `mejaPinjam3D`. Letakkan di sisi kiri.
3. **Box** untuk meja counter pengembalian. Rename: `mejaKembali3D`. Letakkan di sisi kanan.
4. **Beberapa Box** untuk rak buku di sekeliling ruangan.

> Tips: Beri jarak yang cukup antara kedua counter agar antrean tidak saling tumpuk.
> Contoh: meja pinjam di (5, 3), meja kembali di (14, 3).

---

## 6. Buat Markup Pedestrian (Paling Penting)

### 6.1 entryLine (garis masuk)

1. Dari **Space Markup** (Pedestrian Library), tambah **Target line**.
2. Rename: `entryLine`.
3. Letakkan di area depan pintu masuk (misal: pojok kiri bawah).

### 6.2 svcPeminjaman (Service with Lines - Counter Pinjam)

1. Dari **Space Markup**, drag **Service with Lines**.
2. Rename: `svcPeminjaman`.
3. Letakkan service point di dekat meja `mejaPinjam3D`.
4. Buka properties `svcPeminjaman`:
   - **Number of services** = `2` (dua petugas di counter pinjam)
   - **N of queues** = `2` (dua antrean, satu per service point)
   - **Queue choice** = Shortest queue (biarkan default)

**Penting untuk pemula:**
- Atur arah queue line dari arah `entryLine` menuju service point.
- Klik queue line, lalu tarik titik-titiknya agar antrean rapi dan tidak memotong tembok.
- Service point harus di depan meja counter (bukan di dalam meja).

### 6.3 svcPengembalian (Service with Lines - Counter Kembali)

1. Dari **Space Markup**, drag **Service with Lines** lagi.
2. Rename: `svcPengembalian`.
3. Letakkan service point di dekat meja `mejaKembali3D`.
4. Buka properties `svcPengembalian`:
   - **Number of services** = `1` (satu petugas di counter kembali)
   - **N of queues** = `1`

> **Kenapa jumlah service berbeda?**
> - Peminjaman lebih banyak (70% pengunjung) -> perlu 2 service point.
> - Pengembalian lebih sedikit (30% pengunjung) -> 1 service point cukup.
> - Ini membuat antrean di kedua counter terlihat **seimbang** di visual.

### 6.4 exitLine (garis keluar)

1. Tambah **Target line** lagi.
2. Rename: `exitLine`.
3. Letakkan di sisi ruangan yang berlawanan dari entry (misal: pojok kanan atas).

### Layout markup (contoh posisi):

```
 entryLine                            exitLine
    |                                    ^
    |    [mejaPinjam]   [mejaKembali]    |
    |    svcPeminjaman  svcPengembalian  |
    +-------->  ~~~~~~~~>  ~~~~~~~~> ----+
```

Garis `~` = arah antrean dan jalur pedestrian.

---

## 7. Bangun Flowchart Pedestrian

Di `Main`, dari **Pedestrian Library**, drag blok berikut:

| Blok | Rename | Keterangan |
|---|---|---|
| PedSource | `srcMasuk` | Tempat lahir pedestrian |
| PedSelectOutput | `selectTujuan` | Pilih tujuan: pinjam (70%) atau kembali (30%) |
| PedService | `srvPinjam` | Service di counter peminjaman |
| PedService | `srvKembali` | Service di counter pengembalian |
| PedWait | `wJalanKeluar` | Jalan ke garis keluar |
| PedSink | `snkSelesai` | Keluar sistem |

**Hubungkan port sebagai berikut:**

```
srcMasuk.out -> selectTujuan.in

selectTujuan.out1 (pinjam 70%) -> srvPinjam.in
selectTujuan.out2 (kembali 30%) -> srvKembali.in

srvPinjam.out -> wJalanKeluar.in
srvKembali.out -> wJalanKeluar.in

wJalanKeluar.out -> snkSelesai.in
```

> **Catatan penting:** Dua blok (`srvPinjam.out` dan `srvKembali.out`) bisa terhubung ke **satu** `wJalanKeluar.in` yang sama. AnyLogic mengizinkan ini.

### Kenapa ada PedWait sebelum sink?

- Supaya pedestrian benar-benar berjalan ke area keluar dulu (bukan menghilang tiba-tiba).
- Memberi waktu 0.2 menit untuk menunjukkan transisi keluar secara visual.

---

## 8. Konfigurasi Detail Setiap Blok

### 8.1 `srcMasuk` (PedSource)

**Properties utama:**

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` |
| Arrive according to | `Interarrival time` |
| Interarrival time | `getInterarrivalTime()` (akan dibuat nanti di bagian fungsi) |
| New pedestrian | `MahasiswaPed` |

#### Action `On exit`:

Salin kode ini persis ke kolom **Action** -> **On exit**:

```java
seqPed++;
ped.idPed = "M-" + seqPed;
ped.tMasuk = time();

// --- Tentukan tipe pengunjung: 20% dosen ---
ped.isDosen = uniform(0, 1) < 0.2;

// --- Tentukan tujuan: 70% pinjam, 30% kembali ---
ped.isPeminjaman = uniform(0, 1) < 0.7;

if (ped.isPeminjaman) {
    // ===== DATA PEMINJAMAN =====
    if (ped.isDosen) {
        ped.jumlahBuku = uniform_discr(2, 8); // dosen pinjam lebih banyak
    } else {
        ped.jumlahBuku = uniform_discr(1, 5); // mahasiswa 1-5 buku
    }

    double r = uniform(0, 1);
    if (r < 0.7) {
        ped.tipePinjaman = "REGULER";
        ped.deadlineHari = 7;
    } else if (r < 0.9) {
        ped.tipePinjaman = "REFERENSI";
        ped.deadlineHari = 3;
    } else {
        ped.tipePinjaman = "RESERVE";
        ped.deadlineHari = 1;
    }

} else {
    // ===== DATA PENGEMBALIAN =====
    // Simulasi: orang ini "sebelumnya" sudah pinjam buku
    ped.jumlahBuku = uniform_discr(1, 4);

    double r = uniform(0, 1);
    if (r < 0.7) {
        ped.tipePinjaman = "REGULER";
        ped.deadlineHari = 7;
    } else if (r < 0.9) {
        ped.tipePinjaman = "REFERENSI";
        ped.deadlineHari = 3;
    } else {
        ped.tipePinjaman = "RESERVE";
        ped.deadlineHari = 1;
    }

    // Simulasi waktu pinjam 1-10 hari yang lalu
    double hariLalu = uniform(1.0, 10.0);
    ped.waktuPinjam = time() - hariLalu * 24 * 60;
}

// --- Statistik tipe pengunjung ---
if (ped.isDosen) {
    totalDosen++;
} else {
    totalMahasiswa++;
}

// --- Log ke console ---
traceln("ARRIVE " + ped.idPed
    + " | " + (ped.isPeminjaman ? "PINJAM" : "KEMBALI")
    + " | " + (ped.isDosen ? "DOSEN" : "MAHASISWA")
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari");
```

> **Penjelasan untuk pemula:**
> - `uniform(0, 1)` menghasilkan angka acak 0.0 - 1.0.
> - `uniform(0, 1) < 0.7` berarti "70% kemungkinan true".
> - `uniform_discr(1, 5)` menghasilkan angka bulat acak antara 1 - 5.

### 8.2 `selectTujuan` (PedSelectOutput)

**Properties utama:**

| Property | Value |
|---|---|
| N outputs | `2` |
| Use probabilities | Centang / checklist |
| Output 1 probability | `0.7` (peminjaman) |
| Output 2 probability | `0.3` (pengembalian) |

> **Penjelasan:** Blok ini mengarahkan 70% pedestrian ke port output 1 (`srvPinjam`) dan 30% ke port output 2 (`srvKembali`). Keputusan sudah ditentukan di `srcMasuk` lewat variabel `ped.isPeminjaman`, tapi di sini kita pakai probabilitas saja.

### 8.3 `srvPinjam` (PedService - Counter Peminjaman)

**Properties utama:**

| Property | Value |
|---|---|
| Services | `svcPeminjaman` (markup yang sudah dibuat) |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuServicePeminjaman(ped)` |
| Recovery delay | `0` |

#### Action `On enter queue`:

```java
int q = srvPinjam.queueSize();
if (q > maxQueuePeminjaman) {
    maxQueuePeminjaman = q;
}
```

#### Action `On begin service`:

```java
ped.tMulaiService = time();
```

#### Action `On end service`:

```java
ped.tSelesaiService = time();
ped.waktuPinjam = time();
ped.nomorResi = "PNJ-" + (long)(time() * 1000) + "-" + ped.idPed;

totalPeminjaman++;

traceln("SERVICE " + ped.idPed
    + " | resi=" + ped.nomorResi
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari"
    + " | scannerError=" + ped.scannerError);
```

### 8.4 `srvKembali` (PedService - Counter Pengembalian)

**Properties utama:**

| Property | Value |
|---|---|
| Services | `svcPengembalian` (markup) |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuServicePengembalian(ped)` |
| Recovery delay | `0` |

#### Action `On enter queue`:

```java
int q = srvKembali.queueSize();
if (q > maxQueuePengembalian) {
    maxQueuePengembalian = q;
}
```

#### Action `On begin service`:

```java
ped.tMulaiService = time();
ped.waktuKembali = time();
```

#### Action `On end service`:

```java
ped.tSelesaiService = time();

totalPengembalian++;

if (ped.jumlahHariTerlambat > 0) {
    traceln("DENDA " + ped.idPed
        + " | telat=" + ped.jumlahHariTerlambat + " hari"
        + " | denda=Rp " + String.format("%,.0f", ped.denda));
}

traceln("RETURN " + ped.idPed
    + " | buku=" + ped.jumlahBuku
    + " | telat=" + ped.jumlahHariTerlambat + " hari"
    + " | denda=Rp " + String.format("%,.0f", ped.denda));
```

> **Catatan:** `String.format("%,.0f", ...)` membuat angka denda tampil dengan pemisah ribuan. Jika error, bisa diganti dengan `ped.denda` saja.

### 8.5 `wJalanKeluar` (PedWait)

**Properties utama:**

| Property | Value |
|---|---|
| Waiting location | `Target line` |
| Target line | `exitLine` |
| Delay ends | `On delay time expiry` |
| Delay time | `0.2` |

Penjelasan:
- Blok ini memaksa pedestrian berjalan dulu ke `exitLine`, baru lanjut keluar.
- `0.2` menit cukup untuk menunjukkan transisi keluar tanpa menahan terlalu lama.

### 8.6 `snkSelesai` (PedSink)

#### Action `On enter`:

```java
totalSelesai++;

if (ped.isPeminjaman) {
    totalBukuDipinjam += ped.jumlahBuku;
    if (ped.scannerError) {
        totalErrorScanner++;
    }
}

double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " menit"
    + " | tipe=" + (ped.isDosen ? "DOSEN" : "MHS")
    + " | " + (ped.isPeminjaman ? "PINJAM" : "KEMBALI"));
```

---

## 9. Variabel Statistik Baru di Main

### 9.1 Semua variabel (baru + lama)

Buka `Main`, lalu pada bagian **Variable** tambahkan semua berikut:

**Variabel yang sudah ada dari tutorial dasar:**

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |
| `maxQueueCounter` | int | 0 |
| `totalBukuDipinjam` | int | 0 |
| `totalErrorScanner` | int | 0 |

> Variabel `maxQueueCounter` sudah tidak dipakai di flowchart baru, tapi boleh dibiarkan saja.

**Variabel baru untuk versi lengkap:**

| Nama | Type | Initial value | Kegunaan |
|---|---|---|---|
| `totalPeminjaman` | int | 0 | Total transaksi peminjaman |
| `totalPengembalian` | int | 0 | Total transaksi pengembalian |
| `totalMahasiswa` | int | 0 | Total pengunjung mahasiswa |
| `totalDosen` | int | 0 | Total pengunjung dosen |
| `totalDenda` | double | 0 | Total denda terkumpul (Rupiah) |
| `maxQueuePeminjaman` | int | 0 | Panjang antrean maks di counter pinjam |
| `maxQueuePengembalian` | int | 0 | Panjang antrean maks di counter kembali |
| `totalBukuRusak` | int | 0 | Total buku yang rusak saat pengembalian |

### 9.2 Inisialisasi variabel

Tidak perlu inisialisasi khusus karena semua punya `Initial value`. AnyLogic akan mengatur ulang setiap kali run.

---

## 10. Fungsi-Fungsi Baru di Main

Klik kanan di area kosong canvas `Main` -> **Add** -> **Function**, atau drag **Function** dari palette **Agent**.

### 10.1 `getInterarrivalTime` (Return type: `double`)

Fungsi ini membuat **peak hour**: kedatangan lebih sering di menit 20-50.

```java
double t = time();

if (t < 20) {
    // SEPI: awal simulasi, 1.5 orang per menit
    return exponential(1.5);
} else if (t < 50) {
    // RAMAI: peak hour, 5.0 orang per menit (antrian mengular!)
    return exponential(5.0);
} else {
    // NORMAL: setelah peak, 2.5 orang per menit
    return exponential(2.5);
}
```

> **Cara kerja:** Nilai `exponential(5.0)` berarti rata-rata 5 orang datang per menit, atau 1 orang setiap 12 detik. Bandingkan dengan `exponential(1.5)` yang 1 orang setiap 40 detik.

### 10.2 `hitungWaktuServicePeminjaman` (Return type: `double`, parameter: `MahasiswaPed ped`)

```java
double dasar;

if (ped.isDosen) {
    // Dosen: prioritas lebih cepat (70% dari waktu normal)
    if (ped.jumlahBuku <= 2) {
        dasar = uniform(1.0, 1.5);
    } else if (ped.jumlahBuku <= 4) {
        dasar = uniform(1.5, 2.5);
    } else {
        dasar = uniform(2.5, 3.5);
    }
} else {
    // Mahasiswa: waktu normal
    if (ped.jumlahBuku <= 2) {
        dasar = uniform(1.5, 2.2);
    } else if (ped.jumlahBuku <= 4) {
        dasar = uniform(2.2, 3.2);
    } else {
        dasar = uniform(3.2, 4.5);
    }
}

// 5% kemungkinan error scanner -> waktu tambahan
ped.scannerError = false;
if (uniform(0, 1) < 0.05) {
    dasar += uniform(0.8, 1.4);
    ped.scannerError = true;
}

return dasar;
```

### 10.3 `hitungWaktuServicePengembalian` (Return type: `double`, parameter: `MahasiswaPed ped`)

Fungsi ini menghitung waktu layanan pengembalian **sekaligus** menghitung denda.

```java
double dasar;

if (ped.jumlahBuku <= 2) {
    dasar = uniform(0.8, 1.5);
} else if (ped.jumlahBuku <= 4) {
    dasar = uniform(1.5, 2.5);
} else {
    dasar = uniform(2.5, 3.5);
}

// 10% kemungkinan buku rusak -> inspeksi tambahan
if (uniform(0, 1) < 0.1) {
    dasar += uniform(1.0, 2.0);
    totalBukuRusak++;
}

// --- Hitung denda keterlambatan ---
double waktuKembali = time();
double hariTerpakai = (waktuKembali - ped.waktuPinjam) / (24.0 * 60.0);

if (hariTerpakai > ped.deadlineHari) {
    ped.jumlahHariTerlambat = (int) Math.ceil(hariTerpakai - ped.deadlineHari);
    // Denda: Rp 2.000 per hari per buku
    ped.denda = ped.jumlahHariTerlambat * ped.jumlahBuku * 2000.0;
    totalDenda += ped.denda;
}

return dasar;
```

> **Penjelasan perhitungan denda:**
> - `waktuKembali - ped.waktuPinjam` = selisih waktu dalam menit.
> - `/(24*60)` = konversi menit ke hari.
> - `Math.ceil()` = pembulatan ke atas (1.5 hari keterlambatan dihitung 2 hari).
> - Rp 2.000 per hari per buku.

### 10.4 Fungsi Dashboard (untuk text dinamis)

**Fungsi `avgWaktuSistem`** (Return type: `double`):

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

**Fungsi `avgBukuPerTransaksi`** (Return type: `double`):

```java
return totalPeminjaman == 0 ? 0 : (1.0 * totalBukuDipinjam / totalPeminjaman);
```

**Fungsi `errorScannerRate`** (Return type: `double`):

```java
return totalPeminjaman == 0 ? 0 : (100.0 * totalErrorScanner / totalPeminjaman);
```

**Fungsi `rataRataDenda`** (Return type: `double`):

```java
return totalPengembalian == 0 ? 0 : totalDenda / totalPengembalian;
```

---

## 11. Dashboard Lengkap

Tambahkan **Text** dinamis di `Main` (dari Presentation palette, drag **Text** ke canvas).

Untuk membuat Text dinamis: klik kanan text -> centang **Dynamic description**, lalu paste kode.

Berikut semua text yang perlu ditambahkan:

### Baris 1: Ringkasan Umum

```
"Total pengunjung: " + totalSelesai + " orang"
```

```
"Total mahasiswa: " + totalMahasiswa + " | Dosen: " + totalDosen
```

```
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

### Baris 2: Statistik Counter

```
"PEMINJAMAN | Selesai: " + totalPeminjaman + " | Antrian skrg: " + srvPinjam.queueSize() + " | Max: " + maxQueuePeminjaman
```

```
"PENGEMBALIAN | Selesai: " + totalPengembalian + " | Antrian skrg: " + srvKembali.queueSize() + " | Max: " + maxQueuePengembalian
```

### Baris 3: Statistik Buku

```
"Total buku dipinjam: " + totalBukuDipinjam + " | Rata-rata: " + String.format("%.1f", avgBukuPerTransaksi()) + "/transaksi"
```

```
"Error scanner: " + totalErrorScanner + " kali (" + String.format("%.1f", errorScannerRate()) + "%)"
```

```
"Buku rusak saat kembali: " + totalBukuRusak + " buah"
```

### Baris 4: Statistik Denda

```
"Total denda: Rp " + String.format("%,.0f", totalDenda)
```

```
"Rata-rata denda/pengembalian: Rp " + String.format("%,.0f", rataRataDenda())
```

### Baris 5: Status Antrian Real-time

```
"Antrian peminjaman: " + srvPinjam.queueSize() + " orang"
```

```
"Antrian pengembalian: " + srvKembali.queueSize() + " orang"
```

> **Tips tata letak:** Kelompokkan text-text di atas dalam 4-5 baris horizontal di pojok atas atau samping 3D window agar mudah dipantau saat simulasi berjalan.

---

## 12. Jalankan Simulasi

### 12.1 Uji awal (60 menit)

1. Set **Stop time** = 60 menit.
2. Klik **Run**.
3. Amati apakah:
   - Ada peak activity di menit 20-50 (antrian mengular di peminjaman).
   - Counter pengembalian lebih sepi tapi tetap aktif.
   - Sesekali ada buku rusak (cek console lewat `traceln`).
   - Denda muncul di log untuk pengembalian terlambat.

### 12.2 Skenario demo (120 menit)

1. Set **Stop time** = 120 menit.
2. Turunkan **speed slider** (jangan maksimum).
3. Fokus kamera ke area yang mencakup **kedua counter** + queue.
4. Pantau dashboard: angka denda, error, dan rata-rata waktu sistem.

### 12.3 Skenario beban tinggi (ekstrem)

Untuk melihat antrian benar-benar panjang:

1. Set **Stop time** = 90 menit.
2. Ubah fungsi `getInterarrivalTime()` sementara:

```java
// SEMENTARA: tes beban tinggi
return exponential(8.0);
```

3. Jalankan. Antrian di peminjaman akan sangat panjang.
4. **Kembalikan** ke fungsi normal setelah selesai uji.

> **Tips untuk pemula:** Jika model terlalu cepat/kompleks, mulai dengan **Stop time 30 menit** dulu. Pastikan setidaknya 10-15 pengunjung selesai memproses.

---

## 13. Troubleshooting

### Masalah A: Semua pengunjung hanya ke satu counter

**Penyebab:** Koneksi `selectTujuan` salah, atau probabilitas tidak ter-set.

**Solusi:**
1. Klik `selectTujuan`.
2. Buka properties.
3. Pastikan **Output 1 probability** = 0.7, **Output 2 probability** = 0.3.
4. Pastikan port terhubung: `out1` ke `srvPinjam`, `out2` ke `srvKembali`.

### Masalah B: Pedestrian tidak bergerak / numpuk di entry

**Penyebab umum:**
- Markup `svcPeminjaman` atau `svcPengembalian` tidak diisi di properti PedService.
- Markup ketutup objek 3D (meja counter menimpa service point).

**Solusi:**
1. Cek properties `srvPinjam` -> `Services` harus = `svcPeminjaman`.
2. Cek properties `srvKembali` -> `Services` harus = `svcPengembalian`.
3. Geser objek 3D agar tidak menutupi service point.
4. Coba matikan sementara visibility rak buku untuk debugging.

### Masalah C: Error "Cannot convert from Agent to MahasiswaPed"

**Penyebab:** Parameter fungsi mengharapkan `MahasiswaPed` tapi dipanggil dengan `agent`.

**Solusi:**
- Di blok Pedestrian, gunakan `ped`, bukan `agent`, untuk mengakses atribut pedestrian.
- Contoh: `hitungWaktuServicePeminjaman(ped)` (bukan `agent`).

### Masalah D: Fungsi `exponential()` error saat dipanggil dari fungsi kustom

**Penyebab:** Fungsi `exponential()` tidak tersedia di konteks non-Agent dalam beberapa versi AnyLogic lama.

**Solusi:**
- Pastikan fungsi `getInterarrivalTime()` dibuat di `Main` (bukan di agent lain).
- Sebagai alternatif, bisa gunakan `uniform()` + transformasi untuk eksponensial:

```java
// Alternatif manual: transformasi inverse exponential
double rate = ...; // rate yang diinginkan
return -Math.log(1.0 - uniform(0, 1)) / rate;
```

### Masalah E: Denda tidak kunjung muncul

**Penyebab:** Nilai `hariTerpakai` tidak pernah melebihi `deadlineHari` (terlalu pendek).

**Solusi:**
1. Di `srcMasuk`, pastikan `hariLalu = uniform(1.0, 10.0)` menghasilkan rentang yang cukup.
2. Untuk debugging, tambahkan log di `hitungWaktuServicePengembalian`:

```java
traceln("CEK DENDA: hariTerpakai=" + hariTerpakai + " deadline=" + ped.deadlineHari);
```

### Masalah F: Fungsi `String.format()` error

**Penyebab:** Beberapa versi AnyLogic tidak mendukung `String.format()`.

**Solusi:** Ganti dengan `Double.toString()` atau concatenation biasa:

```java
// String.format("%,.2f", value) -> alternative:
((long)(value * 100)) / 100.0
```

Contoh: `"Rp " + ((long)(totalDenda * 100)) / 100.0`

### Masalah G: Antrean di satu counter lebih panjang terus

**Penyebab alami:** Peminjaman (70% pengunjung, 2 service point) vs Pengembalian (30%, 1 service point). Ini NORMAL.

**Yang perlu dicek:**
- Jika antrean peminjaman terus bertambah tanpa pernah turun, berarti arrival rate terlalu tinggi. Turunkan parameter di `getInterarrivalTime()`.
- Jika antrean pengembalian kosong terus, pastikan `selectTujuan` probability = 0.3 untuk output 2.

### Masalah H: Semua pengunjung adalah mahasiswa (tidak ada dosen)

**Penyebab:** Kode `ped.isDosen = uniform(0, 1) < 0.2` mungkin tidak terpanggil.

**Solusi:**
1. Cek action `On exit` di `srcMasuk` — pastikan kode ada dan tidak error.
2. Cek console output: `ARRIVE ... DOSEN` seharusnya muncul ~20%.

---

## 14. Checklist Final

Sebelum demo, pastikan semua poin ini tercentang:

### Flowchart & Koneksi
- [ ] `srcMasuk` terhubung ke `selectTujuan.in`
- [ ] `selectTujuan.out1` (0.7) terhubung ke `srvPinjam.in`
- [ ] `selectTujuan.out2` (0.3) terhubung ke `srvKembali.in`
- [ ] `srvPinjam.out` terhubung ke `wJalanKeluar.in`
- [ ] `srvKembali.out` terhubung ke `wJalanKeluar.in`
- [ ] `wJalanKeluar.out` terhubung ke `snkSelesai.in`

### Markup
- [ ] `entryLine`, `svcPeminjaman`, `svcPengembalian`, `exitLine` sudah dibuat
- [ ] `srvPinjam.Services` = `svcPeminjaman`
- [ ] `srvKembali.Services` = `svcPengembalian`
- [ ] Queue line mengarah dari area datang ke service point
- [ ] Service point tidak tertutup objek 3D

### Blok Configuration
- [ ] `srcMasuk.New pedestrian` = `MahasiswaPed`
- [ ] `srcMasuk.Interarrival time` = `getInterarrivalTime()`
- [ ] `srcMasuk.On exit` berisi kode generate data lengkap
- [ ] `selectTujuan` probabilities: 0.7 dan 0.3
- [ ] `srvPinjam.Delay time` = `hitungWaktuServicePeminjaman(ped)`
- [ ] `srvKembali.Delay time` = `hitungWaktuServicePengembalian(ped)`
- [ ] `wJalanKeluar.Target line` = `exitLine`
- [ ] `wJalanKeluar.Delay time` = 0.2

### Data & Statistik
- [ ] Semua variabel di `MahasiswaPed` sudah dibuat (15 variabel)
- [ ] Semua variabel di `Main` sudah dibuat (13+ variabel)
- [ ] Fungsi `getInterarrivalTime` sudah dibuat
- [ ] Fungsi `hitungWaktuServicePeminjaman` sudah dibuat
- [ ] Fungsi `hitungWaktuServicePengembalian` sudah dibuat
- [ ] Fungsi dashboard (`avgWaktuSistem`, `avgBukuPerTransaksi`, `errorScannerRate`, `rataRataDenda`) sudah dibuat

### Visual
- [ ] 3D window dan kamera terpasang
- [ ] Objek ruangan (lantai, meja, rak) minimal ada
- [ ] Dua meja counter terlihat berbeda
- [ ] Dashboard text muncul dan berubah saat run

### Pengujian
- [ ] Run 60 menit: semua pedestrian lahir, antri, service, jalan keluar
- [ ] Ada pengunjung yang meminjam (70%) dan mengembalikan (30%)
- [ ] Ada mahasiswa (80%) dan dosen (20%)
- [ ] Peak hour terlihat: antrian memanjang di menit 20-50
- [ ] Scanner error sesekali muncul dan menambah waktu service
- [ ] Buku rusak sesekali muncul saat pengembalian
- [ ] Denda muncul untuk pengembalian terlambat
- [ ] Dashboard memperbarui angka secara real-time

### Code Clean
- [ ] Tidak ada kode manual `setXYZ()` / `jumpTo()` di flow pedestrian
- [ ] Tidak ada action error di console
- [ ] Semua akses atribut via `ped.` bukan `agent.`

Jika semua poin ini lolos, selamat! Model simulasi perpustakaan 3D kamu sudah **lengkap dengan dua counter, peak hour, dua tipe pengunjung, dan sistem denda**.

---

## 15. Referensi

Dokumentasi resmi AnyLogic yang dipakai:

- AnyLogic Agent API (`setXYZ`, `jumpTo`, `moveTo`, `moveToInTime`)
- Pedestrian Library Overview: blocks, markup, dan konsep
- `PedSource` — konfigurasi interarrival time dan custom pedestrian type
- `PedService` — service with lines dan queue choice policy
- `PedSelectOutput` — routing berdasarkan probabilitas
- `PedWait` — target line untuk delayed exit
- `PedSink` — action on enter
- `Service with Lines` markup — multi-service configuration
- `Custom pedestrian types` — variabel dan parameter kustom
- `Animating pedestrians` — 3D animation selection
- `3D Window` dan `Camera` — presentation
- Java Math API (`Math.ceil`) untuk perhitungan denda

> Catatan penting: Untuk model orang berjalan realistis di AnyLogic, selalu gunakan Pedestrian Library + markup pedestrian. `setXYZ()` hanya untuk inisialisasi posisi awal, bukan untuk perpindahan runtime.
