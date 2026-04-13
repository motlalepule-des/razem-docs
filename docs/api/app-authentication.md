---
layout: default
title: App Authentication
parent: API Reference
nav_order: 2
---

# App Authentication
{: .no_toc }

How Razem apps authenticate users through My Razem Account.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Razem apps (portals, SPAs) authenticate users via the **My Razem Account** service, which acts as the central OAuth 2.0 / OpenID Connect identity provider. The server is powered by [OpenIddict](https://documentation.openiddict.com/).

Apps never handle passwords directly — they exchange user credentials for short-lived access tokens and long-lived refresh tokens at the `/connect/token` endpoint.

---

## Endpoints

| Endpoint              | Method      | Purpose                          |
|:----------------------|:------------|:---------------------------------|
| `POST /connect/token` | `POST`      | Obtain or refresh tokens         |
| `GET /connect/userinfo` | `GET`     | Retrieve authenticated user info |
| `GET /connect/authorize` | `GET`   | Start authorization code flow    |

---

## Authentication Flow

### Password Grant (SPA / First-Party Apps)

Used by first-party Razem portals (e.g. My Razem Account, Admin Portal, Business) where the app is trusted to collect user credentials directly.

#### 1. Request an Access Token

```
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

| Parameter      | Value                                          |
|:---------------|:-----------------------------------------------|
| `grant_type`   | `password`                                     |
| `client_id`    | Your registered client ID (e.g. `razem-account-spa`) |
| `username`     | User's email address                           |
| `password`     | User's password                                |
| `scope`        | `openid profile email offline_access razem_api` |

**Example:**

```bash
curl -X POST https://api.razem.co.za/connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=razem-account-spa" \
  -d "username=user@example.com" \
  -d "password=s3cur3P@ssw0rd" \
  -d "scope=openid%20profile%20email%20offline_access%20razem_api"
```

#### 2. Token Response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rt_1234567890abcdefghijklmnop",
  "scope": "openid profile email offline_access razem_api"
}
```

| Field           | Description                                      |
|:----------------|:-------------------------------------------------|
| `access_token`  | JWT to include in API requests. Expires in 1 hour. |
| `token_type`    | Always `Bearer`                                  |
| `expires_in`    | Seconds until access token expires (3600 = 1 hr) |
| `refresh_token` | Opaque token to obtain new access tokens. Valid for 30 days. |

#### 3. Use the Access Token

Include the access token in the `Authorization` header of every API request:

```bash
curl -X GET https://api.razem.co.za/v1/account \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Refreshing Tokens

Access tokens expire after **1 hour**. Use the refresh token to obtain a new access token without prompting the user for credentials again.

```
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

| Parameter       | Value                        |
|:----------------|:-----------------------------|
| `grant_type`    | `refresh_token`              |
| `client_id`     | Your registered client ID    |
| `refresh_token` | The refresh token from the original login response |

**Example:**

```bash
curl -X POST https://api.razem.co.za/connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "client_id=razem-account-spa" \
  -d "refresh_token=rt_1234567890abcdefghijklmnop"
```

**Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rt_newtoken_abcdefghijklmnop"
}
```

> Permissions are re-evaluated on every token refresh. If a user's permissions have been revoked, the new access token will immediately reflect the change.

---

## Token Claims

Access tokens issued to app users carry the following claims:

| Claim          | Description                                      |
|:---------------|:-------------------------------------------------|
| `sub`          | Unique user identifier                           |
| `email`        | User's email address                             |
| `name`         | User's display name                              |
| `system_role`  | System-level roles (e.g. `admin`, `support`)     |
| `tenant_id`    | Active tenant (organisation) identifier          |
| `app_user_id`  | Tenant-scoped user identifier                    |
| `tenant_role`  | Roles within the active tenant                   |
| `permission`   | Granular permission strings (e.g. `payments:write`) |

---

## Token Storage

Store tokens securely in the client:

- **Access token**: `razem_access_token` in `localStorage`
- **Refresh token**: `razem_refresh_token` in `localStorage`

> **Security note:** `localStorage` is accessible to JavaScript running on the page. Ensure your app implements a strict Content Security Policy (CSP) and avoids loading untrusted third-party scripts.

---

## Authorization Code Flow (PKCE)

For scenarios where a first-party app needs to delegate authentication to a browser (e.g. opening a login page), use the Authorization Code flow with PKCE.

#### 1. Redirect the user to the authorization endpoint

```
GET /connect/authorize
  ?client_id=razem-account-spa
  &response_type=code
  &redirect_uri=https%3A%2F%2Fapp.razem.co.za%2Fcallback
  &scope=openid%20profile%20email%20razem_api
  &state=random_state_value
  &code_challenge=BASE64URL(SHA256(code_verifier))
  &code_challenge_method=S256
```

If the user is not authenticated, they will be redirected to the login page.

#### 2. Exchange the code for tokens

After the user authenticates, they are redirected to your `redirect_uri` with a `code` parameter:

```
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

| Parameter       | Value                        |
|:----------------|:-----------------------------|
| `grant_type`    | `authorization_code`         |
| `client_id`     | Your registered client ID    |
| `code`          | The authorization code       |
| `redirect_uri`  | Must match the original redirect URI |
| `code_verifier` | The original PKCE code verifier |

---

## Registered App Clients

| Client ID                  | App                        |
|:---------------------------|:---------------------------|
| `razem-account-spa`        | My Razem Account portal    |
| `razem-admin-portal`       | Admin Portal               |
| `razem-developer-portal`   | Developer Portal           |
| `razem-business-portal`    | Business Portal            |

---

## Rate Limiting

Token endpoint requests are rate limited to **10 requests per minute** per client. Exceeding this returns `HTTP 429 Too Many Requests`.

---

## Error Responses

| HTTP Status | Error                   | Description                                  |
|:------------|:------------------------|:---------------------------------------------|
| `400`       | `invalid_grant`         | Invalid credentials or expired refresh token |
| `400`       | `invalid_client`        | Unknown or unauthorized client ID            |
| `400`       | `unsupported_grant_type`| Grant type not permitted for this client     |
| `401`       | `invalid_token`         | Access token missing, invalid, or expired    |
| `429`       | —                       | Rate limit exceeded                          |
