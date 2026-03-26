# Nomus Connector

**Reusable integration layer between Nomus ERP and external systems.**

> Python-based connector normalizing 59+ Nomus ERP REST endpoints into structured, English-named schemas. Deploy as CLI, REST API, or microservice.

---

## Overview

Nomus Connector bridges the gap between Nomus ERP (a Brazilian enterprise resource planning system) and modern data platforms. It provides centralized, normalized access to the entire Nomus API surface — translating Portuguese field names to English, validating data with strict types, and outputting to any downstream system via pluggable adapters.

### Key Capabilities

- **Full API Coverage** — 59+ REST endpoints across products, customers, orders, invoices, inventory, production, HR, purchasing, and finance
- **Automatic Field Translation** — Portuguese-to-English mapping at both query and response layers
- **Strict Type Safety** — Pydantic v2 models with custom types (NomusFloat, NomusDate)
- **Pluggable Adapters** — Built-in PostgreSQL upsert and webhook POST; extensible for custom integrations
- **Multiple Interfaces** — CLI tool, REST API server, or importable Python library
- **Async Throughout** — High-throughput data pipelines using Python async/await
- **API Schema Monitoring** — Automated daily checks for Nomus Postman collection changes

---

## Architecture

```
                    ┌───────────────────────────────────┐
                    │         Nomus ERP API             │
                    │    (59+ REST endpoints, PT-BR)    │
                    └───────────────┬───────────────────┘
                                    │
                          HTTP (httpx async)
                        + retry / rate-limit
                                    │
                    ┌───────────────▼───────────────────┐
                    │        NomusClient Core           │
                    │  ┌─────────────────────────────┐  │
                    │  │  Field Translation Layer    │  │
                    │  │  (PT-BR ↔ EN automatic)     │  │
                    │  ├─────────────────────────────┤  │
                    │  │  26 Pydantic v2 Models      │  │
                    │  │  (strict validation)        │  │
                    │  ├─────────────────────────────┤  │
                    │  │  Pagination / Streaming     │  │
                    │  │  (async generators)         │  │
                    │  └─────────────────────────────┘  │
                    └───────┬───────────┬───────────────┘
                            │           │
                   ┌────────▼──┐  ┌─────▼──────┐
                   │    CLI    │  │  REST API  │
                   │  (Typer)  │  │  (FastAPI) │
                   └────────┬──┘  └─────┬──────┘
                            │           │
              ┌─────────────┼───────────┼─────────────┐
              │             │           │             │
     ┌────────▼───┐  ┌─────▼────┐  ┌───▼──────┐  ┌──▼──────┐
     │  PostgreSQL │  │ Webhook  │  │  JSON    │  │ Custom  │
     │  (upsert)  │  │  (POST)  │  │ (stdout) │  │ Adapter │
     └────────────┘  └──────────┘  └──────────┘  └─────────┘
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Language** | Python 3.11+ |
| **HTTP Client** | httpx (async, retry, rate limiting) |
| **Data Validation** | Pydantic v2, custom types |
| **CLI** | Typer + Rich (table output) |
| **REST API** | FastAPI + Uvicorn |
| **Database** | PostgreSQL (asyncpg), SQLAlchemy 2.0, Alembic |
| **Containerization** | Docker (multi-stage, Python 3.12-slim) |
| **Deployment** | Fly.io (Sao Paulo region) |
| **CI/CD** | GitHub Actions (Python 3.11 + 3.12 matrix) |
| **Linting** | Ruff, mypy (strict) |
| **Testing** | pytest, pytest-asyncio, respx, Playwright |

---

## Entity Coverage

### Core
Products, Customers, Contacts, Orders, Purchase Orders, Invoices (products & services), Companies, Users

### Financial
Accounts Receivable/Payable, Payments, Receipts, Price Tables, Payment Methods, Payment Conditions, Bank Accounts, Bank Slips, Cost Centers

### Inventory & Warehousing
Inventories, Stock Movements, Stock Balances, Stock Lot/Serial, Stock Sectors, Inventory Documents, Movement Types

### Production
Work Orders, Work Centers, Work Appointments, Allocations, Production Requisitions, Product Routes, Order Route Operations, Resources, Processes

### Purchasing
Supplier Products, BOM Components, Units of Measure, Service Contracts, Proposals, Purchase Requests, Credit Notes

### HR & Admin
Employees, Persons, User Groups, Activities

---

## Features

### Data Normalization
- 26 Pydantic v2 models covering major ERP entities
- Automatic Portuguese > English field name translation
- Custom types handle Nomus-specific formats (decimals, dates)
- `extra="ignore"` policy gracefully handles unknown API fields

### Adapter Pattern
- **PostgreSQL** — Auto-creates tables from model schema, performs UPSERT with configurable conflict keys, connection pooling via asyncpg
- **Webhook** — POSTs JSON batches to any HTTP endpoint with exponential backoff retry
- **Extensible** — Write custom adapters by inheriting `AbstractAdapter` (~30 lines)

### Resilience
- Exponential backoff retry logic
- Rate limiting (429 handling)
- Configurable timeouts
- Automatic pagination via async generators

### API Schema Monitoring
- Daily GitHub Actions workflow downloads Nomus Postman collection
- Detects schema changes and versions history
- Notifies via Slack/webhook when API changes

### Multi-Tenant Support
- Built-in SQLAlchemy backend for tenant/API key management
- SaaS-ready credential isolation

---

## Interfaces

### CLI
```
nomus-connector fetch products --query "active==true" --format table
nomus-connector sync orders --adapter postgres
nomus-connector serve --host 0.0.0.0 --port 8000
```

### REST API
```
GET  /health                         # Liveness check
GET  /api/{entity}?page=1&query=...  # List normalized records
GET  /api/{entity}/{id}              # Fetch single record
GET  /api/v1/purchase-orders/{id}    # Specialized SaaS endpoint
```

---

## Deployment

| Target | Configuration |
|--------|--------------|
| **Docker** | Multi-stage build, Python 3.12-slim, ~256MB |
| **Docker Compose** | connector + postgres (health checks) |
| **Fly.io** | Sao Paulo region (gru), auto-scale 0-N, persistent volume |
| **CI/CD** | GitHub Actions: lint > type-check > test (Python 3.11 + 3.12) |

---

## Related Projects

- [Invoice Flow](https://github.com/aleavenger/invoice-flow-overview) — Uses Nomus Connector for PO validation in the invoice automation pipeline
- [Dev Company](https://github.com/aleavenger/dev-company-overview) — AI-powered autonomous company built on PaperclipAI

---

## License

This is a proprietary project. Source code is not publicly available.

Architecture overview and technical details available on request.

---

**Built by [Alexandre Pereira dos Santos](https://github.com/aleavenger)**
