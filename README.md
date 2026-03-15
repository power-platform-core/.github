# Power Platform Core

**Universal microservice backend platform for building scalable web, mobile and AI systems.**

Power Platform Core provides a modular architecture that allows developers to quickly assemble backend systems like a puzzle — using battle-tested building blocks instead of writing boilerplate from scratch.

---

## Architecture

The platform is built on modern backend architecture principles:

- **Microservices** — independent, loosely coupled services
- **API Gateway** — single entry point for all external requests
- **Shared Core Library** — common utilities reused across all services
- **Stateless Services** — horizontal scaling out of the box
- **Horizontal Scalability** — add more instances as load grows

---

## Main Components

### Core
Shared libraries providing:
- Configuration management
- Database connection (PostgreSQL + PgBouncer)
- Redis integration
- Structured logging
- Authentication & authorization helpers
- Common utilities

### Service Template
A template repository to quickly spin up new services with the full stack pre-configured.

### API Gateway
Single entry point that routes external requests to the correct microservice, handles rate-limiting, auth validation, and observability.

### Infrastructure
Production-ready deployment stack:
- **Docker** — containerised services
- **PostgreSQL** — primary relational database
- **Redis** — caching and pub/sub
- **PgBouncer** — connection pooling
- **Nginx** — reverse proxy & TLS termination

---

## Technologies

| Layer | Technology |
|---|---|
| API Framework | FastAPI |
| Database | PostgreSQL |
| Cache / Broker | Redis |
| Containerisation | Docker |
| Reverse Proxy | Nginx |

---

## Use Cases

Power Platform Core can be used to build:

- SaaS platforms
- AI / ML inference services
- Mobile backends
- Marketplace systems
- Education platforms
- Real-time services

---

## Getting Started

Each component lives in its own repository under the [`power-platform-core`](https://github.com/power-platform-core) organisation. Start with the **Service Template** to create a new microservice, then wire it into the API Gateway.

See [CONTRIBUTING.md](CONTRIBUTING.md) to learn how to contribute.

---

## Community

- [Code of Conduct](CODE_OF_CONDUCT.md)
- [Security Policy](SECURITY.md)
- [Contributing Guidelines](CONTRIBUTING.md)
