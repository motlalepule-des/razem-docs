# Razem Platform — Testing Documentation

Central reference for testing across the Razem platform. This folder collects
the **automated test inventory** for every Razem repository and the
**test-case specifications** for the platform's key user flows.

> Maintained by Digital Edge Solutions (DES). Counts in this folder were last
> reconciled on **2026-05-15**.

## Contents

| Folder / file | Purpose |
|:---|:---|
| [`test-strategy.md`](test-strategy.md) | Testing philosophy, the test pyramid, tooling per repo, and how to run the suites. |
| [`inventory/`](inventory/) | Catalogue of the automated tests that exist today, per repository. |
| [`test-cases/`](test-cases/) | Structured test-case specifications for key user flows (automated **and** manual). |

## How this folder is organized

```
testing/
├── README.md                       # You are here
├── test-strategy.md                # Approach, tooling, how to run
├── inventory/                      # What automated tests exist today
│   ├── README.md
│   ├── razem-api.md
│   ├── razem-admin-portal.md
│   ├── razem-sme-hub.md
│   └── my-razem-account.md
└── test-cases/                     # Test-case specs for key flows
    ├── README.md
    ├── authentication.md
    ├── business-registration.md
    ├── payments.md
    └── account-management.md
```

## Platform test summary

| Repository | Stack | Test runner | Test files | Test cases |
|:---|:---|:---|---:|---:|
| `razem-api` | .NET (C#) | xUnit | 87 | 814 |
| `razem-admin-portal` | React + TypeScript | Vitest | 3 | 3 |
| `razem-sme-hub` | React + TypeScript | Vitest (+ Playwright configured) | 1 | 14 |
| `my-razem-account` | React + TypeScript | Vitest (+ Playwright configured) | 13 | 119 |
| **Total** | | | **104** | **950** |

See [`inventory/`](inventory/) for the per-repository breakdown.

## Conventions

- **Inventory** documents describe automated tests that already exist — they are
  a map of current coverage, not a wishlist.
- **Test-case** documents describe *what should be verified* for a flow. Each
  case is marked `Automated` or `Manual` and, where automated, links to the
  test that covers it.
- When you add or remove a test, update the matching inventory file. When you
  add a new flow, add a test-case file and link it from
  [`test-cases/README.md`](test-cases/README.md).
