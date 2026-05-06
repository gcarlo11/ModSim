# Modul 6: Peminjaman & Pengembalian Buku
## Dua Counter (Pinjam 2 titik + Kembali 1 titik) + Error Scanner + Denda

**Basis tutorial:** `TUTORIAL_3D_SIMULASI_PERPUSTAKAAN_LENGKAP.md`

**Satuan waktu:** minute

---

## 1. Hasil Akhir

Modul ini mensimulasikan layanan sirkulasi buku di perpustakaan:

1. Pengunjung datang dan memilih: **pinjam (70%)** atau **kembali (30%)**
2. **Peminjaman**: 2 counter, data buku diproses, 5% error scanner → service tambahan
3. **Pengembalian**: 1 counter, deteksi buku rusak (10%), hitung denda keterlambatan
4. Selesai → keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
selectJenisLayanan (PedSelectOutput)
  ├── srvPinjam (70%, PedService, 2 titik)
  │   - delay: hitungWaktuServicePeminjaman(ped)
  │   - 5% scanner error (waktu tambahan)
  │   - Dosen 30% lebih cepat
  │
  └── srvKembali (30%, PedService, 1 titik)
      - delay: hitungWaktuServicePengembalian(ped)
      - 10% buku rusak
      - Hitung denda: Rp 2.000/hari/buku
    ↓
snkSelesai (temporary)
```

---

## 2. Komponen

### Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` (TEMPORARY) |
| PedSelectOutput | `selectJenisLayanan` |
| PedService | `srvPinjam` |
| PedService | `srvKembali` |
| PedSink | `snkSelesai` (TEMPORARY) |

### Markup

| Markup | Nama | Detail |
|---|---|---|
| Service with Lines | `svcPeminjaman` | 2 services, 2 queues |
| Service with Lines | `svcPengembalian` | 1 service, 1 queue |

### 3D

| Objek | Nama |
|---|---|
| Box | `floor3D` |
| Box | `mejaPinjam3D` |
| Box | `mejaKembali3D` |
| Box | `rakBuku3D` (opsional) |

### Variabel baru di PengunjungPed

| Nama | Type | Initial value |
|---|---|---|
| `isPeminjaman` | boolean | `true` | true=pinjam, false=kembali |
| `jumlahBuku` | int | `1` |
| `tipePinjaman` | String | `"REGULER"` |
| `deadlineHari` | int | `7` |
| `nomorResi` | String | `""` |
| `scannerError` | boolean | `false` |
| `waktuPinjam` | double | `0` |
| `waktuKembali` | double | `0` |
| `jumlahHariTerlambat` | int | `0` |
| `denda` | double | `0` |

### Variabel baru di Main

| Nama | Type | Initial value |
|---|---|---|
| `totalPeminjaman` | int | 0 |
| `totalPengembalian` | int | 0 |
| `totalDenda` | double | 0 |
| `totalBukuRusak` | int | 0 |
| `maxQueuePeminjaman` | int | 0 |
| `maxQueuePengembalian` | int | 0 |
| `totalBukuDipinjam` | int | 0 |
| `totalErrorScanner` | int | 0 |

---

## 3. Buat Project

1. **File → New → Model** → nama: `Perpustakaan3D_PinjamKembali`.
2. **Time units**: `minute`.
3. Buat **Pedestrian Type**: `PengunjungPed`.
4. Tambah variabel dari tabel di atas ke diagram `PengunjungPed`.

> **Tips pemula:** Ada 10 variabel — pastikan semuanya ditambahkan. Nama variabel case-sensitive. Contoh: `isPeminjaman` (i kecil, P besar).

---

## 4. Layout 3D & Markup

### 4.1 3D

1. **3D Window** → `win3D`. **Camera** → `camMain`.
2. **Box** → `floor3D`, skala 20x15x0.2.
3. **Box** → `mejaPinjam3D`, ukuran 3x1x1.2, posisi (8, 10).
4. **Box** → `mejaKembali3D`, ukuran 3x1x1.2, posisi (14, 10).
5. **Box** (beberapa) → `rakBuku3D`, di sekeliling ruangan.

### 4.2 Markup

#### Target lines

1. **Target line** → `entryLine` (untuk standalone).
2. **Target line** → `exitLine` (untuk standalone).

#### svcPeminjaman

1. **Service with Lines** → rename `svcPeminjaman`.
2. **Number of services** = `2` (dua petugas/petugas counter).
3. **N of queues** = `2` (dua antrean, shortest queue).
4. Letakkan service point di depan `mejaPinjam3D`.

#### svcPengembalian

1. **Service with Lines** → rename `svcPengembalian`.
2. **Number of services** = `1` (satu petugas).
3. **N of queues** = `1`.
4. Letakkan di depan `mejaKembali3D`.

---

## 5. Flowchart

### 5.1 Koneksi

```
srcMasuk.out → selectJenisLayanan.in

selectJenisLayanan.out1 (70%) → srvPinjam.in
selectJenisLayanan.out2 (30%) → srvKembali.in

srvPinjam.out → snkSelesai.in
srvKembali.out → snkSelesai.in
```

### 5.2 `srcMasuk` (PedSource — TEMPORARY)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` |
| Interarrival time | `exponential(2.0)` |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "P-" + seqPed;
ped.tMasuk = time();

// Tipe pengunjung: 20% dosen
ped.isDosen = uniform(0, 1) < 0.2;

// Data akan digenerate di masing-masing service
traceln("ARRIVE " + ped.idPed + " | " + (ped.isDosen ? "DOSEN" : "MHS"));
```

> **Catatan:** Di tutorial lengkap sebelumnya, data peminjaman/pengembalian digenerate di `onExit`. Di versi modular ini, data digenerate di **masing-masing service** agar lebih rapi.

### 5.3 `selectJenisLayanan` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `2` |
| Use probabilities | Centang |
| Output 1 | `0.70` (pinjam) |
| Output 2 | `0.30` (kembali) |

### 5.4 `srvPinjam` (PedService)

| Property | Value |
|---|---|
| Services | `svcPeminjaman` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuServicePeminjaman(ped)` |
| Recovery delay | `0` |

**Action On enter queue:**
```java
int q = srvPinjam.queueSize();
if (q > maxQueuePeminjaman) {
    maxQueuePeminjaman = q;
}
```

**Action On begin service:**
```java
ped.tMulaiService = time();

// Generate data peminjaman di sini
ped.isPeminjaman = true;

if (ped.isDosen) {
    ped.jumlahBuku = uniform_discr(2, 8); // dosen pinjam lebih banyak
} else {
    ped.jumlahBuku = uniform_discr(1, 5);
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
```

**Action On end service:**
```java
ped.tSelesaiService = time();
ped.waktuPinjam = time();
ped.nomorResi = "PNJ-" + (long)(time() * 1000) + "-" + ped.idPed;

totalPeminjaman++;
totalBukuDipinjam += ped.jumlahBuku;

traceln("SERVICE " + ped.idPed
    + " | PINJAM"
    + " | resi=" + ped.nomorResi
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari"
    + " | scannerError=" + ped.scannerError);
```

### 5.5 `srvKembali` (PedService)

| Property | Value |
|---|---|
| Services | `svcPengembalian` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuServicePengembalian(ped)` |
| Recovery delay | `0` |

**Action On enter queue:**
```java
int q = srvKembali.queueSize();
if (q > maxQueuePengembalian) {
    maxQueuePengembalian = q;
}
```

**Action On begin service:**
```java
ped.tMulaiService = time();
ped.waktuKembali = time();
ped.isPeminjaman = false;

// Generate data untuk simulasi pengembalian
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
```

**Action On end service:**
```java
ped.tSelesaiService = time();
totalPengembalian++;

if (ped.jumlahHariTerlambat > 0) {
    traceln("DENDA " + ped.idPed
        + " | telat=" + ped.jumlahHariTerlambat + " hari"
        + " | denda=Rp " + String.format("%,.0f", ped.denda));
}

traceln("SERVICE " + ped.idPed
    + " | KEMBALI"
    + " | buku=" + ped.jumlahBuku
    + " | telat=" + ped.jumlahHariTerlambat + " hari"
    + " | denda=Rp " + String.format("%,.0f", ped.denda));
```

### 5.6 `snkSelesai` (PedSink — TEMPORARY)

Action On enter:
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " mnt"
    + " | " + (ped.isPeminjaman ? "PINJAM" : "KEMBALI"));
```

---

## 6. Fungsi di Main

### hitungWaktuServicePeminjaman

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double dasar;

if (ped.isDosen) {
    // Dosen: 30% lebih cepat
    if (ped.jumlahBuku <= 2)       dasar = uniform(1.0, 1.5);
    else if (ped.jumlahBuku <= 4)  dasar = uniform(1.5, 2.5);
    else                            dasar = uniform(2.5, 3.5);
} else {
    // Mahasiswa: waktu normal
    if (ped.jumlahBuku <= 2)       dasar = uniform(1.5, 2.2);
    else if (ped.jumlahBuku <= 4)  dasar = uniform(2.2, 3.2);
    else                            dasar = uniform(3.2, 4.5);
}

// 5% error scanner
ped.scannerError = false;
if (uniform(0, 1) < 0.05) {
    dasar += uniform(0.8, 1.4);
    ped.scannerError = true;
    totalErrorScanner++;
}

return dasar;
```

### hitungWaktuServicePengembalian

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double dasar;

if (ped.jumlahBuku <= 2)       dasar = uniform(0.8, 1.5);
else if (ped.jumlahBuku <= 4)  dasar = uniform(1.5, 2.5);
else                            dasar = uniform(2.5, 3.5);

// 10% buku rusak -> inspeksi tambahan
if (uniform(0, 1) < 0.1) {
    dasar += uniform(1.0, 2.0);
    totalBukuRusak++;
}

// Hitung denda keterlambatan
double waktuSekarang = time();
double hariTerpakai = (waktuSekarang - ped.waktuPinjam) / (24.0 * 60.0);

if (hariTerpakai > ped.deadlineHari) {
    ped.jumlahHariTerlambat = (int) Math.ceil(hariTerpakai - ped.deadlineHari);
    ped.denda = ped.jumlahHariTerlambat * ped.jumlahBuku * 2000.0;
    totalDenda += ped.denda;
}

return dasar;
```

### Fungsi dashboard

```java
// avgWaktuSistem (return: double)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;

// avgBukuPerTransaksi (return: double)
return totalPeminjaman == 0 ? 0 : (1.0 * totalBukuDipinjam / totalPeminjaman);

// errorScannerRate (return: double)
return totalPeminjaman == 0 ? 0 : (100.0 * totalErrorScanner / totalPeminjaman);

// rataRataDenda (return: double)
return totalPengembalian == 0 ? 0 : totalDenda / totalPengembalian;
```

### Variabel tambahan di Main

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 7. Dashboard

```
"Peminjaman: " + totalPeminjaman + " | Kembali: " + totalPengembalian
```

```
"Queue pinjam: " + srvPinjam.queueSize() + " (max " + maxQueuePeminjaman + ")"
```

```
"Queue kembali: " + srvKembali.queueSize() + " (max " + maxQueuePengembalian + ")"
```

```
"Buku dipinjam: " + totalBukuDipinjam + " | Error: " + totalErrorScanner
```

```
"Buku rusak: " + totalBukuRusak + " | Total denda: Rp " + String.format("%,.0f", totalDenda)
```

---

## 8. Jalankan Uji Coba

1. **Stop time** = 60 menit.
2. Run.
3. Harus: muncul → antri → service (pinjam/kembali) → keluar.
4. Cek console: `ARRIVE`, `SERVICE ... PINJAM`, `SERVICE ... KEMBALI`, `DENDA`, `DONE`.

---

## 9. Integrasi & Troubleshooting

### Yang diubah saat integrasi

1. **Hapus** `srcMasuk` → koneksi dari `selectTujuan.out1` (pinjam) dan `out2` (kembali) skeleton.
2. **Hapus** `selectJenisLayanan` → langsung colok `srvPinjam` ke `selectTujuan.out1` dan `srvKembali` ke `selectTujuan.out2`.
3. **Hapus** `snkSelesai` → colok ke `wJalanKeluar.in` skeleton.

### Masalah: Error "Cannot convert from Agent"

**Penyebab:** Fungsi `hitungWaktuServicePeminjaman(ped)` dipanggil dengan `agent` bukan `ped`.

**Solusi:** Di properties PedService, pastikan "Delay time" berisi `hitungWaktuServicePeminjaman(ped)`.

### Masalah: Denda tidak keluar

**Penyebab:** Waktu pinjam tidak cukup lama.

**Solusi:** Di `srvKembali.onBeginService`, pastikan `hariLalu = uniform(1.0, 10.0)` dengan 24*60.

---

## 10. Checklist

- [ ] `svcPeminjaman` (2 services) dan `svcPengembalian` (1 service) sudah dibuat
- [ ] `srvPinjam.Services` = `svcPeminjaman`, `srvKembali.Services` = `svcPengembalian`
- [ ] `selectJenisLayanan` 70/30
- [ ] `hitungWaktuServicePeminjaman(ped)` dengan error scanner 5%
- [ ] `hitungWaktuServicePengembalian(ped)` dengan buku rusak 10% + denda
- [ ] Statistik tercatat
- [ ] Test standalone 60 menit berhasil
