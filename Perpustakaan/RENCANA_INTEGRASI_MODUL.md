# Rencana Integrasi Modul Simulasi Perpustakaan 3D

## Tujuan

Membagi simulasi perpustakaan menjadi 5 modul independen yang masing-masing bisa dikerjakan oleh orang berbeda, lalu digabungkan jadi satu model AnyLogic utuh.

---

## Arsitektur

Semua modul dihubungkan lewat satu **hub sentral** (`selectTujuan`) di Main:

```
srcMasuk -> selectTujuan (hub)

  -> srvPinjamKembali   (Modul 5 - 45%)
  -> srvCariDuduk       (Modul 2 - 30%)
  -> srvToilet          (Modul 3 - 10%)
  -> srvFotokopi        (Modul 4 - 5%)
  -> (keluar langsung)  (Modul 1 - 10%)

-> wJalanKeluar -> snkSelesai
```

Setiap modul hanya berisi **PedService + markup + variabel/fungsi terkait**.

---

## 6 File Tutorial yang Akan Dibuat

### 1. `MODUL_1_KEDATANGAN_SKELETON.md` — Base / Skeleton
**Dikerjakan oleh:** Orang 1 (paling awal, jadi fondasi)

Apa yang ada di modul ini:
| Komponen | Detail |
|---|---|
| **PengunjungPed** | Pedestrian type dengan variabel CORE saja (idPed, tMasuk, isDosen, jenisKelamin) |
| **entryLine** | Target line masuk |
| **exitLine** | Target line keluar |
| **floor3D + win3D + camMain** | Layout ruangan dasar |
| **srcMasuk** | PedSource dengan peak hour |
| **selectTujuan** | PedSelectOutput dengan **5 output port** (calon untuk 5 tujuan) |
| **wJalanKeluar** | PedWait ke exitLine |
| **snkSelesai** | PedSink |
| **Fungsi** | `getInterarrivalTime()`, `avgWaktuSistem()` |
| **Variabel Main** | `seqPed`, `totalSelesai`, `totalWaktuSistem` |
| **Properti 3D** | Lantai, dinding (kotak sederhana) |

> **Status:** Dimiliki penuh oleh Orang 1. Setelah selesai, model ini jadi template untuk semua modul lain.

---

### 2. `MODUL_2_CARI_TEMPAT_DUDUK.md` — Cari Tempat Duduk
**Dikerjakan oleh:** Orang 2

```
Pengunjung -> srvCariDuduk -> keluar
```

| Komponen | Detail |
|---|---|
| **svcAreaSepi** | Service with Lines — zona silent study (10 kursi individu, 1 antrean) |
| **svcAreaDiskusi** | Service with Lines — zona diskusi (4 meja @4 kursi, 1 antrean) |
| **srvCariDuduk** | PedService dengan pilihan zona |
| **Variabel baru di PengunjungPed** | `waktuMulaiDuduk`, `areaDuduk` (String) |
| **Variabel baru di Main** | `totalDudukSepi`, `totalDudukDiskusi`, `maxAntrianDuduk` |
| **Fungsi** | `hitungDurasiBelajar(ped)` |
| **Integrasi** | Output `srvCariDuduk` ke `wJalanKeluar`, input dari `selectTujuan.out2` |

---

### 3. `MODUL_3_ANTRIAN_TOILET.md` — Antrian Toilet
**Dikerjakan oleh:** Orang 3

```
Pengunjung -> (gender check) -> srvToiletPria / srvToiletWanita -> keluar
```

| Komponen | Detail |
|---|---|
| **svcToiletPria** | Service with Lines — 2 urinal + 2 bilik = 4 service point |
| **svcToiletWanita** | Service with Lines — 4 bilik = 4 service point |
| **srvToiletPria** | PedService untuk pria |
| **srvToiletWanita** | PedService untuk wanita |
| **Variabel baru di PengunjungPed** | `waktuMulaiToilet` |
| **Variabel baru di Main** | `totalToiletPria`, `totalToiletWanita`, `maxAntrianToilet` |
| **Fungsi** | `hitungWaktuToilet()` |
| **Integrasi** | Output ke `wJalanKeluar`, input dari `selectTujuan.out3` |

---

### 4. `MODUL_4_FOTOKOPI_SCAN.md` — Fotokopi / Scan
**Dikerjakan oleh:** Orang 4

```
Pengunjung -> srvFotokopi -> keluar
```

| Komponen | Detail |
|---|---|
| **svcFotokopi** | Service with Lines — 2 mesin fotokopi |
| **srvFotokopi** | PedService |
| **Variabel baru di PengunjungPed** | `jumlahHalaman`, `tipeLayanan` (String) |
| **Variabel baru di Main** | `totalFotokopi`, `totalScan`, `maxAntrianFotokopi`, `totalMesinRusak` |
| **Fungsi** | `hitungWaktuFotokopi(ped)` |
| **Integrasi** | Output ke `wJalanKeluar`, input dari `selectTujuan.out4` |

---

### 5. `MODUL_5_PEMINJAMAN_PENGEMBALIAN.md` — Peminjaman & Pengembalian
**Dikerjakan oleh:** Orang 5

Ini adalah **adaptasi modular** dari tutorial `TUTORIAL_3D_SIMULASI_PERPUSTAKAAN_LENGKAP.md` yang sudah ada.

```
Pengunjung -> selectTujuanPinjamKembali (70/30) -> srvPinjam / srvKembali -> keluar
```

| Komponen | Detail |
|---|---|
| **svcPeminjaman** | Service with Lines — 2 service point |
| **svcPengembalian** | Service with Lines — 1 service point |
| **srvPinjam** | PedService untuk peminjaman |
| **srvKembali** | PedService untuk pengembalian |
| **Variabel baru di PengunjungPed** | `isPeminjaman`, `jumlahBuku`, `tipePinjaman`, `deadlineHari`, `nomorResi`, `scannerError`, `waktuPinjam`, `waktuKembali`, `jumlahHariTerlambat`, `denda` |
| **Variabel baru di Main** | `totalPeminjaman`, `totalPengembalian`, `totalDenda`, `totalBukuRusak`, `maxQueuePeminjaman`, `maxQueuePengembalian`, `totalBukuDipinjam`, `totalErrorScanner` |
| **Fungsi** | `hitungWaktuServicePeminjaman(ped)`, `hitungWaktuServicePengembalian(ped)` |
| **Integrasi** | Output ke `wJalanKeluar`, input dari `selectTujuan.out1` |

---

### 6. `PANDUAN_INTEGRASI.md` — Panduan Integrasi
**Dokumen final** yang menjelaskan cara menggabungkan semua modul.

| Bagian | Detail |
|---|---|
| **Persiapan** | Duplikasi model, naming convention |
| **Langkah 1** | Copy skeleton Modul 1 sebagai base |
| **Langkah 2-5** | Copy masing-masing modul (markup, blok, variabel, fungsi) |
| **Konfigurasi Hub** | Set `selectTujuan` probabilities untuk 5 output |
| **Merge Pedestrian Type** | Daftar komplit semua variabel PengunjungPed |
| **Merge Variabel Main** | Daftar komplit semua variabel Main dari semua modul |
| **Merge Fungsi** | Daftar komplit semua fungsi |
| **Dashboard Final** | Gabungan dashboard dari semua modul |
| **Layout 3D Final** | Posisi semua markup dalam satu ruangan |
| **Testing** | Uji coba terintegrasi |
| **Troubleshooting** | Konflik nama, koneksi, variabel |

---

## Aturan Naming (Wajib, untuk integrasi lancar)

| Kategori | Format | Contoh |
|---|---|---|
| Pedestrian type | `PengunjungPed` | — |
| Target line entry | `entryLine` | — |
| Target line exit | `exitLine` | — |
| PedSource | `srcMasuk` | — |
| PedSelectOutput | `selectTujuan` | — |
| PedWait ke exit | `wJalanKeluar` | — |
| PedSink | `snkSelesai` | — |
| Service markup | `svc{Nama}` | `svcPeminjaman`, `svcToiletPria` |
| PedService block | `srv{Nama}` | `srvPinjam`, `srvCariDuduk` |
| Variabel statistik | `total{X}`, `max{X}` | `totalPeminjaman`, `maxAntrianToilet` |
| Fungsi hitung service | `hitung{Nama}(ped)` | `hitungWaktuServicePeminjaman(ped)` |

---

## Alur Pengerjaan

```
Orang 1: Kerjakan Modul 1 (skeleton)
      |
      v (setelah skeleton jadi)
      |
Orang 2,3,4,5: Kerjakan modul masing-masing (independen, bisa paralel)
      |
      v (semua selesai)
      |
Siapa saja: Ikuti PANDUAN_INTEGRASI untuk menggabungkan
      |
      v
Satu model AnyLogic utuh dengan 5 layanan!
```

---

## Timeline Estimasi

| Modul | Perkiraan Halaman | Tingkat Kesulitan |
|---|---|---|
| Modul 1: Skeleton | ~300 baris | Mudah (fondasi) |
| Modul 2: Cari Duduk | ~250 baris | Mudah |
| Modul 3: Toilet | ~250 baris | Mudah |
| Modul 4: Fotokopi | ~250 baris | Sedang (ada mesin rusak) |
| Modul 5: Pinjam/Kembali | ~350 baris | Sedang (paling kompleks) |
| Panduan Integrasi | ~300 baris | Kritis (paling penting) |

---

Setuju dengan rencana ini? Jika ada yang perlu diubah, saya bisa sesuaikan sebelum mulai menulis semua file.
