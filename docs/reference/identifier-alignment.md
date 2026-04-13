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
| `razem-sme-hub` | React / TypeScript | SME tenant front-end |
| `my-razem-account` | React / TypeScript | End-user account portal |

---

## Shared Core Types

All three front-ends consume the same `/api/v1/app/configuration` endpoint and share
these foundational types. **Any change to the API contract must be mirrored in all three.**

### `ApiResponse<T>` — API envelope

```typescript
// Canonical shape — matches razem-api Result<T> serialisation
interface ApiResponse<T> {
  isSuccess: boolean;
  code: string;         // e.g. "Ok", "ValidationFailed", "SYSTEM.UNEXPECTED"
  message?: string;
  value?: T;
  errors: ErrorItem[];
  correlationId?: string;
}
```

> **Fixed (2025-04-13):** `razem-admin-portal` had `code?: number` — corrected to `code: string`.
> The API always returns string result codes; the numeric type was wrong and would silently
> drop error-code comparisons at runtime.

### `ResultCode` enum

Both `razem-admin-portal` and `razem-sme-hub` maintain a `src/constants/result-code.ts`
that mirrors the API's `ResultCode` C# enum:

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

> **Fixed (2025-04-13):** `razem-admin-portal` was **missing** `src/constants/result-code.ts`
> even though `src/types/result.ts` imports `ResultCode` from it — causing a compile error.
> File added to match `razem-sme-hub`.

### `ErrorItem`

```typescript
// Identical across all three front-ends (same SHA in git)
interface ErrorItem {
  code: string;
  message: string;
  field?: string;
  meta?: Record<string, unknown>;
}
```

### `AppConfiguration`

```typescript
interface AppConfiguration {
  currentUser: CurrentUser;
  tenant?: TenantInfo | null;
  roles: string[];           // e.g. ["admin", "SME.Owner"]
  permissions: string[];     // dot-notation strings, e.g. "Users.Read"
  features: Record<string, string>; // feature-flag key → "true" / "false"
  settings: Record<string, string>;
}
```

### `CurrentUser`

```typescript
interface CurrentUser {
  identityUserId: string;  // GUID — maps to IdentityServer subject
  email: string;
  fullName: string | null; // null when profile is incomplete
}
```

> **Fixed (2025-04-13):** `razem-admin-portal` had `fullName?: string | null` (doubly-optional).
> Aligned to `string | null` — the API always emits the field; it is nullable, not absent.

### `TenantInfo`

```typescript
interface TenantInfo {
  tenantId: string;      // GUID
  tenancyName: string;   // URL-safe slug
  displayName: string;
}
```

### `CurrentTenant`

```typescript
interface CurrentTenant {
  id: string | null;   // GUID — null when no tenant context is active
  name: string | null;
  isAvailable: boolean;
}
```

> **Fixed (2025-04-13):** `razem-sme-hub` had `id: number | null`. Tenant IDs are GUIDs
> (strings) in the API domain model (`SeedIds.cs`, entity keys). Changed to `string | null`
> to match `razem-admin-portal` and the API.

### `RegisterUserResponse`

```typescript
interface RegisterUserResponse {
  appUserId: string;        // platform user ID (GUID)
  identityUserId: string;   // IdentityServer user ID (GUID)
  email: string;
  userName: string;
  tenantLinked: boolean;
  tenantId: string | null;  // GUID of auto-created tenant, or null
}
```

---

## API Endpoint Versioning

All backend routes are under the `/api/v1/` version prefix. Front-ends **must** include
this prefix in every endpoint constant.

### Versioned platform endpoints (shared across front-ends)

| Endpoint | Correct path |
|:---|:---|
| App configuration | `/api/v1/app/configuration` |
| Current user profile | `/api/v1/platform/users/me` |
| User's tenant list | `/api/v1/platform/users/me/tenants` |
| User by ID | `/api/v1/platform/users/{id}` |
| User by identity ID | `/api/v1/platform/users/by-identity/{identityUserId}` |
| Tenant memberships list | `/api/v1/platform/tenants/{tenantId}/memberships` |
| Membership by ID | `/api/v1/platform/tenants/{tenantId}/memberships/{membershipId}` |
| Add role to membership | `/api/v1/platform/tenants/{tenantId}/memberships/{membershipId}/roles` |
| Remove role from membership | `/api/v1/platform/tenants/{tenantId}/memberships/{membershipId}/roles/{roleName}` |
| Send invite | `/api/v1/platform/tenants/{tenantId}/invites` |
| Accept invite | `/api/v1/platform/invites/accept` |
| Active subscription | `/api/v1/tenants/{tenantId}/subscriptions/active` |
| Create subscription | `/api/v1/tenants/{tenantId}/subscriptions` |
| Activate subscription | `/api/v1/tenants/{tenantId}/subscriptions/{id}/activate` |
| Suspend subscription | `/api/v1/tenants/{tenantId}/subscriptions/{id}/suspend` |
| Cancel subscription | `/api/v1/tenants/{tenantId}/subscriptions/{id}/cancel` |
| User registration | `/api/v1/identity/register` |
| Send email OTP | `/api/v1/identity/otp/send` |
| Verify email OTP | `/api/v1/identity/otp/verify` |
| Change password | `/api/v1/identity/password/change` |
| User app access list | `/api/v1/access/my-apps` |

> **Fixed (2025-04-13):** `razem-admin-portal` was calling all platform, membership,
> invite, and subscription endpoints **without** the `/api/v1/` prefix — requests were
> resolving to 404 or hitting the wrong base URL.

### OIDC/Connect endpoints (no version prefix — these are standard OIDC)

| Endpoint | Path |
|:---|:---|
| Token | `/connect/token` |
| Logout | `/connect/logout` |
| Authorize | `/connect/authorize` |
| UserInfo | `/connect/userinfo` |
| Login (custom) | `/connect/login` |

---

## `ENDPOINTS.SUBSCRIPTION` Naming

The subscription endpoint group key must be singular (`SUBSCRIPTION`) across all front-ends.

```typescript
// Correct — singular
ENDPOINTS.SUBSCRIPTION.ACTIVE(tenantId)
ENDPOINTS.SUBSCRIPTION.CANCEL(tenantId, id)
```

> **Fixed (2025-04-13):** `razem-admin-portal` used `ENDPOINTS.SUBSCRIPTIONS` (plural).
> Renamed to `ENDPOINTS.SUBSCRIPTION` to match `razem-sme-hub`.

---

## Feature Flag Constants

Feature flags use a dot-notation key that is **intentionally different** per application
because they map to distinct flag namespaces in the API:

| Repo | `BASE_FEATURE_KEY` | Scope |
|:---|:---|:---|
| `razem-admin-portal` | `Razem.Features` | Platform-wide admin flags |
| `razem-sme-hub` | `Razem.Features.Tenant.SmeHub` | SME tenant-scoped flags |

This is **by design** — do not unify these values.

### SME Hub feature flags (tenant-scoped)

| Constant | Full key |
|:---|:---|
| `FEATURES.ENABLED` | `Razem.Features.Tenant.SmeHub.Enabled` |
| `FEATURES.BANKING` | `Razem.Features.Tenant.SmeHub.Banking.Enabled` |
| `FEATURES.WALLET` | `Razem.Features.Tenant.SmeHub.Wallet.Enabled` |
| `FEATURES.COMPLIANCE` | `Razem.Features.Tenant.SmeHub.Compliance.Enabled` |
| `FEATURES.FINANCIAL_HEALTH` | `Razem.Features.Tenant.SmeHub.FinancialHealth.Enabled` |
| `FEATURES.CASHFLOW` | `Razem.Features.Tenant.SmeHub.Cashflow.Enabled` |
| `FEATURES.INVOICES` | `Razem.Features.Tenant.SmeHub.Invoicing.Enabled` |
| `FEATURES.RECONCILIATION` | `Razem.Features.Tenant.SmeHub.Reconciliation.Enabled` |
| `FEATURES.DAILY_CLOSE` | `Razem.Features.Tenant.SmeHub.DailyClose.Enabled` |
| `FEATURES.CLIENTS` | `Razem.Features.Tenant.SmeHub.Clients.Enabled` |
| `FEATURES.SUBSCRIPTION` | `Razem.Features.Tenant.SmeHub.Subscription.Enabled` |

### Admin Portal feature flags (platform-wide)

| Constant | Full key |
|:---|:---|
| `FEATURES.ENABLED` | `Razem.Features.Enabled` |
| `FEATURES.API_MANAGEMENT` | `Razem.Features.ApiManagement.Enabled` |
| `FEATURES.IMPERSONATION` | `Razem.Features.Impersonation.Enabled` |
| `FEATURES.SUBSCRIPTION` | `Razem.Features.Subscription.Enabled` |

---

## Membership / Team / Organization Naming

The underlying API resource is `memberships` in all cases. The three portals label
it differently in the UI — this is intentional:

| Repo | UI label | API resource path |
|:---|:---|:---|
| `my-razem-account` | Organizations | `/api/v1/platform/tenants/{id}/memberships` |
| `razem-admin-portal` | Memberships | `/api/v1/platform/tenants/{id}/memberships` |
| `razem-sme-hub` | Team | `/api/v1/platform/tenants/{id}/memberships` |

When writing shared code or documentation, always use the API term **membership**.

---

## Phone / Mobile Field Naming

The user profile has two different field names depending on context:

| Context | Field name | Notes |
|:---|:---|:---|
| Registration request | `mobileNumber` | Used in `RegisterUserRequest` (my-razem-account) |
| Profile update request | `phoneNumber` | Used in `UpdateProfileRequest` (my-razem-account) |

This is a known inconsistency in the API surface. The registration endpoint (`/api/v1/identity/register`)
accepts `mobileNumber` while the profile update endpoint (`/api/v1/platform/users/me`) uses `phoneNumber`.
Front-ends should use the field name that matches the specific endpoint being called.

---

## Error Response Format

The internal API response envelope differs from the external (Open Banking) API format.
Use the following format for all internal calls:

```json
{
  "isSuccess": false,
  "code": "VALIDATION.FAILED",
  "message": "One or more fields failed validation.",
  "errors": [
    {
      "code": "VALIDATION.REQUIRED",
      "message": "Email is required.",
      "field": "email"
    }
  ],
  "correlationId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

### Common `code` values (string)

| Code | Meaning |
|:---|:---|
| `Ok` | Request succeeded |
| `Created` | Resource created |
| `ValidationFailed` | One or more fields invalid |
| `NotFound` | Resource does not exist |
| `Conflict` | Duplicate or state conflict |
| `Unauthorized` | Token missing or invalid |
| `Forbidden` | Valid token, insufficient permission |
| `DependencyFailure` | External service call failed |
| `UnexpectedError` | Unhandled server error |

---

## Disjoint Summary Table

| # | Identifier | Repo | Was | Fixed To | Status |
|:--|:---|:---|:---|:---|:---|
| 1 | `ApiResponse.code` type | `razem-admin-portal` | `number?` | `string` | **Fixed** |
| 2 | `src/constants/result-code.ts` | `razem-admin-portal` | Missing | Added | **Fixed** |
| 3 | `ENDPOINTS.CONFIG` | `razem-admin-portal` | `/app/configuration` | `/api/v1/app/configuration` | **Fixed** |
| 4 | `ENDPOINTS.USERS.*` | `razem-admin-portal` | `/platform/users/...` | `/api/v1/platform/users/...` | **Fixed** |
| 5 | `ENDPOINTS.MEMBERSHIPS.*` | `razem-admin-portal` | `/platform/tenants/...` | `/api/v1/platform/tenants/...` | **Fixed** |
| 6 | `ENDPOINTS.INVITES.*` | `razem-admin-portal` | `/platform/invites/...` | `/api/v1/platform/invites/...` | **Fixed** |
| 7 | `ENDPOINTS.SUBSCRIPTIONS` | `razem-admin-portal` | `SUBSCRIPTIONS` (plural) | `SUBSCRIPTION` (singular) | **Fixed** |
| 8 | `ENDPOINTS.SUBSCRIPTION.*` | `razem-admin-portal` | `/tenants/...` | `/api/v1/tenants/...` | **Fixed** |
| 9 | `CurrentTenant.id` type | `razem-sme-hub` | `number \| null` | `string \| null` | **Fixed** |
| 10 | `CurrentUser.fullName` type | `razem-admin-portal` | `string \| null \| undefined` | `string \| null` | **Fixed** |
| 11 | Feature flag base key | `admin-portal` vs `sme-hub` | — | — | By design |
| 12 | Route structure (flat vs nested) | `admin-portal` vs `sme-hub` | — | — | By design |
| 13 | UI term for memberships | All portals | — | — | By design |
| 14 | `mobileNumber` vs `phoneNumber` | `my-razem-account` | — | — | API surface inconsistency |

---

## Checklist: Adding a New Shared Endpoint

When a new API route is added to `razem-api`, follow this checklist before merging:

- [ ] Route path starts with `/api/v1/`
- [ ] Path added to `ENDPOINTS` in every front-end that needs it
- [ ] TypeScript request/response types match DTO field names exactly (case-sensitive)
- [ ] `code` comparisons use `ResultCode` enum values (strings), not numbers
- [ ] Any new error codes are added to `ResultCode` enum in all three front-ends
- [ ] Documentation updated in `razem-docs/docs/reference/identifier-alignment.md`
