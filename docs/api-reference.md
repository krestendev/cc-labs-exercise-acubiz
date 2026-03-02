# API Reference

**Base URL**: `http://localhost:8001`
**Interactive docs**: `http://localhost:8001/docs` (Swagger UI)

All filter parameters are optional. Omit them (or pass `"all"`) to return unfiltered results.

---

## Root

### `GET /`
Returns API metadata.

**Response**
```json
{
  "message": "Factory Inventory Management API",
  "version": "1.0.0"
}
```

---

## Inventory

### `GET /api/inventory`
Returns all inventory items, optionally filtered.

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `warehouse` | string | Filter by warehouse name |
| `category` | string | Filter by item category |

**Response**: `Array<InventoryItem>` — see [Data Models](data-models.md#inventoryitem)

---

### `GET /api/inventory/{item_id}`
Returns a single inventory item by ID.

**Path Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `item_id` | string | Inventory item ID |

**Response**: `InventoryItem` | `404` if not found

---

## Orders

### `GET /api/orders`
Returns all orders, optionally filtered.

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `warehouse` | string | Filter by warehouse |
| `category` | string | Filter by item category |
| `status` | string | Filter by order status (`Delivered`, `Shipped`, `Processing`, `Backordered`) |
| `month` | string | Filter by month (`2025-01`) or quarter (`Q1-2025`) |

**Response**: `Array<Order>` — see [Data Models](data-models.md#order)

---

### `GET /api/orders/{order_id}`
Returns a single order by ID.

**Path Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `order_id` | string | Order ID |

**Response**: `Order` | `404` if not found

---

## Dashboard

### `GET /api/dashboard/summary`
Returns aggregate summary metrics, all filters supported.

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `warehouse` | string | Filter by warehouse |
| `category` | string | Filter by category |
| `status` | string | Filter by order status |
| `month` | string | Filter by month or quarter |

**Response**: `DashboardSummary`
```json
{
  "total_inventory_value": 1250000.00,
  "low_stock_items": 4,
  "pending_orders": 12,
  "total_backlog_items": 7,
  "total_orders_value": 485000.00
}
```

---

## Demand Forecasts

### `GET /api/demand`
Returns all demand forecasts. No filters supported.

**Response**: `Array<DemandForecast>` — see [Data Models](data-models.md#demandforecast)

---

## Backlog

### `GET /api/backlog`
Returns all backlog items. No filters supported.

**Response**: `Array<BacklogItem>` — see [Data Models](data-models.md#backlogitem)

Each item includes a computed `has_purchase_order` boolean.

---

## Spending

### `GET /api/spending/summary`
Returns overall spending totals and period-over-period changes.

**Response**
```json
{
  "total_procurement_cost": 520000.00,
  "total_operational_cost": 185000.00,
  "total_labor_cost": 210000.00,
  "total_overhead": 95000.00,
  "procurement_change": -3.2,
  "operational_change": 1.8,
  "labor_change": 0.5,
  "overhead_change": -1.1
}
```
`*_change` fields are percentage deltas vs. prior period.

---

### `GET /api/spending/monthly`
Returns spending broken down by month.

**Response**: `Array<MonthlySpending>`
```json
[
  {
    "month": "2025-01",
    "procurement": 45000.00,
    "operational": 15000.00,
    "labor": 18000.00,
    "overhead": 8000.00
  }
]
```

---

### `GET /api/spending/categories`
Returns spending aggregated by category.

**Response**: `Array<CategorySpending>`
```json
[
  {
    "category": "Circuit Boards",
    "amount": 120000.00,
    "percentage": 23.1
  }
]
```

---

### `GET /api/spending/transactions`
Returns recent spending transactions.

**Response**: `Array<Transaction>` — see [Data Models](data-models.md#transaction)

---

## Reports

### `GET /api/reports/quarterly`
Returns performance statistics grouped by quarter.

**Response**: `Array<QuarterlyReport>`
```json
[
  {
    "quarter": "Q1-2025",
    "total_orders": 145,
    "total_revenue": 2400000.00,
    "delivered_orders": 120,
    "fulfillment_rate": 82.8,
    "avg_order_value": 16551.72
  }
]
```

---

### `GET /api/reports/monthly-trends`
Returns order and revenue trends grouped by month.

**Response**: `Array<MonthlyTrend>`
```json
[
  {
    "month": "2025-01",
    "order_count": 48,
    "revenue": 800000.00,
    "delivered_count": 40
  }
]
```

---

## Purchase Orders

### `POST /api/purchase-orders`
Creates a purchase order for a backlog item.

**Request Body**
```json
{
  "backlog_item_id": "BL-001",
  "supplier_name": "Acme Electronics",
  "quantity": 500,
  "unit_cost": 12.50,
  "expected_delivery_date": "2025-03-15",
  "notes": "Expedited shipping requested"
}
```

**Response**: `PurchaseOrder` — see [Data Models](data-models.md#purchaseorder)

---

### `GET /api/purchase-orders/{backlog_item_id}`
Returns the purchase order linked to a specific backlog item.

**Path Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `backlog_item_id` | string | Backlog item ID |

**Response**: `PurchaseOrder` | `404` if not found
