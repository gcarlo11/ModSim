# Modul 3: Cari Tempat Duduk
## Aktivitas Sebelum Duduk (Toilet, Cari Buku, Loker, Langsung) + Zona Belajar

**Satuan waktu:** minute

---

## 1. Hasil Akhir

Modul ini mensimulasikan mahasiswa yang ingin **belajar di perpustakaan**:

1. Memilih **aktivitas sebelum duduk** (mampir toilet, cari buku, loker, atau langsung)
2. Setelah aktivitas selesai → **pilih zona duduk** (silent zone / diskusi zone)
3. **Duduk dan belajar** (waktu sesuai zona)
4. **Selesai dan keluar**

**Flowchart standalone:**

```
srcMasuk (temporary)
    │
    ▼
selectAktivitas (PedSelectOutput)
  ├── srvToiletSinggah (15%) → toilet dulu 2-4 mnt
  ├── srvCariBuku (30%)      → eksplor rak buku 5-15 mnt
  ├── srvLoker (25%)         → taruh barang 1-3 mnt
  └── langsung (30%)         → langsung cari kursi
    │
    ▼ (semua bergabung di sini)
selectZonaDuduk (PedSelectOutput)
  ├── srvDudukSepi (60%)     → silent zone ~45 mnt belajar
  └── srvDudukDiskusi (40%)  → diskusi zone ~90 mnt belajar
    │
    ▼
snkSelesai (temporary)
```

---

## 2. Komponen

### Blok flowchart

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` (TEMPORARY — untuk standalone) |
| PedSelectOutput | `selectAktivitas` (4 output) |
| PedService | `srvToiletSinggah` |
| PedService | `srvCariBuku` |
| PedService | `srvLoker` |
| PedSelectOutput | `selectZonaDuduk` (2 output) |
| PedService | `srvDudukSepi` |
| PedService | `srvDudukDiskusi` |
| PedSink | `snkSelesai` (TEMPORARY) |

### Markup

| Markup | Nama | Detail |
|---|---|---|
| Service with Lines | `svcToiletSinggah` | 2 toilet, 1 antrean |
| Service with Lines | `svcCariBuku` | 2 area rak buku, 1 antrean per area |
| Service with Lines | `svcLoker` | 3 slot loker, 1 antrean |
| Service with Lines | `svcAreaSepi` | 10 kursi individu, shortest queue |
| Service with Lines | `svcAreaDiskusi` | 4 meja (1 meja = 1 service point), 1 antrean per meja |

### 3D

| Objek | Nama | Posisi (contoh) |
|---|---|---|
| Floor (Box) | `floor3D` | 20x20x0.2 |
| Meja silent | `mejaSepi3D_1` s/d `mejaSepi3D_10` | 10 meja kecil |
| Meja diskusi | `mejaDiskusi3D_1` s/d `mejaDiskusi3D_4` | 4 meja besar |
| Rak buku | `rakBuku3D_1`, `rakBuku3D_2` | Di area cari buku |
| Loker | `loker3D` | 3 bilik loker |

### Variabel baru di PengunjungPed

| Nama | Type | Initial value | Keterangan |
|---|---|---|---|
| `waktuMulaiAktivitas` | double | `0` | Waktu mulai aktivitas (toilet/cari buku/loker) |
| `waktuMulaiDuduk` | double | `0` | Waktu mulai duduk belajar |
| `durasiBelajar` | double | `0` | Durasi belajar (menit) |
| `zonaDuduk` | String | `""` | "SEPI" atau "DISKUSI" |
| `aktivitasSebelumDuduk` | String | `""` | "TOILET", "BUKU", "LOKER", "LANGSUNG" |

### Variabel baru di Main

| Nama | Type | Initial value |
|---|---|---|
| `totalToiletSinggah` | int | 0 |
| `totalCariBuku` | int | 0 |
| `totalLoker` | int | 0 |
| `totalDudukSepi` | int | 0 |
| `totalDudukDiskusi` | int | 0 |
| `maxAntrianDudukSepi` | int | 0 |
| `maxAntrianDudukDiskusi` | int | 0 |
| `totalDurasiBelajar` | double | 0 |

---

## 3. Buat Project

1. **File → New → Model** → nama: `Perpustakaan3D_CariDuduk`.
2. **Time units**: `minute`.
3. Buat **Pedestrian Type**: `PengunjungPed`.
4. Tambah variabel ke `PengunjungPed` sesuai tabel di atas (waktuMulaiAktivitas, waktuMulaiDuduk, durasiBelajar, zonaDuduk, aktivitasSebelumDuduk).

> **Penting untuk pemula:** Variabel ditambahkan di diagram `PengunjungPed`. Buka tab diagram agent tersebut, lalu drag **Variable** dari palette **Agent**.

---

## 4. Siapkan Layout 3D

1. Drag **3D Window** → `win3D`.
2. Drag **Camera** → `camMain`.
3. Drag **Box** → `floor3D`, skala 20x20x0.2, posisi (0,0,0).
4. Buat **Box** untuk meja (atau gunakan **RoundedBox** dari palette 3D):
   - 10 meja kecil untuk silent zone: berjajar rapi di sisi kiri
   - 4 meja besar untuk diskusi zone: berkelompok di sisi kanan
   - 2 rak buku: di area tengah
   - 1 loker: di dekat pintu (entryLine)

---

## 5. Buat Markup

### 5.1 Service with Lines — Toilet Singgah

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcToiletSinggah`.
3. **Number of services** = `2` (dua bilik toilet).
4. **N of queues** = `1` (satu antrean).

### 5.2 Service with Lines — Cari Buku

1. **Service with Lines**.
2. Rename: `svcCariBuku`.
3. **Number of services** = `2` (dua area rak).
4. **N of queues** = `2`.

### 5.3 Service with Lines — Loker

1. **Service with Lines**.
2. Rename: `svcLoker`.
3. **Number of services** = `3` (3 bilik loker).
4. **N of queues** = `1`.

### 5.4 Service with Lines — Silent Zone

1. **Service with Lines**.
2. Rename: `svcAreaSepi`.
3. **Number of services** = `10` (10 kursi).
4. **N of queues** = `2` (2 antrean, shortest queue).

### 5.5 Service with Lines — Diskusi Zone

1. **Service with Lines**.
2. Rename: `svcAreaDiskusi`.
3. **Number of services** = `4` (4 meja).
4. **N of queues** = `2` (2 antrean, shortest queue).

### 5.6 Layout markup (contoh posisi)

```
entryLine                                           exitLine
  |                                                    ^
  |  [svcToilet]  [svcLoker]                           |
  |  [svcCariBuku]                                     |
  |     |                                              |
  |     v                                              |
  |  [svcAreaSepi ———— 10 kursi ———]  [svcAreaDiskusi] |
  +-------------------------------------------------->+
```

---

## 6. Bangun Flowchart dan Koneksi

### 6.1 Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` (TEMPORARY) |
| PedSelectOutput | `selectAktivitas` (4 outputs) |
| PedService | `srvToiletSinggah` |
| PedService | `srvCariBuku` |
| PedService | `srvLoker` |
| PedSelectOutput | `selectZonaDuduk` (2 outputs) |
| PedService | `srvDudukSepi` |
| PedService | `srvDudukDiskusi` |
| PedSink | `snkSelesai` (TEMPORARY) |

### 6.2 Koneksi

```
srcMasuk.out → selectAktivitas.in

selectAktivitas.out1 (15%) → srvToiletSinggah.in
selectAktivitas.out2 (30%) → srvCariBuku.in
selectAktivitas.out3 (25%) → srvLoker.in
selectAktivitas.out4 (30%) → langsung → selectZonaDuduk.in

srvToiletSinggah.out → selectZonaDuduk.in  (bergabung)
srvCariBuku.out      → selectZonaDuduk.in  (bergabung)
srvLoker.out         → selectZonaDuduk.in  (bergabung)

selectZonaDuduk.out1 (60%) → srvDudukSepi.in
selectZonaDuduk.out2 (40%) → srvDudukDiskusi.in

srvDudukSepi.out    → snkSelesai.in
srvDudukDiskusi.out → snkSelesai.in
```

> **Catatan penting:** Di AnyLogic, beberapa blok bisa terhubung ke 1 input yang sama. Jadi `srvToiletSinggah.out`, `srvCariBuku.out`, `srvLoker.out`, dan `selectAktivitas.out4` semuanya bisa colok ke `selectZonaDuduk.in`.

---

## 7. Konfigurasi Detail Setiap Blok

### 7.1 `srcMasuk` (PedSource — TEMPORARY)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` (buat sendiri untuk standalone) |
| Interarrival time | `exponential(2.0)` |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "D-" + seqPed;
ped.tMasuk = time();
```

### 7.2 `selectAktivitas` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `4` |
| Use probabilities | Centang |
| Output 1 | `0.15` (ke toilet) |
| Output 2 | `0.30` (cari buku) |
| Output 3 | `0.25` (loker) |
| Output 4 | `0.30` (langsung) |

### 7.3 `srvToiletSinggah` (PedService — mampir toilet)

| Property | Value |
|---|---|
| Services | `svcToiletSinggah` |
| Delay time | `hitungWaktuToiletSinggah(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "TOILET";
```

**Action On end service:**
```java
totalToiletSinggah++;
```

### 7.4 `srvCariBuku` (PedService — eksplor rak)

| Property | Value |
|---|---|
| Services | `svcCariBuku` |
| Delay time | `hitungWaktuCariBuku(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "BUKU";
```

**Action On end service:**
```java
totalCariBuku++;
```

### 7.5 `srvLoker` (PedService — simpan barang)

| Property | Value |
|---|---|
| Services | `svcLoker` |
| Delay time | `hitungWaktuLoker(ped)` |

**Action On begin service:**
```java
ped.waktuMulaiAktivitas = time();
ped.aktivitasSebelumDuduk = "LOKER";
```

**Action On end service:**
```java
totalLoker++;
```

### 7.6 `selectZonaDuduk` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `2` |
| Use probabilities | Centang |
| Output 1 | `0.60` (silent zone) |
| Output 2 | `0.40` (diskusi zone) |

### 7.7 `srvDudukSepi` (PedService — belajar di silent zone)

| Property | Value |
|---|---|
| Services | `svcAreaSepi` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungDurasiBelajar(ped)` |

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
```

**Action On enter queue:**
```java
int q = srvDudukSepi.queueSize();
if (q > maxAntrianDudukSepi) {
    maxAntrianDudukSepi = q;
}
```

### 7.8 `srvDudukDiskusi` (PedService — belajar di diskusi zone)

| Property | Value |
|---|---|
| Services | `svcAreaDiskusi` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungDurasiBelajar(ped)` |

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
    + " | zona=" + ped.zonaDuduk
    + " | durasi=" + String.format("%.1f", ped.durasiBelajar) + " mnt");
```

**Action On enter queue:**
```java
int q = srvDudukDiskusi.queueSize();
if (q > maxAntrianDudukDiskusi) {
    maxAntrianDudukDiskusi = q;
}
```

### 7.9 `snkSelesai` (PedSink — TEMPORARY)

Action On enter:
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

## 8. Fungsi di Main

### hitungWaktuToiletSinggah

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
return uniform(2.0, 4.0);
```

### hitungWaktuCariBuku

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
return uniform(5.0, 15.0);
```

### hitungWaktuLoker

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
return uniform(1.0, 3.0);
```

### hitungDurasiBelajar

Return type: `double`. Parameter: `PengunjungPed ped`.

```java
if (ped.zonaDuduk.equals("SEPI")) {
    // Silent zone: belajar individual ~45 menit
    return exponential(45.0);
} else {
    // Diskusi zone: belajar kelompok ~90 menit
    return exponential(90.0);
}
```

> **Penjelasan exponential:** `exponential(45.0)` = rata-rata 45 menit, tapi bisa lebih pendek atau lebih panjang. Ini lebih realistis daripada waktu tetap.

---

## 9. Dashboard

Tambahkan Text dinamis di Main:

```
"Total belajar: " + (totalDudukSepi + totalDudukDiskusi)
```

```
"Silent zone: " + totalDudukSepi + " | Diskusi: " + totalDudukDiskusi
```

```
"Toilet: " + totalToiletSinggah + " | Cari Buku: " + totalCariBuku + " | Loker: " + totalLoker
```

```
"Antrian silent: " + srvDudukSepi.queueSize() + " (max: " + maxAntrianDudukSepi + ")"
```

```
"Antrian diskusi: " + srvDudukDiskusi.queueSize() + " (max: " + maxAntrianDudukDiskusi + ")"
```

```
"Rata-rata durasi: " + String.format("%.1f", (totalDudukSepi+totalDudukDiskusi)==0 ? 0 : totalDurasiBelajar/(totalDudukSepi+totalDudukDiskusi)) + " mnt"
```

---

## 10. Jalankan untuk Uji Coba

1. Set **Stop time** = 60 menit.
2. Pastikan semua outlet selectAktivitas terkoneksi.
3. Run.
4. Harus terlihat: pengunjung muncul → pilih aktivitas → antri → duduk → belajar → keluar.
5. Cek console: `BELAJAR D-1 | zona=SEPI | durasi=38.5 mnt`, `DONE D-1 | ...`.

---

## 11. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → ganti dengan koneksi dari `selectTujuan.out3` (skeleton).
2. **Hapus** `snkSelesai` (temporary) → colok `srvDudukSepi.out` dan `srvDudukDiskusi.out` ke `wJalanKeluar.in` (skeleton).

---

## 12. Troubleshooting

### Masalah: Semua langsung belajar, tidak ada aktivitas

Cek `selectAktivitas` properties → pastikan **Use probabilities** dicentang dan 4 probabilities terisi.

### Masalah: Antrian di silent zone selalu penuh

Karena hanya 10 kursi dengan 2 antrean, wajar jika peak hour antrian panjang. Coba perbesar `svcAreaSepi.Number of services` menjadi 12-15.

### Masalah: Service time terlalu lama

`exponential(90.0)` untuk diskusi zone bisa menghasilkan waktu >2 jam. Untuk testing 30 menit, turunkan sementara jadi `exponential(30.0)`.

---

## 13. Checklist Final

- [ ] Semua blok dan markup sudah dibuat sesuai tabel
- [ ] `selectAktivitas` 4 output dengan probabilitas (15/30/25/30)
- [ ] `selectZonaDuduk` 2 output dengan probabilitas (60/40)
- [ ] `srvDudukSepi` menggunakan `svcAreaSepi` (10 kursi)
- [ ] `srvDudukDiskusi` menggunakan `svcAreaDiskusi` (4 meja)
- [ ] Semua fungsi sudah dibuat di Main
- [ ] Statistik tercatat di console
- [ ] Dashboard menampilkan angka real-time
- [ ] Test standalone 60 menit berhasil
