# QUICK START GUIDE - SIMULASI SUPPLY CHAIN
## 5-Member Team, 5-Week Project

---

## 🎯 DALAM 30 DETIK

**Topik:** Simulasi Supply Chain end-to-end sebuah manufaktur dengan 5 modul independen

**5 Modul (1 per anggota):**
1. **Supplier** → Raw material sourcing & delivery
2. **Manufacturing** → Production with QC
3. **Warehouse** → Inventory management
4. **Distribution** → Transport & routing
5. **Retail/Customer** → Sales & demand

**Objective:** Liat bagaimana 5 tahapan ini interact, cari bottleneck, optimize efficiency

**Output:** Model AnyLogic terintegrasi + scenario analysis + recommendations

---

## 📁 FILE STRUCTURE

```
Buka workspace -> folder "ModSim" sudah ada 4 files:

✅ SUPPLY_CHAIN_CONCEPT.md
   → Baca ini PERTAMA! Overview semua modul, data flow, timeline

✅ MODUL5_RETAIL_CUSTOMER_DETAIL.md
   → Contoh detail untuk 1 modul (Retail)
   → Shows bagaimana strukturnya

✅ PROJECT_CHECKLIST.md
   → Week-by-week tasks
   → Siapa yang ngerjain apa

✅ PARAMETERS_REFERENCE.md
   → Nilai-nilai ready-to-use untuk AnyLogic
   → Copy-paste ke spreadsheet parameters Anda
```

---

## 🚀 TAHAP 1: KICK-OFF (Hari 1)

**Meeting 1 jam:**

1. **Baca bersama** SUPPLY_CHAIN_CONCEPT.md (20 min)
   - Discuss: Setuju dengan konsep ini?
   - Setuju dengan 5 modul ini?

2. **Assign member** ke setiap modul (5 min):
   ```
   Anggota 1 → Supplier
   Anggota 2 → Manufacturing
   Anggota 3 → Warehouse
   Anggota 4 → Distribution
   Anggota 5 → Retail/Customer
   ```

3. **Setup logistik** (15 min):
   - Pemimpin tim: Designate 1 org (suggest: yang paling detail-oriented)
   - Tools: Agree on WhatsApp, shared drive (Google Drive / Dropbox)
   - Schedule: Weekly meeting same time
   - File versioning: Naming convention (e.g., `Module1_v1.0.alp`)

4. **Quick walkthrough** of Modul5 example (10 min)
   - Lihat MODUL5_RETAIL_CUSTOMER_DETAIL.md section 2-4
   - Ini adalah contoh detail seperti apa yang diharapkan

5. **Start planning** week 1 (10 min):
   - Setiap anggota: Identify TOP 3 flows di modul Anda
   - Preview soal parameter yang akan digunakan

---

## 💻 TAHAP 2: INDIVIDUAL DEVELOPMENT (MINGGU 1-3)

**Timeline: 3 minggu per member**

### Workflow tiap member:

**MINGGU 1: Desain & Setup (40% effort)**
```
1. Baca spec modul Anda (SUPPLY_CHAIN_CONCEPT + PARAMETERS_REFERENCE)
   → 30 min

2. Buat AnyLogic project skeleton
   → File: [ModulName]_Module.alp
   → Identify: entity types, resources, processes
   → 1-2 hours

3. Define entity structures (in code)
   → Example in MODUL5 section 3
   → 30 min

4. Setup basic parameters
   → Copy values dari PARAMETERS_REFERENCE
   → Input ke spreadsheet atau constants dalam model
   → 30 min

5. Create basic flow (skeleton)
   → Draw main process boxes
   → Connect entities, resources
   → 1-2 hours

6. Weekly checkpoint (Week 1 end)
   → Module runs without errors (even if minimal)
   → Output format agreed
   → Next week: detailed logic
```

**MINGGU 2: Logic & Calculations (40% effort)**
```
1. Implement core business logic
   → Decision nodes, calculations, branching
   → Example: Stockout handling in MODUL5 section 4.2
   → 2-3 hours

2. Add KPI tracking
   → Charts, statistics, data collectors
   → Example: MODUL5 section 5
   → 1 hour

3. Parameter sensitivity
   → Test: What if we change queue time by 10%?
   → Verify output makes sense
   → 1 hour

4. Sample output generation
   → Run simulation 1 week
   → Export data to Excel
   → 30 min

5. Weekly checkpoint (Week 2 end)
   → Core logic working correctly
   → Sample output defined
   → Ready for documentation next week
```

**MINGGU 3: Testing & Documentation (20% effort)**
```
1. Unit testing
   → Test specific scenarios in your module
   → Example MODUL5 section 9
   → 1 hour

2. Write specification document
   → Copy template dari MODUL5 section 12
   → Fill in YOUR module details
   → Include assumptions, parameters, output format
   → 1-2 hours

3. Prepare for integration
   → Document output format
   → Identify what you need from OTHER modules
   → Create sample "dummy" input from other module
   → 30 min

4. Deliverable packaging
   → File: [Module]_v1.0.alp
   → File: [Module]_Parameters.xlsx
   → File: [Module]_Specification.docx
   → ZIP everything + ready to hand off
   → 30 min

5. Weekly checkpoint (Week 3 end)
   → Module complete
   → Documentation done
   → Ready for integration phase
```

### Daily Standup Template (5 min WhatsApp):
```
9am: "What are you working on today?"
3pm: "Any blockers? Need help?"
6pm: "Done for today. Tomorrow's plan?"

→ Keep momentum, catch issues early
```

---

## 🔗 TAHAP 3: INTEGRATION (MINGGU 4)

**Timeline: 1 minggu, ALL MEMBERS WORK TOGETHER**

**Early Week (Mon-Wed):**
```
1. Agreement meeting (2 hours):
   - Finalize data formats between modules
   - Each pair (Supplier↔Mfg, Mfg↔Warehouse, etc.) sync
   - Create master parameter file: Global_Parameters.xlsx
   - Decision: Embedded simulation vs linked models?

2. Integration testing pairs:
   - Supplier + Manufacturing: Test data flow
   - Manufacturing + Warehouse: Test shipment format
   - Warehouse + Distribution: Test replenishment orders
   - Distribution + Retail: Test delivery arrivals
   → Each couple does 2-hour integration test session

3. Create Master model:
   - Integration lead assembles all modules
   - File: SupplyChain_Master_v1.0.alp
   - Test: Can it run 1 full week without error?
```

**Mid-Late Week (Thu-Fri):**
```
1. End-to-end simulation (1-2 days):
   - Run full supply chain: Supplier → Retail → feedback
   - Watch data flow through all stages
   - Record: Any bottlenecks? Data format mismatches?

2. Dashboard creation:
   - Create Excel dashboard
   - Pull all KPI from master model output
   - Charts: Revenue trends, inventory levels, stockout rate
   - Make it VISUAL and easy to read

3. Troubleshoot & fix:
   - If something breaks → ALL HANDS meeting
   - Pair programming if needed
```

**Deliverable (End Week 4):**
```
✅ SupplyChain_Master_v1.0.alp (integrated model file)
✅ SupplyChain_Dashboard.xlsx (KPI visualization)
✅ Integration_Test_Report.docx (what was tested, results)
```

---

## 📊 TAHAP 4: ANALYSIS & REPORTING (MINGGU 5-6)

**Scenario Simulation (Week 5):**

**Scenario 1: Baseline** (Normal operations)
```
Run 30 simulated days
Collect KPI: Revenue, stockouts, inventory levels, costs
Baseline for comparison
```

**Scenario 2: Demand Spike** (Customer demand +50%)
```
What happens if customer demand doubles?
Does system break?
Where's the bottleneck?
```

**Scenario 3: Supply Disruption** (Supplier delayed)
```
Lead time supplier 10 days instead of 3
How much inventory safety stock needed?
Cost of buffer vs lost sales?
```

**Analysis Questions:**
```
1. Which stage is bottleneck? Why?
2. What's the cost of stockouts?
3. How much inventory is too much / too little?
4. If we added 1 more truck, savings?
5. If we improved quality, impact on costs?
```

**Deliverables (Week 5-6):**
```
✅ Scenario_Analysis_Report.docx (findings + recommendations)
✅ PowerPoint Presentation (30-40 slides)
   - Modules overview
   - Integration architecture
   - Results & dashboards
   - Scenario analysis
   - Recommendations
   
✅ Executive Summary (1 page - top 3 findings)
✅ Demo video (5 min animation)
```

---

## ✅ CHECKLISTS

### Pre-Development:
- [ ] Team meeting done, roles assigned
- [ ] All members have read SUPPLY_CHAIN_CONCEPT.md
- [ ] AnyLogic installed & tested
- [ ] Shared drive setup (Google Drive / Dropbox)
- [ ] WhatsApp group created

### End of Week 1 (Each Member):
- [ ] Module skeleton created
- [ ] Entity definitions done
- [ ] Parameters identified (even if not final)
- [ ] Basic flow diagram in AnyLogic

### End of Week 2 (Each Member):
- [ ] Core logic implemented & tested
- [ ] Sample output generated
- [ ] Dashboard/KPI working
- [ ] Peer review completed

### End of Week 3 (Each Member):
- [ ] Module complete & fully functional
- [ ] Documentation written
- [ ] Testing completed
- [ ] Deliverables zipped & ready

### Mid Week 4 (Team):
- [ ] Data format standards agreed
- [ ] Pair integration tests completed
- [ ] Master model running
- [ ] Dashboard live

### End Week 4 (Team):
- [ ] Full 30-day simulation complete
- [ ] No errors or data losses
- [ ] All KPI reporting correctly
- [ ] Documentation updated

### Week 5 (Team):
- [ ] All 3 scenarios simulated
- [ ] Analysis & findings documented
- [ ] Presentation drafted
- [ ] Demo video recorded

### Week 6 (Team):
- [ ] Final presentation ready
- [ ] All documentation polished
- [ ] Submit/present to instructor

---

## 🎓 DARI SINI KE SANA

**📌 Sekarang (Kick-off Meeting):**
1. Read SUPPLY_CHAIN_CONCEPT.md (everyone)
2. Assign modul per anggota
3. Agree on timeline & communication

**📌 Next (Start Development - Week 1):**
1. Each member buka PARAMETERS_REFERENCE.md
2. Baca spec modul Anda (ada di SUPPLY_CHAIN_CONCEPT section Modul 1-5)
3. Create AnyLogic project skeleton
4. Weekly checkpoint end of week

**📌 Pertanyaan? Reference:**
- Modul overview → SUPPLY_CHAIN_CONCEPT.md
- Modul detail example → MODUL5_RETAIL_CUSTOMER_DETAIL.md (use as template)
- Parameter values → PARAMETERS_REFERENCE.md
- Timeline/tasks → PROJECT_CHECKLIST.md

---

## 💡 TIPS SUKSES

1. **Start early, communicate often**
   - Jangan tunggu minggu akhir untuk mulai
   - Daily standup (even just WhatsApp msg)

2. **Focus on correctness, not perfection**
   - "Good enough" logic is okay, refine later
   - Get the flow right first, optimize after integration

3. **Document as you go**
   - Jangan tunggu akhir untuk nulis dokumentasi
   - Write template week 1, fill details week 3

4. **Pair programming for blockers**
   - If someone stuck > 2 hours → get help
   - 30 min pair session often solves it

5. **Test incrementally**
   - Don't wait until end to test
   - Test your module daily

6. **Communicate assumptions**
   - If you assume X → tell the team
   - Might affect other modules

---

## 📞 PROBLEM? ASK

- **"Saya bahingung modul saya apa"** → Baca SUPPLY_CHAIN_CONCEPT section modul mu
- **"Ngga tahu mau input apa"** → Baca MODUL5 section 4 (processes) dan section 8 (input/output)
- **"Parameter apa yang realistic?"** → Lihat PARAMETERS_REFERENCE (copy + adjust)
- **"Gimana caranya merge model?"** → Tanya di week 4, ada meeting khusus
- **"Saya stuck pada..."** → Report di daily standup, bisa pair programming

---

## 🏁 SUCCESS = 

✅ All 5 modules independent & working
✅ All modules integrated seamlessly
✅ Master model runs 30 days without error
✅ KPI dashboard accurate & insightful
✅ Scenario analysis provides actionable insights
✅ Professional documentation & presentation

**You've got this! 🚀**

---

**Last Update:** April 2026
**Document Status:** READY TO START
**Next Action:** Schedule kick-off meeting!

