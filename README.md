# Healthy 365

Wellness activity **MVP**: log movement-style metrics, browse a simple **community feed**, and run the stack with **Node.js**, **React**, **PostgreSQL**, **Docker**, and **nginx**. The project is **not** affiliated with HPB, Apple, or LumiHealth; it is a demo scaffold for citizen-style health engagement patterns.

### Documentation by audience

| Who | Document |
|-----|----------|
| **Product / programme / delivery managers** who need programme context plus APIs and gaps | [**Product manager briefing (technical)**](docs/product-manager-guide.md) |
| **End users / participants** (plain language “how do I use it?”) | [**User guide**](docs/user-guide.md) |

This **`README`** remains the canonical **engineering** setup guide for the repo.

## Contents

- [Features](#features)
- [Repository layout](#repository-layout)
- [Prerequisites](#prerequisites)
- [Quick start](#quick-start)
- [Configuration](#configuration)
- [API](#api)
- [Docker Compose services](#docker-compose-services)
- [Kubernetes](#kubernetes)
- [CI/CD](#cicd)
- [Compliance & security (production)](#compliance--security-production)
- [Operations](#operations)
- [Tech mapping](#tech-mapping)

## Features

- Responsive **React** (Vite) UI
- **PWA-oriented** `manifest.webmanifest` (add a service worker for full offline support)
- **REST API** with **liveness** (`/health`) and **readiness** (`/ready`) for orchestration
- **PostgreSQL** persistence with optional init SQL
- **docker compose** for one-command full stack
- Example **Kubernetes** manifests and **GitHub Actions** workflow

## Repository layout

| Path | Purpose |
|------|---------|
| `docs/` | [Product PM briefing](docs/product-manager-guide.md), [participant user guide](docs/user-guide.md) |
| `frontend/` | React app, Vite build, nginx config for SPA + API proxy |
| `backend/` | Express API |
| `db/init/` | Postgres bootstrap SQL |
| `k8s/` | Example Deployment / Service |
| `.github/workflows/` | CI scaffold |

## Prerequisites

- **Node.js 20+** (local development)
- **Docker Desktop** (or Docker Engine + Compose v2), for Postgres and optional full stack

## Quick start

### Full stack (recommended)

From the repo root:

```bash
docker compose up --build
```

Open **http://localhost:8080** (nginx serves the UI and proxies `/api` to the API).

**Windows (PowerShell):**

```powershell
docker compose up --build
```

### Local development (API + Postgres in Docker)

**Terminal 1 — database:**

```bash
docker compose up -d db
```

**Terminal 2 — API:**

```bash
cd backend
npm install
npm run dev
```

**Terminal 3 — UI:**

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173**. Vite proxies `/api` to `http://localhost:4000`.

## Configuration

| File | Purpose |
|------|---------|
| `backend/.env.example` | `PORT`, `DATABASE_URL` |
| `frontend/.env.example` | Optional `VITE_API_BASE` when the UI does not share origin with the API |

Copy examples to `.env` / `.env.local` as needed (do not commit secrets).

Default local DB URL (matches Compose):

`postgresql://healthy365:healthy365@localhost:5432/healthy365`

## API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Process liveness |
| GET | `/ready` | Database connectivity probe |
| GET | `/api/v1/stats/summary` | Aggregate counts |
| GET | `/api/v1/entries` | Recent entries (newest first, capped) |
| POST | `/api/v1/entries` | Body: `{ "nickname": string, "activity_type?": string, "amount": number, "note?": string }` |

No authentication in this MVP — treat as demo-only.

## Docker Compose services

| Service | Image / build | Port (host) | Role |
|---------|---------------|-------------|------|
| `db` | `postgres:16-alpine` | (internal) | Primary datastore |
| `api` | `backend/Dockerfile` | (internal) | REST API |
| `web` | `frontend/Dockerfile` | **8080 → 80** | Static UI + reverse proxy to API |

Data volume: `pgdata`.

## Kubernetes

See `k8s/example-deployment.yaml` for an illustrative Deployment, Service, and Secret placeholder. Adjust image registry, resource limits, **Ingress**, **NetworkPolicy**, secrets management, and HPA before production.

## CI/CD

`.github/workflows/ci.yml` installs dependencies and builds the frontend plus a quick syntax check on the backend. Extend with:

- Container image build and push
- Dependency and image scanning (e.g. Trivy, Dependabot)
- SBOM generation
- Staged deployments to AWS/Azure

## Compliance & security (production)

This repo is **not production-ready** for personal data.

**PDPA-style themes to address before go-live:**

1. Purpose limitation and documented lawful basis  
2. Notice and consent where required  
3. Data minimisation (avoid NRIC-class identifiers where possible)  
4. Retention schedules and secure disposal  
5. Encryption in transit and at rest; secret management (**AWS Secrets Manager**, **Azure Key Vault**, cluster secret stores)  
6. Breach notification readiness and playbook alignment  
7. Audit trails for access and privileged operations  

**Security baseline:**

- Prefer API gateways / WAF, least-privilege IAM, segmented networks  
- No secrets in git; scan dependencies and containers in CI  
- Operational monitoring and alerting (see observability hints under [Tech mapping](#tech-mapping))

## Operations

### Suggested backlog

1. **P0** — Authentication (e.g. OIDC / national IdP patterns), encryption, audit logs  
2. **P1** — Rich domain model, consent records, rate limits, admin APIs  
3. **P2** — Mobile apps, notifications, privacy-conscious analytics  
4. **P3** — Gamification, integrations, stronger PWA offline  

### Release checklist (short)

- [ ] Security and privacy reviews completed  
- [ ] Load/stress validation on staging  
- [ ] Runbooks and ownership documented  
- [ ] Safe rollout (feature flags / canaries)  

### Incident response (outline)

1. Detect (alerts on errors, latency, saturation)  
2. Triage severity and communicate  
3. Contain (flags, throttles, failover)  
4. Eradicate root cause  
5. Recover (verify `/ready`, critical user paths)  
6. Post-incident review and follow-ups  

### Production support notes

Define **SLOs** for critical paths, run **on-call** rotations with escalation paths, and **test restores** regularly for Postgres (or equivalent RDB).

## Tech mapping

| Area | MVP in this repo | Common enterprise alternatives |
|------|------------------|--------------------------------|
| Runtime / API | Node.js + Express | Java **Spring Boot**, Python **Django** |
| UI | **React** (Vite) | **Angular**, or hybrid mobile via Flutter/RN |
| Database | **PostgreSQL** | **Oracle**, **Azure SQL / MSSQL**, **Cosmos DB** (via adapter) |
| Hosting | Compose + Dockerfiles | **AWS** (ECS/EKS/RDS), **Azure** (AKS/App Service) |
| Observability | `/health`, `/ready` | APM, metrics stack, alerting, **OpenTelemetry** |

---

**Branding:** *Healthy 365* is used as a demo name; verify naming and trademarks with your legal team before external use.
