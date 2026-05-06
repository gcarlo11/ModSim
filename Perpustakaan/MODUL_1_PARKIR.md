# Modul 1: Simulasi Parkir Perpustakaan
## Mobil dan Motor Datang, Parkir, Jalan Kaki ke Perpustakaan

**AnyLogic versi:** 8.9.8 (cek: Help → About)
**Satuan waktu:** `minute`
**Project asli:** `SimulasiParkirPerpustakaan/SimulasiParkirPerpustakaan.alp`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan parkir di perpustakaan:

1. Mobil datang dan cari parkir (lebih lama: 2-8 menit)
2. Motor datang dan cari parkir (lebih cepat: 1-3 menit)
3. Pengemudi turun, jalan kaki ke pintu perpustakaan
4. Saat pulang: orang balik dari perpustakaan ke parkir, lalu keluar

**Flowchart standalone:**

```
srcParkirMobil → srvParkirMobil (cari parkir) → jalanKePerpus
srcParkirMotor → srvParkirMotor (cari parkir) → jalanKePerpus (bergabung)
                                                      ↓
                                                 snkParkir (SELESAI → masuk perpus)

=== MODUL PULANG ===  (untuk integrasi nanti)
srcPulang (dari perpus) → jalanKeParkiran → snkPulang
```

**Konsep cepat untuk pemula:**
- **PedSource**: tempat keluarnya orang (entity) — di sini orang yang baru selesai parkir
- **PedService**: tempat orang dilayani — di sini proses "cari parkir" (delay waktu)
- **PedGoTo**: perintah jalan dari titik A ke titik B
- **PedSink**: tempat orang selesai dan keluar dari simulasi

---

## 1. Yang Sudah Ada di Project (Jika Pakai Project Lama)

Jika Anda membuka `SimulasiParkirPerpustakaan.alp`, komponen berikut **sudah jadi** dan tidak perlu dibuat ulang.

### 1.1 Blok flowchart yang sudah ada

| Blok | Nama | Fungsi |
|---|---|---|
| PedSource | `orangDatang` | Orang datang dari parkir |
| PedSource | `orangPulang` | Orang pulang ke parkir |
| PedGoTo | `jalanKePerpus` | Jalan dari parkir ke perpustakaan |
| PedGoTo | `jalanKeParkiran` | Jalan dari perpustakaan ke parkir |
| PedSink | `pedSink` | Selesai (masuk perpus) |
| PedSink | `pedSink1` | Selesai (pulang naik kendaraan) |

### 1.2 3D Objects yang sudah di-import

| File | Objek |
|---|---|
| `car.dae` | Mobil 3D |
| `motorcycle.dae` | Motor 3D |
| `building_1.dae` | Gedung perpustakaan |
| `person.dae` | Orang (pejalan kaki) |
| `tree_1.dae`, `tree_2.dae`, `tree_3.dae` | Pohon (3 variasi) |

### 1.3 Markup yang sudah ada

| Markup | Nama | Fungsi |
|---|---|---|
| TargetLine | `areaPejalanKaki` | Area jalan untuk pedestrian |
| RectangularNode | `pintuPerpus` | Posisi pintu masuk perpustakaan |

### 1.4 Koneksi flowchart yang sudah ada

```
orangDatang.out  → jalanKePerpus.in → pedSink.in
orangPulang.out  → jalanKeParkiran.in → pedSink1.in
```

---

## 2. Yang Perlu Ditambahkan (Jika Pakai Project Lama)

Jika project sudah ada, Anda hanya perlu **menambahkan** komponen di bawah ini.

### 2.1 Tambah Variabel di PengunjungPed

1. Klik 2x diagram **PengunjungPed** (di panel Projects).
2. Dari palette **Agent**, drag **Variable** ke canvas kosong. Ulangi 5 kali.
3. Atur properti setiap variable seperti tabel berikut:

| Nama | Type | Initial Value | Keterangan |
|---|---|---|---|
| `idPed` | String | `""` | ID unik (diisi otomatis: "Mobil-1", "Motor-2", dll) |
| `tMasuk` | double | `0` | Waktu masuk sistem (diisi di srcParkir) |
| `isParkir` | boolean | `false` | `true` = naik kendaraan. Dipakai untuk routing pulang |
| `jenisKendaraan` | String | `""` | "MOBIL" atau "MOTOR" (diisi di srcParkir) |
| `waktuParkir` | double | `0` | Waktu mulai parkir (diisi di onBeginService) |

> **Tips pemula:** Variabel adalah tempat penyimpanan data yang nempel ke setiap orang. Setiap orang punya data masing-masing. `isParkir` nanti dipakai Modul 2 untuk menentukan apakah orang balik ke parkir atau langsung keluar.

### 2.2 Tambah Variabel di Main

Buka **Main** (klik 2x di panel Projects). Dari palette **Agent**, drag **Variable** ke canvas:

| Nama | Type | Initial value | Fungsi |
|---|---|---|---|
| `seqPed` | int | 0 | Counter untuk generate ID unik |
| `totalSelesai` | int | 0 | Total orang yang selesai parkir |
| `totalWaktuSistem` | double | 0 | Akumulasi waktu semua orang (untuk hitung rata-rata) |
| `totalMobil` | int | 0 | Total mobil yang parkir |
| `totalMotor` | int | 0 | Total motor yang parkir |

> **Tips pemula:** Variabel di Main bersifat global — nilainya sama untuk semua orang. Dipakai untuk statistik.

### 2.3 Tambah Fungsi di Main

Klik **Main** di panel Projects. Dari palette **Agent**, drag **Function** ke canvas. Ulangi 3 kali.

**Fungsi 1: `hitungWaktuCariParkirMobil`**
- Return type: `double` (pilih dari dropdown)
- Parameter: tidak ada (kosongkan)

```java
// Mobil membutuhkan waktu 2-8 menit untuk mencari tempat parkir
// Lebih lama karena mobil butuh tempat yang lebih besar
return uniform(2.0, 8.0);
```

**Fungsi 2: `hitungWaktuCariParkirMotor`**
- Return type: `double`

```java
// Motor lebih lincah, parkir lebih cepat: 1-3 menit
return uniform(1.0, 3.0);
```

**Fungsi 3: `avgWaktuSistem`**
- Return type: `double`

```java
// Rata-rata waktu parkir = total waktu parkir / jumlah orang yang selesai
// Mencegah division by zero dengan ternary operator
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

> **Tips pemula:** Fungsi di AnyLogic ditulis **tanpa** deklarasi method lengkap. Cukup tulis isi kodenya saja di bagian body function. Pastikan `Return type` sudah dipilih.

---

## 3. Buat Project Baru (Jika Mulai dari Nol)

Lewati bagian ini jika Anda sudah punya project `SimulasiParkirPerpustakaan.alp` dan hanya perlu menambahkan komponen.

### 3.1 Langkah-langkah

1. Buka AnyLogic.
2. **File → New → Model**.
3. Isi **Model name**: `Perpustakaan3D_Parkir`.
4. **Time units**: pilih `minute`.
5. Klik **Finish**.

### 3.2 Buat Agent Type: PengunjungPed

1. Di panel **Projects**, klik kanan nama model (`Perpustakaan3D_Parkir`).
2. Pilih **New → Agent Type**.
3. Isi **Name**: `PengunjungPed`.
4. Pilih template: **Pedestrian** (bukan Agent).
5. Pilih animasi 3D orang (pilih salah satu, misal "Man" atau "Woman").
6. Klik **Finish**.
7. Buka diagram `PengunjungPed` (klik 2x).
8. Tambah 5 variable seperti di tabel bagian 2.1.

### 3.3 Buat Layout 3D

1. Dari palette **Presentation**, drag **3D Window** ke canvas Main. Rename: `win3D`.
2. Dari palette **Presentation**, drag **Camera** ke canvas Main. Rename: `camMain`. Atur posisi kamera (X=30, Y=-20, Z=20) agar area parkir terlihat.
3. Dari palette **3D Objects**, drag **Box** ke canvas. Rename: `lantaiParkir3D`. Skala: 40x30x0.2. Posisi: (20, 15, 0).
4. Import aset 3D: Palette 3D → **Import 3D** → cari file `.dae` → Import.

### 3.4 Buat Markup Pedestrian

1. **Palette → Space Markup → Target line**. Klik di canvas untuk menggambar garis. Rename: `entryParkir`. Ini adalah garis tempat mobil/motor muncul.
2. **Target line** lagi. Rename: `exitParkir`. Garis tempat kendaraan keluar.

---

## 4. Bangun Flowchart Lengkap

### 4.1 Blok flowchart

Dari palette **Pedestrian Library**, drag blok-blok berikut. Atur posisi agar mudah dilihat.

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcParkirMobil` | Sumber mobil masuk parkir |
| PedSource | `srcParkirMotor` | Sumber motor masuk parkir |
| PedService | `srvParkirMobil` | Proses mobil cari parkir |
| PedService | `srvParkirMotor` | Proses motor cari parkir |
| PedGoTo | `jalanKePerpus` | Jalan dari parkir ke perpustakaan |
| PedGoTo | `jalanKeParkiran` | Jalan dari perpustakaan ke parkir |
| PedSink | `snkParkir` | SELESAI sementara |
| PedSink | `snkPulang` | SELESAI pulang sementara |

### 4.2 Cara rename blok

Klik kanan blok → **Properties** → rename di field **Name** paling atas. Atau klik blok, lihat Properties di panel bawah, ganti field Name.

### 4.3 Koneksi flowchart

Hubungkan port (segitiga kecil) antars blok dengan cara:
- Klik port **out** blok pertama
- Tarik garis ke port **in** blok kedua
- Lepaskan

Buat koneksi seperti ini:

```
srcParkirMobil.out → srvParkirMobil.in → jalanKePerpus.in → snkParkir.in
srcParkirMotor.out → srvParkirMotor.in → jalanKePerpus.in   ^
                                        (satu garis sama)    |
                                                             (bergabung)
=== MODUL PULANG ===
srcPulang.out → jalanKeParkiran.in → snkPulang.in
```

PENTING: `jalanKePerpus.in` menerima koneksi dari DUA blok (`srvParkirMobil.out` DAN `srvParkirMotor.out`). Di AnyLogic, satu input bisa menerima koneksi dari banyak output.

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcParkirMobil` (PedSource) — Mobil Datang

Klik blok `srcParkirMobil`. Lihat panel **Properties** di bawah. Atur:

| Property | Nilai | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di garis (target line) |
| `Target line` | `entryParkir` | Pilih markup target line yang sudah dibuat |
| `Arrive according to` | `Interarrival time` | Kedatangan diatur oleh waktu antar-kedatangan |
| `Interarrival time` | `exponential(0.5)` | Rata-rata 1 mobil setiap 2 menit |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian yang akan muncul |

**Cara setting:**
1. Klik `srcParkirMobil`.
2. Di **Properties → General**:
   - `Appears at`: pilih `line`.
   - `Target line`: pilih `entryParkir` dari dropdown.
3. **Properties → arrivals**:
   - `Arrive according to`: pilih `Interarrival time`.
   - `Interarrival time`: ketik `exponential(0.5)`.
4. **Properties → Agent**:
   - `New pedestrian`: pilih `PengunjungPed`.

**Action On exit** (tab **Actions** di Properties):
Klik tab **Actions**, cari **On exit**, paste kode berikut:

```java
seqPed++;
ped.idPed = "Mobil-" + seqPed;
ped.tMasuk = time();
ped.isParkir = true;
ped.jenisKendaraan = "MOBIL";

traceln("MOBIL datang: " + ped.idPed);
```

> **Tips pemula:** Kode `seqPed++` menambah counter sehingga ID selalu unik. `ped.tMasuk = time()` menyimpan jam saat orang masuk — nanti dipakai hitung rata-rata waktu sistem.

### 5.2 `srcParkirMotor` (PedSource) — Motor Datang

| Property | Nilai | Penjelasan |
|---|---|---|
| `Appears at` | `line` | — |
| `Target line` | `entryParkir` | Satu garis dengan mobil (bergantian) |
| `Arrive according to` | `Interarrival time` | — |
| `Interarrival time` | `exponential(0.3)` | Rata-rata 1 motor setiap ~3.3 menit |
| `New pedestrian` | `PengunjungPed` | — |

**Action On exit:**
```java
seqPed++;
ped.idPed = "Motor-" + seqPed;
ped.tMasuk = time();
ped.isParkir = true;
ped.jenisKendaraan = "MOTOR";

traceln("MOTOR datang: " + ped.idPed);
```

> **Tips pemula:** `exponential(angka)` artinya rata-rata kedatangan tiap `1/angka` menit. `exponential(0.5)` = 1/0.5 = 2 menit. `exponential(0.3)` = 1/0.3 ≈ 3.3 menit. Makin besar angka, makin sering datang.

### 5.3 `srvParkirMobil` (PedService) — Mobil Cari Parkir

| Property | Nilai | Penjelasan |
|---|---|---|
| `Services` | (pilih markup) | Tidak perlu Service with Lines — bisa pakai RectangularNode sebagai area parkir |
| `Queue choice policy` | Shortest queue | — |
| `Delay time` | `hitungWaktuCariParkirMobil()` | Panggil fungsi yang sudah dibuat |
| `Recovery delay` | `0` | — |

**Action On begin service** (saat mulai parkir):
```java
ped.waktuParkir = time();
```

**Action On end service** (saat selesai parkir):
```java
totalMobil++;
traceln("PARKIR Mobil: " + ped.idPed + " | durasi=" + String.format("%.1f", time() - ped.waktuParkir) + " mnt");
```

> **Tips pemula:** `Delay time` adalah waktu yang dihabiskan di blok ini. Kita pakai fungsi `hitungWaktuCariParkirMobil()` yang menghasilkan angka acak 2-8 menit. Jadi setiap mobil berbeda-beda waktu parkirnya, seperti realistisnya.

### 5.4 `srvParkirMotor` (PedService) — Motor Cari Parkir

| Property | Nilai |
|---|---|
| `Delay time` | `hitungWaktuCariParkirMotor()` |
| `Recovery delay` | `0` |

**Action On begin service:**
```java
ped.waktuParkir = time();
```

**Action On end service:**
```java
totalMotor++;
traceln("PARKIR Motor: " + ped.idPed + " | durasi=" + String.format("%.1f", time() - ped.waktuParkir) + " mnt");
```

### 5.5 `jalanKePerpus` (PedGoTo) — Jalan ke Perpustakaan

| Property | Nilai |
|---|---|
| `Target` | `pintuPerpus` (RectangularNode) |

PedGoTo membuat pedestrian berjalan dari posisi saat ini menuju target. `pintuPerpus` adalah RectangularNode yang sudah ada di markup.

### 5.6 `snkParkir` (PedSink) — TEMPORARY

Ini bersifat **sementara** untuk testing standalone. Saat integrasi, blok ini akan dihapus dan diganti dengan koneksi ke `selectTujuan` (Modul 2).

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("SELESAI: " + ped.idPed + " | tSistem=" + String.format("%.2f", tSistem) + " menit");
```

### 5.7 `jalanKeParkiran` dan `snkPulang` (TEMPORARY)

`jalanKeParkiran` adalah PedGoTo yang mengarah ke `exitParkir`. Ini untuk simulasi pulang (dari perpustakaan ke parkir). Saat integrasi, blok ini tetap dipakai tetapi inputnya berasal dari `selectPulang` (Modul 2).

`snkPulang` → PedSink biasa, action on enter kosong.

---

## 6. Dashboard

Tambahkan **Text** dinamis untuk monitoring saat run.

Cara: Palette **Presentation** → drag **Text** ke canvas Main. Di Properties, ganti **Type** dari `Plain text` menjadi `Dynamic`. Isi **Expression** dengan kode di bawah.

Buat 3-4 Text terpisah untuk dashboard:

```java
// Text 1: Total kendaraan
"Total kendaraan: " + (totalMobil + totalMotor)
```

```java
// Text 2: Detail mobil dan motor
"Mobil: " + totalMobil + " | Motor: " + totalMotor
```

```java
// Text 3: Waktu rata-rata
"Rata-rata waktu parkir: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

```java
// Text 4: Keaktifan real-time
"Mobil antri: " + srvParkirMobil.queueSize() + " | Motor antri: " + srvParkirMotor.queueSize()
```

> **Tips pemula:** `String.format("%.2f", angka)` digunakan untuk membatasi angka desimal jadi 2 digit di belakang koma. Misal 3.14159 → 3.14.

---

## 7. Jalankan untuk Uji Coba

### Test 1: Jalan cepat (10 menit)

1. Buka experiment **Simulation** (biasanya sudah auto-generate namanya `Simulation`).
2. Di Properties → **Model time** → **Stop time**: ketik `10`.
3. Klik **Run** (tombol hijau).
4. Amati: apakah mobil dan motor muncul? Apakah mereka jalan ke `pintuPerpus`?

### Test 2: Test penuh (30 menit)

1. Stop time = `30`.
2. Run.
3. Cek console (biasanya di bawah, tab Console):
   - Harus ada: `MOBIL datang: Mobil-1`, `PARKIR Mobil: Mobil-1 | durasi=...`
   - Harus ada: `MOTOR datang: Motor-1`, `PARKIR Motor: Motor-1 | durasi=...`
   - Harus ada: `SELESAI: Mobil-1 | tSistem=...`

### Perkiraan hasil wajar

- Total mobil parkir: ~10-15 (setiap ~2 menit ada mobil, 30 menit)
- Total motor parkir: ~7-10 (setiap ~3.3 menit ada motor)
- Rata-rata waktu parkir mobil: 4-6 menit (antara 2-8)
- Rata-rata waktu parkir motor: 1.5-2.5 menit (antara 1-3)

---

## 8. Yang Perlu Diubah Saat Integrasi

Nanti saat semua modul siap, seorang integrator akan melakukan:

**Blok yang dihapus:**
1. `snkParkir` → output `jalanKePerpus` akan colok ke `srvScanKTM.in` (Modul 2)
2. `snkPulang` → input `jalanKeParkiran` berasal dari `selectPulang.out1` (Modul 2)

**Blok yang tetap:**
- `srcParkirMobil`, `srcParkirMotor` (tetap sebagai sumber)
- `srvParkirMobil`, `srvParkirMotor` (tetap)
- `jalanKePerpus` (output dialihkan)
- `jalanKeParkiran` (input dialihkan)

**Koneksi baru setelah integrasi:**
```
jalanKePerpus.out → srvScanKTM.in (bukan snkParkir)
wJalanBalikParkir.out → jalanKeParkiran.in (bukan dari srcPulang)
```

---

## 9. Troubleshooting

### Masalah: "idPed cannot be resolved or is not a field"

**Penyebab:** Variabel `idPed` belum dibuat di diagram `PengunjungPed`.

**Solusi:** Buka diagram PengunjungPed, tambah Variable dengan nama `idPed`, type `String`, initial value `""`.

### Masalah: "seqPed cannot be resolved"

**Penyebab:** Variabel `seqPed` belum dibuat di Main.

**Solusi:** Buka diagram Main, tambah Variable `seqPed` type `int`, initial value `0`.

### Masalah: Mobil/motor tidak muncul sama sekali

**Penyebab 1:** `Target line` belum dipilih di properti PedSource.
**Solusi:** Cek `srcParkirMobil → Target line` harus指向 `entryParkir`.

**Penyebab 2:** `entryParkir` ketutup atau terlalu kecil.
**Solusi:** Perbesar target line atau pindahkan ke area yang tidak ada halangannya.

### Masalah: Semua orang jalan menembus gedung

**Penyebab:** Pedestrian library mencari jalur terpendek, dan `pintuPerpus` mungkin terhalang.

**Solusi:** Pindahkan `pintuPerpus` ke posisi yang terbuka. Atau tambahkan dinding/barrier di markup.

### Masalah: "Function hitungWaktuCariParkirMobil() not found"

**Penyebab:** Fungsi belum dibuat di Main.

**Solusi:** Buka Main → tambah Function → nama `hitungWaktuCariParkirMobil` → return type `double` → isi body dengan `return uniform(2.0, 8.0);`.

### Masalah: "agent cannot be cast to PengunjungPed"

**Penyebab:** Ada kode di suatu tempat yang mencoba meng-cast `agent` ke `PengunjungPed` tapi tipe pedestrian-nya berbeda.

**Solusi:** Pastikan `New pedestrian` di PedSource adalah `PengunjungPed`, bukan pedestrian type lain.

### Masalah: Error "Delay time must be specified"

**Penyebab:** Field `Delay time` di PedService kosong.

**Solusi:** Isi dengan `hitungWaktuCariParkirMobil()` atau `hitungWaktuCariParkirMotor()`.

---

## 10. Fallback / Alternatif

### Jika tidak punya file .dae 3D

Ganti dengan **Box** sederhana:
- `mobil3D` → Box ukuran 2x1x0.8
- `motor3D` → Box ukuran 1.5x0.8x0.8

Caranya: Palette **3D Objects** → **Box** → rename → atur ukuran.

### Jika Pedestrian Library tidak punya animasi orang

Pilih pedestrian animation "Simple Man" atau "Simple Woman" — ini yang paling ringan dan tidak perlu file 3D terpisah.

---

## 11. Mode Presentasi

Saat demo, atur:
- **Stop time:** 15-20 menit (cukup lihat pola)
- **Speed slider runtime:** geser ke kiri agar gerakan tidak terlalu cepat
- **Fokus kamera 3D:** area parkir dan pintu perpustakaan

Tips: Jika terlalu banyak antrean parkir, turunkan arrival rate sementara:
- `exponential(0.3)` untuk mobil (lebih jarang)
- `exponential(0.2)` untuk motor (lebih jarang)

---

## 12. Checklist Final

- [ ] **PengunjungPed** sudah punya variabel: `idPed`, `tMasuk`, `isParkir`, `jenisKendaraan`, `waktuParkir`
- [ ] **Variabel Main**: `seqPed`, `totalSelesai`, `totalWaktuSistem`, `totalMobil`, `totalMotor`
- [ ] **Fungsi**: `hitungWaktuCariParkirMobil()`, `hitungWaktuCariParkirMotor()`, `avgWaktuSistem()`
- [ ] **srcParkirMobil**: Target line = `entryParkir`, interarrival = `exponential(0.5)`
- [ ] **srcParkirMotor**: Target line = `entryParkir`, interarrival = `exponential(0.3)`
- [ ] **srvParkirMobil**: Delay time = `hitungWaktuCariParkirMobil()`
- [ ] **srvParkirMotor**: Delay time = `hitungWaktuCariParkirMotor()`
- [ ] **jalanKePerpus**: Target = `pintuPerpus`
- [ ] **All connections**: src → srv → jalanKePerpus → snkParkir
- [ ] **Dashboard** minimal 3 metrik tampil
- [ ] **Test run** 30 menit berhasil tanpa error
