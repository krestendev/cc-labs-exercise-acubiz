# Data Models

All data is stored as JSON files in `server/data/` and loaded into memory at startup by `server/mock_data.py`. Pydantic models in `server/main.py` validate every API response.

> **Important**: When changing a JSON file's structure, update the corresponding Pydantic model in `server/main.py` to match.

---

## InventoryItem

**Source file**: `server/data/inventory.json`

```typescript
{
  id: string               // "INV-001"
  sku: string              // "CB-001"
  name: string             // "Industrial Circuit Board A"
  category: string         // "Circuit Boards"
  warehouse: string        // "San Francisco" | "London" | "Tokyo"
  quantity_on_hand: number // 450
  reorder_point: number    // 100
  unit_cost: number        // 125.00
  location: string         // "Aisle 3, Shelf B"
  last_updated: string     // ISO 8601 datetime
}
```

An item is considered **low stock** when `quantity_on_hand <= reorder_point`.

---

## Order

**Source file**: `server/data/orders.json`

```typescript
{
  id: string               // "ORD-001"
  order_number: string     // "PO-2025-001"
  customer: string         // "Acme Manufacturing"
  items: OrderItem[]
  status: "Delivered" | "Shipped" | "Processing" | "Backordered"
  warehouse: string        // "San Francisco" | "London" | "Tokyo"
  category: string         // Derived from primary item
  order_date: string       // ISO 8601 datetime
  expected_delivery: string
  actual_delivery?: string // Present only when status is "Delivered"
  total_value: number
}

// Nested type
OrderItem {
  sku: string
  name: string
  quantity: number
  unit_price: number
}
```

---

## DemandForecast

**Source file**: `server/data/demand_forecasts.json`

```typescript
{
  id: string                                    // "DF-001"
  item_sku: string                              // "CB-001"
  item_name: string                             // "Industrial Circuit Board A"
  current_demand: number                        // units
  forecasted_demand: number                     // units
  trend: "increasing" | "stable" | "decreasing"
  period: string                                // "2025-Q1"
}
```

An item is **stable** when the absolute percentage change between `current_demand` and `forecasted_demand` is less than 2%.

---

## BacklogItem

**Source file**: `server/data/backlog_items.json`

```typescript
{
  id: string                          // "BL-001"
  order_id: string                    // "ORD-042"
  item_sku: string
  item_name: string
  quantity_needed: number
  quantity_available: number
  days_delayed: number
  priority: "high" | "medium" | "low"
  has_purchase_order?: boolean        // Computed at runtime from purchase_orders.json
}
```

`has_purchase_order` is not stored in JSON — it is derived by checking `purchase_orders.json` for a matching `backlog_item_id`.

---

## PurchaseOrder

**Source file**: `server/data/purchase_orders.json`

```typescript
{
  id: string                      // "PO-001"
  backlog_item_id: string         // Links to BacklogItem.id
  supplier_name: string
  quantity: number
  unit_cost: number
  expected_delivery_date: string  // ISO 8601 date
  status: string                  // "Pending" | "Confirmed" | "Shipped"
  created_date: string            // ISO 8601 datetime
  notes?: string
}
```

---

## DashboardSummary

Computed at request time by aggregating inventory and orders data.

```typescript
{
  total_inventory_value: number  // Sum of (quantity_on_hand * unit_cost) for all items
  low_stock_items: number        // Count where quantity_on_hand <= reorder_point
  pending_orders: number         // Count where status != "Delivered"
  total_backlog_items: number    // Total count of backlog items
  total_orders_value: number     // Sum of total_value across matching orders
}
```

---

## Spending

**Source files**: `server/data/spending.json`, `server/data/transactions.json`

### SpendingSummary
```typescript
{
  total_procurement_cost: number
  total_operational_cost: number
  total_labor_cost: number
  total_overhead: number
  procurement_change: number    // % change vs. prior period
  operational_change: number
  labor_change: number
  overhead_change: number
}
```

### MonthlySpending
```typescript
{
  month: string         // "2025-01"
  procurement: number
  operational: number
  labor: number
  overhead: number
}
```

### CategorySpending
```typescript
{
  category: string
  amount: number
  percentage: number    // Share of total spending
}
```

### Transaction

**Source file**: `server/data/transactions.json`

```typescript
{
  id: string
  date: string          // ISO 8601 date
  description: string
  category: string
  type: string          // "procurement" | "operational" | "labor" | "overhead"
  amount: number
  vendor?: string
}
```

---

## Reference Data

**Warehouses** (used across inventory, orders, dashboard):
- San Francisco
- London
- Tokyo

**Categories** (item types):
- Circuit Boards
- Sensors
- Motors
- Pressure Equipment
- Springs
- Power Supplies

**Order Statuses**: `Delivered`, `Shipped`, `Processing`, `Backordered`

**Time period format for `month` query param**:
- Specific month: `"2025-01"` through `"2025-12"`
- Quarter: `"Q1-2025"` through `"Q4-2025"`
