# Tutorial AnyLogic Pemula Total
## Simulasi 3D Berjalan Nyata: Datang -> Antri -> Service -> Pergi

Dokumen ini dibuat ulang dari nol untuk kasus kamu: model harus benar-benar terlihat berjalan, antri, dilayani di counter, lalu pergi.

Status verifikasi dokumentasi resmi AnyLogic: dicek pada 17 April 2026.
Referensi yang dipakai ada di bagian akhir dokumen.

---

## 0. Kenapa Simulasi Sebelumnya Terasa Aneh

Masalah paling umum:

1. Model memakai Process Modeling biasa (Source/Queue/Service/Sink) lalu posisi diubah pakai `setXYZ()`.
2. Perpindahan posisi dilakukan dengan "lompat titik" (teleport), bukan alur pedestrian fisik.
3. Queue visual tidak sinkron dengan logika service.

Dari dokumentasi Agent API AnyLogic:
- `setXYZ()` untuk inisialisasi posisi, bukan untuk perpindahan dinamis saat runtime.
- Perpindahan dinamis seharusnya via movement API, atau lebih aman untuk kasus orang berjalan: pakai Pedestrian Library.

Karena itu, tutorial ini pakai Pedestrian Library penuh agar gerak 3D natural.

---

## 1. Hasil Akhir Yang Akan Kamu Dapat

Setelah selesai, modelmu akan punya alur:

1. Mahasiswa muncul di pintu masuk.
2. Mahasiswa berjalan ke area counter.
3. Jika counter sibuk, mahasiswa otomatis antri di garis queue.
4. Saat service, data peminjaman buku diproses (jumlah buku, tipe pinjaman, deadline, nomor resi).
5. Jika terjadi error scanner, waktu service otomatis bertambah.
6. Setelah layanan selesai, mahasiswa berjalan ke area keluar.
7. Mahasiswa keluar model.

Flowchart utama:

`PedSource -> PedService -> PedWait -> PedSink`

Markup utama:

- `entryLine` (garis masuk)
- `svcCounter` (Service with Lines: service point + queue line)
- `exitLine` (garis keluar)

---

## 2. Komponen Yang Dipakai

### 2.1 Library

- Pedestrian Library
- Presentation (untuk 3D window, kamera, objek ruangan)

### 2.2 Blok flowchart

- `srcMasuk` (PedSource)
- `srvCounter` (PedService)
- `wJalanKeluar` (PedWait)
- `snkSelesai` (PedSink)

### 2.3 Agent pedestrian custom

- `MahasiswaPed` (Pedestrian Type custom)

### 2.4 Variabel statistik di Main

- `seqPed` (int)
- `totalSelesai` (int)
- `totalWaktuSistem` (double)
- `maxQueueCounter` (int)
- `totalBukuDipinjam` (int)
- `totalErrorScanner` (int)

---

## 3. Buat Project Baru

1. Buka AnyLogic.
2. Klik File -> New -> Model.
3. Nama model: `Perpustakaan3D_AntriCounter`.
4. Time units: `minute`.
5. Klik Finish.

Catatan:
- Semua nilai waktu di tutorial ini mengikuti satuan `minute`.

---

## 4. Buat Pedestrian Type Custom (Untuk Avatar 3D)

Langkah ini mengikuti dokumentasi "Creating custom pedestrian types".

1. Buka `Main`.
2. Dari Palette Pedestrian Library, drag elemen `Pedestrian Type` ke canvas.
3. Wizard akan muncul.
4. Isi nama: `MahasiswaPed`.
5. Pilih animasi 3D (pilih model orang yang tersedia di daftar bawaan AnyLogic).
6. Klik Next sampai selesai.

Setelah tipe dibuat, buka diagram `MahasiswaPed` lalu tambah Variable berikut:

| Nama | Type | Initial value |
|---|---|---|
| idPed | String | "" |
| jumlahBuku | int | 1 |
| tipePinjaman | String | "REGULER" |
| deadlineHari | int | 7 |
| nomorResi | String | "" |
| scannerError | boolean | false |
| tMasuk | double | 0 |
| tMulaiService | double | 0 |
| tSelesaiService | double | 0 |

Tips pemula:
- Di flowchart Pedestrian, variabel lokal yang dipakai biasanya `ped`, bukan `agent`.
- Karena `srcMasuk` nanti diset menghasilkan `MahasiswaPed`, kamu bisa akses `ped.idPed`, `ped.tMasuk`, dst.

---

## 5. Siapkan Layout 3D Di Main

### 5.1 Tambah 3D window dan kamera

1. Dari Palette Presentation, drag `3D Window` ke `Main`. Rename: `win3D`.
2. Drag `Camera`. Rename: `camMain`.
3. Atur posisi kamera sampai area counter terlihat jelas.

Contoh posisi awal kamera (boleh disesuaikan):
- X = 22
- Y = -16
- Z = 14

### 5.2 Tambah objek ruangan sederhana

Dari 3D Objects:

1. Tambah `Box` untuk lantai. Rename: `floor3D`.
2. Tambah `Box` untuk meja counter. Rename: `counter3D`.
3. Tambah beberapa `Box` untuk rak buku agar scene terlihat perpustakaan.

Tidak perlu terlalu detail dulu. Fokus dulu jalur masuk, area antri, dan area keluar.

---

## 6. Buat Markup Pedestrian (Bagian Paling Penting)

### 6.1 Garis masuk

1. Dari Space Markup (Pedestrian Library), tambah `Target line`.
2. Rename: `entryLine`.
3. Letakkan di area depan pintu masuk.

### 6.2 Service with Lines (counter + queue)

1. Dari Space Markup, drag `Service with Lines`.
2. Rename: `svcCounter`.
3. Atur lokasi service point dekat meja counter.
4. Buka properties `svcCounter`:
- Number of services = 2
- N of queues = 2 (mode pemula paling mudah)

Catatan:
- Queue line harus mengarah dari area kedatangan ke service point.
- Pastikan queue line tidak tertutup objek yang menjadi penghalang jalan.

### 6.3 Garis keluar

1. Tambah `Target line` lagi.
2. Rename: `exitLine`.
3. Letakkan di sisi ruangan yang akan jadi pintu keluar.

---

## 7. Bangun Flowchart Pedestrian

Di `Main`, dari Pedestrian Library, drag blok berikut dan hubungkan berurutan:

1. `PedSource` -> rename `srcMasuk`
2. `PedService` -> rename `srvCounter`
3. `PedWait` -> rename `wJalanKeluar`
4. `PedSink` -> rename `snkSelesai`

Hubungkan port:

`srcMasuk.out -> srvCounter.in -> wJalanKeluar.in -> snkSelesai.in`

Kenapa ada `PedWait` sebelum sink:
- Supaya setelah service, pedestrian benar-benar berjalan ke area keluar dulu.
- Kalau langsung ke sink, sering terasa terlalu cepat hilangnya.

---

## 8. Konfigurasi Detail Setiap Blok

## 8.1 `srcMasuk` (PedSource)

Atur properti utama:

- Appears at: `line`
- Target line: `entryLine`
- Arrive according to: `Interarrival time`
- Interarrival time: `exponential(1.8)`
- New pedestrian: `MahasiswaPed`

Action `On exit`:

```java
seqPed++;
ped.idPed = "M-" + seqPed;
ped.tMasuk = time();

ped.jumlahBuku = uniform_discr(1, 5);

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

traceln("ARRIVE " + ped.idPed
    + " | buku=" + ped.jumlahBuku
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari");
```

## 8.2 `srvCounter` (PedService)

Atur properti utama:

- Services: `svcCounter`
- Queue choice policy: `Shortest queue`
- Delay time: `hitungWaktuServicePeminjaman(ped)`
- Recovery delay: `0`

Action `On enter queue`:

```java
int q = srvCounter.queueSize();
if (q > maxQueueCounter) {
    maxQueueCounter = q;
}
```

Action `On begin service`:

```java
ped.tMulaiService = time();
```

Action `On end service`:

```java
ped.tSelesaiService = time();
ped.nomorResi = "REC-" + (long)(time() * 1000) + "-" + ped.idPed;

traceln("SERVICE " + ped.idPed
    + " | resi=" + ped.nomorResi
    + " | buku=" + ped.jumlahBuku
    + " | scannerError=" + ped.scannerError);
```

## 8.3 `wJalanKeluar` (PedWait)

Atur properti:

- Waiting location: `Target line`
- Target line: `exitLine`
- Delay ends: `On delay time expiry`
- Delay time: `0.2`

Penjelasan:
- Blok ini memaksa pedestrian jalan dulu ke `exitLine`, baru lanjut keluar.
- `0.2` menit cukup untuk menunjukkan transisi keluar tanpa menahan flow terlalu lama.

## 8.4 `snkSelesai` (PedSink)

Action `On enter`:

```java
totalSelesai++;

totalBukuDipinjam += ped.jumlahBuku;
if (ped.scannerError) {
    totalErrorScanner++;
}

double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;

traceln("DONE " + ped.idPed
    + " | tSistem=" + tSistem
    + " | tipe=" + ped.tipePinjaman
    + " | deadline=" + ped.deadlineHari + " hari");
```

---

## 9. Variabel Dan Function Statistik Di Main

Tambahkan Variable berikut di `Main`:

| Nama | Type | Initial value |
|---|---|---|
| seqPed | int | 0 |
| totalSelesai | int | 0 |
| totalWaktuSistem | double | 0 |
| maxQueueCounter | int | 0 |
| totalBukuDipinjam | int | 0 |
| totalErrorScanner | int | 0 |

Tambahkan Function `hitungWaktuServicePeminjaman` di `Main` (Return type: double, parameter: `MahasiswaPed ped`):

```java
double dasar;

if (ped.jumlahBuku <= 2) {
    dasar = uniform(1.5, 2.2);
} else if (ped.jumlahBuku <= 4) {
    dasar = uniform(2.2, 3.2);
} else {
    dasar = uniform(3.2, 4.5);
}

ped.scannerError = false;
if (uniform(0, 1) < 0.05) {
    dasar += uniform(0.8, 1.4);
    ped.scannerError = true;
}

return dasar;
```

Tambahkan Function `avgWaktuSistem` di `Main` (Return type: double):

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

Tambahkan Function `avgBukuPerTransaksi` di `Main` (Return type: double):

```java
return totalSelesai == 0 ? 0 : (1.0 * totalBukuDipinjam / totalSelesai);
```

Tambahkan Function `errorScannerRate` di `Main` (Return type: double):

```java
return totalSelesai == 0 ? 0 : (100.0 * totalErrorScanner / totalSelesai);
```

---

## 10. Dashboard Sederhana Untuk Laporan

Tambahkan Text dinamis di `Main`:

```java
"Total selesai: " + totalSelesai
```

```java
"Avg waktu sistem: " + avgWaktuSistem() + " menit"
```

```java
"Queue saat ini: " + srvCounter.queueSize()
```

```java
"Max queue: " + maxQueueCounter
```

```java
"Total buku dipinjam: " + totalBukuDipinjam
```

```java
"Avg buku/transaksi: " + avgBukuPerTransaksi()
```

```java
"Error scanner rate: " + errorScannerRate() + "%"
```

---

## 11. Jalankan Simulasi Dengan Setting Aman

### 11.1 Uji awal

1. Stop time = 60 menit.
2. Run.
3. Pastikan visual ini terjadi:
- Muncul di `entryLine`
- Jalan ke queue/service
- Menunggu jika service sibuk
- Dilayani
- Jalan ke `exitLine`
- Keluar di sink

### 11.2 Mode presentasi

Agar gerakan terlihat jelas:

1. Pakai stop time 90-120 menit.
2. Turunkan speed slider (jangan maksimum).
3. Fokus kamera ke area queue + counter + exit.

---

## 12. Troubleshooting (Kasus Yang Sering Bikin "Aneh")

### Masalah A: Pedestrian tidak bergerak, numpuk di awal

Penyebab umum:
- `srvCounter.Services` belum diarahkan ke `svcCounter`.
- Markup queue/service ketutup objek penghalang.

Solusi:
1. Cek nama persis `svcCounter` di properties `srvCounter`.
2. Geser objek 3D penghalang.
3. Coba dulu layout simpel tanpa banyak furnitur.

### Masalah B: Pedestrian tidak antri, langsung hilang

Penyebab umum:
- Flow belum lewat `srvCounter` atau `wJalanKeluar`.

Solusi:
- Pastikan urutan blok: `srcMasuk -> srvCounter -> wJalanKeluar -> snkSelesai`.

### Masalah C: Queue tidak terlihat rapi

Penyebab umum:
- Bentuk queue line di `svcCounter` belum diarahkan dengan baik.

Solusi:
1. Klik queue line di markup `svcCounter`.
2. Tarik titik line agar arah antri jelas menuju counter.

### Masalah D: Model error saat kompilasi kode aksi

Penyebab umum:
- Salah pakai `agent` padahal blok pedestrian memakai `ped`.

Solusi:
- Ganti semua akses atribut pedestrian menjadi `ped.namaAtribut`.

### Masalah E: Terasa teleport lagi

Penyebab umum:
- Masih ada kode manual `setXYZ()`/`jumpTo()` di actions lama.

Solusi:
1. Hapus kode pemindahan posisi manual dari blok lama.
2. Biarkan Pedestrian Library yang mengatur gerak fisik.

---

## 13. Checklist Final Sebelum Demo

- `MahasiswaPed` sudah dibuat dan dipakai di `srcMasuk`
- `entryLine`, `svcCounter`, `exitLine` sudah ada
- Flow blok terhubung lengkap sampai sink
- `srvCounter` mengarah ke `svcCounter`
- Pedestrian terlihat jalan, antri, service, lalu jalan keluar
- Statistik dashboard berubah saat run
- Data peminjaman buku terisi (`jumlahBuku`, `tipePinjaman`, `deadlineHari`, `nomorResi`)
- Error scanner sesekali muncul dan memengaruhi waktu service
- Tidak ada kode manual teleport (`setXYZ`, `jumpTo`) di flow pedestrian

Jika semua poin ini lolos, modelmu sudah sesuai permintaan: 3D berjalan nyata, bukan teleport.

---

## 14. Referensi Dokumentasi Resmi Yang Dipakai

- AnyLogic Agent API (`setXYZ`, `jumpTo`, `moveTo`, `moveToInTime`)
- Pedestrian Library overview
- Pedestrian Library blocks
- `PedSource`
- `PedService`
- `PedWait`
- `PedSink`
- Markup `Services` dan `Service with lines`
- `Creating custom pedestrian types`
- `Animating pedestrians`

Catatan penting dari docs:
- Untuk model orang berjalan realistis, gunakan Pedestrian Library + markup pedestrian.
- `setXYZ()` adalah inisialisasi posisi, bukan mekanisme utama perpindahan runtime.
