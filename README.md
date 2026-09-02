# 🌱 AgriTrack Pro — Cloud-Based Fertilizer Usage Tracking Platform

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#license)
[![Node](https://img.shields.io/badge/node-%3E%3D18.0.0-green)](#tech-stack)
[![Coverage](https://img.shields.io/badge/coverage-92%25-brightgreen)](#testing)
[![Uptime](https://img.shields.io/badge/uptime-99.9%25-blue)](#)

**AgriTrack Pro** is a cloud-based platform for recording, monitoring, analyzing, and
reporting fertilizer application across farms, fields, and crop cycles. It helps
agronomists, farm managers, and compliance officers reduce input waste, stay within
regulatory nutrient limits, and make data-driven decisions on fertilizer usage —
accessible from any device, anywhere.

---
website link : https://fe646z-hgw03p6nc-arcadawebapps5.vercel.app
## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Architecture](#architecture)
4. [Tech Stack](#tech-stack)
5. [System Requirements](#system-requirements)
6. [Getting Started](#getting-started)
7. [Environment Variables](#environment-variables)
8. [Project Structure](#project-structure)
9. [Database Schema (Core Entities)](#database-schema-core-entities)
10. [API Reference](#api-reference)
11. [User Roles & Permissions](#user-roles--permissions)
12. [Reporting & Analytics](#reporting--analytics)
13. [Notifications & Alerts](#notifications--alerts)
14. [Integrations](#integrations)
15. [Security & Compliance](#security--compliance)
16. [Testing](#testing)
17. [Deployment](#deployment)
18. [Monitoring & Observability](#monitoring--observability)
19. [Backup & Disaster Recovery](#backup--disaster-recovery)
20. [Roadmap](#roadmap)
21. [Contributing](#contributing)
22. [Support](#support)
23. [License](#license)

---

## Overview

Fertilizer over-application drives up input costs, harms soil health, and contributes
to nutrient runoff into waterways. **AgriTrack Pro** gives agricultural operations of
any size a single source of truth for:

- What fertilizer was applied, where, when, by whom, and at what rate.
- How actual usage compares against agronomic recommendations and regulatory caps
  (e.g., nitrogen/phosphorus limits per acre per season).
- Inventory levels, reorder points, and cost-per-acre trends.
- Field-level and farm-level compliance reporting for audits and certifications
  (e.g., 4R Nutrient Stewardship, organic certification, nutrient management plans).

The platform is delivered as a multi-tenant SaaS product with a responsive web
dashboard, a mobile-friendly field data-entry experience, and a public REST/GraphQL
API for integration with farm equipment, ERP systems, and third-party agronomy tools.

---

## Key Features

### 🌾 Field & Crop Management
- Define farms, fields, sub-plots, and boundaries (GeoJSON/KML import & map view).
- Track crop type, planting date, growth stage, and rotation history per field.
- Soil test result logging (N-P-K, pH, organic matter) with historical trends.

### 📋 Fertilizer Application Logging
- Log applications by product, rate, method (broadcast, banded, fertigation, foliar),
  equipment used, weather conditions, and applicator.
- Bulk import via CSV/Excel or equipment telemetry (ISOBUS/ISO-XML) upload.
- Offline-first mobile entry with automatic sync when connectivity resumes.

### 📦 Inventory & Procurement
- Real-time stock levels per warehouse/site with automatic deduction on application.
- Low-stock alerts, reorder suggestions, and supplier/vendor management.
- Cost tracking per product, per field, and per crop cycle.

### 📊 Analytics & Compliance Dashboards
- Nutrient balance (applied vs. recommended vs. regulatory limit) by field/season.
- Cost-per-acre, yield-correlated ROI, and year-over-year usage comparisons.
- One-click exportable compliance reports (PDF/CSV) for auditors and regulators.

### 🔔 Smart Alerts
- Threshold breach alerts (e.g., nitrogen limit exceeded, restricted application window).
- Weather-based application-risk warnings (rain forecast, high wind, frost).
- Task reminders for scheduled applications and re-application windows.

### 🔗 Integrations
- Weather data providers, GPS/telematics from farm equipment, accounting/ERP export,
  and agronomy advisory platforms.

### 👥 Multi-Tenant & Role-Based Access
- Organization → Farm → Field hierarchy with scoped permissions.
- Roles for Admins, Agronomists, Farm Managers, Field Applicators, and Auditors.

---

## Architecture

AgriTrack Pro follows a cloud-native, service-oriented architecture deployed on
Kubernetes, designed for horizontal scalability and high availability.

```
                              ┌──────────────────────────┐
                              │        CDN / WAF         │
                              │   (CloudFront + AWS WAF) │
                              └────────────┬─────────────┘
                                           │
                              ┌────────────▼─────────────┐
                              │     Web App (Next.js)     │
                              │  Server-rendered + PWA    │
                              └────────────┬─────────────┘
                                           │ HTTPS / REST + GraphQL
                              ┌────────────▼─────────────┐
                              │       API Gateway          │
                              │ (Auth, rate limiting, WAF) │
                              └───┬─────────┬─────────┬───┘
                    ┌─────────────┘         │         └─────────────┐
          ┌─────────▼────────┐   ┌──────────▼─────────┐   ┌─────────▼─────────┐
          │ Application Svc   │   │ Reporting/Analytics │   │ Notification Svc  │
          │ (Node.js/NestJS)  │   │ Svc (Node.js + BI)  │   │ (Node.js + SES/SNS)│
          └─────────┬────────┘   └──────────┬─────────┘   └─────────┬─────────┘
                    │                        │                        │
          ┌─────────▼────────────────────────▼────────────────────────▼─────────┐
          │                     Message Queue (Amazon SQS / Kafka)               │
          └─────────┬────────────────────────┬────────────────────────┬─────────┘
                    │                        │                        │
          ┌─────────▼────────┐   ┌──────────▼─────────┐   ┌─────────▼─────────┐
          │  PostgreSQL (RDS) │   │  TimescaleDB (usage │   │  Redis (cache /   │
          │  Core relational  │   │  time-series data)  │   │  sessions/queues) │
          └───────────────────┘   └──────────────────────┘   └───────────────────┘
                    │
          ┌─────────▼────────┐
          │  S3 (documents,   │
          │  reports, imports)│
          └───────────────────┘
```

- **Multi-tenancy:** row-level isolation via `organization_id` on every table, enforced
  through PostgreSQL Row-Level Security (RLS).
- **Event-driven processing:** application logs, inventory deductions, and alert
  evaluations are processed asynchronously via the message queue for resilience.
- **Time-series storage:** TimescaleDB stores high-volume application/usage events
  for fast aggregation and trend analysis.

---

## Tech Stack

| Layer                | Technology                                                        |
|-----------------------|--------------------------------------------------------------------|
| Frontend              | Next.js 14 (React 18, TypeScript), TailwindCSS, Recharts, Mapbox GL |
| Mobile / Offline PWA  | Next.js PWA module, IndexedDB (Dexie.js) for offline sync           |
| Backend API           | Node.js 20, NestJS, GraphQL (Apollo Server) + REST                  |
| Authentication        | Auth0 / AWS Cognito, JWT, OAuth2, SAML SSO for enterprise tenants    |
| Primary Database      | PostgreSQL 15 (Amazon RDS, Multi-AZ)                                |
| Time-Series Database  | TimescaleDB extension                                               |
| Caching / Sessions    | Redis (Amazon ElastiCache)                                          |
| Message Queue         | Amazon SQS / Apache Kafka                                           |
| File & Report Storage | Amazon S3                                                           |
| Search                | OpenSearch (Elasticsearch)                                          |
| Infrastructure        | Docker, Kubernetes (EKS), Terraform (IaC)                           |
| CI/CD                 | GitHub Actions, ArgoCD                                              |
| Monitoring            | Datadog, Prometheus + Grafana, Sentry                               |
| Notifications         | Amazon SES (email), Amazon SNS (SMS/push), Twilio (fallback)         |

---

## System Requirements

**For local development:**

- Node.js ≥ 18.x and npm ≥ 9.x (or pnpm ≥ 8.x)
- Docker Desktop ≥ 24.x and Docker Compose ≥ 2.x
- PostgreSQL ≥ 15 (via Docker, provided in `docker-compose.yml`)
- Redis ≥ 7 (via Docker)
- Git

**For production deployment:**

- Kubernetes cluster ≥ 1.27 (EKS/GKE/AKS)
- Helm ≥ 3.12
- Terraform ≥ 1.6
- Valid TLS certificates (managed via cert-manager or ACM)

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/agritrack-pro.git
cd agritrack-pro
```

### 2. Install dependencies

```bash
# Root workspace (monorepo managed with pnpm workspaces / Turborepo)
pnpm install
```

### 3. Configure environment variables

```bash
cp .env.example .env
# then edit .env with your local values (see Environment Variables section)
```

### 4. Start local infrastructure (Postgres, Redis, LocalStack for S3/SQS)

```bash
docker compose up -d
```

### 5. Run database migrations and seed sample data

```bash
pnpm --filter api migrate:latest
pnpm --filter api seed:dev
```

### 6. Start the development servers

```bash
pnpm dev
```

This starts:
- Web app at `http://localhost:3000`
- API (REST + GraphQL) at `http://localhost:4000`
- GraphQL Playground at `http://localhost:4000/graphql`

### 7. Log in with a seeded demo account

```
Email:    demo.admin@agritrackpro.io
Password: DemoPass123!
```

---

## Environment Variables

| Variable                  | Description                                         | Example                              |
|----------------------------|------------------------------------------------------|---------------------------------------|
| `DATABASE_URL`             | PostgreSQL connection string                         | `postgres://user:pass@localhost:5432/agritrack` |
| `REDIS_URL`                | Redis connection string                              | `redis://localhost:6379`             |
| `JWT_SECRET`               | Secret for signing access tokens                     | `supersecretvalue`                    |
| `AUTH0_DOMAIN`             | Auth0 tenant domain                                  | `agritrack.us.auth0.com`             |
| `AUTH0_CLIENT_ID`          | Auth0 application client ID                          | `abc123...`                           |
| `AWS_REGION`               | AWS region for S3/SQS/SES                            | `us-east-1`                          |
| `S3_BUCKET_NAME`           | Bucket for reports/imports                           | `agritrack-prod-reports`             |
| `MAPBOX_ACCESS_TOKEN`      | Mapbox GL token for field mapping                    | `pk.eyJ1...`                          |
| `WEATHER_API_KEY`          | Third-party weather API key                          | `xxxxxxxx`                            |
| `SENTRY_DSN`               | Error monitoring endpoint                            | `https://xxxx@sentry.io/xxxx`        |
| `NODE_ENV`                 | Environment mode                                     | `development` / `staging` / `production` |

A full annotated list is provided in `.env.example`.

---

## Project Structure

```
agritrack-pro/
├── apps/
│   ├── web/                  # Next.js frontend (dashboard, mobile PWA)
│   └── api/                  # NestJS backend (REST + GraphQL)
├── packages/
│   ├── ui/                   # Shared React component library
│   ├── config/               # Shared ESLint/TSConfig/Tailwind config
│   └── types/                # Shared TypeScript types & DTOs
├── infra/
│   ├── terraform/            # IaC for AWS resources
│   └── k8s/                  # Helm charts & Kubernetes manifests
├── docs/                     # Architecture decisions, API specs, runbooks
├── docker-compose.yml
├── turbo.json
├── .env.example
└── README.md
```

---

## Database Schema (Core Entities)

| Table                  | Purpose                                                        |
|--------------------------|------------------------------------------------------------------|
| `organizations`          | Tenant accounts (top-level customer)                             |
| `users`                  | User accounts with role assignments                              |
| `farms`                  | Farm properties belonging to an organization                     |
| `fields`                 | Individual fields/plots with GeoJSON boundaries                  |
| `crops`                  | Crop cycles planted per field (season, variety, planting date)   |
| `fertilizer_products`    | Product catalog (name, N-P-K composition, unit, vendor)          |
| `inventory_items`        | Stock levels of products per warehouse/site                      |
| `applications`           | Individual fertilizer application events (rate, method, date)    |
| `soil_tests`             | Lab soil test results per field over time                        |
| `compliance_limits`      | Regulatory/agronomic nutrient limits by region/crop               |
| `alerts`                 | Generated alerts (threshold breach, weather risk, reminders)      |
| `reports`                | Generated/exported report metadata and storage references         |
| `audit_logs`             | Immutable record of all create/update/delete actions              |

All tables include `organization_id`, `created_at`, `updated_at`, and `deleted_at`
(soft-delete) columns; PostgreSQL RLS policies enforce tenant isolation at the
database level in addition to application-layer checks.

---

## API Reference

The platform exposes both REST and GraphQL endpoints. Full interactive documentation
is available via Swagger/OpenAPI (`/api/docs`) and GraphQL Playground (`/graphql`)
in non-production environments.

### Sample REST Endpoints

| Method | Endpoint                                | Description                              |
|--------|-------------------------------------------|--------------------------------------------|
| `POST` | `/api/v1/auth/login`                      | Authenticate and receive JWT tokens          |
| `GET`  | `/api/v1/farms`                           | List farms for the current organization      |
| `POST` | `/api/v1/fields`                          | Create a new field with boundary geometry     |
| `GET`  | `/api/v1/fields/:id/applications`         | List all fertilizer applications for a field  |
| `POST` | `/api/v1/applications`                    | Log a new fertilizer application               |
| `POST` | `/api/v1/applications/import`             | Bulk import applications via CSV                |
| `GET`  | `/api/v1/inventory`                       | Get current inventory levels                    |
| `GET`  | `/api/v1/reports/nutrient-balance`        | Generate nutrient balance report                |
| `GET`  | `/api/v1/reports/compliance/:fieldId`     | Generate compliance report for a field           |
| `GET`  | `/api/v1/alerts`                          | List active alerts for the organization          |

### Sample GraphQL Query

```graphql
query FieldUsageSummary($fieldId: ID!, $season: String!) {
  field(id: $fieldId) {
    name
    crop(season: $season) {
      name
      applications {
        product { name npk }
        rateKgPerHectare
        applicationDate
        method
      }
      nutrientBalance {
        nitrogenApplied
        nitrogenRecommended
        complianceStatus
      }
    }
  }
}
```

All authenticated requests require a `Bearer` JWT in the `Authorization` header.
Rate limiting is enforced at 100 requests/minute per API key on standard tiers.

---

## User Roles & Permissions

| Role               | Permissions                                                                |
|---------------------|------------------------------------------------------------------------------|
| **Org Admin**       | Full access: manage users, billing, farms, and all data                      |
| **Agronomist**      | View/edit recommendations, soil tests, compliance reports across all farms    |
| **Farm Manager**    | Manage fields, crops, inventory, and applications for assigned farms          |
| **Field Applicator**| Log applications via mobile app; read-only access to recommendations         |
| **Auditor**         | Read-only access to applications, compliance reports, and audit logs          |

Permissions are enforced via role-based access control (RBAC) at both the API layer
and the database layer (PostgreSQL RLS).

---

## Reporting & Analytics

- **Nutrient Balance Reports** — applied vs. recommended vs. regulatory cap, by field/season.
- **Cost & Usage Trends** — cost-per-acre, cost-per-crop, and multi-year comparisons.
- **Compliance Exports** — audit-ready PDF/CSV exports formatted for common nutrient
  management plan (NMP) templates.
- **Custom Dashboards** — drag-and-drop widgets (built on Recharts) for organization-
  specific KPIs.
- **Scheduled Reports** — automated weekly/monthly report generation delivered by email.

---

## Notifications & Alerts

Delivered via in-app notification center, email (Amazon SES), and optional SMS
(Twilio/SNS):

- Nutrient limit exceeded for a field/season
- Low fertilizer inventory / reorder point reached
- Unfavorable weather forecast for a scheduled application
- Restricted application window (regulatory blackout dates)
- Overdue application logging reminders

---

## Integrations

- **Weather providers:** NOAA, OpenWeatherMap, Tomorrow.io
- **Farm equipment telemetry:** ISO-XML/ISOBUS file import (John Deere, Case IH, Trimble)
- **Accounting/ERP:** QuickBooks, NetSuite export connectors
- **Mapping:** Mapbox GL, GeoJSON/KML/Shapefile import
- **Single Sign-On:** SAML 2.0 and OAuth2 for enterprise customers

---

## Security & Compliance

- TLS 1.2+ enforced end-to-end; HSTS enabled.
- Data encrypted at rest (AES-256 via RDS/S3 encryption) and in transit.
- Role-based access control with row-level tenant isolation.
- Full audit trail of all data mutations (`audit_logs` table, immutable/append-only).
- Regular dependency vulnerability scanning (Dependabot, Snyk).
- Annual third-party penetration testing.
- SOC 2 Type II control alignment; GDPR-compliant data handling for EU customers.
- Automated secrets management via AWS Secrets Manager (no secrets in source control).

---

## Testing

```bash
# Unit tests
pnpm test

# Integration tests (spins up test containers via Testcontainers)
pnpm test:integration

# End-to-end tests (Playwright)
pnpm test:e2e

# Test coverage report
pnpm test:coverage
```

- **Unit tests:** Jest, ≥ 90% coverage enforced in CI on core business logic.
- **Integration tests:** Testcontainers spin up ephemeral Postgres/Redis instances.
- **E2E tests:** Playwright covering critical flows (login, field creation, application
  logging, report generation).
- **Load testing:** k6 scripts under `infra/load-tests/` for API throughput validation.

---

## Deployment

The platform uses a GitOps deployment model:

1. Merges to `main` trigger GitHub Actions CI (lint, test, build, containerize).
2. Docker images are pushed to Amazon ECR with immutable tags (git SHA).
3. ArgoCD watches the `infra/k8s` manifests and syncs changes to the EKS cluster.
4. Blue/green rollout strategy via Argo Rollouts, with automatic rollback on
   failed health checks.

```bash
# Build and run production containers locally
docker compose -f docker-compose.prod.yml up --build

# Deploy infrastructure changes
cd infra/terraform
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

Environments: `development` → `staging` → `production`, each in isolated AWS accounts
under an AWS Organizations structure.

---

## Monitoring & Observability

- **Metrics & dashboards:** Prometheus + Grafana for infrastructure and application metrics.
- **APM & tracing:** Datadog APM for distributed tracing across services.
- **Error tracking:** Sentry for frontend and backend exception monitoring.
- **Logging:** Structured JSON logs shipped to OpenSearch via Fluent Bit.
- **Alerting:** PagerDuty integration for on-call escalation on SLA-impacting incidents.
- **Uptime target:** 99.9% monthly uptime SLA for production tenants.

---

## Backup & Disaster Recovery

- Automated daily RDS snapshots with 30-day retention; point-in-time recovery enabled.
- Cross-region S3 replication for report/document storage.
- Infrastructure fully defined as code (Terraform) enabling full environment rebuild.
- Documented RTO: 4 hours, RPO: 1 hour, validated via quarterly DR drills.

---

## Roadmap

- [ ] AI-based fertilizer rate recommendations using historical yield + soil data
- [ ] Satellite/NDVI imagery overlay for field health visualization
- [ ] Native iOS/Android apps (currently PWA-based)
- [ ] Carbon footprint estimation per application
- [ ] Marketplace for fertilizer procurement with vendor price comparison
- [ ] Expanded regulatory template library (EU Nitrates Directive, state-level US rules)

---

## Contributing

We welcome contributions from the team and approved external partners.

1. Fork the repository and create a feature branch: `git checkout -b feature/my-feature`
2. Follow the code style enforced by ESLint/Prettier (`pnpm lint`).
3. Write or update tests for any behavioral change.
4. Submit a pull request referencing the related issue/ticket.
5. Ensure all CI checks pass before requesting review.

See `CONTRIBUTING.md` for detailed coding standards, commit message conventions
(Conventional Commits), and the release process.

---

## Support

- **Documentation:** available under `/docs` and the in-app Help Center.
- **Issue tracking:** GitHub Issues for bugs and feature requests.
- **Enterprise support:** dedicated Slack Connect channel and 24/5 support SLA
  for enterprise-tier customers.
- **Contact:** support@agritrackpro.io

---

## License

This project is proprietary software licensed for use by authorized customers and
partners of AgriTrack Pro. See `LICENSE` for full terms. (Swap for MIT/Apache-2.0
if open-sourcing a component.)

---

<p align="center">Built with 🌾 for a more sustainable and data-driven agriculture.</p>
