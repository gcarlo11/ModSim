# PARAMETER & DATA REFERENCE
## Quick Start Values untuk Simulasi Supply Chain

---

## 📊 GLOBAL PARAMETERS (SHARED BY ALL MODULES)

### Time Unit: MINUTE (semua timing dalam menit)
```
Simulation Unit = 1 minute
Display Unit = hours/days
Warm-up period = 3 days (4320 minutes)
Steady-state observation = 30 days (43,200 minutes)
```

### Currency: USD ($)
```
Base unit price = $15/unit
All costs calculated in USD
```

---

## 🚚 SUPPLIER MODULE PARAMETERS

### Warehouse Initial Stock
```
Initial Inventory = 5,000 units
Max Capacity = 10,000 units
Safety Stock = 1,000 units
```

### Order Processing
```
Order Processing Time = 30 minutes ± 10 min (Normal distribution)
QC Check Time = 15 minutes ± 5 min (for batch of 100 units)
QC Accept Rate = 98% (Reject 2% of batches)
```

### Shipping
```
Lead Time (to Manufacturing) = {
  Min: 2 days = 2,880 minutes
  Max: 5 days = 7,200 minutes
  Mode: 3 days = 4,320 minutes (most likely)
  Distribution: Triangular
}

Shipping Cost = $0.10 per unit
```

### Purchase Order Characteristics
```
Avg Order Size = 1,000 units ± 200 (Normal)
Min Order = 500 units
Max Order = 2,000 units
Order Frequency = Triggered by Manufacturing when stock < threshold
```

### KPI Targets
```
On-time Delivery = 95%+
Order Fulfillment Time = < 3 days average
Stock-out Incidents = 0 (should not happen)
Supplier Utilization = 70-80%
```

---

## 🏭 MANUFACTURING MODULE PARAMETERS

### Production Line Setup
```
Number of Workstations = 4 (Assembly line)
  Station 1: Assembly Part A (resources: 1 operator)
  Station 2: Assembly Part B (resources: 1 operator)
  Station 3: Assembly Part C (resources: 1 operator)
  Station 4: Final Assembly (resources: 2 operators - bottleneck!)

Production Capacity = 1,000 units/day (max)
                   ≈ 42 units/hour (42 minutes per unit avg)
```

### Processing Time per Station
```
Station 1: 10 minutes ± 2 min per unit (Normal)
Station 2: 12 minutes ± 2 min per unit
Station 3: 8 minutes ± 1.5 min per unit
Station 4: 15 minutes ± 3 min per unit (operator bottleneck)

Total cycle time (best case) = 45 minutes/unit
Total cycle time (average) = 48 minutes/unit
Total cycle time (worst case) = 54 minutes/unit
```

### Quality Control
```
Defect Rate at Production = 3-5% (Normal product defects)
QC Inspection Time = 5 minutes per unit
QC Detection Rate = 95% (5% defects slip through)
Rework Time = 20 minutes per defective unit
Rework Success Rate = 90% (10% become scrap)
```

### Finished Goods Buffer
```
Buffer Capacity = 500 units (temporary storage before Warehouse)
Buffer Hold Time = 4 hours (max, then must ship)
```

### Overtime/Shift
```
Regular Hours = 06:00-22:00 (16 hours/day)
Weekend Production = 50% capacity
Maintenance Days = 1 day per week (random)
```

### KPI Targets
```
Production Rate = 900-1000 units/day
Average Cycle Time = 48 minutes ± 5 min
Defect Rate = < 5%
Machine Utilization = 80-90%
First-Pass Yield = > 95%
```

---

## 📦 WAREHOUSE MODULE PARAMETERS

### Storage System
```
Total Capacity = 10,000 units
Allocated by Region:
  - Quick Pick Area = 2,000 units (fast moving items)
  - Standard Area = 6,000 units (normal items)
  - Archive Area = 2,000 units (slow moving items)

Shelf Life = 365 days (no spoilage in this simulation)
```

### Inventory Management
```
Reorder Point (Safety Stock Estimation):
  Min Stock Level = 1,500 units
  Max Stock Level = 8,000 units
  
Reorder Quantity = Based on 7-day demand forecast
  Formula: Reorder Qty = MAX(minStock, avgDaily_Demand × 7)
  Typical Reorder = 3,000-4,000 units

Reorder Trigger Logic:
  IF current_stock ≤ min_stock AND NO pending order:
    THEN place_replenishment_order(to Manufacturing)
```

### Receiving Process
```
Receiving Inspection Time = 15 minutes per shipment
Unloading Time = 20 minutes per 500 units
Put-away Time = 5 minutes per pallet (100 units)

Total receiving: ~30-40 minutes per shipment
```

### Picking & Shipping
```
Picking Time = 2 minutes per order (batch pick)
Packing Time = 3 minutes per order
Shipping Prep = 5 minutes per shipment

Total fulfillment: ~10 minutes per distribution order
```

### Inventory Holding Cost
```
Daily Holding Cost = $0.50 per unit per day
  (includes: space, handling, insurance)
  
Monthly Cost = 10,000 units × $0.50 × 30 days = $150,000
```

### KPI Targets
```
Average Inventory Level = 4,000-5,000 units
Inventory Turnover = 3-4 times per year
Warehouse Utilization = 60-70%
Pick Accuracy = > 99%
Days of Supply = 4-5 days
Stock-out Incidents = 0-1 per month
```

---

## 🚛 DISTRIBUTION MODULE PARAMETERS

### Distribution Center
```
Receiving Capacity = 3,000 units/day
DC Storage Capacity = 5,000 units (short-term buffer)
Max DC Holding Time = 48 hours
```

### Retail Locations (3 stores)
```
Location 1: Downtown Store
  Distance from DC = 10 km
  Lead Time = 1 day (next morning delivery)
  Demand Factor = 1.0 (high traffic area)
  
Location 2: Mall
  Distance from DC = 15 km
  Lead Time = 1 day (next morning delivery)
  Demand Factor = 0.8 (medium traffic)
  
Location 3: Suburban
  Distance from DC = 40 km
  Lead Time = 2 days (scheduled pickup)
  Demand Factor = 0.6 (lower traffic)
```

### Vehicle & Routing
```
Number of Vehicles = 2 trucks
Truck Capacity = 2,000 units per truck
Vehicle Operating Hours = 06:00-22:00 (16 hours)

Route Optimization:
  Route 1: DC → Store_1 → Store_2 → Return
    Distance: 25 km total
    Time: 2 hours + 1 hour stops = 3 hours
    Frequency: Daily (morning)
    
  Route 2: DC → Store_3 → Return
    Distance: 80 km total
    Time: 3 hours + 1 hour stop = 4 hours
    Frequency: Alternate days (morning)
```

### Shipping Costs
```
Transportation Cost = $0.15 per unit
Fuel Cost = $5 per km driven
Vehicle Maintenance = $50 per trip

Total Daily Transport Cost ≈ $300-500
```

### Lead Time Variability
```
Store_1 (Downtown): 1 day (24±2 hours) - Uniform
Store_2 (Mall): 1 day (24±2 hours) - Uniform
Store_3 (Suburban): 2 days (48±4 hours) - Uniform
```

### KPI Targets
```
On-time Delivery Rate = 95%+
Average Delivery Time = 1.5 days (all stores avg)
Vehicle Utilization = 75%+
Delivery Cost per Unit = < $0.25
DC Inventory Turns = 10+ per month
```

---

## 🛍️ RETAIL MODULE PARAMETERS

### Store Operating Hours
```
Weekday (Mon-Fri):
  06:00 - 09:00 = Morning Peak (20% daily customers)
  09:00 - 16:00 = Daytime (30% daily customers)
  16:00 - 21:00 = Evening Peak (40% daily customers)
  21:00 - 06:00 = Closed
  
Weekends (Sat-Sun):
  07:00 - 23:00 = Extended hours
  23:00 - 07:00 = Closed
```

### Customer Arrival Pattern
```
STORE 1 (Downtown) - HIGH TRAFFIC
  Morning (6-9):   24 customers/hour = 180/day
  Daytime (9-16):  10 customers/hour = 70/day
  Evening (16-21): 18 customers/hour = 90/day
  Daily Total ≈ 340 customers/day

STORE 2 (Mall) - MEDIUM TRAFFIC
  Morning:   18 customers/hour = 135/day
  Daytime:    8 customers/hour = 56/day
  Evening:   14 customers/hour = 70/day
  Daily Total ≈ 260 customers/day

STORE 3 (Suburban) - LOW TRAFFIC
  Morning:   12 customers/hour = 90/day
  Daytime:    6 customers/hour = 42/day
  Evening:   10 customers/hour = 50/day
  Daily Total ≈ 180 customers/day
```

### Customer Purchase Behavior
```
Purchase Quantity per Customer:
  Mean = 50 units
  StdDev = 15 units
  Min = 5 units
  Max = 100 units
  Distribution: Normal (truncated)

Purchase Probability (given customer arrives):
  If stock available = 85% probability to buy
  If partial stock = 70% willing to buy reduced qty
  If no stock = 0% (walk out empty-handed)

Time in Store:
  Mean = 15 minutes
  StdDev = 5 minutes
  Distribution: Normal
  
Checkout Time:
  Mean = 5 minutes
  StdDev = 2 minutes
  Distribution: Normal
```

### Inventory Per Store
```
STORE 1 (Downtown):
  Shelf Capacity = 500 units
  Min Stock Level = 150 units
  Target Reorder = 350 units when hits min
  
STORE 2 (Mall):
  Shelf Capacity = 400 units
  Min Stock Level = 120 units
  Target Reorder = 280 units
  
STORE 3 (Suburban):
  Shelf Capacity = 300 units
  Min Stock Level = 80 units
  Target Reorder = 220 units
```

### Restock Arrival
```
Incoming shipment from Distribution Center:
  Trigger: When shelf stock ≤ min stock level
  Quantity: Reorder amount (calculated above)
  Lead Time: 1 day (Store_1, Store_2) or 2 days (Store_3)
  Frequency: As needed (multiple times per week typically)
  
Restock Process:
  Stock is added immediately to shelf
  Previous inventory tracked for turnover calc
```

### Sales Transactions
```
Unit Price = $15
Daily Revenue (each store):
  Store_1: $4,500 - $5,500
  Store_2: $3,500 - $4,000
  Store_3: $2,000 - $2,500
  
Expected Daily Total Revenue = $10,000 - $12,000
Expected Monthly Revenue = $300,000 - $360,000
```

### KPI Targets
```
Daily Sales Volume:
  Store_1: 300-370 units
  Store_2: 230-270 units
  Store_3: 130-170 units
  Total: 660-810 units/day

Conversion Rate = 75-85% (purchase / customer arrival)
Average Stockout Rate = 2-5% (customer wants item but out of stock)
Lost Sales per Month = 1-3% of potential demand

Customer Satisfaction (proxy):
  Target: 95%+ (reflects stock availability)
  Formula: 1 - (lost_sales_qty / total_demand_qty)
  
Revenue per Store per Day = Store_1: $4,500-5,500
                           Store_2: $3,500-4,000
                           Store_3: $2,500-3,000
```

---

## 📈 GLOBAL SUPPLY CHAIN KPI

### Baseline Expectations
```
Total Lead Time (Supplier Order → Customer Purchase):
  Average = 5-7 days
  Target = < 7 days
  
Fill Rate (% demand fulfilled on time):
  Target = 95%+
  
Total System Cost per Unit:
  Production = $10
  Holding (in transit & storage) = $1-2
  Transportation = $0.25-0.35
  Total = $11.25 - $12.35 per unit sold
  
Inventory Turns (Company-wide):
  Annual = 3-4 times/year
  Monthly = 0.25-0.33 turns/month
  
Service Level (retail no stockout):
  Target = 98%+
  
Daily System Throughput:
  Expected = 800-900 units/day
  Revenue = $12,000 - $13,500/day
```

### Cost Breakdown
```
Production Cost (Manufacturing):
  Variable cost = $10/unit
  Labor = $8/unit
  Materials = $2/unit
  Overhead = Fixed $5,000/day
  Monthly = $10 × 24,000 units + overhead = $245,000

Inventory Holding Cost (Warehouse + DC + Retail):
  Warehouse avg = 4,500 units × $0.50/day = $2,250/day
  DC avg = 1,500 units × $0.50/day = $750/day
  Retail avg = 1,200 units × $0.50/day = $600/day
  Total holding = $3,600/day = $108,000/month

Transportation Cost:
  Supplier → Mfg = $100/shipment × 24/month = $2,400/month
  Mfg → Warehouse = $200/shipment × 24/month = $4,800/month
  Warehouse → DC = $300/shipment × 24/month = $7,200/month
  DC → Retail = $500/day × 30 days = $15,000/month
  Total transport = $29,400/month

Total System Cost = $245,000 + $108,000 + $29,400 = $382,400/month
Revenue (900 units/day × $15 × 30 days) = $405,000/month
Gross Profit = $405,000 - $382,400 = $22,600/month
Profit Margin = 5.6%
```

---

## 🎲 RANDOM DISTRIBUTIONS (FOR ANYLOGIC)

```
Normal Distribution Examples:
  new Normal(mean, stddev).sample()
  Example: new Normal(50, 15) = Normal with mean 50, stdDev 15

Triangular Distribution:
  new Triangular(min, mode, max).sample()
  Example: new Triangular(2880, 4320, 7200) = 2-5 days peaked at 3

Uniform Distribution:
  new Uniform(min, max).sample()
  Example: new Uniform(20, 40) = random number 20-40

Exponential Distribution (for inter-arrival times):
  new Exponential(mean).sample()
  Example: new Exponential(5) = mean time 5 minutes between arrivals
```

---

## 📥 SAMPLE DATA VALIDATION

Test your simulation output with these checks:

```
✓ Supplier Module:
  - Daily orders received: 2-4 orders/day
  - Order processing time: avg 30 min
  - Lead time distribution: peaks at 3 days
  
✓ Manufacturing Module:
  - Production: 850-1000 units/day
  - Cycle time: 45-55 minutes
  - Defect rate: 3-5%
  
✓ Warehouse Module:
  - Inventory level: 3,000-5,000 units range
  - Reorder frequency: 2-3 times/week
  - Stock-outs: 0-1 incidents/month
  
✓ Distribution Module:
  - DC shipments: 2-3/day
  - Route 1 delivery time: 20-26 hours
  - Route 3 delivery time: 44-52 hours
  
✓ Retail Module:
  - Total customers: 780-850/day
  - Sales volume: 650-850 units/day
  - Stockout incidents: 10-30/month
  - Revenue: $9,750-12,750/day
  
✓ Overall:
  - No accumulation of indefinite entities
  - KPI converges after 3-day warm-up
  - Statistics stable after 7 days simulation
```

---

## 💾 PARAMETER FILE TEMPLATE

Save as: `Global_Parameters.xlsx`

```
Sheet 1: TIME_PARAMETERS
Parameter | Value | Unit | Notes
──────────────────────────────────
LEAD_TIME_SUPPLIER | 4320 | minutes | 3 days mean
LEAD_TIME_MFG_WH | 1440 | minutes | 1 day
LEAD_TIME_WH_DC | 2880 | minutes | 2 days
LEAD_TIME_DC_RETAIL1 | 1440 | minutes | 1 day
LEAD_TIME_DC_RETAIL2 | 1440 | minutes | 1 day
LEAD_TIME_DC_RETAIL3 | 2880 | minutes | 2 days

Sheet 2: COST_PARAMETERS
UNIT_PROD_COST | 10
UNIT_HOLDING_COST | 0.5
UNIT_SHORTAGE_COST | 5
TRANSPORT_COST | 0.15

Sheet 3: DEMAND_PARAMETERS
DAILY_DEMAND_MEAN | 800
DAILY_DEMAND_STD | 120
CUSTOMER_ARRIVAL_PEAK | 20
CUSTOMER_ARRIVAL_NORMAL | 8

Sheet 4: CAPACITY_PARAMETERS
WH_CAPACITY | 10000
DC_CAPACITY | 5000
MFG_CAPACITY | 1000
STORE1_CAPACITY | 500
STORE2_CAPACITY | 400
STORE3_CAPACITY | 300
```

---

**Ready to use!** Copy these values into your AnyLogic models. ✅

