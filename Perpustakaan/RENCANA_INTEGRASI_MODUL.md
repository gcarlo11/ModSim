# Rencana Integrasi Modul Simulasi Perpustakaan 3D

## Tujuan

Membagi simulasi perpustakaan menjadi **modul-modul independen** yang masing-masing bisa dikerjakan oleh orang berbeda, lalu digabungkan jadi satu model AnyLogic utuh.

---

## Arsitektur Final

```
                          ┌── srvPinjam (25%)
                          ├── srvKembali (15%)
srcMasuk ──→ selectTujuan ┼── srvCariDuduk (25%)   ──→ wJalanKeluar ──→ ...
(jalan kaki)              ├── srvToilet (10%)
    ^                     ├── srvFotokopi (10%)
    |                     └── srvLangsungKeluar (15%)
    |                                        ↓       ← Modul 1.
    |                                  selectPulang (isParkir?)
    |                                   ├── ya → wJalanBalikParkir → snkParkir
    |                                   └── tidak → snkSelesai
    |
srcParkir ──→ wJalanKePintu ──→ selectTujuan
(mobil)  (jalan parkir→pintu)      (sama persis di atas)
```

## 7 File Tutorial

| # | File | Penanggung Jawab | Isi Singkat |
|---|---|---|---|
| 1 | `MODUL_1_KEDATANGAN_KEPERGIAN.md` | Orang 1 | **Skeleton**: entry/exit, hub selectTujuan, srcMasuk, 3D dasar |
| 2 | `MODUL_2_PARKIR.md` | Orang 2 | Parkir mobil: srcParkir → wJalanKePintu, 3D parkir |
| 3 | `MODUL_3_CARI_TEMPAT_DUDUK.md` | Orang 3 | Silent zone + diskusi zone, pilih area duduk |
| 4 | `MODUL_4_ANTRIAN_TOILET.md` | Orang 4 | Toilet pria (4 titik) + wanita (4 titik), pisah gender |
| 5 | `MODUL_5_FOTOKOPI_SCAN.md` | Orang 5 | 2 mesin, 5% chance rusak, repair time |
| 6 | `MODUL_6_PEMINJAMAN_PENGEMBALIAN.md` | Orang 6 | Adaptasi dari tutorial lengkap yang sudah ada |
| 7 | `PANDUAN_INTEGRASI.md` | — | Integrator: cara gabung semua jadi 1 model |

## Aturan Naming

| Kategori | Format | Contoh |
|---|---|---|
| Pedestrian type | `PengunjungPed` | — |
| PedSource | `srcMasuk`, `srcParkir` | — |
| PedSelectOutput | `selectTujuan`, `selectPulang` | — |
| PedWait | `wJalanKeluar`, `wJalanKePintu`, `wJalanBalikParkir` | — |
| PedSink | `snkSelesai`, `snkParkir` | — |
| Service markup | `svc{Nama}` | `svcPeminjaman`, `svcToiletPria` |
| PedService | `srv{Nama}` | `srvPinjam`, `srvCariDuduk` |
| Variabel | `total{X}`, `max{X}` | `totalPeminjaman`, `maxAntrianToilet` |
| Fungsi service | `hitungWaktu{X}(ped)` | `hitungWaktuParkir(ped)` |
| 3D objects | `{nama}3D` | `mejaPinjam3D`, `toilet3D` |

## Pembagian 3D

**Setiap modul punya 3D-nya sendiri** untuk development. Saat integrasi, posisi disesuaikan:

| Modul | 3D Dibuat Sendiri | Koordinat Final (Integrasi) |
|---|---|---|
| 1. Kedatangan | floor, dinding, entry, exit | Tetap |
| 2. Parkir | area parkir, mobil | (2,2) - (8,5) |
| 3. Cari Duduk | meja belajar, kursi | (20,10) - (28,20) |
| 4. Toilet | bilik toilet | (2,16) - (8,24) |
| 5. Fotokopi | mesin fotokopi | (10,16) - (14,20) |
| 6. Pinjam/Kembali | meja counter, rak buku | (12,6) - (22,14) |

## Alur Pengerjaan

```
Orang 1: Kerjakan Modul 1 (skeleton)
    ↓ (setelah skeleton jadi)
Orang 2-6: Kerjakan modul masing-masing (PARALEL, independen)
    ↓ (semua selesai)
Integrator: Ikuti PANDUAN_INTEGRASI.md
    ↓
Satu model AnyLogic utuh — Perpustakaan3D_Full.alp
```

## Catatan Penting

- Setiap modul bisa **dijalankan standalone** (dengan srcMasuk dan snkSelesai sendiri)
- Semua modul pake nama pedestrian type **sama**: `PengunjungPed`
- Semua modul pake satuan waktu **sama**: `minute`
- Integrasi = copy-paste blok, markup, variabel, fungsi — 100% manual di AnyLogic
