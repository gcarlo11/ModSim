# KONSEP SIMULASI SUPPLY CHAIN - AnyLogic
## Untuk 5 Anggota Tim

**Topik Utama:** Simulasi end-to-end proses supply chain sebuah perusahaan manufaktur dengan 5 modul independen tapi terintegrasi.

---

## 📋 OVERVIEW SISTEM

```
[SUPPLIER] → [MANUFACTURING] → [WAREHOUSE] → [DISTRIBUTION] → [RETAIL/CUSTOMER]
   Modul 1       Modul 2          Modul 3        Modul 4          Modul 5
```

**Objektif Simulasi:**
- Analisis efficiency keseluruhan supply chain
- Identifikasi bottleneck di setiap stage
- Optimasi inventory level
- Minimize total cost dan lead time

---

## 🔧 DESKRIPSI 5 MODUL

### **MODUL 1: SUPPLIER (Anggota 1)**

**Deskripsi:**
Modul ini mensimulasikan supplier yang menerima purchase order dari manufacturing, memproses order, dan mengirimkan raw material.

**Elemen Utama:**
- **Incoming Orders** - PO datang dari Manufacturing dengan quantity dan jenis material
- **Warehouse Supplier** - Raw material tersimpan di supplier warehouse
- **Processing** - Picking & packing order, QC basic
- **Shipping** - Pengiriman dengan lead time variabel (2-5 hari)

**Entity yang diproses:** Purchase Order (PO)

**Input dari modul lain:** Order requests dari Manufacturing

**Output:** Shipment dengan material yang dikirim ke Manufacturing

**KPI yang diukur:**
- Order fulfillment time
- On-time delivery rate
- Supplier inventory level
- Stock-out frequency

**Perkiraan Setup:** 
- 3-4 agen/resource (Warehouse staff, QC, Truck)
- Konstanta: Lead time, stock initial, order processing time

---

### **MODUL 2: MANUFACTURING (Anggota 2)**

**Deskripsi:**
Modul ini menerima raw material dari supplier, melakukan proses manufacturing, dan menghasilkan finished goods untuk warehouse.

**Elemen Utama:**
- **Receiving Dock** - Material masuk dari supplier, inspeksi
- **Production Line** - 3-4 assembly workstations
- **Quality Control** - Inspect finished goods, reject rate 5-10%
- **Finished Goods Buffer** - Temporary storage sebelum ke warehouse

**Entity yang diproses:** Finished Goods (units)

**Input dari modul lain:** Raw materials dari Supplier

**Output:** Finished goods dikirim ke Warehouse, Order requests ke Supplier

**KPI yang diukur:**
- Production rate (units/day)
- Average cycle time
- Defect rate
- Machine utilization
- Inventory di buffer

**Perkiraan Setup:**
- 5-6 resource (workstation, QC inspector, forklift)
- Konstanta: Processing time per unit, defect rate, batch size

---

### **MODUL 3: WAREHOUSE (Anggota 3)**

**Deskripsi:**
Modul ini mengelola inventory finished goods, menerima dari manufacturing, dan mendistribusikan ke sales/distribution.

**Elemen Utama:**
- **Receiving Area** - Penerimaan finished goods dari manufacturing
- **Storage System** - Rak dengan kapasitas terbatas (FIFO/LIFO)
- **Inventory Tracking** - Real-time inventory level
- **Picking Station** - Ambil barang sesuai demand dari Distribution
- **Shipping Area** - Persiapan pengiriman ke Distribution Center

**Entity yang diproses:** Finished Goods (units)

**Input dari modul lain:** 
- Goods dari Manufacturing
- Demand dari Distribution/Retail

**Output:** 
- Replenishment orders ke Manufacturing
- Goods shipment ke Distribution

**KPI yang diukur:**
- Inventory level trends
- Warehouse utilization (%)
- Pick accuracy rate
- Days of supply (DOS)
- Stock-out incidents

**Perkiraan Setup:**
- 4-5 resource (Receiving staff, Warehouse operator, Picker)
- Konstanta: Warehouse capacity, shelf life, FIFO rule, picking time

---

### **MODUL 4: DISTRIBUTION (Anggota 4)**

**Deskripsi:**
Modul ini mengelola distribution center, menerima goods dari warehouse pusat, dan mendistribusikan ke retail outlets di berbagai lokasi.

**Elemen Utama:**
- **DC Receiving** - Barang masuk dari Warehouse Pusat
- **DC Storage** - Short-term buffer inventory
- **Route Planning** - Menentukan rute delivery ke retail outlets (3-5 locations)
- **Vehicle Management** - Truck/van dengan kapasitas terbatas
- **Last-mile Delivery** - Pengiriman final ke retail

**Entity yang diproses:** Shipment (pallets/boxes)

**Input dari modul lain:**
- Goods dari Warehouse Pusat
- Demand dari Retail stores

**Output:**
- Goods ke Retail stores
- Stock status balik ke Warehouse Pusat

**KPI yang diukur:**
- Delivery on-time rate
- Average delivery time per location
- Vehicle utilization
- Fuel cost / distance traveled
- Out-of-stock at retail

**Perkiraan Setup:**
- 4-5 resource (DC staff, Drivers, Routing optimizer)
- Konstanta: Lead time per route (1-3 hari), vehicle capacity, demand pattern per location

---

### **MODUL 5: RETAIL/CUSTOMER (Anggota 5)**

**Deskripsi:**
Modul ini mensimulasikan retail stores dan customer demand. Memproses pembelian customer dan mengelola retail inventory.

**Elemen Utama:**
- **Customer Arrival** - Customers datang ke stores (stochastic)
- **Point of Sale (POS)** - Proses checkout purchase
- **Retail Shelf** - Display product di toko
- **Inventory Management** - Reorder dari Distribution ketika stock rendah
- **Lost Sales** - Jika stock habis, customer pergi (lost opportunity)

**Entity yang diproses:** Customer transactions

**Input dari modul lain:**
- Customer demand signals
- Stock dari Distribution

**Output:**
- Demand forecast/actual sales ke Distribution
- Revenue generated

**KPI yang diukur:**
- Sales volume
- Average inventory level
- Shelf stock-out rate
- Lost sales (%)
- Customer satisfaction (stock availability)
- Revenue per store

**Perkiraan Setup:**
- 3-4 resource (Cashier, Inventory staff, Store manager)
- Konstanta: Customer arrival rate (hourly), purchase probability, demand pattern

---

## 🔄 DATA FLOW & INTEGRASI

```
SUPPLIER
  ↓ (Raw Material)
  ├─→ MANUFACTURING
       ↓ (Finished Goods)
       └─→ WAREHOUSE (Central)
            ↓ (Goods Distribution)
            └─→ DISTRIBUTION CENTER
                 ↓ (Last Mile Delivery)
                 └─→ RETAIL STORES
                      ↓ (Customer Demand/Sales)
                      └─→ Feedback ke Warehouse (Replenishment signal)
```

**Connection Points:**

| Dari | Ke | Apa yang ditransfer | Format |
|------|-----|-------------------|--------|
| Supplier | Manufacturing | Raw Material (units) | Shipment entity |
| Manufacturing | Warehouse | Finished Goods | Batch/Pallet |
| Warehouse | Distribution | Goods | Shipment order |
| Distribution | Retail | Products | Load per truck |
| Retail | Distribution | Demand signal | Stock level alert |
| Retail | Warehouse | Sales data | Daily report |
| Manufacturing | Supplier | Purchase Order | PO entity |

---

## 📊 SISTEM GLOBAL PARAMETER

**Semua modul akan mengakses parameter ini:**

```
// Time constants
LEAD_TIME_SUPPLIER_TO_MFG = 3 days (random: 2-5)
LEAD_TIME_MFG_TO_WH = 1 day
LEAD_TIME_WH_TO_DC = 2 days (random: 1-3)
LEAD_TIME_DC_TO_RETAIL = {1 day untuk Local, 3 days untuk Regional}

// Demand
DAILY_CUSTOMER_DEMAND = 500 units (normal distribution)
DEMAND_VARIABILITY = 20%

// Costs
UNIT_PRODUCTION_COST = $10
UNIT_HOLDING_COST = $0.50/day
UNIT_SHORTAGE_COST = $5 (lost sale)
TRANSPORTATION_COST = $0.10/unit

// Quality
MFG_DEFECT_RATE = 5%
QC_REJECT_RATE = 2%

// Capacity
WAREHOUSE_CAPACITY = 10,000 units
DC_CAPACITY = 5,000 units
RETAIL_SHELF_CAPACITY = 500 units/store
PRODUCTION_CAPACITY = 1,000 units/day
```

---

## 🎯 TUJUAN SIMULASI & KPI UTAMA

**Overall Supply Chain KPI:**
1. **Total Lead Time** - Dari supplier order → customer punya di tangan (target: < 12 hari)
2. **Fill Rate** - % order yang fulfill tepat waktu (target: > 95%)
3. **Total Cost** - Production + Holding + Transportation + Shortage (minimize)
4. **Inventory Turns** - Berapa kali inventory rotating per tahun (maximize)
5. **Customer Service Level** - % retail yang tidak stockout (target: > 98%)

**Skenario Simulasi:**
- **Baseline Scenario** - Normal operation current state
- **Demand Spike** - Sudden 50% increase in demand kelihatan gimana sistem response?
- **Supplier Delay** - Lead time supplier naik jadi 10 hari, impact apa?
- **Capacity Increase** - Tambahin production capacity jadi 1500 units/day, savings apa?

---

## 💼 DELIVERABLES SETIAP ANGGOTA

### Anggota 1 (Supplier Module):
- [ ] AnyLogic model (Supplier.alp)
- [ ] Documentation: Supplier flow, assumptions, KPI
- [ ] Test report: Verify output correctly to Manufacturing

### Anggota 2 (Manufacturing Module):
- [ ] AnyLogic model (Manufacturing.alp)
- [ ] Documentation: Production process, QC logic
- [ ] Interface definition: Input format from Supplier, Output format to Warehouse

### Anggota 3 (Warehouse Module):
- [ ] AnyLogic model (Warehouse.alp)
- [ ] Documentation: Inventory rules, reorder logic
- [ ] Integration test: Receive from Manufacturing, send to Distribution

### Anggota 4 (Distribution Module):
- [ ] AnyLogic model (Distribution.alp)
- [ ] Documentation: Route optimization, vehicle usage
- [ ] KPI dashboard: On-time delivery, utilization metrics

### Anggota 5 (Retail/Customer Module):
- [ ] AnyLogic model (Retail.alp)
- [ ] Documentation: Demand generation, customer behavior
- [ ] Sales report: Revenue, stock-out incidents

### Team (Integration):
- [ ] Master model (SupplyChain_Master.alp) - Gabungan semua modul
- [ ] Dashboard: Overall KPI visualization
- [ ] Report: Scenario analysis & recommendations

---

## 🔧 TECHNICAL SETUP TIPS

1. **Shared Database/Excel:**
   - Buat parameter file (Excel) yang shared, semua modul baca dari sini
   - Mudah adjust parameter untuk simulasi scenario berbeda

2. **Entity Definition:**
   - Tentukan structure entity (Product, Shipment, Order) yang konsisten
   - Dokumentasi attribute yang dibawa entity through supply chain

3. **Integration Strategy:**
   - Use AnyLogic "Embedded simulation" atau "model import"
   - Atau: Setiap modul standalone, hasil di-export ke Excel, master model import dan orchestrate

4. **Testing Protocol:**
   - Unit test setiap modul individual dulu
   - Integration test: data flow antar modul correct
   - End-to-end test: full simulation dengan berbagai scenario

5. **Version Control:**
   - Gunakan naming convention: `SupplyChain_Modul1_v1.1.alp`
   - Agree on final values sebelum integration

---

## 📈 EXPECTED OUTCOME

Setelah integration, Tim akan memiliki:
✅ Fully functional supply chain simulation
✅ Interactive dashboard showing real-time KPI
✅ Scenario comparison (baseline vs optimization)
✅ Business insights (bottleneck analysis, cost drivers)
✅ Professional documentation & presentation
✅ Model reusable untuk industry case studies berbeda

---

## 🚀 TIMELINE ESTIMASI

| Phase | Waktu | Owner |
|-------|--------|-------|
| Individual Module Development | 2-3 weeks | Setiap anggota |
| Integration & Testing | 1 week | All team |
| Dashboard & Reporting | 1 week | Team lead |
| Final Presentation | 1 week | All team |

**Total: 5-6 weeks**

---

Siap untuk dimulai? Setiap anggota bisa langsung mulai develop modulnya berdasarkan spec ini! 🎯

