# Modul 2: Kedatangan & Kepergian — SKELETON UTAMA
## Scan KTM, Entry/Exit perpustakaan, dan Hub Integration untuk semua modul

**Ini adalah SKELETON (kerangka utama).** Semua modul lain akan mencolok ke modul ini.

**Satuan waktu:** minute

---

## 1. Hasil Akhir

Modul ini adalah pusat dari seluruh simulasi perpustakaan. Fungsinya:

1. Menerima **pejalan kaki** (langsung ke perpus) + **orang dari parkir** (Modul 1)
2. Antri **scan KTM** di pintu masuk
3. Memilih tujuan: pinjam/kembali, cari duduk, toilet, fotokopi, atau langsung keluar
4. Setelah selesai: jika naik kendaraan → balik ke parkir, jika jalan kaki → keluar

**Flowchart standalone:**

```
srcJalanKaki → srvScanKTM → selectTujuan (6 output)
  ├── out1: srvPinjam (25%)       → masuk Modul 6
  ├── out2: srvKembali (15%)      → masuk Modul 6
  ├── out3: srvCariDuduk (25%)    → masuk Modul 3
  ├── out4: srvToilet (10%)       → masuk Modul 4
  ├── out5: srvFotokopi (10%)     → masuk Modul 5
  └── out6: srvLangsungKeluar (15%) → wJalanKeluar → selesai
                      ↓
                selectPulang
                  ├── ya (isParkir?) → wJalanBalikParkir → snkParkir
                  └── tidak → snkSelesai
```

---

## 2. Komponen yang Dipakai

### Blok flowchart

| Blok | Nama | Fungsi |
|---|---|---|
| PedSource | `srcJalanKaki` | Pejalan kaki langsung ke perpus |
| PedService | `srvScanKTM` | Antri scan kartu tanda pengenal |
| PedSelectOutput | `selectTujuan` | Hub: 6 output ke 5 layanan + 1 langsung keluar |
| PedSelectOutput | `selectPulang` | Routing: parkir atau selesai |
| PedWait | `wJalanKeluar` | Jalan ke garis keluar |
| PedSink | `snkSelesai` | Selesai (jalan kaki) |
| PedWait | `wJalanBalikParkir` | Jalan balik ke parkir (untuk yang naik kendaraan) |
| PedSink | `snkParkir` | Selesai (parkir) — TEMPORARY |

### Markup

| Markup | Nama | Fungsi |
|---|---|---|
| TargetLine | `entryLine` | Garis masuk perpustakaan |
| TargetLine | `exitLine` | Garis keluar perpustakaan |
| Service with Lines | `svcScanKTM` | 2 mesin scan KTM + antrean |

### 3D

| Objek | Nama | Posisi contoh |
|---|---|---|
| 3D Window | `win3D` | — |
| Camera | `camMain` | (22, -16, 14) |
| Floor (Box) | `floor3D` | (15, 10, 0) skala 30x20x0.2 |
| Meja resepsionis | `mejaResepsionis3D` | (8, 10) |
| Strip/Pintu masuk | `pintuMasuk3D` | Polygon atau box |

### Agent Type

| Agent | Nama |
|---|---|
| Pedestrian Type | `PengunjungPed` |

---

## 3. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_Skeleton`.
4. **Time units**: `minute`.
5. Klik **Finish**.

## 4. Buat Pedestrian Type (PengunjungPed)

Semua modul akan menggunakan **tipe yang sama**. Variabel-variabel di sini adalah **core** (dipakai semua modul). Modul lain akan **menambah variabel baru** ke tipe ini.

1. Dari palette **Pedestrian Library**, drag **Pedestrian Type** ke canvas.
2. Wizard → nama: `PengunjungPed`.
3. Pilih animasi 3D orang (bebas, yang penting kelihatan).
4. Klik **Finish**.

Buka diagram `PengunjungPed`, lalu **tambah Variable** berikut:

| Nama | Type | Initial value | Dipakai oleh |
|---|---|---|---|
| `idPed` | String | `""` | Semua modul |
| `tMasuk` | double | `0` | Semua modul |
| `isDosen` | boolean | `false` | Semua modul |
| `isParkir` | boolean | `false` | Modul 1 (Parkir) |
| `noktp` | String | `""` | Modul 2 (Scan KTM) |
| `isValidKTM` | boolean | `false` | Modul 2 (Scan KTM) |
| `tujuanLayanan` | String | `""` | Semua modul (isi di selectTujuan) |

> **Kenapa hanya ini?** Setiap modul akan menambah variabelnya sendiri nanti. Saat integrasi, semua variabel digabung ke satu `PengunjungPed`.

---

## 5. Siapkan Layout 3D

1. Drag **3D Window** → rename `win3D`.
2. Drag **Camera** → rename `camMain`. Atur posisi:
   - X = 22, Y = -16, Z = 14
3. Drag **Box** → rename `floor3D`, skala 30x20x0.2, posisi (15, 10, 0).
4. (Opsional) Drag **Box** → rename `mejaResepsionis3D`, posisi (8, 10).

---

## 6. Buat Markup Pedestrian

### 6.1 Garis masuk (entryLine)

1. Dari **Space Markup** → **Target line**.
2. Rename: `entryLine`.
3. Letakkan di pojok kiri bawah (misal: x=2, y=3 sampai x=8, y=3).

### 6.2 Garis keluar (exitLine)

1. **Target line** lagi.
2. Rename: `exitLine`.
3. Letakkan di pojok kanan atas (misal: x=22, y=18 sampai x=28, y=18).

### 6.3 Service with Lines — Scan KTM

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcScanKTM`.
3. Letakkan di dekat pintu masuk (misal: dekat `entryLine`).
4. Properties: **Number of services** = `2`, **N of queues** = `2`.
5. Tarik queue line dari `entryLine` menuju service point.

---

## 7. Bangun Flowchart

### 7.1 Blok

| Blok | Rename |
|---|---|
| PedSource | `srcJalanKaki` |
| PedService | `srvScanKTM` |
| PedSelectOutput | `selectTujuan` |
| PedWait | `wJalanKeluar` |
| PedSelectOutput | `selectPulang` |
| PedWait | `wJalanBalikParkir` (ini akan colok ke Modul 1 nanti) |
| PedSink | `snkSelesai` |
| PedSink | `snkParkir` (TEMPORARY) |

### 7.2 Koneksi

```
srcJalanKaki.out → srvScanKTM.in

srvScanKTM.out → selectTujuan.in

selectTujuan.out1 (25% layanan) → [Modul 6: srvPinjam]
selectTujuan.out2 (15% layanan) → [Modul 6: srvKembali]
selectTujuan.out3 (25% layanan) → [Modul 3: srvCariDuduk]
selectTujuan.out4 (10% layanan) → [Modul 4: srvToilet]
selectTujuan.out5 (10% layanan) → [Modul 5: srvFotokopi]
selectTujuan.out6 (15% langsung) → wJalanKeluar.in

wJalanKeluar.out → selectPulang.in

selectPulang.out1 (isParkir = true) → wJalanBalikParkir.in → snkParkir.in
selectPulang.out2 (isParkir = false) → snkSelesai.in
```

> **Catatan penting:** `selectTujuan.out1` sampai `out5` akan dikoneksikan **setelah modul tujuan sudah jadi**. Saat standalone, output ini bisa dibiarkan tidak terkoneksi (AnyLogic akan warning tapi tetap jalan) atau dikoneksikan langsung ke `wJalanKeluar.in` sementara.

---

## 8. Konfigurasi Detail Setiap Blok

### 8.1 `srcJalanKaki` (PedSource)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` |
| Arrive according to | `Interarrival time` |
| Interarrival time | `getInterarrivalTime()` |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "J-" + seqPed;
ped.tMasuk = time();
ped.isParkir = false; // jalan kaki, bukan parkir
ped.isDosen = uniform(0, 1) < 0.2; // 20% dosen

// Generate nomor KTM
ped.noktp = "KTM-" + (1000 + seqPed);
ped.isValidKTM = true; // default valid

// 3% KTM invalid
if (uniform(0, 1) < 0.03) {
    ped.isValidKTM = false;
}

// Statistik
if (ped.isDosen) {
    totalDosen++;
} else {
    totalMahasiswa++;
}

traceln("ARRIVE " + ped.idPed
    + " | " + (ped.isDosen ? "DOSEN" : "MHS")
    + " | valid=" + ped.isValidKTM
    + " | noktp=" + ped.noktp);
```

### 8.2 `srvScanKTM` (PedService)

| Property | Value |
|---|---|
| Services | `svcScanKTM` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuScanKTM(ped)` |
| Recovery delay | `0` |

**Action On end service:**
```java
if (!ped.isValidKTM) {
    traceln("GAGAL " + ped.idPed + " KTM tidak valid - Ditolak");
    // Pedestrian akan diarahkan ke out6 (langsung keluar)
    ped.tujuanLayanan = "DITOLAK";
}
```

### 8.3 `selectTujuan` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `6` |
| Use probabilities | Centang |
| Output 1 | `0.25` (Pinjam - Modul 6) |
| Output 2 | `0.15` (Kembali - Modul 6) |
| Output 3 | `0.25` (Cari Duduk - Modul 3) |
| Output 4 | `0.10` (Toilet - Modul 4) |
| Output 5 | `0.10` (Fotokopi - Modul 5) |
| Output 6 | `0.15` (Langsung keluar / ditolak) |

Penjelasan: Probabilitas total = 0.25+0.15+0.25+0.10+0.10+0.15 = 1.0 (100%).

### 8.4 `wJalanKeluar` (PedWait)

| Property | Value |
|---|---|
| Waiting location | `Target line` |
| Target line | `exitLine` |
| Delay ends | `On delay time expiry` |
| Delay time | `0.2` |

### 8.5 `selectPulang` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `2` |
| Use probabilities | Tidak centang (gunakan kondisi) |
| Output 1 condition | `ped.isParkir == true` |
| Output 2 condition | `ped.isParkir == false` |

> **Penjelasan:** Jika `ped.isParkir == true`, berarti dia naik kendaraan → arahkan ke `wJalanBalikParkir` (menuju Modul 1). Jika false → ke `snkSelesai` (langsung keluar).

### 8.6 `snkSelesai` (PedSink)

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem)
    + " | tipe=" + (ped.isDosen ? "DOSEN" : "MHS"));
```

### 8.7 `wJalanBalikParkir` dan `snkParkir` (TEMPORARY)

`wJalanBalikParkir` adalah PedWait yang mengarah ke target line area parkir.

| Property | Value |
|---|---|
| Waiting location | `Target line` |
| Target line | `entryLine` (sementara, akan diganti saat integrasi) |
| Delay time | `0.5` |

`snkParkir` → PedSink biasa, action on enter kosong (sementara).

---

## 9. Variabel di Main

### 9.1 Variabel

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |
| `totalMahasiswa` | int | 0 |
| `totalDosen` | int | 0 |

### 9.2 Fungsi `getInterarrivalTime`

Return type: `double`. Menciptakan peak hour seperti tutorial sebelumnya.

```java
double t = time();
if (t < 20) {
    return exponential(2.0); // sepi
} else if (t < 50) {
    return exponential(5.0); // peak: 5 orang per menit
} else {
    return exponential(3.0); // normal
}
```

### 9.3 Fungsi `hitungWaktuScanKTM`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double dasar;

if (ped.isDosen) {
    dasar = uniform(0.2, 0.5); // dosen lebih cepat
} else {
    dasar = uniform(0.3, 1.0); // mahasiswa 0.3-1 menit
}

// Jika KTM invalid, waktu scan lebih lama
if (!ped.isValidKTM) {
    dasar += uniform(0.5, 1.5); // verifikasi manual
    traceln("WARNING: KTM invalid " + ped.idPed);
}

return dasar;
```

### 9.4 Fungsi `avgWaktuSistem`

Return type: `double`.

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

---

## 10. Dashboard Standalone

Tambahkan Text dinamis di Main:

```
"Total pengunjung: " + totalSelesai
```

```
"Mahasiswa: " + totalMahasiswa + " | Dosen: " + totalDosen
```

```
"Antrian scan KTM: " + srvScanKTM.queueSize()
```

```
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

```
"Pejalan kaki: " + totalSelesai + " | Parkir: " + totalParkir
```

> **Catatan:** `totalParkir` akan terisi setelah Modul 1 (Parkir) diintegrasikan.

---

## 11. Jalankan untuk Uji Coba

1. **Stop time** = 30 menit.
2. **Untuk sementara**, hubungkan `selectTujuan.out1` sampai `out5` langsung ke `wJalanKeluar.in` (karena modul lain belum ada).
3. Run.
4. Harus terlihat: orang muncul di `entryLine`, antri di `svcScanKTM`, lalu jalan ke `exitLine`.
5. Cek console: `ARRIVE J-1 | MHS | valid=true`, `DONE J-1 | tSistem=...`.

---

## 12. Yang Perlu Diubah Saat Integrasi

1. `selectTujuan.out1` sampai `out5` → colok ke PedService modul masing-masing.
2. `wJalanBalikParkir.out` → colok ke `jalanKeParkiran.in` (PedGoTo dari Modul 1).
3. `snkParkir` dihapus (ganti dengan Modul 1 flow).s
4. Mesin scan KTM di 3D bisa dipindah ke posisi yang lebih sesuai dengan layout gabungan.

---

## 13. Troubleshooting

### Masalah: Pedestrian numpuk di entryLine

**Penyebab:** `srvScanKTM.Services` belum diisi `svcScanKTM`.

**Solusi:** Cek properties `srvScanKTM` → `Services` = `svcScanKTM`.

### Masalah: Semua orang langsung keluar

**Penyebab:** `selectTujuan.out1`-`out5` tidak terkoneksi → error atau warning. Saat standalone, memang harus dikoneksikan langsung ke `wJalanKeluar`.

**Solusi:** Untuk testing, koneksikan semua output langsung ke `wJalanKeluar.in`.

### Masalah: selectPulang tidak bekerja

**Penyebab:** `selectPulang` menggunakan kondisi, bukan probabilitas.

**Solusi:** Centang **Use conditions** (bukan Use probabilities). Atur:
- Output 1 condition: `ped.isParkir == true`
- Output 2 condition: `ped.isParkir == false`

---

## 14. Checklist Final

- [ ] `PengunjungPed` sudah dengan variabel core (idPed, tMasuk, isDosen, isParkir, noktp, isValidKTM, tujuanLayanan)
- [ ] `entryLine`, `exitLine`, `svcScanKTM` sudah dibuat
- [ ] `srcJalanKaki` → `srvScanKTM` → `selectTujuan` → `wJalanKeluar` → `selectPulang` terhubung
- [ ] `selectTujuan` 6 output dengan probabilitas
- [ ] `selectPulang` 2 output dengan kondisi `isParkir`
- [ ] `getInterarrivalTime()` berfungsi dengan peak hour
- [ ] `hitungWaktuScanKTM(ped)` membedakan dosen/mahasiswa dan KTM invalid
- [ ] Dashboard memperbarui angka saat run
- [ ] Test standalone berjalan 30 menit tanpa error
