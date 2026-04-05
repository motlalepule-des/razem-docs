---
layout: default
title: Client Credentials
parent: API Reference
nav_order: 3
---

# Client Credentials
{: .no_toc }

How third-party clients and backend services authenticate with the Razem API.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The **Client Credentials** grant is the OAuth 2.0 flow for machine-to-machine (M2M) authentication. It is intended for:

- Third-party integrations accessing the Razem API on their own behalf (not on behalf of a user)
- Backend services calling the Razem API server-to-server
- Automated jobs and scheduled processes

In this flow there is no user login — the client authenticates directly using its own `client_id` and `client_secret` issued by Razem.

---

## Prerequisites

Before using the Client Credentials flow you need:

1. A registered Razem API client with `client_credentials` grant type enabled
2. A `client_id` and `client_secret` — obtained from the [Razem Developer Portal](https://developers.razem.co.za)
3. The specific API scopes your integration requires

> Keep your `client_secret` confidential. Never embed it in client-side code, mobile apps, or public repositories.

---

## Endpoint

```
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

---

## Obtaining an Access Token

| Parameter       | Value                                         |
|:----------------|:----------------------------------------------|
| `grant_type`    | `client_credentials`                          |
| `client_id`     | Your client ID                                |
| `client_secret` | Your client secret                            |
| `scope`         | Space-separated list of requested scopes      |

**Example:**

```bash
curl -X POST https://api.razem.co.za/connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=your_client_id" \
  -d "client_secret=your_client_secret" \
  -d "scope=razem_api"
```

**Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "razem_api"
}
```

> Client Credentials tokens do **not** include a `refresh_token`. Request a new token when the current one expires.

---

## Using the Access Token

Include the access token as a Bearer token in the `Authorization` header of every API request:

```bash
curl -X GET https://api.razem.co.za/v1/payments \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Available Scopes

Request only the scopes your integration needs (principle of least privilege).

| Scope                  | Description                                           |
|:-----------------------|:------------------------------------------------------|
| `razem_api`            | Standard Razem API access (payments, accounts, etc.)  |
| `razem_developer_api`  | Developer platform API access                         |
| `razem_admin_api`      | Administrative API access (restricted; requires approval) |

---

## Token Introspection

To validate a token (e.g. in a resource server or middleware), use the introspection endpoint:

```
POST /connect/introspect
Content-Type: application/x-www-form-urlencoded
```

| Parameter       | Value                                         |
|:----------------|:----------------------------------------------|
| `token`         | The access token to validate                  |
| `client_id`     | Your introspection client ID                  |
| `client_secret` | Your introspection client secret              |

**Example:**

```bash
curl -X POST https://api.razem.co.za/connect/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d "client_id=razem-api-introspection" \
  -d "client_secret=your_introspection_secret"
```

**Response:**

```json
{
  "active": true,
  "sub": "client_id_here",
  "scope": "razem_api",
  "exp": 1712345678
}
```

| Field    | Description                                           |
|:---------|:------------------------------------------------------|
| `active` | `true` if the token is valid and not expired          |
| `sub`    | Subject — the client ID that owns the token           |
| `scope`  | Scopes granted to the token                           |
| `exp`    | Expiry as a Unix timestamp                            |

If `active` is `false`, the token is invalid or expired and the request must be rejected.

---

## Token Expiry and Rotation

- Access tokens expire after **1 hour** (`expires_in: 3600`)
- There are no refresh tokens in the client credentials flow
- Request a new access token before or shortly after the current one expires
- Cache tokens and reuse them across requests until they are near expiry — do not request a new token per API call

**Recommended token lifecycle:**

```
1. Request token → cache with expiry timestamp
2. Before each API call → check if token expires within 60 seconds
3. If near expiry → request a new token
4. If API returns 401 → request a new token and retry once
```

---

## Rate Limiting

Token endpoint requests are rate limited to **10 requests per minute** per client. Cache and reuse tokens rather than requesting one per API call.

---

## Security Best Practices

- **Never** embed `client_secret` in frontend code, mobile apps, or version control
- Store secrets in environment variables or a secrets manager (e.g. AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)
- Use the minimum required scopes for each client
- Rotate client secrets periodically and immediately if compromised
- Monitor client usage in the [Razem Developer Portal](https://developers.razem.co.za) for unusual activity
- Use HTTPS for all API calls — never send tokens over plaintext HTTP

---

## Error Responses

| HTTP Status | Error                   | Description                                       |
|:------------|:------------------------|:--------------------------------------------------|
| `400`       | `invalid_client`        | Unknown client ID or incorrect client secret      |
| `400`       | `invalid_scope`         | One or more requested scopes are not permitted    |
| `400`       | `unsupported_grant_type`| Client not authorised for client credentials flow |
| `401`       | `invalid_token`         | Access token missing, invalid, or expired         |
| `429`       | —                       | Rate limit exceeded on the token endpoint         |
