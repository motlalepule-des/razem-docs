# Automated Test Inventory

A catalogue of the automated tests that exist **today** in each Razem
repository. Use it to see where coverage is strong and where it is thin before
adding new tests.

| Repository | Inventory | Test files | Test cases |
|:---|:---|---:|---:|
| `razem-api` | [razem-api.md](razem-api.md) | 87 | 814 |
| `razem-admin-portal` | [razem-admin-portal.md](razem-admin-portal.md) | 3 | 3 |
| `razem-sme-hub` | [razem-sme-hub.md](razem-sme-hub.md) | 1 | 14 |
| `my-razem-account` | [my-razem-account.md](my-razem-account.md) | 13 | 119 |
| **Total** | | **104** | **950** |

> `razem-docs` is a static Jekyll documentation site and has no automated test
> suite, so it has no inventory file.

## How to read these files

- **Test files** — a file containing one or more tests.
- **Test cases** — individual test methods: xUnit `[Fact]`/`[Theory]` on the
  backend, Vitest `it()`/`test()` on the frontend.
- Counts were reconciled on **2026-05-15**. Re-run the count when you change a
  suite (see commands in each file) and update the totals here.
