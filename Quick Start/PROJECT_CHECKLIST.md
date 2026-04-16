# PROJECT CHECKLIST - SIMULASI SUPPLY CHAIN
## 5-Member Team Implementation

---

## 📌 KICK-OFF MEETING
- [ ] Semua anggota memahami konsep overall
- [ ] Disepakati: Topik = Supply Chain, 5 Modul
- [ ] Disepakati: Timeline (5-6 minggu)
- [ ] Disepakati: Meeting schedule (weekly standup)
- [ ] Dibuat WhatsApp group / email untuk koordinasi
- [ ] Repository setup (Drive folder atau GitHub untuk share file)

---

## 👤 INDIVIDUAL ASSIGNMENT (MINGGU 1-3)

### ✅ ANGGOTA 1 - SUPPLIER MODULE
**Role:** Supplier Management System

**Pre-Development:**
- [ ] Baca SUPPLY_CHAIN_CONCEPT.md section "MODUL 1"
- [ ] Definisikan: Entity PO, Shipment, Warehouse Supplier
- [ ] Tentukan parameter awal (lead time, stock level, order processing time)

**Development:**
- [ ] Create AnyLogic project: `Supplier_Module.alp`
- [ ] Model incoming orders dari Manufacturing
- [ ] Model supplier warehouse dengan inventory logic
- [ ] Model QC/processing stage
- [ ] Model shipping dengan lead time variability
- [ ] Create output untuk tracking PO status
- [ ] Add KPI chart: Order fulfillment time, On-time delivery rate

**Testing:**
- [ ] Unit test: Order arrival processing
- [ ] Unit test: Stock availability check
- [ ] Unit test: Shipping lead time randomness
- [ ] Integration test interface: Output format siap untuk Manufacturing menerima

**Documentation:**
- [ ] Document: `Supplier_Module_Specification.docx`
  - Assumptions, parameters, flow diagram
  - Entity definitions, output format
  

**Deliverables (End of Week 3):**
- [ ] `Supplier_Module_v1.0.alp` (AnyLogic file)
- [ ] `Supplier_Parameters.xlsx` (Parameter values)
- [ ] `Supplier_Output_Sample.xlsx` (Sample output)
- [ ] `Supplier_Specification.docs` (Full documentation)

---

### ✅ ANGGOTA 2 - MANUFACTURING MODULE
**Role:** Production System

**Pre-Development:**
- [ ] Baca SUPPLY_CHAIN_CONCEPT.md section "MODUL 2"
- [ ] Definisikan: Entity RawMaterial, FinishedGoods, Batch
- [ ] Design production line: 3-4 workstations, resources, processing time
- [ ] Tentukan: Defect rate, QC reject rate

**Development:**
- [ ] Create AnyLogic project: `Manufacturing_Module.alp`
- [ ] Model receiving dock untuk raw materials
- [ ] Model 3-4 assembly stations (paralel processing)
- [ ] Model QC stage dengan accept/reject logic
- [ ] Model buffer storage untuk finished goods
- [ ] Create demand generation internal (connected to Warehouse)
- [ ] Add KPI chart: Production rate, cycle time, defect rate, machine utilization

**Testing:**
- [ ] Unit test: Item processing through workstations
- [ ] Unit test: QC defect rate accuracy
- [ ] Unit test: Buffer capacity limits
- [ ] Integration test: Receive raw materials format dari Supplier
- [ ] Integration test: Output finished goods format untuk Warehouse

**Documentation:**
- [ ] Document: `Manufacturing_Module_Specification.docx`
  - Production flow, workstation details, assumptions
  - Defect rate logic, buffer management
  

**Deliverables (End of Week 3):**
- [ ] `Manufacturing_Module_v1.0.alp`
- [ ] `Manufacturing_Parameters.xlsx`
- [ ] `Manufacturing_Output_Sample.xlsx`
- [ ] `Manufacturing_Specification.docs`

---

### ✅ ANGGOTA 3 - WAREHOUSE MODULE
**Role:** Inventory Management

**Pre-Development:**
- [ ] Baca SUPPLY_CHAIN_CONCEPT.md section "MODUL 3"
- [ ] Definisikan: Entity FinishedGoods, StorageSlot, ReplenishmentOrder
- [ ] Design warehouse: Capacity limit, FIFO rule, shelf organization
- [ ] Tentukan: Min reorder level, reorder quantity logic

**Development:**
- [ ] Create AnyLogic project: `Warehouse_Module.alp`
- [ ] Model receiving area dari Manufacturing
- [ ] Model storage system dengan FIFO tracking
- [ ] Model inventory counter (real-time level)
- [ ] Model picking station untuk demand dari Distribution
- [ ] Implement automatic reorder logic ke Manufacturing
- [ ] Add KPI chart: Inventory level trend, warehouse utilization, days of supply

**Testing:**
- [ ] Unit test: Item receiving dan FIFO sorting
- [ ] Unit test: Picking accuracy
- [ ] Unit test: Reorder trigger threshold
- [ ] Integration test: Receive goods dari Manufacturing
- [ ] Integration test: Send shipment ke Distribution dengan correct format

**Documentation:**
- [ ] Document: `Warehouse_Module_Specification.docx`
  - Inventory management logic, FIFO rules
  - Reorder point calculation, storage constraints
  

**Deliverables (End of Week 3):**
- [ ] `Warehouse_Module_v1.0.alp`
- [ ] `Warehouse_Parameters.xlsx`
- [ ] `Warehouse_Output_Sample.xlsx`
- [ ] `Warehouse_Specification.docs`

---

### ✅ ANGGOTA 4 - DISTRIBUTION MODULE
**Role:** Distribution Network

**Pre-Development:**
- [ ] Baca SUPPLY_CHAIN_CONCEPT.md section "MODUL 4"
- [ ] Definisikan: Entity Shipment, DeliveryRoute, Vehicle
- [ ] Design DC: Receiving area, storage, routing center
- [ ] Tentukan: 3-5 retail locations, lead time per route, vehicle capacity

**Development:**
- [ ] Create AnyLogic project: `Distribution_Module.alp`
- [ ] Model DC receiving dari Warehouse Pusat
- [ ] Model route planning untuk multiple retail locations
- [ ] Model vehicle dispatch dengan capacity constraint
- [ ] Model delivery dengan lead time variability per route
- [ ] Model stock status feedback ke Warehouse Pusat
- [ ] Add KPI chart: On-time delivery %, vehicle utilization, delivery cost

**Testing:**
- [ ] Unit test: Vehicle loading capacity logic
- [ ] Unit test: Route assignment optimization
- [ ] Unit test: Lead time distribution per location
- [ ] Integration test: Receive shipment format dari Warehouse
- [ ] Integration test: Send delivery format ke Retail module

**Documentation:**
- [ ] Document: `Distribution_Module_Specification.docx`
  - Route design, vehicle management, optimization logic
  - Lead time assumptions per location
  

**Deliverables (End of Week 3):**
- [ ] `Distribution_Module_v1.0.alp`
- [ ] `Distribution_Parameters.xlsx`
- [ ] `Distribution_Output_Sample.xlsx`
- [ ] `Distribution_Specification.docs`

---

### ✅ ANGGOTA 5 - RETAIL MODULE
**Role:** Customer & Sales

**Pre-Development:**
- [ ] Baca SUPPLY_CHAIN_CONCEPT.md section "MODUL 5" & MODUL5_RETAIL_CUSTOMER_DETAIL.md
- [ ] Definisikan: Entity Customer, SaleTransaction, Product
- [ ] Design 3 retail stores dengan berbeda demand characteristics
- [ ] Tentukan: Customer arrival rate, purchase behavior, operating hours

**Development:**
- [ ] Create AnyLogic project: `Retail_Module.alp`
- [ ] Model customer arrival dengan time-of-day variation (peak/off-peak)
- [ ] Model shopping behavior: browse time, purchase decision
- [ ] Model POS system: checkout, transaction recording
- [ ] Model inventory shelf management: stock depletion, reorder trigger
- [ ] Model partial fulfillment ka lost sales logic
- [ ] Add KPI chart: Daily revenue, stockout rate, conversion rate, customer satisfaction

**Testing:**
- [ ] Unit test: Customer arrival rate distribution
- [ ] Unit test: Stockout vs. partial fulfillment decision logic
- [ ] Unit test: Automatic reorder trigger
- [ ] Integration test: Receive restock format dari Distribution
- [ ] Integration test: Send sales/demand report format ke distribution

**Documentation:**
- [ ] Document: `Retail_Module_Specification.docx` (detailed spec sudah di file MODUL5)
  - Customer behavior model, demand pattern
  - Reorder logic, stockout handling
  

**Deliverables (End of Week 3):**
- [ ] `Retail_Module_v1.0.alp`
- [ ] `Retail_Parameters.xlsx`
- [ ] `Retail_Output_Sample.xlsx`
- [ ] `Retail_Specification.docs`

---

## 🔗 INTEGRATION PHASE (MINGGU 4)

### Joint Tasks - All Members
**Week 4 Activities:**

#### Step 1: Interface Definition (Start of Week 4)
- [ ] Finalize data format agreement untuk inter-module communication
- [ ] Create Excel template for all data exchanges
- [ ] Example: Format untuk Supplier→Manufacturing order
- [ ] Example: Format untuk Manufacturing→Warehouse shipment
- [ ] Example: Format untuk Retail demand signal

```
Data Exchange Format Template:
── Order/Shipment Record ─────────────────────
| Timestamp | SourceID | DestinationID | ItemID | 
| Quantity | UnitPrice | TotalCost | Status |
────────────────────────────────────────────
```

#### Step 2: Integration Testing (Mid Week 4)
- [ ] **ANGGOTA 1 & 2:**
  - [ ] Test Supplier output → Manufacturing input compatibility
  - [ ] Verify data format matching
  - [ ] Run mini-simulation: Supplier order processing through Manufacturing receiving
  
- [ ] **ANGGOTA 2 & 3:**
  - [ ] Test Manufacturing output → Warehouse input
  - [ ] Verify FinishedGoods entity flow
  - [ ] Run mini-simulation: Production batch through warehouse receiving
  
- [ ] **ANGGOTA 3 & 4:**
  - [ ] Test Warehouse output → Distribution input
  - [ ] Verify shipment data matching
  - [ ] Run mini-simulation: Warehouse replenishment reaching DC
  
- [ ] **ANGGOTA 4 & 5:**
  - [ ] Test Distribution output → Retail input
  - [ ] Verify delivery arrival at stores
  - [ ] Run mini-simulation: Shipment delivery → inventory update

#### Step 3: Create Master Model (Mid-Late Week 4)
- [ ] Designate: 1 person sebagai Integration Lead (suggest: Anggota 3)
- [ ] Create master project: `SupplyChain_Master_v1.0.alp`
- [ ] Strategy: Embedded simulation atau sequential model linking
  - Option A: All modules dalam 1 file (simpler)
  - Option B: Linked models dengan data exchange (more modular)
- [ ] Setup global parameter file: `Global_Parameters.xlsx`
- [ ] Create master-level KPI dashboard

#### Step 4: End-to-End Simulation (Late Week 4)
- [ ] Run full supply chain simulation: 1 simulated week
- [ ] Verify data flow: Supplier → Manufacturing → Warehouse → Distribution → Retail → Demand feedback
- [ ] Validate KPI calculations
- [ ] Identify & resolve any discrepancies

---

## 📊 VALIDATION & DASHBOARD (MINGGU 5)

### Phase 1: Dashboard Creation
- [ ] **Dashboard Developer** (assign to detail-oriented member):
  - [ ] Create Excel dashboard with pivot tables
  - [ ] KPI visualization: Revenue trend, inventory levels, stockout rate
  - [ ] Store-by-store comparison charts
  - [ ] Bottleneck analysis visualization
  
- [ ] **AnyLogic Animation:**
  - [ ] Create visual animation showing flow through stages
  - [ ] Color coding: Bottlenecks, idle resources, full utilization
  - [ ] Real-time KPI display overlay

### Phase 2: Scenario Simulation
Run 3 scenarios, compare results:

**Scenario 1: Baseline**
- [ ] Current state, normal demand
- [ ] Expected: Balanced supply-demand
- [ ] Measure: Baseline KPI

**Scenario 2: Demand Spike** 
- [ ] Customer demand +50%
- [ ] Expected: Possible stockouts, revenue increase
- [ ] Question: Can system handle? Bottleneck?
- [ ] Measure: Stockout rate, lost revenue, overtime cost

**Scenario 3: Supply Disruption**
- [ ] Supplier lead time doubled (2-5 days → 4-10 days)
- [ ] Expected: Inventory safety stock needed
- [ ] Question: What inventory level required to maintain service?
- [ ] Measure: Additional holding cost vs. lost sales prevention

- [ ] Document results in: `Scenario_Analysis_Report.docx`

---

## 📝 REPORTING & PRESENTATION (MINGGU 5-6)

### Deliverables

#### Phase 1: Technical Documentation
- [ ] **Master Model Document:**
  - [ ] Architecture & integration approach
  - [ ] Global parameter definitions
  - [ ] Inter-module interfaces
  - [ ] File: `SupplyChain_Integration_Guide.docx`

- [ ] **Results Package:**
  - [ ] Individual module outputs (from each member)
  - [ ] Consolidated KPI report
  - [ ] File: `SupplyChain_Simulation_Results.xlsx`

- [ ] **Scenario Analysis Report:**
  - [ ] Baseline vs. Scenario 2 vs. Scenario 3 comparison
  - [ ] Recommendations for optimization
  - [ ] File: `Scenario_Analysis_Report.docx`

#### Phase 2: Presentation Materials
- [ ] PowerPoint presentation (30-40 slides):
  - Slide 1-5: Project overview & objectives
  - Slide 6-15: Individual module descriptions (3 slides each)
  - Slide 16-20: Integration architecture & data flow
  - Slide 21-30: Simulation results & KPI dashboards
  - Slide 31-35: Scenario analysis findings
  - Slide 36-40: Conclusions & recommendations

- [ ] Live Demo video (5-10 minutes):
  - Animation of supply chain operation
  - Dashboard showing real-time KPI
  - Scenario comparison side-by-side

#### Phase 3: Final Summary
- [ ] **Executive Summary (1 page):**
  - Top 3 findings from simulation
  - Recommended actions
  - Expected business impact

---

## 📋 WEEKLY CHECKLIST TEMPLATE

**WEEK 1 STATUS (CHECKPOINT):**
- [ ] Module skeleton created & basic flow defined
- [ ] Parameters identified & initial values set
- [ ] Team meeting: Confirm interface definitions

**WEEK 2 STATUS (CHECKPOINT):**
- [ ] Core logic implemented & tested individually
- [ ] Sample output generated
- [ ] Peer review by 1 other team member

**WEEK 3 STATUS (CHECKPOINT):**
- [ ] Module complete with KPI & documentation
- [ ] Ready for integration testing
- [ ] All deliverables zipped & ready

**WEEK 4 STATUS (CHECKPOINT):**
- [ ] Integration testing completed
- [ ] Master model functional
- [ ] End-to-end simulation running

**WEEK 5 STATUS (CHECKPOINT):**
- [ ] Dashboard complete
- [ ] Scenario simulations done
- [ ] Analysis report drafted

**WEEK 6 STATUS (CHECKPOINT):**
- [ ] Presentation ready
- [ ] All documentation finalized
- [ ] Ready for submission/presentation

---

## 💾 FILE STRUCTURE (Recommended)

```
SupplyChain_Project/
├── 01_Concept/
│   ├── SUPPLY_CHAIN_CONCEPT.md ✓
│   ├── MODUL5_RETAIL_CUSTOMER_DETAIL.md ✓
│   └── PROJECT_CHECKLIST.md (this file)
│
├── 02_Individual_Modules/
│   ├── Modul1_Supplier/
│   │   ├── Supplier_Module_v1.0.alp
│   │   ├── Supplier_Parameters.xlsx
│   │   └── Supplier_Specification.docs
│   ├── Modul2_Manufacturing/
│   │   ├── Manufacturing_Module_v1.0.alp
│   │   ├── Manufacturing_Parameters.xlsx
│   │   └── Manufacturing_Specification.docs
│   ├── Modul3_Warehouse/
│   │   ├── Warehouse_Module_v1.0.alp
│   │   ├── Warehouse_Parameters.xlsx
│   │   └── Warehouse_Specification.docs
│   ├── Modul4_Distribution/
│   │   ├── Distribution_Module_v1.0.alp
│   │   ├── Distribution_Parameters.xlsx
│   │   └── Distribution_Specification.docs
│   └── Modul5_Retail/
│       ├── Retail_Module_v1.0.alp
│       ├── Retail_Parameters.xlsx
│       └── Retail_Specification.docs
│
├── 03_Integration/
│   ├── SupplyChain_Master_v1.0.alp
│   ├── Global_Parameters.xlsx
│   ├── Integration_Test_Results.xlsx
│   └── SupplyChain_Integration_Guide.docs
│
├── 04_Simulation_Results/
│   ├── SupplyChain_Simulation_Results.xlsx
│   ├── Dashboard.xlsx
│   ├── Scenario_Analysis_Report.docs
│   ├── SampleOutput.xlsx
│   └── Animation_Screenshots/
│
├── 05_Presentation/
│   ├── SupplyChain_Presentation.pptx
│   ├── Demo_Video.mp4
│   └── Executive_Summary.docs
│
└── 06_Documentation/
    ├── FINAL_PROJECT_REPORT.docs
    └── README.md (project overview)
```

---

## 🎯 SUCCESS CRITERIA

### Individual Module Level
- ✅ Module runs without errors for 7+ simulated days
- ✅ Output data format matches specification
- ✅ KPI dashboard displays correctly
- ✅ Unit tests pass

### Integration Level
- ✅ All 5 modules link correctly
- ✅ Data flows from Supplier → Retail → feedback
- ✅ End-to-end simulation completes 30 days without error
- ✅ Global KPI reflects all modules

### Deliverable Level
- ✅ All documentation complete & professional
- ✅ Presentation clear & engaging
- ✅ Scenario analysis provides realistic insights
- ✅ Recommendations are actionable

---

## 📞 COORDINATION TIPS

**Weekly Meeting Template (1 hour):**
- 10 min: Status updates (each member 2 min)
- 20 min: Problem solving for blockers
- 20 min: Architecture/interface review
- 10 min: Next week plan & assignments

**Communication Channels:**
- WhatsApp: Daily updates, quick questions
- Email: Formal documentation, file sharing
- Weekly Zoom: Structured meetings
- Shared Drive: Version control

**Red Flags to Watch:**
- ⚠️ Module not running by end of Week 2 → help needed
- ⚠️ Interface mismatch discovered in Week 4 → need design review
- ⚠️ Scope creep (too many features) → simplify focus on core
- ⚠️ Team member stuck → pair programming session

---

**Last Updated:** April 2026
**Status:** Ready for kick-off
**Next Step:** Schedule team meeting, assign roles, start development

