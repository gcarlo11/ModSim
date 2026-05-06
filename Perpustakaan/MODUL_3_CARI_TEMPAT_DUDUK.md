# Modul 3: Cari Tempat Duduk
## Aktivitas Sebelum Duduk (Toilet, Cari Buku, Loker, Langsung) + Zona Belajar

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan mahasiswa/dosen yang ingin **belajar di perpustakaan**:

1. Datang (dari `selectTujuan.out3` skeleton) atau dari `srcMasuk` temporary
2. Memilih **aktivitas sebelum duduk**: mampir toilet, cari buku, loker, atau langsung
3. Setelah aktivitas selesai → **pilih zona duduk**: silent zone atau diskusi zone
4. **Duduk dan belajar** (waktu sesuai zona: ~45 menit atau ~90 menit)
5. Selesai → jalan keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    │
    ▼
selectAktivitas (4 output — PedSelectOutput)
  ├── out1 (15%) → srvToiletSinggah → toilet dulu 2-4 mnt
  ├── out2 (30%) → srvCariBuku → eksplor rak buku 5-15 mnt
  ├── out3 (25%) → srvLoker → taruh barang 1-3 mnt
  └── out4 (30%) → langsung → (tanpa aktivitas)
    │
    ▼ (semua bergabung)
selectZonaDuduk (2 output — PedSelectOutput)
  ├── out1 (60%) → srvDudukSepi → silent zone ~45 mnt
  └── out2 (40%) → srvDudukDiskusi → diskusi zone ~90 mnt
    │
    ▼
snkSelesai (temporary)
```

**Konsep cepat untuk pemula:**
- **PedSelectOutput**: percabangan — mengarahkan orang berdasarkan probabilitas
- **PedService**: tempat antri + dilayani — di sini: toilet, cari buku, loker, duduk
- **Service with Lines (markup)**: area fisik di peta — menentukan titik service dan antrean
- **Shortest queue**: pedestrian otomatis pilih antrean terpendek

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMasuk` | TEMPORARY — untuk standalone |
| PedSelectOutput | `selectAktivitas` | 4 output: pilih aktivitas sebelum duduk |
| PedService | `srvToiletSinggah` | Mampir toilet 2-4 menit |
| PedService | `srvCariBuku` | Cari/eksplor buku 5-15 menit |
| PedService | `srvLoker` | Simpan barang di loker 1-3 menit |
| PedSelectOutput | `selectZonaDuduk` | 2 output: pilih zona sepi/diskusi |
| PedService | `srvDudukSepi` | Belajar di silent zone |
| PedService | `srvDudukDiskusi` | Belajar di diskusi zone |
| PedSink | `snkSelesai` | TEMPORARY — untuk standalone |

### 1.2 Markup

| Markup | Nama | Service Points | Antrean | Keterangan |
|---|---|---|---|---|
| Service with Lines | `svcToiletSinggah` | 2 | 1 | 2 bilik toilet |
| Service with Lines | `svcCariBuku` | 2 | 2 | 2 area rak buku |
| Service with Lines | `svcLoker` | 3 | 1 | 3 slot loker |
| Service with Lines | `svcAreaSepi` | 10 | 2 | 10 kursi individu |
| Service with Lines | `svcAreaDiskusi` | 4 | 2 | 4 meja diskusi |

### 1.3 3D (opsional untuk standalone)

| Objek | Nama | Jumlah |
|---|---|---|
| Floor | `floor3D` | 1 |
| Meja silent | `mejaSepi3D_1` s/d `_10` | 10 |
| Meja diskusi | `mejaDiskusi3D_1` s/d `_4` | 4 |
| Rak buku | `rakBuku3D_1`, `rakBuku3D_2` | 2 |
| Loker | `loker3D` | 1 (representasi 3 slot) |

### 1.4 Variabel baru di PengunjungPed

Tambah ke diagram `PengunjungPed`:

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `waktuMulaiAktivitas` | double | `0` | Waktu mulai aktivitas (toilet/cari buku/loker) |
| `waktuMulaiDuduk` | double | `0` | Waktu mulai duduk belajar |
| `durasiBelajar` | double | `0` | Total durasi belajar (menit) |
| `zonaDuduk` | String | `""` | "SEPI" atau "DISKUSI" |
| `aktivitasSebelumDuduk` | String | `""` | "TOILET", "BUKU", "LOKER", atau "LANGSUNG" |
| `jenisKelamin` | String | `""` | (sama dengan Modul 4 — integrasi nanti) |

> **Tips pemula:** Variabel-variabel ini akan menempel ke setiap pedestrian. Jadi data si A tidak tercampur dengan si B.

### 1.5 Variabel baru di Main

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `totalToiletSinggah` | int | 0 | Hitung pengguna toilet singgah |
| `totalCariBuku` | int | 0 | Hitung pencari buku |
| `totalLoker` | int | 0 | Hitung pengguna loker |
| `totalDudukSepi` | int | 0 | Hitung belajar di silent zone |
| `totalDudukDiskusi` | int | 0 | Hitung belajar di diskusi zone |
| `maxAntrianDudukSepi` | int | 0 | Pantau antrean maksimum silent zone |
| `maxAntrianDudukDiskusi` | int | 0 | Pantau antrean maksimum diskusi zone |
| `totalDurasiBelajar` | double | 0 | Akumulasi durasi belajar |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_CariDuduk`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 2.1 Buat/ubah PengunjungPed

Jika sudah ada pedestrian type `PengunjungPed` dari modul lain, buka saja dan **tambah variabel baru** (bagian 1.4). Jika belum ada:

1. Palette **Pedestrian Library** → drag **Pedestrian Type** → nama `PengunjungPed`.
2. Pilih animasi 3D.
3. Buka diagram `PengunjungPed` → tambah variabel sesuai tabel 1.4.

### 2.2 Tambah variabel Main

Buka Main → drag **Variable** → isi sesuai tabel 1.5.

---

## 3. Layout 3D dan Markup

### 3.1 3D dasar

1. **3D Window** → rename `win3D`.
2. **Camera** → rename `camMain`. Posisi: atur lihat area belajar.
3. **Box** → `floor3D` → skala 25x20x0.2.
4. Buat meja-meja kecil (Box) untuk `svcAreaSepi` (10 meja).
5. Buat meja besar (Box) untuk `svcAreaDiskusi` (4 meja).
6. Buat rak buku dan loker.

### 3.2 Target lines

1. **Target line** → `entryLine` (untuk standalone — letakkan di pojok kiri).
2. **Target line** → `exitLine` (untuk standalone — letakkan di pojok kanan).

### 3.3 Service with Lines — Toilet Singgah

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcToiletSinggah`.
3. **Number of services** = `2`.
4. **N of queues** = `1`.
5. Letakkan service point di area toilet.

### 3.4 Service with Lines — Cari Buku

1. **Service with Lines** → rename `svcCariBuku`.
2. **Number of services** = `2`.
3. **N of queues** = `2`.
4. Letakkan di dekat rak buku 3D.

### 3.5 Service with Lines — Loker

1. **Service with Lines** → rename `svcLoker`.
2. **Number of services** = `3`.
3. **N of queues** = `1`.
4. Letakkan di dekat pintu/lobby.

### 3.6 Service with Lines — Silent Zone

1. **Service with Lines** → rename `svcAreaSepi`.
2. **Number of services** = `10` (10 kursi).
3. **N of queues** = `2` (2 antrean, pilih terpendek).
4. Sebarkan 10 service point di area silent zone.

> **Tips pemula:** 10 service points berarti 10 kursi. Pedestrian otomatis pilih kursi dengan antrean terpendek di depan service point. Karena 2 queues, 5 kursi per queue.

### 3.7 Service with Lines — Diskusi Zone

1. **Service with Lines** → rename `svcAreaDiskusi`.
2. **Number of services** = `4` (4 meja).
3. **N of queues** = `2`.
4. Letakkan di area diskusi.

### 3.8 Posisi layout (contoh)

```
entryLine                                      exitLine
  |                                               ^
  |  [svcToiletSinggah]  [svcLoker]               |
  |  [svcCariBuku]                                |
  |     |                                         |
  |     v                                         |
  |  [svcAreaSepi — 10 kursi]  [svcAreaDiskusi]   |
  +----------------------------------------------->+
```

---

## 4. Bangun Flowchart

### 4.1 Blok

Drag dari **Pedestrian Library**:

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` |
| PedSelectOutput | `selectAktivitas` |
| PedService | `srvToiletSinggah` |
| PedService | `srvCariBuku` |
| PedService | `srvLoker` |
| PedSelectOutput | `selectZonaDuduk` |
| PedService | `srvDudukSepi` |
| PedService | `srvDudukDiskusi` |
| PedSink | `snkSelesai` |

### 4.2 Koneksi

```
srcMasuk.out → selectAktivitas.in

selectAktivitas.out1 (15%) → srvToiletSinggah.in
selectAktivitas.out2 (30%) → srvCariBuku.in
selectAktivitas.out3 (25%) → srvLoker.in
selectAktivitas.out4 (30%) → LANGSUNG → selectZonaDuduk.in

srvToiletSinggah.out → selectZonaDuduk.in  (bergabung)
srvCariBuku.out      → selectZonaDuduk.in  (bergabung)
srvLoker.out         → selectZonaDuduk.in  (bergabung)

selectZonaDuduk.out1 (60%) → srvDudukSepi.in
selectZonaDuduk.out2 (40%) → srvDudukDiskusi.in

srvDudukSepi.out    → snkSelesai.in
srvDudukDiskusi.out → snkSelesai.in
```

> **Catatan penting:** Di AnyLogic, satu input bisa menerima koneksi dari banyak output. Jadi `selectZonaDuduk.in` bisa menerima 4 koneksi sekaligus.

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcMasuk` (PedSource) — TEMPORARY

| Property | Value |
|---|---|
| `Appears at` | `line` |
| `Target line` | `entryLine` |
| `Interarrival time` | `exponential(2.0)` (1 orang setiap 2 menit) |
| `New pedestrian` | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "D-" + seqPed;
ped.tMasuk = time();
```

### 5.2 `selectAktivitas` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `4` |
| `Use probabilities` | Centang |
| Output 1 | `0.15` (15% ke toilet singgah) |
| Output 2 | `0.30` (30% cari buku) |
| Output 3 | `0.25` (25% loker) |
| Output 4 | `0.30` (30% langsung ke kursi) |

> **Verifikasi:** 0.15 + 0.30 + 0.25 + 0.30 = **1.0 (100%)**.

### 5.3 `srvToiletSinggah` (PedService)

| Property | Value |
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
traceln("AKTIVITAS " + ped.idPed + " | TOILET | durasi=" + String.format("%.1f", time()-ped.waktuMulaiAktivitas) + " mnt");
```

### 5.4 `srvCariBuku` (PedService)

| Property | Value |
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
traceln("AKTIVITAS " + ped.idPed + " | BUKU | durasi=" + String.format("%.1f", time()-ped.waktuMulaiAktivitas) + " mnt");
```

### 5.5 `srvLoker` (PedService)

| Property | Value |
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
traceln("AKTIVITAS " + ped.idPed + " | LOKER | durasi=" + String.format("%.1f", time()-ped.waktuMulaiAktivitas) + " mnt");
```

### 5.6 `selectZonaDuduk` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | Centang |
| Output 1 | `0.60` (60% silent zone) |
| Output 2 | `0.40` (40% diskusi zone) |

### 5.7 `srvDudukSepi` (PedService)

| Property | Value |
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

traceln("BELAJAR " + ped.idPed
    + " | zona=SEPI"
    + " | durasi=" + String.format("%.1f", ped.durasiBelajar) + " mnt");
```

### 5.8 `srvDudukDiskusi` (PedService)

| Property | Value |
|---|---|
| `Services` | `svcAreaDiskusi` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungDurasiBelajar(ped)` |

**Action On enter queue:**
```java
int q = srvDudukDiskusi.queueSize();
if (q > maxAntrianDudukDiskusi) {
    maxAntrianDudukDiskusi = q;
}
```

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

traceln("BELAJAR " + ped.idPed
    + " | zona=DISKUSI"
    + " | durasi=" + String.format("%.1f", ped.durasiBelajar) + " mnt");
```

### 5.9 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " mnt"
    + " | aktivitas=" + ped.aktivitasSebelumDuduk
    + " | zona=" + ped.zonaDuduk);
```

---

## 6. Fungsi di Main

### 6.1 `hitungWaktuToiletSinggah`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
// Toilet singgah: 2-4 menit
return uniform(2.0, 4.0);
```

### 6.2 `hitungWaktuCariBuku`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
// Eksplor rak buku: 5-15 menit
return uniform(5.0, 15.0);
```

### 6.3 `hitungWaktuLoker`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
// Simpan barang di loker: 1-3 menit
return uniform(1.0, 3.0);
```

### 6.4 `hitungDurasiBelajar`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
if (ped.zonaDuduk.equals("SEPI")) {
    // Silent zone: belajar individual, rata-rata 45 menit
    return exponential(45.0);
} else {
    // Diskusi zone: belajar kelompok, rata-rata 90 menit
    return exponential(90.0);
}
```

> **Penjelasan `exponential(45.0)`:** Rata-rata 45 menit, tapi distribusinya eksponensial — ada yang hanya 10 menit, ada yang sampai 2 jam. Ini lebih realistis daripada waktu tetap.

### 6.5 Fungsi tambahan (opsional)

```java
// avgWaktuSistem (return: double)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

### 6.6 Variabel tambahan di Main

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 7. Dashboard

Text dinamis (Palette **Presentation** → **Text** → Type: `Dynamic`):

```java
// Total belajar
"Total belajar: " + (totalDudukSepi + totalDudukDiskusi)
```

```java
// Detail zona
"Silent zone: " + totalDudukSepi + " | Diskusi: " + totalDudukDiskusi
```

```java
// Aktivitas sebelum duduk
"Toilet: " + totalToiletSinggah + " | Cari Buku: " + totalCariBuku + " | Loker: " + totalLoker
```

```java
// Antrian
"Antrian silent: " + srvDudukSepi.queueSize() + " (max: " + maxAntrianDudukSepi + ")"
```

```java
"Antrian diskusi: " + srvDudukDiskusi.queueSize() + " (max: " + maxAntrianDudukDiskusi + ")"
```

```java
// Rata-rata durasi
"Rata-rata durasi: " + String.format("%.1f", (totalDudukSepi+totalDudukDiskusi)==0 ? 0 : totalDurasiBelajar/(totalDudukSepi+totalDudukDiskusi)) + " mnt"
```

---

## 8. Jalankan untuk Uji Coba

### Test 1: Coba 15 menit

1. **Stop time** = `15`.
2. Run.
3. Amati: semua aktivitas terlihat? Ada yang ke toilet, cari buku, loker?

### Test 2: Test penuh 60 menit

1. **Stop time** = `60`.
2. Run.
3. Cek console: `AKTIVITAS D-1 | TOILET | ...`, `BELAJAR D-5 | zona=SEPI | ...`, `DONE D-1`.

### Perkiraan hasil wajar

- Dari ~30 pengunjung:
  - ~4-5 ke toilet singgah
  - ~9-10 cari buku
  - ~7-8 loker
  - ~9-10 langsung
- Dari total belajar:
  - ~60% pilih silent zone
  - ~40% pilih diskusi zone
- Antrean paling panjang di silent zone (10 kursi untuk ~15 orang)

---

## 9. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → ganti dengan koneksi dari `selectTujuan.out3` (skeleton).
2. **Hapus** `snkSelesai` (temporary) → colok `srvDudukSepi.out` dan `srvDudukDiskusi.out` ke `wJalanKeluar.in` (skeleton).

---

## 10. Troubleshooting

### Masalah: Semua langsung belajar, tidak ada aktivitas

**Penyebab:** `selectAktivitas` belum diatur probabilities.

**Solusi:** Cek Properties → **Use probabilities** di-centang → Output 1 = 0.15, dst.

### Masalah: Antrean silent zone selalu penuh

**Penyebab:** 10 kursi untuk banyak pengunjung, dengan exponential(45) menit waktu belajar — antrean wajar panjang.

**Solusi:** Untuk testing, kurangi waktu belajar jadi `exponential(20.0)` atau tambah kursi jadi 15.

### Masalah: Ada error "Queue choice policy"

**Penyebab:** `srvDudukSepi.Services` belum diisi `svcAreaSepi`.

**Solusi:** Pastikan setiap PedService pointing ke markup yang benar.

### Masalah: `hitungDurasiBelajar(ped)` error "ped cannot be resolved"

**Penyebab:** Parameter fungsi salah nama.

**Solusi:** Cek parameter function — harus `PengunjungPed ped` (bukan `Agent agent`).

### Masalah: Semua orang ke toilet, tidak ada yang belajar

**Penyebab:** Probabilitas `selectAktivitas` salah input.

**Solusi:** Cek: 0.15 + 0.30 + 0.25 + 0.30 = 1.0. Jika total < 1, output sisanya akan error.

---

## 11. Fallback / Alternatif

### Jika model terlalu lambat

Kurangi jumlah service point:
- `svcAreaSepi`: 10 → 6
- `svcAreaDiskusi`: 4 → 2

### Jika ingin testing cepat

Setel `hitungDurasiBelajar(ped)` sementara return `uniform(5.0, 10.0)` saja.

---

## 12. Mode Presentasi

- **Stop time:** 30-40 menit (cukup untuk menunjukkan pola)
- **Fokus kamera 3D:** area silent zone dan diskusi zone
- **Speed slider runtime:** geser ke kiri agar gerakan belajar terlihat

---

## 13. Checklist Final

- [ ] **Variabel PengunjungPed**: `waktuMulaiAktivitas`, `waktuMulaiDuduk`, `durasiBelajar`, `zonaDuduk`, `aktivitasSebelumDuduk`
- [ ] **Markup**: 5 Service with Lines (svcToiletSinggah, svcCariBuku, svcLoker, svcAreaSepi, svcAreaDiskusi)
- [ ] **selectAktivitas**: 4 output (15/30/25/30) ✅ total 1.0
- [ ] **selectZonaDuduk**: 2 output (60/40)
- [ ] **srvDudukSepi**: Services = `svcAreaSepi` (10 kursi), Delay = `hitungDurasiBelajar(ped)`
- [ ] **srvDudukDiskusi**: Services = `svcAreaDiskusi` (4 meja), Delay = `hitungDurasiBelajar(ped)`
- [ ] **Fungsi**: `hitungWaktuToiletSinggah`, `hitungWaktuCariBuku`, `hitungWaktuLoker`, `hitungDurasiBelajar`
- [ ] **Dashboard** menampilkan statistik real-time
- [ ] **Test standalone** 60 menit berhasil
