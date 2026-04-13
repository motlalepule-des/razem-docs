---
layout: default
title: Cross-Repo Identifier Alignment
parent: Reference
nav_order: 3
---

# Cross-Repo Identifier Alignment
{: .no_toc }

This document records all shared identifiers (types, endpoint paths, constants, field names)
across the four Razem repositories and documents where they must stay in sync.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Repositories in Scope

| Repository | Stack | Role |
|:---|:---|:---|
| `razem-api` | .NET / C# | Backend — source of truth for all identifiers |
| `razem-admin-portal` | React / TypeScript | Platform admin front-end |
| `razem-business` | React / TypeScript | Business front-end |
| `my-razem-account` | React / TypeScript | End-user account portal |

---

## Axios Client Setup — The Golden Rule

All three front-ends use two axios instances. The version prefix **must live in
`apiClient.baseURL`**, never in the endpoint path strings.

```typescript
// Correct — all three apps must follow this pattern exactly
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

// Versioned API client — for all /api/v1/* calls
export const apiClient = axios.create({
  baseURL: `${API_BASE_URL}/api/v1`,
  // ...
});

// Auth client — OIDC /connect/* calls only (no version prefix)
export const authClient = axios.create({
  baseURL: API_BASE_URL,
  // ...
});
```

| Client | baseURL | Used for |
|:---|:---|:---|
| `apiClient` | `${API_BASE_URL}/api/v1` | All REST API calls |
| `authClient` | `${API_BASE_URL}` | OIDC `/connect/*` flows only |

> **Fixed (2025-04-13):** `my-razem-account` had `apiClient.baseURL = API_BASE_URL` (no
> version). Updated to `${API_BASE_URL}/api/v1` to match `razem-admin-portal` and
> `razem-business`.

---

## Endpoint Path Convention

Because `apiClient.baseURL` already includes `/api/v1`, **endpoint path strings must
not start with `/api/v1`**.

```typescript
// ✅ Correct
CONFIG: '/app/configuration'
USERS: { ME: '/platform/users/me' }

// ❌ Wrong — results in /api/v1/api/v1/app/configuration
CONFIG: '/api/v1/app/configuration'
```

OIDC paths are the **only exception** — they are called via `authClient` (bare
baseURL) and naturally have no version prefix:

```typescript
// ✅ Correct — authClient, no version needed
AUTH: {
  TOKEN:    '/connect/token',
  LOGOUT:   '/connect/logout',
  USERINFO: '/connect/userinfo',
}
```

> **Fixed (2025-04-13):** `razem-business` had `/api/v1/` embedded in CONFIG, USERS,
> TEAM, PLATFORM, SUBSCRIPTION, and AUTH.REGISTER paths. All stripped.
> `my-razem-account` endpoint paths similarly stripped after its axios.ts was aligned.

---

## Versioned Platform Endpoints (shared across front-ends)

All paths below are relative to `apiClient.baseURL` (i.e. already under `/api/v1`).

| Endpoint group | Path |
|:---|:---|
| App configuration | `/app/configuration` |
| Current user profile | `/platform/users/me` |
| User's tenant list | `/platform/users/me/tenants` |
| User by ID | `/platform/users/{id}` |
| User by identity ID | `/platform/users/by-identity/{identityUserId}` |
| Tenant memberships list | `/platform/tenants/{tenantId}/memberships` |
| Membership by ID | `/platform/tenants/{tenantId}/memberships/{membershipId}` |
| Add role to membership | `/platform/tenants/{tenantId}/memberships/{membershipId}/roles` |
| Remove role from membership | `/platform/tenants/{tenantId}/memberships/{membershipId}/roles/{roleName}` |
| Send invite | `/platform/tenants/{tenantId}/invites` |
| Accept invite | `/platform/invites/accept` |
| Active subscription | `/tenants/{tenantId}/subscriptions/active` |
| Create subscription | `/tenants/{tenantId}/subscriptions` |
| Activate subscription | `/tenants/{tenantId}/subscriptions/{id}/activate` |
| Suspend subscription | `/tenants/{tenantId}/subscriptions/{id}/suspend` |
| Cancel subscription | `/tenants/{tenantId}/subscriptions/{id}/cancel` |
| User registration | `/identity/register` |
| Send email OTP | `/identity/otp/send` |
| Verify email OTP | `/identity/otp/verify` |
| Change password | `/identity/password/change` |
| User app access list | `/access/my-apps` |

---

## `ENDPOINTS.SUBSCRIPTION` Naming

The subscription endpoint group key must be **singular** (`SUBSCRIPTION`) across
all front-ends.

```typescript
// ✅ Correct
ENDPOINTS.SUBSCRIPTION.ACTIVE(tenantId)
ENDPOINTS.SUBSCRIPTION.CANCEL(tenantId, id)

// ❌ Wrong
ENDPOINTS.SUBSCRIPTIONS.ACTIVE(tenantId)
```

> **Fixed (2025-04-13):** `razem-admin-portal` used `ENDPOINTS.SUBSCRIPTIONS` (plural).
> Renamed to `ENDPOINTS.SUBSCRIPTION` to match `razem-business`.

---

## Shared Core Types

All three front-ends consume `/app/configuration` and share these foundational
types. **Any change to the API contract must be mirrored in all three.**

### `ApiResponse<T>` — API envelope

```typescript
interface ApiResponse<T> {
  isSuccess: boolean;
  code: string;         // e.g. "Ok", "ValidationFailed", "NotFound"
  message?: string;
  value?: T;
  errors: ErrorItem[];
  correlationId?: string;
}
```

> **Fixed (2025-04-13):** `razem-admin-portal` had `code?: number`. Corrected to
> `code: string` — the API always returns string result codes.

### `ResultCode` enum

Both `razem-admin-portal` and `razem-business` maintain `src/constants/result-code.ts`
mapping the API's C# `ResultCode` enum:

```typescript
export enum ResultCode {
  Ok = 'Ok',
  Created = 'Created',
  Accepted = 'Accepted',
  ValidationFailed = 'ValidationFailed',
  NotFound = 'NotFound',
  Conflict = 'Conflict',
  Unauthorized = 'Unauthorized',
  Forbidden = 'Forbidden',
  DependencyFailure = 'DependencyFailure',
  ParseFailed = 'ParseFailed',
  DatabaseFailure = 'DatabaseFailure',
  UnexpectedError = 'UnexpectedError',
}
```

> **Fixed (2025-04-13):** `razem-admin-portal` was missing this file even though
> `src/types/result.ts` imports it. File created.

### `ErrorItem`

```typescript
// Identical across all three front-ends
interface ErrorItem {
  code: string;
  message: string;
  field?: string;
  meta?: Record<string, unknown>;
}
```

### `CurrentUser`

```typescript
interface CurrentUser {
  identityUserId: string;
  email: string;
  fullName: string | null; // always present from API; null when profile incomplete
}
```

> **Fixed (2025-04-13):** `razem-admin-portal` had `fullName?: string | null`
> (doubly-optional). Aligned to `string | null`.

### `CurrentTenant`

```typescript
interface CurrentTenant {
  id: string | null;   // GUID — null when no tenant context is active
  name: string | null;
  isAvailable: boolean;
}
```

> **Fixed (2025-04-13):** `razem-business` had `id: number | null`. Tenant IDs are
> GUIDs (strings) in the API domain model. Changed to `string | null`.

### `AppConfiguration`

```typescript
interface AppConfiguration {
  currentUser: CurrentUser;
  tenant?: TenantInfo | null;
  roles: string[];
  permissions: string[];           // dot-notation, e.g. "Users.Read"
  features: Record<string, string>; // feature key → "true" / "false"
  settings: Record<string, string>;
}
```

### `TenantInfo`

```typescript
interface TenantInfo {
  tenantId: string;    // GUID
  tenancyName: string; // URL-safe slug
  displayName: string;
}
```

### `RegisterUserResponse`

```typescript
interface RegisterUserResponse {
  appUserId: string;       // platform user ID (GUID)
  identityUserId: string;  // IdentityServer user ID (GUID)
  email: string;
  userName: string;
  tenantLinked: boolean;
  tenantId: string | null; // GUID of auto-created tenant, or null
}
```

---

## Feature Flag Constants

Feature flag base keys differ **by design** — they map to distinct flag namespaces:

| Repo | `BASE_FEATURE_KEY` | Scope |
|:---|:---|:---|
| `razem-admin-portal` | `Razem.Features` | Platform-wide admin flags |
| `razem-business` | `Razem.Features.Tenant.Business` | Business tenant-scoped flags |

Do not unify these values.

---

## Membership / Team / Organization UI Naming

The API resource is always `memberships`. The three portals label it differently
in the UI — this is intentional:

| Repo | UI label | API path |
|:---|:---|:---|
| `my-razem-account` | Organizations | `/platform/tenants/{id}/memberships` |
| `razem-admin-portal` | Memberships | `/platform/tenants/{id}/memberships` |
| `razem-business` | Team | `/platform/tenants/{id}/memberships` |

When writing shared code or documentation, always use the API term **membership**.

---

## Phone / Mobile Field Naming

| Context | Field name | Endpoint |
|:---|:---|:---|
| Registration | `mobileNumber` | `/identity/register` |
| Profile update | `phoneNumber` | `/platform/users/me` |

This is a known API surface inconsistency. Use the field name that matches the
specific endpoint being called.

---

## Error Response Format

```json
{
  "isSuccess": false,
  "code": "ValidationFailed",
  "message": "One or more fields failed validation.",
  "errors": [
    { "code": "VALIDATION.REQUIRED", "message": "Email is required.", "field": "email" }
  ],
  "correlationId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

---

## Disjoint Summary Table

| # | Identifier | Repo | Was | Fixed To | Status |
|:--|:---|:---|:---|:---|:---|
| 1 | `ApiResponse.code` type | `razem-admin-portal` | `number?` | `string` | **Fixed** |
| 2 | `src/constants/result-code.ts` | `razem-admin-portal` | Missing | Added | **Fixed** |
| 3 | `apiClient.baseURL` | `my-razem-account` | `API_BASE_URL` | `${API_BASE_URL}/api/v1` | **Fixed** |
| 4 | `/api/v1/` in endpoint paths | `razem-business` | Present in CONFIG, USERS, TEAM, PLATFORM, SUBSCRIPTION, AUTH.REGISTER | Stripped | **Fixed** |
| 5 | `/api/v1/` in endpoint paths | `my-razem-account` | Present in non-OIDC paths | Stripped | **Fixed** |
| 6 | `ENDPOINTS.SUBSCRIPTIONS` | `razem-admin-portal` | `SUBSCRIPTIONS` (plural) | `SUBSCRIPTION` (singular) | **Fixed** |
| 7 | `CurrentTenant.id` type | `razem-business` | `number \| null` | `string \| null` | **Fixed** |
| 8 | `CurrentUser.fullName` type | `razem-admin-portal` | `string \| null \| undefined` | `string \| null` | **Fixed** |
| 9 | Feature flag base key | `admin-portal` vs `business` | — | — | By design |
| 10 | Route structure (flat vs nested) | `admin-portal` vs `business` | — | — | By design |
| 11 | UI term for memberships | All portals | — | — | By design |
| 12 | `mobileNumber` vs `phoneNumber` | `my-razem-account` | — | — | API surface inconsistency |

---

## Checklist: Adding a New Shared Endpoint

- [ ] API route is under `api/v1/` in `razem-api`
- [ ] Endpoint path added to `ENDPOINTS` in relevant front-ends **without** the `/api/v1/` prefix
- [ ] OIDC / connect endpoints use `authClient` (via `useAuthClient: true`); all others use `apiClient`
- [ ] TypeScript request/response types match DTO field names exactly (case-sensitive)
- [ ] `code` comparisons use `ResultCode` enum string values, not numbers
- [ ] New error codes added to `ResultCode` in all three front-ends
- [ ] This document updated in `razem-docs/docs/reference/identifier-alignment.md`
