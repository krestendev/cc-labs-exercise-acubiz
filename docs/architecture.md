# Architecture

## Tech Stack

| Layer | Technology | Port |
|-------|-----------|------|
| Frontend | Vue 3 + Composition API + Vite | 3000 |
| Backend | Python FastAPI + Pydantic | 8001 |
| Data | In-memory JSON (no database) | — |
| Tests | pytest + FastAPI TestClient | — |

## Directory Structure

```
cc-labs-exercise-acubiz/
├── client/                      # Vue 3 frontend
│   └── src/
│       ├── views/               # Page-level components (7 views)
│       ├── components/          # Reusable UI components (9 components)
│       ├── composables/         # Shared logic (useFilters, useAuth, useI18n)
│       ├── locales/             # i18n translation files (en.js, ja.js)
│       ├── utils/               # Helpers (currency.js)
│       ├── api.js               # Centralized axios-based API client
│       ├── App.vue              # Root component with routing and nav
│       └── main.js              # Vue app entry point
├── server/                      # FastAPI backend
│   ├── main.py                  # All API endpoints and Pydantic models
│   ├── mock_data.py             # JSON data loading module
│   └── data/                    # Mock data JSON files
│       ├── inventory.json
│       ├── orders.json
│       ├── demand_forecasts.json
│       ├── backlog_items.json
│       ├── purchase_orders.json
│       ├── spending.json
│       └── transactions.json
├── tests/
│   └── backend/                 # pytest test suite (55 tests)
│       ├── conftest.py
│       ├── test_inventory.py
│       ├── test_orders.py
│       ├── test_dashboard.py
│       └── test_misc_endpoints.py
├── scripts/
│   ├── start.sh                 # Start both servers (macOS/Linux)
│   └── stop.sh                  # Stop both servers
├── docs/                        # Project documentation
└── README.md
```

## Data Flow

```
User interaction
      |
      v
FilterBar.vue            (4 filters: period, warehouse, category, status)
      |
      v
useFilters.js composable (reactive filter state, shared across all views)
      |
      v
api.js                   (builds query params, calls axios)
      |
      v  HTTP GET with query params
FastAPI main.py          (applies filters, validates with Pydantic)
      |
      v
mock_data.py             (in-memory data loaded from server/data/*.json)
      |
      v  filtered + validated JSON
api.js                   (returns data to view)
      |
      v
View component           (stores raw data in ref(), derives display in computed())
      |
      v
Template                 (v-for with unique keys, renders UI)
```

## Filter System

Four global filters apply consistently across all views:

| Filter | Composable key | Query param | Example values |
|--------|---------------|-------------|----------------|
| Time Period | `selectedPeriod` | `month` | `"all"`, `"2025-01"`, `"Q1-2025"` |
| Warehouse | `selectedLocation` | `warehouse` | `"all"`, `"San Francisco"`, `"London"`, `"Tokyo"` |
| Category | `selectedCategory` | `category` | `"all"`, `"Circuit Boards"`, `"Sensors"` |
| Order Status | `selectedStatus` | `status` | `"all"`, `"Delivered"`, `"Shipped"`, `"Processing"`, `"Backordered"` |

- Filter state lives in `useFilters.js` as a module-level singleton (shared across all views without prop drilling)
- `"all"` values are stripped before sending to the API (treated as "no filter")
- The backend applies filters case-insensitively
- Inventory endpoints do **not** support the `month` filter (inventory has no time dimension)

## Key Design Decisions

- **No database**: All data is loaded from JSON files into memory at server startup. Changes do not persist across restarts.
- **Raw vs. derived data**: Views store unfiltered API responses in `ref()` and use `computed()` for any derived/display logic.
- **Pydantic validation**: Every API response is typed via Pydantic models. When changing JSON structure, update `server/main.py` models.
- **CORS open**: The backend allows all origins (`*`) — development only; not production-ready.
- **i18n**: English and Japanese supported via `useI18n` composable and locale files.
