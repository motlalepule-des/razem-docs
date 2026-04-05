---
layout: default
title: Changelog
parent: Reference
grand_parent: Developer Documentation
nav_order: 2
---

# Changelog

Track API changes, new features, and breaking changes across Razem API versions.

---

{: .note }
For a non-technical summary of recent updates, see [What's New](/razem-docs/docs/account/whats-new).

## API v1 (Current)

### 2025-04-05 — Initial Platform Release

**New Features:**
- Account information and balance retrieval
- EFT payment initiation and management
- Real-Time Payment (RTP) support
- Beneficiary management (CRUD operations)
- Transaction history with filtering and pagination
- Account statements (PDF/CSV download)
- Webhook event subscriptions
- Sandbox environment for development

**Authentication:**
- JWT Bearer token authentication
- API key + secret token exchange
- Token refresh flow
- Granular permission scopes

---

## Upcoming

{: .note }
The following features are in development and will be released in upcoming versions.

- **Batch payments** — Process multiple payments in a single API call
- **Recurring payments** — Schedule automated recurring payments via API
- **Multi-currency** — Support for USD and EUR transactions
- **Business accounts API** — Sub-accounts and team member management
- **Open Banking** — Third-party account linking
- **Payment links** — Generate shareable payment request links
- **QR Code payments** — Generate and scan QR codes for instant payments

---

## Deprecation Policy

Razem maintains at least 12 months of backward compatibility for all API versions. Before deprecating any API feature:

1. Announcement in the developer newsletter (3 months before)
2. Deprecation notice in API responses (`X-Razem-Deprecation` header)
3. Documentation updated with migration guide
4. Deprecation date confirmed (6 months after announcement)

Subscribe to developer updates at [developers.razem.co.za](https://developers.razem.co.za).

---
