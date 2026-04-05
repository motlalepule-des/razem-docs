---
layout: default
title: Developer Documentation
nav_order: 3
has_children: true
permalink: /docs/technical
---

# Developer Documentation

Complete technical reference for integrating with the Razem platform via the REST API.

This section is for **developers, engineers, and technical teams** building integrations with Razem.

{: .note }
Looking to manage your Razem account? See the [Business Guide](/razem-docs/docs/business) for account setup, payments, and security.

---

## Sections

| Section | Description |
|:---|:---|
| [Guides](/razem-docs/docs/guides) | Step-by-step integration guides with code examples |
| [API Reference](/razem-docs/docs/api) | Complete REST API endpoint documentation |
| [Architecture](/razem-docs/docs/architecture) | System design, infrastructure, and deployment |
| [Reference](/razem-docs/docs/reference) | Error codes, changelog, and technical reference |

---

## Quick Start

```bash
# Authenticate and get a session token
curl -X POST https://api.razem.co.za/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "api_secret": "YOUR_API_SECRET"
  }'
```

See the full [Getting Started guide](/razem-docs/docs/guides/getting-started) for prerequisites, SDKs, and next steps.

---

## API Base URLs

| Environment | Base URL |
|:---|:---|
| Production | `https://api.razem.co.za/v1` |
| Sandbox | `https://sandbox.api.razem.co.za/v1` |

All requests require HTTPS. See [Authentication](/razem-docs/docs/api/authentication) for token setup.
