# Tutorial Integrasi 5 Modul Simulasi Perpustakaan 3D

**Berdasarkan modul aktual yang sudah dikerjakan — BUKAN versi tutorial ideal**

| Detail | Keterangan |
|---|---|
| **Software** | AnyLogic 8.9.8 |
| **Satuan waktu** | `minute` (semua moduk dikonversi ke MINUTE) |
| **Library utama** | Pedestrian Library |
| **Agent type** | `PengunjungPed` (disamakan untuk semua modul) |
| **File output** | `Perpustakaan3D_Full.alp` |

---

## Daftar Isi

1. [Pendahuluan](#1-pendahuluan)
2. [Langkah 1: Siapkan Base Project](#2-langkah-1-siapkan-base-project)
3. [Langkah 2: Unifikasi Agent Type (PengunjungPed)](#3-langkah-2-unifikasi-agent-type-pengunjungped)
4. [Langkah 3: Merge Variabel & Fungsi di Main](#4-langkah-3-merge-variabel--fungsi-di-main)
5. [Langkah 4: Bangun Ulang Entry Flow (Scan KTM + Routing)](#5-langkah-4-bangun-ulang-entry-flow-scan-ktm--routing)
6. [Langkah 5: Integrasi Modul Parkir](#6-langkah-5-integrasi-modul-parkir)
7. [Langkah 6: Integrasi Modul Tempat Duduk](#7-langkah-6-integrasi-modul-tempat-duduk)
8. [Langkah 7: Integrasi Modul Helpdesk](#8-langkah-7-integrasi-modul-helpdesk)
9. [Langkah 8: Merge Markup & Layout 3D](#9-langkah-8-merge-markup--layout-3d)
10. [Langkah 9: Hubungkan Semua Rute Final](#10-langkah-9-hubungkan-semua-rute-final)
11. [Langkah 10: Dashboard & Testing](#11-langkah-10-dashboard--testing)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. Pendahuluan

### 1.1 Apa yang Akan Kita Lakukan

Kita akan menggabungkan **5 modul AnyLogic** yang sudah dikerjakan terpisah menjadi **1 file simulasi utuh** — sebuah simulasi perpustakaan 3D yang lengkap dari parkir hingga keluar.

### 1.2 Alur Final yang Akan Dicapai

```
PARKIR (Modul Parkir) ──┐
                         ├──→ srvScanKTM (scan KTM di pintu masuk)
JALAN KAKI (baru dibuat) ┘       │
                                  ▼
                           selectTujuan (5 output)
      ├── out1 (25%) → PINJAM BUKU
      │    goToRakBuku → wKelilingRak → selectJadiPinjam
      │      ├── (90%) srvPinjam (2 counter)
      │      └── (10%) BATAL → wJalanKeluar
      │
      ├── out2 (20%) → KEMBALIKAN BUKU
      │    goToCounterKembali → srvKembali (1 counter)
      │
      ├── out3 (30%) → CARI TEMPAT DUDUK
      │    selectAktivitas → [toilet/buku/loker/langsung]
      │    → selectZonaDuduk → srvDudukSepi / srvDudukDiskusi
      │
      ├── out4 (10%) → HELPDESK
      │    goToHelpdesk → antreHelpdesk → srvHelpdesk (2 petugas)
      │    → selectStatusBuku → [rak/pinjam/keluar]
      │
      └── out5 (15%) → LANGSUNG KELUAR
                           │
                           ▼
                     wJalanKeluar → selectPulang
      ├── (isParkir=true) → wJalanBalikParkir → jalanKeParkiran
      └── (isParkir=false) → snkSelesai (keluar)
```

### 1.3 Modul yang Akan Diintegrasikan

| No | Modul | File | Library | Time Unit | Kondisi |
|---|---|---|---|---|---|
| 1 | **BASE** Peminjaman Pengembalian | `Modul Peminjaman Pengembalian.alp` | Pedestrian | MINUTE | ✅ Paling lengkap, jadi base |
| 2 | Kedatangan | `Modul Kedatangan.alp` | **Process Modeling** | SECOND | ❌ Harus dibangun ulang pakai Pedestrian |
| 3 | Parkir | `Modul Parkir.alp` | Road Traffic + Pedestrian | MINUTE | ✅ Kompatibel, tinggal copy |
| 4 | Cari Tempat Duduk | `Modul Cari tempat duduk.alp` | Pedestrian | **SECOND** | ⚠️ Konversi ke MINUTE |
| 5 | Helpdesk | `Perpustakaan3D_Helpdesk1.alp` | Pedestrian + Queue PM | MINUTE | ✅ Kompatibel, ada Queue PM → konversi |

### 1.4 Prasyarat

- AnyLogic 8.9.8 terinstal
- Semua 5 file module `.alp` sudah siap
- Konsep dasar: Pedestrian Library (PedSource, PedService, PedGoTo, PedWait, PedSelectOutput, PedSink)

---

## 2. Langkah 1: Siapkan Base Project

Kita gunakan **Modul Peminjaman Pengembalian** sebagai base karena:
- Paling lengkap (flowchart, variabel, fungsi, markup, 3D)
- Sudah pakai Pedestrian Library
- Time unit MINUTE (lebih mudah untuk waktu layanan)
- Sudah punya agent `PengunjungPed` dengan banyak variabel

### 2.1 Copy & Rename

1. Buka folder `Modul Pengembalian\`
2. Copy file `Modul Peminjaman Pengembalian.alp`
3. Paste di folder `D:\vscode\Kuliahan\ModSim\Perpustakaan\`
4. Rename menjadi `Perpustakaan3D_Full.alp`

### 2.2 Buka di AnyLogic

1. Buka AnyLogic
2. **File → Open** → pilih `Perpustakaan3D_Full.alp`
3. Tunggu sampai loading selesai (panel Projects akan tampil)
4. **File → Save As** (untuk memastikan path benar)

### 2.3 Verifikasi Isi Base

Di panel Projects, pastikan terlihat:

**Main** berisi:
- Flowchart: `srcMasuk → selectJenisLayanan → ... → snkSelesai`
- Variabel: `seqPed`, `totalPeminjaman`, `totalPengembalian`, dll (14 total)
- Fungsi: `hitungWaktuServicePeminjaman`, `hitungWaktuServicePengembalian`, dll (9 total)

**PengunjungPed** berisi:
- Agent type dengan animasi 3D
- Variabel: `idPed`, `tMasuk`, `isDosen`, `isPeminjaman`, `jumlahBuku`, dll

> Jika ada perbedaan nama agent type (misal `MahasiswaPed` bukan `PengunjungPed`), catat namanya. Kita akan samakan nanti.

---

## 3. Langkah 2: Unifikasi Agent Type (PengunjungPed)

Semua modul harus menggunakan agent type yang SAMA agar pedestrian bisa melewati semua blok tanpa error.

### 3.1 Cek Agent Type di Base

Buka panel Projects. Cari agent type di bawah nama project. Jika namanya bukan `PengunjungPed`, rename:

1. Klik kanan agent type → **Properties**
2. Ganti **Name** menjadi `PengunjungPed`
3. Klik **OK**

### 3.2 Tambah Variabel dari Modul Parkir

Buka diagram **PengunjungPed** (klik 2x). Dari palette **Agent**, drag **Variable** ke canvas. Tambahkan:

| Nama | Type | Initial Value | Modul Asal |
|---|---|---|---|
| `isParkir` | boolean | `false` | Parkir |
| `jenisKendaraan` | String | `""` | Parkir |
| `waktuParkir` | double | `0` | Parkir |

### 3.3 Tambah Variabel dari Modul Tempat Duduk

| Nama | Type | Initial Value | Modul Asal |
|---|---|---|---|
| `waktuMulaiAktivitas` | double | `0` | Tempat Duduk |
| `waktuMulaiDuduk` | double | `0` | Tempat Duduk |
| `durasiBelajar` | double | `0` | Tempat Duduk |
| `zonaDuduk` | String | `""` | Tempat Duduk |
| `aktivitasSebelumDuduk` | String | `""` | Tempat Duduk |
| `jenisKelamin` | String | `""` | Tempat Duduk |

### 3.4 Tambah Variabel dari Modul Helpdesk

| Nama | Type | Initial Value | Modul Asal |
|---|---|---|---|
| `butuhHelpdesk` | boolean | `false` | Helpdesk |
| `statusBuku` | String | `""` | Helpdesk |
| `waktuMulaiAntre` | double | `0` | Helpdesk |
| `waktuMulaiLayanan` | double | `0` | Helpdesk |
| `waktuSelesaiLayanan` | double | `0` | Helpdesk |
| `waktuTunggu` | double | `0` | Helpdesk |
| `waktuLayanan` | double | `0` | Helpdesk |

### 3.5 Variabel yang SUDAH ADA di Base (jangan dibuat duplikat)

| Variabel | Base | Helpdesk | Parkir | Duduk |
|---|---|---|---|---|
| `idPed` | ✅ | ✅ | ✅ | — |
| `tMasuk` | ✅ | ✅ | ✅ | — |
| `isDosen` | ✅ | — | — | — |
| `isPeminjaman` | ✅ | — | — | — |
| `jumlahBuku` | ✅ | — | — | — |
| `tipePinjaman` | ✅ | — | — | — |
| `deadlineHari` | ✅ | — | — | — |
| `nomorResi` | ✅ | — | — | — |
| `scannerError` | ✅ | — | — | — |
| `waktuPinjam` | ✅ | — | — | — |
| `waktuKembali` | ✅ | — | — | — |
| `jumlahHariTerlambat` | ✅ | — | — | — |
| `denda` | ✅ | — | — | — |
| `waktuMulaiBrowsing` | ✅ | — | — | — |
| `durasiBrowsing` | ✅ | — | — | — |
| `jadiPinjam` | ✅ | — | — | — |

> **Tips:** Jika variabel sudah ada, jangan dibuat lagi. Cek daftar dengan membaca satu per satu di canvas PengunjungPed.

### 3.6 Verifikasi Agent Type

Setelah selesai, diagram `PengunjungPed` harus memiliki **minimal 29 variabel**:

```
Dari Base (16):  idPed, tMasuk, isDosen, isPeminjaman, jumlahBuku, tipePinjaman,
                 deadlineHari, nomorResi, scannerError, waktuPinjam, waktuKembali,
                 jumlahHariTerlambat, denda, waktuMulaiBrowsing, durasiBrowsing, jadiPinjam
Dari Parkir (3):  isParkir, jenisKendaraan, waktuParkir
Dari Duduk (6):   waktuMulaiAktivitas, waktuMulaiDuduk, durasiBelajar, zonaDuduk,
                  aktivitasSebelumDuduk, jenisKelamin
Dari Helpdesk (7): butuhHelpdesk, statusBuku, waktuMulaiAntre, waktuMulaiLayanan,
                   waktuSelesaiLayanan, waktuTunggu, waktuLayanan
```

---

## 4. Langkah 3: Merge Variabel & Fungsi di Main

### 4.1 Variabel yang SUDAH ADA di Base

Base project sudah punya 14 variabel ini:
`seqPed`, `totalPeminjaman`, `totalPengembalian`, `totalDenda`, `totalBukuRusak`,
`maxQueuePeminjaman`, `maxQueuePengembalian`, `totalBukuDipinjam`, `totalErrorScanner`,
`totalBrowsing`, `totalBatalPinjam`, `totalDurasiBrowsing`, `totalSelesai`, `totalWaktuSistem`

### 4.2 Tambah Variabel dari Modul Parkir

Buka **Main**. Dari palette **Agent**, drag **Variable** ke canvas:

| Nama | Type | Initial | Modul |
|---|---|---|---|
| `totalMobil` | int | 0 | Parkir |
| `totalMotor` | int | 0 | Parkir |

### 4.3 Tambah Variabel dari Modul Tempat Duduk

| Nama | Type | Initial | Modul |
|---|---|---|---|
| `totalToiletSinggah` | int | 0 | Tempat Duduk |
| `totalCariBuku` | int | 0 | Tempat Duduk |
| `totalLoker` | int | 0 | Tempat Duduk |
| `totalDudukSepi` | int | 0 | Tempat Duduk |
| `totalDudukDiskusi` | int | 0 | Tempat Duduk |
| `maxAntrianDudukSepi` | int | 0 | Tempat Duduk |
| `maxAntrianDudukDiskusi` | int | 0 | Tempat Duduk |
| `totalDurasiBelajar` | double | 0 | Tempat Duduk |

### 4.4 Tambah Variabel dari Modul Helpdesk

| Nama | Type | Initial | Modul |
|---|---|---|---|
| `totalMahasiswa` | int | 0 | Helpdesk |
| `totalKeHelpdesk` | int | 0 | Helpdesk |
| `totalBukuTersedia` | int | 0 | Helpdesk |
| `totalBukuDipinjamHelpdesk` | int | 0 | Helpdesk |
| `totalBukuTidakAda` | int | 0 | Helpdesk |
| `maxAntreanHelpdesk` | int | 0 | Helpdesk |
| `totalWaktuTunggu` | double | 0 | Helpdesk |
| `totalWaktuLayanan` | double | 0 | Helpdesk |

> **Catatan:** `totalBukuDipinjam` sudah ada di base. Untuk Helpdesk, gunakan `totalBukuDipinjamHelpdesk` agar tidak konflik.

### 4.5 Tambah Variabel untuk Entry Flow

| Nama | Type | Initial | Modul |
|---|---|---|---|
| `totalScanKTM` | int | 0 | Entry (baru) |
| `totalDosen` | int | 0 | Entry (baru) |

### 4.6 Fungsi yang SUDAH ADA di Base

Base punya 9 fungsi. Jangan buat duplikat:
- `hitungWaktuServicePeminjaman(ped)` — return double
- `hitungWaktuServicePengembalian(ped)` — return double
- `hitungWaktuBrowsing()` — return double
- `avgWaktuSistem()` — return double
- `avgBukuPerTransaksi()` — return double
- `errorScannerRate()` — return double
- `rataRataDenda()` — return double
- `avgWaktuBrowsing()` — return double
- `tingkatBatalPinjam()` — return double

### 4.7 Tambah Fungsi dari Modul Parkir

Dari palette **Agent**, drag **Function** ke canvas Main:

**Fungsi: `hitungWaktuCariParkirMobil`**
- Return type: `double`
- Parameter: tidak ada

```java
// Mobil membutuhkan waktu 2-8 menit untuk mencari tempat parkir
return uniform(2.0, 8.0);
```

**Fungsi: `hitungWaktuCariParkirMotor`**
- Return type: `double`

```java
// Motor lebih lincah, parkir lebih cepat: 1-3 menit
return uniform(1.0, 3.0);
```

### 4.8 Tambah Fungsi dari Modul Tempat Duduk

**Fungsi: `hitungWaktuToiletSinggah`**
- Return type: `double`
- Parameter: `PengunjungPed ped`

```java
return uniform(2.0, 4.0);
```

**Fungsi: `hitungWaktuCariBuku`**
- Return type: `double`
- Parameter: `PengunjungPed ped`

```java
return uniform(5.0, 15.0);
```

**Fungsi: `hitungWaktuLoker`**
- Return type: `double`
- Parameter: `PengunjungPed ped`

```java
return uniform(1.0, 3.0);
```

**Fungsi: `hitungDurasiBelajar`**
- Return type: `double`
- Parameter: `PengunjungPed ped`

```java
if (ped.zonaDuduk.equals("SEPI")) {
    return exponential(45.0);
} else {
    return exponential(90.0);
}
```

### 4.9 Tambah Fungsi untuk Entry Flow

**Fungsi: `getInterarrivalTime`**
- Return type: `double`
- Parameter: tidak ada

```java
double t = time();
if (t < 20) {
    return exponential(2.0);
} else if (t < 50) {
    return exponential(5.0);
} else {
    return exponential(3.0);
}
```

**Fungsi: `hitungWaktuScanKTM`**
- Return type: `double`
- Parameter: `PengunjungPed ped`

```java
double dasar;
if (ped.isDosen) {
    dasar = uniform(0.2, 0.5);
} else {
    dasar = uniform(0.3, 1.0);
}
return dasar;
```

### 4.10 Tambah Fungsi dari Modul Helpdesk

**Fungsi: `avgWaktuTunggu`**
- Return type: `double`

```java
return totalKeHelpdesk == 0 ? 0 : totalWaktuTunggu / totalKeHelpdesk;
```

**Fungsi: `avgWaktuLayanan`**
- Return type: `double`

```java
return totalKeHelpdesk == 0 ? 0 : totalWaktuLayanan / totalKeHelpdesk;
```

**Fungsi: `tingkatKeberhasilanHelpdesk`**
- Return type: `double`

```java
return totalKeHelpdesk == 0 ? 0
    : 100.0 * (totalBukuTersedia + totalBukuDipinjamHelpdesk) / totalKeHelpdesk;
```

### 4.11 Verifikasi Total Variabel & Fungsi

**Total variabel Main:** 14 (base) + 2 (parkir) + 8 (duduk) + 8 (helpdesk) + 2 (entry) = **34 variabel**

**Total fungsi Main:** 9 (base) + 2 (parkir) + 4 (duduk) + 2 (entry) + 3 (helpdesk) = **20 fungsi**

---

## 5. Langkah 4: Bangun Ulang Entry Flow (Scan KTM + Routing)

**Modul Kedatangan** yang asli pakai Process Modeling Library (`Source`, `Service` dengan resource) — tidak kompatibel dengan Pedestrian Library. Kita harus **membangun ulang** entry flow menggunakan Pedestrian Library.

### 5.1 Hapus Temporary Blocks dari Base

Base project (`Perpustakaan3D_AntriCounter1`) punya `srcMasuk` dan `selectJenisLayanan` yang bersifat temporary. Kita akan ganti dengan entry flow yang lebih lengkap.

1. Klik blok `srcMasuk` → tekan **Delete** (atau klik kanan → Delete)
2. Klik blok `selectJenisLayanan` → **Delete**
3. Klik blok `snkSelesai` → **Delete** (kita akan buat yang baru)
4. Jangan hapus blok lain (goToRakBuku, wKelilingRak, dll)

Sekarang flowchart akan terputus. Kita akan sambungkan lagi.

### 5.2 Buat Blok Entry Baru

Dari palette **Pedestrian Library**, drag blok berikut:

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcJalanKaki` | Pejalan kaki langsung ke perpus |
| PedService | `srvScanKTM` | Antri scan KTM |
| PedSelectOutput | `selectTujuan` | **Hub utama** — 5 output routing |
| PedWait | `wJalanKeluar` | Jalan ke garis keluar |
| PedSelectOutput | `selectPulang` | Routing pulang: parkir atau selesai |
| PedWait | `wJalanBalikParkir` | Jalan balik ke parkir |
| PedSink | `snkSelesai` | Selesai (keluar) |

### 5.3 Koneksi Entry Flow (Sementara)

Hubungkan port untuk testing sementara:

```
srcJalanKaki.out → srvScanKTM.in

srvScanKTM.out → selectTujuan.in

selectTujuan.out1 (25%) → wJalanKeluar.in (SEMENTARA)
selectTujuan.out2 (20%) → wJalanKeluar.in (SEMENTARA)
selectTujuan.out3 (30%) → wJalanKeluar.in (SEMENTARA)
selectTujuan.out4 (10%) → wJalanKeluar.in (SEMENTARA)
selectTujuan.out5 (15%) → wJalanKeluar.in (SEMENTARA)

wJalanKeluar.out → selectPulang.in

selectPulang.out1 (isParkir=true) → wJalanBalikParkir.in → snkSelesai.in
selectPulang.out2 (isParkir=false) → snkSelesai.in
```

> Koneksi sementara ini akan diganti ke modul masing-masing nanti.

### 5.4 Konfigurasi `srcJalanKaki`

Klik blok `srcJalanKaki` → atur Properties:

| Property | Nilai |
|---|---|
| `Appears at` | `line` |
| `Target line` | `entryLine` |
| `Arrive according to` | `Interarrival time` |
| `Interarrival time` | `getInterarrivalTime()` |
| `New pedestrian` | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "J-" + seqPed;
ped.tMasuk = time();
ped.isParkir = false;
ped.isDosen = uniform(0, 1) < 0.2; // 20% dosen

if (ped.isDosen) {
    totalDosen++;
} else {
    totalMahasiswa++;
}

traceln("ARRIVE " + ped.idPed + " | " + (ped.isDosen ? "DOSEN" : "MHS"));
```

### 5.5 Konfigurasi `srvScanKTM`

| Property | Nilai |
|---|---|
| `Services` | `svcScanKTM` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungWaktuScanKTM(ped)` |
| `Recovery delay` | `0` |

**Action On end service:**
```java
totalScanKTM++;
traceln("SCAN " + ped.idPed + " | valid");
```

### 5.6 Konfigurasi `selectTujuan`

| Property | Nilai |
|---|---|
| `N outputs` | `5` |
| `Use probabilities` | Centang |
| Output 1 | `0.25` (PINJAM) |
| Output 2 | `0.20` (KEMBALI) |
| Output 3 | `0.30` (CARI DUDUK) |
| Output 4 | `0.10` (HELPDESK) |
| Output 5 | `0.15` (LANGSUNG KELUAR) |

> **Verifikasi:** 0.25 + 0.20 + 0.30 + 0.10 + 0.15 = **1.0** ✅

**Action per On exit port:**

- **On exit port 1:**
```java
ped.tujuanLayanan = "PINJAM";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 2:**
```java
ped.tujuanLayanan = "KEMBALI";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 3:**
```java
ped.tujuanLayanan = "CARIDUDUK";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 4:**
```java
ped.tujuanLayanan = "HELPDESK";
ped.butuhHelpdesk = true;
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 5:**
```java
ped.tujuanLayanan = "LANGSUNG";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

### 5.7 Konfigurasi `wJalanKeluar`

| Property | Nilai |
|---|---|
| `Waiting location` | `Target line` |
| `Target line` | `exitLine` |
| `End of delay` | `On delay time expiry` |
| `Delay time` | `0.2` |

### 5.8 Konfigurasi `selectPulang`

⚠️ **GUNAKAN KONDISI, BUKAN PROBABILITAS**

| Property | Nilai |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | **JANGAN centang** |
| `Use conditions` | **Centang** |
| Output 1 condition | `ped.isParkir == true` |
| Output 2 condition | `ped.isParkir == false` |

### 5.9 Konfigurasi `snkSelesai`

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem)
    + " | tujuan=" + ped.tujuanLayanan);
```

### 5.10 Konfigurasi `wJalanBalikParkir`

| Property | Nilai |
|---|---|
| `Waiting location` | `Target line` |
| `Target line` | `exitLine` (sementara) |
| `Delay time` | `0.5` |

---

## 6. Langkah 5: Integrasi Modul Parkir

Modul Parkir (`SimulasiParkirPerpustakaan`) sudah dalam MINUTE dan menggunakan **Road Traffic Library** untuk mobil + **Pedestrian Library** untuk orang + **Process Modeling Library** untuk Delay/SelectOutput.

### 6.1 Arsitektur Parkir — Konsep Dasar

Di AnyLogic, **Road Traffic Library** punya blok khusus untuk kendaraan:
- **CarSource** — sumber mobil/motor (seperti PedSource tapi untuk kendaraan)
- **CarMoveTo** — perintah jalan ke tujuan (seperti PedGoTo tapi untuk kendaraan)
- **CarDispose** — buang kendaraan (seperti PedSink)

> ⚠️ **PENTING:** `CarMoveTo` berbeda dengan `PedService`. CarMoveTo **TIDAK** punya properti `Delay time`, `On begin service`, atau `On end service`. CarMoveTo hanya menggerakkan mobil dari titik A ke titik B di jalan raya. Waktu tempuh = jarak / kecepatan mobil.

Untuk mensimulasikan "waktu cari parkir", kita gunakan blok **Delay** (dari Process Modeling Library) setelah mobil mencapai tempat parkir.

### 6.2 Diagram Alur Parkir — Mobil

```
srcParkirMobil (CarSource)
    │  mobil muncul di road
    ▼
moveKeParkirMobil (CarMoveTo)
    │  mobil jalan ke parkingLot
    │  onExit: trigger orangDatang.inject(1)
    ▼
cariParkirMobil (Delay)
    │  delay = hitungWaktuCariParkirMobil() — simulasi cari tempat
    ▼
keluarParkirMobil (CarMoveTo)
    │  mobil jalan keluar
    ▼
buangMobil (CarDispose)
    │  mobil dihapus
```

### 6.3 Diagram Alur Parkir — Motor

```
srcParkirMotor (CarSource)
    ▼
moveKeParkirMotor (CarMoveTo)
    │  onExit: trigger orangDatang.inject(1)
    ▼
cariParkirMotor (Delay)
    │  delay = hitungWaktuCariParkirMotor()
    ▼
keluarParkirMotor (CarMoveTo)
    ▼
buangMotor (CarDispose)
```

### 6.4 Diagram Alur — Pejalan Kaki dari Parkir

```
orangDatang (PedSource — MANUAL, dipicu inject())
    ▼
jalanKePerpus (PedGoTo)
    ▼
srvScanKTM.in (bergabung dengan jalur pejalan kaki lain)
```

### 6.5 Diagram Alur — Kembali ke Parkir (Pulang)

```
selectPulang.out1 (isParkir == true)
    ▼
jalanKeParkiran (PedGoTo — jalan ke area parkir)
    ▼
snkParkirKeluar (PedSink — selesai)
```

### 6.6 Buat Markup untuk Parkir

Sebelum membuat blok, siapkan markup Road Traffic:

1. Dari palette **Space Markup**, drag **Road** ke canvas Main → rename `road`
   - Gambar garis road (panjang sesuai kebutuhan, misal dari X=0 ke X=30)
   - Atur jumlah lane (misal 2 lane)

2. Dari palette **Space Markup**, drag **Parking Lot** → rename `parkingLot`
   - Letakkan di samping road
   - Atur jumlah kapasitas (misal total 20: 10 mobil + 10 motor)
   - Atau buat 2 parking lot terpisah: `parkingLotMobil` dan `parkingLotMotor`

3. Dari palette **Space Markup**, drag **Stop Line** → rename `garisTunggu` (opsional)
   - Letakkan di road sebelum parking lot

4. **TargetLine** → `areaPejalanKaki` (area pejalan kaki dari parkir ke perpus)

### 6.7 Blok yang Harus Dibuat

Dari **Road Traffic Library**:

| Blok | Rename | Fungsi |
|---|---|---|
| CarSource | `srcParkirMobil` | Mobil masuk parkir |
| CarSource | `srcParkirMotor` | Motor masuk parkir |
| CarMoveTo | `moveKeParkirMobil` | Mobil jalan ke tempat parkir |
| CarMoveTo | `moveKeParkirMotor` | Motor jalan ke tempat parkir |
| CarMoveTo | `keluarParkirMobil` | Mobil jalan keluar parkir |
| CarMoveTo | `keluarParkirMotor` | Motor jalan keluar parkir |
| CarDispose | `buangMobil` | Mobil dihapus |
| CarDispose | `buangMotor` | Motor dihapus |

Dari **Process Modeling Library** (untuk delay):

| Blok | Rename | Fungsi |
|---|---|---|
| Delay | `cariParkirMobil` | Waktu cari parkir mobil |
| Delay | `cariParkirMotor` | Waktu cari parkir motor |

Dari **Pedestrian Library** (untuk orang):

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `orangDatang` | Orang dari parkir (dipicu manual) |
| PedGoTo | `jalanKePerpus` | Jalan dari parkir ke perpus |
| PedGoTo | `jalanKeParkiran` | Jalan dari perpus ke parkir |
| PedSink | `snkParkirKeluar` | Selesai parkir |

### 6.8 Cara Membuat Blok

Semua blok dibuat langsung di canvas **Main** project `Perpustakaan3D_Full.alp`:

**Langkah per langkah:**

1. Buka palette **Road Traffic Library**
2. Drag **CarSource** ke canvas → rename `srcParkirMobil`
3. Drag **CarSource** lagi → rename `srcParkirMotor`
4. Drag **CarMoveTo** → rename `moveKeParkirMobil`
5. Drag **CarMoveTo** → rename `moveKeParkirMotor`
6. Drag **CarMoveTo** → rename `keluarParkirMobil`
7. Drag **CarMoveTo** → rename `keluarParkirMotor`
8. Drag **CarDispose** → rename `buangMobil`
9. Drag **CarDispose** → rename `buangMotor`
10. Buka palette **Process Modeling Library**
11. Drag **Delay** → rename `cariParkirMobil`
12. Drag **Delay** → rename `cariParkirMotor`
13. Buka palette **Pedestrian Library**
14. Drag **PedSource** → rename `orangDatang`
15. Drag **PedGoTo** → rename `jalanKePerpus`
16. Drag **PedGoTo** → rename `jalanKeParkiran`
17. Drag **PedSink** → rename `snkParkirKeluar`

### 6.9 Konfigurasi `srcParkirMobil` (CarSource)

Klik blok `srcParkirMobil`. Di panel **Properties**, atur:

**Tab General:**
| Property | Nilai | Penjelasan |
|---|---|---|
| `Appears at` | `road` | Mobil muncul di jalan raya |
| `Road` | `road` | Pilih markup road yang sudah dibuat |

**Tab Arrivals:**
| Property | Nilai | Penjelasan |
|---|---|---|
| `Arrive according to` | `Interarrival time` | Diatur waktu antar kedatangan |
| `Interarrival time` | `exponential(0.5)` | Rata-rata 1 mobil setiap 2 menit |

**Tab Actions → On exit:**
```java
seqPed++;
// Catat: CarSource menghasilkan agent Car, bukan pedestrian.
// Kita simpan data di variabel global agar bisa dipakai pedestrian nanti
tempIdPed = "Mobil-" + seqPed;
tempIsParkir = true;
tempJenisKendaraan = "MOBIL";

totalMobil++;

traceln("MOBIL datang: " + tempIdPed + " | waktu=" + String.format("%.1f", time()) + " mnt");
```

> **Catatan:** `seqPed++` menggunakan variabel yang sudah ada di Main. `tempIdPed`, `tempIsParkir`, `tempJenisKendaraan` adalah variabel global baru yang perlu dibuat di Main (String, boolean, String).

### 6.10 Konfigurasi `srcParkirMotor` (CarSource)

**Tab Arrivals:**
| Property | Nilai |
|---|---|
| `Interarrival time` | `exponential(0.3)` | Rata-rata 1 motor setiap ~3.3 menit |

**Tab Actions → On exit:**
```java
seqPed++;
tempIdPed = "Motor-" + seqPed;
tempIsParkir = true;
tempJenisKendaraan = "MOTOR";

totalMotor++;

traceln("MOTOR datang: " + tempIdPed + " | waktu=" + String.format("%.1f", time()) + " mnt");
```

### 6.11 Konfigurasi `moveKeParkirMobil` (CarMoveTo)

> ⚠️ **Perhatikan:** `CarMoveTo` **TIDAK** memiliki `Delay time`, `On begin service`, atau `On end service`. Blok ini hanya menggerakkan mobil ke target.

**Tab General:**
| Property | Nilai | Penjelasan |
|---|---|---|
| `Target type` | `parkingLot` | Target adalah tempat parkir |
| `Target parking lot` | `parkingLot` | Pilih markup parkingLot |
| `Speed` | (kosongkan) | Gunakan kecepatan default mobil (1 MPS) |

**Tab Actions → On exit** (saat mobil sampai di parkir):
```java
// Trigger pedestrian spawn — orang turun dari mobil
orangDatang.inject(1);

traceln("PARKIR Mobil: " + tempIdPed + " selesai parkir — orang jalan ke perpus");
```

### 6.12 Konfigurasi `moveKeParkirMotor` (CarMoveTo)

**Tab General:**
| Property | Nilai |
|---|---|
| `Target type` | `parkingLot` |
| `Target parking lot` | `parkingLot` |

**Tab Actions → On exit:**
```java
orangDatang.inject(1);
traceln("PARKIR Motor: " + tempIdPed + " selesai parkir — orang jalan ke perpus");
```

### 6.13 Konfigurasi `cariParkirMobil` (Delay — Process Modeling)

> Blok `Delay` dari **Process Modeling Library** digunakan untuk mensimulasikan waktu yang dihabiskan mencari tempat parkir dan waktu mobil parkir.

**Tab General:**
| Property | Nilai | Penjelasan |
|---|---|---|
| `Delay time` | `hitungWaktuCariParkirMobil()` | Panggil fungsi yang sudah dibuat |
| `Maximum capacity` | Centang | Biarkan tak terbatas |

### 6.14 Konfigurasi `cariParkirMotor` (Delay)

**Tab General:**
| Property | Nilai |
|---|---|
| `Delay time` | `hitungWaktuCariParkirMotor()` |
| `Maximum capacity` | Centang |

### 6.15 Konfigurasi `keluarParkirMobil` (CarMoveTo) & `keluarParkirMotor` (CarMoveTo)

**Tab General — `keluarParkirMobil`:**
| Property | Nilai |
|---|---|
| `Target type` | `road` |
| `Target road` | `road` |

**Tab General — `keluarParkirMotor`:**
| Property | Nilai |
|---|---|
| `Target type` | `road` |
| `Target road` | `road` |

### 6.16 Konfigurasi `buangMobil` & `buangMotor` (CarDispose)

Tidak ada properti khusus yang perlu diatur. Cukup rename saja.

### 6.17 Konfigurasi `orangDatang` (PedSource — dipicu MANUAL)

PedSource ini tidak berjalan otomatis. Dipicu oleh `inject(1)` dari blok CarMoveTo.

**Tab General:**
| Property | Nilai | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di target line |
| `Target line` | `areaPejalanKaki` | Garis area pejalan kaki |
| `Arrive according to` | `Rate` | Mode rate (tapi rate = 0, manual) |
| `Rate` | `0` | **0 = tidak ada kedatangan otomatis** |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian |

**Tab Actions → On exit:**
```java
// Ambil data dari variabel global yang di-set oleh CarSource
ped.idPed = tempIdPed;
ped.tMasuk = time();
ped.isParkir = tempIsParkir;
ped.jenisKendaraan = tempJenisKendaraan;
ped.waktuParkir = time();

traceln("ORANG " + ped.idPed + " jalan dari parkir ke perpus");
```

### 6.18 Konfigurasi `jalanKePerpus` (PedGoTo)

| Property | Nilai | Penjelasan |
|---|---|---|
| `Target type` | `Attractor` or `Node` | Pilih node |
| `Target node` | (pilih node dekat `srvScanKTM`) | Biarkan pedestrian jalan ke area scan KTM |

> **Tips:** Buat **PointNode** baru di markup beri nama `nodePintuPerpus`, letakkan di dekat entryLine. Jadikan node ini sebagai target `jalanKePerpus`.

### 6.19 Konfigurasi `jalanKeParkiran` (PedGoTo — untuk pulang)

**Tab General:**
| Property | Nilai |
|---|---|
| `Target type` | `line` |
| `Target line` | `areaPejalanKaki` |

### 6.20 Konfigurasi `snkParkirKeluar` (PedSink)

Tidak perlu properti khusus. Bisa ditambahkan action on enter jika ingin tracking:
```java
traceln("PULANG " + ped.idPed + " — sampai di parkir, naik kendaraan");
```

### 6.21 Tambah Variabel Global di Main

Untuk passing data dari Car ke Pedestrian, buat 3 variabel baru di **Main**:

| Nama | Type | Initial | Fungsi |
|---|---|---|---|
| `tempIdPed` | String | `""` | Menyimpan ID pedestrian sementara |
| `tempIsParkir` | boolean | `false` | Menyimpan status parkir sementara |
| `tempJenisKendaraan` | String | `""` | Menyimpan jenis kendaraan sementara |

### 6.22 Koneksi Flow Parkir

Hubungkan port (drag dari port out ke port in):

**Flow Mobil:**
```
srcParkirMobil.out → moveKeParkirMobil.in → cariParkirMobil.in → keluarParkirMobil.in → buangMobil.in
```

**Flow Motor:**
```
srcParkirMotor.out → moveKeParkirMotor.in → cariParkirMotor.in → keluarParkirMotor.in → buangMotor.in
```

**Flow Pejalan Kaki (berangkat):**
```
orangDatang.out → jalanKePerpus.in → srvScanKTM.in
```
> `jalanKePerpus.out` colok ke `srvScanKTM.in` (bergabung dengan `srcJalanKaki.out`)

**Flow Pejalan Kaki (pulang):**
```
selectPulang.out1 (isParkir==true) → jalanKeParkiran.in → snkParkirKeluar.in
```

### 6.23 Verifikasi Flow Parkir

Pastikan:
- `srcParkirMobil` → CarSource dengan `road` dan `exponential(0.5)`
- `moveKeParkirMobil` → CarMoveTo dengan target `parkingLot`, **BUKAN PedService**
- `cariParkirMobil` → Delay (Process Modeling), **BUKAN CarMoveTo**
- `keluarParkirMobil` → CarMoveTo dengan target `road`
- `buangMobil` → CarDispose
- `orangDatang.inject(1)` dipanggil di **On exit** `moveKeParkirMobil` (bukan On end service)
- `jalanKePerpus.out` colok ke `srvScanKTM.in`
- `selectPulang.out1` colok ke `jalanKeParkiran.in`

### 6.24 Test Parkir

1. **Stop time** = `10` menit
2. Run
3. Cek console: apakah ada `MOBIL datang`, `PARKIR Mobil`, `ORANG ... jalan dari parkir`?
4. Jika error `parkingLot not found` → cek markup Parking Lot sudah dibuat
5. Jika error `road not found` → cek markup Road sudah dibuat
6. Jika orang dari parkir tidak muncul → cek `orangDatang.inject(1)` ada di onExit CarMoveTo

---

## 7. Langkah 6: Integrasi Modul Tempat Duduk

Modul Cari Tempat Duduk (`ModelPerpus_3D_fix`) menggunakan Pedestrian Library dengan time unit **SECOND**. Kita perlu mengonversi ke **MINUTE** saat memindahkan ke base project.

### 7.1 Blok yang Akan Dibuat

Dari palette **Pedestrian Library**, buat blok berikut di `Perpustakaan3D_Full.alp`:

| Blok | Rename | Fungsi |
|---|---|---|
| PedSelectOutput | `selectAktivitas` | 4 output: pilih aktivitas sebelum duduk |
| PedService | `srvToiletSinggah` | Mampir toilet 2-4 menit |
| PedService | `srvCariBuku` | Cari/eksplor buku 5-15 menit |
| PedService | `srvLoker` | Simpan barang di loker 1-3 menit |
| PedSelectOutput | `selectZonaDuduk` | 2 output: pilih zona sepi/diskusi |
| PedService | `srvDudukSepi` | Belajar di silent zone |
| PedService | `srvDudukDiskusi` | Belajar di diskusi zone |

### 7.2 Konfigurasi `selectAktivitas`

| Property | Nilai |
|---|---|
| `N outputs` | `4` |
| `Use probabilities` | Centang |
| Output 1 | `0.15` (15% ke toilet singgah) |
| Output 2 | `0.30` (30% cari buku) |
| Output 3 | `0.25` (25% loker) |
| Output 4 | `0.30` (30% langsung ke kursi) |

> Verifikasi: 0.15 + 0.30 + 0.25 + 0.30 = **1.0** ✅

### 7.3 Konfigurasi `srvToiletSinggah`

| Property | Nilai |
|---|---|
| `Services` | `svcToiletSinggah` |
| `Delay time` | `hitungWaktuToiletSinggah(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "TOILET";
```

**Action On end service:**
```java
totalToiletSinggah++;
traceln("AKTIVITAS " + ped.idPed + " | TOILET | selesai");
```

### 7.4 Konfigurasi `srvCariBuku`

| Property | Nilai |
|---|---|
| `Services` | `svcCariBuku` |
| `Delay time` | `hitungWaktuCariBuku(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "BUKU";
```

**Action On end service:**
```java
totalCariBuku++;
traceln("AKTIVITAS " + ped.idPed + " | BUKU | selesai");
```

### 7.5 Konfigurasi `srvLoker`

| Property | Nilai |
|---|---|
| `Services` | `svcLoker` |
| `Delay time` | `hitungWaktuLoker(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "LOKER";
```

**Action On end service:**
```java
totalLoker++;
traceln("AKTIVITAS " + ped.idPed + " | LOKER | selesai");
```

### 7.6 Konfigurasi `selectZonaDuduk`

| Property | Nilai |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | Centang |
| Output 1 | `0.60` (60% silent zone) |
| Output 2 | `0.40` (40% diskusi zone) |

### 7.7 Konfigurasi `srvDudukSepi`

| Property | Nilai |
|---|---|
| `Services` | `svcAreaSepi` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungDurasiBelajar(ped)` |

**Action On enter queue:**
```java
int q = srvDudukSepi.queueSize();
if (q > maxAntrianDudukSepi) {
    maxAntrianDudukSepi = q;
}
```

**Action On begin service:**
```java
ped.waktuMulaiDuduk = time();
ped.zonaDuduk = "SEPI";
```

**Action On end service:**
```java
ped.durasiBelajar = time() - ped.waktuMulaiDuduk;
totalDudukSepi++;
totalDurasiBelajar += ped.durasiBelajar;
traceln("BELAJAR " + ped.idPed + " | zona=SEPI | durasi=" + String.format("%.1f", ped.durasiBelajar) + " mnt");
```

### 7.8 Konfigurasi `srvDudukDiskusi`

| Property | Nilai |
|---|---|
| `Services` | `svcAreaDiskusi` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungDurasiBelajar(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiDuduk = time();
ped.zonaDuduk = "DISKUSI";
```

**Action On end service:**
```java
ped.durasiBelajar = time() - ped.waktuMulaiDuduk;
totalDudukDiskusi++;
totalDurasiBelajar += ped.durasiBelajar;
traceln("BELAJAR " + ped.idPed + " | zona=DISKUSI | durasi=" + String.format("%.1f", ped.durasiBelajar) + " mnt");
```

### 7.9 Koneksi Flow Tempat Duduk

```
selectTujuan.out3 → selectAktivitas.in

selectAktivitas.out1 (15%) → srvToiletSinggah.in
selectAktivitas.out2 (30%) → srvCariBuku.in
selectAktivitas.out3 (25%) → srvLoker.in
selectAktivitas.out4 (30%) → LANGSUNG → selectZonaDuduk.in

srvToiletSinggah.out → selectZonaDuduk.in
srvCariBuku.out → selectZonaDuduk.in
srvLoker.out → selectZonaDuduk.in

selectZonaDuduk.out1 (60%) → srvDudukSepi.in
selectZonaDuduk.out2 (40%) → srvDudukDiskusi.in

srvDudukSepi.out → wJalanKeluar.in
srvDudukDiskusi.out → wJalanKeluar.in
```

---

## 8. Langkah 7: Integrasi Modul Helpdesk

Modul Helpdesk (`Perpustakaan3D_Helpdesk1.alp`) sudah dalam MINUTE dan menggunakan Pedestrian Library. Ada satu blok `antreHelpdesk` yang menggunakan **Queue dari Process Modeling Library** — kita perlu menggantinya dengan PedQueue.

### 8.1 Blok yang Akan Dibuat

| Blok | Rename | Fungsi |
|---|---|---|
| PedGoTo | `goToHelpdesk` | Jalan ke area helpdesk |
| PedQueue | `antreHelpdesk` | Antrean helpdesk |
| PedService | `srvHelpdesk` | Pelayanan petugas helpdesk |
| PedSelectOutput | `selectStatusBuku` | 3 output: status buku |
| PedGoTo | `hdGoToRakBuku` | Jalan ke rak buku |
| PedGoTo | `hdGoToPinjam` | Jalan ke area peminjaman |

### 8.2 Konfigurasi `goToHelpdesk`

| Property | Nilai |
|---|---|
| `Target` | `areaAntreHelpdesk` (RectangularNode) |

### 8.3 Konfigurasi `antreHelpdesk` (PedQueue)

| Property | Nilai |
|---|---|
| `Queue capacity` | Kosong (unlimited) |
| `Waiting location` | `areaAntreHelpdesk` |
| `Enable exit on timeout` | JANGAN centang |

**Action On enter:**
```java
ped.waktuMulaiAntre = time();

int q = antreHelpdesk.size();
if (q > maxAntreanHelpdesk) {
    maxAntreanHelpdesk = q;
}

traceln("ANTRE " + ped.idPed + " di helpdesk | posisi=" + q);
```

### 8.4 Konfigurasi `srvHelpdesk`

| Property | Nilai |
|---|---|
| `Services` | `svcHelpdesk` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `triangular(2, 4, 6)` |
| `Recovery delay` | `0` |

**Action On begin service:**
```java
ped.waktuMulaiLayanan = time();
ped.waktuTunggu = ped.waktuMulaiLayanan - ped.waktuMulaiAntre;

totalWaktuTunggu += ped.waktuTunggu;

traceln("LAYANI " + ped.idPed + " di helpdesk | tunggu=" + String.format("%.1f", ped.waktuTunggu) + " mnt");
```

**Action On end service:**
```java
ped.waktuSelesaiLayanan = time();
ped.waktuLayanan = ped.waktuSelesaiLayanan - ped.waktuMulaiLayanan;

totalKeHelpdesk++;
totalWaktuLayanan += ped.waktuLayanan;

traceln("SELESAI_LAYANAN " + ped.idPed + " | durasi=" + String.format("%.1f", ped.waktuLayanan) + " mnt");
```

### 8.5 Konfigurasi `selectStatusBuku`

| Property | Nilai |
|---|---|
| `N outputs` | `3` |
| `Use probabilities` | Centang |
| Output 1 | `0.40` (40% buku tersedia) |
| Output 2 | `0.35` (35% sedang dipinjam) |
| Output 3 | `0.25` (25% tidak ditemukan) |

> Verifikasi: 0.40 + 0.35 + 0.25 = **1.0** ✅

**On exit port 1:**
```java
ped.statusBuku = "TERSEDIA";
totalBukuTersedia++;
traceln("STATUS " + ped.idPed + " | BUKU TERSEDIA → ke rak buku");
```

**On exit port 2:**
```java
ped.statusBuku = "DIPINJAM";
totalBukuDipinjamHelpdesk++;
traceln("STATUS " + ped.idPed + " | SEDANG DIPINJAM → reservasi");
```

**On exit port 3:**
```java
ped.statusBuku = "TIDAK_ADA";
totalBukuTidakAda++;
traceln("STATUS " + ped.idPed + " | TIDAK DITEMUKAN → keluar");
```

### 8.6 Konfigurasi `hdGoToRakBuku`

| Property | Nilai |
|---|---|
| `Target` | `nodeRakBuku` |

### 8.7 Konfigurasi `hdGoToPinjam`

| Property | Nilai |
|---|---|
| `Target` | `nodePinjam` |

### 8.8 Koneksi Flow Helpdesk

```
selectTujuan.out4 → goToHelpdesk.in → antreHelpdesk.in → srvHelpdesk.in

srvHelpdesk.out → selectStatusBuku.in

selectStatusBuku.out1 (40%, tersedia) → hdGoToRakBuku.in → hdGoToPinjam.in → srvPinjam.in (Modul Peminjaman)
selectStatusBuku.out2 (35%, dipinjam) → hdGoToPinjam.in → srvPinjam.in (Modul Peminjaman)
selectStatusBuku.out3 (25%, tidak ada) → wJalanKeluar.in
```

> **Catatan penting:** `srvPinjam` sudah ada di base project (dari Modul Peminjaman). Output helpdesk yang mau pinjam buku diarahkan ke `srvPinjam.in`.

---

## 9. Langkah 8: Merge Markup & Layout 3D

### 9.1 Markup yang SUDAH ADA di Base

Base project sudah punya markup ini:
- `entryLine` — TargetLine (garis masuk)
- `nodeRakBuku` — PointNode/RectangularNode
- `nodeCounterPinjam` — PointNode
- `nodeCounterKembali` — PointNode
- `svcPeminjaman` — Service with Lines (2 services)
- `svcPengembalian` — Service with Lines (1 service)

### 9.2 Markup yang Perlu Ditambah

| Markup | Type | Services | Queues | Untuk Modul |
|---|---|---|---|---|
| `svcScanKTM` | Service with Lines | 2 | 2 | Entry |
| `exitLine` | TargetLine | — | — | Entry |
| `svcToiletSinggah` | Service with Lines | 2 | 1 | Tempat Duduk |
| `svcCariBuku` | Service with Lines | 2 | 2 | Tempat Duduk |
| `svcLoker` | Service with Lines | 3 | 1 | Tempat Duduk |
| `svcAreaSepi` | Service with Lines | 10 | 2 | Tempat Duduk |
| `svcAreaDiskusi` | Service with Lines | 4 | 2 | Tempat Duduk |
| `nodeHelpdesk` | PointNode | — | — | Helpdesk |
| `areaAntreHelpdesk` | RectangularNode | — | — | Helpdesk |
| `svcHelpdesk` | Service with Lines | 2 | 1 | Helpdesk |
| `nodePinjam` | PointNode | — | — | Helpdesk |
| `areaPejalanKaki` | TargetLine | — | — | Parkir |

### 9.3 Cara Membuat Service with Lines

1. Dari palette **Space Markup**, drag **Service with Lines** ke canvas Main
2. Rename sesuai tabel
3. Atur **Number of services** dan **N of queues**
4. Letakkan service point (lingkaran hijau) di posisi yang sesuai
5. Tarik queue line (garis antrean) menjauh dari service point

### 9.4 Layout Posisi (Koordinat Contoh)

```
Y=24  [svcAreaDiskusi - 4 meja]
Y=20  [svcAreaSepi - 10 kursi]     [svcLoker]
Y=16  [svcCariBuku]     [svcToiletSinggah]
Y=12  [nodeRakBuku]     [nodeHelpdesk] [svcHelpdesk]
Y=8   [svcPeminjaman]   [svcPengembalian]
Y=4   [svcScanKTM]
Y=2   [entryLine]                      [exitLine]
      X=2     X=6     X=10    X=14    X=18    X=22
```

### 9.5 3D Objects

Base project sudah punya objek 3D dari Modul Peminjaman (person.dae, couch, locker, table, chair, reception). Objek 3D tidak perlu di-copy dari modul lain kecuali ingin ditambah.

**Cara tambah objek 3D baru:**
1. Palette **3D Objects** → **Box** → buat meja, kursi, rak buku sesuai kebutuhan
2. Atur posisi (X, Y) agar sesuai dengan markup
3. Atur ukuran (Size X, Y, Z) agar proporsional

---

## 10. Langkah 9: Hubungkan Semua Rute Final

### 10.1 Diagram Flowchart Lengkap

Sekarang kita sambungkan SEMUA modul menjadi satu kesatuan:

```
=== ENTRY FLOW ===
srcJalanKaki.out → srvScanKTM.in
orangDatang.out → pedJalanKePerpus.in → srvScanKTM.in

=== SCAN KTM → ROUTING ===
srvScanKTM.out → selectTujuan.in

=== ROUTING 5 OUTPUT ===
selectTujuan.out1 (25%, PINJAM) → goToRakBuku.in [DARI BASE]
selectTujuan.out2 (20%, KEMBALI) → goToCounterKembali.in [DARI BASE]
selectTujuan.out3 (30%, CARI DUDUK) → selectAktivitas.in
selectTujuan.out4 (10%, HELPDESK) → goToHelpdesk.in
selectTujuan.out5 (15%, LANGSUNG) → wJalanKeluar.in

=== PINJAM (DARI BASE) ===
goToRakBuku.out → wKelilingRak.in → goToCounterPinjam.in → selectJadiPinjam.in
selectJadiPinjam.out1 (90%) → srvPinjam.in → wJalanKeluar.in
selectJadiPinjam.out2 (10%) → wJalanKeluar.in

=== KEMBALI (DARI BASE) ===
goToCounterKembali.out → srvKembali.in → wJalanKeluar.in

=== CARI DUDUK ===
selectAktivitas.out1 (15%) → srvToiletSinggah.in → selectZonaDuduk.in
selectAktivitas.out2 (30%) → srvCariBuku.in → selectZonaDuduk.in
selectAktivitas.out3 (25%) → srvLoker.in → selectZonaDuduk.in
selectAktivitas.out4 (30%) → (LANGSUNG) → selectZonaDuduk.in
selectZonaDuduk.out1 (60%) → srvDudukSepi.in → wJalanKeluar.in
selectZonaDuduk.out2 (40%) → srvDudukDiskusi.in → wJalanKeluar.in

=== HELPDESK ===
goToHelpdesk.out → antreHelpdesk.in → srvHelpdesk.in → selectStatusBuku.in
selectStatusBuku.out1 (40%) → hdGoToRakBuku.in → hdGoToPinjam.in → srvPinjam.in
selectStatusBuku.out2 (35%) → hdGoToPinjam.in → srvPinjam.in
selectStatusBuku.out3 (25%) → wJalanKeluar.in

=== PULANG ===
wJalanKeluar.out → selectPulang.in
selectPulang.out1 (isParkir=true) → wJalanBalikParkir.in → jalanKeParkiran.in
selectPulang.out2 (isParkir=false) → snkSelesai.in
```

### 10.2 Verifikasi Semua Koneksi

Buat checklist dengan cara centang manual di AnyLogic:

- [ ] `srcJalanKaki.out → srvScanKTM.in`
- [ ] `srvScanKTM.out → selectTujuan.in`
- [ ] `selectTujuan.out1 → goToRakBuku.in`
- [ ] `selectTujuan.out2 → goToCounterKembali.in`
- [ ] `selectTujuan.out3 → selectAktivitas.in`
- [ ] `selectTujuan.out4 → goToHelpdesk.in`
- [ ] `selectTujuan.out5 → wJalanKeluar.in`
- [ ] Semua aktivitas → `selectZonaDuduk.in`
- [ ] `selectZonaDuduk.out1 → srvDudukSepi.in`
- [ ] `selectZonaDuduk.out2 → srvDudukDiskusi.in`
- [ ] `srvDudukSepi.out → wJalanKeluar.in`
- [ ] `srvDudukDiskusi.out → wJalanKeluar.in`
- [ ] `goToHelpdesk.out → antreHelpdesk.in`
- [ ] `antreHelpdesk.out → srvHelpdesk.in`
- [ ] `srvHelpdesk.out → selectStatusBuku.in`
- [ ] Semua helpdesk output ke tujuan masing-masing
- [ ] `wJalanKeluar.out → selectPulang.in`
- [ ] `selectPulang.out1 → wJalanBalikParkir.in`
- [ ] `selectPulang.out2 → snkSelesai.in`

### 10.3 Verifikasi Probabilitas

| SelectOutput | Output 1 | Output 2 | Output 3 | Output 4 | Output 5 | Total |
|---|---|---|---|---|---|---|
| `selectTujuan` | 0.25 | 0.20 | 0.30 | 0.10 | 0.15 | **1.0** |
| `selectJadiPinjam` | 0.90 | 0.10 | — | — | — | **1.0** |
| `selectAktivitas` | 0.15 | 0.30 | 0.25 | 0.30 | — | **1.0** |
| `selectZonaDuduk` | 0.60 | 0.40 | — | — | — | **1.0** |
| `selectStatusBuku` | 0.40 | 0.35 | 0.25 | — | — | **1.0** |

`selectPulang` harus pakai **KONDISI** (bukan probabilitas):
- Out 1: `ped.isParkir == true`
- Out 2: `ped.isParkir == false`

---

## 11. Langkah 10: Dashboard & Testing

### 11.1 Dashboard Monitoring

Tambahkan **Text** dinamis di Main (Palette **Presentation** → **Text** → Type: `Dynamic`):

```java
// Text 1 — Judul
"SIMULASI PERPUSTAKAAN 3D — INTEGRASI 5 MODUL"
```

```java
// Text 2 — Total pengunjung
"Total: " + totalSelesai + " | Jalan: " + (totalSelesai) + " | Parkir: " + (totalMobil + totalMotor)
```

```java
// Text 3 — Entry & Scan
"SCAN KTM: " + totalScanKTM + " | Antrian: " + srvScanKTM.queueSize()
```

```java
// Text 4 — Peminjaman & Pengembalian
"PINJAM: " + totalPeminjaman + " (max antri: " + maxQueuePeminjaman + ")"
+ " | KEMBALI: " + totalPengembalian + " (max antri: " + maxQueuePengembalian + ")"
+ " | Buku: " + totalBukuDipinjam
+ " | Error: " + totalErrorScanner
+ " | Denda: Rp " + String.format("%,.0f", totalDenda)
```

```java
// Text 5 — Tempat Duduk
"BELAJAR: " + (totalDudukSepi + totalDudukDiskusi)
+ " | Sepi: " + totalDudukSepi + " | Diskusi: " + totalDudukDiskusi
+ " | Antri sepi: " + srvDudukSepi.queueSize() + " | Antri diskusi: " + srvDudukDiskusi.queueSize()
```

```java
// Text 6 — Helpdesk
"HELPDESK: " + totalKeHelpdesk
+ " | Tersedia: " + totalBukuTersedia
+ " | Dipinjam: " + totalBukuDipinjamHelpdesk
+ " | Tdk Ada: " + totalBukuTidakAda
+ " | Antri: " + antreHelpdesk.size()
```

```java
// Text 7 — Waktu sistem
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

### 11.2 Test 1: Run 10 Menit (Verifikasi Dasar)

1. Buka experiment **Simulation**
2. **Stop time** = `10` menit
3. Klik **Run**
4. Amati di console: apakah ada log `ARRIVE`, `SCAN`, `TUJUAN`?
5. Cek visual 3D: pedestrian muncul di entryLine?

**Jika error "xxx not found":**
- Cek apakah semua markup sudah dibuat
- Cek apakah nama blok sesuai

**Jika error "Agent cannot be cast":**
- Cek semua `Delay time` di PedService — parameter harus `ped` bukan `agent`

### 11.3 Test 2: Run 30 Menit (Semua Route)

1. **Stop time** = `30` menit
2. Run
3. Cek console: semua jenis layanan muncul?
   - `TUJUAN → PINJAM`, `TUJUAN → KEMBALI`, `TUJUAN → CARIDUDUK`, `TUJUAN → HELPDESK`, `TUJUAN → LANGSUNG`
4. Jika ada route yang tidak muncul — cek koneksi blok

### 11.4 Test 3: Run 60 Menit (Full Simulation)

1. **Stop time** = `60` menit
2. Run
3. Pantau:
   - Antrean di counter pinjam dan kembali
   - Aktivitas belajar (silent zone vs diskusi)
   - Helpdesk: berapa yang dilayani
   - Tidak ada error stack trace

### 11.5 Perkiraan Hasil Wajar (60 Menit, exponential(2.0-5.0))

| Metrik | Perkiraan | Keterangan |
|---|---|---|
| Total pengunjung | ~40-60 orang | Tergantung peak |
| Peminjaman | ~10-15 orang | 25% dari total |
| Pengembalian | ~8-12 orang | 20% dari total |
| Cari tempat duduk | ~12-18 orang | 30% dari total |
| Helpdesk | ~4-6 orang | 10% dari total |
| Langsung keluar | ~6-9 orang | 15% dari total |

---

## 12. Troubleshooting

### Error: "Duplicate name 'xxx'"

Ada dua blok dengan nama sama. Cari di panel Projects, rename salah satu.

### Error: "Agent cannot be cast to PengunjungPed"

Ada blok PedService yang masih memakai parameter `agent` bukan `ped`.

**Cek semua fungsi** yang dipanggil di `Delay time` — pastikan parameter bernama `ped`:
- ✅ `hitungWaktuServicePeminjaman(ped)`
- ❌ `hitungWaktuServicePeminjaman(agent)`

### Error: "Function xxx not found"

Fungsi yang dipanggil di Delay time belum dibuat di Main. Cek daftar fungsi di [Langkah 4](#47-tambah-fungsi-dari-modul-parkir).

### Error: "Service xxx not found"

Blok PedService menggunakan markup yang belum ada. Pastikan setiap PedService → `Services` → pilih markup Service with Lines yang sudah dibuat.

### Antrean kosong / pedestrian tidak muncul

1. Cek probabilitas `selectTujuan` — total harus **1.0**
2. Cek koneksi — semua output terhubung
3. Cek `Target line` di PedSource — harus指向 markup yang ada

### selectPulang tidak memisah orang parkir vs jalan

**Penyebab:** `selectPulang` masih pakai probabilitas, bukan kondisi.

**Solusi:**
- **Use conditions** = centang
- **Use probabilities** = jangan centang
- Out 1: `ped.isParkir == true`
- Out 2: `ped.isParkir == false`

### Queue PM error "antreHelpdesk.size()"

Jika `antreHelpdesk` masih menggunakan Queue (Process Modeling), ganti `.size()` dengan `.queueSize()` atau `.length()`. Atau lebih baik: ganti Queue dengan PedQueue.

### Pedestrian berjalan menembus tembok

Tambahkan **Wall** atau **Fence** di Space Markup untuk membatasi area pedestrian. Atau atur posisi node lebih berhati-hati.

### Tidak ada log "BELAJAR" atau "HELPDESK"

Route tersebut mungkin jarang terpilih karena probabilitas kecil. Untuk testing, naikkan probabilitas sementara:
- `selectTujuan.out3` (CARI DUDUK) = 0.50
- `selectTujuan.out4` (HELPDESK) = 0.20
- Kurangi out1 dan out5

### Simulasi berjalan lambat

1. Komentari `traceln(...)` yang tidak perlu
2. Kurangi jumlah service point (misal `svcAreaSepi` dari 10 jadi 6)
3. Kurangi arrival rate

---

## Lampiran

### A. Daftar Lengkap Blok di Flowchart Final

| Blok | Tipe | Modul Asal | Keterangan |
|---|---|---|---|
| `srcJalanKaki` | PedSource | Entry (baru) | Pejalan kaki langsung |
| `srvScanKTM` | PedService | Entry (baru) | Scan KTM |
| `selectTujuan` | PedSelectOutput | Entry (baru) | Hub routing 5 output |
| `goToRakBuku` | PedGoTo | Base | Jalan ke rak |
| `wKelilingRak` | PedWait | Base | Browsing rak |
| `goToCounterPinjam` | PedGoTo | Base | Jalan ke counter pinjam |
| `selectJadiPinjam` | PedSelectOutput | Base | 90% jadi / 10% batal |
| `srvPinjam` | PedService | Base | Service peminjaman |
| `goToCounterKembali` | PedGoTo | Base | Jalan ke counter kembali |
| `srvKembali` | PedService | Base | Service pengembalian |
| `wJalanKeluar` | PedWait | Entry (baru) | Jalan ke garis keluar |
| `selectPulang` | PedSelectOutput | Entry (baru) | Routing pulang |
| `wJalanBalikParkir` | PedWait | Entry (baru) | Balik ke parkir |
| `snkSelesai` | PedSink | Entry (baru) | Keluar sistem |
| `selectAktivitas` | PedSelectOutput | Tempat Duduk | 4 aktivitas sebelum duduk |
| `srvToiletSinggah` | PedService | Tempat Duduk | Toilet singgah |
| `srvCariBuku` | PedService | Tempat Duduk | Cari buku |
| `srvLoker` | PedService | Tempat Duduk | Simpan barang |
| `selectZonaDuduk` | PedSelectOutput | Tempat Duduk | 60/40 zona duduk |
| `srvDudukSepi` | PedService | Tempat Duduk | Silent zone |
| `srvDudukDiskusi` | PedService | Tempat Duduk | Diskusi zone |
| `goToHelpdesk` | PedGoTo | Helpdesk | Jalan ke helpdesk |
| `antreHelpdesk` | PedQueue | Helpdesk | Antrean helpdesk |
| `srvHelpdesk` | PedService | Helpdesk | Layanan helpdesk |
| `selectStatusBuku` | PedSelectOutput | Helpdesk | 40/35/25 status buku |
| `hdGoToRakBuku` | PedGoTo | Helpdesk | Jalan ke rak buku |
| `hdGoToPinjam` | PedGoTo | Helpdesk | Jalan ke pinjam |
| `srcParkirMobil` | CarSource | Parkir | Mobil masuk |
| `srcParkirMotor` | CarSource | Parkir | Motor masuk |
| `srvParkirMobil` | CarMoveTo | Parkir | Mobil cari parkir |
| `srvParkirMotor` | CarMoveTo | Parkir | Motor cari parkir |
| `orangDatang` | PedSource | Parkir | Trigger dari parkir |
| `pedJalanKePerpus` | PedGoTo | Parkir | Jalan ke perpus |
| `jalanKeParkiran` | PedGoTo | Parkir | Jalan ke parkiran |

### B. Daftar Lengkap Variabel di Main

Base: `seqPed`, `totalPeminjaman`, `totalPengembalian`, `totalDenda`, `totalBukuRusak`,
`maxQueuePeminjaman`, `maxQueuePengembalian`, `totalBukuDipinjam`, `totalErrorScanner`,
`totalBrowsing`, `totalBatalPinjam`, `totalDurasiBrowsing`, `totalSelesai`, `totalWaktuSistem`

Parkir: `totalMobil`, `totalMotor`

Tempat Duduk: `totalToiletSinggah`, `totalCariBuku`, `totalLoker`,
`totalDudukSepi`, `totalDudukDiskusi`, `maxAntrianDudukSepi`, `maxAntrianDudukDiskusi`,
`totalDurasiBelajar`

Helpdesk: `totalMahasiswa`, `totalKeHelpdesk`, `totalBukuTersedia`,
`totalBukuDipinjamHelpdesk`, `totalBukuTidakAda`, `maxAntreanHelpdesk`,
`totalWaktuTunggu`, `totalWaktuLayanan`

Entry: `totalScanKTM`, `totalDosen`

**Total: 34 variabel**

### C. Daftar Lengkap Fungsi di Main

Base (9): `hitungWaktuServicePeminjaman`, `hitungWaktuServicePengembalian`,
`hitungWaktuBrowsing`, `avgWaktuSistem`, `avgBukuPerTransaksi`,
`errorScannerRate`, `rataRataDenda`, `avgWaktuBrowsing`, `tingkatBatalPinjam`

Parkir (2): `hitungWaktuCariParkirMobil`, `hitungWaktuCariParkirMotor`

Tempat Duduk (4): `hitungWaktuToiletSinggah`, `hitungWaktuCariBuku`,
`hitungWaktuLoker`, `hitungDurasiBelajar`

Entry (2): `getInterarrivalTime`, `hitungWaktuScanKTM`

Helpdesk (3): `avgWaktuTunggu`, `avgWaktuLayanan`, `tingkatKeberhasilanHelpdesk`

**Total: 20 fungsi**

### D. Daftar Lengkap Markup

`entryLine`, `exitLine`, `nodeRakBuku`, `nodeCounterPinjam`, `nodeCounterKembali`,
`svcPeminjaman` (2 services), `svcPengembalian` (1 service),
`svcScanKTM` (2 services, 2 queues),
`svcToiletSinggah` (2 services, 1 queue), `svcCariBuku` (2 services, 2 queues),
`svcLoker` (3 services, 1 queue), `svcAreaSepi` (10 services, 2 queues),
`svcAreaDiskusi` (4 services, 2 queues),
`nodeHelpdesk`, `areaAntreHelpdesk`, `svcHelpdesk` (2 services, 1 queue),
`nodePinjam`, `areaPejalanKaki`

**Total: ~18 markup elements**
