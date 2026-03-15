# Power Platform Core

**Universal microservice backend platform for building scalable web, mobile and AI systems.**

Power Platform Core provides a modular, production-ready foundation so developers can assemble backend systems like building blocks — without writing boilerplate from scratch.

---

## Architecture

```
┌─────────────────────────────────────────┐
│              Clients                    │
│   (Web / Mobile / AI services)         │
└────────────────┬────────────────────────┘
                 │
         ┌───────▼────────┐
         │  API Gateway   │  ← single entry point
         └───────┬────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼───┐  ┌───▼───┐  ┌────▼────┐
│Service│  │Service│  │ Service │
│   A   │  │   B   │  │    C   │
└───┬───┘  └───┬───┘  └────┬────┘
    │           │            │
    └───────────┴────────────┘
                 │
         ┌───────▼────────┐
         │   Core Lib     │  ← shared utilities
         └───────┬────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼───┐  ┌───▼───┐  ┌────▼────┐
│Postgr │  │ Redis │  │  Nginx  │
│  eSQL │  │       │  │         │
└───────┘  └───────┘  └─────────┘
```

---

## Main Components

| Repository | Description |
|---|---|
| `core` | Shared library — config, DB, Redis, logging, auth, utilities |
| `service-template` | Starter template for new microservices |
| `api-gateway` | Routing, rate-limiting, auth validation |
| `infrastructure` | Docker Compose + Nginx + PostgreSQL + Redis + PgBouncer |

---

## Technologies

FastAPI · PostgreSQL · Redis · Docker · Nginx · PgBouncer

---

## Use Cases

SaaS platforms · AI services · Mobile backends · Marketplaces · Education platforms · Real-time services

---

## Contributing

We welcome contributions! Please read our [Contributing Guidelines](../CONTRIBUTING.md) and [Code of Conduct](../CODE_OF_CONDUCT.md) before getting started.
