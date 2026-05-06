# Modul 5: Fotokopi / Scan
## 2 Mesin Fotokopi dengan Kemungkinan Rusak (+ Repair Time)

**Satuan waktu:** minute

---

## 1. Hasil Akhir

Modul ini mensimulasikan layanan fotokopi dan scan di perpustakaan:

1. Pengunjung datang dengan jumlah halaman tertentu
2. Pilih layanan: **fotokopi (70%)** atau **scan (30%)**
3. Antri di 2 mesin (shortest queue)
4. Service time = jumlah halaman × waktu per lembar
5. **5% kemungkinan mesin rusak** → repair time uniform(5, 15) menit
6. Selesai → keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
srvFotokopi (PedService, 2 mesin)
  - delay: hitungWaktuFotokopi(ped)
  - 5% chance mesin rusak → repair delay
    ↓
snkSelesai (temporary)
```

---

## 2. Komponen

### Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` (TEMPORARY) |
| PedService | `srvFotokopi` |
| PedSink | `snkSelesai` (TEMPORARY) |

### Markup

| Markup | Nama | Detail |
|---|---|---|
| Service with Lines | `svcFotokopi` | 2 mesin, 1 queue (shortest queue) |

### 3D

| Objek | Nama | Posisi (contoh) |
|---|---|---|
| Box | `lantaiFotokopi3D` | 12x10x0.2 |
| Box | `mesinFotokopi1_3D` | Mesin 1 |
| Box | `mesinFotokopi2_3D` | Mesin 2 |

### Variabel baru di PengunjungPed

| Nama | Type | Initial value |
|---|---|---|
| `jumlahHalaman` | int | `0` |
| `tipeLayananFotokopi` | String | `""` | "FOTOKOPI" atau "SCAN" |
| `mesinRusak` | boolean | `false` |
| `waktuMulaiFotokopi` | double | `0` |

### Variabel baru di Main

| Nama | Type | Initial value |
|---|---|---|
| `totalFotokopi` | int | 0 |
| `totalScan` | int | 0 |
| `maxAntrianFotokopi` | int | 0 |
| `totalMesinRusak` | int | 0 |
| `totalWaktuRepair` | double | 0 |
| `totalHalamanDiproses` | int | 0 |

---

## 3. Buat Project

1. **File → New → Model** → nama: `Perpustakaan3D_Fotokopi`.
2. **Time units**: `minute`.
3. Buat **Pedestrian Type**: `PengunjungPed`.
4. Tambah variabel ke `PengunjungPed`: `jumlahHalaman` (int), `tipeLayananFotokopi` (String), `mesinRusak` (boolean), `waktuMulaiFotokopi` (double).

---

## 4. Layout 3D & Markup

### 4.1 3D

1. **3D Window** → `win3D`. **Camera** → `camMain`.
2. **Box** → `lantaiFotokopi3D`, skala 12x10x0.2.
3. **Box** → `mesinFotokopi1_3D`, ukuran 2x1x1.5 (posisi di kiri).
4. **Box** → `mesinFotokopi2_3D`, ukuran 2x1x1.5 (posisi di kanan).

### 4.2 Markup

1. **Space Markup** → **Target line** → `entryLine` (buat standalone).
2. **Space Markup** → **Target line** → `exitLine` (buat standalone).
3. **Service with Lines** → rename `svcFotokopi`.
   - **Number of services** = `2` (2 mesin).
   - **N of queues** = `1` (1 antrean, shortest queue otomatis ke mesin yg kosong).
   - Letakkan service point di depan masing-masing mesin.

> **Tips pemula:** Service point adalah titik di mana pedestrian berhenti untuk dilayani. Letakkan tepat di depan mesin (Box 3D).

---

## 5. Flowchart

### 5.1 Blok dan koneksi

```
srcMasuk.out → srvFotokopi.in → snkSelesai.in
```

Sederhana. Hanya 1 PedService dengan 2 service points.

### 5.2 `srcMasuk` (PedSource — TEMPORARY)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` |
| Interarrival time | `exponential(0.8)` |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "F-" + seqPed;
ped.tMasuk = time();

// Generate jumlah halaman: 5-50 lembar
ped.jumlahHalaman = uniform_discr(5, 50);

// Pilih layanan: 70% fotokopi, 30% scan
if (uniform(0, 1) < 0.7) {
    ped.tipeLayananFotokopi = "FOTOKOPI";
} else {
    ped.tipeLayananFotokopi = "SCAN";
}

ped.mesinRusak = false;

traceln("ARRIVE " + ped.idPed
    + " | " + ped.tipeLayananFotokopi
    + " | " + ped.jumlahHalaman + " halaman");
```

### 5.3 `srvFotokopi` (PedService)

| Property | Value |
|---|---|
| Services | `svcFotokopi` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuFotokopi(ped)` |
| Recovery delay | `hitungWaktuRepairMesin()` |

> **Penting:** `Recovery delay` adalah waktu yang dibutuhkan untuk "memperbaiki" mesin jika terjadi kerusakan. AnyLogic akan otomatis memanggil ini saat ada error. Tapi untuk simulasi kita, kita akan handle kerusakan di fungsi `hitungWaktuFotokopi`.

**Action On enter queue:**
```java
int q = srvFotokopi.queueSize();
if (q > maxAntrianFotokopi) {
    maxAntrianFotokopi = q;
}
```

**Action On begin service:**
```java
ped.waktuMulaiFotokopi = time();
```

**Action On end service:**
```java
if (ped.tipeLayananFotokopi.equals("FOTOKOPI")) {
    totalFotokopi++;
} else {
    totalScan++;
}
totalHalamanDiproses += ped.jumlahHalaman;

traceln("SERVICE " + ped.idPed
    + " | " + ped.tipeLayananFotokopi
    + " | " + ped.jumlahHalaman + " halaman"
    + " | rusak=" + ped.mesinRusak
    + " | durasi=" + String.format("%.1f", time() - ped.waktuMulaiFotokopi) + " mnt");
```

### 5.4 `snkSelesai` (PedSink — TEMPORARY)

Action On enter:
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("DONE " + ped.idPed + " | tSistem=" + String.format("%.2f", tSistem) + " mnt");
```

---

## 6. Fungsi di Main

### hitungWaktuFotokopi

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double waktuPerLembar;

if (ped.tipeLayananFotokopi.equals("FOTOKOPI")) {
    // Fotokopi: 0.05-0.12 menit per lembar
    waktuPerLembar = uniform(0.05, 0.12);
} else {
    // Scan: 0.08-0.15 menit per lembar (lebih lama)
    waktuPerLembar = uniform(0.08, 0.15);
}

double totalWaktu = ped.jumlahHalaman * waktuPerLembar;

// 5% kemungkinan mesin rusak → tambah waktu repair
if (uniform(0, 1) < 0.05) {
    double waktuRepair = uniform(5.0, 15.0);
    totalWaktu += waktuRepair;
    ped.mesinRusak = true;
    totalMesinRusak++;
    totalWaktuRepair += waktuRepair;
    traceln("RUSAK Mesin fotokopi! " + ped.idPed + " repair=" + String.format("%.1f", waktuRepair) + " mnt");
}

return totalWaktu;
```

> **Cara kerja:** Jika jumlah halaman = 20 dan waktu per lembar = 0.1 menit, maka total = 20 × 0.1 = 2 menit. Jika mesin rusak (+5-15 menit) → total = 2 + 7.5 (misal) = 9.5 menit.

### hitungWaktuRepairMesin

Return type: `double`.

```java
return uniform(5.0, 15.0);
```

Fungsi ini dipakai untuk `Recovery delay` di `srvFotokopi` (jika menggunakan fitur internal AnyLogic).

### Variabel tambahan di Main

Tambahkan:
| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 7. Dashboard

```
"Total fotokopi: " + totalFotokopi + " | Scan: " + totalScan
```

```
"Antrian: " + srvFotokopi.queueSize() + " (max: " + maxAntrianFotokopi + ")"
```

```
"Mesin rusak: " + totalMesinRusak + " kali"
```

```
"Halaman diproses: " + totalHalamanDiproses
```

```
"Rata-rata waktu: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## 8. Jalankan Uji Coba

1. **Stop time** = 30 menit.
2. Run.
3. Harus terlihat: pedestrian muncul → antri → service di mesin → keluar.
4. Cek console: `ARRIVE F-1 | FOTOKOPI | 23 halaman`, `RUSAK Mesin fotokopi!`, `DONE F-1`.

---

## 9. Integrasi

### Yang diubah saat integrasi

1. **Hapus** `srcMasuk` → ganti dengan koneksi dari `selectTujuan.out5` (skeleton).
2. **Hapus** `snkSelesai` → colok `srvFotokopi.out` ke `wJalanKeluar.in` (skeleton).

### Masalah: Mesin rusak terlalu sering

**Penyebab:** Probabilitas 5% bisa terasa sering jika banyak pengunjung.

**Solusi:** Turunkan jadi `uniform(0, 1) < 0.02` (2%) atau `uniform(0, 1) < 0.01` (1%).

### Masalah: Waktu service terlalu singkat

**Penyebab:** Jumlah halaman sedikit atau waktu per lembar terlalu kecil.

**Solusi:** Naikkan range `uniform_discr(10, 100)` atau `uniform(0.1, 0.2)` untuk waktu per lembar.

---

## 10. Checklist

- [ ] `svcFotokopi` dengan 2 services, 1 queue
- [ ] `srvFotokopi.Services` = `svcFotokopi`
- [ ] `hitungWaktuFotokopi(ped)` membedakan fotokopi/scan
- [ ] 5% chance mesin rusak → repair time
- [ ] Statistik tercatat (totalFotokopi, totalScan, totalMesinRusak)
- [ ] Test standalone 30 menit berhasil
