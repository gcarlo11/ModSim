# 📚 TUTORIAL LENGKAP: SIMULASI PEMINJAMAN BUKU DI COUNTER (AnyLogic)

**Modul**: MODUL 4 - Peminjaman (Scan & Check-out)
**Pembuat**: Orang ke-4
**Durasi Tutorial**: 2-3 jam
**Level**: Beginner to Intermediate

---

## 🎯 TUJUAN

Membuat simulasi **Counter Peminjaman Buku** di perpustakaan dengan fitur:
- Mahasiswa antri di checkout counter
- Petugas scan buku dan ID mahasiswa
- Sistem set deadline pengembalian otomatis
- Hitung total biaya (jika ada)
- Print receipt
- Output data untuk modul berikutnya

---

## 📋 DAFTAR ISI

1. [Setup Project AnyLogic](#setup-project)
2. [Membuat Entity Mahasiswa](#buat-entity)
3. [Membuat Source (Kedatangan)](#buat-source)
4. [Membuat Service (Counter Peminjaman)](#buat-service)
5. [Logic dan Atribut](#logic-atribut)
6. [Collect Statistics](#statistics)
7. [Testing dan Debugging](#testing)
8. [Run Simulasi](#run-simulasi)

---

---

## 1️⃣ SETUP PROJECT ANYLOGIC {#setup-project}

### **Step 1.1: Buka AnyLogic**

```
File → New → Simulation
Pilih: Library type "Standard"
Enter project name: "Simulasi_Perpustakaan"
```

### **Step 1.2: Rename Main Diagram**

```
Rename dari "Main" → "PeminjamanBuku"
```

Sekarang Anda punya canvas kosong untuk mulai simulasi.

### **Step 1.3: Set Simulation Parameters**

```
Tools → Model settings
Tab: Simulation
- Start time: 0
- Stop time: 480 (menit) = 8 jam simulasi
- Time unit: minute
```

**Atau bisa juga di Run configurations:**
```
Run → Simulation → New
- Name: "Hari_Normal"
- Duration: 480 menit
```

---

---

## 2️⃣ MEMBUAT ENTITY MAHASISWA {#buat-entity}

Entity adalah **object yang akan bergerak** dalam simulasi (dalam hal ini: Mahasiswa dengan buku).

### **Step 2.1: Buat Custom Entity Class**

```
Palette → Agents → Agent (klik kanan) → New → Agent Class
Name: "Mahasiswa"
```

### **Step 2.2: Tambah Atribut Mahasiswa**

Double-click agent "Mahasiswa" → Edit properties

Di bagian **Properties**, tambah atribut:

```
1. mahasiswa_id: String
   - Default: "M-0" (akan auto-increment)

2. nama: String
   - Default: "Mahasiswa " + mahasiswa_id

3. num_books: int (berapa buku yang dipinjam)
   - Default: random dari 1-5
   
4. books_list: String[] (nama-nama buku)
   - Default: ["Buku A", "Buku B"]

5. checkout_time: long (waktu mulai checkout)
   - Default: 0

6. service_duration: double (berapa lama di service)
   - Default: 0

7. loan_deadline: int (deadline return dalam hari)
   - Default: 7 (regular), 3 (reference), 1 (reserve)

8. total_cost: double (biaya peminjaman)
   - Default: 0

9. receipt_number: String
   - Default: "REC-" + receipt_counter

10. payment_method: String
    - Default: "NO_CHARGE" (perpustakaan gratis)
```

### **Step 2.3: Code Dalam Agent**

Di dalam Agent class "Mahasiswa", tambahkan method:

```java
public void generateReceipt() {
    this.receipt_number = "REC-" + System.currentTimeMillis();
}

public void setLoanDeadline(String bookType) {
    // bookType: "REGULAR", "REFERENCE", "RESERVE"
    switch(bookType) {
        case "REGULAR":
            this.loan_deadline = 7;  // 7 hari
            break;
        case "REFERENCE":
            this.loan_deadline = 3;  // 3 hari
            break;
        case "RESERVE":
            this.loan_deadline = 1;  // 1 hari
            break;
        default:
            this.loan_deadline = 7;
    }
}
```

---

---

## 3️⃣ MEMBUAT SOURCE (KEDATANGAN MAHASISWA) {#buat-source}

Source adalah **titik awal** di mana mahasiswa mulai masuk simulasi.

### **Step 3.1: Drag Source dari Palette**

```
Palette → Process Modeling → Source
Letakkan di canvas diagram
```

### **Step 3.2: Configure Source**

**Rename:** `SourceMahasiswa`

**Double-click → Edit Properties:**

#### **Tab: Main**
```
New Agent Class: Mahasiswa
Agent pool/population: mahasiswa_pool (akan dibuat otomatis)
```

#### **Tab: Arrivals**
```
Interarrival time:
  Distribution: Exponential
  Mean: 10 (menit)
  Rate: 1/10 = 0.1 per menit
  
Atau: Arrival rate
  Rate: 6 mahasiswa/jam
```

**Arti:** Rata-rata mahasiswa datang ke counter setiap 10 menit (bisa lebih cepat, bisa lebih lambat).

#### **Tab: Initial Population**
```
Create initial population: OFF (unchecked)
```

#### **Tab: Actions (On agent created)**

```java
// Di sini set atribut awal ketika mahasiswa dibuat
agent.mahasiswa_id = "M-" + source.generated;
agent.nama = "Mahasiswa " + agent.mahasiswa_id;

// Random jumlah buku yang mau dipinjam (1-5 buku)
agent.num_books = (int) uniform(1, 6);  // 1-5

// Random buku name (simulasi)
ArrayList<String> books = new ArrayList<>();
for (int i = 0; i < agent.num_books; i++) {
    books.add("Buku-" + random(1, 1000));
}
agent.books_list = books.toArray(new String[0]);

// Set deadline: 70% regular, 20% reference, 10% reserve
double rand = uniform(0, 1);
if (rand < 0.7) {
    agent.loan_deadline = 7;      // Regular
} else if (rand < 0.9) {
    agent.loan_deadline = 3;      // Reference
} else {
    agent.loan_deadline = 1;      // Reserve
}

traceln("Created: " + agent.mahasiswa_id + " with " + agent.num_books + " books");
```

---

---

## 4️⃣ MEMBUAT SERVICE (COUNTER PEMINJAMAN) {#buat-service}

Service adalah **stasiun pelayanan** di mana mahasiswa dilayani.

### **Step 4.1: Drag Service dari Palette**

```
Palette → Process Modeling → Service
Letakkan di sebelah kanan Source
```

**Rename:** `CounterCheckout`

### **Step 4.2: Connect Source ke Service**

```
Klik Source → Tarik ke Service
Otomatis membuat connection/arrow
```

### **Step 4.3: Configure Service**

**Double-click Service → Edit Properties:**

#### **Tab: Main**

```
Capacity: 2
(Artinya ada 2 counter pembayaran paralel, 2 petugas)
```

#### **Tab: Service time**

Di sini kita set **berapa lama service berlangsung**.

```
Service time type: By expression
Expression:

double service_time;
int num_books = agent.num_books;

// Service time tergantung jumlah buku
if (num_books <= 3) {
    service_time = uniform(2, 3);      // 2-3 menit (cepat)
} else if (num_books <= 5) {
    service_time = uniform(3, 4);      // 3-4 menit (normal)
} else {
    service_time = uniform(4, 6);      // 4-6 menit (banyak buku)
}

// Ada kemungkinan System error (5% kasus)
if (uniform(0, 1) < 0.05) {
    service_time += uniform(2, 3);     // Tambah 2-3 menit delay
    traceln("System error untuk " + agent.mahasiswa_id);
}

return service_time;
```

#### **Tab: On service complete (Actions)**

```java
// Di sini logic yang terjadi SETELAH service selesai

// 1. Generate receipt
String timestamp = dateToString(now(), "HH:mm:ss");
agent.receipt_number = "REC-" + timestamp + "-" + agent.mahasiswa_id;

// 2. Record waktu checkout
agent.checkout_time = (long) now();

// 3. Set due date (berapa hari dari sekarang)
// Misal: hari ini 2025-01-15, due date = hari 22 (7 hari kemudian)
// (Bisa pakai variable global untuk tracking)

// 4. Hitung total cost (untuk perpus, biasanya free, tapi ada yg charge)
agent.total_cost = 0;  // Perpustakaan gratis

// 5. Catat di log
traceln("CHECKOUT: ID=" + agent.mahasiswa_id + 
        " Books=" + agent.num_books + 
        " Receipt=" + agent.receipt_number + 
        " Deadline=" + agent.loan_deadline + " hari");

// 6. (Optional) Tracking untuk statistik
// (akan dibuat di section Statistics)
```

---

---

## 5️⃣ LOGIC DAN ATRIBUT {#logic-atribut}

### **Step 5.1: Buat Sink (Output/Exit Point)**

```
Palette → Process Modeling → Sink
Drag ke canvas, sebelah kanan Service
```

**Rename:** `SinkExit`

### **Step 5.2: Connect Service ke Sink**

```
Service → Tarik ke Sink
```

Ini berarti mahasiswa keluar dari simulasi setelah checkout.

### **Step 5.3: Code dalam Sink (Optional)**

**Double-click Sink → Edit Actions → On entering:**

```java
// Ini opsional, untuk recording final data
traceln("EXIT: " + agent.mahasiswa_id + 
        " - Total time: " + (now() - agent.checkout_time) + " minutes");
```

### **Step 5.4: Buat Variabel Global (Data Collection)**

Di Object properties panel (sebelah kanan), tambah variabel:

```
Global variables:

1. total_mahasiswa: int = 0
   (track total mahasiswa yang selesai checkout)

2. total_buku_dipinjam: int = 0
   (track total buku yang dipinjam semua orang)

3. avg_checkout_time: double = 0
   (rata-rata waktu checkout)

4. max_queue_length: int = 0
   (queue paling panjang)
```

### **Step 5.5: Update Variabel di Sink**

**Di Sink Actions → On entering:**

```java
total_mahasiswa++;
total_buku_dipinjam += agent.num_books;
avg_checkout_time = (avg_checkout_time * (total_mahasiswa - 1) + agent.service_duration) / total_mahasiswa;
```

---

---

## 6️⃣ COLLECT STATISTICS {#statistics}

Statistics untuk **mengukur performa simulasi**.

### **Step 6.1: Buat Data Collection**

Di Object properties → Variables, tambah:

```
1. stat_service_time: Average
   Track: agent.service_duration
   
2. stat_queue_length: Timeseries
   Track: panjang queue di CounterCheckout
   
3. stat_utilization: Average
   Track: CounterCheckout.resourceUtilization()
   
4. stat_books_per_mahasiswa: Average
   Track: agent.num_books
```

### **Step 6.2: Collect Queue Length**

Di Object properties → Datasets, tambah:

```
Queue Length Dataset:
- Source: CounterCheckout
- Metric: Queue Length
- Refresh rate: Every 1 minute
- Chart type: Timeseries
```

### **Step 6.3: Buat Function untuk Statistics**

```java
public double getAvgServiceTime() {
    return stat_service_time.mean();
}

public double getMaxQueueLength() {
    return stat_queue_length.max();
}

public double getUtilization() {
    return stat_utilization.mean() * 100;  // Convert to percentage
}

public double getAvgBooksPerMahasiswa() {
    return stat_books_per_mahasiswa.mean();
}
```

---

---

## 7️⃣ TESTING DAN DEBUGGING {#testing}

### **Step 7.1: Setup Trace Output**

```
Tools → Preferences → Running Experiments
Enable "Show trace" (checklist)
Simulation can output trace messages dengan traceln()
```

### **Step 7.2: Run Simulation Test (Short Run)**

```
Run → Simulation → Run
Duration: 60 menit (test singkat)
Speed: 1x atau 2x (lihat log terlebih dahulu)
```

**Expected Output di Trace/Console:**
```
Created: M-1 with 3 books
Created: M-2 with 5 books
...
CHECKOUT: ID=M-1 Books=3 Receipt=REC-09:15:30-M-1 Deadline=7 hari
CHECKOUT: ID=M-2 Books=5 Receipt=REC-09:18:45-M-2 Deadline=3 hari
EXIT: M-1 - Total time: 2.5 minutes
EXIT: M-2 - Total time: 3.2 minutes
```

### **Step 7.3: Check Queue Behavior**

Jalankan simulasi dan lihat:
- Queue panjang tidak panjang?
- Service time reasonable?
- Trace messages muncul dengan benar?
- No errors di output?

### **Step 7.4: Debugging Tips**

**Jika Queue terlalu panjang:**
```
// Tambah capacity di Service
CounterCheckout.setCapacity(3);  // dari 2 jadi 3
```

**Jika Service time tidak pas:**
```
// Adjust di Service time expression
if (num_books <= 3) {
    service_time = uniform(2, 3);  // Ubah range ini
}
```

**Jika trace tidak muncul:**
```
// Pastikan pakai traceln()
System.out.println() TIDAK akan muncul di Trace
traceln() AKAN muncul di Trace
```

---

---

## 8️⃣ RUN SIMULASI PENUH {#run-simulasi}

### **Step 8.1: Set Skenario**

```
Run configurations:
Name: "Hari_Normal_Weekday"
Stop time: 480 menit (8 jam operasi: 08:00-17:00)
Runs: 1 (single run untuk analisis detail)
```

### **Step 8.2: Run**

```
Run → Simulation → Hari_Normal_Weekday
```

Biarkan simulasi jalan sampai selesai.

### **Step 8.3: Analisis Hasil**

Setelah selesai, lihat output:

```
METRIK KUNCI:

1. Total Mahasiswa yang selesai checkout: total_mahasiswa
   → Expected: ~48 mahasiswa (6 per jam × 8 jam)

2. Rata-rata Service Time: getAvgServiceTime()
   → Expected: 3-4 menit

3. Max Queue Length: stat_queue_length.max()
   → Expected: 5-10 orang (jika 2 counter, cukup)

4. Utilization: getUtilization()
   → Expected: 60-80% (optimal)

5. Rata-rata Buku per Mahasiswa: getAvgBooksPerMahasiswa()
   → Expected: 2.5-3 buku
```

### **Step 8.4: Generate Report**

```
Tools → Export Results
Option: Export to CSV / Excel
Simpan dengan nama: "Peminjaman_Buku_Results.csv"
```

---

---

## 📊 FINAL STRUCTURE

Canvas diagram Anda seharusnya seperti ini:

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────┐
│  Source         │────▶│  Service         │────▶│  Sink    │
│ (Mahasiswa⏰)   │     │ (Counter Checkout)│    │ (Exit)   │
│                 │     │ Capacity: 2      │     │          │
│ Arrivals:       │     │ Service time:    │     │          │
│ Exp(10 min)     │     │ 2-6 min          │     │          │
└─────────────────┘     └──────────────────┘     └──────────┘
                              ▲
                              │
                         Queue
                      (max ~10 orang)
```

**Properties Panel:**
- Variables untuk tracking
- Datasets untuk statistik
- Functions untuk reporting

---

---

## ✅ CHECKLIST IMPLEMENTASI

- [ ] 1. Setup project AnyLogic
- [ ] 2. Buat entity Mahasiswa dengan 10 atribut
- [ ] 3. Buat Source dengan Exponential distribution
- [ ] 4. Configure Source actions (set atribut awal)
- [ ] 5. Buat Service (Counter) dengan capacity 2
- [ ] 6. Set Service time expression (tergantung num_books)
- [ ] 7. Buat Sink untuk exit
- [ ] 8. Tambah global variables untuk tracking
- [ ] 9. Setup trace output (traceln)
- [ ] 10. Test dengan 60 menit simulasi
- [ ] 11. Check trace output dan queue
- [ ] 12. Adjust service time / capacity jika perlu
- [ ] 13. Run full simulation (480 menit)
- [ ] 14. Analisis hasil & metrics
- [ ] 15. Export hasil ke CSV

---

---

## 🎓 NEXT STEPS

Setelah modul ini selesai:

1. **Integrate dengan Modul Lain:**
   - Input dari Modul 3 (Ambil Buku)?
   - Output ke Modul 5 (Ruang Baca)?

2. **Add Complexity:**
   - Payment processing (jika ada biaya)
   - Book type differentiation (regular/reference/reserve)
   - Error handling (system down, printer jam, dll)

3. **Optimize:**
   - Berapa counter optimal? (2 atau 3?)
   - Peak hour management
   - Service time tuning

---

## 📌 COMMON ISSUES & SOLUTIONS

| Issue | Penyebab | Solusi |
|-------|---------|--------|
| Trace tidak muncul | Pakai `System.out.println()` | Pakai `traceln()` |
| Queue sangat panjang | Service time terlalu lama / capacity kurang | Turunkan service time atau tambah capacity |
| Mahasiswa tidak keluar | Sink tidak terconnect | Hubungkan Service ke Sink |
| Atribut tidak terupdate | Action block tidak di-trigger | Check "On service complete" actions |
| Statistik tidak recorded | Tidak pakai `stat_*` variable | Buat variabel dataset dan track dengan `agent.*` |

---

## 🚀 FULL CODE REFERENCE

**Jika ingin lihat semua code together (lebih cepat copy-paste):**

### **Source - On agent created:**
```java
agent.mahasiswa_id = "M-" + source.generated;
agent.nama = "Mahasiswa " + agent.mahasiswa_id;
agent.num_books = (int) uniform(1, 6);

ArrayList<String> books = new ArrayList<>();
for (int i = 0; i < agent.num_books; i++) {
    books.add("Buku-" + random(1, 1000));
}
agent.books_list = books.toArray(new String[0]);

double rand = uniform(0, 1);
if (rand < 0.7) {
    agent.loan_deadline = 7;
} else if (rand < 0.9) {
    agent.loan_deadline = 3;
} else {
    agent.loan_deadline = 1;
}

traceln("Created: " + agent.mahasiswa_id + " with " + agent.num_books + " books");
```

### **Service - Service time:**
```java
double service_time;
int num_books = agent.num_books;

if (num_books <= 3) {
    service_time = uniform(2, 3);
} else if (num_books <= 5) {
    service_time = uniform(3, 4);
} else {
    service_time = uniform(4, 6);
}

if (uniform(0, 1) < 0.05) {
    service_time += uniform(2, 3);
    traceln("System error untuk " + agent.mahasiswa_id);
}

return service_time;
```

### **Service - On service complete:**
```java
String timestamp = dateToString(now(), "HH:mm:ss");
agent.receipt_number = "REC-" + timestamp + "-" + agent.mahasiswa_id;
agent.checkout_time = (long) now();
agent.total_cost = 0;

traceln("CHECKOUT: ID=" + agent.mahasiswa_id + 
        " Books=" + agent.num_books + 
        " Receipt=" + agent.receipt_number + 
        " Deadline=" + agent.loan_deadline + " hari");
```

### **Sink - On entering:**
```java
total_mahasiswa++;
total_buku_dipinjam += agent.num_books;
avg_checkout_time = (avg_checkout_time * (total_mahasiswa - 1) + agent.service_duration) / total_mahasiswa;

traceln("EXIT: " + agent.mahasiswa_id + 
        " - Service time: " + agent.service_duration + " minutes");
```

---

**SELESAI! Sekarang Anda bisa membuat simulasi modul 4 (Peminjaman Buku) di AnyLogic! 🎯**

Jika ada pertanyaan, tanya saja. Semoga berhasil! 🚀
