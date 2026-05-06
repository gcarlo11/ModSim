# Modul 5: Fotokopi / Scan
## 2 Mesin Fotokopi dengan Kemungkinan Rusak (+ Repair Time)

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan layanan fotokopi dan scan di perpustakaan:

1. Pengunjung datang dengan jumlah halaman tertentu (acak 5-50 lembar)
2. Pilih layanan: **fotokopi (70%)** atau **scan (30%)**
3. Antri di 2 mesin (shortest queue — pilih mesin yang kosong)
4. Waktu service = jumlah halaman × waktu per lembar
5. **5% kemungkinan mesin rusak** → tambah waktu repair 5-15 menit
6. Selesai → jalan keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
srvFotokopi (PedService, 2 mesin — svcFotokopi)
  - delay: hitungWaktuFotokopi(ped)
  - 5% chance mesin rusak → repair time ditambah di dalam fungsi
    ↓
snkSelesai (temporary)
```

**Konsep cepat untuk pemula:**
- Hanya perlu **1 PedService** dengan **2 service points** (2 mesin)
- Antrean otomatis pilih mesin yang kosong (shortest queue)
- Kerusakan mesin disimulasikan dengan menambah waktu delay di fungsi, bukan fitur recovery delay bawaan

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMasuk` | TEMPORARY — untuk standalone |
| PedService | `srvFotokopi` | Layanan fotokopi/scan (2 mesin) |
| PedSink | `snkSelesai` | TEMPORARY — untuk standalone |

### 1.2 Markup

| Markup | Nama | Service Points | Antrean |
|---|---|---|---|
| Service with Lines | `svcFotokopi` | 2 | 1 |

> **Penjelasan:** 2 service points = 2 mesin fotokopi. 1 queue = satu antrean, ambil mesin yang kosong pertama.

### 1.3 3D (opsional)

| Objek | Nama | Jumlah |
|---|---|---|
| Floor | `lantaiFotokopi3D` | 1 |
| Mesin fotokopi 1 | `mesinFotokopi1_3D` | 1 |
| Mesin fotokopi 2 | `mesinFotokopi2_3D` | 1 |

### 1.4 Variabel baru di PengunjungPed

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `jumlahHalaman` | int | `0` | Jumlah halaman yang akan difotokopi/discan |
| `tipeLayananFotokopi` | String | `""` | "FOTOKOPI" atau "SCAN" |
| `mesinRusak` | boolean | `false` | `true` jika mesin rusak saat melayani |
| `waktuMulaiFotokopi` | double | `0` | Waktu mulai dilayani |

### 1.5 Variabel baru di Main

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `totalFotokopi` | int | 0 | Hitung transaksi fotokopi |
| `totalScan` | int | 0 | Hitung transaksi scan |
| `maxAntrianFotokopi` | int | 0 | Pantau antrean maksimum |
| `totalMesinRusak` | int | 0 | Hitung berapa kali mesin rusak |
| `totalWaktuRepair` | double | 0 | Akumulasi waktu repair |
| `totalHalamanDiproses` | int | 0 | Total halaman yang diproses |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_Fotokopi`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 2.1 Buat/ubah PengunjungPed

Jika sudah ada pedestrian type `PengunjungPed`:
- Buka diagram → **tambah 4 variable** sesuai tabel 1.4

Jika belum ada:
1. **Pedestrian Library** → drag **Pedestrian Type** → nama `PengunjungPed`.
2. Pilih animasi 3D.
3. Buka diagram → tambah: `jumlahHalaman` (int, 0), `tipeLayananFotokopi` (String, ""), `mesinRusak` (boolean, false), `waktuMulaiFotokopi` (double, 0).

### 2.2 Tambah variabel Main

Buka Main → drag **Variable** 6 kali → isi sesuai tabel 1.5.

---

## 3. Layout 3D dan Markup

### 3.1 3D

1. **3D Window** → `win3D`.
2. **Camera** → `camMain`. Atur posisi agar 2 mesin terlihat.
3. **Box** → `lantaiFotokopi3D`, skala 12x10x0.2.
4. **Box** → `mesinFotokopi1_3D`, ukuran 2x1x1.5. Letakkan di kiri.
5. **Box** → `mesinFotokopi2_3D`, ukuran 2x1x1.5. Letakkan di kanan (berjarak ~3 meter).

### 3.2 Target lines (untuk standalone)

1. **Target line** → `entryLine` (letakkan di pojok kiri).
2. **Target line** → `exitLine` (letakkan di pojok kanan).

### 3.3 svcFotokopi (Service with Lines)

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcFotokopi`.
3. **Number of services** = `2` (2 mesin).
4. **N of queues** = `1` (1 antrean — otomatis ke mesin kosong).
5. Letakkan service point 1 di depan `mesinFotokopi1_3D` dan service point 2 di depan `mesinFotokopi2_3D`.
6. Queue line (antrean) tarik menjauh dari mesin — misal ke arah pintu masuk.

> **Tips pemula:** Service point adalah lingkaran hijau di markup. Taruh tepat di depan Box mesin. Queue line adalah garis kuning — pedestrian akan antri di garis ini menunggu giliran.

---

## 4. Bangun Flowchart

### 4.1 Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` |
| PedService | `srvFotokopi` |
| PedSink | `snkSelesai` |

### 4.2 Koneksi

```
srcMasuk.out → srvFotokopi.in → snkSelesai.in
```

Sederhana. Hanya 3 blok.

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcMasuk` (PedSource) — TEMPORARY

| Property | Value | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di target line |
| `Target line` | `entryLine` | Garis masuk |
| `Interarrival time` | `exponential(0.8)` | Rata-rata 1 orang setiap 1.25 menit |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian |

**Action On exit:**
```java
seqPed++;
ped.idPed = "F-" + seqPed;
ped.tMasuk = time();

// Generate jumlah halaman: antara 5-50 lembar
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

> **Tips pemula:** `uniform_discr(5, 50)` = bilangan bulat (integer) acak antara 5 sampai 50. Bedanya dengan `uniform(5, 50)` yang menghasilkan angka desimal (double).

### 5.2 `srvFotokopi` (PedService)

| Property | Value | Penjelasan |
|---|---|---|
| `Services` | `svcFotokopi` | Pilih markup yang sudah dibuat |
| `Queue choice policy` | `Shortest queue` | Ambil mesin yang antreannya paling pendek |
| `Delay time` | `hitungWaktuFotokopi(ped)` | Panggil fungsi hitung waktu |
| `Recovery delay` | `0` | Tidak pakai recovery internal |

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

### 5.3 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("DONE " + ped.idPed + " | tSistem=" + String.format("%.2f", tSistem) + " mnt");
```

---

## 6. Fungsi di Main

### 6.1 `hitungWaktuFotokopi`

Return type: `double`. Parameter: `PengunjungPed ped`.

**Fungsi ini adalah inti modul** — menghitung waktu service + menangani kerusakan mesin.

```java
// Tentukan waktu per lembar berdasarkan jenis layanan
double waktuPerLembar;

if (ped.tipeLayananFotokopi.equals("FOTOKOPI")) {
    // Fotokopi: 0.05-0.12 menit per lembar (lebih cepat)
    waktuPerLembar = uniform(0.05, 0.12);
} else {
    // Scan: 0.08-0.15 menit per lembar (lebih lambat karena resolusi)
    waktuPerLembar = uniform(0.08, 0.15);
}

// Hitung total waktu = jumlah halaman × waktu per lembar
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

**Contoh perhitungan:**
- 20 halaman × 0.1 menit/lembar = **2 menit** (fotokopi normal)
- 20 halaman × 0.12 menit/lembar = **2.4 menit** + repair 7.5 menit = **9.9 menit** (fotokopi + rusak)
- 30 halaman × 0.1 menit/lembar = **3 menit** (scan normal)

### 6.2 Fungsi tambahan

```java
// hitungWaktuRepairMesin (return: double) — alternatif jika pakai Recovery delay
return uniform(5.0, 15.0);
```

```java
// avgWaktuSistem (return: double)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

### 6.3 Variabel tambahan

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 7. Dashboard

```java
// Text 1
"Total fotokopi: " + totalFotokopi + " | Scan: " + totalScan
```

```java
// Text 2 — antrian
"Antrian: " + srvFotokopi.queueSize() + " (max: " + maxAntrianFotokopi + ")"
```

```java
// Text 3 — kerusakan
"Mesin rusak: " + totalMesinRusak + " kali | Total repair: " + String.format("%.1f", totalWaktuRepair) + " mnt"
```

```java
// Text 4 — halaman
"Halaman diproses: " + totalHalamanDiproses
```

```java
// Text 5 — rata-rata
"Rata-rata waktu: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## 8. Jalankan untuk Uji Coba

### Test 1: Coba 10 menit

1. **Stop time** = `10`.
2. Run.
3. Apakah pedestrian muncul, antri, dan dilayani?

### Test 2: Test penuh 30 menit

1. **Stop time** = `30`.
2. Run.
3. Cek console: `ARRIVE F-1 | FOTOKOPI | 23 halaman`, `SERVICE F-1 | FOTOKOPI | ...`, mungkin ada `RUSAK Mesin fotokopi!`.

### Perkiraan hasil wajar

- Total pengunjung: ~20-25 orang (exponential 0.8 = 1 orang per 1.25 menit)
- ~70% fotokopi, ~30% scan
- Mesin rusak 1-2 kali (5% dari 20 = ~1)
- Rata-rata durasi layanan: 2-5 menit (tergantung jumlah halaman)

---

## 9. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → ganti dengan koneksi dari `selectTujuan.out5` (skeleton).
2. **Hapus** `snkSelesai` (temporary) → colok `srvFotokopi.out` ke `wJalanKeluar.in` (skeleton).

---

## 10. Troubleshooting

### Masalah: Mesin rusak terlalu sering

**Penyebab:** Probabilitas 5% bisa terasa sering jika banyak pengunjung.

**Solusi:** Turunkan jadi `uniform(0, 1) < 0.02` (2%) atau `uniform(0, 1) < 0.01` (1%).

### Masalah: Waktu service terlalu singkat (pedestrian cuma 1 detik di mesin)

**Penyebab:** `jumlahHalaman` terlalu kecil atau `waktuPerLembar` terlalu kecil.

**Solusi:** Naikkan range jadi `uniform_discr(20, 100)` atau `uniform(0.1, 0.2)`.

### Masalah: Kedua mesin selalu kosong

**Penyebab:** Arrival rate terlalu lambat atau jumlah halaman terlalu sedikit.

**Solusi:** Ubah `exponential(0.8)` jadi `exponential(1.5)` (lebih sering datang).

### Masalah: Error "queueSize() is not available"

**Penyebab:** Di AnyLogic versi tertentu, method queueSize() berada di PedService bukan di markup.

**Solusi:** Coba `srvFotokopi.queueSize()` (langsung dari blok service).

### Masalah: `hitungWaktuFotokopi(ped)` error

**Penyebab:** Parameter fungsi salah.

**Solusi:** Cek parameter function di Main → harus `PengunjungPed ped` (bukan `Agent agent`).

### Masalah: "equals" error

**Penyebab:** `equals` case-sensitive.
**Solusi:** Pastikan kode: `ped.tipeLayananFotokopi.equals("FOTOKOPI")`.

---

## 11. Fallback / Alternatif

### Jika ingin 3 mesin

Ubah `svcFotokopi.Number of services` = `3`, tambah 1 Box 3D.

### Jika ingin repair time via Recovery delay

Hapus kode kerusakan dari `hitungWaktuFotokopi`, aktifkan `Recovery delay` di srvFotokopi:
- **Failure condition**: `uniform(0, 1) < 0.05`
- **Recovery delay**: `hitungWaktuRepairMesin()`

---

## 12. Mode Presentasi

- **Stop time:** 15-20 menit
- **Fokus kamera 3D:** 2 mesin fotokopi dan antrean
- Bisa komentar: "lihat, ketika mesin rusak, antrean mengular karena hanya 1 mesin beroperasi"

---

## 13. Checklist Final

- [ ] **PengunjungPed**: `jumlahHalaman`, `tipeLayananFotokopi`, `mesinRusak`, `waktuMulaiFotokopi`
- [ ] **Variabel Main**: `totalFotokopi`, `totalScan`, `maxAntrianFotokopi`, `totalMesinRusak`, `totalWaktuRepair`, `totalHalamanDiproses`
- [ ] **Markup**: `svcFotokopi` (2 services, 1 queue)
- [ ] **srvFotokopi.Services** = `svcFotokopi`
- [ ] **srvFotokopi.Delay time** = `hitungWaktuFotokopi(ped)`
- [ ] **Fungsi**: `hitungWaktuFotokopi(ped)` — bedakan fotokopi/scan, 5% mesin rusak
- [ ] **Dashboard**: statistik real-time
- [ ] **Test 30 menit**: semua route jalan, ada log RUSAK
