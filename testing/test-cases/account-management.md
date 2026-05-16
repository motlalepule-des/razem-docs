# Test Cases — Account Management

Covers the My Razem Account portal: profile, security settings, organizations,
invitations, connected apps, and permission-gated access.

Related docs: [Account Setup](../../docs/account/setup.md),
[Account Security](../../docs/account/security.md).
Related inventory: [my-razem-account](../inventory/my-razem-account.md),
[razem-admin-portal](../inventory/razem-admin-portal.md).

---

### TC-ACCT-001 — View and update profile details

| Field | Value |
|:---|:---|
| **Area** | Profile |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/profile/services/profile.service.test.ts` |

**Preconditions** — an authenticated user.

**Steps**
1. Open the profile page.
2. Update an editable field and save.

**Expected result** — the profile loads current values and the update persists.

---

### TC-ACCT-002 — Change password from security settings

| Field | Value |
|:---|:---|
| **Area** | Security |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/security/services/security.service.test.ts` |

**Preconditions** — an authenticated user.

**Steps**
1. Open security settings.
2. Enter the current password and a new password.
3. Save.

**Expected result** — the password is changed; an incorrect current password is
rejected.

---

### TC-ACCT-003 — View and revoke active sessions

| Field | Value |
|:---|:---|
| **Area** | Security |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/security/services/security.service.test.ts` |

**Preconditions** — an authenticated user with active sessions.

**Steps**
1. Open the sessions list in security settings.
2. Revoke a session.

**Expected result** — the session list is shown and the revoked session is
invalidated.

---

### TC-ACCT-004 — App configuration loads before the portal renders

| Field | Value |
|:---|:---|
| **Area** | App bootstrap |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/app/providers/AppConfigProvider.test.tsx`, `src/services/config.service.test.ts` |

**Preconditions** — none.

**Steps**
1. Load the portal.

**Expected result** — the config provider resolves loading, error, and ready
states correctly; the portal renders only after config is available.

---

### TC-ACCT-005 — Manage organization membership

| Field | Value |
|:---|:---|
| **Area** | Organizations |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/organizations/services/organizations.service.test.ts` |

**Preconditions** — an authenticated user belonging to at least one
organization.

**Steps**
1. Open the organizations page.
2. View members and perform a management action.

**Expected result** — organization data loads and management actions persist.

---

### TC-ACCT-006 — Accept or decline an invitation

| Field | Value |
|:---|:---|
| **Area** | Invitations |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/invitations/services/invitations.service.test.ts` |

**Preconditions** — a pending invitation addressed to the user.

**Steps**
1. Open the invitation.
2. Accept (or decline) it.

**Expected result** — accepting grants the associated membership; declining
removes the invitation. An expired/invalid invitation is rejected.

---

### TC-ACCT-007 — Manage connected apps

| Field | Value |
|:---|:---|
| **Area** | Connected apps |
| **Priority** | Low |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/apps/services/apps.service.test.ts` |

**Preconditions** — an authenticated user.

**Steps**
1. Open the connected-apps page.
2. Revoke an app's access.

**Expected result** — the app list loads and revoking removes the app's access.

---

### TC-ACCT-008 — Protected routes declare a required permission

| Field | Value |
|:---|:---|
| **Area** | Permissions |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-admin-portal` — `src/app/components/AnimatedRoutes.permission-contract.test.ts` |

**Preconditions** — none (static contract check).

**Steps**
1. Enumerate the route table.

**Expected result** — every protected route declares a `requiredPermission`;
the test fails if a protected route is added without one.

---

### TC-ACCT-009 — Permission gating happens at the route layer

| Field | Value |
|:---|:---|
| **Area** | Permissions |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-admin-portal` — `src/test/permissions-architecture.test.ts` |

**Preconditions** — none (static architecture check).

**Steps**
1. Scan feature page entry files.

**Expected result** — no page entry file uses a page-level deny guard;
permission enforcement is centralised at the route layer.

---

### TC-ACCT-010 — Scoped permission policy encoding

| Field | Value |
|:---|:---|
| **Area** | Permissions |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Razem.AspNetCore.Tests/ScopedPermissionPolicyEncodingTests.cs` |

**Preconditions** — none.

**Steps**
1. Encode and decode a scoped permission policy.

**Expected result** — the policy round-trips correctly and scope information is
preserved.
