# Modul 3: Helpdesk Pencarian Buku
## Antrean Bantuan Petugas Ketika Mahasiswa Gagal Menemukan Buku Sendiri

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan **helpdesk pencarian buku** di perpustakaan — tempat mahasiswa meminta bantuan petugas ketika kesulitan menemukan buku di rak:

1. Mahasiswa datang dan **mencari buku secara mandiri** terlebih dahulu
2. **70% berhasil** menemukan buku sendiri → langsung ke peminjaman
3. **30% gagal** menemukan buku → menuju **helpdesk** untuk minta bantuan petugas
4. Di helpdesk: **antre**, lalu **dilayani petugas** (2 petugas, waktu triangular(2, 4, 6) menit)
5. Setelah dilayani, petugas memberi informasi **status buku**:
   - **40% buku tersedia** → mahasiswa ke rak buku, lalu pinjam
   - **35% buku sedang dipinjam** → mahasiswa bisa pinjam (reservasi) atau keluar
   - **25% buku tidak ditemukan** → mahasiswa keluar
6. Selesai → keluar sistem

**Flowchart standalone:**

```
srcMahasiswa (pedSource, 20 mahasiswa/jam)
    │
    ▼
cariBukuMandiri (mahasiswa cari sendiri dulu — aktivitas mandiri)
    │
    ▼
selectButuhBantuan (pedSelectOutput — 70/30)
  ├── out1 (70%) → bukuDitemukan → langsung pinjam/keluar
  │
  └── out2 (30%) → goToHelpdesk → antreHelpdesk (queue)
                                    │
                                    ▼
                              srvHelpdesk (pedService, 2 petugas)
                              delay: triangular(2, 4, 6) menit
                                    │
                                    ▼
                              selectStatusBuku (pedSelectOutput)
                                ├── out1 (40%) → buku tersedia → rakBuku → pinjam
                                ├── out2 (35%) → sedang dipinjam → reservasi/pinjam
                                └── out3 (25%) → tidak ditemukan → keluar
                                      │
                                      ▼
                                snkSelesai (pedSink)
```

**Konsep cepat untuk pemula:**
- **pedSource**: sumber mahasiswa — di sini 20 mahasiswa per jam
- **pedSelectOutput**: percabangan probabilistik — 70% berhasil mandiri, 30% butuh helpdesk
- **pedGoTo**: perintah jalan dari titik A ke titik B
- **pedQueue**: tempat antre sebelum dilayani — mahasiswa menunggu giliran
- **pedService**: proses pelayanan — petugas mencari informasi buku
- **pedSink**: tempat mahasiswa keluar dari simulasi
- **triangular(min, mode, max)**: distribusi triangular — waktu paling sering di mode (4 menit), dengan rentang min-max (2-6 menit)

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMahasiswa` | TEMPORARY — sumber mahasiswa masuk |
| PedSelectOutput | `selectButuhBantuan` | 70% berhasil sendiri, 30% butuh helpdesk |
| PedGoTo | `goToHelpdesk` | Jalan menuju area helpdesk |
| PedQueue | `antreHelpdesk` | Antrean sebelum dilayani petugas |
| PedService | `srvHelpdesk` | Pelayanan petugas helpdesk |
| PedSelectOutput | `selectStatusBuku` | Hasil pencarian: tersedia/dipinjam/tidak ada |
| PedGoTo | `goToRakBuku` | Jalan ke rak buku |
| PedGoTo | `goToPinjam` | Jalan ke area peminjaman |
| PedSink | `snkSelesai` | TEMPORARY — mahasiswa keluar |

### 1.2 Resource Pool (opsional — jika pakai blok Service Process Modeling)

| Resource | Nama | Capacity |
|---|---|---|
| ResourcePool | `petugasHelpdesk` | 2 petugas |

> **Catatan:** Untuk modul Pedestrian Library, kita bisa menggunakan **PedService** yang sudah memiliki konsep kapasitas internal (2 service points). ResourcePool dari Process Modeling Library bisa ditambahkan jika ingin lebih eksplisit.

### 1.3 Markup Pedestrian

| Markup | Nama | Detail |
|---|---|---|
| TargetLine | `entryLine` | Garis masuk mahasiswa |
| TargetLine | `exitLine` | Garis keluar |
| Path | `jalurHelpdesk` | Jalur pejalan kaki ke helpdesk |
| Path | `jalurRakBuku` | Jalur ke rak buku |
| Path | `jalurPinjam` | Jalur ke area peminjaman |
| Point Node | `nodeHelpdesk` | Titik lokasi helpdesk |
| Point Node | `nodeRakBuku` | Titik lokasi rak buku |
| Point Node | `nodePinjam` | Titik lokasi peminjaman |
| Rectangular Node | `areaAntreHelpdesk` | Area antrean di depan meja helpdesk |
| Service with Lines | `svcHelpdesk` | 2 petugas, 1 antrean |

### 1.4 3D

| Objek | Nama | Penjelasan |
|---|---|---|
| 3D Window | `win3D` | Jendela 3D |
| Camera | `camMain` | Kamera utama |
| Floor (Box) | `lantaiPerpus3D` | Lantai perpustakaan |
| Box | `mejaHelpdesk3D` | Meja petugas helpdesk |
| Box | `komputer3D` | Komputer di meja helpdesk |
| Box | `kursiPetugas3D` | Kursi petugas |
| Box (beberapa) | `rakBuku3D_1` s/d `_4` | Rak buku berbagai kategori |
| Walls | `dindingDepan3D`, dll | Dinding ruangan |
| Box | `papanPetunjuk3D` | Papan petunjuk arah |

### 1.5 Variabel baru di PengunjungPed

Tambah ke diagram `PengunjungPed`:

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `idPed` | String | `""` | ID unik (contoh: "Mhs-1") |
| `tMasuk` | double | `0` | Waktu masuk sistem |
| `butuhHelpdesk` | boolean | `false` | `true` = gagal cari sendiri, butuh bantuan |
| `statusBuku` | String | `""` | "TERSEDIA", "DIPINJAM", atau "TIDAK_ADA" |
| `waktuMulaiAntre` | double | `0` | Waktu mulai antre di helpdesk |
| `waktuMulaiLayanan` | double | `0` | Waktu mulai dilayani petugas |
| `waktuSelesaiLayanan` | double | `0` | Waktu selesai dilayani |
| `waktuTunggu` | double | `0` | Lama menunggu antrean |
| `waktuLayanan` | double | `0` | Lama dilayani petugas |

### 1.6 Variabel baru di Main

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `seqPed` | int | 0 | Counter ID mahasiswa |
| `totalMahasiswa` | int | 0 | Total mahasiswa masuk |
| `totalSelesai` | int | 0 | Total mahasiswa selesai |
| `totalKeHelpdesk` | int | 0 | Total yang ke helpdesk |
| `totalBukuTersedia` | int | 0 | Total buku tersedia |
| `totalBukuDipinjam` | int | 0 | Total buku sedang dipinjam |
| `totalBukuTidakAda` | int | 0 | Total buku tidak ditemukan |
| `maxAntreanHelpdesk` | int | 0 | Antrean maksimum helpdesk |
| `totalWaktuTunggu` | double | 0 | Akumulasi waktu tunggu |
| `totalWaktuLayanan` | double | 0 | Akumulasi waktu layanan |
| `totalWaktuSistem` | double | 0 | Akumulasi waktu di sistem |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Isi **Model name**: `Perpustakaan3D_Helpdesk`.
4. **Time units**: pilih `minute`.
5. Klik **Finish**.

### 2.1 Buat Agent Type: PengunjungPed

1. Di panel Projects, klik kanan nama model.
2. **New → Agent Type**.
3. Name: `PengunjungPed`.
4. Template: **Pedestrian** (bukan Agent biasa).
5. Pilih animasi 3D (contoh: "Man" atau "Student").
6. Klik **Finish**.
7. Buka diagram `PengunjungPed` (klik 2x).
8. Dari palette **Agent**, drag **Variable** ke canvas. Tambah sesuai tabel 1.5 satu per satu.

> **Tips pemula:** Nama variabel case-sensitive. `butuhHelpdesk` berbeda dengan `butuhhelpdesk`.

### 2.2 Tambah Variabel di Main

Buka **Main** → drag **Variable** 11 kali → isi sesuai tabel 1.6.

---

## 3. Layout 3D

### 3.1 Buka jendela 3D

1. Dari palette **Presentation**, drag **3D Window** ke canvas Main.
2. Rename: `win3D`.
3. Perbesar ukurannya (minimal setengah layar).

### 3.2 Tambah kamera

1. **Presentation** → drag **Camera** → rename `camMain`.
2. Atur posisi kamera agar area helpdesk, rak buku, dan pintu masuk terlihat.
3. Contoh posisi: X=20, Y=-15, Z=15.

### 3.3 Bangun area perpustakaan

1. **3D Objects** → **Box** → `lantaiPerpus3D`.
   - Size: 30 x 25 x 0.2
   - Posisi: (15, 12, 0)

2. **3D Objects** → **Box** → `mejaHelpdesk3D`.
   - Size: 3 x 1.5 x 1.2
   - Posisi: (10, 15, 0)

3. **3D Objects** → **Box** → `komputer3D`.
   - Size: 0.8 x 0.6 x 0.8
   - Posisi: di atas meja helpdesk

4. **3D Objects** → **Box** → `kursiPetugas3D`.
   - Size: 0.8 x 0.8 x 1
   - Posisi: di belakang meja helpdesk

5. Buat **rak buku 3D** (Box) beberapa buah:
   - `rakBuku3D_1`: size 1x3x2, posisi (5, 10)
   - `rakBuku3D_2`: size 1x3x2, posisi (8, 10)
   - `rakBuku3D_3`: size 1x3x2, posisi (5, 18)
   - `rakBuku3D_4`: size 1x3x2, posisi (8, 18)

6. **3D Objects** → **Walls** → beberapa dinding untuk membentuk ruangan.

7. **3D Objects** → **Box** → `papanPetunjuk3D` (ukuran kecil, tipis).

> **Tips pemula:** Tidak perlu semua objek 3D sempurna di awal. Mulai dari lantai, meja helpdesk, dan 2-3 rak buku. Sisanya bisa ditambah nanti.

---

## 4. Markup Pedestrian

### 4.1 Target lines

1. **Space Markup** → **Target line** → rename `entryLine`.
   - Letakkan di dekat pintu masuk (kiri).
2. **Space Markup** → **Target line** → rename `exitLine`.
   - Letakkan di dekat pintu keluar (kanan).

### 4.2 Nodes (titik koordinat)

1. **Space Markup** → **Point Node** → rename `nodeHelpdesk`.
   - Letakkan di depan meja helpdesk.
2. **Space Markup** → **Point Node** → rename `nodeRakBuku`.
   - Letakkan di area rak buku.
3. **Space Markup** → **Point Node** → rename `nodePinjam`.
   - Letakkan di area peminjaman (bisa di dekat pintu keluar).

### 4.3 Paths (jalur)

1. **Space Markup** → **Path** → dari entryLine ke nodeHelpdesk → rename `jalurHelpdesk`.
2. **Space Markup** → **Path** → dari nodeHelpdesk ke nodeRakBuku → rename `jalurRakBuku`.
3. **Space Markup** → **Path** → dari nodeRakBuku ke nodePinjam → rename `jalurPinjam`.

### 4.4 Area antrean

1. **Space Markup** → **Rectangular Node** → rename `areaAntreHelpdesk`.
   - Letakkan di depan meja helpdesk, di sepanjang jalur.
   - Ukuran sesuaikan dengan perkiraan antrean (misal 4x2 meter).

### 4.5 Service with Lines — Helpdesk

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcHelpdesk`.
3. **Number of services** = `2` (2 petugas).
4. **N of queues** = `1` (1 antrean — mahasiswa ke petugas yang kosong lebih dulu).
5. Letakkan **service point** di depan meja helpdesk (posisi petugas).
6. Tarik **queue line** menjauh dari meja — membentuk baris antrean.

---

## 5. Bangun Flowchart

### 5.1 Blok

Dari palette **Pedestrian Library**, drag blok-blok berikut:

| Blok | Rename |
|---|---|
| PedSource | `srcMahasiswa` |
| PedSelectOutput | `selectButuhBantuan` |
| PedGoTo | `goToHelpdesk` |
| PedQueue | `antreHelpdesk` |
| PedService | `srvHelpdesk` |
| PedSelectOutput | `selectStatusBuku` |
| PedGoTo | `goToRakBuku` |
| PedGoTo | `goToPinjam` |
| PedSink | `snkSelesai` |

### 5.2 Koneksi flowchart

```
srcMahasiswa.out → selectButuhBantuan.in

selectButuhBantuan.out1 (70%, berhasil sendiri) → goToPinjam.in → snkSelesai.in
selectButuhBantuan.out2 (30%, butuh bantuan)    → goToHelpdesk.in → antreHelpdesk.in

antreHelpdesk.out → srvHelpdesk.in

srvHelpdesk.out → selectStatusBuku.in

selectStatusBuku.out1 (40%, buku tersedia)  → goToRakBuku.in → goToPinjam.in → snkSelesai.in
selectStatusBuku.out2 (35%, sedang dipinjam) → goToPinjam.in → snkSelesai.in
selectStatusBuku.out3 (25%, tidak ditemukan) → snkSelesai.in
```

> **Catatan:** `antreHelpdesk` adalah PedQueue — mahasiswa menunggu di sini sebelum dilayani `srvHelpdesk`. Blok PedQueue otomatis menampung antrean.

---

## 6. Konfigurasi Detail Setiap Blok

### 6.1 `srcMahasiswa` (PedSource)

| Property | Value | Penjelasan |
|---|---|---|
| `Appears at` | `line` | Muncul di target line |
| `Target line` | `entryLine` | Garis masuk perpustakaan |
| `Arrive according to` | `Interarrival time` | Waktu antar kedatangan |
| `Interarrival time` | `exponential(3.0)` | Rata-rata 20 mahasiswa/jam (60/20 = 3 menit antar kedatangan) |
| `New pedestrian` | `PengunjungPed` | Tipe pedestrian |

> **Tips pemula:** 20 mahasiswa/jam = 1 mahasiswa setiap 3 menit. Angka 3.0 di `exponential(3.0)` artinya rata-rata jeda 3 menit antar kedatangan.

**Action On exit:**
```java
seqPed++;
ped.idPed = "Mhs-" + seqPed;
ped.tMasuk = time();
ped.butuhHelpdesk = false; // default: belum butuh
ped.statusBuku = "";

totalMahasiswa++;

traceln("DATANG " + ped.idPed + " | waktu=" + String.format("%.1f", time()) + " mnt");
```

### 6.2 `selectButuhBantuan` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `2` |
| `Use probabilities` | Centang |
| Output 1 | `0.70` (70% berhasil cari sendiri) |
| Output 2 | `0.30` (30% butuh bantuan helpdesk) |

**Penjelasan logika bisnis:** Berdasarkan data perpustakaan, ~70% mahasiswa berhasil menemukan buku secara mandiri. Sisanya 30% membutuhkan bantuan — entah karena buku salah letak, tidak tahu sistem klasifikasi, atau rak terlalu penuh.

**Action On exit (out2):**
```java
ped.butuhHelpdesk = true;
traceln("BUTUH_HELP " + ped.idPed + " — pergi ke helpdesk");
```

### 6.3 `goToHelpdesk` (PedGoTo)

| Property | Value |
|---|---|
| `Target` | `nodeHelpdesk` (atau `areaAntreHelpdesk`) |

Mengarahkan mahasiswa berjalan dari posisi saat ini menuju titik helpdesk.

### 6.4 `antreHelpdesk` (PedQueue)

| Property | Value | Penjelasan |
|---|---|---|
| `Queue capacity` | Kosong (unlimited) | Antrean tanpa batas |
| `Waiting location` | `areaAntreHelpdesk` | Area tunggu |
| `Enable exit on timeout` | **JANGAN centang** | Biarkan unchecked |

**Action On enter:**
```java
ped.waktuMulaiAntre = time();

int q = antreHelpdesk.size();
if (q > maxAntreanHelpdesk) {
    maxAntreanHelpdesk = q;
}

traceln("ANTRE " + ped.idPed + " | posisi=" + q);
```

### 6.5 `srvHelpdesk` (PedService)

| Property | Value | Penjelasan |
|---|---|---|
| `Services` | `svcHelpdesk` | Service with Lines untuk 2 petugas |
| `Queue choice policy` | `Shortest queue` | Ambil petugas yang kosong pertama |
| `Delay time` | `triangular(2, 4, 6)` | Waktu layanan: min=2, mode=4, max=6 menit |
| `Recovery delay` | `0` | — |

**Action On begin service:**
```java
ped.waktuMulaiLayanan = time();
ped.waktuTunggu = ped.waktuMulaiLayanan - ped.waktuMulaiAntre;

totalWaktuTunggu += ped.waktuTunggu;

traceln("LAYANI " + ped.idPed + " | tunggu=" + String.format("%.1f", ped.waktuTunggu) + " mnt");
```

**Action On end service:**
```java
ped.waktuSelesaiLayanan = time();
ped.waktuLayanan = ped.waktuSelesaiLayanan - ped.waktuMulaiLayanan;

totalKeHelpdesk++;
totalWaktuLayanan += ped.waktuLayanan;

traceln("SELESAI_LAYANAN " + ped.idPed + " | durasi=" + String.format("%.1f", ped.waktuLayanan) + " mnt");
```

> **Penjelasan triangular(2, 4, 6):** Distribusi triangular dengan:
> - Minimum: 2 menit (kasus sederhana — buku ada di rak sebelah)
> - Mode (paling sering): 4 menit (kasus normal — perlu cek sistem, lalu tunjuk rak)
> - Maximum: 6 menit (kasus rumit — buku hilang, perlu cek database lain)

### 6.6 `selectStatusBuku` (PedSelectOutput)

| Property | Value |
|---|---|
| `N outputs` | `3` |
| `Use probabilities` | Centang |
| Output 1 | `0.40` (40% buku tersedia di rak) |
| Output 2 | `0.35` (35% buku sedang dipinjam orang lain) |
| Output 3 | `0.25` (25% buku tidak ditemukan sama sekali) |

**Action On exit (setiap port terpisah):**

> ⚠️ **PENTING:** JANGAN pakai `getExitPortIndex()` — method itu hanya ada di action **On exit port**, dan di beberapa versi AnyLogic malah undefined. Sebagai gantinya, isi kode **LANGSUNG** di masing-masing **On exit port** (klik tab **Actions → On exit port** di Properties blok ini). Atau lebih aman: pakai cara di bawah ini dengan menulis kode di **On exit** tapi tanpa `getExitPortIndex()`.

**Cara paling aman (tanpa getExitPortIndex):**

Klik 2x properties `selectStatusBuku`, tab **Actions**:

- **On exit port 1** (buku tersedia):
```java
ped.statusBuku = "TERSEDIA";
totalBukuTersedia++;
traceln("STATUS " + ped.idPed + " | BUKU TERSEDIA → ke rak buku");
```

- **On exit port 2** (sedang dipinjam):
```java
ped.statusBuku = "DIPINJAM";
totalBukuDipinjam++;
traceln("STATUS " + ped.idPed + " | SEDANG DIPINJAM → reservasi");
```

- **On exit port 3** (tidak ditemukan):
```java
ped.statusBuku = "TIDAK_ADA";
totalBukuTidakAda++;
traceln("STATUS " + ped.idPed + " | TIDAK DITEMUKAN → keluar");
```

### 6.7 `goToRakBuku` (PedGoTo)

| Property | Value |
|---|---|
| `Target` | `nodeRakBuku` |

Mengarahkan mahasiswa ke rak buku setelah dapat info dari petugas.

### 6.8 `goToPinjam` (PedGoTo)

| Property | Value |
|---|---|
| `Target` | `nodePinjam` |

Mengarahkan mahasiswa ke area peminjaman.

### 6.9 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
```java
totalSelesai++;
totalWaktuSistem += (time() - ped.tMasuk);

traceln("KELUAR " + ped.idPed
    + " | status=" + ped.statusBuku
    + " | helpdesk=" + ped.butuhHelpdesk
    + " | tSistem=" + String.format("%.2f", time()-ped.tMasuk) + " mnt");
```

---

## 7. Fungsi di Main

### 7.1 `avgWaktuTunggu`

Return type: `double`.

```java
// Rata-rata waktu tunggu di helpdesk (hanya yang ke helpdesk)
return totalKeHelpdesk == 0 ? 0 : totalWaktuTunggu / totalKeHelpdesk;
```

### 7.2 `avgWaktuLayanan`

Return type: `double`.

```java
// Rata-rata waktu dilayani petugas
return totalKeHelpdesk == 0 ? 0 : totalWaktuLayanan / totalKeHelpdesk;
```

### 7.3 `avgWaktuSistem`

Return type: `double`.

```java
// Rata-rata waktu total di sistem (semua mahasiswa)
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

### 7.4 `tingkatKeberhasilanHelpdesk`

Return type: `double`.

```java
// Persentase buku ditemukan setelah bantuan helpdesk
// (tersedia + dipinjam = buku yang ada)
return totalKeHelpdesk == 0 ? 0
    : 100.0 * (totalBukuTersedia + totalBukuDipinjam) / totalKeHelpdesk;
```

---

## 8. Dashboard

Tambahkan **Text** dinamis (Palette **Presentation** → **Text** → Type: `Dynamic`):

```java
// Text 1 — Total mahasiswa
"Total mahasiswa: " + totalMahasiswa + " | Selesai: " + totalSelesai
```

```java
// Text 2 — Helpdesk
"Ke helpdesk: " + totalKeHelpdesk + " | Antrean: " + antreHelpdesk.size() + " (max: " + maxAntreanHelpdesk + ")"
```

```java
// Text 3 — Status buku
"Buku Tersedia: " + totalBukuTersedia + " | Dipinjam: " + totalBukuDipinjam + " | Tidak Ada: " + totalBukuTidakAda
```

```java
// Text 4 — Waktu
"Rata-rata tunggu: " + String.format("%.2f", avgWaktuTunggu()) + " mnt | Layanan: " + String.format("%.2f", avgWaktuLayanan()) + " mnt"
```

```java
// Text 5 — Keberhasilan
"Tingkat keberhasilan helpdesk: " + String.format("%.1f", tingkatKeberhasilanHelpdesk()) + "%"
```

```java
// Text 6 — Waktu sistem
"Rata-rata waktu sistem: " + String.format("%.2f", avgWaktuSistem()) + " mnt"
```

---

## 9. Jalankan untuk Uji Coba

### Test 1: Coba 15 menit

1. Buka **Simulation** experiment.
2. **Stop time** = `15`.
3. Klik **Run**.
4. Amati: mahasiswa muncul? Ada yang ke helpdesk? Ada yang langsung pinjam?

### Test 2: Test penuh 60 menit

1. **Stop time** = `60`.
2. Run.
3. Cek console: `DATANG`, `BUTUH_HELP`, `ANTRE`, `LAYANI`, `SELESAI_LAYANAN`, `STATUS`, `KELUAR`.

### Test 3: Test 120 menit (2 jam)

1. **Stop time** = `120`.
2. Pantau: antrean helpdesk mulai terbentuk? Max antrean berapa?

### Perkiraan hasil wajar (60 menit)

- Total mahasiswa: ~20 orang (exponential 3.0 = 1 orang/3 menit)
- Ke helpdesk: ~6 orang (30% dari 20)
- Buku tersedia: ~2-3 orang (40% dari 6)
- Buku dipinjam: ~2 orang (35% dari 6)
- Buku tidak ada: ~1-2 orang (25% dari 6)
- Rata-rata waktu layanan: ~4 menit (mode triangular)
- Max antrean: 1-3 orang

---

## 10. Yang Perlu Diubah Saat Integrasi

Saat modul lain siap, integrator akan:

1. **Hapus** `srcMahasiswa` → diganti koneksi dari **Modul 4 (Cari Tempat Duduk)**: setelah `srvCariBuku`, 30% yang gagal menemukan buku diarahkan ke `goToHelpdesk.in`.
2. **Hapus** `snkSelesai` → output `goToPinjam.out` colok ke `srvPinjam.in` (Modul 7), output `snkSelesai` untuk yang keluar colok ke `wJalanKeluar.in` (Modul 2 skeleton).
3. **Alternatif**: modul helpdesk bisa juga dipanggil langsung dari `selectTujuan` skeleton sebagai output tambahan (out7), jika ingin semua mahasiswa bisa akses helpdesk tanpa harus melalui aktivitas "cari buku" dulu.

### Koneksi integrasi (opsi utama)

```
dari srvCariBuku.out (Modul 4)
    │
    ▼ (30% yang gagal)
goToHelpdesk.in (Modul 3) → antreHelpdesk → srvHelpdesk → selectStatusBuku
    │                                                           │
    ├── out1 (tersedia) → goToRakBuku → goToPinjam     ←───────┘
    │                              │
    │                              └→ srvPinjam.in (Modul 7)
    ├── out2 (dipinjam) → goToPinjam → srvPinjam.in (Modul 7)
    └── out3 (tidak ada) → wJalanKeluar.in (Modul 2)
```

---

## 11. Troubleshooting

### Masalah: Antrean tidak pernah terbentuk

**Penyebab:** Arrival rate terlalu lambat atau probabilitas 30% jarang terjadi.

**Solusi:** Naikkan arrival rate jadi `exponential(1.0)` (60 mahasiswa/jam) atau naikkan probabilitas ke helpdesk jadi `0.50`.

### Masalah: "triangular" tidak dikenal

**Penyebab:** Beberapa versi AnyLogic menggunakan nama berbeda.

**Solusi:** Ganti dengan `uniform(2, 6)` atau `triangular(2, 6, 4)`. Cek autocomplete di editor untuk nama yang tepat.

### Masalah: PedQueue error — "Cannot resolve antreHelpdesk.size()"

**Penyebab:** Di versi AnyLogic tertentu, PedQueue tidak punya method `.size()`.

**Solusi:** Ganti `antreHelpdesk.size()` dengan `antreHelpdesk.queueSize()`.

### Masalah: PedGoTo tidak bergerak

**Penyebab:** Target node tidak terhubung ke path/jalur pedestrian.

**Solusi:** Pastikan semua Point Node terhubung dengan Path. Pedestrian hanya bisa berjalan di jalur yang sudah ditentukan.

### Masalah: Service point tidak muncul

**Penyebab:** `srvHelpdesk.Services` belum diset ke `svcHelpdesk`.

**Solusi:** Cek properties srvHelpdesk → `Services` = `svcHelpdesk`.

### Masalah: Buku selalu "TERSEDIA"

**Penyebab:** `selectStatusBuku` probabilities belum diatur.

**Solusi:** Cek: out1=0.40, out2=0.35, out3=0.25. Total harus 1.0.

---

## 12. Skenario Uji Untuk Laporan

Jalankan 3 skenario ini dan bandingkan:

### Skenario A (Normal — 2 petugas)
- Arrival: `exponential(3.0)` (20 mhs/jam)
- Petugas: 2
- Hasil: antrean pendek, waktu tunggu rendah

### Skenario B (Ramai — 2 petugas)
- Arrival: `exponential(1.5)` (40 mhs/jam)
- Petugas: 2
- Hasil: antrean mulai mengular, waktu tunggu naik

### Skenario C (Perbaikan — 3 petugas)
- Arrival: `exponential(1.5)` (40 mhs/jam)
- Petugas: 3 (ganti `svcHelpdesk.Number of services` jadi 3)
- Hasil: antrean lebih terkendali

**Interpretasi:**
- Kedatangan makin rapat → antrean naik → butuh petugas lebih banyak
- Tambah 1 petugas bisa signifikan mengurangi waktu tunggu

---

## 13. Mode Presentasi

- **Stop time:** 30-40 menit (perlihatkan pola antrean)
- **Fokus kamera 3D:** area helpdesk + antrean + rak buku
- **Speed slider:** geser ke kiri agar gerakan natural
- Bisa komentar: "30% mahasiswa butuh bantuan — ini menunjukkan pentingnya helpdesk di perpustakaan"

---

## 14. Checklist Final

- [ ] **PengunjungPed**: variabel `idPed`, `tMasuk`, `butuhHelpdesk`, `statusBuku`, `waktuMulaiAntre`, `waktuMulaiLayanan`, `waktuSelesaiLayanan`, `waktuTunggu`, `waktuLayanan`
- [ ] **Variabel Main**: 11 variabel statistik (tabel 1.6)
- [ ] **Markup**: `entryLine`, `exitLine`, `nodeHelpdesk`, `nodeRakBuku`, `nodePinjam`, paths, `areaAntreHelpdesk`, `svcHelpdesk`
- [ ] **3D**: lantai, meja helpdesk, komputer, kursi, rak buku, dinding, kamera
- [ ] **Flowchart**: `srcMahasiswa → selectButuhBantuan → goToHelpdesk → antreHelpdesk → srvHelpdesk → selectStatusBuku → goToRakBuku/goToPinjam → snkSelesai`
- [ ] **selectButuhBantuan**: 70/30
- [ ] **selectStatusBuku**: 40/35/25 ✅ total 1.0
- [ ] **srvHelpdesk.Delay time** = `triangular(2, 4, 6)`
- [ ] **Fungsi**: `avgWaktuTunggu`, `avgWaktuLayanan`, `avgWaktuSistem`, `tingkatKeberhasilanHelpdesk`
- [ ] **Dashboard**: 6 text dinamis
- [ ] **Test 60 menit**: semua route berfungsi
- [ ] **3 skenario**: Normal, Ramai, Perbaikan
