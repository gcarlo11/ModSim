# Laporan Simulasi Modul 7: Peminjaman & Pengembalian Buku
## Simulasi Sirkulasi Buku dengan Browsing Rak, Error Scanner, dan Denda Keterlambatan

**Software:** AnyLogic 8.9.8 | **Satuan Waktu:** Menit | **Durasi Simulasi:** 60 menit

---

## 1. Pendahuluan

### 1.1 Latar Belakang

Layanan sirkulasi (peminjaman dan pengembalian) adalah fungsi inti perpustakaan. Efisiensi layanan ini berdampak langsung pada kepuasan pengunjung. Antrean panjang di counter peminjaman maupun pengembalian menjadi indikator bahwa kapasitas layanan perlu dievaluasi.

Simulasi ini memodelkan proses lengkap sirkulasi buku — mulai dari pengunjung memilih jenis layanan, browsing rak buku, antre di counter, hingga transaksi selesai — untuk mengukur metrik kinerja dan mengidentifikasi potensi bottleneck.

### 1.2 Tujuan

1. Mengukur rata-rata waktu sistem pengunjung dari masuk hingga selesai
2. Menganalisis panjang antrean maksimum di counter peminjaman dan pengembalian
3. Mengevaluasi dampak error scanner (5%) terhadap waktu layanan peminjaman
4. Menghitung akumulasi denda keterlambatan pada layanan pengembalian
5. Membandingkan performa sistem pada 3 skenario: Normal, Ramai, dan Perbaikan

---

## 2. Deskripsi Sistem

### 2.1 Alur Simulasi

```
Pengunjung Datang (exponential 2.0)
    │
    ▼
selectJenisLayanan (70% Pinjam / 30% Kembali)
    │
    ├── JALUR PINJAM (70%)
    │   ├── goToRakBuku → jalan ke area rak
    │   ├── wKelilingRak → browsing 2-5 menit
    │   ├── goToCounterPinjam → ke counter
    │   ├── selectJadiPinjam (90% jadi / 10% batal)
    │   └── srvPinjam → 2 counter, error scanner 5%
    │
    └── JALUR KEMBALI (30%)
        ├── goToCounterKembali → langsung ke counter
        └── srvKembali → 1 counter, cek buku rusak 10%, hitung denda
```

### 2.2 Komponen Blok

| Blok | Jenis | Fungsi |
|---|---|---|
| `srcMasuk` | PedSource | Sumber pengunjung (temporary standalone) |
| `selectJenisLayanan` | PedSelectOutput | Routing 70% pinjam / 30% kembali |
| `goToRakBuku` | PedGoTo | Jalan ke area rak buku |
| `wKelilingRak` | PedWait | Browsing rak (uniform 2-5 menit) |
| `goToCounterPinjam` | PedGoTo | Jalan ke counter peminjaman |
| `selectJadiPinjam` | PedSelectOutput | 90% jadi pinjam / 10% batal |
| `srvPinjam` | PedService | Counter peminjaman (2 titik) |
| `goToCounterKembali` | PedGoTo | Jalan ke counter pengembalian |
| `srvKembali` | PedService | Counter pengembalian (1 titik) |
| `snkSelesai` | PedSink | Keluar sistem (temporary) |

### 2.3 Variabel Statistik Utama

| Variabel | Keterangan |
|---|---|
| `totalPeminjaman` | Total transaksi peminjaman |
| `totalPengembalian` | Total transaksi pengembalian |
| `totalBrowsing` | Total pengunjung yang browsing rak |
| `totalBatalPinjam` | Total yang batal setelah browsing |
| `totalBukuDipinjam` | Akumulasi buku dipinjam |
| `totalErrorScanner` | Total kejadian scanner error |
| `totalBukuRusak` | Total buku rusak saat pengembalian |
| `totalDenda` | Akumulasi denda (Rupiah) |
| `maxQueuePeminjaman` | Antrean maksimum counter pinjam |
| `maxQueuePengembalian` | Antrean maksimum counter kembali |
| `totalDurasiBrowsing` | Akumulasi waktu browsing |

---

## 3. Parameter Simulasi

### 3.1 Pola Kedatangan

| Parameter | Nilai | Keterangan |
|---|---|---|
| Distribusi antar kedatangan | `exponential(2.0)` | Rata-rata 1 orang setiap 2 menit |
| Tipe pengunjung | 20% Dosen, 80% Mahasiswa | Dosen pinjam lebih banyak (2-8 buku) |
| Stop time | 60 menit | — |

### 3.2 Probabilitas Routing

| Titik Keputusan | Output | Probabilitas |
|---|---|---|
| `selectJenisLayanan` | Pinjam | 70% |
| | Kembali | 30% |
| `selectJadiPinjam` | Jadi pinjam | 90% |
| | Batal | 10% |
| Tipe Pinjaman | REGULER (7 hari) | 70% |
| | REFERENSI (3 hari) | 20% |
| | RESERVE (1 hari) | 10% |

### 3.3 Waktu Layanan

| Layanan | Distribusi | Parameter |
|---|---|---|
| Browsing rak | `uniform(2.0, 5.0)` | 2-5 menit |
| Service peminjaman (Mhs, ≤2 buku) | `uniform(1.5, 2.2)` | 1.5-2.2 menit |
| Service peminjaman (Mhs, 3-4 buku) | `uniform(2.2, 3.2)` | 2.2-3.2 menit |
| Service peminjaman (Mhs, ≥5 buku) | `uniform(3.2, 4.5)` | 3.2-4.5 menit |
| Service peminjaman (Dosen) | 70% × waktu Mhs | Dosen 30% lebih cepat |
| Service pengembalian (≤2 buku) | `uniform(0.8, 1.5)` | 0.8-1.5 menit |
| Service pengembalian (3-4 buku) | `uniform(1.5, 2.5)` | 1.5-2.5 menit |
| Service pengembalian (≥5 buku) | `uniform(2.5, 3.5)` | 2.5-3.5 menit |

### 3.4 Kejadian Acak (Events)

| Kejadian | Probabilitas | Dampak |
|---|---|---|
| Scanner error (pinjam) | 5% | Tambah `uniform(0.8, 1.4)` menit |
| Buku rusak (kembali) | 10% | Tambah `uniform(1.0, 2.0)` menit |
| Denda keterlambatan | Jika `hariTerpakai > deadlineHari` | Rp 2.000 × hari telat × jumlah buku |

---

## 4. Skenario Pengujian

Tiga skenario diuji untuk menganalisis performa sistem pada kondisi berbeda.

### 4.1 Skenario A — Normal

| Parameter | Nilai |
|---|---|
| Interarrival time | `exponential(2.0)` (30 orang/jam) |
| Counter pinjam | 2 titik |
| Counter kembali | 1 titik |
| Durasi simulasi | 60 menit |

### 4.2 Skenario B — Ramai (Peak Hour)

| Parameter | Nilai |
|---|---|
| Interarrival time | `exponential(1.0)` (60 orang/jam) |
| Counter pinjam | 2 titik |
| Counter kembali | 1 titik |
| Durasi simulasi | 60 menit |

### 4.3 Skenario C — Perbaikan Layanan

| Parameter | Nilai |
|---|---|
| Interarrival time | `exponential(1.0)` (60 orang/jam) |
| Counter pinjam | **3 titik** (tambah 1) |
| Counter kembali | **2 titik** (tambah 1) |
| Durasi simulasi | 60 menit |

---

## 5. Hasil Simulasi (Estimasi)

### 5.1 Skenario A — Normal

| Metrik | Nilai Estimasi | Keterangan |
|---|---|---|
| Total pengunjung | ~30 orang | 60 mnt / 2 mnt per orang |
| Total peminjaman | ~21 orang | 70% dari 30 |
| Total pengembalian | ~9 orang | 30% dari 30 |
| Total browsing | ~21 orang | Semua jalur pinjam |
| Batal pinjam | ~2 orang | 10% dari 21 |
| Total buku dipinjam | ~63 buku | Rata-rata 3 buku × 21 |
| Error scanner | ~1 kejadian | 5% dari 21 |
| Buku rusak | ~1 buku | 10% dari 9 |
| Rata-rata browsing | ~3.5 menit | Tengah 2-5 menit |
| Max antrean pinjam | 2-3 orang | — |
| Max antrean kembali | 1-2 orang | — |
| Denda | Variatif | Tergantung keterlambatan |

### 5.2 Skenario B — Ramai

| Metrik | Nilai Estimasi | Keterangan |
|---|---|---|
| Total pengunjung | ~60 orang | 60 mnt / 1 mnt per orang |
| Total peminjaman | ~42 orang | 70% dari 60 |
| Total pengembalian | ~18 orang | 30% dari 60 |
| Total browsing | ~42 orang | — |
| Batal pinjam | ~4 orang | 10% dari 42 |
| Error scanner | ~2 kejadian | 5% dari 42 |
| Buku rusak | ~2 buku | 10% dari 18 |
| Max antrean pinjam | **5-8 orang** | ⚠️ Bottleneck |
| Max antrean kembali | **3-5 orang** | ⚠️ Bottleneck |

### 5.3 Skenario C — Perbaikan Layanan

| Metrik | Nilai Estimasi | Keterangan |
|---|---|---|
| Total pengunjung | ~60 orang | Sama dengan Skenario B |
| Max antrean pinjam | **2-4 orang** | Turun ~50% |
| Max antrean kembali | **1-2 orang** | Turun ~60% |
| Waktu tunggu rata-rata | **Turun 40-50%** | Signifikan lebih baik |

---

## 6. Analisis

### 6.1 Bottleneck Counter Peminjaman

Pada Skenario B (ramai), antrean peminjaman mencapai 5-8 orang. Ini terjadi karena:
- 70% pengunjung memilih jalur pinjam → beban counter pinjam 2.3× lebih berat dari counter kembali
- Setiap transaksi membutuhkan 1-4.5 menit tergantung jumlah buku
- 2 counter tidak cukup untuk 42 transaksi dalam 60 menit

**Utilisasi counter pinjam** pada Skenario B:
- Total waktu layanan ≈ 42 transaksi × rata-rata 2.5 menit = 105 menit kerja
- Kapasitas tersedia = 2 counter × 60 menit = 120 menit
- Utilisasi ≈ 105/120 = **87.5%** (tinggi, mendekati overload)

### 6.2 Dampak Error Scanner

Error scanner 5% menambah waktu `uniform(0.8, 1.4)` menit per kejadian. Dari 42 transaksi pinjam (Skenario B), sekitar 2 transaksi terkena error. Total tambahan waktu ≈ 2 × rata-rata 1.1 menit = **2.2 menit** tambahan — relatif kecil terhadap total kapasitas.

### 6.3 Dampak Browsing Rak

Setiap pengunjung jalur pinjam menghabiskan 2-5 menit untuk browsing. Ini **bukan bottleneck** karena browsing dilakukan sebelum antre — justru menyebarkan kedatangan ke counter secara lebih merata (tidak semua langsung ke counter bersamaan).

### 6.4 Fenomena Batal Pinjam

10% pengunjung batal setelah browsing — ini **mengurangi beban counter**. Dari 42 yang browsing (Skenario B), 4 orang batal → counter hanya melayani 38 orang. Efek: **mengurangi utilisasi ~10%**.

### 6.5 Perbandingan Skenario

| Metrik | Normal (A) | Ramai (B) | Perbaikan (C) |
|---|---|---|---|
| Max antrean pinjam | 2-3 | 5-8 ⚠️ | 2-4 ✅ |
| Max antrean kembali | 1-2 | 3-5 ⚠️ | 1-2 ✅ |
| Utilisasi counter pinjam | ~50% | ~87% ⚠️ | ~58% |
| Utilisasi counter kembali | ~35% | ~70% | ~35% |
| Waktu sistem rata-rata | ~12-15 mnt | ~18-25 mnt | ~12-16 mnt |

Skenario C menunjukkan bahwa **menambah 1 counter pinjam dan 1 counter kembali** secara signifikan menurunkan antrean dan utilisasi saat peak hour.

---

## 7. Kesimpulan

1. **Skenario Normal** dengan 2 counter pinjam dan 1 counter kembali sudah memadai untuk 30 pengunjung/jam — utilisasi counter ~50%, antrean pendek.

2. **Skenario Ramai** (60 pengunjung/jam) menyebabkan bottleneck di counter peminjaman dengan utilisasi ~87% dan antrean 5-8 orang. Counter pengembalian juga tertekan dengan utilisasi ~70%.

3. **Penambahan 1 counter** pada masing-masing layanan (Skenario C) menurunkan utilisasi ke level aman (~58% pinjam, ~35% kembali) dan memotong antrean maksimum hingga 50%.

4. **Error scanner 5%** memiliki dampak minimal terhadap total kapasitas sistem — hanya ~2 menit tambahan per 60 menit operasi.

5. **Browsing rak** berfungsi sebagai buffer alami yang menyebarkan kedatangan ke counter secara lebih merata.

6. **Fenomena batal pinjam 10%** mengurangi beban counter peminjaman — perlu dipertimbangkan dalam perencanaan kapasitas.

### 7.1 Rekomendasi

- Untuk operasional normal: **2 counter pinjam + 1 counter kembali** sudah optimal
- Untuk peak hour (jam 10-14): tambah **1 counter pinjam** (total 3) dan **1 counter kembali** (total 2)
- Pertimbangkan **self-service kiosk** untuk mengurangi beban counter pengembalian
- Maintenance scanner barcode rutin untuk menjaga error rate tetap di bawah 5%

---

## 8. Lampiran

### 8.1 Fungsi-Fungsi Utama

**`hitungWaktuServicePeminjaman(ped)`** — menghitung durasi layanan peminjaman berdasarkan jumlah buku, tipe pengunjung (dosen 30% lebih cepat), dan kemungkinan error scanner 5%.

**`hitungWaktuServicePengembalian(ped)`** — menghitung durasi layanan pengembalian, termasuk inspeksi buku rusak (10%) dan perhitungan denda keterlambatan (Rp 2.000/hari/buku).

**`hitungWaktuBrowsing()`** — mengembalikan `uniform(2.0, 5.0)` menit untuk simulasi waktu keliling rak.

### 8.2 Rumus Denda

```
hariTerpakai = (waktuSekarang - waktuPinjam) / (24 × 60)
jika hariTerpakai > deadlineHari:
    hariTerlambat = ceil(hariTerpakai - deadlineHari)
    denda = hariTerlambat × jumlahBuku × Rp 2.000
```

### 8.3 Dashboard Monitoring

| No | Teks Dashboard | Metrik |
|---|---|---|
| 1 | `Peminjaman: X | Kembali: Y` | Total transaksi |
| 2 | `Browsing rak: X | Rata-rata: Y mnt | Batal: Z%` | Statistik browsing |
| 3 | `Queue pinjam: X (max Y) | Queue kembali: X (max Y)` | Antrean real-time |
| 4 | `Buku dipinjam: X | Error scanner: Y%` | Throughput & error |
| 5 | `Buku rusak: X | Total denda: Rp Y` | Kerusakan & denda |
| 6 | `Rata-rata waktu sistem: X menit` | Overall performance |
