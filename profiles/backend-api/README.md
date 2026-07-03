# Profile: Backend API

> **Purpose:** Backend API and service projects  > **When to Use:** REST APIs, GraphQL, gRPC, microservices

---

## Description

Optimized for backend development.

Supports:
- Python (FastAPI, Django, Flask)
- JavaScript/TypeScript (Express, NestJS)
- Go (Gin, Echo, Fiber)
- Rust (Actix, Axum)
- Java/Kotlin (Spring Boot)
- Ruby (Rails, Sinatra)

---

## Detection Hints

**Indicators:**
- `pyproject.toml`, `requirements.txt` → Python
- `package.json` with `express`, `fastify` → Node.js
- `go.mod` → Go
- `Cargo.toml` → Rust
- `pom.xml`, `build.gradle` → Java
- `Gemfile` → Ruby

**Confidence:**
- High when package manifest + API routes found

---

## Capabilities

All 9 flows with backend-specific patterns:
- API design (REST, GraphQL, gRPC)
- Database integration
- Authentication/authorization
- Error handling
- Logging and monitoring
- Docker/containerization

## Slots Discovered

- project-name, source-directory, primary-language
- framework (FastAPI/Django/Express/Go/Rust)
- database (PostgreSQL/MongoDB/MySQL/Redis)
- auth (JWT/OAuth/Basic/API keys)
- api-style (REST/GraphQL/gRPC)
- containerization (Docker/Kubernetes)

---

## Backend-Specific Conventions

- Layered architecture (Controller/Service/Repository)
- Input validation
- Error handling with proper HTTP status codes
- Database transactions
- API versioning

See `rules.md` for complete convention list.
