---
layout: default
title: Deployment Guide
parent: Architecture
grand_parent: Developer Documentation
nav_order: 2
---

# Deployment Guide
{: .no_toc }

Infrastructure setup and deployment processes for Razem services.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Environments

Razem runs three environments:

| Environment | Purpose | Data |
|:---|:---|:---|
| **Development** | Local developer machines | Synthetic test data |
| **Staging** | Pre-production testing | Anonymised production data |
| **Production** | Live system | Real customer data |

---

## Local Development Setup

### Prerequisites

- Node.js 20+ (or Python 3.11+ depending on service)
- Docker and Docker Compose
- PostgreSQL 15+
- Redis 7+
- Git

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/motlalepule-des/razem-api.git
cd razem-api

# Copy environment configuration
cp .env.example .env

# Start dependencies
docker compose up -d postgres redis

# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Seed development data
npm run db:seed

# Start the development server
npm run dev
```

The API will be available at `http://localhost:3000`.

---

## Environment Variables

### razem-api

| Variable | Description | Required |
|:---|:---|:---|
| `DATABASE_URL` | PostgreSQL connection string | Yes |
| `REDIS_URL` | Redis connection string | Yes |
| `JWT_SECRET` | Secret for signing JWTs | Yes |
| `API_PORT` | Port to run on (default: 3000) | No |
| `LOG_LEVEL` | Logging level (debug/info/warn/error) | No |
| `WEBHOOK_SECRET` | Default webhook signing secret | Yes |
| `BANK_API_KEY` | Integration key for banking rails | Yes (prod) |
| `SENTRY_DSN` | Error tracking DSN | No |

### my-razem-account

| Variable | Description | Required |
|:---|:---|:---|
| `NEXT_PUBLIC_API_URL` | Razem API base URL | Yes |
| `NEXTAUTH_SECRET` | NextAuth.js secret | Yes |
| `NEXTAUTH_URL` | Base URL of the portal | Yes |
| `NEXT_PUBLIC_GA_ID` | Google Analytics ID | No |

---

## CI/CD Pipeline

Razem uses GitHub Actions for continuous integration and deployment.

### Pipeline Stages

```yaml
# Triggered on push to main/develop branches and pull requests
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    # Run unit and integration tests
    # Lint code (ESLint, Prettier)
    # Type checking (TypeScript)
    
  security:
    # Dependency vulnerability scanning
    # SAST (Static Application Security Testing)
    # Secrets scanning
    
  build:
    # Build Docker image
    # Tag with commit SHA and semver tag
    
  deploy-staging:
    # Deploy to staging environment
    # Run smoke tests
    
  deploy-production:
    # Deploy to production (manual approval required)
    # Zero-downtime rolling deployment
    # Automated rollback on health check failure
```

### Deployment Strategy

Razem uses **blue-green deployments** for zero-downtime releases:

1. New version deployed to "blue" environment
2. Health checks verify the new version
3. Traffic gradually shifted from "green" to "blue"
4. Old "green" environment kept for instant rollback
5. If health checks pass for 10 minutes, "green" is decommissioned

---

## Database Migrations

Database schema changes follow a strict process:

```bash
# Create a new migration
npm run db:migration:create -- --name add_payment_notes_column

# Run pending migrations
npm run db:migrate

# Rollback last migration
npm run db:migrate:rollback

# Check migration status
npm run db:migrate:status
```

### Migration Guidelines

- All migrations must be reversible
- Never drop columns (mark as deprecated, remove in a future release)
- Add indexes concurrently on large tables
- Test migrations against a production-sized dataset in staging first

---

## Health Checks

All services expose health check endpoints:

```
GET /health          # Returns 200 if the service is healthy
GET /health/ready    # Returns 200 when ready to serve traffic
GET /health/live     # Returns 200 if the process is alive
```

Response format:
```json
{
  "status": "healthy",
  "version": "1.2.3",
  "database": "connected",
  "redis": "connected",
  "timestamp": "2025-04-05T10:30:00Z"
}
```
