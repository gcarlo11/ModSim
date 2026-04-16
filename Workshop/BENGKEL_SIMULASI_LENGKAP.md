# 📋 SIMULASI BENGKEL PERBAIKAN MOBIL - PANDUAN LENGKAP

**Topik**: Sistem Operasional Bengkel Perbaikan Kendaraan
**Jumlah Orang**: 6 orang
**Target**: Setiap orang membuat 1 mini-simulation, lalu digabung menjadi 1 sistem utuh

---

## 🎯 GAMBARAN UMUM SISTEM

### Alur Pelanggan di Bengkel:
```
Pelanggan Tiba
    ↓
[ORANG 1] Penerimaan Mobil - Keluhan dicatat
    ↓
[ORANG 2] Diagnostic - Cek masalah
    ↓
[ORANG 3] Siapkan Onderdil - Ambil spare parts
    ↓
[ORANG 4] Perbaikan - Kerjakan di workshop
    ↓
[ORANG 5] Test Drive - Verifikasi hasil
    ↓
[ORANG 6] Pembayaran & Serah Kunci
    ↓
Pelanggan Pulang dengan Mobil
```

### Durasi Total: 2-6 jam (tergantung kompleksitas kerusakan)

### Sumber Data Simulasi:
- **Waktu antar pelanggan**: 10-30 menit (random)
- **Jumlah pelanggan/hari**: 8-15 mobil
- **Jam operasi**: 08:00 - 17:00 (9 jam)

---

---

## 👤 MODUL 1: PENERIMAAN MOBIL (ORANG 1)

### 📝 DESKRIPSI PROSES

**Tujuan**: Menerima mobil pelanggan, mencatat keluhan, dan membuat work order.

**Proses Detail**:
1. Pelanggan tiba dengan mobil
2. Receptionist (Orang 1) menyapa & mencatat data:
   - Nama pelanggan
   - Nomor telepon
   - Jenis mobil (Avanza, Honda Civic, dll)
   - Keluhan / masalah?
   - Kapan perlu selesai?
3. Data diinput ke sistem
4. Work order dicetak & diberikan nomor (misal: WO-001)
5. Mobil dibawa ke area parking sebelum diagnostic

**Waktu Proses**: 
- Input data normal: 3-5 menit
- Jika data tidak jelas, perlu klarifikasi: +3 menit
- Jika sistem komputer lambat: +2 menit

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|-------|--------|
| **Input** | Pelanggan datang berkendara |
| **Output** | Entity: Mobil + Work Order + Data pelanggan |
| **Resource** | 1 reception desk, 1 receptionist, komputer |
| **Kapasitas** | 1 mobil/waktu proses |

### 📈 METRIK DIUKUR

- Berapa rata-rata waktu penerimaan per mobil?
- Apakah ada kemacetan di reception (mobil harus tunggu receptionist)?
- Berapa % data input error?
- Utilization receptionist (% sibuk)

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Buat Source (Pelanggan Tiba)**

```
Source: "PelangganTiba"
- Buat entity baru: "Mobil"
- Arrival distribution: Exponential(mean=20 menit)
- Qty: ~12 mobil/hari (8 jam operasi)
```

#### **Step 2: Buat Service Desk Penerimaan**

```
Service: "Penerimaan"
- Capacity: 1 receptionist
- Service time: 4 menit (uniform 3-5 menit)
- Delay probabilitas: 10% kasus butuh +3 menit klarifikasi

Pseudocode:
  On service start:
    random_val = uniform(0, 1)
    if random_val < 0.1:
      service_time = 5 + uniform(2, 4)  // normal 5 min + delay 2-4 min
    else:
      service_time = uniform(3, 5)
```

#### **Step 3: Atribut Entity Mobil**

```
Mobil Entity attributes:
- work_order_id: String (auto-generated: "WO-" + counter)
- customer_name: String
- phone: String
- car_type: String (pilihan: Avanza, Honda Civic, Toyota Camry, dll)
- keluhan: String (pilihan: Mesin panas, AC rusak, Rem blong, dll)
- waktu_penerimaan: double (timestamp)
- priority: int (1=High, 2=Medium, 3=Low)
- estimated_ready: double (target selesai)
```

#### **Step 4: Buat Output Data**

```
Setiap mobil keluar dari service Penerimaan:
- Catat: work_order_id, waktu masuk, waktu keluar
- Kirim ke Orang 2 (Diagnostic)
```

### 📋 PARAMETER SIMULASI

```
PENERIMAAN (Orang 1):
- Jam operasi: 08:00 - 17:00 (9 jam)
- Hari simulasi: Monday (normal)
- Jumlah receptionist: 1 orang
- Service time: 4 ± 1 menit (mean 4, uniform 3-5)
- Klarifikasi tambahan: 10% kasus, +3 menit
- Target: Selesai penerimaan dalam 5 menit
```

### 📤 OUTPUT YANG DIKIRIM KE ORANG 2

```json
{
  "work_order_id": "WO-001",
  "customer_name": "Budi Hartono",
  "car_type": "Honda Civic",
  "keluhan": "Mesin panas",
  "waktu_penerimaan": "08:30",
  "priority": 2
}
```

---

---

## 🔍 MODUL 2: DIAGNOSTIC (ORANG 2)

### 📝 DESKRIPSI PROSES

**Tujuan**: Cek masalah mobil secara detail dan buat list perbaikan.

**Proses Detail**:
1. Mobil dari penerimaan dibawa ke area diagnostic
2. Mekanik (Orang 2) check:
   - Buka kap mesin, lihat kondisi
   - Jalankan mesin, dengarkan suara
   - Check level oli, air radiator
   - Periksa rem, suspensi
   - Baca kode error dengan diagnostic tool
3. Hasil diagnostic: Penyebab masalah + List onderdil yang dibutuhkan
4. Estimasi biaya & waktu perbaikan
5. Laporan diagnostic dicetak

**Waktu Proses**:
- Diagnostic sederhana (1 masalah): 15-20 menit
- Diagnostic kompleks (multiple issues): 30-45 menit
- Jika diagnostic tool bermasalah: +10 menit

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|-------|--------|
| **Input** | Mobil dari Orang 1 + Work Order |
| **Output** | Laporan Diagnostic, List onderdil, Estimasi biaya & waktu |
| **Resource** | 2 mekanik, diagnostic tools, area diagnostic |
| **Kapasitas** | 2 mobil dapat di-diagnostic bersamaan |

### 📈 METRIK DIUKUR

- Rata-rata waktu diagnostic?
- Apakah 2 mekanik cukup? Apakah ada bottleneck?
- Berapa % kasus diagnostic tool error?
- Berapa rata-rata jumlah onderdil yang dibutuhkan?

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Service Diagnostic**

```
Service: "Diagnostic"
- Capacity: 2 mekanik (paralel)
- Service time: DEPENDS on complexity
  - Sederhana (50% kasus): uniform(15, 20) menit
  - Kompleks (50% kasus): uniform(30, 45) menit

Pseudocode:
  On service start:
    if complexity == "Simple":
      service_time = uniform(15, 20)
    else:
      service_time = uniform(30, 45)
    
    // 5% chance diagnostic tool error → +10 menit
    if uniform(0, 1) < 0.05:
      service_time += 10
```

#### **Step 2: Tentukan Kompleksitas**

```
Atribut Mobil ditambah:
- complexity: String (Simple / Complex)
- parts_needed: Array of Strings (daftar onderdil)
- estimated_cost: double (rupiah)
- estimated_duration: double (menit)
- diagnostic_time: double

On entry diagnostic:
  // Berdasarkan keluhan dari Orang 1, tentukan kompleksitas
  if keluhan == "Mesin panas":
    complexity = "Simple"  // hanya ganti coolant
    parts_needed = ["Coolant", "Fan belt"]
  else if keluhan == "Rem blong":
    complexity = "Complex"  // perlu cek seluruh sistem rem
    parts_needed = ["Brake pads", "Brake fluid", "Brake disc"]
```

#### **Step 3: Generate Parts List**

```
Function: Generate_Parts_List(keluhan)
  switch(keluhan):
    case "Mesin panas":
      return ["Coolant (1L)", "Fan belt"]
      cost = 200000
    case "AC rusak":
      return ["AC Compressor", "AC Filter", "Freon"]
      cost = 1500000
    case "Rem blong":
      return ["Brake pads (1 set)", "Brake fluid (1L)"]
      cost = 500000
    default:
      return ["General inspection parts"]
      cost = 250000
```

### 📋 PARAMETER SIMULASI

```
DIAGNOSTIC (Orang 2):
- Jumlah mekanik: 2 orang
- Kapasitas bersamaan: 2 mobil
- Waktu diagnostic:
  - Simple: 15-20 menit (50% kasus)
  - Complex: 30-45 menit (50% kasus)
- Tool error: 5% kasus, +10 menit
- Resource: 2 diagnostic bay, 2 diagnostic scanner
```

### 📤 OUTPUT YANG DIKIRIM KE ORANG 3

```json
{
  "work_order_id": "WO-001",
  "car_type": "Honda Civic",
  "keluhan": "Mesin panas",
  "complexity": "Simple",
  "parts_needed": ["Coolant (1L)", "Fan belt"],
  "estimated_cost": 200000,
  "estimated_duration": 45,
  "diagnostic_time": 18
}
```

---

---

## 🔧 MODUL 3: SIAPKAN ONDERDIL (ORANG 3)

### 📝 DESKRIPSI PROSES

**Tujuan**: Ambil & siapkan spare parts yang diperlukan dari gudang.

**Proses Detail**:
1. Terima list onderdil dari diagnostic (Orang 2)
2. Cek list di sistem inventory:
   - Berapa stock setiap part?
   - Apakah semua tersedia?
3. Jika tersedia → ambil dari rak gudang (1-2 menit per item)
4. Jika tidak tersedia → 
   - Cari supplier alternatif (online order)
   - Lead time: 30 menit - 2 jam tergantung supplier
5. Kumpulkan semua parts ke work station
6. Verifikasi & siapkan di tray untuk mekanik

**Waktu Proses**:
- Semua part tersedia: 5-10 menit (2-3 parts)
- Ada 1 part tidak ada: +30 menit (order + tunggu)
- Semua parts tidak ada: +2 jam (kompleks, jarang)

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|------|--------|
| **Input** | List parts dari Orang 2 |
| **Output** | Parts siap di tray, inventory terupdate |
| **Resource** | 1-2 staff gudang, rak onderdil, packaging |
| **Kapasitas** | Process 2-3 work order paralel |

### 📈 METRIK DIUKUR

- Rata-rata waktu persiapan parts?
- % kasus ada part tidak tersedia?
- Rata-rata lead time order online?
- Utilization staff gudang?
- Inventory turnover?

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Inventory Database**


Inventory Table (inisial):
| Part Name | Stock | Min Level | Supplier |
|------|--------|--------|--------|
| Coolant 1L | 30 | 5 | Lokal |
| Fan belt | 15 | 3 | Lokal |
| AC Compressor | 3 | 1 | Import (2jam) |
| Brake pads set | 20 | 5 | Lokal |
| Brake fluid 1L | 25 | 5 | Lokal |


#### **Step 2: Cek Ketersediaan**

```
Function: Check_Parts_Availability(parts_list)
  available_all = true
  lead_time = 0
  
  for each part in parts_list:
    if Inventory[part].stock >= 1:
      lead_time = 0  // semua tersedia
    else:
      // Part tidak ada
      available_all = false
      lead_time = max(lead_time, Inventory[part].supplier_leadtime)
      // Misal: lokal = 0.5jam, import = 2jam
  
  return {available: available_all, wait_time: lead_time}
```

#### **Step 3: Service: Prepare Parts**

```
Service: "Siapkan Parts"
- Capacity: 2 staff
- Service time:

IF parts_available == true:
  service_time = uniform(5, 10)  // ambil dari gudang
  THEN:
    Update inventory: stock -= 1 untuk setiap part
    
ELSE IF ada part tidak ada:
  service_time = lead_time (30 min - 2 hours)
  THEN:
    Simulate order: delay, tunggu kedatangan
    Update inventory: stock += part yang baru datang
    Lanjut prepare

Pseudocode:
  availability = Check_Parts_Availability(parts_list)
  
  if availability.available:
    service_time = uniform(5, 10)
  else:
    service_time = availability.wait_time
    // Misal: 30-120 menit tergantung supplier
    service_time = uniform(30, availability.wait_time * 60)
```

#### **Step 4: Atribut Mobil ditambah**

```
Mobil attributes (tambahan):
- parts_ready: boolean (false → jadi true setelah service selesai)
- parts_preparation_time: double
- parts_cost_actual: double
- supplier_wait_time: double
- stock_issue: boolean (true jika ada part out of stock)
```

### 📋 PARAMETER SIMULASI

```
SIAPKAN ONDERDIL (Orang 3):
- Staff gudang: 2 orang
- Kapasitas paralel: 2-3 work order
- Stock level awal: 
  - Lokal parts: 10-30 units
  - Import parts: 1-5 units
- Waktu ambil parts:
  - Tersedia: 5-10 menit (1-2 part)
  - Order lokal: +30 menit
  - Order import: +120 menit
- Restock interval: 1x per hari otomatis
```

### 📤 OUTPUT YANG DIKIRIM KE ORANG 4

```json
{
  "work_order_id": "WO-001",
  "parts_ready": true,
  "parts_preparation_time": 7,
  "parts_cost_actual": 200000,
  "supplier_wait_time": 0,
  "stock_issue": false,
  "ready_timestamp": "09:15"
}
```

---

---

## 🔨 MODUL 4: PERBAIKAN MESIN (ORANG 4)

### 📝 DESKRIPSI PROSES

**Tujuan**: Melakukan perbaikan mobil sesuai hasil diagnostic.

**Proses Detail**:
1. Terima mobil dari Orang 3 (parts sudah siap)
2. Buka mobil & kerjakan perbaikan sesuai work order:
   - Copot part lama
   - Install part baru
   - Jalankan test kecil
3. Jika ada komplikasi:
   - Part rusak saat dipasang → gunakan backup
   - Lepas part tapia punya kesulitan → perlu bantuan
4. Setelah selesai → mobil ready untuk test drive

**Waktu Proses**:
- Perbaikan sederhana: 1-2 jam
- Perbaikan kompleks: 2-4 jam
- Ada komplikasi: +30-60 menit

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|------|--------|
| **Input** | Mobil + Parts siap dari Orang 3 |
| **Output** | Mobil selesai diperbaiki, ready test drive |
| **Resource** | 2-3 mekanik, 2-3 bay perbaikan, tools |
| **Kapasitas** | 2-3 mobil paralel (tergantung bay tersedia) |

### 📈 METRIK DIUKUR

- Rata-rata waktu perbaikan?
- Apakah 2 bay cukup? 3 bay?
- % kasus ada komplikasi?
- Utilization mekanik?
- Apakah ada mobil yang menunggu bay yang kosong?

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Service Perbaikan**

```
Service: "Perbaikan"
- Capacity: 2-3 bay (sesuai jumlah mekanik)
- Service time:

IF complexity == "Simple":
  base_time = uniform(60, 120)  // 1-2 jam
ELSE:
  base_time = uniform(120, 240)  // 2-4 jam

// 15% chance ada komplikasi → +30-60 menit
IF uniform(0, 1) < 0.15:
  base_time += uniform(30, 60)
  
service_time = base_time
```

#### **Step 2: Assign Mekanik ke Bay**

```
Each bay:
- Bay 1: Mekanik A
- Bay 2: Mekanik B
- Bay 3: Mekanik C (optional)

Jika semua bay sibuk → mobil menunggu di queue
```

#### **Step 3: Komplikasi Logic**

```
Function: Check_Complication()
  random_val = uniform(0, 1)
  
  if random_val < 0.15:  // 15% komplikasi
    complication_type = random choice:
      - "Part rusak saat pasang": delay +30 menit
      - "Perlu bantuan 2 mekanik": delay +20 menit
      - "Tool tidak cocok": delay +15 menit
    
    return {has_issue: true, delay: delay_value}
  else:
    return {has_issue: false, delay: 0}
```

#### **Step 4: Atribut Mobil ditambah**

```
Mobil attributes (tambahan):
- repair_start_time: double
- repair_duration: double
- bay_assigned: int (1, 2, atau 3)
- mechanic_assigned: String (Mekanik A/B/C)
- complication: boolean
- complication_type: String
- repair_status: String (IN_PROGRESS, COMPLETED)
```

### 📋 PARAMETER SIMULASI

```
PERBAIKAN (Orang 4):
- Jumlah bay: 2-3 (tergantung budget)
- Jumlah mekanik: 2-3 orang
- Waktu perbaikan:
  - Simple: 60-120 menit (1-2 jam)
  - Complex: 120-240 menit (2-4 jam)
- Komplikasi: 15% kasus, delay 15-60 menit
- Tools: Standard garage tools available
- Working hours: 08:00 - 17:00
```

### 📤 OUTPUT YANG DIKIRIM KE ORANG 5

```json
{
  "work_order_id": "WO-001",
  "repair_duration": 95,
  "bay_assigned": 1,
  "mechanic_assigned": "Mekanik A",
  "complication": false,
  "repair_status": "COMPLETED",
  "ready_for_testdrive": true
}
```

---

---

## 🏁 MODUL 5: TEST DRIVE (ORANG 5)

### 📝 DESKRIPSI PROSES

**Tujuan**: Verifikasi hasil perbaikan dengan test drive.

**Proses Detail**:
1. Mobil dari Orang 4 (perbaikan selesai) → ambil kunci
2. Test drive: 15-30 menit. Cek:
   - Apakah masalah sudah terselesaikan?
   - Ada bunyi aneh?
   - AC dingin?
   - Rem responsif?
3. Hasil test drive:
   - ✅ **PASS** (85%): Masalah resolved, siap diserahkan
   - ⚠️ **FAIL** (10%): Perlu perbaikan tambahan, kembali ke Orang 4
   - 🔴 **CRITICAL FAIL** (5%): Serius, butuh override/konsultasi

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|------|--------|
| **Input** | Mobil selesai perbaikan dari Orang 4 |
| **Output** | Test drive report, mobil PASS/FAIL/CRITICAL |
| **Resource** | 1 test driver, rute test drive |
| **Kapasitas** | 1 mobil/waktu |

### 📈 METRIK DIUKUR

- Rata-rata waktu test drive?
- % pass rate?
- Jika FAIL, berapa lama additional repair?
- Apakah ada rework (mobil kembali ke bay)?

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Service Test Drive**

```
Service: "Test Drive"
- Capacity: 1 test driver
- Service time: uniform(15, 30) menit
```

#### **Step 2: Determine Pass/Fail**

```
Function: Evaluate_Test_Drive()
  random_val = uniform(0, 1)
  
  if random_val < 0.85:
    return "PASS"
  else if random_val < 0.95:
    return "FAIL"
  else:
    return "CRITICAL_FAIL"
```

#### **Step 3: Handle Hasil**

```
On test drive complete:
  result = Evaluate_Test_Drive()
  
  if result == "PASS":
    status = "ReadyForPickup"  // Go to Orang 6 (bayar)
    
  else if result == "FAIL":
    // Kembali ke bay untuk perbaikan tambahan
    DetourToBay()
    additional_repair_time = uniform(30, 60)  // 30-60 menit
    // Setelah perbaikan, test drive lagi
    
  else if result == "CRITICAL_FAIL":
    // Konsultasi dengan manager / owner
    manager_decision_time = uniform(30, 120)
    status = "NeedsManagerApproval"
```

#### **Step 4: Atribut Mobil ditambah**

```
Mobil attributes (tambahan):
- testdrive_start: double
- testdrive_duration: double
- testdrive_result: String (PASS/FAIL/CRITICAL_FAIL)
- testdrive_notes: String
- rework_count: int (berapa kali kembali ke bay?)
```

### 📋 PARAMETER SIMULASI

```
TEST DRIVE (Orang 5):
- Test driver: 1 orang
- Service time: 15-30 menit per mobil
- Pass rate: 85%
- Fail rate: 10% (butuh rework)
- Critical fail rate: 5% (butuh manager decision)
- Rework time: +30-60 menit
- Rework kapasitas: Max 1-2x per mobil
```

### 📤 OUTPUT YANG DIKIRIM KE ORANG 6 (ATAU KEMBALI KE ORANG 4)

```json
{
  "work_order_id": "WO-001",
  "testdrive_duration": 22,
  "testdrive_result": "PASS",
  "testdrive_notes": "Mesin normal, AC dingin, rem responsif",
  "rework_count": 0,
  "ready_for_payment": true
}
```

---

---

## 💰 MODUL 6: PEMBAYARAN & SERAH KUNCI (ORANG 6)

### 📝 DESKRIPSI PROSES

**Tujuan**: Finalisasi, pembayaran, dan serahkan mobil ke pelanggan.

**Proses Detail**:
1. Mobil PASS test drive → ke counter administrasi
2. Hitungtotal biaya:
   - Biaya parts (dari Orang 3)
   - Biaya labor/kerja mekanik
   - Biaya diagnostic
   - Pajak/fee (10%)
3. Buat invoice & terima pembayaran:
   - Cash / Debit / Credit card
4. Jika pembayaran tidak cukup:
   - Negotiasi diskon?
   - Cicilan?
5. Serahkan kunci + dokumentasi (invoice, garansi service)
6. Minta feedback & rating dari pelanggan

**Waktu Proses**:
- Normal: 5-10 menit
- Ada negosiasi harga: +10 menit
- Masalah pembayaran (salah transfer, dll): +15 menit

### 📊 INPUT & OUTPUT

| Aspek | Detail |
|------|--------|
| **Input** | Mobil PASS test drive dari Orang 5 |
| **Output** | Pelanggan keluar dengan mobil, invoice dibayar |
| **Resource** | 1 admin finance, POS sistem, printer |
| **Kapasitas** | 1-2 pelanggan paralel |

### 📈 METRIK DIUKUR

- Rata-rata waktu checkout?
- % payment success rate?
- Rata-rata invoice amount?
- Customer satisfaction rating?

### 🛠️ CARA IMPLEMENTASI DI ANYLOGIC

#### **Step 1: Hitung Total Biaya**

```
Function: Calculate_Total_Cost(mobil)
  parts_cost = mobil.estimated_cost
  
  IF mobil.complexity == "Simple":
    labor_cost = 300000  // mekanik 1-2 jam
  ELSE:
    labor_cost = 600000  // mekanik 3-4 jam
  
  diagnostic_cost = 100000
  tax = (parts_cost + labor_cost + diagnostic_cost) * 0.1  // 10%
  
  total = parts_cost + labor_cost + diagnostic_cost + tax
  
  return total
```

#### **Step 2: Service: Administrasi & Bayar**

```
Service: "Bayar & Serah Mobil"
- Capacity: 2 admin finance
- Service time:

base_time = uniform(5, 10)  // normal process

IF ada_negosiasi_harga:
  base_time += uniform(10, 15)

IF ada_payment_issue:
  base_time += uniform(15, 30)

service_time = base_time
```

#### **Step 3: Payment Process**

```
Function: Process_Payment(total_amount)
  payment_method = random choice:
    - "Cash": 40%
    - "Debit Card": 50%
    - "Credit Card": 10%
  
  IF payment_method == "Debit Card" OR "Credit Card":
    // 5% chance payment fail (insufficient fund, system error)
    if uniform(0, 1) < 0.05:
      return {status: "FAILED", issue: "Payment declined"}
  
  return {status: "SUCCESS", method: payment_method}
```

#### **Step 4: Atribut Mobil ditambah**

```
Mobil attributes (tambahan):
- total_cost: double
- parts_cost_final: double
- labor_cost: double
- diagnostic_cost: double
- tax_amount: double
- payment_status: String (PENDING/SUCCESS/FAILED)
- payment_method: String (CASH/DEBIT/CREDIT)
- checkout_time: double
- customer_rating: int (1-5)
- customer_feedback: String
```

#### **Step 5: Customer Rating**

```
Function: Get_Customer_Satisfaction()
  // Tergantung total waktu di bengkel dan hasil
  total_duration = now() - mobil.waktu_penerimaan
  
  IF total_duration <= 240 menit AND mobil.rework_count == 0:
    // Cepat, first try pass
    rating = random choice: 4 atau 5 (85% rating baik)
  ELSE IF total_duration <= 480 menit:
    // Normal
    rating = random choice: 3 atau 4 (60% cukup baik)
  ELSE:
    // Lama
    rating = random choice: 2 atau 3 (40% kurang puas)
  
  return rating
```

### 📋 PARAMETER SIMULASI

```
PEMBAYARAN & SERAH MOBIL (Orang 6):
- Admin finance: 2 orang
- Service time: 5-10 menit (normal)
- Negosiasi: 15% kasus, +10 menit
- Payment issue: 5% kasus, +15 menit
- Payment methods:
  - Cash: 40%
  - Debit: 50%
  - Credit: 10%
- Payment failure rate: 5% (debit/credit)
```

### 📤 OUTPUT FINAL

```json
{
  "work_order_id": "WO-001",
  "customer_name": "Budi Hartono",
  "total_cost": 220000,
  "parts_cost": 200000,
  "labor_cost": 0,
  "tax": 22000,
  "payment_status": "SUCCESS",
  "payment_method": "CASH",
  "checkout_time": 8,
  "total_duration": 185,
  "customer_rating": 5,
  "customer_feedback": "Pelayanan cepat, mekanik profesional",
  "finish_timestamp": "11:15:00"
}
```

---

---

## 🎯 RINGKASAN 6 MODUL

| Modul | Orang | Job | Input | Output | Resource | Time |
|-------|-------|-----|-------|--------|----------|------|
| 1 | 1 | Penerimaan | Pelanggan | Work Order | 1 receptionist | 4±1 min |
| 2 | 2 | Diagnostic | Mobil | Parts List | 2 mekanik | 15-45 min |
| 3 | 3 | Siapkan Parts | Parts List | Parts Ready | 2 staff | 5-120 min |
| 4 | 4 | Perbaikan | Mobil+Parts | Selesai Perbaiki | 2-3 bay | 60-240 min |
| 5 | 5 | Test Drive | Mobil | PASS/FAIL | 1 driver | 15-30 min |
| 6 | 6 | Bayar & Serah | Hasil | Invoice+Mobil | 2 admin | 5-40 min |

---

## 📊 METRIK UTAMA YANG DIUKUR (INTEGRATION)

```
KEY METRICS:
1. Total turnaround time: Dari penerimaan → bayar (target: 4-6 jam)
2. Throughput: Berapa mobil selesai per hari? (target: 8-12 mobil)
3. Queue length: Berapa mobil menunggu per fase?
4. Utilization:
   - Receptionist: target >80%
   - Mekanik: target >70%
   - Bay: target >60%
5. Rework rate: Berapa % test drive fail? (target: <10%)
6. Revenue: Total income per hari (target: Rp 50-100 juta)
7. Customer satisfaction: Rating rata-rata (target: 4+/5)
```

---

## ✅ CHECKLIST IMPLEMENTASI ANYLOGIC

Untuk setiap orang:
- [ ] Buat Source/Input (dari orang sebelumnya)
- [ ] Tentukan atribut Entity
- [ ] Buat Service node dengan capacity & service time
- [ ] Tentukan probabilistic events (error, delay, dll)
- [ ] Collect statistics (waktu, queue length)
- [ ] Buat output/sink
- [ ] Test dengan data sample 8-12 mobil
- [ ] Integrate dengan modul orang lain
- [ ] Jalankan full simulation 1 hari operasi
- [ ] Analisis hasil & buat dashboard

---

## 🚀 NEXT STEPS

1. **Fase 1 (Week 1-2)**: Setiap orang develop modul mereka sendiri
2. **Fase 2 (Week 3)**: Integration & testing semua modul jadi satu
3. **Fase 3 (Week 4)**: Optimization & analisis skenario
   - Skenario A: Berapa bay optimal?
   - Skenario B: Berapa mekanik optimal?
   - Skenario C: Bagaimana jika parts sering out of stock?
4. **Fase 4 (Week 5)**: Final presentation & dokumentasi

---

**SEMOGA BERHASIL! 🎯**
