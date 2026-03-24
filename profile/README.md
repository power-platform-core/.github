# ⚡ Power Platform Core

**Power Platform Core** is a production-grade, modular microservice backend platform built with FastAPI. It enables developers to assemble scalable backend systems from pre-built, independent services — like building blocks.

```text
Internet → Nginx → API Gateway (:9000) → 35+ Microservices
                                        → PostgreSQL (per-service DB)
                                        → Redis (per-service DB)
                                        → RabbitMQ (async events)
```

---

## 🏗️ Architecture Principles

| Principle | Details |
| --- | --- |
| **Microservices** | Each service owns its data, migrations, and deployment |
| **Shared Core Library** | `power-fastapi-core` — eliminates duplication across all services |
| **API Gateway** | Single entry point, catch-all proxy with 503/504 handling |
| **Stateless Services** | JWT-based auth, Redis for sessions/cache |
| **UUID Primary Keys** | All models use `UUID(as_uuid=True)` |
| **Standardized Responses** | `DataResponse` / `PaginatedResponse` / `ErrorResponse` |
| **Exception Mapping** | `@handle_exceptions` → HTTP 400/401/403/404/409/422/500 |
| **Test Isolation** | SQLite in-memory per service (no Postgres required for tests) |

---

## 📦 Repositories

### 🔧 Foundation

| Repository | Description |
| --- | --- |
| **power-fastapi-core** | Shared library: BaseRepository, responses, exceptions, JWT, Redis, cache, middleware, decorators |
| **power-service-fastapi-template** | Starter template for new services |
| **power-fastapi-gateway** | API Gateway — reverse proxy routing all `/api/v1/*` traffic to upstream services |
| **power-infra** | Docker Compose stacks, Nginx config, PostgreSQL init scripts, per-service env files |

---

### 🔐 Identity & Access

| Repository | Port | Tables |
| --- | --- | --- |
| **power-auth-fastapi-service** | 8000 | users, sessions, refresh_tokens, password_resets, verification_tokens, social_accounts, api_keys |
| **power-user-fastapi-service** | 8001 | user_profiles, user_settings, user_activity |

---

### 🛒 Commerce

| Repository | Port | Tables |
| --- | --- | --- |
| **power-product-fastapi-service** | 8002 | brands, products, product_variants, product_images |
| **power-cart-fastapi-service** | 8003 | carts, cart_items |
| **power-order-fastapi-service** | 8004 | orders, order_items |
| **power-payment-fastapi-service** | 8005 | payments, payment_events |
| **power-promo-fastapi-service** | — | promo_codes, promo_usages |
| **power-subscription-fastapi-service** | — | subscription_plans, user_subscriptions |

---

### 📝 Content & Media

| Repository | Port | Tables |
| --- | --- | --- |
| **power-post-fastapi-service** | — | posts, post_categories |
| **power-category-fastapi-service** | — | categories |
| **power-tag-fastapi-service** | — | tags, entity_tags |
| **power-media-fastapi-service** | 8011 | media_items |
| **power-media-storage-fastapi-service** | 8007 | files |
| **power-file-upload-fastapi-service** | — | file uploads, MinIO/S3 adapter |
| **power-translation-fastapi-service** | 8009 | translations (entity_type, entity_id, field, language, value) |
| **power-review-fastapi-service** | — | reviews, review_replies |

---

### 💬 Social & Community

| Repository | Port | Tables |
| --- | --- | --- |
| **power-social-fastapi-service** | 8012 | follows |
| **power-social-interaction-fastapi-service** | 8008 | comments, likes, ratings, views |
| **power-chat-fastapi-service** | — | chat_rooms, chat_messages, chat_participants |
| **power-support-fastapi-service** | — | support_tickets, ticket_messages |

---

### 🤖 AI & ML Ecosystem

| Repository | Description |
| --- | --- |
| **power-ai-fastapi-service** | LLM orchestration — OpenAI / OpenRouter / DeepSeek / Mistral, chat, streaming, function calling, tool registry, cost tracking |
| **power-vector-fastapi-service** | Vector storage & retrieval — embeddings, chunking, Qdrant, collection management |
| **power-websearch-fastapi-service** | Web search adapters — Tavily / Serper / Brave, result normalization, caching |

---

### 📡 Notifications & Communication

| Repository | Port | Description |
| --- | --- | --- |
| **power-notification-fastapi-service** | 8006 | Notification inbox, preferences (email/sms/push) |
| **power-notifier-fastapi-service** | — | Multi-channel delivery orchestration |
| **power-email-fastapi-service** | — | SMTP / SendGrid / AWS SES email delivery, templates |
| **power-sms-fastapi-service** | — | SMS / OTP delivery (Eskiz, Play Mobile, Twilio) |

---

### 🧠 Platform Intelligence

| Repository | Description |
| --- | --- |
| **power-analytics-fastapi-service** | Event tracking, daily stats, dashboards |
| **power-search-fastapi-service** | Full-text search (Meilisearch / Elasticsearch adapter) |
| **power-recommendation-fastapi-service** | Personalized content and product recommendations |
| **power-audit-fastapi-service** | Universal audit log — admin/user/system actions, entity change diffs, security events |
| **power-config-fastapi-service** | Centralized config management — app settings, feature flags, targeting rules, Redis cache |

---

## 🗺️ Gateway Route Map

| Path Prefix | Upstream Service | Port |
| --- | --- | --- |
| `/api/v1/auth` | power-auth-fastapi-service | 8000 |
| `/api/v1/users` | power-user-fastapi-service | 8001 |
| `/api/v1/products` | power-product-fastapi-service | 8002 |
| `/api/v1/cart` | power-cart-fastapi-service | 8003 |
| `/api/v1/orders` | power-order-fastapi-service | 8004 |
| `/api/v1/payments` | power-payment-fastapi-service | 8005 |
| `/api/v1/notifications` | power-notification-fastapi-service | 8006 |
| `/api/v1/media-storage` | power-media-storage-fastapi-service | 8007 |
| `/api/v1/social-interaction` | power-social-interaction-fastapi-service | 8008 |
| `/api/v1/translations` | power-translation-fastapi-service | 8009 |
| `/api/v1/analytics` | power-analytics-fastapi-service | 8010 |
| `/api/v1/media` | power-media-fastapi-service | 8011 |
| `/api/v1/follows` | power-social-fastapi-service | 8012 |
| `/api/v1/ai` | power-ai-fastapi-service | — |
| `/api/v1/vector` | power-vector-fastapi-service | — |
| `/api/v1/search` | power-websearch-fastapi-service | — |
| `/api/v1/audit` | power-audit-fastapi-service | — |
| `/api/v1/config` | power-config-fastapi-service | — |
| `/health` | gateway-local | 9000 |
| `/health/services` | gateway (polls all upstreams) | 9000 |

---

## 🔢 Message Code Registry

| Range | Service |
| --- | --- |
| 1000–1999 | platform / power-fastapi-core |
| 2000–2999 | power-auth-fastapi-service |
| 3000–3999 | power-user-fastapi-service |
| 4000–4999 | validation |
| 5000–5999 | power-product-fastapi-service |
| 6000–6999 | power-payment-fastapi-service / power-order-fastapi-service |
| 7000–7999 | power-cart-fastapi-service |
| 8000–8999 | power-notification-fastapi-service |
| 9000–9999 | power-media-fastapi-service / power-media-storage-fastapi-service |
| 10000–10999 | power-analytics-fastapi-service |
| 11000–11999 | power-social-interaction-fastapi-service |
| 12000–12999 | power-social-fastapi-service |
| 13000–13999 | power-translation-fastapi-service |
| 14000–14999 | power-chat-fastapi-service |
| 15000–15999 | power-post-fastapi-service |
| 16000–16999 | power-tag-fastapi-service |
| 17000–17999 | power-support-fastapi-service |
| 18000–18999 | power-subscription-fastapi-service |
| 19000–19999 | power-ai-fastapi-service |
| 20000–20999 | power-vector-fastapi-service / power-websearch-fastapi-service |
| 21000–21999 | power-audit-fastapi-service |
| 22000–22999 | power-config-fastapi-service |
| 23000+ | reserved for future services |

---

## 🛠️ Technology Stack

| Layer | Technologies |
| --- | --- |
| **Framework** | FastAPI, Pydantic v2, pydantic-settings |
| **ORM** | SQLAlchemy (async), Alembic |
| **Database** | PostgreSQL (per-service), SQLite (tests) |
| **Cache** | Redis (per-service DB index) |
| **Message Broker** | RabbitMQ (aio-pika) |
| **AI Providers** | OpenAI, OpenRouter, DeepSeek, Mistral |
| **Vector DB** | Qdrant |
| **Search** | Meilisearch / Elasticsearch |
| **File Storage** | MinIO / AWS S3 |
| **Auth** | JWT (HS256), bcrypt, refresh token rotation |
| **HTTP Client** | httpx (async), tenacity (retry) |
| **Server** | Uvicorn / Gunicorn |
| **Container** | Docker, Docker Compose |
| **Proxy** | Nginx |
| **Language** | Python 3.12 |

---

## 🧪 Test Coverage

| Metric | Value |
| --- | --- |
| Total tests | 371+ |
| Test isolation | SQLite in-memory (no external deps) |
| Auth bypass | `X-Test-Admin` header (TEST_MODE only) |
| Coverage | All CRUD, pagination, error paths, auth flows |

---

## ⚙️ power-fastapi-core — Shared Library

The backbone of the platform. Every service installs it via:

```bash
pip install -e ../power-fastapi-core
```

Key modules:

| Module | Purpose |
| --- | --- |
| `power_core.app` | `create_app()` factory — auto-wires health, middleware, lifecycle |
| `power_core.repository.base` | `BaseRepository[T]` — CRUD, pagination, search, bulk ops, upsert |
| `power_core.responses` | `DataResponse`, `PaginatedResponse`, `ErrorResponse` |
| `power_core.exceptions` | `NotFoundError`, `ConflictError`, `ForbiddenError`, `ValidationError`, … |
| `power_core.utils.decorators` | `@handle_exceptions` — maps exceptions to HTTP status codes |
| `power_core.security` | JWT encode/decode, bcrypt, token blacklist |
| `power_core.cache` | Redis model cache, TTL helpers |
| `power_core.middleware` | HTTP request logging |
| `power_core.messages` | `ResponseMessages` base class for typed message codes |
| `power_core.database` | Async SQLAlchemy engine manager, `get_db` dependency |
| `power_core.config` | `Settings` base (pydantic-settings), `TagInfo` type |

---

## 🚀 Quick Start

```bash
# 1. Start infrastructure
cd power-infra
docker compose up -d

# 2. Install shared core
pip install -e power-fastapi-core

# 3. Run any service
cd power-auth-fastapi-service
pip install -r requirements.txt
alembic upgrade head
uvicorn main:app --reload --port 8000
```

---

## 📁 Standard Service Structure

```text
power-<name>-fastapi-service/
├── main.py                         # FastAPI app entry point
├── app/
│   ├── api/v1/routers/             # Route handlers (@handle_exceptions)
│   ├── services/                   # Business logic
│   ├── repositories/               # DB layer (extends BaseRepository)
│   ├── models/                     # SQLAlchemy models
│   ├── schemas/                    # Pydantic request/response schemas
│   ├── messages/                   # Typed message codes (errors + success)
│   └── config/
│       ├── settings.py             # Service-specific env settings
│       └── tags_info.py            # OpenAPI tag groups
├── migrations/                     # Alembic per-service migrations
├── tests/                          # Pytest, SQLite in-memory
├── requirements.txt
└── docker-compose.yml
```
- Redis
- Docker
- Nginx

## Use Cases

Power Platform Core can be used to build:

- SaaS platforms
- AI services
- mobile backends
- marketplace systems
- education platforms
- real-time services

## Goal

The goal of this platform is to make backend development faster by allowing developers to assemble services like building blocks.
