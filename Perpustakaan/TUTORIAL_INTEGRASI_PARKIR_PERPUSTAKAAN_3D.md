# Tutorial Integrasi: Simulasi Parkir + Perpustakaan 3D (AnyLogic)

## Menggabungkan Dua Model Menjadi Satu Simulasi Lengkap

**Versi:** 1.0
**Status:** Berdasarkan analisis kode merge pada proyek `Perpustakaan3D_AntriCounter1` dan `SimulasiParkirPerpustakaan`.

---

## Daftar Isi

1. [Tentang Tutorial Ini](#1-tentang-tutorial-ini)
2. [Apa yang Akan Dibangun](#2-apa-yang-akan-dibangun)
3. [Arsitektur Model Terintegrasi](#3-arsitektur-model-terintegrasi)
4. [Prasyarat](#4-prasyarat)
5. [Cara 1: Manual di AnyLogic (Langkah demi Langkah)](#5-cara-1-manual-di-anylogic-langkah-demi-langkah)
   - [5.1 Migrasi Project](#51-migrasi-project)
   - [5.2 Tambah Agent Type MobilPengunjung](#52-tambah-agent-type-mobilpengunjung)
   - [5.3 Tambah Variabel Main](#53-tambah-variabel-di-main)
   - [5.4 Tambah Variabel di MahasiswaPed](#54-tambah-variabel-di-mahasiswaped)
   - [5.5 Bangun Road Network & Markup Parkir](#55-bangun-road-network--markup-parkir)
   - [5.6 Bangun Flowchart Kendaraan](#56-bangun-flowchart-kendaraan)
   - [5.7 Ubah Flowchart Pedestrian](#57-ubah-flowchart-pedestrian)
   - [5.8 Tambah Schedule & Event](#58-tambah-schedule--event)
   - [5.9 Konfigurasi Detail Setiap Blok](#59-konfigurasi-detail-setiap-blok)
   - [5.10 Integrasi Antar Flowchart](#510-integrasi-antar-flowchart)
   - [5.11 Tambah Objek 3D Parkir](#511-tambah-objek-3d-parkir)
   - [5.12 Dashboard & Statistik](#512-dashboard--statistik)
6. [Cara 2: Merge Otomatis dengan Python Script](#6-cara-2-merge-otomatis-dengan-python-script)
   - [6.1 Persiapan](#61-persiapan)
   - [6.2 Jalankan Merge Step by Step](#62-jalankan-merge-step-by-step)
   - [6.3 Verifikasi Hasil](#63-verifikasi-hasil)
7. [Konfigurasi Lengkap Setiap Blok](#7-konfigurasi-lengkap-setiap-blok)
8. [Skenario Simulasi](#8-skenario-simulasi)
9. [Troubleshooting](#9-troubleshooting)
10. [Checklist Final](#10-checklist-final)

---

## 1. Tentang Tutorial Ini

Tutorial ini menjelaskan cara **mengintegrasikan dua model AnyLogic** yang sudah ada:

| Model | Direktori | Fokus |
|---|---|---|
| **Perpustakaan3D_AntriCounter1** | `Perpustakaan3D_AntriCounter1/` | Simulasi antrean perpustakaan 3D dengan pedestrian, counter peminjaman, interior ruangan |
| **SimulasiParkirPerpustakaan** | `SimulasiParkirPerpustakaan/` | Simulasi parkir kendaraan dengan CarSource, CarMoveTo, CarDispose, jadwal kedatangan |

Hasil integrasi adalah model bernama **`Perpustakaan3D_Lengkap`** yang menggabungkan keduanya: pengunjung datang dengan mobil, parkir, turun, berjalan ke perpustakaan, antre di counter, lalu kembali ke mobil dan pergi.

---

## 2. Apa yang Akan Dibangun

### Alur Lengkap Model Terintegrasi

```
                        ┌───────────────────────────┐
                        │      P A R K I R          │
                        │                           │
  carSource ──► moveMenujuAntrian ──► prosesParkirMobil
                        │                               │
                        │                          selectOutput
                        │                               │
                        │                         ┌─────┴─────┐
                        │                         │           │
                        │                    outT (true)   outF (false)
                        │                         │           │
                        │                    tungguPemilik    │
                        │                         │           │
                        │                    moveKeluar ◄────┘
                        │                         │
                        │                    buangMobil
                        │
                        ▼
               ┌────────────────────┐
               │    P E R P U S     │
               │                    │
   orangTurun ──► jalanKePerpus ──► srvCounter ──► jalanKeParkiran ──► snkSelesai
     PedSource      PedGoTo         PedService        PedGoTo          PedSink
```

### Skenario Lengkap

1. **Kendaraan datang** ke parkir sesuai jadwal (`jadwalKedatangan`)
2. Mobil masuk area parkir dan parkir di slot (`prosesParkirMobil`)
3. Saat mobil parkir, **penumpang turun** (`orangTurun.inject(1)`)
4. Penumpang berjalan ke **pintu perpustakaan** (`jalanKePerpus` menuju `pintuPerpus`)
5. Penumpang antre di **counter peminjaman** (`srvCounter`)
6. Setelah selesai, penumpang berjalan **kembali ke parkiran** (`jalanKeParkiran`)
7. Penumpang menunggu mobilnya (`tungguPemilik.free(myCar)`)
8. Mobil keluar dari parkir dan meninggalkan sistem

---

## 3. Arsitektur Model Terintegrasi

### Agent Types

| Agent Type | Sumber | Keterangan |
|---|---|---|
| `Main` | Kedua model | Agent utama, akan digabung |
| `MahasiswaPed` | Perpustakaan3D | Pedestrian type (pengunjung perpustakaan) |
| `MobilPengunjung` | SimulasiParkir | Agent type baru: mobil pengunjung |

### Variabel Kunci untuk Integrasi

| Variabel | Lokasi | Type | Fungsi |
|---|---|---|---|
| `currentCar` | `Main` | `MobilPengunjung` | Menyimpan referensi mobil yg baru parkir |
| `myCar` | `MahasiswaPed` | `MobilPengunjung` | Menyimpan referensi mobil yg ditumpangi pedestrian |
| `jadwalKedatangan` | `Main` | `Schedule` | Jadwal arrival rate mobil (jam sibuk / normal) |
| `timerPalangTurun` | `Main` | `Event` | Timer untuk simulasi palang parkir |

---

## 4. Prasyarat

1. **AnyLogic** terinstal (minimal versi 8.7+ dengan Pedestrian Library dan Road Traffic Library)
2. Kedua project sudah ada dan bisa di-run sendiri-sendiri
3. **Python 3.10+** (hanya untuk Cara 2 - merge otomatis)
4. Paham dasar AnyLogic: agent, flowchart, markup, variable

---

## 5. Cara 1: Manual di AnyLogic (Langkah demi Langkah)

Cara ini dilakukan langsung di AnyLogic IDE. Cocok jika ingin memahami setiap perubahan.

### 5.1 Migrasi Project

1. Buka project **Perpustakaan3D_AntriCounter1** di AnyLogic.
2. Klik **File -> Save As...** dengan nama `Perpustakaan3D_Lengkap`.
3. Simpan di folder baru `Perpustakaan3D_Lengkap/`.
4. Tutup project lama, buka project `Perpustakaan3D_Lengkap`.

### 5.2 Tambah Agent Type MobilPengunjung

Kita perlu agent type baru untuk mobil.

1. Klik kanan pada model item di Project tree -> **New -> Agent**.
2. Isi nama: `MobilPengunjung`.
3. **Agent class name**: `MobilPengunjung`.
4. Klik **Finish**.

> **Catatan:** Agent `MobilPengunjung` bisa kosong saja (tidak perlu variabel tambahan). Namun di model parkir aslinya, agent ini bisa dikustomisasi jika diperlukan (misal: tipe mobil, plat nomor).

### 5.3 Tambah Variabel di Main

Buka diagram `Main`, tambahkan variabel berikut:

| Nama | Type | Initial Value | Keterangan |
|---|---|---|---|
| `currentCar` | `MobilPengunjung` | `null` | Menampung mobil yg baru parkir, untuk dioper ke pedestrian |

### 5.4 Tambah Variabel di MahasiswaPed

Buka diagram `MahasiswaPed`, tambahkan:

| Nama | Type | Initial Value | Keterangan |
|---|---|---|---|
| `myCar` | `MobilPengunjung` | `null` | Referensi ke mobil yg ditumpangi |
| `tMasuk` | `double` | `0` | Waktu masuk sistem (mungkin sudah ada) |

> Jika `tMasuk` sudah ada di `MahasiswaPed`, cukup tambahkan `myCar`.

### 5.5 Bangun Road Network & Markup Parkir

#### 5.5.1 Road Network

Dari palette **Road Traffic Library**:

1. Drag elemen **Road Network** ke canvas `Main`.
2. Biarkan nama default atau rename `roadNetwork`.

#### 5.5.2 Road dan Area Parkir

1. Dari **Road Traffic Library**, drag **Road** ke canvas 3D.
2. Buat jalan masuk dari luar area menuju area parkir.
3. Tambahkan **Parking Lot** atau cukup gunakan **Road** biasa sebagai area parkir.
4. Dari **Pedestrian Library** > **Space Markup**:
   - Tambah **Target line** untuk `areaPejalanKaki` (area pejalan kaki di sekitar parkir).
   - Tambah **Node** untuk `pintuPerpus` (titik masuk ke perpustakaan).

#### 5.5.3 Markup Tambahan

Dari **Pedestrian Library** > **Space Markup**:

| Nama Markup | Tipe | Fungsi |
|---|---|---|
| `areaPejalanKaki` | Target line | Area tempat pedestrian berjalan setelah turun dari mobil |
| `pintuPerpus` | Node | Titik masuk ke perpustakaan, jadi target `jalanKePerpus` |

### 5.6 Bangun Flowchart Kendaraan

Di `Main`, dari palette **Road Traffic Library**, drag blok berikut:

| Blok | Rename | Fungsi |
|---|---|---|
| CarSource | `carSource` | Sumber kendaraan |
| CarMoveTo | `moveMenujuAntrian` | Mobil bergerak menuju area parkir |
| CarMoveTo | `prosesParkirMobil` | Mobil parkir di slot |
| SelectOutput | `selectOutput` | Seleksi: mobil nunggu pemilik (true) atau langsung keluar (false) |
| Delay | `tungguPemilik` | Mobil nunggu sampai pedestrian selesai di perpus |
| CarMoveTo | `moveKeluar` | Mobil keluar dari parkir |
| CarDispose | `buangMobil` | Mobil keluar sistem |

**Hubungkan port:**

```
carSource.out -> moveMenujuAntrian.in
moveMenujuAntrian.out -> prosesParkirMobil.in
prosesParkirMobil.out -> selectOutput.in

selectOutput.outT (true) -> tungguPemilik.in
selectOutput.outF (false) -> moveKeluar.in

tungguPemilik.out -> moveKeluar.in
moveKeluar.out -> buangMobil.in
```

### 5.7 Ubah Flowchart Pedestrian

Diagram flowchart pedestrian **lama** (sebelum integrasi):

```
srcMasuk -> srvCounter -> wJalanKeluar -> snkSelesai
```

Diagram flowchart pedestrian **baru** (setelah integrasi):

```
orangTurun -> jalanKePerpus -> srvCounter -> jalanKeParkiran -> snkSelesai
```

Perubahan:

| Blok Lama | Blok Baru | Keterangan |
|---|---|---|
| `srcMasuk` (PedSource) | `orangTurun` (PedSource) | Pedestrian tidak muncul sendiri, tapi di-inject dari mobil |
| (tidak ada) | `jalanKePerpus` (PedGoTo) | Berjalan dari parkir ke pintu perpus |
| `srvCounter` | `srvCounter` (sama) | Service counter tetap sama |
| `wJalanKeluar` (PedWait) | `jalanKeParkiran` (PedGoTo) | Tidak langsung keluar, tapi kembali ke parkir |
| `snkSelesai` | `snkSelesai` (sama) | Sink tetap sama |

#### Langkah modifikasi:

1. **Ganti** `srcMasuk` dengan blok baru `orangTurun` (PedSource):
   - Hapus `srcMasuk`.
   - Drag **PedSource** baru, rename `orangTurun`.
   - Atur `Appears at` = `line`, pilih `areaPejalanKaki`.
   - Atur `Arrive according to` = `Manual` (karena akan di-inject dari mobil).

2. **Tambah** `jalanKePerpus` (PedGoTo):
   - Drag **PedGoTo**, rename `jalanKePerpus`.
   - Set `Target node` = `pintuPerpus`.

3. **Tambah** `jalanKeParkiran` (PedGoTo):
   - Drag **PedGoTo**, rename `jalanKeParkiran`.
   - Set `Target line` = `areaPejalanKaki`.

4. **Sambungkan ulang**:
   ```
   orangTurun.out -> jalanKePerpus.in
   jalanKePerpus.out -> srvCounter.in
   srvCounter.out -> jalanKeParkiran.in
   jalanKeParkiran.out -> snkSelesai.in
   ```

### 5.8 Tambah Schedule & Event

#### 5.8.1 Schedule: `jadwalKedatangan`

1. Dari palette **Agent**, drag **Schedule** ke canvas `Main`.
2. Rename: `jadwalKedatangan`.
3. Atur **Value type** = `double`.
4. Atur jadwal (contoh):
   | Time | Value |
   |---|---|
   | 0 | `5.0` | Awal: 5 mobil per menit |
   | 20 | `12.0` | Peak hour: 12 mobil per menit |
   | 50 | `7.0` | Normal: 7 mobil per menit |

#### 5.8.2 Event: `timerPalangTurun`

1. Dari palette **Agent**, drag **Event** ke canvas `Main`.
2. Rename: `timerPalangTurun`.
3. Atur **Trigger type** = `Cyclical`.
4. **Occurrence time**: bisa diisi `uniform(0.5, 2.0)` untuk simulasi palang parkir acak.

### 5.9 Konfigurasi Detail Setiap Blok

#### 5.9.1 `carSource` (CarSource)

| Property | Value |
|---|---|
| Arrive according to | `Rate schedule` |
| Rate schedule | `jadwalKedatangan` |
| New car | `MobilPengunjung` |
| Road | `road` (nama road network) |

#### 5.9.2 `moveMenujuAntrian` (CarMoveTo)

| Property | Value |
|---|---|
| Road | `road` |
| On exit | (biarkan kosong) |

#### 5.9.3 `prosesParkirMobil` (CarMoveTo)

| Property | Value |
|---|---|
| Road | `road` |

**Action `On exit`: (SANGAT PENTING)**

```java
currentCar = (MobilPengunjung) agent;
orangTurun.inject(1);
```

Kode ini akan:
- Menyimpan referensi mobil ke `currentCar`.
- Memicu `orangTurun` untuk menghasilkan 1 pedestrian.

#### 5.9.4 `selectOutput` (SelectOutput)

| Property | Value |
|---|---|
| N outputs | `2` |
| Condition (true) | `uniform(0, 1) < 0.95` |
| Use probabilities | Tidak dicentang (gunakan kondisi) |

> 95% mobil menunggu pemiliknya (true), 5% langsung keluar (false). Ini bisa disesuaikan.

#### 5.9.5 `tungguPemilik` (Delay)

| Property | Value |
|---|---|
| Delay type | `Hold` (tahan sampai di-free manual) |
| Maximum capacity | `100` |

Blok ini akan menahan mobil sampai dipanggil oleh `free()` dari kode pedestrian.

#### 5.9.6 `moveKeluar` (CarMoveTo)

| Property | Value |
|---|---|
| Road | `road` |
| On exit | (biarkan kosong) |

#### 5.9.7 `buangMobil` (CarDispose)

Tidak ada konfigurasi khusus. Cukup dihubungkan dari `moveKeluar`.

#### 5.9.8 `orangTurun` (PedSource)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `areaPejalanKaki` |
| Arrive according to | `Manual` (karena di-inject dari prosesParkirMobil) |
| New pedestrian | `MahasiswaPed` |

**Action `On exit`:**

```java
seqPed++;
ped.idPed = "M-" + seqPed;
ped.tMasuk = time();
ped.jumlahBuku = uniform_discr(1, 5);

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

// Simpan referensi mobil ke pedestrian
ped.myCar = currentCar;

traceln("ARRIVE " + ped.idPed
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari");
```

#### 5.9.9 `jalanKePerpus` (PedGoTo)

| Property | Value |
|---|---|
| Target node | `pintuPerpus` |
| Mode | `self.LOCATION_NODE` |

#### 5.9.10 `srvCounter` (PedService) - Sama seperti model lama

| Property | Value |
|---|---|
| Services | (markup service counter yang sudah ada) |
| Delay time | `exponential(1.0, 2.0, MINUTE)` atau fungsi kustom |

#### 5.9.11 `jalanKeParkiran` (PedGoTo)

| Property | Value |
|---|---|
| Target line | `areaPejalanKaki` |
| Mode | `self.LOCATION_LINE` |

#### 5.9.12 `snkSelesai` (PedSink)

**Action `On enter`: (DITAMBAH kode free car)**

```java
// --- kode statistik yang sudah ada ---
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

// --- TAMBAHKAN: free mobil yang menunggu ---
if (ped.myCar != null) {
    tungguPemilik.free(ped.myCar);
}

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " menit");
```

Kode `tungguPemilik.free(ped.myCar)` akan:
- Mencari mobil yang sesuai di blok `tungguPemilik`.
- Melepaskan (free) mobil tersebut agar lanjut ke `moveKeluar`.
- Jika `ped.myCar` null (tidak naik mobil), tidak terjadi apa-apa.

### 5.10 Integrasi Antar Flowchart

Inilah inti integrasi: **dua flowchart terpisah (mobil + pedestrian) terhubung lewat variabel dan kode**.

```
 FLOWCHART MOBIL                    FLOWCHART PEDESTRIAN
 ──────────────────────             ──────────────────────────
 carSource                          (tidak langsung terhubung)
    │
    ▼
 moveMenujuAntrian
    │
    ▼
 prosesParkirMobil ──onExit──> orangTurun.inject(1) ──> jalanKePerpus
    │                    │                                   │
    ▼                    │                                   ▼
 selectOutput            │                               srvCounter
    │                    │                                   │
    ├──► tungguPemilik   │                               jalanKeParkiran
    │        ▲           │                                   │
    └──► moveKeluar      │                               snkSelesai
            │            │                                   │
            ▼            │                        onEnter: tungguPemilik.free(myCar)
         buangMobil      │                                   │
                         └───────────────────────────────────┘
                              (via currentCar + ped.myCar)
```

**Alur data:**

1. `prosesParkirMobil.onExit()`:
   - `currentCar = (MobilPengunjung) agent` → simpan mobil yg baru parkir.
   - `orangTurun.inject(1)` → lahirkan pedestrian.

2. `orangTurun.onExit()`:
   - `ped.myCar = currentCar` → pedestrian "tahu" mobilnya.

3. `snkSelesai.onEnter()`:
   - `tungguPemilik.free(ped.myCar)` → mobil yg menunggu dilepas.

### 5.11 Tambah Objek 3D Parkir

Dari **3D Objects** palette, tambahkan objek-objek parkir:

| Objek | Nama | Fungsi |
|---|---|---|
| Box | `road3D` | Lantai jalan dan parkir (warna abu-abu) |
| Box/Car | (dari 3D assets `car.dae`) | Dekorasi mobil |
| Tree | (dari `tree_1.dae` dll) | Pohon di area parkir |
| Building | (dari `building_1.dae`) | Gedung perpustakaan dari sisi luar |

> Semua asset 3D sudah tersedia di folder `3d/` kedua proyek.

**Tata letak 3D yang disarankan:**

```
 ┌─────────────────────────────────────────────┐
 │              GEDUNG PERPUSTAKAAN             │
 │  ┌─────────────────────────────────┐        │
 │  │   Interior (3D Window)         │        │
 │  │   [counter] [rak buku]         │        │
 │  │                         pintuPerpus      │
 │  └─────────────────────────────────┘        │
 │            ☻ areaPejalanKaki ☻              │
 │  ┌─────────────────────────────────┐        │
 │  │         AREA PARKIR             │        │
 │  │  [mobil] [mobil] [mobil]       │        │
 │  │  ========================= road         │
 │  │              ← masuk / keluar           │
 │  └─────────────────────────────────┘        │
 └─────────────────────────────────────────────┘
```

### 5.12 Dashboard & Statistik

Tambahkan **Text** dinamis untuk memantau statistik integrasi:

**Statistik kendaraan:**
```
"Total mobil masuk: " + carSource.countEntered()
```

```
"Mobil parkir: " + prosesParkirMobil.queueSize()
```

```
"Mobil menunggu: " + tungguPemilik.size()
```

```
"Mobil keluar: " + buangMobil.countEntered()
```

**Statistik pedestrian (yang sudah ada):**
```
"Total pengunjung: " + totalSelesai + " orang"
```

```
"Antrian counter: " + srvCounter.queueSize() + " orang"
```

---

## 6. Cara 2: Merge Otomatis dengan Python Script

Cara ini menggunakan script Python yang sudah ada di folder `Perpustakaan3D_AntriCounter1/` untuk memodifikasi file `.alp` secara otomatis.

### 6.1 Persiapan

1. Pastikan **Python 3.10+** terinstal.
2. Siapkan file `.alp` yang diperlukan:
   - `Perpustakaan3D_AntriCounter1/Perpustakaan3D_AntriCounter1.alp` (model dasar perpustakaan 3D)
   - `SimulasiParkirPerpustakaan/SimulasiParkirPerpustakaan.alp` (model parkir)
3. Copy file `SimulasiParkirPerpustakaan.alp` ke folder `Perpustakaan3D_AntriCounter1/`.
4. Backup file `Perpustakaan3D_AntriCounter1.alp` sebelum di-merge.

### 6.2 Struktur Script Merge

Script-script merger sudah tersedia dengan urutan sebagai berikut:

| Script | Fungsi |
|---|---|
| `merge_step1.py` | Tambah variabel `currentCar` (MobilPengunjung) ke Main |
| `merge_step2.py` | Tambah `myCar` ke MahasiswaPed, tambah MobilPengunjung agent, tambah car flow blocks, schedule & event, modifikasi snkSelesai |
| `merge_step3.py` | Gabung elemen presentation (3D objects, tata letak) dari model parkir |
| `merge_step4.py` | Tambah Events/Sections dan MobilPengunjung yang mungkin terlewat |
| `merge_step5.py` | Finalisasi: tambah missing car flow blocks dan code hooks |
| `merge_all.py` | **Script lengkap** yang melakukan semua langkah dalam satu kali jalan |

### 6.3 Jalankan Merge Step by Step

**Cara termudah: jalankan `merge_all.py` langsung:**

```bash
cd Perpustakaan3D_AntriCounter1
python merge_all.py
```

Script ini akan:
1. Mengganti `srcMasuk` (PedSource) dengan `orangTurun` (PedSource yang di-inject manual).
2. Menambah `jalanKePerpus` dan `jalanKeParkiran` (PedGoTo) setelah `orangTurun`.
3. Menambah car flow `EmbeddedObjects` (carSource, moveMenujuAntrian, dll).
4. Mengganti semua konektor agar sesuai dengan flowchart baru.
5. Menyimpan hasil sebagai `Perpustakaan3D_Lengkap.alp`.

**Jika ingin menjalankan step by step:**

```bash
python merge_step1.py    # Tambah currentCar variable
python merge_step2.py    # Tambah myCar, MobilPengunjung, car flow, schedule, event
python merge_step3.py    # Gabung presentation elements
python merge_step4.py    # Tambah missing Events/Schedules
python merge_step5.py    # Finalisasi car flow + code hooks
```

### 6.4 Verifikasi Hasil

Setelah merge selesai, buka `Perpustakaan3D_Lengkap.alp` di AnyLogic dan verifikasi:

1. **Agent types** yang ada: `Main`, `MahasiswaPed`, `MobilPengunjung`
2. **Main** memiliki variabel: `seqPed`, `totalSelesai`, `totalWaktuSistem`, `currentCar`, dll.
3. **MahasiswaPed** memiliki variabel: `idPed`, `jumlahBuku`, `myCar`, `tMasuk`, dll.
4. **Flowchart** sudah sesuai dengan diagram di [Bagian 2](#2-apa-yang-akan-dibangun).
5. **Markup** sudah lengkap: `areaPejalanKaki`, `pintuPerpus`, road network.
6. **Schedule & Event**: `jadwalKedatangan`, `timerPalangTurun`.

---

## 7. Konfigurasi Lengkap Setiap Blok

### 7.1 Tabel Semua Blok

#### Flowchart Kendaraan

| Blok | Type | Property Penting | Value |
|---|---|---|---|
| `carSource` | CarSource | Arrival type | `Rate schedule` |
| | | Rate schedule | `jadwalKedatangan` |
| | | New car | `MobilPengunjung` |
| | | Road | `road` |
| `moveMenujuAntrian` | CarMoveTo | Road | `road` |
| `prosesParkirMobil` | CarMoveTo | Road | `road` |
| | | On exit | `currentCar = (MobilPengunjung) agent; orangTurun.inject(1);` |
| `selectOutput` | SelectOutput | Condition | `uniform(0, 1) < 0.95` |
| `tungguPemilik` | Delay | Delay type | `Hold` |
| | | Max capacity | `100` |
| `moveKeluar` | CarMoveTo | Road | `road` |
| `buangMobil` | CarDispose | - | - |

#### Flowchart Pedestrian

| Blok | Type | Property Penting | Value |
|---|---|---|---|
| `orangTurun` | PedSource | Target line | `areaPejalanKaki` |
| | | Arrival type | `Manual` |
| | | New pedestrian | `MahasiswaPed` |
| | | On exit | (kode generate data + `ped.myCar = currentCar`) |
| `jalanKePerpus` | PedGoTo | Target node | `pintuPerpus` |
| `srvCounter` | PedService | Services | (markup service counter) |
| | | Delay time | `exponential(1.0, 2.0, MINUTE)` |
| `jalanKeParkiran` | PedGoTo | Target line | `areaPejalanKaki` |
| `snkSelesai` | PedSink | On enter | (statistik + `tungguPemilik.free(ped.myCar)`) |

### 7.2 Kode Lengkap On Exit `orangTurun`

```java
seqPed++;
ped.idPed = "M-" + seqPed;
ped.tMasuk = time();

ped.jumlahBuku = uniform_discr(1, 5);

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

ped.myCar = currentCar;

traceln("ARRIVE " + ped.idPed
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari");
```

### 7.3 Kode Lengkap On Enter `snkSelesai`

```java
totalSelesai++;

double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

if (ped.myCar != null) {
    tungguPemilik.free(ped.myCar);
}

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " menit");
```

---

## 8. Skenario Simulasi

### 8.1 Skenario Normal (60 menit)

**Konfigurasi:**
- `jadwalKedatangan`: 5 → 12 → 7 mobil/menit
- Stop time: 60 menit

**Yang diamati:**
- Mobil datang, parkir, penumpang turun.
- Penumpang antre di counter perpustakaan.
- Mobil menunggu di `tungguPemilik`.
- Penumpang selesai, mobil di-free, mobil keluar.
- Antrean memanjang saat peak (menit 20-50).

### 8.2 Skenario Macet Total (Ekstrem)

**Konfigurasi:**
- Ubah `selectOutput` kondisi menjadi `uniform(0, 1) < 0.01` (hanya 1% mobil langsung keluar).
- Stop time: 120 menit.

**Yang diamati:**
- `tungguPemilik` penuh dengan mobil menunggu.
- Parkir penuh, mobil antre di `prosesParkirMobil`.
- Pedestrian tetap bisa turun dan antre di counter.

### 8.3 Skenario Parkir Cepat

**Konfigurasi:**
- Ubah `selectOutput` kondisi menjadi `uniform(0, 1) < 0.5` (50% mobil langsung keluar).
- Stop time: 30 menit.

**Yang diamati:**
- Banyak mobil keluar tanpa menunggu pemilik.
- Jumlah mobil parkir lebih sedikit.
- Arus lebih cepat.

---

## 9. Troubleshooting

### Masalah 1: Pedestrian tidak muncul setelah mobil parkir

**Penyebab:** Kode `orangTurun.inject(1)` tidak terpanggil.

**Solusi:**
1. Cek `prosesParkirMobil.onExit()` — pastikan kode ada.
2. Cek bahwa `currentCar` dideklarasikan di `Main` bertipe `MobilPengunjung`.
3. Coba tambahkan traceln untuk debug:
   ```java
   traceln("MOBIL PARKIR: inject pedestrian");
   currentCar = (MobilPengunjung) agent;
   orangTurun.inject(1);
   ```

### Masalah 2: Mobil tidak pernah di-free (tertahan di tungguPemilik)

**Penyebab:** `snkSelesai.onEnter()` tidak memanggil `free()`.

**Solusi:**
1. Cek kode `snkSelesai.onEnter()` — pastikan ada `tungguPemilik.free(ped.myCar)`.
2. Cek bahwa `ped.myCar` tidak null. Di `orangTurun.onExit()`, pastikan `ped.myCar = currentCar`.

### Masalah 3: Error "Cannot cast from Agent to MobilPengunjung"

**Penyebab:** `currentCar` belum dideklarasikan atau typenya salah.

**Solusi:**
1. Buka `Main`, cek variabel `currentCar` — harus bertipe `MobilPengunjung`.
2. Jika belum ada, buat variabel baru: type `MobilPengunjung`, initial value `null`.

### Masalah 4: Road Network tidak terlihat / error

**Penyebab:** Road belum ditambahkan ke Road Network.

**Solusi:**
1. Pastikan sudah ada `Road` dari palette Road Traffic Library.
2. Pastikan `roadNetwork` sudah ada di `Main`.
3. Drag road ke dalam roadNetwork.

### Masalah 5: Merge script error "assertion failed"

**Penyebab:** File `.alp` tidak memiliki struktur yang diharapkan (mungkin versi berbeda).

**Solusi:**
1. Pastikan menggunakan file `.alp` yang benar sebagai input.
2. Periksa apakah ID unik di file cocok dengan yang di-hardcode di script.
3. Buka file `.alp` dengan text editor, cari ID yang disebut di error.
4. Sesuaikan ID di script Python dengan yang ada di file.

---

## 10. Checklist Final

### Persiapan
- [ ] Project `Perpustakaan3D_AntriCounter1` sudah di-save as `Perpustakaan3D_Lengkap`
- [ ] File `SimulasiParkirPerpustakaan.alp` sudah dicopy ke folder project

### Agent & Variabel
- [ ] `MobilPengunjung` agent type sudah dibuat
- [ ] `currentCar` (MobilPengunjung) sudah ditambah di Main
- [ ] `myCar` (MobilPengunjung) sudah ditambah di MahasiswaPed

### Road Network & Markup
- [ ] `roadNetwork` sudah ada
- [ ] `road` sudah dibuat dan terhubung
- [ ] `areaPejalanKaki` (Target line) sudah dibuat
- [ ] `pintuPerpus` (Node) sudah dibuat

### Flowchart Kendaraan
- [ ] `carSource` → `moveMenujuAntrian` → `prosesParkirMobil`
- [ ] `prosesParkirMobil` → `selectOutput`
- [ ] `selectOutput.outT` → `tungguPemilik`
- [ ] `selectOutput.outF` → `moveKeluar`
- [ ] `tungguPemilik` → `moveKeluar` → `buangMobil`
- [ ] `carSource.Rate schedule` = `jadwalKedatangan`
- [ ] `carSource.New car` = `MobilPengunjung`
- [ ] `prosesParkirMobil.onExit` berisi `orangTurun.inject(1)`

### Flowchart Pedestrian
- [ ] `orangTurun` → `jalanKePerpus` → `srvCounter` → `jalanKeParkiran` → `snkSelesai`
- [ ] `orangTurun.Target line` = `areaPejalanKaki`
- [ ] `orangTurun.Arrive according to` = `Manual`
- [ ] `orangTurun.New pedestrian` = `MahasiswaPed`
- [ ] `orangTurun.onExit` berisi `ped.myCar = currentCar`
- [ ] `jalanKePerpus.Target node` = `pintuPerpus`
- [ ] `jalanKeParkiran.Target line` = `areaPejalanKaki`
- [ ] `snkSelesai.onEnter` berisi `tungguPemilik.free(ped.myCar)`

### Schedule & Event
- [ ] `jadwalKedatangan` Schedule sudah dibuat
- [ ] `timerPalangTurun` Event sudah dibuat (opsional)

### Verifikasi
- [ ] Run 30 menit: mobil datang, parkir, penumpang turun
- [ ] Run 30 menit: penumpang antre di counter
- [ ] Run 30 menit: penumpang kembali ke parkir, mobil keluar
- [ ] Tidak ada error di console
- [ ] Semua agent type terdefinisi dengan benar
- [ ] Dashboard menampilkan statistik real-time

---

> **Catatan Akhir:** Integrasi ini menggabungkan dua model AnyLogic yang awalnya terpisah — simulasi parkir dan simulasi perpustakaan 3D — menjadi satu model utuh. Pendekatan yang digunakan adalah **flowchart separation with data coupling**: flowchart kendaraan dan pedestrian tetap terpisah secara blok, tetapi terhubung lewat variabel bersama (`currentCar` / `myCar`) dan method (`inject()` / `free()`). Pola ini bisa dipakai untuk integrasi model AnyLogic lain di masa depan.
