# Tutorial AnyLogic Dari Nol
## Fitur Peminjaman Buku di Counter Perpustakaan

Dokumen ini sudah dirapikan khusus untuk pemula total.
Ikuti urutan langkah dari atas ke bawah.

## Ringkasan Hasil Akhir
Model yang akan dibuat:

Mahasiswa datang -> antre di counter -> dilayani petugas -> selesai.

Blok yang dipakai:
- Source
- Queue
- Service
- Sink

Metrik yang dihasilkan:
- Total mahasiswa selesai
- Rata-rata waktu tunggu antrean
- Rata-rata waktu layanan
- Rata-rata waktu total di sistem
- Panjang antrean maksimum
- Persentase error scanner

---

## 1. Persiapan

### 1.1 Yang dibutuhkan
- AnyLogic (PLE atau versi lain)
- Satuan waktu model: minute
- Durasi simulasi utama: 480 menit (8 jam)

### 1.2 Konsep cepat
- Entity: objek yang bergerak di simulasi (di sini: Mahasiswa)
- Queue: tempat menunggu
- Service: proses dilayani di counter
- Sink: proses selesai, entity keluar

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. Klik File -> New -> Model.
3. Isi Model name: PerpustakaanCounter.
4. Pilih Time units: minute.
5. Klik Finish.

Setelah itu, Anda akan melihat agent utama bernama Main.

---

## 3. Buat Agent Type Mahasiswa

### 3.1 Buat agent baru
1. Di panel Projects (kiri), klik kanan nama model.
2. Pilih New -> Agent Type.
3. Isi Name: Mahasiswa.
4. Klik Finish.

### 3.2 Tambahkan atribut (variable) di agent Mahasiswa
Cara tambah variable:
1. Buka agent Mahasiswa.
2. Dari Palette, bagian Agent, drag Variable ke canvas.
3. Ulangi sampai semua atribut di bawah dibuat.

Gunakan daftar berikut:

| Nama Atribut | Type | Initial Value | Fungsi |
|---|---|---|---|
| idMahasiswa | String | "" | ID unik mahasiswa |
| jumlahBuku | int | 1 | Jumlah buku dipinjam |
| tipePinjaman | String | "REGULER" | REGULER/REFERENSI/RESERVE |
| deadlineHari | int | 7 | Deadline pengembalian (hari) |
| scannerError | boolean | false | Penanda error barcode/scanner |
| nomorResi | String | "" | Nomor bukti checkout |
| tMasukSistem | double | 0 | Waktu masuk proses |
| tMulaiLayanan | double | 0 | Waktu mulai dilayani |
| tSelesaiLayanan | double | 0 | Waktu layanan selesai |
| waktuTunggu | double | 0 | Lama menunggu antrean |
| waktuLayanan | double | 0 | Lama layanan counter |

---

## 4. Buat Variabel Global di Main

Buka Main, lalu tambahkan Variable untuk rekap statistik:

| Nama Variabel | Type | Initial Value |
|---|---|---|
| seqMahasiswa | int | 0 |
| totalSelesai | int | 0 |
| totalBukuDipinjam | int | 0 |
| totalErrorScanner | int | 0 |
| totalWaktuTunggu | double | 0 |
| totalWaktuLayanan | double | 0 |
| totalWaktuSistem | double | 0 |
| maxQueueObserved | double | 0 |

---

## 5. Buat Function Bantu di Main

Tambahkan 5 Function di Main.

### 5.1 avgWaktuTunggu
- Return type: double

```java
return totalSelesai == 0 ? 0 : totalWaktuTunggu / totalSelesai;
```

### 5.2 avgWaktuLayanan
- Return type: double

```java
return totalSelesai == 0 ? 0 : totalWaktuLayanan / totalSelesai;
```

### 5.3 avgWaktuSistem
- Return type: double

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

### 5.4 errorScannerRate
- Return type: double

```java
return totalSelesai == 0 ? 0 : (100.0 * totalErrorScanner / totalSelesai);
```

### 5.5 hitungWaktuService
- Return type: double
- Parameters: `Agent ag`

```java
Mahasiswa a = (Mahasiswa) ag;
double durasiDasar;

if (a.jumlahBuku <= 2) {
    durasiDasar = uniform(2, 3);
} else if (a.jumlahBuku <= 4) {
    durasiDasar = uniform(3, 4);
} else {
    durasiDasar = uniform(4, 6);
}

// 5% kemungkinan error scanner
a.scannerError = false;
if (uniform(0, 1) < 0.05) {
    durasiDasar += uniform(2, 3);
    a.scannerError = true;
}

return durasiDasar;
```

Catatan penting:
- Di editor Function AnyLogic, isi kode body saja (jangan tulis deklarasi method lengkap).
- Pastikan parameter benar-benar ditambahkan di Properties function dengan nama `ag`.

---

## 6. Buat Flow Proses di Main

### 6.1 Drag blok
Dari Process Modeling Library, drag blok:
- Source
- Queue
- Service
- Sink

Susun kiri ke kanan dan hubungkan semua port.

### 6.2 Rename blok
Ganti nama blok menjadi:
- srcMahasiswa
- qCheckout
- srvCheckout
- snkSelesai

Catatan penting: Nama ini harus sama persis dengan kode tutorial.

---

## 7. Konfigurasi Source (Kedatangan Mahasiswa)

Klik srcMahasiswa.

### 7.1 Properti utama
- New agent: Mahasiswa

### 7.2 Pola kedatangan
- Arrivals defined by: Interarrival time
- Interarrival time: exponential(10)

Artinya rata-rata 1 mahasiswa datang setiap 10 menit.

### 7.3 Action di Source -> On exit
Tempel kode ini:

```java
seqMahasiswa++;
agent.idMahasiswa = "M-" + seqMahasiswa;
agent.tMasukSistem = time();

// Random jumlah buku: 1 sampai 5
agent.jumlahBuku = uniform_discr(1, 5);

// Tipe pinjaman: 70% REGULER, 20% REFERENSI, 10% RESERVE
double r = uniform(0, 1);
if (r < 0.7) {
    agent.tipePinjaman = "REGULER";
    agent.deadlineHari = 7;
} else if (r < 0.9) {
    agent.tipePinjaman = "REFERENSI";
    agent.deadlineHari = 3;
} else {
    agent.tipePinjaman = "RESERVE";
    agent.deadlineHari = 1;
}

traceln("ARRIVE " + agent.idMahasiswa
    + " | buku=" + agent.jumlahBuku
    + " | tipe=" + agent.tipePinjaman);
```

---

## 8. Konfigurasi Queue

Klik qCheckout.

Set dasar:
- Biarkan default dulu

Opsional:
- Maximum capacity: 50

Untuk pemula, default sudah cukup.

---

## 9. Konfigurasi Service (Counter Peminjaman)

Klik srvCheckout.

### 9.1 Kapasitas counter
- Capacity: 2

### 9.2 Action di Service -> On enter
Tempel:

```java
agent.tMulaiLayanan = time();
agent.waktuTunggu = agent.tMulaiLayanan - agent.tMasukSistem;
```

### 9.3 Service time -> Expression
Tempel:

```java
hitungWaktuService(agent)
```

Catatan penting:
- Jika Anda mengetik kode ini di tab Actions (On enter/On exit), AnyLogic akan menganggapnya `void`, sehingga `return` memicu error.
- Pastikan kode `return ...` hanya ada di function `hitungWaktuService` (langkah 5.5), bukan di Actions.

### 9.4 Action di Service -> On exit
Tempel:

```java
agent.tSelesaiLayanan = time();
agent.waktuLayanan = agent.tSelesaiLayanan - agent.tMulaiLayanan;

agent.nomorResi = "REC-" + (int)(time() * 1000) + "-" + agent.idMahasiswa;

traceln("DONE " + agent.idMahasiswa
    + " | wait=" + agent.waktuTunggu
    + " | service=" + agent.waktuLayanan
    + " | deadline=" + agent.deadlineHari + " hari"
    + " | scannerError=" + agent.scannerError);
```

---

## 10. Konfigurasi Sink (Rekap Statistik)

Klik snkSelesai.

### Action di Sink -> On enter
Tempel:

```java
totalSelesai++;
totalBukuDipinjam += agent.jumlahBuku;
if (agent.scannerError) {
    totalErrorScanner++;
}

totalWaktuTunggu += agent.waktuTunggu;
totalWaktuLayanan += agent.waktuLayanan;
totalWaktuSistem += (agent.tSelesaiLayanan - agent.tMasukSistem);

if (qCheckout.size() > maxQueueObserved) {
    maxQueueObserved = qCheckout.size();
}
```

---

## 11. Tambahkan Dashboard Sederhana

Di Main, dari Palette Presentation, tambahkan beberapa Text (dynamic).

Gunakan expression berikut:

```java
"Total selesai: " + totalSelesai
```

```java
"Rata-rata tunggu: " + avgWaktuTunggu() + " menit"
```

```java
"Rata-rata layanan: " + avgWaktuLayanan() + " menit"
```

```java
"Rata-rata waktu sistem: " + avgWaktuSistem() + " menit"
```

```java
"Queue saat ini: " + qCheckout.size()
```

```java
"Max queue: " + maxQueueObserved
```

```java
"Error scanner rate: " + errorScannerRate() + "%"
```

Opsional (jika method tersedia):

```java
"Utilisasi counter: " + (100 * srvCheckout.utilization()) + "%"
```

Jika baris utilisasi error di versi AnyLogic Anda, hapus text itu dulu.

---

## 12. Jalankan Simulasi

### 12.1 Tes cepat (60 menit)
1. Buka experiment Simulation.
2. Set Stop time = 60.
3. Klik Run.

Cek hasil:
- Entity bergerak dari Source ke Sink
- Queue kadang terisi
- Console menampilkan log ARRIVE dan DONE

### 12.2 Simulasi penuh (480 menit)
1. Set Stop time = 480.
2. Klik Run.
3. Catat semua metrik.

Perkiraan hasil wajar:
- Total selesai sekitar 40-60 mahasiswa
- Rata-rata tunggu masih masuk akal
- Max queue tidak terus naik tanpa turun

---

## 13. Skenario Uji Untuk Laporan

Jalankan 3 skenario ini:

### Skenario A (Normal)
- Interarrival: exponential(10)
- Capacity service: 2

### Skenario B (Ramai)
- Interarrival: exponential(7)
- Capacity service: 2

### Skenario C (Perbaikan layanan)
- Interarrival: exponential(7)
- Capacity service: 3

Bandingkan:
- avgWaktuTunggu
- maxQueueObserved
- totalSelesai
- errorScannerRate

Interpretasi umum:
- Kedatangan makin rapat -> antrean naik
- Counter ditambah -> waktu tunggu turun

---

## 14. Troubleshooting Cepat

### Masalah 1: agent.idMahasiswa merah
Penyebab: atribut belum dibuat di agent Mahasiswa.
Solusi: buat variable idMahasiswa di Mahasiswa.

### Masalah 2: totalSelesai tetap 0
Penyebab: kode On enter di snkSelesai kosong/salah.
Solusi: tempel ulang kode rekap statistik di Sink.

### Masalah 3: Entity tidak bergerak ke kanan
Penyebab: blok belum terhubung.
Solusi: cek koneksi srcMahasiswa -> qCheckout -> srvCheckout -> snkSelesai.

### Masalah 4: Queue terlalu panjang
Penyebab: arrival terlalu rapat atau capacity kecil.
Solusi: ubah interarrival ke exponential(12) atau capacity jadi 3.

### Masalah 5: Error di utilization
Penyebab: perbedaan versi/method.
Solusi: hapus dulu text utilisasi, lanjut pakai metrik lain.

---

## 15. Checklist Final

- Agent Mahasiswa dan semua atribut sudah dibuat
- Flow Source -> Queue -> Service -> Sink sudah terhubung
- Kode Source On exit sudah terpasang
- Kode Service On enter, Service time, On exit sudah terpasang
- Kode Sink On enter sudah terpasang
- Dashboard minimal 4 metrik sudah tampil
- Simulasi tes 60 menit berhasil
- Simulasi 480 menit berhasil
- Tiga skenario pembanding sudah dijalankan

---

## 16. Data Output Untuk Integrasi Ke Modul Teman

Data minimal yang dikirim ke modul berikutnya:
- idMahasiswa
- jumlahBuku
- tipePinjaman
- deadlineHari
- nomorResi
- tSelesaiLayanan

Dengan ini fitur peminjaman buku di counter siap diintegrasikan ke modul ruang baca, return, dan denda.

---

## 17. Upgrade Ke Simulasi 3D (Dari Model Yang Sudah Jalan)

Bagian ini khusus untuk kebutuhan presentasi 3D.
Anda tidak perlu mengulang model dari awal; cukup upgrade model saat ini.

### 17.1 Pilih Mode 3D Yang Aman Untuk Pemula

Gunakan mode berikut dulu:
- Logika proses tetap: Source -> Queue -> Service -> Sink
- Tambah visual ruangan 3D + kamera 3D
- Tambah avatar 3D untuk agent Mahasiswa

Ini paling stabil untuk tugas kuliah karena:
- model tidak rusak,
- metrik tetap konsisten,
- tampilan jadi 3D saat demo.

### 17.2 Tambah Jendela 3D

1. Buka agent Main.
2. Dari Palette -> Presentation, drag elemen `3D Window` ke canvas.
3. Rename menjadi `win3D`.
4. Atur ukuran jendela lebih besar agar scene mudah dilihat.

### 17.3 Tambah Kamera 3D

1. Dari Palette -> Presentation, drag `Camera` ke Main.
2. Rename: `camMain`.
3. Di Properties kamera, atur posisi sampai area counter terlihat jelas.

Contoh posisi awal (silakan sesuaikan):
- X = 25
- Y = -20
- Z = 16

### 17.4 Bangun Ruang Perpustakaan 3D

Di Main, dari Palette -> 3D Objects, tambahkan objek ini:

1. Lantai (Box)
- Name: `floor3D`
- Size kira-kira: 30 x 20 x 0.2

2. Counter peminjaman (Box)
- Name: `counter3D`
- Size kira-kira: 6 x 1.2 x 1.1

3. Rak buku (beberapa Box)
- Name: `shelfA`, `shelfB`, `shelfC`

4. Kursi/meja (opsional)
- Tambah Box kecil untuk detail visual

Tips:
- Fokus dulu area antrean + counter.
- Tidak perlu terlalu detail agar model tetap ringan.

### 17.5 Buat Avatar 3D Untuk Mahasiswa

Masuk ke agent `Mahasiswa`, lalu tambahkan shape 3D sederhana:

Opsi paling mudah:
- Cylinder (badan)
- Sphere (kepala)

Alternatif:
- Jika tersedia, gunakan objek manusia siap pakai dari 3D Objects.

Tujuan tahap ini:
- setiap entity mahasiswa terlihat sebagai objek 3D, bukan titik biasa.

### 17.6 Sinkronkan Logika Proses Dengan Posisi 3D (Walking Halus, Tetap Stabil)

Supaya agent terlihat berjalan (bukan teleport), pakai aturan ini:
- `setXYZ(...)` hanya untuk posisi awal saat agent baru dibuat.
- Perpindahan antar fase proses pakai `moveToInTime(...)`.
- Tambahkan 1 blok `Delay` sebelum `Sink` agar gerak keluar sempat terlihat.

#### Langkah A: Sisipkan blok Delay sebelum Sink

Di flow, ubah menjadi:

`srcMahasiswa -> qCheckout -> srvCheckout -> dlyKeluar -> snkSelesai`

Konfigurasi `dlyKeluar`:
- Delay time: `1.2` menit (boleh 0.8-1.5 sesuai selera visual)

#### Langkah B: Atur gerak dari Source ke area antre

Di `srcMahasiswa` -> On exit, tambahkan:

```java
agent.setXYZ(2, 3, 0);           // spawn awal
agent.moveToInTime(10, 6, 0, 1); // jalan ke area antre
```

#### Langkah C: Bentuk baris antre yang terlihat rapi

Di `qCheckout` -> On enter, tambahkan:

```java
double idx = Math.max(0, qCheckout.size() - 1);
idx = Math.min(idx, 8);

double xAntre = 10 - idx * 0.9;
double yAntre = 6;

agent.moveToInTime(xAntre, yAntre, 0, 0.6);
```

#### Langkah D: Gerakkan ke counter saat mulai dilayani

Di `srvCheckout` -> On enter, tambahkan baris ini di paling atas:

```java
agent.moveToInTime(16, 6, 0, 0.8);
```

Lalu biarkan kode statistik yang sudah ada tetap dipakai:

```java
agent.tMulaiLayanan = time();
agent.waktuTunggu = agent.tMulaiLayanan - agent.tMasukSistem;
```

#### Langkah E: Gerak keluar sebelum benar-benar selesai

Di `dlyKeluar` -> On enter, tambahkan:

```java
agent.moveToInTime(26, 3, 0, 1.0);
```

Di `snkSelesai` -> On enter:
- Jangan isi `setXYZ(...)` lagi.
- Biarkan hanya kode rekap statistik (yang sudah dibuat di langkah 10).

### 17.7 Kenapa Perlu Delay Sebelum Sink

`Sink` langsung mengeluarkan agent dari model.
Kalau animasi keluar dipasang di `Sink`, gerakan sering tidak sempat terlihat.

Dengan `dlyKeluar`, alurnya jadi:
- selesai dilayani,
- jalan ke pintu keluar,
- baru hilang dari model.

Hasil visual jauh lebih natural saat presentasi.

### 17.8 Fallback Jika moveToInTime Tidak Tersedia

Pada beberapa versi AnyLogic lama, jika `moveToInTime(...)` tidak dikenali:
- ganti sementara dengan `moveTo(...)` untuk perpindahan visual,
- jangan gunakan `setXYZ(...)` untuk perpindahan dinamis.

Contoh fallback sederhana:

```java
agent.moveTo(16, 6, 0);
```

Jika model Anda saat ini sudah mengenali `moveToInTime(...)`, tetap gunakan itu karena durasi gerak bisa dikontrol lebih presisi.

### 17.9 Checklist 3D

- `win3D` tampil saat run
- Kamera mengarah ke area counter
- Objek ruangan (lantai, counter, rak) terlihat
- Agent mahasiswa terlihat sebagai objek 3D
- Posisi agent berubah sesuai fase proses (masuk -> antre -> counter -> keluar)
- Metrik statistik tetap berjalan normal

Jika semua poin di atas lolos, model Anda sudah 3D dan siap demo.

### 17.10 Mode Presentasi (Agar Tidak Terasa Terlalu Cepat)

Saat demo, kecepatan simulasi sering terasa terlalu cepat jika dibiarkan default.

Gunakan pengaturan ini:
- Stop time presentasi: 90-120 menit (cukup untuk menunjukkan pola antrean).
- Speed slider runtime: geser ke kiri sampai gerakan antar titik terlihat jelas.
- Fokus kamera ke area antre + counter, jangan terlalu zoom out.

Tips tambahan:
- Jika antrean terlalu padat saat demo, ubah sementara interarrival ke `exponential(12)`.
- Setelah demo, kembalikan ke skenario asli untuk hasil analisis laporan.
