# Test Inventory — `razem-api`

Backend REST API. Tests use **xUnit** and live under `tests/`, with one test
project per production project.

- **Test projects:** 23
- **Test files:** 87
- **Test cases (`[Fact]` + `[Theory]`):** 814

Recount:

```bash
grep -rh "\[Fact\]\|\[Theory\]" --include="*.cs" tests | wc -l
```

## Layout

```
tests/
├── Razem.Shared.Tests/             # cross-cutting helpers & extensions
├── Razem.Domain.Tests/             # base domain types
├── Razem.Application.Tests/        # cross-cutting application services
├── Razem.AspNetCore.Tests/         # web-layer concerns
├── Razem.Infrastructure.Tests/     # persistence & cross-cutting infrastructure
└── Modules/<Module>/{Domain,Application,Infrastructure}.Tests/
```

## Cross-cutting projects (211 cases)

| Project | Files | Cases | Covers |
|:---|---:|---:|:---|
| `Razem.Shared.Tests` | 12 | 103 | String/date/claims/reflection helpers, `Result`, `Check`, AES encryption, `TypeList`, `DisposeAction`, hashing. |
| `Razem.Domain.Tests` | 1 | 17 | `TenantProfile` domain behaviour. |
| `Razem.Application.Tests` | 3 | 21 | `FeatureChecker`, `OtpService`, `ReferenceGenerator`. |
| `Razem.AspNetCore.Tests` | 1 | 10 | Scoped permission policy encoding. |
| `Razem.Infrastructure.Tests` | 6 | 60 | Audit interceptor, current-tenant resolution, data-protection extensions, domain-event dispatch interceptor, global query filters, hierarchy scoping. |

## Module projects (603 cases)

### Identity (35 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Application.Tests/PasswordResetAppServiceTests.cs` | 13 | Password reset request/confirm flow. |
| `Application.Tests/ApplicationModuleTests.cs` | 1 | Module wiring. |
| `Domain.Tests/AppIdentityUserTests.cs` | 8 | User aggregate behaviour. |
| `Domain.Tests/UserSessionTests.cs` | 5 | Session lifecycle. |
| `Domain.Tests/EmailOtpTests.cs` | 3 | Email OTP issuance/validation. |
| `Domain.Tests/AppIdentityRoleTests.cs` | 2 | Role aggregate. |
| `Domain.Tests/RolePermissionTests.cs` | 2 | Role-permission association. |
| `Infrastructure.Tests/IdentityDbContextTests.cs` | 1 | DbContext wiring. |

### PlatformAccess (121 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Application.Tests/QuotaCheckerTests.cs` | 16 | Plan quota enforcement. |
| `Application.Tests/AppUserServiceTests.cs` | 10 | Platform user service. |
| `Application.Tests/SubscriptionUsageRollupServiceTests.cs` | 8 | Usage roll-up. |
| `Application.Tests/CurrentUserAppServiceTests.cs` | 7 | Current-user resolution. |
| `Application.Tests/TenantScopingFlipTests.cs` | 7 | Tenant scoping toggles. |
| `Application.Tests/StorageLimitParserTests.cs` | 7 | Storage-limit parsing. |
| `Application.Tests/BillingPeriodCalculatorTests.cs` | 6 | Billing-period maths. |
| `Application.Tests/InMemoryApiCallCounterTests.cs` | 3 | API-call counter. |
| `Application.Tests/InMemorySubmissionCounterTests.cs` | 3 | Submission counter. |
| `Application.Tests/MembershipRevocationHandlersTests.cs` | 3 | Membership revocation events. |
| `Application.Tests/ApiCallCounterFlushServiceTests.cs` | 2 | API-call counter flush. |
| `Application.Tests/SubmissionCounterFlushServiceTests.cs` | 2 | Submission counter flush. |
| `Domain.Tests/PlanTests.cs` | 13 | Plan aggregate. |
| `Domain.Tests/TenantTests.cs` | 10 | Tenant aggregate. |
| `Domain.Tests/SubscriptionRenewTests.cs` | 7 | Subscription renewal. |
| `Domain.Tests/BranchMembershipTests.cs` | 6 | Branch membership. |
| `Domain.Tests/SubscriptionSnapshotTests.cs` | 6 | Subscription snapshots. |
| `Domain.Tests/BusinessProfileMembershipTests.cs` | 5 | Business-profile membership. |

### Finance (229 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Domain.Tests/StatementProcessingJobTests.cs` | 26 | Statement-processing job lifecycle. |
| `Application.Tests/FinancialAccountAppServiceTests.cs` | 26 | Financial-account service. |
| `Domain.Tests/ReconciliationRunTests.cs` | 24 | Reconciliation run aggregate. |
| `Domain.Tests/ReconciliationMatchTests.cs` | 24 | Reconciliation match aggregate. |
| `Application.Tests/StatementImportAppServiceTests.cs` | 24 | Statement import service. |
| `Application.Tests/FnbCsvTransactionHistoryParserV1Tests.cs` | 19 | FNB CSV statement parser. |
| `Domain.Tests/FinancialAccountTests.cs` | 18 | Financial-account aggregate. |
| `Domain.Tests/LedgerTransactionTests.cs` | 17 | Ledger transaction. |
| `Application.Tests/OfxTransactionHistoryParserV1Tests.cs` | 16 | OFX statement parser. |
| `Domain.Tests/FinancialStatementTests.cs` | 15 | Financial statement aggregate. |
| `Domain.Tests/FinancialStatementTransactionTests.cs` | 11 | Statement transaction. |
| `Application.Tests/AccountNormalizationTests.cs` | 6 | Account normalization. |
| `Application.Tests/FinancialStatementIngestionRepositoryTests.cs` | 2 | Ingestion repository. |
| `Application.Tests/FinancialProviderRepositoryTests.cs` | 1 | Provider repository. |

### Business (117 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Application.Tests/BusinessAccountsServiceTests.cs` | 27 | Business accounts service. |
| `Application.Tests/BusinessInvoiceServiceTests.cs` | 18 | Invoicing service. |
| `Application.Tests/BusinessPayrollServiceTests.cs` | 14 | Payroll service. |
| `Application.Tests/BusinessExpenseServiceTests.cs` | 13 | Expense service. |
| `Application.Tests/Common/AccountNormalizationTests.cs` | 12 | Shared account normalization. |
| `Application.Tests/BranchTests.cs` | 10 | Branch service. |
| `Application.Tests/BusinessStatementImportsServiceTests.cs` | 8 | Statement-import wiring. |
| `Application.Tests/BusinessHealthServiceTests.cs` | 6 | Business-health metrics. |
| `Application.Tests/BusinessCashflowServiceTests.cs` | 5 | Cashflow service. |
| `Domain.Tests/BranchClosedEventTests.cs` | 2 | Branch-closed domain event. |
| `Domain.Tests/BusinessProfileClosedEventTests.cs` | 2 | Profile-closed domain event. |

### OpenFinance (51 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Domain.Tests/BusinessProfileAssignmentTests.cs` | 10 | Profile assignment. |
| `Domain.Tests/ReconciliationRuleSetTests.cs` | 10 | Reconciliation rule sets. |
| `Application.Tests/CreatorServiceWiringTests.cs` | 9 | Creator-service wiring. |
| `Domain.Tests/BranchAssignmentTests.cs` | 8 | Branch assignment. |
| `Domain.Tests/FinancialAccountBranchTests.cs` | 8 | Account-to-branch linkage. |
| `Domain.Tests/MatchingRuleTests.cs` | 6 | Matching rules. |

### Consents (31 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Application.Tests/ConsentsAppServiceTests.cs` | 17 | Consent service. |
| `Domain.Tests/ConsentTests.cs` | 14 | Consent aggregate. |

### Audit (17 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Domain.Tests/AuditEventChainTests.cs` | 10 | Tamper-evident audit chain. |
| `Application.Tests/AuditChainSweepServiceTests.cs` | 4 | Chain sweep service. |
| `Application.Tests/AuditChainVerifierPagingTests.cs` | 3 | Chain verifier paging. |

### Tenants (2 cases)

| File | Cases | Covers |
|:---|---:|:---|
| `Application.Tests/ApplicationModuleTests.cs` | 1 | Module wiring. |
| `Infrastructure.Tests/TenantsDbContextTests.cs` | 1 | DbContext wiring. |

## Observations

- Coverage is strongest in **Finance** (229) and **PlatformAccess** (121),
  reflecting the platform's reconciliation and subscription complexity.
- **Tenants** is thin (2 cases) — only wiring is verified.
- All tests are unit / service-level; there are no HTTP-level integration tests
  against the running API.
