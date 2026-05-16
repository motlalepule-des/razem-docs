# Test-Case Specifications

Structured test cases for the Razem platform's key user flows. Unlike the
[inventory](../inventory/), which records what is *already* automated, these
files describe **what should be verified** for each flow — covering both
automated and manual cases.

## Flows

| Flow | File | Cases |
|:---|:---|---:|
| Authentication & sessions | [authentication.md](authentication.md) | 12 |
| Business registration | [business-registration.md](business-registration.md) | 9 |
| Payments & reconciliation | [payments.md](payments.md) | 10 |
| Account management | [account-management.md](account-management.md) | 10 |

## Test-case format

Every case follows the same template:

> ### TC-`<AREA>`-`<NNN>` — Short title
>
> | Field | Value |
> |:---|:---|
> | **Area** | Functional area |
> | **Priority** | High / Medium / Low |
> | **Type** | Automated / Manual |
> | **Coverage** | Link to the automated test, or `None` |
>
> **Preconditions** — state the system must be in before the test.
>
> **Steps**
> 1. ...
>
> **Expected result** — what must be true for the case to pass.

### ID scheme

`TC-<AREA>-<NNN>` — area codes: `AUTH`, `BIZREG`, `PAY`, `ACCT`. Numbers are
zero-padded and never reused; retired cases keep their number with a
`(retired)` note.

### Type

- **Automated** — a test in one of the repos verifies this case. The
  **Coverage** field links to it. Keep these in sync with the
  [inventory](../inventory/).
- **Manual** — executed by hand before a release until automation exists.
  Manual cases are the working list for new automated tests.

## Adding a flow

1. Create `<flow>.md` using the template above.
2. Add a row to the **Flows** table.
3. Cross-link any related inventory file.
