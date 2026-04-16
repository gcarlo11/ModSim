# MODUL 5: RETAIL & CUSTOMER - SPESIFIKASI DETAIL
## Supply Chain Simulation - AnyLogic Implementation

---

## 1. DESKRIPSI SINGKAT

Modul ini mensimulasikan **3 retail stores** yang menerima customer secara stochastic, melayani penjualan, dan mengelola inventory lokal. Modul akan generate demand signal ke distribution center ketika stock rendah.

---

## 2. FLOW DIAGRAM

```
┌─────────────────────────────────────────────────────┐
│          RETAIL MODULE - 3 STORES                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────┐                                  │
│  │ STORE 1      │                                  │
│  │ (Downtown)   │                                  │
│  │ Capacity:500 │                                  │
│  └──────────────┘                                  │
│                                                     │
│  ┌──────────────┐                                  │
│  │ STORE 2      │ ← All connected to Distribution  │
│  │ (Mall)       │     Center (Modul 4)             │
│  │ Capacity:400 │                                  │
│  └──────────────┘                                  │
│                                                     │
│  ┌──────────────┐                                  │
│  │ STORE 3      │                                  │
│  │ (Suburban)   │                                  │
│  │ Capacity:300 │                                  │
│  └──────────────┘                                  │
│                                                     │
└─────────────────────────────────────────────────────┘

                        ↓ (Inventory replenishment)
                        
            ┌────────────────────────┐
            │  DISTRIBUTION CENTER   │
            │  (Module 4)            │
            └────────────────────────┘
```

---

## 3. ENTITIES

### Entity 1: CUSTOMER
```
Customer {
  int id                          // unique ID
  double arrivalTime              // waktu customer masuk store
  String storeID                  // "Store_1", "Store_2", "Store_3"
  int quantityWanted              // berapa unit yang mau dibeli
  boolean isStockAvailable        // ada/tidak barang yang dicari
  double serviceTime              // waktu di kasir
  double departureTime            // waktu customer keluar
  boolean isSale_Completed        // true = purchase, false = leave empty-handed
}
```

### Entity 2: PRODUCT
```
Product {
  String productID                // "PROD_001"
  String productName              // "Widget A"
  double unitPrice                // harga per unit
  int quantityInStock             // stock count di shelf
  int minStockLevel               // threshold untuk reorder
  int reorderQuantity             // berapa order ke DC
  double lastRestockTime          // kapan terakhir restock
}
```

### Entity 3: SALE TRANSACTION
```
SaleTransaction {
  String transactionID            // "TXN_202604150001"
  String storeID                  
  double timestamp                
  int quantitySold                
  double revenue                  
  String status                   // "COMPLETED" atau "LOST_SALE"
}
```

---

## 4. PROCESSES

### 4.1 CUSTOMER ARRIVAL PROCESS

**Input:** Arrival rate per store
**Output:** New Customer entity

**Logic:**
```
For each Store in [Store_1, Store_2, Store_3]:
  
  IF time = 06:00 - 09:00:      // Morning rush
    Arrival rate = 20 customers/hour
    
  ELSE IF time = 09:00 - 16:00: // Regular hours
    Arrival rate = 8 customers/hour
    
  ELSE IF time = 16:00 - 21:00: // Evening rush
    Arrival rate = 15 customers/hour
    
  ELSE:                          // Night (closed)
    Arrival rate = 0
    
  // Customer arrives with random quantity want to buy
  Customer.quantityWanted = Normal(50, 15)  // mean=50, std=15
  Customer.quantityWanted = MAX(1, MIN(100, quantityWanted))
```

**Distribution:**
```
Morning peak:    06:00 - 09:00   (20% daily sales)
Mid-day:         09:00 - 16:00   (30% daily sales)
Evening peak:    16:00 - 21:00   (40% daily sales)
Late night:      21:00 - 06:00   (10% daily sales)
```

### 4.2 CUSTOMER SHOPPING PROCESS

**Input:** Customer arrives at store
**Output:** Transaction (COMPLETED or LOST_SALE)

**Logic:**
```
STEP 1: Customer checks shelf for product
  status = checkInventory(Store, product)
  
STEP 2: If available
  quantity_available = ProductStock[Store]
  
  IF quantity_available >= customer.quantityWanted:
    // Proceed to checkout
    quantity_to_buy = customer.quantityWanted
    isStockAvailable = TRUE
    
  ELSE IF quantity_available > 0:
    // Partial fulfillment - customer might buy partial qty
    IF random(0,1) < 0.7:  // 70% willing to buy partial
      quantity_to_buy = quantity_available
      isStockAvailable = TRUE
    ELSE:
      quantity_to_buy = 0
      isStockAvailable = FALSE  // Leave store
      
  ELSE:
    // Stockout - Lost sale
    quantity_to_buy = 0
    isStockAvailable = FALSE
    recordLostSale()

STEP 3: If sale proceeds
  serviceTime = Normal(5, 2) minutes  // checkout time
  revenue = quantity_to_buy * unitPrice
  
  updateInventory(Store, -quantity_to_buy)
  recordTransaction(sale_completed=TRUE, revenue, quantity)
  
  // Check if reorder needed
  IF ProductStock[Store] < minStockLevel:
    placeReplenishmentOrder()
```

### 4.3 INVENTORY REPLENISHMENT PROCESS

**Trigger:** Automatic when stock falls below reorder point
**Output:** Replenishment order to Distribution Center

**Logic:**
```
FOR EACH Store:
  FOR EACH Product in Store:
    
    IF currentStock <= minStockLevel AND NOT (pending_order_exists):
      
      // Calculate reorder quantity
      reorderQty = MAX(minStockLevel, avgDailyDemand * 7)
      
      // Create replenishment order
      order = {
        storeID: Store.id,
        productID: Product.id,
        quantity: reorderQty,
        orderTime: currentTime,
        expectedDelivery: currentTime + LEAD_TIME_DC_TO_RETAIL
      }
      
      sendOrderToDistributionCenter(order)
      record("Reorder placed", Store, Product, reorderQty)
      
      // Update pending flag
      pendingOrders[Store][Product] = TRUE
    
    ELSE IF orderDelivered:
      ProductStock[Store] += reorderQty
      pendingOrders[Store][Product] = FALSE
      record("Restock received", Store, Product, reorderQty)
```

**Replenishment Parameters:**
```
minStockLevel_Store1 = 150 units    // Downtown store
minStockLevel_Store2 = 120 units    // Mall store  
minStockLevel_Store3 = 80 units     // Suburban store

reorderQuantity = 7-days worth of avg demand
LEAD_TIME_DC_TO_RETAIL = {
  Store_1: 1 day (local)
  Store_2: 1 day (mall nearby DC)
  Store_3: 2 days (suburban)
}
```

---

## 5. CALCULATIONS & METRICS

### 5.1 DAILY KPI PER STORE

```python
# 1. Sales volume
DAILY_SALES_QTY = SUM(quantity_sold in all transactions today)
DAILY_REVENUE = SUM(revenue in all transactions today)

# 2. Stock availability
STOCK_OUT_INCIDENTS = COUNT(where quantityAvailable = 0)
STOCKOUT_RATE = STOCK_OUT_INCIDENTS / TOTAL_CUSTOMER_ATTEMPTS * 100%

# 3. Lost sales (demand not met)
LOST_SALES_QTY = SUM(quantity_wanted where stock unavailable)
LOST_REVENUE = LOST_SALES_QTY * unitPrice

# 4. Inventory efficiency
AVG_INVENTORY_LEVEL = AVG(stock_count throughout day)
INVENTORY_TURNOVER_DAY = DAILY_SALES_QTY / AVG_INVENTORY_LEVEL

# 5. Customer metrics
CUSTOMER_ARRIVAL_COUNT = Total customers entered store today
CONVERSION_RATE = SALES_COMPLETED / CUSTOMER_ARRIVAL_COUNT * 100%
CUSTOMER_SATISFACTION = 1 - (LOST_SALES_QTY / TOTAL_DEMAND) * 100%
```

### 5.2 WEEKLY/MONTHLY AGGREGATION

```
WEEKLY_SALES_VOLUME = SUM(daily sales for 7 days)
WEEKLY_REVENUE = SUM(daily revenue for 7 days)
WEEKLY_STOCKOUT_RATE = AVG(daily stockout rates)
HOLDING_COST = AVG_INVENTORY * COST_PER_UNIT_PER_DAY * 7 days
LOST_REVENUE = SUM(lost sales cost)

WEEKLY_PROFIT = WEEKLY_REVENUE - HOLDING_COST - [opportunity cost of lost sales]
ROI_BY_STORE = Store1_Profit vs Store2_Profit vs Store3_Profit
```

---

## 6. PARAMETERS (TO BE CONFIGURED)

### Store Configuration
```java
STORE_1_DEMAND_FACTOR = 1.0      // Downtown (baseline)
STORE_2_DEMAND_FACTOR = 0.8      // Mall (slightly lower)
STORE_3_DEMAND_FACTOR = 0.6      // Suburban (lowest)

STORE_OPERATING_HOURS = {
  weekday: 06:00 - 22:00
  weekend: 07:00 - 23:00
}

STORE_SHELF_CAPACITY = {
  Store_1: 500 units
  Store_2: 400 units
  Store_3: 300 units
}
```

### Customer Behavior
```
CUSTOMER_ARRIVAL_RATE = {
  Morning (6-9):   20 customers/hour
  Mid-day (9-16):  8 customers/hour
  Evening (16-21): 15 customers/hour
}

PURCHASE_QTY_DISTRIBUTION = Normal(50 units, StdDev=15)
PARTIAL_FULFILLMENT_ACCEPTANCE = 70%  // willing to buy partial qty

CUSTOMER_TIME_IN_STORE = Normal(15 minutes, StdDev=5)
CHECKOUT_TIME = Normal(5 minutes, StdDev=2)
```

### Product Parameters
```
UNIT_PRICE = $15
UNIT_HOLDING_COST = $0.50 per day
SHORTAGE_COST = $5 (lost sale penalty)

MIN_STOCK_LEVEL = {
  Store_1: 150 units
  Store_2: 120 units
  Store_3: 80 units
}

REORDER_QUANTITY = 7-day average demand
RESTOCK_LEAD_TIME = 1-2 days from DC
```

---

## 7. OUTPUT & DASHBOARD

### Real-time Display (Updated every hour)
```
┌─────────────────────────────────────────────────────┐
│         RETAIL STORES - REAL-TIME DASHBOARD         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  STORE 1 (Downtown)                                │
│    Current Stock: 245 / 500 units                  │
│    Today's Sales: 478 units | Revenue: $7,170     │
│    Stockout Incidents: 2                            │
│    Pending Reorder: 350 units (arrives tomorrow)   │
│                                                     │
│  STORE 2 (Mall)                                    │
│    Current Stock: 156 / 400 units                  │
│    Today's Sales: 312 units | Revenue: $4,680     │
│    Stockout Incidents: 0                            │
│    Status: ✓ Adequate stock                         │
│                                                     │
│  STORE 3 (Suburban)                                │
│    Current Stock: 45 / 300 units  ⚠️ LOW           │
│    Today's Sales: 187 units | Revenue: $2,805     │
│    Stockout Incidents: 3                            │
│    Pending Reorder: 280 units (arrives in 2 days) │
│                                                     │
├─────────────────────────────────────────────────────┤
│  TOTAL (All Stores)                                │
│    Daily Revenue: $14,655                          │
│    Total Sales: 977 units                          │
│    Avg Stockout Rate: 1.67%                        │
│    Total Lost Sales: $567                          │
│                                                     │
│  Cumulative (This Week)                            │
│    Revenue: $98,450                                │
│    Units Sold: 6,580                               │
│    Avg Inventory Value: $12,400                    │
│    Avg Stockout Rate: 2.1%                         │
└─────────────────────────────────────────────────────┘
```

### Daily Report (End of day summary)
```
Date: 2024-04-15
Period: 06:00 - 22:00 (16 hours operation)

SALES PERFORMANCE:
├─ Store_1: 478 units, $7,170 (Conversion: 82%)
├─ Store_2: 312 units, $4,680 (Conversion: 76%)
└─ Store_3: 187 units, $2,805 (Conversion: 68%)
Total Daily Revenue: $14,655

INVENTORY:
├─ Total Stock Held: 446 units
├─ Holding Cost: $223
├─ Reorder Quantity: 630 units (placed 3 orders)
└─ Estimated Restock Tomorrow: ~560 units

CUSTOMER ANALYTICS:
├─ Total Customers: 1,247
├─ Purchases Made: 977
├─ Lost Sales: 32 (2.6% conversion loss)
├─ Lost Revenue: $567
└─ Avg Satisfaction: 97.4%

ALERTS:
⚠️  Store_3 stock critically low (45 units)
⚠️  Reorder not arrived yet (expected 2024-04-16)
```

---

## 8. INTERFACE WITH OTHER MODULES

### Input from Distribution Center (Module 4)
```
Event: OnReplenishmentArrival()
Data: {
  storeID: "Store_1"
  productID: "PROD_001"
  quantity: 350
  deliveryTime: 1440  // minutes
  deliveryCost: $35
}
Action: Add quantity to store inventory
```

### Output to Distribution Center (Module 4)
```
Event: ReplenishmentOrderPlaced()
Data: {
  storeID: "Store_1"
  productID: "PROD_001"
  quantity: 350
  orderTime: [current simulation time]
  targetDelivery: [current time + LEAD_TIME]
  priority: "MEDIUM"  // or HIGH if stockout
}

Event: SalesDataReport() // Daily/Hourly
Data: {
  storeID: "Store_1"
  timestamp: [time]
  totalSalesQty: 478
  totalRevenue: $7,170
  stockoutIncidents: 2
  averageInventoryLevel: 245
}
```

---

## 9. TESTING & VALIDATION

### Unit Test Cases
```
Test 1: Customer Arrival
  Expected: 20 customers/hour during 06-09
  Verify: Arrival rate matches distribution
  
Test 2: Stock Depletion
  Given: Initial stock = 500, daily demand = 480
  Expected: Stock drops to ~20 after 1 day
  Verify: Stock levels decrease correctly
  
Test 3: Reorder Logic
  Given: minStock = 150, currentStock = 145
  Expected: Reorder triggered automatically
  Verify: Order sent to DC at correct time
  
Test 4: Stockout Handling
  Given: Stock = 0, customer wants 50
  Expected: Lost sale recorded, transaction not completed
  Verify: Lost revenue calculated correctly
  
Test 5: Partial Fulfillment
  Given: Stock = 30, customer wants 50, acceptance = 70%
  Expected: 70% probability customer buys 30 units
  Verify: Either sale 30 units or 0 units (not 50)
  
Test 6: Lead Time Variation
  Given: Store_1 lead time = 1 day, Store_3 = 2 days
  Given: Reorders placed simultaneously
  Expected: Store_1 arrives 1 day earlier than Store_3
  Verify: Delivery times match parameters
```

### Integration Test
```
- Send demand signal from Retail to Distribution
- Verify Distribution processes the order
- Verify Distribution ships at correct time
- Verify Retail receives goods and updates inventory
- Verify KPI reported correctly upstream
```

---

## 10. SUCCESS CRITERIA

✅ Module runs without errors for 30 simulated days
✅ Daily revenue within 10% of baseline expectations
✅ Stockout rate < 5% under normal demand
✅ Reorder logic responds correctly to low stock
✅ Data output format matches Distribution Center expectations
✅ Dashboard displays all KPI accurately
✅ Customer arrival rate matches hourly distribution
✅ Conversion rate within realistic range (68-82%)

---

## 11. DELIVERABLES

- [x] AnyLogic model file: `Retail_Module.alp`
- [x] Parameter configuration file: `RetailParameters_v1.0.xlsx`
- [x] Output data: Daily sales report, inventory log, transaction details
- [x] Integration guide document
- [x] Test report showing all unit/integration tests pass

---

## 12. NOTES FOR IMPLEMENTATION

1. **Use Array of Objects** untuk multiple stores
2. **Use Lookup Table** untuk demand distribution by time-of-day
3. **Use Decision Node** untuk stockout vs partial fulfillment logic
4. **Use Sink** untuk completed transactions (archive)
5. **Schedule restock arrivals** sebagai input event, bukan internal calculation
6. **Export daily KPI** ke Excel file untuk dashboard visualization
7. **Use animation** untuk visualize customer flow dan inventory changes

---

**Estimasi Development Time:** 1.5 - 2 weeks (termasuk testing & refinement)
**Complexity Level:** Medium (Intermediate AnyLogic user)

