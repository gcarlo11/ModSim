# Modul 2: Kedatangan & Kepergian — SKELETON UTAMA
## Scan KTM, Entry/Exit Perpustakaan, dan Hub Integration untuk Semua Modul

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

> ⚠️ **INi adalah SKELETON (kerangka utama).** Semua modul lain akan mencolok ke modul ini. Kerjakan modul ini dengan teliti karena menjadi fondasi integrasi.

---

## Ringkasan Hasil Akhir

Modul ini adalah **pusat simulasi perpustakaan**. Fungsinya:

1. Menerima **pejalan kaki** (langsung ke perpus) + **orang dari parkir** (Modul 1)
2. Antri **scan KTM** di pintu masuk
3. Memilih tujuan: pinjam buku, kembali buku, cari tempat duduk, toilet, fotokopi, atau langsung keluar
4. Setelah selesai: jika naik kendaraan → balik ke parkir, jika jalan kaki → keluar

**Flowchart standalone:**

```
srcJalanKaki → srvScanKTM (antri scan KTM)
                        ↓
                 selectTujuan (6 output)
    ├── out1 (25%) → srvPinjam           (Modul 7)
    ├── out2 (15%) → srvKembali           (Modul 7)
    ├── out3 (25%) → srvCariDuduk         (Modul 4)
    ├── out4 (10%) → selectGenderToilet   (Modul 5)
    ├── out5 (10%) → srvFotokopi          (Modul 6)
    └── out6 (15%) → wJalanKeluar (langsung keluar)
                        ↓
                 selectPulang (isParkir?)
    ├── ya (true)  → wJalanBalikParkir → snkParkir
    └── tidak (false) → snkSelesai (keluar)
```

**Konsep cepat untuk pemula:**
- **PedSource**: tempat keluarnya orang — di sini pejalan kaki yang datang ke perpus
- **PedService**: tempat antri dan dilayani — di sini antri scan KTM
- **PedSelectOutput**: percabangan — mengarahkan orang ke tujuan berdasarkan probabilitas atau kondisi
- **PedWait**: tempat orang menunggu — di sini menunggu jalan keluar
- **PedSink**: tempat orang selesai dan keluar dari simulasi

---

## 1. Komponen yang Dipakai

### 1.1 Blok flowchart

| Blok | Nama | Fungsi |
|---|---|---|
| PedSource | `srcJalanKaki` | Pejalan kaki langsung ke perpus |
| PedService | `srvScanKTM` | Antri scan kartu tanda pengenal |
| PedSelectOutput | `selectTujuan` | **Hub utama**: 6 output ke 5 layanan + 1 langsung keluar |
| PedWait | `wJalanKeluar` | Jalan ke garis keluar |
| PedSelectOutput | `selectPulang` | Routing: balik parkir atau selesai |
| PedWait | `wJalanBalikParkir` | Jalan balik ke parkir (yang naik kendaraan) |
| PedSink | `snkSelesai` | Selesai (jalan kaki) |
| PedSink | `snkParkir` | Selesai (parkir) — TEMPORARY |

### 1.2 Markup

| Markup | Nama | Fungsi |
|---|---|---|
| TargetLine | `entryLine` | Garis masuk perpustakaan |
| TargetLine | `exitLine` | Garis keluar perpustakaan |
| Service with Lines | `svcScanKTM` | 2 mesin scan KTM + antrean |

### 1.3 3D (disediakan, tapi bisa ditambah sendiri)

| Objek | Nama | Posisi contoh |
|---|---|---|
| 3D Window | `win3D` | — |
| Camera | `camMain` | Sesuaikan agar lobby perpus terlihat |
| Floor (Box) | `floor3D` | Skala 30x20x0.2, posisi (15,10,0) |
| Meja resepsionis | `mejaResepsionis3D` | Dekat entryLine |

### 1.4 Agent Type

**Wajib:** `PengunjungPed` — semua modul WAJIB memakai tipe ini agar bisa diintegrasikan.

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Isi **Model name**: `Perpustakaan3D_Skeleton`.
4. **Time units**: pilih `minute`.
5. Klik **Finish**.

Setelah ini, Anda akan melihat agent utama bernama `Main`.

---

## 3. Buat Pedestrian Type (PengunjungPed)

**Ini WAJIB.** Semua modul akan memakai tipe pedestrian yang SAMA.

1. Di palette **Pedestrian Library**, drag **Pedestrian Type** ke canvas Main. (Atau di panel Projects, klik kanan model → **New → Agent Type** → pilih template `Pedestrian`.)
2. Akan muncul wizard. Isi **Name**: `PengunjungPed`.
3. Pilih animasi 3D orang (bebas — "Man" atau "Woman" atau "Simple Man" — yang penting ada).
4. Klik **Finish**.

Sekarang buka diagram `PengunjungPed` (klik 2x di panel Projects). Dari palette **Agent**, drag **Variable** ke canvas. Tambahkan satu per satu:

| Nama | Type | Initial value | Dipakai oleh | Penjelasan |
|---|---|---|---|---|
| `idPed` | String | `""` | Semua modul | ID unik: "J-1", "Mobil-1", dll |
| `tMasuk` | double | `0` | Semua modul | Waktu masuk sistem (untuk hitung rata-rata) |
| `isDosen` | boolean | `false` | Semua modul | `true` = dosen, `false` = mahasiswa |
| `isParkir` | boolean | `false` | Modul 1 | `true` = naik kendaraan, `false` = jalan kaki |
| `noktp` | String | `""` | Modul 2 | Nomor KTM/KTP untuk scan |
| `isValidKTM` | boolean | `false` | Modul 2 | `true` = KTM valid, `false` = ditolak |
| `tujuanLayanan` | String | `""` | Semua modul | Catatan tujuan: "PINJAM", "KEMBALI", dll |

> **Tips pemula:** Variabel di PengunjungPed bersifat **per-instance** — setiap orang punya salinan nilai sendiri. `idPed` si J-1 berbeda dengan `idPed` si J-2.

---

## 4. Siapkan Layout 3D

### 4.1 3D Window

1. Dari palette **Presentation**, drag **3D Window** ke canvas Main.
2. Rename: `win3D`.
3. Perbesar ukurannya agar scene 3D terlihat jelas saat run.

### 4.2 Camera

1. Dari palette **Presentation**, drag **Camera** ke canvas Main.
2. Rename: `camMain`.
3. Di Properties, atur posisi kamera. Contoh:
   - X = 22, Y = -16, Z = 14

### 4.3 Floor

1. Dari palette **3D Objects**, drag **Box** ke canvas Main.
2. Rename: `floor3D`.
3. Properties: **Size** → X = `30`, Y = `20`, Z = `0.2`.
4. Posisi (X, Y): `(15, 10)`.

### 4.4 Meja resepsionis (opsional)

1. **Box** lain → rename `mejaResepsionis
`.
2. Ukuran: 3x1x1.2. Posisi: (10, 3).

---

## 5. Buat Markup Pedestrian

### 5.1 entryLine (Garis Masuk)

1. Dari palette **Space Markup**, drag **Target line** ke canvas.
2. Klik di canvas untuk menentukan titik awal, klik lagi untuk titik akhir.
3. Rename: `entryLine`.
4. Letakkan di pojok kiri bawah (contoh: x1=2,y1=3 → x2=8,y2=3).

> **Tips pemula:** Target line adalah garis imajiner. Saat pedestrian menginjak garis ini, trigger tertentu terjadi. Untuk PedSource, pedestrian muncul di garis ini.

### 5.2 exitLine (Garis Keluar)

1. **Target line** lagi.
2. Rename: `exitLine`.
3. Letakkan di pojok kanan atas (contoh: x1=22,y1=18 → x2=28,y2=18).

### 5.3 svcScanKTM (Service with Lines)

Ini adalah area antrian + service point untuk scan KTM.

1. Dari **Space Markup**, drag **Service with Lines** ke canvas.
2. Rename: `svcScanKTM`.
3. **Properties → General**:
   - **Number of services** = `2` (2 mesin scan).
   - **N of queues** = `2` (2 antrean, pilih yang terpendek).
4. Atur posisi: letakkan queue line dari `entryLine` menuju service point.
5. Service point taruh di dekat pintu masuk (contoh: antara entryLine dan floor).

**Yang terjadi di runtime:**
- Pedestrian datang → masuk ke antrean terpendek dari 2 antrean
- Jika salah satu mesin kosong → langsung dilayani
- Jika semua mesin penuh → antri sampai ada yang kosong

---

## 6. Bangun Flowchart

### 6.1 Cara drag dan rename blok

Dari palette **Pedestrian Library**, drag blok berikut ke canvas Main:

| Blok di Palette | Rename menjadi |
|---|---|
| PedSource | `srcJalanKaki` |
| PedService | `srvScanKTM` |
| PedSelectOutput | `selectTujuan` |
| PedWait | `wJalanKeluar` |
| PedSelectOutput | `selectPulang` |
| PedWait | `wJalanBalikParkir` |
| PedSink | `snkSelesai` |
| PedSink | `snkParkir` |

**Cara rename:** Klik blok → di Properties panel bawah, ganti field **Name**.

### 6.2 Koneksi flowchart

Hubungkan port (segitiga) dengan cara drag dari port out ke port in:

```
srcJalanKaki.out → srvScanKTM.in

srvScanKTM.out → selectTujuan.in

selectTujuan.out1 (25%, PINJAM) → [nanti colok ke Modul 7]
selectTujuan.out2 (15%, KEMBALI) → [nanti colok ke Modul 7]
selectTujuan.out3 (25%, CARI DUDUK) → [nanti colok ke Modul 4]
selectTujuan.out4 (10%, TOILET) → [nanti colok ke Modul 5]
selectTujuan.out5 (10%, FOTOKOPI) → [nanti colok ke Modul 6]
selectTujuan.out6 (15%, LANGSUNG) → wJalanKeluar.in

wJalanKeluar.out → selectPulang.in

selectPulang.out1 (isParkir = true) → wJalanBalikParkir.in → snkParkir.in
selectPulang.out2 (isParkir = false) → snkSelesai.in
```

> **Catatan penting:** Untuk testing standalone, hubungkan `selectTujuan.out1` sampai `out5` SEMENTARA ke `wJalanKeluar.in`. Nanti saat integrasi, koneksi ini akan diganti ke modul masing-masing.

---

## 7. Konfigurasi Detail Setiap Blok

### 7.1 `srcJalanKaki` (PedSource)

Klik blok `srcJalanKaki`. Atur Properties:

| Property | Nilai | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di target line |
| `Target line` | `entryLine` | Garis masuk perpus |
| `Arrive according to` | `Interarrival time` | Diatur waktu kedatangan |
| `Interarrival time` | `getInterarrivalTime()` | Fungsi peak hour (naik-turun) |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian |

**Action On exit** (tab Actions → On exit):

```java
seqPed++;
ped.idPed = "J-" + seqPed;
ped.tMasuk = time();
ped.isParkir = false; // jalan kaki, bukan dari parkir
ped.isDosen = uniform(0, 1) < 0.2; // 20% dosen

// Generate nomor KTM
ped.noktp = "KTM-" + (1000 + seqPed);
ped.isValidKTM = true; // default valid

// 3% KTM invalid — kartu rusak/tidak terbaca
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

### 7.2 `srvScanKTM` (PedService)

| Property | Nilai | Penjelasan |
|---|---|---|
| `Services` | `svcScanKTM` | Pilih markup Service with Lines |
| `Queue choice policy` | `Shortest queue` | Ambil antrean terpendek |
| `Delay time` | `hitungWaktuScanKTM(ped)` | Panggil fungsi waktu scan |
| `Recovery delay` | `0` | Tidak pakai recovery |

**Action On end service:**
```java
if (!ped.isValidKTM) {
    traceln("GAGAL " + ped.idPed + " KTM tidak valid - Ditolak");
    ped.tujuanLayanan = "DITOLAK";
}
```

### 7.3 `selectTujuan` (PedSelectOutput)

**Ini adalah blok paling penting** — menentukan ke mana orang pergi.

| Property | Nilai | Penjelasan |
|---|---|---|
| `N outputs` | `6` | 6 jalur tujuan |
| `Use probabilities` | **Centang** | Pakai probabilitas |
| Output 1 | `0.25` | 25% → PINJAM (Modul 7) |
| Output 2 | `0.15` | 15% → KEMBALI (Modul 7) |
| Output 3 | `0.25` | 25% → CARI DUDUK (Modul 4) |
| Output 4 | `0.10` | 10% → TOILET (Modul 5) |
| Output 5 | `0.10` | 10% → FOTOKOPI (Modul 6) |
| Output 6 | `0.15` | 15% → LANGSUNG KELUAR |

> **Verifikasi:** 0.25 + 0.15 + 0.25 + 0.10 + 0.10 + 0.15 = **1.0 (100%)**. Jika kurang atau lebih dari 1.0, AnyLogic akan kasih error.

**Action (isi per port di tab Actions → On exit port):**

> ⚠️ **PENTING:** Jangan pakai kode yang sama untuk semua port. Isi **masing-masing On exit port** secara terpisah.

- **On exit port 1:**
```java
ped.tujuanLayanan = "PINJAM";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 2:**
```java
ped.tujuanLayanan = "KEMBALI";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 3:**
```java
ped.tujuanLayanan = "CARIDUDUK";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 4:**
```java
ped.tujuanLayanan = "TOILET";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 5:**
```java
ped.tujuanLayanan = "FOTOKOPI";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

- **On exit port 6:**
```java
ped.tujuanLayanan = "LANGSUNG";
traceln("TUJUAN " + ped.idPed + " → " + ped.tujuanLayanan);
```

### 7.4 `wJalanKeluar` (PedWait)

| Property | Nilai |
|---|---|
| `Waiting location` | `Target line` |
| `Target line` | `exitLine` |
| `End of delay` | `On delay time expiry` |
| `Delay time` | `0.2` (sedikit waktu untuk animasi jalan) |

### 7.5 `selectPulang` (PedSelectOutput)

**GUNAKAN KONDISI, BUKAN PROBABILITAS!**

| Property | Nilai |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | **JANGAN centang** |
| `Use conditions` | **Centang** |
| Output 1 condition | `ped.isParkir == true` |
| Output 2 condition | `ped.isParkir == false` |

> **Tips pemula:** `isParkir == true` berarti orang itu datang dengan kendaraan (mobil/motor). Mereka harus balik ke parkir. `isParkir == false` berarti jalan kaki → langsung keluar.

### 7.6 `snkSelesai` (PedSink)

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + String.format("%.2f", tSistem)
    + " | tipe=" + (ped.isDosen ? "DOSEN" : "MHS")
    + " | tujuan=" + ped.tujuanLayanan);
```

### 7.7 `wJalanBalikParkir` (PedWait) — TEMPORARY

| Property | Nilai |
|---|---|
| `Waiting location` | `Target line` |
| `Target line` | `entryLine` (sementara — akan diarahkan ke parkir saat integrasi) |
| `Delay time` | `0.5` |

### 7.8 `snkParkir` (PedSink) — TEMPORARY

Buat PedSink biasa. Action on enter kosong.

---

## 8. Variabel dan Fungsi di Main

### 8.1 Variabel

Buka Main. Dari palette **Agent**, drag **Variable** ke canvas:

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `seqPed` | int | 0 | Counter ID |
| `totalSelesai` | int | 0 | Total selesai |
| `totalWaktuSistem` | double | 0 | Akumulasi waktu |
| `totalMahasiswa` | int | 0 | Total mahasiswa |
| `totalDosen` | int | 0 | Total dosen |

### 8.2 Fungsi `getInterarrivalTime`

Return type: `double`. Tanpa parameter.

```java
// Peak hour simulation:
// 0-20 menit: sepi (exponential 2.0 = 1 orang per 2 menit)
// 20-50 menit: PEAK (exponential 5.0 = 5 orang per menit!)
// 50+ menit: normal (exponential 3.0 = 1 orang per ~20 detik)
double t = time();
if (t < 20) {
    return exponential(2.0);
} else if (t < 50) {
    return exponential(5.0);
} else {
    return exponential(3.0);
}
```

### 8.3 Fungsi `hitungWaktuScanKTM`

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
double dasar;

if (ped.isDosen) {
    // Dosen lebih cepat scan-nya (0.2-0.5 menit)
    dasar = uniform(0.2, 0.5);
} else {
    // Mahasiswa: 0.3-1 menit
    dasar = uniform(0.3, 1.0);
}

// Jika KTM invalid, perlu verifikasi manual
if (!ped.isValidKTM) {
    dasar += uniform(0.5, 1.5);
    traceln("WARNING: KTM invalid " + ped.idPed + " — verifikasi manual " + String.format("%.1f", dasar) + " mnt");
}

return dasar;
```

### 8.4 Fungsi `avgWaktuSistem`

Return type: `double`.

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

---

## 9. Dashboard Standalone

Buat Text dinamis (Palette **Presentation** → **Text** → Type: `Dynamic`):

```java
// Text 1 — Judul
"=== SKELETON PERPUSTAKAAN 3D ==="
```

```java
// Text 2 — Total pengunjung
"Total pengunjung: " + totalSelesai + " | Mhs: " + totalMahasiswa + " | Dosen: " + totalDosen
```

```java
// Text 3 — Antrian
"Antrian scan KTM: " + srvScanKTM.queueSize()
```

```java
// Text 4 — Waktu rata-rata
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## 10. Jalankan untuk Uji Coba

### Persiapan: sambungkan output sementara

Karena modul 3-6 belum ada, untuk testing kita perlu sambungkan `selectTujuan.out1` sampai `out5` ke `wJalanKeluar.in` dulu.

**Caranya:**
1. Klik `selectTujuan.out1` (port segitiga di sisi kanan blok).
2. Tarik ke `wJalanKeluar.in`.
3. Ulangi untuk `out2`, `out3`, `out4`, `out5`.

### Test 1: Coba 10 menit

1. Buka experiment **Simulation**.
2. **Stop time** = `10`.
3. Klik **Run**.
4. Amati: orang muncul di entryLine? Antri di svcScanKTM?

### Test 2: Test penuh 30 menit

1. **Stop time** = `30`.
2. Run.
3. Cek console: `ARRIVE`, `TUJUAN`, `DONE`.
4. Perhatikan peak hour di menit 20-30: kedatangan harus lebih cepat.

### Perkiraan hasil wajar

- Total pengunjung: ~30-50 orang dalam 30 menit
- Antrean scan KTM paling panjang saat peak (menit 20-30)
- Tidak ada error stack trace di console

---

## 11. Yang Perlu Diubah Saat Integrasi

Saat modul lain siap, lakukan ini:

1. **`selectTujuan.out1`** → colok ke `srvPinjam.in` (Modul 7)
2. **`selectTujuan.out2`** → colok ke `srvKembali.in` (Modul 7)
3. **`selectTujuan.out3`** → colok ke `selectAktivitas.in` (Modul 4)
4. **`selectTujuan.out4`** → colok ke `selectGenderToilet.in` (Modul 5)
5. **`selectTujuan.out5`** → colok ke `srvFotokopi.in` (Modul 6)
6. **`wJalanBalikParkir.out`** → colok ke `jalanKeParkiran.in` (Modul 1)
7. **`snkParkir`** → hapus (karena sudah di-handle Modul 1)

---

## 12. Troubleshooting

### Masalah: "getInterarrivalTime() cannot be resolved"

**Penyebab:** Fungsi `getInterarrivalTime` belum dibuat.

**Solusi:** Buka Main → tambah Function → nama `getInterarrivalTime` → return type `double` → isi body.

### Masalah: Semua orang langsung ke out6 (langsung keluar)

**Penyebab:** `selectTujuan.out1` sampai `out5` tidak terkoneksi. Di beberapa versi, output yang tidak terkoneksi bisa menyebabkan error.

**Solusi:** Untuk standalone, koneksikan semua output ke `wJalanKeluar.in`.

### Masalah: selectPulang tidak memisah orang

**Penyebab:** `selectPulang` masih pakai probabilitas, bukan kondisi.

**Solusi:** Buka properties `selectPulang` → **Use conditions** di-centang, **Use probabilities** di-uncentang. Output 1 condition: `ped.isParkir == true`, Output 2 condition: `ped.isParkir == false`.

### Masalah: "Queue choice policy" error

**Penyebab:** `srvScanKTM.Services` kosong atau salah.

**Solusi:** Pastikan `srvScanKTM.Services` = `svcScanKTM` (markup yang sudah dibuat).

### Masalah: "ped cannot be cast" atau "agent cannot be cast"

**Penyebab:** Fungsi menggunakan parameter `agent` bukan `ped`.

**Solusi:** Cek fungsi `hitungWaktuScanKTM(...)` — parameter harus bernama `ped`, bukan `agent`.

### Masalah: Console penuh dengan tracing dan bikin lambat

**Solusi:** Komentari `traceln(...)` yang tidak perlu, atau kurangi frekuensinya.

---

## 13. Fallback / Alternatif

### Jika "Use conditions" tidak ditemukan di PedSelectOutput

Di versi AnyLogic yang lebih baru, nama field bisa berbeda. Cari opsi yang memungkinkan Anda memilih antara probabilities dan conditions — biasanya berupa radio button atau dropdown.

### Jika getInterarrivalTime() terlalu rumit

Ganti sementara dengan `exponential(2.0)` (konstan) saat testing:
```
Interarrival time: exponential(2.0)
```

### Jika hanya punya AnyLogic PLE

PLE (Personal Learning Edition) memiliki fitur yang sama untuk pedestrian library. Tidak ada perbedaan signifikan untuk modul ini.

---

## 14. Mode Presentasi

- **Stop time:** 20-25 menit (perlihatkan sebelum peak, saat peak, dan sesudah)
- **Speed slider runtime:** geser ke kiri agar gerakan terlihat natural
- **Fokus kamera 3D:** area scan KTM + antrean

---

## 15. Checklist Final

- [ ] **PengunjungPed** sudah dengan variabel: `idPed`, `tMasuk`, `isDosen`, `isParkir`, `noktp`, `isValidKTM`, `tujuanLayanan`
- [ ] **Markup**: `entryLine`, `exitLine`, `svcScanKTM` (2 services, 2 queues)
- [ ] **Flowchart**: `srcJalanKaki → srvScanKTM → selectTujuan → wJalanKeluar → selectPulang`
- [ ] **selectTujuan**: 6 output dengan probabilitas (25/15/25/10/10/15) ✅ total = 1.0
- [ ] **selectPulang**: 2 output dengan KONDISI (bukan probabilitas)
- [ ] **Fungsi**: `getInterarrivalTime()`, `hitungWaktuScanKTM(ped)`, `avgWaktuSistem()`
- [ ] **Dashboard**: minimal 3 metrik tampil real-time
- [ ] **Test 30 menit**: semua route berjalan, console mencetak log
- [ ] **Peak hour**: di menit 20-30 kedatangan meningkat signifikan
