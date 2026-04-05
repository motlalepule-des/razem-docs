---
layout: default
title: Authentication
parent: API Reference
nav_order: 1
---

# Authentication
{: .no_toc }

Razem uses OAuth 2.0 / OpenID Connect for authentication across all clients.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The Razem API uses **JWT (JSON Web Token)** based authentication backed by [OpenIddict](https://documentation.openiddict.com/). All API requests must include a valid Bearer token in the `Authorization` header.

Two authentication flows are supported depending on the type of client:

| Client Type | Flow | Guide |
|:------------|:-----|:------|
| Razem app or portal (browser / SPA) | Password Grant or Authorization Code + PKCE | [App Authentication](./app-authentication) |
| Third-party service or backend integration | Client Credentials | [Client Credentials](./client-credentials) |

---

## Making Authenticated Requests

Once you have obtained an access token, include it in every API request:

```bash
curl -X GET https://api.razem.co.za/v1/account \
  -H "Authorization: Bearer <your_access_token>"
```

Requests without a valid token will receive `HTTP 401 Unauthorized`.

---

## Token Lifetimes

| Token         | Lifetime  | Notes                                         |
|:--------------|:----------|:----------------------------------------------|
| Access token  | 1 hour    | Include in `Authorization` header             |
| Refresh token | 30 days   | Available in password/authorization code flows only |

---

## API Key Types

Certain Razem integrations use API keys as credentials to obtain tokens:

| Key Type       | Prefix            | Environment         |
|:---------------|:------------------|:--------------------|
| Live API Key   | `rz_live_`        | Production          |
| Sandbox API Key| `rz_sandbox_`     | Testing/development |
| Restricted Key | `rz_restricted_`  | Limited scopes      |

---

## Permissions & Scopes

| Scope                  | Description                                           |
|:-----------------------|:------------------------------------------------------|
| `openid`               | OpenID Connect identity                               |
| `profile`              | User's name and profile information                   |
| `email`                | User's email address                                  |
| `offline_access`       | Issue a refresh token                                 |
| `razem_api`            | Standard Razem API access                             |
| `razem_developer_api`  | Developer platform API access                         |
| `razem_admin_api`      | Administrative API access                             |
| `account:read`         | Read account information and balance                  |
| `transactions:read`    | Read transaction history                              |
| `payments:write`       | Initiate payments                                     |
| `beneficiaries:read`   | Read beneficiaries                                    |
| `beneficiaries:write`  | Add/modify beneficiaries                              |
| `statements:read`      | Download account statements                           |
| `webhooks:write`       | Manage webhooks                                       |

---

## Security Best Practices

> **Never expose credentials (API keys, client secrets, passwords) in client-side code, public repositories, or logs.**

- Store secrets in environment variables or a secrets manager
- Request only the minimum scopes required
- Rotate credentials periodically and immediately if compromised
- Use HTTPS for all API calls
- Monitor usage in the [My Razem Account portal](https://account.razem.co.za) for unusual activity
