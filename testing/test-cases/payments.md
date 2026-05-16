# Test Cases — Payments & Reconciliation

Covers moving money, beneficiaries, statement import, and bank reconciliation.

Related docs: [Payments API](../../docs/api/payments.md),
[Beneficiaries API](../../docs/api/beneficiaries.md),
[Managing Payments](../../docs/account/payments.md).
Related inventory: [razem-api](../inventory/razem-api.md).

---

### TC-PAY-001 — Send a payment to a beneficiary

| Field | Value |
|:---|:---|
| **Area** | Payments |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — an authenticated user with a funded account and a saved
beneficiary.

**Steps**
1. Open the send-payment flow.
2. Select the beneficiary, enter an amount within the available balance.
3. Confirm.

**Expected result** — the payment is accepted, the balance is reduced, and a
transaction record is created.

---

### TC-PAY-002 — Reject a payment that exceeds the available balance

| Field | Value |
|:---|:---|
| **Area** | Payments |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — an authenticated user whose account balance is below the
intended payment amount.

**Steps**
1. Enter an amount greater than the available balance.
2. Attempt to confirm.

**Expected result** — the payment is rejected with an insufficient-funds error
and no transaction is created.

---

### TC-PAY-003 — Add and manage a beneficiary

| Field | Value |
|:---|:---|
| **Area** | Beneficiaries |
| **Priority** | Medium |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — an authenticated user.

**Steps**
1. Add a beneficiary with account details.
2. Edit, then remove the beneficiary.

**Expected result** — the beneficiary is created, updated, and deleted, with
validation on the account details.

---

### TC-PAY-004 — Ledger transaction records debits and credits

| Field | Value |
|:---|:---|
| **Area** | Ledger |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Finance/Domain.Tests/LedgerTransactionTests.cs` |

**Preconditions** — a financial account exists.

**Steps**
1. Post a ledger transaction.

**Expected result** — the debit/credit is recorded with correct sign, amount,
and resulting balance behaviour.

---

### TC-PAY-005 — Reference generation for transactions

| Field | Value |
|:---|:---|
| **Area** | Payments |
| **Priority** | Low |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Razem.Application.Tests/ReferenceGeneratorTests.cs` |

**Preconditions** — none.

**Steps**
1. Generate a transaction reference.

**Expected result** — the reference is unique and matches the expected format.

---

### TC-PAY-006 — Import a bank statement (FNB CSV)

| Field | Value |
|:---|:---|
| **Area** | Statement import |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Finance/Application.Tests/FnbCsvTransactionHistoryParserV1Tests.cs`, `Modules/Finance/Application.Tests/StatementImportAppServiceTests.cs` |

**Preconditions** — a financial account and a valid FNB CSV statement file.

**Steps**
1. Upload the CSV statement.
2. Run the import.

**Expected result** — transactions are parsed and ingested; malformed rows are
reported rather than silently dropped.

---

### TC-PAY-007 — Import a bank statement (OFX)

| Field | Value |
|:---|:---|
| **Area** | Statement import |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Finance/Application.Tests/OfxTransactionHistoryParserV1Tests.cs` |

**Preconditions** — a financial account and a valid OFX statement file.

**Steps**
1. Upload the OFX statement.
2. Run the import.

**Expected result** — transactions are parsed and ingested correctly.

---

### TC-PAY-008 — Statement processing job lifecycle

| Field | Value |
|:---|:---|
| **Area** | Statement import |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Finance/Domain.Tests/StatementProcessingJobTests.cs` |

**Preconditions** — a statement has been uploaded.

**Steps**
1. Start, progress, and complete (or fail) the processing job.

**Expected result** — job state transitions are valid and terminal states are
enforced.

---

### TC-PAY-009 — Reconcile imported transactions against the ledger

| Field | Value |
|:---|:---|
| **Area** | Reconciliation |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Finance/Domain.Tests/ReconciliationRunTests.cs`, `Modules/Finance/Domain.Tests/ReconciliationMatchTests.cs` |

**Preconditions** — imported statement transactions and ledger entries exist.

**Steps**
1. Start a reconciliation run.
2. Apply matching rules.

**Expected result** — matched transactions are linked, unmatched items are
flagged, and the run reports a correct summary.

---

### TC-PAY-010 — Matching rules drive reconciliation

| Field | Value |
|:---|:---|
| **Area** | Reconciliation |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/OpenFinance/Domain.Tests/MatchingRuleTests.cs`, `Modules/OpenFinance/Domain.Tests/ReconciliationRuleSetTests.cs` |

**Preconditions** — a reconciliation rule set is configured.

**Steps**
1. Evaluate transactions against the rule set.

**Expected result** — rules match the intended transactions and rule-set
precedence is respected.
