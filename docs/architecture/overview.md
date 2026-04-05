---
layout: default
title: System Architecture
parent: Architecture
nav_order: 1
---

# System Architecture
{: .no_toc }

An overview of the Razem platform architecture, components, and design principles.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Platform Overview

Razem is built as a modern cloud-native platform following microservices principles. The architecture is designed for:

- **High Availability** — 99.9% uptime SLA
- **Scalability** — Horizontal scaling to handle traffic spikes
- **Security** — Defence-in-depth security model
- **Compliance** — POPIA, FICA, and PCI-DSS compliance
- **Observability** — Full distributed tracing, metrics, and logging

---

## Core Components

```
┌──────────────────────────────────────────────────────────────────┐
│                         Clients                                  │
│         Web Browser   Mobile App   Third-party Apps             │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                      API Gateway / CDN                           │
│              Rate Limiting, Auth, SSL Termination                │
└───────────┬───────────────────────────────────┬─────────────────┘
            │                                   │
┌───────────▼───────────┐           ┌───────────▼───────────┐
│   My Razem Account    │           │      Razem API         │
│   (my-razem-account)  │           │     (razem-api)        │
│   React/Next.js SPA   │           │   REST API Backend     │
└───────────────────────┘           └───────────┬────────┘
                                                │
                         ┌──────────────────────┼──────────────────┐
                         │                      │                  │
              ┌──────────▼───┐     ┌──────────▼────┐  ┌─────────▼────────┐
              │   Database   │     │   Message Queue  │  │  External APIs   │
              │  PostgreSQL  │     │   (async jobs)   │  │  (Banks, SARS)   │
              └──────────────┘     └─────────────────┘  └──────────────────┘
```

---

## razem-api

The backend API is the heart of the Razem platform.

### Responsibilities
- Authentication and authorization
- Business logic for payments, accounts, beneficiaries
- Integration with banking rails and payment processors
- Webhook delivery
- Rate limiting and request validation

### Technology Stack
- **Runtime:** Node.js / Python (FastAPI)
- **Database:** PostgreSQL with read replicas
- **Cache:** Redis for sessions and rate limiting
- **Queue:** Message broker for async payment processing
- **Search:** Transaction search and filtering

### API Design Principles
- RESTful resource-based routing
- Consistent JSON responses with `data` and `meta` envelope
- Idempotent payment operations using `X-Request-ID`
- Pagination for all list endpoints
- Comprehensive error responses with actionable messages

---

## my-razem-account

The customer web portal is a single-page application.

### Responsibilities
- User authentication and session management
- Account dashboard and real-time balance updates
- Payment initiation and management
- Beneficiary management
- Document upload for KYC
- API key management for developers

### Technology Stack
- **Framework:** React / Next.js
- **State Management:** Context API / Redux
- **Styling:** Tailwind CSS
- **Authentication:** JWT tokens stored in httpOnly cookies
- **Real-time:** WebSocket for live balance and notification updates

---

## Data Architecture

### Database Design Principles

- **Immutable transactions** — Transactions are append-only; no updates or deletes
- **Event sourcing** — Account balance is derived from the transaction ledger
- **Soft deletes** — Records are never hard deleted; status flags are used
- **Audit logging** — All changes are logged with timestamps and user context

### Data Residency

All Razem data is stored in South Africa in compliance with POPIA requirements:
- Primary: Cape Town data centre
- Disaster recovery: Johannesburg data centre
- Backups: Encrypted, geographically distributed

---

## Security Architecture

### Authentication Flow

```
Client → API Gateway → JWT Validation → Rate Limit Check → Business Logic
```

### Security Layers

1. **Edge security** — DDoS protection, WAF (Web Application Firewall)
2. **Transport** — TLS 1.3 for all connections
3. **Application** — JWT authentication, RBAC authorization
4. **Data** — AES-256 encryption at rest, column-level encryption for PII
5. **Network** — Private VPC, no direct database access from internet
6. **Compliance** — PCI-DSS for card data, POPIA for personal data

---

## Observability

### Monitoring Stack

- **Metrics:** Application performance, error rates, latency percentiles
- **Logging:** Structured JSON logs, centralized log aggregation
- **Tracing:** Distributed tracing across all service boundaries
- **Alerting:** PagerDuty integration for on-call incident management
- **Dashboards:** Real-time operational dashboards

### Key SLIs / SLOs

| Metric | Target | Alert Threshold |
|:---|:---|:---|
| API availability | 99.9% | < 99.5% |
| p99 API latency | < 500ms | > 1000ms |
| Payment success rate | > 99.5% | < 98% |
| Webhook delivery rate | > 99% | < 97% |
