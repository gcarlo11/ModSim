# Modul 1: Simulasi Parkir Perpustakaan
## Mobil dan Motor Datang, Parkir, Jalan Kaki ke Perpustakaan

**AnyLogic versi:** 8.9.8 (cek: Help → About)
**Satuan waktu:** minute
**Project asli:** `SimulasiParkirPerpustakaan/SimulasiParkirPerpustakaan.alp`

---

## 1. Hasil Akhir

Modul ini mensimulasikan:
1. Kendaraan (mobil/motor) masuk area parkir
2. Mencari tempat parkir kosong (waktu cari tergantung jenis kendaraan)
3. Parkir
4. Pengemudi turun dan jalan kaki ke pintu perpustakaan
5. Saat pulang: jalan kaki dari perpustakaan ke parkir
6. Keluar area parkir

**Flowchart standalone:**

```
srcParkirMobil → cariParkirMobil → jalanKePerpus → pedSink
srcParkirMotor → cariParkirMotor → jalanKePerpus → pedSink (bergabung)

orangPulang (dari perpus) → jalanKeParkiran → pedSink1
```

---

## 2. Yang Sudah Ada di Project

Jika membuka project `SimulasiParkirPerpustakaan.alp`, komponen berikut **sudah jadi**:

### Blok

| Blok | Nama | Fungsi |
|---|---|---|
| PedSource | `orangDatang` | Orang datang dari parkir |
| PedSource | `orangPulang` | Orang pulang ke parkir |
| PedGoTo | `jalanKePerpus` | Jalan dari parkir ke perpustakaan |
| PedGoTo | `jalanKeParkiran` | Jalan dari perpustakaan ke parkir |
| PedSink | `pedSink` | Selesai (masuk perpus) |
| PedSink | `pedSink1` | Selesai (pulang naik kendaraan) |

### 3D Objects

| File | Objek |
|---|---|
| `car.dae` | Mobil 3D |
| `motorcycle.dae` | Motor 3D |
| `building_1.dae` | Gedung perpustakaan |
| `person.dae` | Orang (pejalan kaki) |
| `tree_1.dae`, `tree_2.dae`, `tree_3.dae` | Pohon (3 variasi) |

### Markup

| Markup | Nama | Fungsi |
|---|---|---|
| TargetLine | `areaPejalanKaki` | Area jalan untuk pedestrian |
| RectangleNode | `pintuPerpus` | Posisi pintu masuk perpustakaan |

### Koneksi flowchart yang sudah ada

```
orangDatang.out  → jalanKePerpus.in → pedSink.in
orangPulang.out  → jalanKeParkiran.in → pedSink1.in
```

---

## 3. Buat Project Baru (Jika Mulai dari Nol)

Jika file `.alp` sudah ada, **lewati bagian ini**.

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_Parkir`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 3.1 Tambah Agent Type: PengunjungPed

1. Klik kanan model → **New → Agent Type**.
2. Nama: `PengunjungPed`.
3. Pilih template: `Pedestrian`.
4. Pilih animasi 3D orang.
5. Klik **Finish**.
6. Buka diagram `PengunjungPed`.
7. Tambah Variable:

| Nama | Type | Initial Value | Keterangan |
|---|---|---|---|
| `idPed` | String | `""` | ID unik |
| `tMasuk` | double | `0` | Waktu masuk sistem |
| `isParkir` | boolean | `false` | Apakah naik kendaraan |
| `jenisKendaraan` | String | `""` | "MOBIL" atau "MOTOR" |
| `waktuParkir` | double | `0` | Waktu mulai parkir |
| `nomorParkir` | int | `0` | Slot parkir (jika ada) |

### 3.2 Buat Layout 3D

1. Drag **3D Window** ke canvas. Rename: `win3D`.
2. Drag **Camera**. Rename: `camMain`.
3. Tambah **Box** untuk lantai parkir: `lantaiParkir3D`, skala 40x30x0.2.
4. Import aset 3D:
   - Palette **3D** → **Import 3D** → pilih file `.dae`.
   - Import: `car.dae`, `motorcycle.dae`, `building_1.dae`, `person.dae`.
5. Letakkan gedung di pojok, area parkir di tengah.

### 3.3 Buat Markup Pedestrian

1. Drag **Target line** → rename: `entryParkir` (garis masuk parkir).
2. Drag **Target line** → rename: `exitParkir` (garis keluar parkir).
3. Drag **RectangularNode** → rename: `areaParkirMobil` (area parkir mobil).
4. Drag **RectangularNode** → rename: `areaParkirMotor` (area parkir motor).

---

## 4. Bangun Flowchart Pedestrian (Lengkap)

### 4.1 Blok flowchart

| Blok | Rename | Keterangan |
|---|---|---|
| PedSource | `srcParkirMobil` | Mobil datang |
| PedSource | `srcParkirMotor` | Motor datang |
| PedGoTo | `cariParkirMobil` | Mobil cari tempat parkir |
| PedGoTo | `cariParkirMotor` | Motor cari tempat parkir |
| PedGoTo | `jalanKePerpus` | Berjalan ke perpustakaan |
| PedService | `srvParkirMobil` | Service untuk mobil |
| PedService | `srvParkirMotor` | Service untuk motor |
| PedWait | `wJalanKePerpus` | Jalan ke pintu perpus |
| PedSelectOutput | `selectJenisKendaraan` | Pilih mobil/motor |
| PedSink | `snkParkir` | Selesai (untuk modul) |
| PedSink | `snkPulang` | Selesai pulang |

### 4.2 Koneksi

```
srcParkirMobil.out → srvParkirMobil.in → jalanKePerpus.in → snkParkir.in
srcParkirMotor.out → srvParkirMotor.in → jalanKePerpus.in (sama)

======= MODUL PULANG (untuk integrasi) =======
srcPulang (temporary) → jalanKeParkiran.in → snkPulang.in
```

> **Penjelasan:** `srvParkirMobil` dan `srvParkirMotor` menggunakan markup TargetLine/RectangularNode yang sama atau berbeda. Di tutorial ini kita gunakan **PedService dengan delay time**, bukan PedGoTo, agar waktu cari parkir bisa dihitung secara statistik.

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcParkirMobil` (PedSource)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryParkir` |
| Arrive according to | `Interarrival time` |
| Interarrival time | `exponential(0.5)` (1 mobil setiap ~2 menit) |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "Mobil-" + seqPed;
ped.tMasuk = time();
ped.isParkir = true;
ped.jenisKendaraan = "MOBIL";
```

### 5.2 `srcParkirMotor` (PedSource)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryParkir` |
| Interarrival time | `exponential(0.3)` (1 motor setiap ~3.3 menit) |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "Motor-" + seqPed;
ped.tMasuk = time();
ped.isParkir = true;
ped.jenisKendaraan = "MOTOR";
```

> **Tips pemula:** `exponential(0.5)` berarti rata-rata 0.5 kendaraan per menit, alias 1 kendaraan setiap 2 menit. Makin besar angka, makin sering datang.

### 5.3 `srvParkirMobil` (PedService)

| Property | Value |
|---|---|
| Services | `svcParkirMobil` (Service with Lines atau RectangularNode) |
| Delay time | `hitungWaktuCariParkirMobil()` |
| Recovery delay | `0` |

**Action On begin service:**
```java
ped.waktuParkir = time();
```

**Action On end service:**
```java
totalMobil++;
traceln("PARKIR Mobil: " + ped.idPed);
```

### 5.4 `srvParkirMotor` (PedService)

| Property | Value |
|---|---|
| Delay time | `hitungWaktuCariParkirMotor()` |
| Recovery delay | `0` |

**Action On end service:**
```java
totalMotor++;
traceln("PARKIR Motor: " + ped.idPed);
```

### 5.5 `jalanKePerpus` (PedGoTo)

| Property | Value |
|---|---|
| Target | `pintuPerpus` (RectangularNode) |

### 5.6 `snkParkir` (PedSink) — TEMPORARY

Action On enter:
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("SELESAI: " + ped.idPed + " tSistem=" + tSistem + " menit");
```

---

## 6. Variabel di Main

### 6.1 Variabel baru

Tambahkan di `Main`:

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |
| `totalMobil` | int | 0 |
| `totalMotor` | int | 0 |

### 6.2 Fungsi hitungWaktuCariParkirMobil

Return type: `double`

```java
// Mobil butuh waktu lebih lama untuk parkir
return uniform(2.0, 8.0);
```

### 6.3 Fungsi hitungWaktuCariParkirMotor

Return type: `double`

```java
// Motor lebih lincah, parkir lebih cepat
return uniform(1.0, 3.0);
```

### 6.4 Fungsi avgWaktuSistem

Return type: `double`

```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

---

## 7. Dashboard

Tambahkan **Text** dinamis (untuk standalone testing):

```
"Total kendaraan: " + (totalMobil + totalMotor)
```

```
"Mobil: " + totalMobil + " | Motor: " + totalMotor
```

```
"Rata-rata waktu parkir: " + String.format("%.2f", avgWaktuSistem()) + " menit"
```

---

## 8. Jalankan untuk Uji Coba

1. **Stop time** = 30 menit.
2. Run.
3. Harus terlihat: mobil/motor muncul, bergerak ke area parkir, lalu orang berjalan ke gedung.
4. Cek console: `PARKIR Mobil: Mobil-1`, `SELESAI: Mobil-1`, dll.

---

## 9. Yang Perlu Diubah Saat Integrasi

**Blok yang dihapus:**
- `snkParkir` (ganti dengan koneksi ke `selectTujuan` di skeleton)
- `snkPulang` (ganti dengan koneksi ke `selectPulang` di skeleton)

**Blok yang tetap:**
- `srcParkirMobil`, `srcParkirMotor`
- `srvParkirMobil`, `srvParkirMotor`
- `jalanKePerpus`

**Koneksi baru (setelah integrasi):**
```
jalanKePerpus.out → selectTujuan.in (skeleton)
(srcParkirMobil + srcParkirMotor tetap seperti di atas)
```

---

## 10. Troubleshooting

### Masalah: Pedestrian tidak muncul

- Cek `srcParkirMobil.Appears at` = `line` dan `Target line` = `entryParkir`.
- Pastikan `entryParkir` ada di markup dan tidak ketutup objek 3D.

### Masalah: Pedestrian jalan menembus gedung

- Pindahkan `pintuPerpus` ke posisi yang lebih dekat dengan gedung.
- Pedestrian secara otomatis mencari jalur terpendek, jadi pastikan area jalan kosong.

### Masalah: Mobil/motor tidak kelihatan

- Modul ini pakai pedestrian (orang), bukan vehicle agent.
- Untuk efek "mobil masuk": bisa ditambahkan nanti di layer presentasi.
- Fokus dulu pada logika antrian dan waktu parkir.

---

## 11. Checklist Final

- [ ] `PengunjungPed` sudah dibuat dengan variabel `isParkir`, `jenisKendaraan`
- [ ] `srcParkirMobil` dan `srcParkirMotor` sudah berfungsi
- [ ] `srvParkirMobil` delay time = `hitungWaktuCariParkirMobil()`
- [ ] `srvParkirMotor` delay time = `hitungWaktuCariParkirMotor()`
- [ ] `jalanKePerpus` mengarah ke `pintuPerpus`
- [ ] Variabel `totalMobil` dan `totalMotor` terisi saat run
- [ ] Dashboard menampilkan statistik
- [ ] Test standalone berhasil (30 menit)
