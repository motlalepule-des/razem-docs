# Test Inventory — `my-razem-account`

Customer-facing account portal. React + TypeScript, tested with **Vitest**
(`jsdom`). Playwright is configured for end-to-end testing but no specs have
been authored yet.

- **Test files:** 13
- **Test cases:** 119
- **Runner:** Vitest (`vitest.config.ts`, setup file `src/test/setup.ts`)
- **E2E:** Playwright installed and configured (`playwright.config.ts`,
  `playwright-fixture.ts`) — **0 specs**

Run:

```bash
bun run test         # vitest run
bun run test:watch   # vitest (watch mode)
```

## Tests by area

### Libraries (`src/lib`) — 37 cases

| File | Cases | Covers |
|:---|---:|:---|
| `pkce.test.ts` | 20 | PKCE code-verifier / code-challenge generation for the OAuth flow. |
| `utils.test.ts` | 10 | Shared utility helpers. |
| `mockEnvironment.test.ts` | 7 | Mock-environment detection and configuration. |

### Services (`src/services`) — 24 cases

| File | Cases | Covers |
|:---|---:|:---|
| `api/apiRequest.test.ts` | 12 | HTTP request wrapper — headers, error handling, response parsing. |
| `api/tokenManager.test.ts` | 8 | Access/refresh token storage and refresh logic. |
| `config.service.test.ts` | 4 | Runtime configuration service. |

### Providers (`src/app/providers`) — 17 cases

| File | Cases | Covers |
|:---|---:|:---|
| `AppConfigProvider.test.tsx` | 17 | App configuration provider — loading, error, and ready states. |

### Feature services (`src/features`) — 40 cases

| File | Cases | Covers |
|:---|---:|:---|
| `security/services/security.service.test.ts` | 16 | Security settings — password change, sessions, MFA. |
| `profile/services/profile.service.test.ts` | 8 | Profile read/update. |
| `organizations/services/organizations.service.test.ts` | 7 | Organization membership and management. |
| `invitations/services/invitations.service.test.ts` | 5 | Invitation accept/decline. |
| `apps/services/apps.service.test.ts` | 4 | Connected-apps management. |

### Scaffolding — 1 case

| File | Cases | Covers |
|:---|---:|:---|
| `src/test/example.test.ts` | 1 | Placeholder test verifying the Vitest setup. |

## Observations

- This is the best-covered frontend repo: service and provider layers are well
  tested, and PKCE/token logic (security-critical) has strong coverage.
- Coverage is concentrated on **services and providers**; React **components and
  pages** are largely untested.
- Playwright is ready but unused — an `e2e/` suite for login and account
  management would close the biggest gap (see
  [`../test-cases/authentication.md`](../test-cases/authentication.md) and
  [`../test-cases/account-management.md`](../test-cases/account-management.md)).
