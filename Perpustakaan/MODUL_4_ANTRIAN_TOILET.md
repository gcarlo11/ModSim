# Modul 4: Antrian Toilet
## Toilet Pria (4 titik) & Toilet Wanita (4 titik) — Terpisah Gender

**AnyLogic versi:** 8.9.8
**Satuan waktu:** `minute`

---

## Ringkasan Hasil Akhir

Modul ini mensimulasikan antrian toilet di perpustakaan dengan **pemisahan gender**:

1. Pengunjung datang dari `selectTujuan.out4` (skeleton) atau dari `srcMasuk` temporary
2. Memilih toilet berdasarkan **jenis kelamin** (kondisi, bukan probabilitas)
3. **Toilet pria**: 4 titik service (2 urinal + 2 bilik) — 4 orang bisa bersamaan
4. **Toilet wanita**: 4 bilik = 4 titik service — 4 orang bisa bersamaan
5. Antri jika semua titik penuh
6. Selesai → jalan keluar

**Flowchart standalone:**

```
srcMasuk (temporary)
    ↓
selectGenderToilet (PedSelectOutput — KONDISI, bukan probabilitas)
  ├── out1 (jika PRIA) → srvToiletPria (PedService, 4 titik)
  └── out2 (jika WANITA) → srvToiletWanita (PedService, 4 titik)
    ↓
snkSelesai (temporary)
```

**Konsep cepat untuk pemula:**
- **selectGenderToilet** pakai **kondisi** (bukan probabilitas) karena gender sudah ditentukan sebelumnya
- **Service with Lines**: area fisik toilet dengan beberapa "bilik" (service points)
- **Shortest queue**: otomatis pilih bilik yang paling cepat kosong

---

## 1. Komponen

### 1.1 Blok flowchart

| Blok | Rename | Fungsi |
|---|---|---|
| PedSource | `srcMasuk` | TEMPORARY — untuk standalone |
| PedSelectOutput | `selectGenderToilet` | Pisah gender pria/wanita pakai kondisi |
| PedService | `srvToiletPria` | Service toilet pria (4 titik) |
| PedService | `srvToiletWanita` | Service toilet wanita (4 titik) |
| PedSink | `snkSelesai` | TEMPORARY — untuk standalone |

### 1.2 Markup

| Markup | Nama | Service Points | Antrean |
|---|---|---|---|
| Service with Lines | `svcToiletPria` | 4 | 1 |
| Service with Lines | `svcToiletWanita` | 4 | 1 |

> **Penjelasan:** 4 service points = 4 bilik/urinal. 1 queue = antrean terpusat, ambil bilik yang kosong pertama.

### 1.3 3D (opsional)

| Objek | Nama | Jumlah |
|---|---|---|
| Floor | `lantaiToilet3D` | 1 |
| Bilik toilet pria | `bilikToiletPria_1` s/d `_4` | 4 |
| Bilik toilet wanita | `bilikToiletWanita_1` s/d `_4` | 4 |
| Label | "PRIA" / "WANITA" | 2 |

### 1.4 Variabel baru di PengunjungPed

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `jenisKelamin` | String | `""` | "PRIA" atau "WANITA" (diisi random saat lahir) |
| `waktuMulaiToilet` | double | `0` | Waktu masuk bilik toilet |

### 1.5 Variabel baru di Main

| Nama | Type | Initial value | Penjelasan |
|---|---|---|---|
| `totalToiletPria` | int | 0 | Total pengguna toilet pria |
| `totalToiletWanita` | int | 0 | Total pengguna toilet wanita |
| `maxAntrianToiletPria` | int | 0 | Antrean maksimum toilet pria |
| `maxAntrianToiletWanita` | int | 0 | Antrean maksimum toilet wanita |

---

## 2. Buat Project Baru

1. Buka AnyLogic.
2. **File → New → Model**.
3. Nama: `Perpustakaan3D_Toilet`.
4. **Time units**: `minute`.
5. Klik **Finish**.

### 2.1 Buat/ubah PengunjungPed

Jika sudah ada pedestrian type `PengunjungPed`:
- Buka diagramnya → **tambah 2 variable**: `jenisKelamin` (String, `""`), `waktuMulaiToilet` (double, `0`)

Jika belum ada:
1. **Pedestrian Library** → drag **Pedestrian Type** → nama `PengunjungPed`.
2. Pilih animasi 3D.
3. Buka diagram → tambah variabel.

### 2.2 Tambah variabel Main

Buka Main → drag **Variable** 4 kali → isi sesuai tabel 1.5.

---

## 3. Layout 3D dan Markup

### 3.1 3D

1. **3D Window** → `win3D`.
2. **Camera** → `camMain`.
3. **Box** → `lantaiToilet3D`, skala 15x15x0.2.
4. Buat 4 Box kecil untuk bilik toilet pria di sisi kiri (ukur ~1x1x2).
5. Buat 4 Box kecil untuk bilik toilet wanita di sisi kanan.

### 3.2 Target lines (untuk standalone)

1. **Target line** → `entryLine`.
2. **Target line** → `exitLine`.

### 3.3 svcToiletPria

1. **Space Markup** → **Service with Lines**.
2. Rename: `svcToiletPria`.
3. **Number of services** = `4`.
4. **N of queues** = `1`.
5. Letakkan 4 service point di depan 4 bilik toilet pria.
6. Atur antrean (queue line) menjauh dari bilik agar tidak menghalangi.

### 3.4 svcToiletWanita

1. **Service with Lines**.
2. Rename: `svcToiletWanita`.
3. **Number of services** = `4`.
4. **N of queues** = `1`.
5. Letakkan 4 service point di depan 4 bilik toilet wanita.

> **Tips pemula:** Letakkan queue line di lorong/area terbuka. Jangan sampai antrean menghalangi jalan ke area lain.

---

## 4. Bangun Flowchart

### 4.1 Blok

| Blok | Rename |
|---|---|
| PedSource | `srcMasuk` |
| PedSelectOutput | `selectGenderToilet` |
| PedService | `srvToiletPria` |
| PedService | `srvToiletWanita` |
| PedSink | `snkSelesai` |

### 4.2 Koneksi

```
srcMasuk.out → selectGenderToilet.in

selectGenderToilet.out1 (kondisi: PRIA) → srvToiletPria.in
selectGenderToilet.out2 (kondisi: WANITA) → srvToiletWanita.in

srvToiletPria.out → snkSelesai.in
srvToiletWanita.out → snkSelesai.in
```

---

## 5. Konfigurasi Detail Setiap Blok

### 5.1 `srcMasuk` (PedSource) — TEMPORARY

| Property | Value |
|---|---|
| `Appears at` | `line` |
| `Target line` | `entryLine` |
| `Interarrival time` | `exponential(1.0)` (1 orang per menit) |
| `New pedestrian` | `PengunjungPed` |

**Action On exit:**
```java
seqPed++;
ped.idPed = "T-" + seqPed;
ped.tMasuk = time();

// Gender acak: 50% pria, 50% wanita
ped.jenisKelamin = uniform(0, 1) < 0.5 ? "PRIA" : "WANITA";

traceln("ARRIVE " + ped.idPed + " | gender=" + ped.jenisKelamin);
```

> **Tips pemula:** `uniform(0,1) < 0.5` artinya 50% kemungkinan true. Jadi 50% pria, 50% wanita.

### 5.2 `selectGenderToilet` (PedSelectOutput)

⚠️ **GUNAKAN KONDISI, BUKAN PROBABILITAS!**

| Property | Value |
|---|---|
| `N outputs` | `2` |
| **Use probabilities** | **JANGAN centang** ❌ |
| **Use conditions** | **WAJIB centang** ✅ |
| Output 1 condition | `ped.jenisKelamin.equals("PRIA")` |
| Output 2 condition | `ped.jenisKelamin.equals("WANITA")` |

> **Kenapa pakai kondisi?** Karena gender sudah ditentukan di `srcMasuk.onExit`. Kalau pakai probabilitas, bisa saja pria masuk toilet wanita atau sebaliknya.

### 5.3 `srvToiletPria` (PedService)

| Property | Value |
|---|---|
| `Services` | `svcToiletPria` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungWaktuToilet()` |

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

### 5.4 `srvToiletWanita` (PedService)

| Property | Value |
|---|---|
| `Services` | `svcToiletWanita` |
| `Queue choice policy` | `Shortest queue` |
| `Delay time` | `hitungWaktuToilet()` |

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

### 5.5 `snkSelesai` (PedSink) — TEMPORARY

**Action On enter:**
```java
totalSelesai++;
double tSistem = time() - ped.tMasuk;
totalWaktuSistem += tSistem;
traceln("DONE " + ped.idPed + " | gender=" + ped.jenisKelamin + " | tSistem=" + String.format("%.2f", tSistem) + " mnt");
```

---

## 6. Fungsi di Main

### 6.1 `hitungWaktuToilet`

Return type: `double`. **Tidak perlu parameter** — durasi toilet sama untuk semua.

```java
// Waktu di toilet: antara 2-5 menit
return uniform(2.0, 5.0);
```

### 6.2 Variabel tambahan

| Nama | Type | Initial value |
|---|---|---|
| `seqPed` | int | 0 |
| `totalSelesai` | int | 0 |
| `totalWaktuSistem` | double | 0 |

### 6.3 Fungsi `avgWaktuSistem`

Return type: `double`.
```java
return totalSelesai == 0 ? 0 : totalWaktuSistem / totalSelesai;
```

---

## 7. Dashboard

```java
// Total pengguna
"Total pengguna toilet: " + (totalToiletPria + totalToiletWanita)
```

```java
// Detail gender
"Pria: " + totalToiletPria + " | Wanita: " + totalToiletWanita
```

```java
// Antrian
"Antrian pria: " + srvToiletPria.queueSize() + " (max: " + maxAntrianToiletPria + ")"
```

```java
"Antrian wanita: " + srvToiletWanita.queueSize() + " (max: " + maxAntrianToiletWanita + ")"
```

---

## 8. Jalankan untuk Uji Coba

### Test 1: Coba 10 menit

1. **Stop time** = `10`.
2. Run.
3. Apakah ada yang ke toilet pria dan wanita?

### Test 2: Test penuh 30 menit

1. **Stop time** = `30`.
2. Run.
3. Cek console: `TOILET Pria: T-3 | durasi=3.2 mnt`, `TOILET Wanita: T-4 | durasi=4.1 mnt`.
4. Cek apakah gender sesuai — tidak boleh pria masuk toilet wanita.

### Perkiraan hasil wajar

- Total ~30 pengunjung dalam 30 menit (arrival exponential(1.0))
- ~15 pria, ~15 wanita (50:50)
- Rata-rata durasi toilet ~3.5 menit (antara 2-5)
- Antrean pria dan wanita relatif seimbang

---

## 9. Yang Perlu Diubah Saat Integrasi

1. **Hapus** `srcMasuk` (temporary) → ganti dengan koneksi dari `selectTujuan.out4` (skeleton).
2. **Hapus** `snkSelesai` (temporary) → colok `srvToiletPria.out` dan `srvToiletWanita.out` ke `wJalanKeluar.in` (skeleton).

---

## 10. Troubleshooting

### Masalah: Semua orang masuk toilet pria

**Penyebab:** `selectGenderToilet` masih pakai **probabilities**, bukan **conditions**.

**Solusi:** Buka properties `selectGenderToilet` → centang **Use conditions** → isi:
- Output 1: `ped.jenisKelamin.equals("PRIA")`
- Output 2: `ped.jenisKelamin.equals("WANITA")`

### Masalah: Ada error "equals" tidak dikenal

**Penyebab:** Salah penulisan method `equals` di Java.
**Solusi:** Pastikan persis: `ped.jenisKelamin.equals("PRIA")`. Case-sensitive! `equals` bukan `Equals`.

### Masalah: Service point tidak kelihatan di 3D

**Penyebab:** Service point ketutup objek Box 3D toilet.

**Solusi:** Pindahkan Box toilet agak ke samping atau kecilkan ukurannya. Service point harus terlihat di depan/dalam bilik.

### Masalah: Antrean mengular sampai keluar area

**Penyebab:** 4 service points tidak cukup untuk 1 orang/menit.

**Solusi:** Tambah service points (misal 6) atau kurangi arrival rate jadi `exponential(1.5)`.

### Masalah: Error "selectGenderToilet" — "Output X condition is empty"

**Penyebab:** Kondisi output belum diisi.

**Solusi:** Pastikan Output 1 condition terisi `ped.jenisKelamin.equals("PRIA")` dan Output 2 `ped.jenisKelamin.equals("WANITA")`.

---

## 11. Fallback / Alternatif

### Jika ingin jumlah toilet tidak sama gender

Edit Number of services:
- Pria: 3 (2 urinal + 1 bilik)
- Wanita: 6 (6 bilik)

Sesuaikan dengan kebutuhan.

### Jika arrival terlalu cepat

Ubah `exponential(1.0)` jadi `exponential(2.0)` — orang datang lebih lambat.

---

## 12. Mode Presentasi

- **Stop time:** 15-20 menit (cukup lihat pola antrian)
- **Fokus kamera 3D:** kedua area toilet (split view atau geser)
- Bisa tambah komentar: "lihat, antrian wanita lebih panjang karena 4 bilik untuk 50% pengunjung"

---

## 13. Checklist Final

- [ ] **PengunjungPed**: variabel `jenisKelamin`, `waktuMulaiToilet`
- [ ] **Variabel Main**: `totalToiletPria`, `totalToiletWanita`, `maxAntrianToiletPria`, `maxAntrianToiletWanita`
- [ ] **Markup**: `svcToiletPria` (4 services, 1 queue), `svcToiletWanita` (4 services, 1 queue)
- [ ] **selectGenderToilet**: menggunakan **kondisi** (bukan probabilitas) ✅
- [ ] **srvToiletPria.Services** = `svcToiletPria`, Delay = `hitungWaktuToilet()`
- [ ] **srvToiletWanita.Services** = `svcToiletWanita`, Delay = `hitungWaktuToilet()`
- [ ] **Fungsi**: `hitungWaktuToilet()`
- [ ] **Dashboard**: statistik real-time
- [ ] **Test 30 menit**: semua gender sesuai tujuan, tidak ada error
