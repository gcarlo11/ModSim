# Modul 7: Peminjaman & Pengembalian Buku
## Dua Counter + Browsing Rak + Error Scanner + Denda

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan layanan sirkulasi buku di perpustakaan dengan alur yang lebih realistis:

1. Pengunjung datang (dari `selectTujuan.out1/out2` skeleton) atau dari `srcMasuk` temporary
2. Memilih: **pinjam buku (70%)** atau **kembalikan buku (30%)**
3. **Jalur PINJAM**: jalan ke rak buku → keliling/browsing rak (2-5 mnt) → ke counter → 90% jadi pinjam / 10% batal → dilayani counter → selesai
4. **Jalur KEMBALI**: langsung ke counter kembali → dilayani → selesai
5. **Peminjaman**: 2 counter, data buku diproses di onBeginService, 5% error scanner
6. **Pengembalian**: 1 counter, deteksi buku rusak (10%), hitung denda Rp 2.000/hari/buku

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
selectJenisLayanan (PedSelectOutput — 70/30)
    │
    ├── PINJAM (70%):
    │   ├── goToRakBuku (PedGoTo) → jalan ke area rak buku
    │   ├── wKelilingRak (PedWait, uniform 2-5 mnt) → browsing/keliling rak
    │   ├── goToCounterPinjam (PedGoTo) → dari rak ke counter pinjam
    │   ├── selectJadiPinjam (PedSelectOutput, 90/10)
    │   │   ├── out1 (90%) → srvPinjam (PedService, 2 titik)
    │   │   │   - delay: hitungWaktuServicePeminjaman(ped)
    │   │   │   - 5% scanner error → tambah 0.8-1.4 menit
    │   │   │   - Dosen 30% lebih cepat
    │   │   └── out2 (10%) → langsung ke snkSelesai (batal)
    │
    └── KEMBALI (30%):
        ├── goToCounterKembali (PedGoTo) → langsung ke counter kembali
        └── srvKembali (PedService, 1 titik)
            - delay: hitungWaktuServicePengembalian(ped)
            - 10% buku rusak → inspeksi tambahan 1-2 menit
            - Hitung denda: Rp 2.000/hari/buku
          ↓
    snkSelesai (temporary)
```

**Konsep cepat untuk pemula:**
- **PedGoTo**: perintah jalan — pedestrian bergerak dari posisi saat ini ke target node
- **PedWait**: perintah menunggu — pedestrian diam di tempat selama waktu tertentu (simulasi browsing)
- **PedSelectOutput**: percabangan probabilistik — mengarahkan pedestrian ke jalur berbeda
- **PedService**: tempat antri + dilayani — counter peminjaman / pengembalian
- **PedSink**: tempat pedestrian keluar dari simulasi

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMasuk` | TEMPORARY — untuk standalone |
| PedSelectOutput | `selectJenisLayanan` | Pilih pinjam (70%) atau kembali (30%) |
| PedGoTo | `goToRakBuku` | Jalan dari titik masuk ke area rak buku |
| PedWait | `wKelilingRak` | Browsing/keliling rak (uniform 2-5 menit) |
| PedGoTo | `goToCounterPinjam` | Jalan dari area rak ke counter peminjaman |
| PedSelectOutput | `selectJadiPinjam` | 90% jadi pinjam, 10% batal (berubah pikiran) |
| PedService | `srvPinjam` | Counter peminjaman (2 titik) |
| PedGoTo | `goToCounterKembali` | Jalan langsung ke counter pengembalian |
| PedService | `srvKembali` | Counter pengembalian (1 titik) |
| PedSink | `snkSelesai` | TEMPORARY — untuk standalone |

### 1.2 Markup

| Markup | Nama | Service Points | Antrean |
|---|---|---|---|
| Service with Lines | `svcPeminjaman` | 2 | 2 (shortest queue) |
| Service with Lines | `svcPengembalian` | 1 | 1 |
| TargetLine | `entryLine` | — | Garis masuk (standalone) |
| TargetLine | `exitLine` | — | Garis keluar (standalone) |
| Rectangular Node | `nodeRakBuku` | — | Area rak buku (target browsing) |
| Rectangular Node | `nodeCounterPinjam` | — | Area depan counter peminjaman |
| Rectangular Node | `nodeCounterKembali` | — | Area depan counter pengembalian |
| Path | `jalurKeRak` | — | Jalur dari entry ke rak buku |
| Path | `jalurRakKePinjam` | — | Jalur dari rak ke counter pinjam |
| Path | `jalurKeKembali` | — | Jalur dari entry ke counter kembali |

### 1.3 3D (opsional)

| Objek | Nama | Ukuran | Posisi (contoh) |
|---|---|---|---|
| Floor | `floor3D` | 20x15x0.2 | (10, 8, 0) |
| Meja counter pinjam | `mejaPinjam3D` | 3x1x1.2 | (14, 4) |
| Meja counter kembali | `mejaKembali3D` | 3x1x1.2 | (18, 4) |
| Rak buku 1 | `rakBukuPinjam3D_1` | 1x3x2 | (6, 8) |
| Rak buku 2 | `rakBukuPinjam3D_2` | 1x3x2 | (9, 8) |
| Rak buku 3 | `rakBukuPinjam3D_3` | 1x3x2 | (6, 12) |

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
| `waktuMulaiBrowsing` | double | `0` | Waktu mulai keliling rak |
| `durasiBrowsing` | double | `0` | Total waktu browsing rak |
| `jadiPinjam` | boolean | `true` | `true`=jadi pinjam, `false`=batal setelah browsing |

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
| `totalBrowsing` | int | 0 | Total yang browsing rak |
| `totalBatalPinjam` | int | 0 | Total yang batal pinjam setelah browsing |
| `totalDurasiBrowsing` | double | 0 | Akumulasi waktu browsing |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_PinjamKembali`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 2.1 Buat/ubah PengunjungPed

Jika sudah ada `PengunjungPed` dari modul lain:
- Buka diagram → **tambah 13 variable** sesuai tabel 1.4

Jika belum ada:
1. **Pedestrian Library** → drag **Pedestrian Type** → nama `PengunjungPed`.
2. Pilih animasi 3D.
3. Buka diagram → tambah 13 variable satu per satu.

> **Tips pemula:** Ada 13 variabel — pastikan semuanya ditambahkan. Nama variabel **case-sensitive**. Contoh: `isPeminjaman` (i kecil, P besar). Tiga variabel baru: `waktuMulaiBrowsing`, `durasiBrowsing`, `jadiPinjam`.

### 2.2 Tambah variabel Main

Buka Main → drag **Variable** 11 kali → isi sesuai tabel 1.5.

---

## 3. Layout 3D & Markup

### 3.1 3D

1. **3D Window** → `win3D`.
2. **Camera** → `camMain`.
3. **Box** → `floor3D`, skala 20x15x0.2. Posisi: (10, 8, 0).
4. **Box** → `mejaPinjam3D`, ukuran 3x1x1.2. Posisi: (14, 4).
5. **Box** → `mejaKembali3D`, ukuran 3x1x1.2. Posisi: (18, 4).
6. **Box** → `rakBukuPinjam3D_1`, ukuran 1x3x2. Posisi: (6, 8).
7. **Box** → `rakBukuPinjam3D_2`, ukuran 1x3x2. Posisi: (9, 8).
8. **Box** → `rakBukuPinjam3D_3`, ukuran 1x3x2. Posisi: (6, 12).

> **Tips pemula:** Rak buku 3D penting sebagai visual target browsing. Letakkan 3-4 rak berjajar seperti lorong perpustakaan asli.

### 3.2 Target lines (untuk standalone)

1. **Space Markup** → **Target line** → rename `entryLine`. Letakkan di kiri (x=2, y=4).
2. **Space Markup** → **Target line** → rename `exitLine`. Letakkan di kanan (x=20, y=4).

### 3.3 Nodes (area tujuan)

> **Kenapa pakai Rectangular Node?** Rectangular Node lebih cocok sebagai target PedGoTo karena mewakili **area** (bukan titik). Node ini langsung dikenali di dropdown Target tanpa perlu terkoneksi path dulu. Cocok untuk pemula.

1. **Space Markup** → **Rectangular Node** → rename `nodeRakBuku`.
   - Gambar persegi di area rak buku (contoh: x=5, y=7 → x=10, y=13).
   - Ini adalah area browsing — pedestrian akan berjalan ke area ini.
2. **Space Markup** → **Rectangular Node** → rename `nodeCounterPinjam`.
   - Gambar persegi di depan `mejaPinjam3D` (contoh: x=13, y=4 → x=16, y=7).
3. **Space Markup** → **Rectangular Node** → rename `nodeCounterKembali`.
   - Gambar persegi di depan `mejaKembali3D` (contoh: x=17, y=4 → x=20, y=7).

> **Tips pemula:** Setelah membuat Rectangular Node, langsung bisa dipilih di dropdown **Target** pada blok PedGoTo. Tidak seperti Point Node yang kadang tidak muncul jika belum terhubung path.

### 3.4 Paths (jalur pejalan kaki — opsional)

Path membuat gerakan pedestrian lebih natural (berjalan mengikuti jalur), tapi **tidak wajib** untuk PedGoTo kalau target-nya Rectangular Node. Jika ingin visual berjalan yang rapi:

1. **Space Markup** → **Path** → dari `entryLine` ke `nodeRakBuku` → rename `jalurKeRak`.
2. **Space Markup** → **Path** → dari `nodeRakBuku` ke `nodeCounterPinjam` → rename `jalurRakKePinjam`.
3. **Space Markup** → **Path** → dari `entryLine` ke `nodeCounterKembali` → rename `jalurKeKembali`.

> **Tips pemula:** Dengan Rectangular Node, PedGoTo tetap berfungsi **tanpa** Path — pedestrian akan berjalan lurus menuju area node. Path hanya dipakai kalau Anda ingin jalur spesifik (misal: belok mengikuti koridor).

### 3.5 svcPeminjaman (Service with Lines)

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcPeminjaman`.
3. **Number of services** = `2` (dua petugas counter).
4. **N of queues** = `2` (dua antrean — pedestrian pilih yang terpendek).
5. Letakkan service point di depan `mejaPinjam3D`, dekat `nodeCounterPinjam`.

### 3.6 svcPengembalian (Service with Lines)

1. **Service with Lines**.
2. Rename: `svcPengembalian`.
3. **Number of services** = `1` (satu petugas).
4. **N of queues** = `1`.
5. Letakkan service point di depan `mejaKembali3D`, dekat `nodeCounterKembali`.

---

## 4. Bangun Flowchart

### 4.1 Blok

Dari palette **Pedestrian Library**, drag blok berikut:

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` |
| PedSelectOutput | `selectJenisLayanan` |
| PedGoTo | `goToRakBuku` |
| PedWait | `wKelilingRak` |
| PedGoTo | `goToCounterPinjam` |
| PedSelectOutput | `selectJadiPinjam` |
| PedService | `srvPinjam` |
| PedGoTo | `goToCounterKembali` |
| PedService | `srvKembali` |
| PedSink | `snkSelesai` |

### 4.2 Koneksi

```
srcMasuk.out → selectJenisLayanan.in

=== JALUR PINJAM (70%) ===
selectJenisLayanan.out1 → goToRakBuku.in
goToRakBuku.out → wKelilingRak.in
wKelilingRak.out → goToCounterPinjam.in
goToCounterPinjam.out → selectJadiPinjam.in

selectJadiPinjam.out1 (90%, jadi pinjam) → srvPinjam.in
selectJadiPinjam.out2 (10%, batal) → snkSelesai.in

=== JALUR KEMBALI (30%) ===
selectJenisLayanan.out2 → goToCounterKembali.in
goToCounterKembali.out → srvKembali.in

=== OUTPUT ===
srvPinjam.out → snkSelesai.in
srvKembali.out → snkSelesai.in
```

> **Catatan:** `snkSelesai.in` menerima koneksi dari 3 sumber: `selectJadiPinjam.out2` (batal), `srvPinjam.out`, dan `srvKembali.out`. Di AnyLogic, satu input bisa menerima banyak koneksi.

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

### 5.2 `selectJenisLayanan` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | Centang |
| Output 1 | `0.70` (70% pinjam → ke rak buku dulu) |
| Output 2 | `0.30` (30% kembali → langsung ke counter) |

### 5.3 `goToRakBuku` (PedGoTo)

| Property | Value | Penjelasan |
|---|---|---|
| `Target` | `nodeRakBuku` | Rectangular Node area rak buku |

Mengarahkan pedestrian berjalan dari posisi saat ini (entryLine) menuju area rak buku untuk browsing.

### 5.4 `wKelilingRak` (PedWait)

| Property | Value | Penjelasan |
|---|---|---|
| `Waiting location` | `nodeRakBuku` | Diam di area rak buku |
| `End of delay` | `On delay time expiry` | Selesai setelah waktu habis |
| `Delay time` | `hitungWaktuBrowsing()` | Fungsi: uniform(2, 5) menit |

**Action On enter:**
```java
ped.waktuMulaiBrowsing = time();
totalBrowsing++;

traceln("BROWSING " + ped.idPed + " — keliling rak...");
```

**Action On exit:**
```java
ped.durasiBrowsing = time() - ped.waktuMulaiBrowsing;
totalDurasiBrowsing += ped.durasiBrowsing;

traceln("BROWSING_DONE " + ped.idPed + " | durasi=" + String.format("%.1f", ped.durasiBrowsing) + " mnt");
```

> **Penjelasan realistik:** Di perpustakaan nyata, pengunjung biasanya keliling rak dulu 2-5 menit sebelum memutuskan buku mana yang akan dipinjam. Ada yang langsung dapat, ada yang perlu banding-banding dulu.

### 5.5 `goToCounterPinjam` (PedGoTo)

| Property | Value | Penjelasan |
|---|---|---|
| `Target` | `nodeCounterPinjam` | Rectangular Node depan counter pinjam |

Setelah selesai browsing, pedestrian berjalan dari area rak ke counter peminjaman untuk antre.

### 5.6 `selectJadiPinjam` (PedSelectOutput)

| Property | Value | Penjelasan |
|---|---|---|
| `N outputs` | `2` | — |
| `Use probabilities` | Centang | — |
| Output 1 | `0.90` (90% jadi pinjam) | Buku ketemu, cocok → ke counter |
| Output 2 | `0.10` (10% batal pinjam) | Tidak jadi — buku tidak cocok, berubah pikiran, dsb |

> **Penjelasan:** Setelah browsing, tidak semua orang jadi meminjam. ~10% memutuskan tidak jadi — mungkin buku yang dicari tidak ada, atau sudah punya edisi yang sama. Ini menambah realisme simulasi.

### 5.7 `srvPinjam` (PedService)

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
ped.jadiPinjam = true;

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
    + " | scannerError=" + ped.scannerError
    + " | browsing=" + String.format("%.1f", ped.durasiBrowsing) + " mnt");
```

### 5.8 `goToCounterKembali` (PedGoTo)

| Property | Value | Penjelasan |
|---|---|---|
| `Target` | `nodeCounterKembali` | Rectangular Node depan counter kembali |

Mengarahkan pedestrian langsung ke counter pengembalian (tanpa browsing — karena mereka sudah punya buku untuk dikembalikan).

### 5.9 `srvKembali` (PedService)

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

### 5.10 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem) + " mnt"
    + " | " + (ped.isPeminjaman ? "PINJAM" : "KEMBALI")
    + (ped.jadiPinjam ? "" : " | BATAL"));
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

### 6.3 `hitungWaktuBrowsing`

Return type: `double`. **Tanpa parameter**.

```java
// Waktu browsing/keliling rak: 2-5 menit
// Simulasi pengunjung yang mencari-cari buku di rak
return uniform(2.0, 5.0);
```

**Contoh skenario browsing:**
- Mahasiswa cari buku algoritma: 2 menit (langsung ketemu di rak "Informatika")
- Mahasiswa cari 3 buku berbeda: 4.5 menit (harus ke 3 rak berbeda)
- Dosen cari referensi: 3 menit (lebih familiar dengan tata letak rak)

### 6.4 Fungsi dashboard

```java
// avgWaktuSistem (return: double)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;

// avgBukuPerTransaksi (return: double)
return totalPeminjaman == 0 ? 0 : (1.0 * totalBukuDipinjam / totalPeminjaman);

// errorScannerRate (return: double) — dalam persen
return totalPeminjaman == 0 ? 0 : (100.0 * totalErrorScanner / totalPeminjaman);

// rataRataDenda (return: double)
return totalPengembalian == 0 ? 0 : totalDenda / totalPengembalian;

// avgWaktuBrowsing (return: double) — rata-rata durasi keliling rak
return totalBrowsing == 0 ? 0 : totalDurasiBrowsing / totalBrowsing;

// tingkatBatalPinjam (return: double) — persentase yang batal setelah browsing
return totalBrowsing == 0 ? 0 : (100.0 * totalBatalPinjam / totalBrowsing);
```

### 6.5 Variabel tambahan di Main

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
// Text 2 — browsing
"Browsing rak: " + totalBrowsing + " | Rata-rata: " + String.format("%.1f", avgWaktuBrowsing()) + " mnt"
+ " | Batal: " + totalBatalPinjam + " (" + String.format("%.1f", tingkatBatalPinjam()) + "%)"
```

```java
// Text 3 — antrian
"Queue pinjam: " + srvPinjam.queueSize() + " (max " + maxQueuePeminjaman + ")"
+ " | Queue kembali: " + srvKembali.queueSize() + " (max " + maxQueuePengembalian + ")"
```

```java
// Text 4 — buku
"Buku dipinjam: " + totalBukuDipinjam + " | Error scanner: " + totalErrorScanner
+ " (" + String.format("%.1f", errorScannerRate()) + "%)"
```

```java
// Text 5 — denda
"Buku rusak: " + totalBukuRusak + " | Total denda: Rp " + String.format("%,.0f", totalDenda)
```

```java
// Text 6 — rata-rata
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## 8. Jalankan untuk Uji Coba

### Test 1: Coba 15 menit

1. **Stop time** = `15`.
2. Run.
3. Amati: pedestrian muncul → pilih pinjam/kembali → apakah yang pinjam jalan ke rak dulu? Apakah ada yang batal?

### Test 2: Test penuh 60 menit

1. **Stop time** = `60`.
2. Run.
3. Cek console: `ARRIVE`, `BROWSING`, `BROWSING_DONE`, `SERVICE ... PINJAM`, `SERVICE ... KEMBALI`, mungkin `DENDA`, `DONE`.

### Perkiraan hasil wajar

- Total pengunjung: ~25-30 orang (exponential 2.0 = 1 orang per 2 menit)
- ~70% pinjam (~17-21 orang) → semuanya browsing dulu
- ~30% kembali (~8-9 orang) → langsung ke counter kembali
- Yang browsing: ~10% batal = ~2 orang tidak jadi pinjam
- Rata-rata browsing: ~3.5 menit (tengah-tengah 2-5 menit)
- Error scanner: 5% dari ~15 peminjaman = ~1 kejadian
- Buku rusak: 10% dari ~8 pengembalian = ~1 buku
- Denda: tergantung keterlambatan

---

## 9. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → `selectTujuan.out1` colok ke `goToRakBuku.in` (Modul 7), `selectTujuan.out2` colok ke `goToCounterKembali.in` (Modul 7).
2. **Hapus** `selectJenisLayanan` → routing pinjam/kembali langsung dari `selectTujuan` skeleton. `selectTujuan.out1` = jalur pinjam, `selectTujuan.out2` = jalur kembali.
3. **Hapus** `snkSelesai` (temporary) → colok `selectJadiPinjam.out2` (batal), `srvPinjam.out`, dan `srvKembali.out` ke `wJalanKeluar.in` skeleton.

---

## 10. Troubleshooting

### Masalah: Node tidak muncul di dropdown Target PedGoTo

**Penyebab:** Point Node kadang tidak terdeteksi dropdown jika belum terhubung path. Ini bug kecil di beberapa versi AnyLogic.

**Solusi:** Ganti Point Node dengan **Rectangular Node** (Space Markup → Rectangular Node). Rectangular Node langsung muncul di dropdown Target tanpa perlu path — lebih stabil untuk pemula. Lihat section 3.3.

### Masalah: PedGoTo tidak bergerak — pedestrian diam di tempat

**Penyebab:** Target node tidak ditemukan atau salah tipe.

**Solusi:** Cek setiap PedGoTo → pastikan `Target` = Rectangular Node yang sudah dibuat. Coba klik ulang dropdown Target — Rectangular Node biasanya muncul dengan nama yang Anda berikan.

### Masalah: Browsing terlalu cepat atau terlalu lama

**Penyebab:** Range `uniform(2.0, 5.0)` tidak sesuai.

**Solusi:** Sesuaikan: untuk testing cepat pakai `uniform(0.5, 1.0)`, untuk realistis pakai `uniform(3.0, 8.0)`.

### Masalah: Semua orang batal pinjam (10% terasa banyak)

**Penyebab:** 10% bisa terasa sering jika jumlah pengunjung sedikit.

**Solusi:** Turunkan jadi `0.05` (5%) atau `0.03` (3%).

### Masalah: Error "Cannot convert from Agent"

**Penyebab:** Fungsi dipanggil dengan `agent` bukan `ped`.

**Solusi:** Cek semua PedService → "Delay time" harus pakai `ped`, contoh: `hitungWaktuServicePeminjaman(ped)`.

### Masalah: Denda selalu Rp 0

**Penyebab:** Waktu pinjam tidak cukup lama.

**Solusi:** Di `srvKembali.onBeginService`, pastikan `hariLalu = uniform(1.0, 10.0)` dikali `24 * 60`.

---

## 11. Fallback / Alternatif

### Jika tidak ingin pakai Path dan Point Node

Ganti `goToRakBuku` (PedGoTo) dengan `wKeRak` (PedWait dengan delay pendek seperti 0.3 mnt). Pedestrian tidak benar-benar berjalan ke node, tapi diam sebentar menunggu. Kurang realistis secara visual tapi tetap berfungsi.

### Jika ingin browsing lebih lama

Ubah `hitungWaktuBrowsing()`: `return uniform(5.0, 15.0);` — untuk skenario perpustakaan besar dengan banyak rak.

### Jika ingin semua orang jadi pinjam (tanpa batal)

Hapus `selectJadiPinjam`. Colok `goToCounterPinjam.out` langsung ke `srvPinjam.in`.

### Jika ingin error scanner lebih sering

Ubah `uniform(0, 1) < 0.05` jadi `uniform(0, 1) < 0.10` (10%).

---

## 12. Mode Presentasi

- **Stop time:** 30-40 menit
- **Fokus kamera 3D:** area rak buku + kedua counter
- **Speed slider runtime:** geser ke kiri agar gerakan browsing terlihat natural
- Bisa tunjukkin: "Lihat, mahasiswa jalan ke rak dulu, keliling rak 3 menit, baru ke counter"
- Bisa tunjukkin: "Ada 10% yang batal pinjam setelah browsing — mungkin bukunya tidak cocok"

---

## 13. Checklist Final

- [ ] **PengunjungPed**: 13 variabel (10 lama + 3 baru: `waktuMulaiBrowsing`, `durasiBrowsing`, `jadiPinjam`)
- [ ] **Variabel Main**: 11 variabel statistik (tabel 1.5) + 3 tambahan (seqPed, totalSelesai, totalWaktuSistem)
- [ ] **Markup**: `svcPeminjaman` (2 services, 2 queues), `svcPengembalian` (1 service, 1 queue)
- [ ] **Markup**: 3 point nodes (`nodeRakBuku`, `nodeCounterPinjam`, `nodeCounterKembali`) + 3 paths
- [ ] **Flowchart**: 10 blok — srcMasuk → selectJenisLayanan → goToRakBuku → wKelilingRak → goToCounterPinjam → selectJadiPinjam → srvPinjam / srvKembali → snkSelesai
- [ ] **selectJenisLayanan**: 70% pinjam, 30% kembali
- [ ] **selectJadiPinjam**: 90% jadi pinjam, 10% batal
- [ ] **wKelilingRak.Delay time** = `hitungWaktuBrowsing()`
- [ ] **srvPinjam.Services** = `svcPeminjaman`, Delay = `hitungWaktuServicePeminjaman(ped)`
- [ ] **srvKembali.Services** = `svcPengembalian`, Delay = `hitungWaktuServicePengembalian(ped)`
- [ ] **Fungsi**: `hitungWaktuServicePeminjaman`, `hitungWaktuServicePengembalian`, `hitungWaktuBrowsing`
- [ ] **Fungsi dashboard**: 5 fungsi statistik termasuk `avgWaktuBrowsing`, `tingkatBatalPinjam`
- [ ] **Dashboard**: 6 text dinamis termasuk statistik browsing
- [ ] **Test 60 menit**: browsing → pinjam/kembali → console log lengkap
