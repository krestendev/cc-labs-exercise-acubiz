# Frontend

The frontend is a Vue 3 single-page application built with Vite, served on port 3000.

**Entry point**: `client/src/main.js`
**Root component**: `client/src/App.vue` (navigation, routing, global layout)
**API client**: `client/src/api.js` (centralized axios wrapper)

---

## Views

Located in `client/src/views/`. Each view is a full-page component mapped to a route.

| View | Route | Purpose |
|------|-------|---------|
| `Dashboard.vue` | `/` | Summary cards (inventory value, low stock, pending orders, backlog) and charts |
| `Inventory.vue` | `/inventory` | Stock list with SKU, location, reorder alerts; opens `InventoryDetailModal` |
| `Orders.vue` | `/orders` | Order list with status, customer, delivery tracking; opens order detail |
| `Spending.vue` | `/spending` | Cost breakdown by type and month, trend charts; opens `CostDetailModal` |
| `Demand.vue` | `/demand` | Demand forecasts with trend indicators (increasing/stable/decreasing) |
| `Reports.vue` | `/reports` | Quarterly and monthly trend aggregations |
| `Backlog.vue` | `/backlog` | Delayed items with priority levels, purchase order management; opens `BacklogDetailModal` |

**Pattern used in every view:**
```js
// Raw API data in refs
const allOrders = ref([])
const inventoryItems = ref([])

// Derived/display data in computed
const filteredOrders = computed(() => allOrders.value.filter(...))

// Reload when filters change
watch(filters, fetchData, { deep: true })
```

---

## Components

Located in `client/src/components/`.

| Component | Purpose |
|-----------|---------|
| `FilterBar.vue` | Global filter controls (Time Period, Warehouse, Category, Order Status). Used in the nav bar; updates shared `useFilters` state |
| `ProfileMenu.vue` | User avatar/name dropdown, triggers profile modal and task modal |
| `ProfileDetailsModal.vue` | Displays and edits user profile details |
| `TasksModal.vue` | Simple task manager — create, toggle complete, delete tasks |
| `LanguageSwitcher.vue` | Toggles between English (`en`) and Japanese (`ja`) |
| `InventoryDetailModal.vue` | Full detail view for a single inventory item |
| `ProductDetailModal.vue` | Product detail overlay (used from Demand/Inventory views) |
| `BacklogDetailModal.vue` | Detail view for a backlog item; shows linked purchase order |
| `CostDetailModal.vue` | Breakdown modal for a spending category |

---

## Composables

Located in `client/src/composables/`.

### `useFilters.js`
Manages the four global filter values as a module-level singleton (no prop drilling needed).

```js
import { useFilters } from '@/composables/useFilters'

const { selectedPeriod, selectedLocation, selectedCategory, selectedStatus } = useFilters()
```

| Export | Type | Default | Maps to query param |
|--------|------|---------|---------------------|
| `selectedPeriod` | `ref<string>` | `"all"` | `month` |
| `selectedLocation` | `ref<string>` | `"all"` | `warehouse` |
| `selectedCategory` | `ref<string>` | `"all"` | `category` |
| `selectedStatus` | `ref<string>` | `"all"` | `status` |

Values set to `"all"` are omitted from API requests.

---

### `useAuth.js`
Provides current user identity for the profile menu and header display.

```js
const { currentUser, login, logout } = useAuth()
```

---

### `useI18n.js`
Switches the active locale and provides a translation function.

```js
const { t, locale, setLocale } = useI18n()

// In template
{{ t('nav.dashboard') }}
```

Supported locales: `en` (English), `ja` (Japanese).
Translation files: `client/src/locales/en.js`, `client/src/locales/ja.js`.

---

## API Client (`api.js`)

All HTTP calls go through `client/src/api.js`. It wraps axios and:
- Sets the base URL (`http://localhost:8001`)
- Strips `"all"` filter values before building query params
- Exports one named function per resource

```js
import { getInventory, getOrders, getDashboardSummary } from '@/api'

// Each function accepts a filters object
const items = await getInventory({ warehouse: 'London', category: 'Sensors' })
```

---

## Routing

Routes are defined in `client/src/App.vue` using Vue Router. The nav sidebar links to each view. No authentication guard is applied (demo app).

---

## Styling

- Scoped CSS in each `.vue` file
- Design tokens (defined in `App.vue` / global styles):
  - Background: `#0f172a` (slate-900)
  - Muted text: `#64748b` (slate-500)
  - Borders/dividers: `#e2e8f0` (slate-200)
- Status colors: green (Delivered), blue (Shipped), yellow (Processing), red (Backordered/high priority)
- Charts: custom SVG — no external charting library
- Layouts: CSS Grid
- No emojis in the UI

---

## i18n

The app supports English and Japanese. Translations live in:
- `client/src/locales/en.js`
- `client/src/locales/ja.js`

Switch language via the `LanguageSwitcher` component in the nav bar. The `useI18n` composable exposes `t(key)` for template usage.

---

## Utilities

`client/src/utils/currency.js` — formats numbers as currency strings:
```js
import { formatCurrency } from '@/utils/currency'
formatCurrency(1250000) // "$1,250,000"
```
