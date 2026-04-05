---
layout: default
title: Authentication
parent: API Reference
nav_order: 1
---

# Authentication
{: .no_toc }

Razem uses Bearer token authentication for all API requests.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The Razem API uses JWT (JSON Web Token) based authentication. To authenticate:

1. Generate an API Key and Secret from the My Razem Account portal
2. Exchange them for an access token
3. Include the access token in all API requests

---

## Obtaining an Access Token

### Request

```
POST /v1/auth/token
```

```json
{
  "api_key": "rz_live_abcdefghijklmnop",
  "api_secret": "sk_live_1234567890abcdef"
}
```

### Response

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhY2NfMTIzNDU2Nzg5MCIsImV4cCI6MTcxMjMxMjM0NSwiaWF0IjoxNzEyMzA4NzQ1fQ.signature",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rt_1234567890abcdefghijklmnop"
}
```

---

## Using the Access Token

Include the access token in the `Authorization` header of every request:

```bash
curl -X GET https://api.razem.co.za/v1/account \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Refreshing the Access Token

Access tokens expire after 1 hour. Use the refresh token to get a new access token without re-entering credentials:

```
POST /v1/auth/refresh
```

```json
{
  "refresh_token": "rt_1234567890abcdefghijklmnop"
}
```

---

## Revoking Tokens

To immediately invalidate an access token:

```
POST /v1/auth/revoke
```

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## API Key Types

| Key Type | Prefix | Usage |
|:---|:---|:---|
| Live API Key | `rz_live_` | Production environment |
| Sandbox API Key | `rz_sandbox_` | Testing and development |
| Restricted Key | `rz_restricted_` | Limited permission scopes |

---

## Permissions & Scopes

API keys can be restricted to specific scopes:

| Scope | Description |
|:---|:---|
| `account:read` | Read account information and balance |
| `transactions:read` | Read transaction history |
| `payments:write` | Initiate payments |
| `beneficiaries:read` | Read beneficiaries |
| `beneficiaries:write` | Add/modify beneficiaries |
| `statements:read` | Download account statements |
| `webhooks:write` | Manage webhooks |
| `admin` | Full access (use with caution) |

---

## Security Best Practices

{: .warning }
**Never expose your API keys in client-side code, public repositories, or logs.**

- Store API keys in environment variables or a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault)
- Use the minimum required scopes for each integration
- Rotate API keys periodically and immediately if compromised
- Use webhook signing secrets to verify webhook authenticity
- Monitor API key usage for unusual activity in the My Razem Account portal
