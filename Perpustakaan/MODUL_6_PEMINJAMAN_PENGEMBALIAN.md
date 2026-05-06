# Modul 6: Peminjaman & Pengembalian Buku
## Dua Counter (Pinjam 2 titik + Kembali 1 titik) + Error Scanner + Denda

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan layanan sirkulasi buku di perpustakaan:

1. Pengunjung datang (dari `selectTujuan.out1/out2` skeleton) atau dari `srcMasuk` temporary
2. Memilih: **pinjam buku (70%)** atau **kembalikan buku (30%)**
3. **Peminjaman**: 2 counter, data buku diproses di onBeginService, 5% error scanner → waktu tambahan
4. **Pengembalian**: 1 counter, deteksi buku rusak (10%), hitung denda keterlambatan Rp 2.000/hari/buku
5. Selesai → jalan keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
selectJenisLayanan (PedSelectOutput — 70/30)
  ├── srvPinjam (70%, PedService, 2 titik)
  │   - delay: hitungWaktuServicePeminjaman(ped)
  │   - 5% scanner error → tambah 0.8-1.4 menit
  │   - Dosen 30% lebih cepat
  │
  └── srvKembali (30%, PedService, 1 titik)
      - delay: hitungWaktuServicePengembalian(ped)
      - 10% buku rusak → inspeksi tambahan 1-2 menit
      - Hitung denda: Rp 2.000/hari/buku
    ↓
snkSelesai (temporary)
```

**Konsep cepat untuk pemula:**
- **selectJenisLayanan**: percabangan 70% pinjam, 30% kembali
- **Service with Lines — svcPeminjaman**: 2 counter pinjam, 2 antrean (shortest queue)
- **Service with Lines — svcPengembalian**: 1 counter kembali, 1 antrean
- Data peminjaman digenerate di **onBeginService** (bukan di srcMasuk) agar lebih rapi dan modular

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMasuk` | TEMPORARY — untuk standalone |
| PedSelectOutput | `selectJenisLayanan` | Pilih pinjam (70%) atau kembali (30%) |
| PedService | `srvPinjam` | Counter peminjaman (2 titik) |
| PedService | `srvKembali` | Counter pengembalian (1 titik) |
| PedSink | `snkSelesai` | TEMPORARY — untuk standalone |

### 1.2 Markup

| Markup | Nama | Service Points | Antrean |
|---|---|---|---|
| Service with Lines | `svcPeminjaman` | 2 | 2 (shortest queue) |
| Service with Lines | `svcPengembalian` | 1 | 1 |

### 1.3 3D (opsional)

| Objek | Nama | Ukuran |
|---|---|---|
| Floor | `floor3D` | 20x15x0.2 |
| Meja counter pinjam | `mejaPinjam3D` | 3x1x1.2 |
| Meja counter kembali | `mejaKembali3D` | 3x1x1.2 |
| Rak buku (opsional) | `rakBuku3D` | — |

### 1.4 Variabel baru di PengunjungPed

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `isPeminjaman` | boolean | `true` | `true`=pinjam, `false`=kembali |
| `jumlahBuku` | int | `1` | Jumlah buku yang dipinjam/dikembalikan |
| `tipePinjaman` | String | `"REGULER"` | "REGULER", "REFERENSI", atau "RESERVE" |
| `deadlineHari` | int | `7` | Deadline pengembalian (hari) |
| `nomorResi` | String | `""` | Nomor bukti transaksi |
| `scannerError` | boolean | `false` | `true` jika scanner barcode error |
| `waktuPinjam` | double | `0` | Waktu transaksi peminjaman |
| `waktuKembali` | double | `0` | Waktu transaksi pengembalian |
| `jumlahHariTerlambat` | int | `0` | Jumlah hari keterlambatan (jika ada) |
| `denda` | double | `0` | Total denda yang harus dibayar |

### 1.5 Variabel baru di Main

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `totalPeminjaman` | int | 0 | Total transaksi peminjaman |
| `totalPengembalian` | int | 0 | Total transaksi pengembalian |
| `totalDenda` | double | 0 | Akumulasi denda |
| `totalBukuRusak` | int | 0 | Total buku rusak saat pengembalian |
| `maxQueuePeminjaman` | int | 0 | Antrean maksimum counter pinjam |
| `maxQueuePengembalian` | int | 0 | Antrean maksimum counter kembali |
| `totalBukuDipinjam` | int | 0 | Total buku yang dipinjam |
| `totalErrorScanner` | int | 0 | Total kejadian error scanner |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_PinjamKembali`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 2.1 Buat/ubah PengunjungPed

Jika sudah ada `PengunjungPed` dari modul lain:
- Buka diagram → **tambah 10 variable** sesuai tabel 1.4

Jika belum ada:
1. **Pedestrian Library** → drag **Pedestrian Type** → nama `PengunjungPed`.
2. Pilih animasi 3D.
3. Buka diagram → tambah 10 variable satu per satu.

> **Tips pemula:** Ada 10 variabel — pastikan semuanya ditambahkan. Nama variabel **case-sensitive**. Contoh: `isPeminjaman` (i kecil, P besar).

### 2.2 Tambah variabel Main

Buka Main → drag **Variable** 8 kali → isi sesuai tabel 1.5.

---

## 3. Layout 3D & Markup

### 3.1 3D

1. **3D Window** → `win3D`.
2. **Camera** → `camMain`.
3. **Box** → `floor3D`, skala 20x15x0.2.
4. **Box** → `mejaPinjam3D`, ukuran 3x1x1.2. Posisi: (8, 10).
5. **Box** → `mejaKembali3D`, ukuran 3x1x1.2. Posisi: (14, 10).
6. (Opsional) **Box** → `rakBuku3D`, beberapa buah di sekeliling.

### 3.2 Target lines (untuk standalone)

1. **Target line** → `entryLine`.
2. **Target line** → `exitLine`.

### 3.3 svcPeminjaman (Service with Lines)

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcPeminjaman`.
3. **Number of services** = `2` (dua petugas counter).
4. **N of queues** = `2` (dua antrean — pedestrian pilih yang terpendek).
5. Letakkan service point di depan `mejaPinjam3D`.

### 3.4 svcPengembalian (Service with Lines)

1. **Service with Lines**.
2. Rename: `svcPengembalian`.
3. **Number of services** = `1` (satu petugas).
4. **N of queues** = `1`.
5. Letakkan service point di depan `mejaKembali3D`.

---

## 4. Bangun Flowchart

### 4.1 Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` |
| PedSelectOutput | `selectJenisLayanan` |
| PedService | `srvPinjam` |
| PedService | `srvKembali` |
| PedSink | `snkSelesai` |

### 4.2 Koneksi

```
srcMasuk.out → selectJenisLayanan.in

selectJenisLayanan.out1 (70%, pinjam) → srvPinjam.in
selectJenisLayanan.out2 (30%, kembali) → srvKembali.in

srvPinjam.out → snkSelesai.in
srvKembali.out → snkSelesai.in
```

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcMasuk` (PedSource) — TEMPORARY

| Property | Value | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di target line |
| `Target line` | `entryLine` | Garis masuk |
| `Interarrival time` | `exponential(2.0)` | Rata-rata 1 orang setiap 2 menit |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian |

**Action On exit:**
```java
seqPed++;
ped.idPed = "P-" + seqPed;
ped.tMasuk = time();

// Tipe pengunjung: 20% dosen
ped.isDosen = uniform(0, 1) < 0.2;

traceln("ARRIVE " + ped.idPed + " | " + (ped.isDosen ? "DOSEN" : "MHS"));
```

> **Catatan:** Data peminjaman/pengembalian (jumlah buku, tipe pinjaman, dll) digenerate di **masing-masing service** (onBeginService), bukan di sini. Ini biar lebih rapi.

### 5.2 `selectJenisLayanan` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | Centang |
| Output 1 | `0.70` (70% pinjam) |
| Output 2 | `0.30` (30% kembali) |

### 5.3 `srvPinjam` (PedService)

| Property | Value | Penjelasan |
|---|---|---|
| `Services` | `svcPeminjaman` | Pilih markup |
| `Queue choice policy` | `Shortest queue` | Pilih antrean terpendek |
| `Delay time` | `hitungWaktuServicePeminjaman(ped)` | Fungsi waktu service |
| `Recovery delay` | `0` | — |

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
    // Dosen pinjam lebih banyak: 2-8 buku
    ped.jumlahBuku = uniform_discr(2, 8);
} else {
    // Mahasiswa: 1-5 buku
    ped.jumlahBuku = uniform_discr(1, 5);
}

// Tipe pinjaman: 70% REGULER, 20% REFERENSI, 10% RESERVE
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

### 5.4 `srvKembali` (PedService)

| Property | Value | Penjelasan |
|---|---|---|
| `Services` | `svcPengembalian` | Pilih markup |
| `Queue choice policy` | `Shortest queue` | — |
| `Delay time` | `hitungWaktuServicePengembalian(ped)` | Fungsi waktu service |
| `Recovery delay` | `0` | — |

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

### 5.5 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
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

### 6.1 `hitungWaktuServicePeminjaman`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double dasar;

if (ped.isDosen) {
    // Dosen: 30% lebih cepat (lebih pengalaman, tahu prosedur)
    if (ped.jumlahBuku <= 2)       dasar = uniform(1.0, 1.5);
    else if (ped.jumlahBuku <= 4)  dasar = uniform(1.5, 2.5);
    else                            dasar = uniform(2.5, 3.5);
} else {
    // Mahasiswa: waktu normal
    if (ped.jumlahBuku <= 2)       dasar = uniform(1.5, 2.2);
    else if (ped.jumlahBuku <= 4)  dasar = uniform(2.2, 3.2);
    else                            dasar = uniform(3.2, 4.5);
}

// 5% error scanner — barcode tidak terbaca
ped.scannerError = false;
if (uniform(0, 1) < 0.05) {
    dasar += uniform(0.8, 1.4); // waktu tambahan scan manual
    ped.scannerError = true;
    totalErrorScanner++;
}

return dasar;
```

### 6.2 `hitungWaktuServicePengembalian`

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
// Rumus: (hariTerpakai - deadlineHari) * jumlahBuku * Rp 2.000
double waktuSekarang = time();
double hariTerpakai = (waktuSekarang - ped.waktuPinjam) / (24.0 * 60.0);

if (hariTerpakai > ped.deadlineHari) {
    ped.jumlahHariTerlambat = (int) Math.ceil(hariTerpakai - ped.deadlineHari);
    ped.denda = ped.jumlahHariTerlambat * ped.jumlahBuku * 2000.0;
    totalDenda += ped.denda;
}

return dasar;
```

**Contoh perhitungan denda:**
- Buku dipinjam 7 hari lalu, deadline 7 hari, 3 buku → **tepat waktu, tidak kena denda**
- Buku dipinjam 10 hari lalu, deadline 7 hari, 2 buku → telat 3 hari × 2 buku × Rp 2.000 = **Rp 12.000**
- Buku dipinjam 5 hari lalu, deadline 1 hari (RESERVE), 4 buku → telat 4 hari × 4 buku × Rp 2.000 = **Rp 32.000**

### 6.3 Fungsi dashboard

```java
// avgWaktuSistem (return: double)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;

// avgBukuPerTransaksi (return: double)
return totalPeminjaman == 0 ? 0 : (1.0 * totalBukuDipinjam / totalPeminjaman);

// errorScannerRate (return: double) — dalam persen
return totalPeminjaman == 0 ? 0 : (100.0 * totalErrorScanner / totalPeminjaman);

// rataRataDenda (return: double)
return totalPengembalian == 0 ? 0 : totalDenda / totalPengembalian;
```

### 6.4 Variabel tambahan di Main

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 7. Dashboard

```java
// Text 1 — total transaksi
"Peminjaman: " + totalPeminjaman + " | Kembali: " + totalPengembalian
```

```java
// Text 2 — antrian pinjam
"Queue pinjam: " + srvPinjam.queueSize() + " (max " + maxQueuePeminjaman + ")"
```

```java
// Text 3 — antrian kembali
"Queue kembali: " + srvKembali.queueSize() + " (max " + maxQueuePengembalian + ")"
```

```java
// Text 4 — buku
"Buku dipinjam: " + totalBukuDipinjam + " | Error scanner: " + totalErrorScanner
```

```java
// Text 5 — denda
"Buku rusak: " + totalBukuRusak + " | Total denda: Rp " + String.format("%,.0f", totalDenda)
```

---

## 8. Jalankan untuk Uji Coba

### Test 1: Coba 15 menit

1. **Stop time** = `15`.
2. Run.
3. Apakah ada yang pinjam dan kembali?

### Test 2: Test penuh 60 menit

1. **Stop time** = `60`.
2. Run.
3. Cek console: `ARRIVE`, `SERVICE ... PINJAM`, `SERVICE ... KEMBALI`, mungkin ada `DENDA`, `DONE`.

### Perkiraan hasil wajar

- Total pengunjung: ~25-30 orang (exponential 2.0 = 1 orang per 2 menit)
- ~70% pinjam (~17-21 orang), ~30% kembali (~8-9 orang)
- Error scanner: 5% dari ~17 peminjaman = ~1 kejadian
- Buku rusak: 10% dari ~8 pengembalian = ~1 buku
- Denda: tergantung keterlambatan

---

## 9. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → `selectTujuan.out1` colok ke `srvPinjam.in`, `selectTujuan.out2` colok ke `srvKembali.in`.
2. **Hapus** `selectJenisLayanan` → routing pinjam/kembali langsung dari `selectTujuan` skeleton.
3. **Hapus** `snkSelesai` (temporary) → colok `srvPinjam.out` dan `srvKembali.out` ke `wJalanKeluar.in` skeleton.

---

## 10. Troubleshooting

### Masalah: Error "Cannot convert from Agent"

**Penyebab:** Fungsi `hitungWaktuServicePeminjaman(ped)` dipanggil dengan `agent` bukan `ped`.

**Solusi:** Cek properties PedService → "Delay time" harus berisi `hitungWaktuServicePeminjaman(ped)` (bukan `hitungWaktuServicePeminjaman(agent)`).

### Masalah: Denda selalu Rp 0

**Penyebab 1:** Waktu pinjam tidak cukup lama (`hariLalu` terlalu kecil).
**Solusi:** Di `srvKembali.onBeginService`, pastikan `hariLalu = uniform(1.0, 10.0)` dikali `24 * 60` (konversi hari ke menit).

**Penyebab 2:** `deadlineHari` lebih besar dari `hariTerpakai`.
**Solusi:** Cek tipe pinjaman — "RESERVE" punya deadline 1 hari, jadi 2 hari sudah telat.

### Masalah: `tMulaiService` tidak dikenal

**Penyebab:** Variabel `tMulaiService` belum ada di PengunjungPed.

**Solusi:** Tambah variable `tMulaiService` (double, 0) atau ganti kode dengan `waktuMulaiFotokopi` atau buat variabel baru.

### Masalah: "String.format error"

**Penyebab:** `String.format("%,.0f", ...)` mungkin tidak kompatibel di semua versi AnyLogic.

**Solusi:** Ganti dengan `Math.round(denda)` + `"Rp "` + `Double.toString(Math.round(denda))`.

### Masalah: Counter pinjam selalu kosong

**Penyebab:** `srvPinjam.Services` belum diset ke `svcPeminjaman`.

**Solusi:** Cek properties → pastikan pointing ke markup yang benar.

---

## 11. Fallback / Alternatif

### Jika ingin tanpa selectJenisLayanan (routing langsung dari skeleton)

Hapus `selectJenisLayanan`. Di `srcMasuk.onExit`, langsung set:
```java
ped.isPeminjaman = uniform(0, 1) < 0.7;
```

Tapi ini hanya untuk standalone — saat integrasi, `selectTujuan` skeleton sudah handle routing.

### Jika ingin error scanner lebih sering

Ubah `uniform(0, 1) < 0.05` jadi `uniform(0, 1) < 0.10` (10%).

---

## 12. Mode Presentasi

- **Stop time:** 30-40 menit
- **Fokus kamera 3D:** kedua counter (pinjam + kembali)
- Bisa tunjukkin: "lihat, antrian pinjam lebih panjang karena 70% orang datang untuk pinjam"
- Sorot denda: "ada yang kena denda Rp 12.000 karena telat 3 hari"

---

## 13. Checklist Final

- [ ] **PengunjungPed**: 10 variabel pinjam/kembali (lihat tabel 1.4)
- [ ] **Variabel Main**: 8 variabel statistik (tabel 1.5) + 3 tambahan
- [ ] **Markup**: `svcPeminjaman` (2 services, 2 queues), `svcPengembalian` (1 service, 1 queue)
- [ ] **selectJenisLayanan**: 70/30
- [ ] **srvPinjam.Services** = `svcPeminjaman`, Delay = `hitungWaktuServicePeminjaman(ped)`
- [ ] **srvKembali.Services** = `svcPengembalian`, Delay = `hitungWaktuServicePengembalian(ped)`
- [ ] **Fungsi**: `hitungWaktuServicePeminjaman` (error 5%, dosen 30% lebih cepat)
- [ ] **Fungsi**: `hitungWaktuServicePengembalian` (buku rusak 10%, denda Rp 2.000/hari/buku)
- [ ] **Fungsi dashboard**: `avgWaktuSistem`, `avgBukuPerTransaksi`, `errorScannerRate`, `rataRataDenda`
- [ ] **Dashboard**: 5 text dinamis
- [ ] **Test 60 menit**: transaksi pinjam & kembali berjalan, console log lengkap
