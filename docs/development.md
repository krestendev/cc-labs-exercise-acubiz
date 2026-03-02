# Development Guide

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 18+ | Frontend dev server and build |
| Python | 3.11+ | Backend runtime |
| [uv](https://github.com/astral-sh/uv) | latest | Python package manager |

---

## Setup & Running

### One-command startup (macOS/Linux)

```bash
./scripts/start.sh
```

This starts both servers and writes PID files to `/tmp/`:
- Backend: `http://localhost:8001`
- Frontend: `http://localhost:3000`
- API docs (Swagger): `http://localhost:8001/docs`

To stop:
```bash
./scripts/stop.sh
```

---

### Manual startup

**Backend**
```bash
cd server
uv venv && uv sync
uv run python main.py
```

**Frontend** (separate terminal)
```bash
cd client
npm install
npm run dev
```

Stop each server with `Ctrl+C`.

---

### Windows

The `scripts/` shell scripts are macOS/Linux only. Use the manual startup commands above, each in its own terminal window.

---

## Environment Variables

Copy `.env.example` to `.env` and adjust if needed. The default values work for local development without any changes.

---

## Running Tests

All tests are in `tests/backend/` and use pytest with FastAPI's `TestClient`.

```bash
# From the repo root
cd tests
pytest backend/ -v
```

Or from the repo root:
```bash
pytest tests/backend/ -v
```

**Test suite**: 55 tests across 4 files, runs in ~0.13 seconds.

| File | Tests | Coverage |
|------|-------|---------|
| `test_inventory.py` | 10 | Inventory endpoint — filtering, detail, 404, data validation |
| `test_orders.py` | 15 | Orders endpoint — all filters, multi-filter, calculations |
| `test_dashboard.py` | 13 | Summary — types, non-negative values, filter combinations |
| `test_misc_endpoints.py` | 17 | Demand, backlog, spending, root endpoint |

See `tests/README.md` and `tests/TEST_SUMMARY.md` for details.

---

## Production Build

```bash
cd client
npm run build
# Output: client/dist/
```

Serve `client/dist/` with any static file host. Point API calls to the deployed FastAPI backend.

---

## Key Conventions

Follow these to avoid common bugs:

### Vue frontend

- **Unique `v-for` keys**: Always use a stable field (`sku`, `id`, `month`) — never array `index`.
  ```html
  <!-- correct -->
  <tr v-for="item in items" :key="item.sku">
  <!-- wrong -->
  <tr v-for="(item, index) in items" :key="index">
  ```

- **Validate dates before calling date methods**:
  ```js
  // correct
  const month = date && new Date(date).getMonth()
  // wrong — throws if date is null/undefined
  const month = new Date(date).getMonth()
  ```

- **Raw data in `ref`, derived data in `computed`** — never store filtered/transformed data directly in a ref.

- **Filter values**: Strip `"all"` before sending to the API. The `api.js` client handles this automatically.

### Backend

- **Sync Pydantic models with JSON**: When you add or rename a field in a `server/data/*.json` file, update the corresponding Pydantic model in `server/main.py`.

- **Inventory has no time dimension**: Do not add a `month` filter to `/api/inventory` — inventory data is point-in-time, not historical.

- **Case-insensitive matching**: The backend filters categories and warehouses case-insensitively. Keep this behavior when adding new filters.

---

## Adding a New View

1. Create `client/src/views/MyView.vue` — delegate to the **vue-expert** agent for `.vue` file creation.
2. Add a route in `client/src/App.vue`.
3. Add a nav link in the sidebar section of `App.vue`.
4. Add a new API function in `client/src/api.js` if a new endpoint is needed.
5. Add the corresponding FastAPI endpoint and Pydantic model in `server/main.py`.

## Adding a New API Endpoint

1. Add the endpoint function in `server/main.py`.
2. Define a Pydantic response model.
3. Add the corresponding call in `client/src/api.js`.
4. Write tests in `tests/backend/` — use the **backend-api-test** skill.

---

## Tooling Notes

- Use the **vue-expert** agent for any `.vue` file creation or significant modification.
- Use the **backend-api-test** skill when writing or modifying pytest tests.
- Use GitHub MCP tools (`mcp__github__*`) for all GitHub operations.
- Use Playwright MCP tools (`mcp__playwright__*`) for browser-level testing.
