# Factory Inventory Management System - Documentation

A full-stack demo application for managing factory inventory, orders, demand forecasting, and spending analytics.

![Dashboard](dashboard-screenshot.png)

## Quick Start

```bash
# One-command startup (macOS/Linux)
./scripts/start.sh

# Backend: http://localhost:8001
# Frontend: http://localhost:3000
# API Docs: http://localhost:8001/docs
```

See [Development Guide](development.md) for manual setup and Windows instructions.

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](architecture.md) | Tech stack, directory structure, data flow |
| [API Reference](api-reference.md) | All endpoints, parameters, and response schemas |
| [Frontend](frontend.md) | Views, components, composables, and i18n |
| [Data Models](data-models.md) | Entity schemas and JSON data files |
| [Development](development.md) | Setup, scripts, testing, and conventions |

## Feature Overview

- **Dashboard** - Summary metrics with interactive filtering
- **Inventory** - Stock levels, reorder alerts, warehouse/category breakdown
- **Orders** - Order lifecycle tracking with status management
- **Demand** - Forecasting with trend indicators (increasing/stable/decreasing)
- **Backlog** - Delayed fulfillment items with priority levels and purchase orders
- **Spending** - Procurement, operational, labor, and overhead cost analytics
- **Reports** - Quarterly and monthly trend reports

## Tech Stack

- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: In-memory mock data loaded from `server/data/*.json`
- **Tests**: pytest with FastAPI TestClient (55 tests)
