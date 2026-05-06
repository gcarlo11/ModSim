# Modul 4: Antrian Toilet
## Toilet Pria (4 titik) & Toilet Wanita (4 titik) — Terpisah Gender

**Satuan waktu:** minute

---

## 1. Hasil Akhir

Modul ini mensimulasikan antrian toilet di perpustakaan dengan **pemisahan gender**:

1. Pengunjung masuk dan memilih toilet berdasarkan **jenis kelamin**
2. **Toilet pria**: 2 urinal + 2 bilik = 4 titik service
3. **Toilet wanita**: 4 bilik = 4 titik service
4. Antri jika semua titik penuh
5. Selesai → keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
selectGenderToilet (PedSelectOutput — 50/50)
  ├── srvToiletPria (PedService, 4 titik)
  └── srvToiletWanita (PedService, 4 titik)
    ↓
snkSelesai (temporary)
```

---

## 2. Komponen

### Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` (TEMPORARY) |
| PedSelectOutput | `selectGenderToilet` |
| PedService | `srvToiletPria` |
| PedService | `srvToiletWanita` |
| PedSink | `snkSelesai` (TEMPORARY) |

### Markup

| Markup | Nama | Detail |
|---|---|---|
| Service with Lines | `svcToiletPria` | 4 services, 1 queue |
| Service with Lines | `svcToiletWanita` | 4 services, 1 queue |

> **Penjelasan:** Toilet pria punya 4 titik (2 urinal + 2 bilik) sehingga 4 orang bisa bersamaan. Toilet wanita 4 bilik, juga 4 orang bersamaan.

### 3D

| Objek | Nama | Posisi (contoh) |
|---|---|---|
| Box | `lantaiToilet3D` | (0,0,0) skala 15x15x0.2 |
| Box | `bilikToiletPria_1` s/d `_4` | 4 bilik di sisi kiri |
| Box | `bilikToiletWanita_1` s/d `_4` | 4 bilik di sisi kanan |
| Text | Label "Pria" / "Wanita" | Di atas masing-masing area |

### Variabel baru di PengunjungPed

| Nama | Type | Initial value |
|---|---|---|
| `jenisKelamin` | String | `""` | "PRIA" atau "WANITA" |
| `waktuMulaiToilet` | double | `0` |

### Variabel baru di Main

| Nama | Type | Initial value |
|---|---|---|
| `totalToiletPria` | int | 0 |
| `totalToiletWanita` | int | 0 |
| `maxAntrianToiletPria` | int | 0 |
| `maxAntrianToiletWanita` | int | 0 |

---

## 3. Buat Project

1. **File → New → Model** → nama: `Perpustakaan3D_Toilet`.
2. **Time units**: `minute`.
3. Buat **Pedestrian Type**: `PengunjungPed`.
4. Tambah variabel `jenisKelamin` (String, `""`) dan `waktuMulaiToilet` (double, `0`) ke diagram `PengunjungPed`.

---

## 4. Layout 3D

1. **3D Window** → `win3D`. **Camera** → `camMain`.
2. **Box** → `lantaiToilet3D`, skala 15x15x0.2.
3. Buat 4 Box kecil untuk bilik toilet pria di sisi kiri.
4. Buat 4 Box kecil untuk bilik toilet wanita di sisi kanan.
5. (Opsional) Tambah label "PRIA" dan "WANITA".

---

## 5. Markup

### 5.1 svcToiletPria

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcToiletPria`.
3. **Number of services** = `4`.
4. **N of queues** = `1`.
5. Letakkan service point di depan bilik toilet pria.

### 5.2 svcToiletWanita

1. **Service with Lines**.
2. Rename: `svcToiletWanita`.
3. **Number of services** = `4`.
4. **N of queues** = `1`.
5. Letakkan di depan bilik toilet wanita.

---

## 6. Flowchart

### 6.1 Blok dan koneksi

```
srcMasuk.out → selectGenderToilet.in

selectGenderToilet.out1 (PRIA, 50%) → srvToiletPria.in → snkSelesai.in
selectGenderToilet.out2 (WANITA, 50%) → srvToiletWanita.in → snkSelesai.in
```

### 6.2 `srcMasuk` (PedSource — TEMPORARY)

| Property | Value |
|---|---|
| Appears at | `line` |
| Target line | `entryLine` (buat sendiri) |
| Interarrival time | `exponential(1.0)` |
| New pedestrian | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "T-" + seqPed;
ped.tMasuk = time();
// Gender acak: 50% pria, 50% wanita
ped.jenisKelamin = uniform(0, 1) < 0.5 ? "PRIA" : "WANITA";
```

### 6.3 `selectGenderToilet` (PedSelectOutput)

| Property | Value |
|---|---|
| N outputs | `2` |
| Use conditions | Centang (bukan probabilities) |
| Output 1 condition | `ped.jenisKelamin.equals("PRIA")` |
| Output 2 condition | `ped.jenisKelamin.equals("WANITA")` |

> **Penjelasan:** Berbeda dengan modul lain yang pakai probabilitas, di sini kita pakai **kondisi** karena gender sudah ditentukan di `srcMasuk.onExit`.

### 6.4 `srvToiletPria` (PedService)

| Property | Value |
|---|---|
| Services | `svcToiletPria` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuToilet()` |

**Action On enter queue:**
```java
int q = srvToiletPria.queueSize();
if (q > maxAntrianToiletPria) {
    maxAntrianToiletPria = q;
}
```

**Action On begin service:**
```java
ped.waktuMulaiToilet = time();
```

**Action On end service:**
```java
totalToiletPria++;
traceln("TOILET Pria: " + ped.idPed + " | durasi=" + String.format("%.1f", time()-ped.waktuMulaiToilet) + " mnt");
```

### 6.5 `srvToiletWanita` (PedService)

| Property | Value |
|---|---|
| Services | `svcToiletWanita` |
| Queue choice policy | `Shortest queue` |
| Delay time | `hitungWaktuToilet()` |

**Action On enter queue:**
```java
int q = srvToiletWanita.queueSize();
if (q > maxAntrianToiletWanita) {
    maxAntrianToiletWanita = q;
}
```

**Action On begin service:**
```java
ped.waktuMulaiToilet = time();
```

**Action On end service:**
```java
totalToiletWanita++;
traceln("TOILET Wanita: " + ped.idPed + " | durasi=" + String.format("%.1f", time()-ped.waktuMulaiToilet) + " mnt");
```

### 6.6 `snkSelesai` (PedSink — TEMPORARY)

Action On enter:
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("DONE " + ped.idPed + " | gender=" + ped.jenisKelamin);
```

---

## 7. Fungsi di Main

### hitungWaktuToilet

Return type: `double`. Tidak perlu parameter — durasi toilet sama untuk semua.

```java
return uniform(2.0, 5.0);
```

### Tambah variabel di Main

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

---

## 8. Dashboard

```
"Total pengguna toilet: " + (totalToiletPria + totalToiletWanita)
```

```
"Pria: " + totalToiletPria + " | Wanita: " + totalToiletWanita
```

```
"Antrian pria: " + srvToiletPria.queueSize() + " (max: " + maxAntrianToiletPria + ")"
```

```
"Antrian wanita: " + srvToiletWanita.queueSize() + " (max: " + maxAntrianToiletWanita + ")"
```

---

## 9. Jalankan Uji Coba

1. **Stop time** = 30 menit.
2. Run.
3. Harus terlihat: pedestrian muncul, pilih toilet sesuai gender, antri (jika penuh), service, keluar.
4. Cek console: `TOILET Pria: T-3 | durasi=3.2 mnt`.

---

## 10. Integrasi & Troubleshooting

### Yang diubah saat integrasi

1. **Hapus** `srcMasuk` → ganti dengan koneksi dari `selectTujuan.out4` (skeleton).
2. **Hapus** `snkSelesai` → colok `srvToiletPria.out` dan `srvToiletWanita.out` ke `wJalanKeluar.in` (skeleton).

### Masalah: Semua orang masuk toilet pria

**Penyebab:** Kondisi `selectGenderToilet` salah.

**Solusi:** Cek **Output 1 condition** = `ped.jenisKelamin.equals("PRIA")`, **Output 2 condition** = `ped.jenisKelamin.equals("WANITA")`. Pastikan **Use conditions** dicentang (bukan Use probabilities).

### Masalah: Service point tidak kelihatan

**Penyebab:** Markup `svcToiletPria` atau `svcToiletWanita` ketutup objek 3D.

**Solusi:** Geser bilik toilet 3D atau kecilkan ukurannya. Service point harus di depan (atau di dalam) bilik.

---

## 11. Checklist

- [ ] `PengunjungPed` dengan variabel `jenisKelamin`, `waktuMulaiToilet`
- [ ] `svcToiletPria` (4 service, 1 queue) dan `svcToiletWanita` (4 service, 1 queue)
- [ ] `selectGenderToilet` menggunakan **kondisi** (bukan probabilitas)
- [ ] `hitungWaktuToilet()` dibuat di Main
- [ ] Dashboard toilet berfungsi
- [ ] Test standalone 30 menit berhasil
