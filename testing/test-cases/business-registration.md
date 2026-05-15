# Test Cases — Business Registration

Covers onboarding a business onto the Razem SME Hub: business profile creation,
branches, plan selection, and quota enforcement.

Related docs: `razem-sme-hub/docs/business-registration.md`,
`razem-api/docs/business-registration.md`.
Related inventory: [razem-api](../inventory/razem-api.md),
[razem-sme-hub](../inventory/razem-sme-hub.md).

---

### TC-BIZREG-001 — Register a new business profile

| Field | Value |
|:---|:---|
| **Area** | Business profile |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None (SME Hub flow has no automated coverage) |

**Preconditions** — an authenticated user with no existing business profile.

**Steps**
1. Open the business registration flow in the SME Hub.
2. Complete the business details form (name, registration number, contact).
3. Submit.

**Expected result** — a business profile is created, the user becomes its owner,
and they are taken to the business dashboard.

---

### TC-BIZREG-002 — Reject registration with missing required fields

| Field | Value |
|:---|:---|
| **Area** | Business profile |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — an authenticated user on the registration form.

**Steps**
1. Leave one or more required fields blank.
2. Attempt to submit.

**Expected result** — submission is blocked and each missing field shows a
validation message.

---

### TC-BIZREG-003 — Tenant provisioned for a new business

| Field | Value |
|:---|:---|
| **Area** | Tenancy |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/PlatformAccess/Domain.Tests/TenantTests.cs`, `Razem.Domain.Tests/TenantProfileTests.cs` |

**Preconditions** — a business registration request is accepted.

**Steps**
1. Complete business registration.
2. Inspect the created tenant and tenant profile.

**Expected result** — a tenant and tenant profile are created with correct
defaults and the business data is scoped to that tenant.

---

### TC-BIZREG-004 — Business data is isolated per tenant

| Field | Value |
|:---|:---|
| **Area** | Tenancy |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Razem.Infrastructure.Tests/GlobalFiltersTests.cs`, `Razem.Infrastructure.Tests/CurrentTenantTests.cs` |

**Preconditions** — at least two business tenants exist.

**Steps**
1. Query business data while acting as tenant A.

**Expected result** — only tenant A's records are returned; tenant B's data is
never visible. Global query filters enforce isolation.

---

### TC-BIZREG-005 — Add a branch to a business profile

| Field | Value |
|:---|:---|
| **Area** | Branches |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Business/Application.Tests/BranchTests.cs` |

**Preconditions** — a registered business profile.

**Steps**
1. Create a branch under the business profile.

**Expected result** — the branch is created and linked to the business profile.

---

### TC-BIZREG-006 — Assign a member to a business profile / branch

| Field | Value |
|:---|:---|
| **Area** | Membership |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/PlatformAccess/Domain.Tests/BusinessProfileMembershipTests.cs`, `Modules/PlatformAccess/Domain.Tests/BranchMembershipTests.cs` |

**Preconditions** — a business profile with at least one branch.

**Steps**
1. Add a user as a member of the business profile and/or a branch.

**Expected result** — the membership is recorded with the correct scope and
role.

---

### TC-BIZREG-007 — Select a subscription plan during onboarding

| Field | Value |
|:---|:---|
| **Area** | Subscription |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/PlatformAccess/Domain.Tests/PlanTests.cs`, `Modules/PlatformAccess/Domain.Tests/SubscriptionSnapshotTests.cs` |

**Preconditions** — a newly registered business.

**Steps**
1. Choose a plan during onboarding.
2. Confirm the selection.

**Expected result** — a subscription is created against the chosen plan with a
correct initial snapshot.

---

### TC-BIZREG-008 — Plan quota blocks actions beyond the limit

| Field | Value |
|:---|:---|
| **Area** | Subscription |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/PlatformAccess/Application.Tests/QuotaCheckerTests.cs` |

**Preconditions** — a business on a plan with a defined quota that is at its
limit.

**Steps**
1. Attempt an action that would exceed the plan quota.

**Expected result** — the action is rejected with a quota-exceeded result.

---

### TC-BIZREG-009 — Closing a business emits a closed event

| Field | Value |
|:---|:---|
| **Area** | Lifecycle |
| **Priority** | Low |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Business/Domain.Tests/BusinessProfileClosedEventTests.cs`, `Modules/Business/Domain.Tests/BranchClosedEventTests.cs` |

**Preconditions** — an active business profile (optionally with branches).

**Steps**
1. Close the business profile (or a branch).

**Expected result** — the corresponding closed domain event is raised so
downstream handlers can react.
