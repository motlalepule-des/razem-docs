# Test Inventory — `razem-admin-portal`

Internal admin portal. React + TypeScript, tested with **Vitest** in a `jsdom`
environment (`@testing-library/react`).

- **Test files:** 3
- **Test cases:** 3
- **Runner:** Vitest (`vitest.config.ts`, setup file `src/test/setup.ts`)

Run:

```bash
bun run test                          # vitest run
bun run check:permissions-architecture # the two permission-contract checks
```

## Tests

| File | Cases | Covers |
|:---|---:|:---|
| `src/test/permissions-architecture.test.ts` | 1 | Asserts feature page entry files do **not** use page-level deny guards — permission gating must happen at the route layer. |
| `src/app/components/AnimatedRoutes.permission-contract.test.ts` | 1 | Asserts every protected route declares a `requiredPermission`. |
| `src/app/components/AnimatedRoutes.permissions.test.tsx` | 1 | Renders `AnimatedRoutes` and verifies unauthenticated direct-URL access redirects to login. |

## Observations

- All three tests are **architectural / security-contract** tests — they guard
  the permissions model rather than feature behaviour.
- There is **no feature-component coverage** (forms, tables, dialogs, data
  fetching). This is the largest gap in this repository.
- See [`../test-cases/`](../test-cases/) for the manual cases that cover admin
  feature flows until component tests are added.
