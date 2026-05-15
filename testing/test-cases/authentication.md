# Test Cases — Authentication & Sessions

Covers sign-in, OAuth/PKCE, token lifecycle, password reset, OTP, and session
management across the Razem platform.

Related docs: [API Authentication](../../docs/api/authentication.md),
[Account Security](../../docs/account/security.md).
Related inventory: [razem-api](../inventory/razem-api.md),
[my-razem-account](../inventory/my-razem-account.md).

---

### TC-AUTH-001 — Sign in with valid email and password

| Field | Value |
|:---|:---|
| **Area** | Sign-in |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — a verified user account exists.

**Steps**
1. Open the portal sign-in page.
2. Enter the registered email and correct password.
3. Submit.

**Expected result** — the user is authenticated, an access token is issued, and
they land on the dashboard.

---

### TC-AUTH-002 — Sign in rejected for invalid credentials

| Field | Value |
|:---|:---|
| **Area** | Sign-in |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — a verified user account exists.

**Steps**
1. Open the sign-in page.
2. Enter the registered email with an incorrect password.
3. Submit.

**Expected result** — sign-in is rejected with a generic error; no token is
issued and the error does not reveal whether the email exists.

---

### TC-AUTH-003 — Unauthenticated direct-URL access redirects to login

| Field | Value |
|:---|:---|
| **Area** | Route guarding |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-admin-portal` — `src/app/components/AnimatedRoutes.permissions.test.tsx` |

**Preconditions** — no active session.

**Steps**
1. Navigate directly to a protected route URL.

**Expected result** — the app redirects to the login page.

---

### TC-AUTH-004 — PKCE code verifier and challenge generation

| Field | Value |
|:---|:---|
| **Area** | OAuth / PKCE |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/lib/pkce.test.ts` |

**Preconditions** — none.

**Steps**
1. Generate a PKCE code verifier.
2. Derive the code challenge from it.

**Expected result** — the verifier meets length/charset requirements and the
challenge is a valid S256 transform of the verifier.

---

### TC-AUTH-005 — Authorization-code exchange completes login

| Field | Value |
|:---|:---|
| **Area** | OAuth / PKCE |
| **Priority** | High |
| **Type** | Manual |
| **Coverage** | None (PKCE primitives covered by TC-AUTH-004) |

**Preconditions** — the user has completed authorization at the identity
provider and a callback `code` is present.

**Steps**
1. Land on the callback route with `code` and `state`.
2. Exchange the code (with the stored verifier) for tokens.

**Expected result** — access and refresh tokens are stored and the user is
redirected to their intended destination.

---

### TC-AUTH-006 — Access token refresh on expiry

| Field | Value |
|:---|:---|
| **Area** | Token lifecycle |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/services/api/tokenManager.test.ts` |

**Preconditions** — a session with an expired access token and a valid refresh
token.

**Steps**
1. Make an API request that triggers token refresh.

**Expected result** — the refresh token is exchanged for a new access token and
the original request succeeds without user interaction.

---

### TC-AUTH-007 — Authenticated request attaches bearer token

| Field | Value |
|:---|:---|
| **Area** | Token lifecycle |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/services/api/apiRequest.test.ts` |

**Preconditions** — an active session.

**Steps**
1. Issue an API request through the shared request wrapper.

**Expected result** — the `Authorization: Bearer` header is attached and error
responses are surfaced consistently.

---

### TC-AUTH-008 — Email OTP issuance and validation

| Field | Value |
|:---|:---|
| **Area** | OTP |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Identity/Domain.Tests/EmailOtpTests.cs`, `Razem.Application.Tests/OtpServiceTests.cs` |

**Preconditions** — an account that requires email verification.

**Steps**
1. Request an email OTP.
2. Submit the received code.

**Expected result** — a correct, unexpired code validates; an incorrect or
expired code is rejected.

---

### TC-AUTH-009 — Password reset request and confirmation

| Field | Value |
|:---|:---|
| **Area** | Password reset |
| **Priority** | High |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Identity/Application.Tests/PasswordResetAppServiceTests.cs` |

**Preconditions** — a verified user account exists.

**Steps**
1. Request a password reset for the account email.
2. Confirm the reset using the issued token and a new password.

**Expected result** — the reset token validates once, the password is updated,
and the token cannot be reused.

---

### TC-AUTH-010 — User session lifecycle

| Field | Value |
|:---|:---|
| **Area** | Sessions |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `razem-api` — `Modules/Identity/Domain.Tests/UserSessionTests.cs` |

**Preconditions** — a user account exists.

**Steps**
1. Create a session.
2. Revoke / expire the session.

**Expected result** — session state transitions correctly and a revoked session
is no longer valid.

---

### TC-AUTH-011 — Sign out / revoke active sessions

| Field | Value |
|:---|:---|
| **Area** | Sessions |
| **Priority** | Medium |
| **Type** | Automated |
| **Coverage** | `my-razem-account` — `src/features/security/services/security.service.test.ts` |

**Preconditions** — a signed-in user with one or more active sessions.

**Steps**
1. Open security settings.
2. Revoke a session (or sign out everywhere).

**Expected result** — the targeted session(s) are invalidated and stored tokens
are cleared.

---

### TC-AUTH-012 — Client-credentials (app) authentication

| Field | Value |
|:---|:---|
| **Area** | API auth |
| **Priority** | Medium |
| **Type** | Manual |
| **Coverage** | None |

**Preconditions** — a registered API client with a valid client ID/secret.

**Steps**
1. Request a token using the client-credentials grant.
2. Call an API endpoint with the returned token.

**Expected result** — a scoped access token is issued and authorizes the call;
invalid credentials are rejected. See [Client Credentials](../../docs/api/client-credentials.md).
