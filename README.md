# Power Platform Core

Power Platform Core — universal microservice backend platform for building scalable applications.

The platform provides a modular architecture that allows developers to quickly assemble backend systems like a puzzle.

## Architecture

The platform is based on modern backend architecture principles:

- Microservices
- API Gateway
- Shared Core Library
- Stateless Services
- Horizontal Scalability

## Main Components

### Core
Shared libraries for:

- configuration
- database connection
- redis
- logging
- authentication
- common utilities

### Service Template

Template repository used to quickly create new services.

### API Gateway

Single entry point that routes external requests to the correct service.

### Infrastructure

Deployment stack including:

- Docker
- PostgreSQL
- Redis
- PgBouncer
- Nginx

## Technologies

- FastAPI
- PostgreSQL
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
