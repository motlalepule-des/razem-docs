# Razem Test Strategy

How testing is approached across the Razem platform — the layers, the tooling,
and how to run each suite locally and in CI.

## Goals

- Catch regressions in business logic before they reach a deployed environment.
- Keep the backend domain and application layers under high unit-test coverage.
- Verify that security-critical contracts (authentication, permissions, tenant
  isolation, audit integrity) hold.
- Give every repository a documented, repeatable way to run its tests.

## The test pyramid

| Layer | Where it lives | Tooling | Notes |
|:---|:---|:---|:---|
| **Unit tests** | `razem-api` domain & shared projects; frontend `*.test.ts(x)` | xUnit, Vitest | The bulk of coverage. Fast, no I/O. |
| **Service / integration tests** | `razem-api` `*.Application.Tests` & `*.Infrastructure.Tests` | xUnit (+ in-memory / EF Core providers) | Exercise application services and persistence wiring. |
| **Component tests** | Frontend `*.test.tsx` | Vitest + Testing Library (jsdom) | Render React components and assert behaviour. |
| **End-to-end tests** | Frontend `e2e/` (Playwright) | Playwright | Configured in `razem-sme-hub` and `my-razem-account`; no specs authored yet. |
| **Manual / QA cases** | [`test-cases/`](test-cases/) | — | Flows that are not yet automated, executed by hand before release. |

## Tooling by repository

### `razem-api` — .NET / xUnit

- One test project per production project, named `<Project>.Tests`.
- Module test projects live under `tests/Modules/<Module>/{Domain,Application,Infrastructure}.Tests`.
- Cross-cutting test projects live directly under `tests/` (`Razem.Shared.Tests`, etc.).
- Run everything:

  ```bash
  dotnet test Razem.Api.slnx
  ```

- Run a single project:

  ```bash
  dotnet test tests/Modules/Finance/Domain.Tests/Razem.Finance.Domain.Tests.csproj
  ```

### `razem-admin-portal` — React / Vitest

```bash
bun install            # or: npm install
bun run test           # vitest run
bun run test:watch     # vitest (watch mode)
bun run check:permissions-architecture   # permission-contract guard checks
```

Vitest runs in a `jsdom` environment with `@testing-library/react`. Setup file:
`src/test/setup.ts`.

### `razem-sme-hub` — React / Vitest + Playwright

- Vitest is configured (`vitest.config.ts`) and used for component tests.
- Playwright is installed and configured (`playwright.config.ts`,
  `playwright-fixture.ts`) but no end-to-end specs have been authored yet.
- There is currently no `test` script in `package.json`; run Vitest directly:

  ```bash
  bunx vitest run
  ```

### `my-razem-account` — React / Vitest + Playwright

```bash
bun install
bun run test           # vitest run
bun run test:watch     # vitest (watch mode)
```

Playwright is installed and configured (specs expected under `e2e/`) but no
end-to-end specs have been authored yet.

## CI

Each repository has Azure Pipelines definitions (`azure-pipelines*.yml`).
Backend builds run `dotnet test`; frontend builds run lint and build. As the
frontend `test` scripts mature they should be wired into the same pipelines.

## Conventions

- **Naming.** Backend test classes end in `Tests` and live in a file of the same
  name. Frontend test files use `*.test.ts(x)`; end-to-end specs use `*.spec.ts`.
- **One concern per test.** Prefer many small `[Fact]`/`it()` tests over a few
  large ones; use `[Theory]`/`it.each` for parameterised cases.
- **Arrange-Act-Assert.** Keep the three phases visually separated.
- **No shared mutable state** between tests; each test sets up its own fixtures.
- **Security contracts are tests, not reviews.** Permission, tenant-isolation
  and audit-integrity guarantees are asserted by dedicated tests
  (`AnimatedRoutes.permission-contract.test.ts`, `GlobalFiltersTests`,
  `AuditEventChainTests`, etc.).

## Known coverage gaps

- No authored end-to-end (Playwright) specs in any frontend repo, despite the
  tooling being configured.
- `razem-admin-portal` has only architectural/permission tests — no
  feature-component coverage.
- `razem-sme-hub` has a single component test (`StatusBadge`) and no `test`
  npm script.
- `razem-docs` is a static Jekyll site with no automated tests (build
  verification only).

These gaps are tracked as `Manual` cases in [`test-cases/`](test-cases/).
