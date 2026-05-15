# Test Inventory — `razem-sme-hub`

SME-facing hub for business customers. React + TypeScript, tested with
**Vitest** (`jsdom`). Playwright is configured for end-to-end testing but no
specs have been authored yet.

- **Test files:** 1
- **Test cases:** 14
- **Runner:** Vitest (`vitest.config.ts`, setup file `src/test/setup.ts`)
- **E2E:** Playwright installed and configured (`playwright.config.ts`,
  `playwright-fixture.ts`) — **0 specs**

> There is currently no `test` script in `package.json`. Run Vitest directly:
>
> ```bash
> bunx vitest run
> ```

## Tests

| File | Cases | Covers |
|:---|---:|:---|
| `src/components/ui/status-badge.test.tsx` | 14 | `StatusBadge` component across 8 groups: success states, warning states, error states, neutral states, custom props, fallback behaviour, and accessibility. |

## Observations

- Only the `StatusBadge` UI primitive is covered. Core SME flows — business
  registration, dashboards, statement import — have **no automated coverage**.
- Playwright is ready to use; authoring an `e2e/` suite for the business
  registration flow would be the highest-value next step (see
  [`../test-cases/business-registration.md`](../test-cases/business-registration.md)).
- A `test` (and ideally `test:e2e`) npm script should be added so the suite can
  be wired into CI.
