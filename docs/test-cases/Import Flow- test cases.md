# Import Flow Test Cases

## Suite 1: Client-Side SHA-256 Fingerprinting

### TC-01: SHA-256 is computed before upload, not after
- **Precondition:** User has selected a valid FNB PDF file (100 KB).
- **Steps:**
	1. Intercept the `POST` to `/statements/imports`.
	2. Inspect the `SourceSha256Hex` field in the `FormData`.
- **Expected:** `SourceSha256Hex` is a 64-character lowercase hex string matching `crypto.subtle.digest('SHA-256', fileBuffer)` computed independently.
- **Assert:** `SourceSha256Hex.length === 64` and `/^[0-9a-f]+$/.test(SourceSha256Hex)`.

### TC-02: Same file produces same hash regardless of filename
- **Precondition:** Two copies of the same PDF with different filenames (`jan.pdf`, `jan_copy.pdf`).
- **Steps:**
	1. Upload `jan.pdf`; capture `SourceSha256Hex` from POST body.
	2. Upload `jan_copy.pdf`; capture `SourceSha256Hex` from POST body.
- **Expected:** Both `SourceSha256Hex` values are identical.
- **Assert:** Hash is content-derived, not filename-derived.

### TC-03: 1-byte change produces a different hash
- **Precondition:** Original file and a tampered copy (one byte changed).
- **Steps:**
	1. Upload original; capture hash A.
	2. Upload tampered copy; capture hash B.
- **Expected:** Hash A ≠ Hash B.

## Suite 2: Statement Import (Sync Path)

### TC-04: Happy path, valid FNB PDF sync import
- **Precondition:** Valid FNB bank statement PDF; account exists with matching account number.
- **Steps:**
	1. Call `importStatementSync(file, accountId)`.
	2. Confirm `POST` hits `/business/statements/imports` with `{ AccountId, File, SourceSha256Hex, StatementType }`.
	3. Confirm backend parses synchronously and returns `StatementBatchStatus`.
- **Expected:** Response contains `{ batchId: string, status: 'Posted' }`.
- **Assert:** No polling is triggered; imports and audit query caches are invalidated on success.

### TC-05: Sync import returns Failed status and error is surfaced
- **Precondition:** Corrupted or password-protected PDF.
- **Steps:**
	1. Call `importStatementSync(corruptedFile, accountId)`.
- **Expected:** Response `{ batchId: string, status: 'Failed' }`.
- **Assert:** Service throws (or returns failed batch); toast displays error; imports cache is still invalidated so the failed batch appears in the import list.

### TC-06: Duplicate file rejected before parsing (sync)
- **Precondition:** File `jan.pdf` was successfully imported previously for this account.
- **Steps:**
	1. Upload the same `jan.pdf` again via sync path.
- **Expected:** Backend returns an error response indicating duplicate SHA-256; `isSuccess: false` in `ResultT`.
- **Assert:** No new `FinancialStatement` record is created; no `LedgerTransaction` records are duplicated; service throws with a meaningful message.

### TC-07: Account ID mismatch (account identifier hash mismatch)
- **Precondition:** FNB statement for account `****1234` uploaded against account `****5678`.
- **Steps:**
	1. Call `importStatementSync(file, wrongAccountId)`.
- **Expected:** Backend rejects with account mismatch error.
- **Assert:** `isSuccess: false`; batch status is `Failed`; error message mentions account number mismatch.

## Suite 3: Statement Import (Queued Path)

### TC-08: Happy path, queued import polls to Posted
- **Precondition:** Valid Standard Bank PDF; `importStatementQueued` is used.
- **Steps:**
	1. `POST` to `/statements/imports`; response `{ batchId, status: 'Received' }`.
	2. `useBatchStatusPolling(batchId, true)` begins polling `GET /statements/:batchId` every 3 s.
	3. Server transitions: `Received → Parsing → Parsed → Posted`.
	4. Poll returns `{ status: 'Posted' }`.
- **Expected:** Polling stops (terminal status reached); imports and audit caches invalidated.
- **Assert:** Exactly 0 polls fire after terminal status is received.

### TC-09: Polling stops immediately on Failed
- **Precondition:** Queued import job fails during parse.
- **Steps:**
	1. `POST` returns `{ batchId, status: 'Received' }`.
	2. Poll cycle 1: `Received` (continues).
	3. Poll cycle 2: `Failed` (terminal).
- **Expected:** `refetchInterval` callback returns `false`; no further polling.
- **Assert:** Failed batch is visible in import list with error state; retry button is shown.

### TC-10: Received status keeps polling at 3 s interval
- **Precondition:** Queued import stuck in `Received` (background worker not yet picked up).
- **Steps:**
	1. Observe `useBatchStatusPolling` with mock clock.
- **Expected:** `refetchInterval` returns `3000` while status is `Received`.
- **Assert:** Hook re-fetches at exactly 3-second intervals; does not back off.

### TC-11: Parsing status keeps polling
- **Precondition:** Large PDF; parse takes 10 s.
- **Steps:**
	1. Poll returns `Parsing` for 3 cycles.
	2. Poll returns `Posted` on cycle 4.
- **Expected:** `refetchInterval` returns `3000` for all non-terminal statuses.
- **Assert:** `POLL_TERMINAL_STATUSES = { Parsed, Validated, Posted, Failed }`; `Parsing` is not in this set.

### TC-12: gcTime 0 ensures stale batch status is not served after unmount
- **Precondition:** Dialog is closed mid-poll (component unmounts).
- **Steps:**
	1. User closes import dialog after `Parsing` status.
	2. User re-opens dialog for same `batchId`.
- **Expected:** Cache is not served; a fresh `GET` is issued immediately.
- **Assert:** `gcTime: 0` on `useBatchStatusPolling` ensures no stale cache entry survives unmount.

## Suite 4: Transaction History Import (Sync Path)

### TC-13: Happy path, FNB CSV layout A sync
- **Precondition:** Valid FNB CSV export (layout A); account exists.
- **Steps:**
	1. Call `importTransactionHistory(file, accountId)`.
	2. `POST` to `/transaction-history/imports` with `FormData`.
- **Expected:** Response `TransactionHistoryBatch { parseStatus: 'Parsed', transactionCount: N, skippedDuplicates: M }`.
- **Assert:** `N > 0`; `historyBatches` cache invalidated for this `accountId`.

### TC-14: Account identifier hash validation rejects wrong account
- **Precondition:** CSV contains account `62194811553`; uploaded against account `062194811553` (leading-zero variant).
- **Steps:**
	1. Call `importTransactionHistory(file, wrongAccountId)`.
- **Expected:** Backend normalizes both account numbers before hashing; if normalization is consistent, import succeeds; if normalization is inconsistent, import fails with hash mismatch.
- **Assert:** This is the leading-zero edge case identified in the architecture audit. Test both `62194811553` and `062194811553` against the same account to confirm consistent normalization.

### TC-15: Duplicate transactions within same CSV are counted, not duplicated
- **Precondition:** CSV contains 100 transactions; 10 are exact duplicates of previously imported transactions.
- **Steps:**
	1. Import the CSV.
- **Expected:** `TransactionHistoryBatch.skippedDuplicates === 10`; `transactionCount === 90`.
- **Assert:** No duplicate `LedgerTransaction` records in database; `SourceImportId` on each new record points to this batch.

### TC-16: OFX import parsed by generic OFX parser
- **Precondition:** Valid `.ofx` file from any bank.
- **Steps:**
	1. Call `importTransactionHistory(ofxFile, accountId)`.
- **Expected:** Parser registry selects OFX generic parser; batch returns `parseStatus: 'Parsed'`.
- **Assert:** `TransactionHistoryBatch.sourceType` reflects `'Ofx'`; transactions created with correct directions (credit/debit).

## Suite 5: Transaction History Import (Queued Path)

### TC-17: Happy path, queued import polls to Parsed
- **Precondition:** Valid CSV; `importTransactionHistoryQueued` is used.
- **Steps:**
	1. `POST` to `/transaction-history/imports/queued`; response `{ parseStatus: 'Received' }`.
	2. `useTransactionHistoryBatchStatusPolling(id, true)` polls `GET /transaction-history/imports/:id` every 3 s.
	3. Server transitions: `Received → Parsing → Parsed`.
- **Expected:** Polling stops; `historyBatches(accountId)` cache invalidated.
- **Assert:** `HISTORY_TERMINAL_STATUSES = { Parsed, Failed }`; `Parsed` (not `Posted`) is the terminal state for history imports, unlike statement imports which go to `Posted`.

### TC-18: History import terminal set differs from statement import terminal set
- **Purpose:** Regression guard; the two polling hooks have different terminal status sets.
- **Assert:** `useBatchStatusPolling` stops on `{ Parsed, Validated, Posted, Failed }`.
- **Assert:** `useTransactionHistoryBatchStatusPolling` stops on `{ Parsed, Failed }`.
- **Assert:** `Validated` and `Posted` do not stop history polling (they are not valid history statuses).

## Suite 6: LedgerTransaction Lineage

### TC-19: Statement import creates LedgerTransaction with StatementId lineage
- **Precondition:** Statement import completes successfully (`Posted`).
- **Steps:**
	1. Fetch `LedgerTransaction` records created by this import.
- **Expected:** Every `LedgerTransaction` has `StatementId !== null`, `StatementTransactionId !== null`.
- **Assert:** `SourceImportId` is null (statement-promoted transactions use Statement lineage, not import lineage).

### TC-20: History import creates LedgerTransaction with SourceImportId lineage
- **Precondition:** Transaction history import completes (`Parsed`).
- **Steps:**
	1. Fetch `LedgerTransaction` records created by this import.
- **Expected:** Every `LedgerTransaction` has `SourceImportId !== null` pointing to this `TransactionHistoryImport.Id`.
- **Assert:** `StatementId` is null; `StatementTransactionId` is null.

### TC-21: No LedgerTransaction without lineage
- **Purpose:** Domain invariant; every transaction must have a traceable origin.
- **Steps:**
	1. Query all `LedgerTransaction` records in the database.
- **Expected:** Every row satisfies `SourceImportId IS NOT NULL OR StatementId IS NOT NULL`.
- **Assert:** 0 rows where both `SourceImportId` and `StatementId` are null.
- **Note:** This catches `CreateDirect()` factory misuse; the known architecture gap.

## Suite 7: Cache Invalidation

### TC-22: Statement sync invalidates imports and audit caches
- **Steps:**
	1. Observe TanStack Query cache before import.
	2. Call `importStatementSync`.
	3. On `onSuccess`, observe cache.
- **Expected:** Queries with key `['money', 'imports']` and `['money', 'audit']` are marked stale and re-fetched.
- **Assert:** Import list in UI reflects new batch without manual page refresh.

### TC-23: History queued import invalidates only historyBatches(accountId), not global imports
- **Steps:**
	1. Import history for `accountId: 'acct-123'`.
- **Expected:** Query key `['money', 'history-batches', 'acct-123']` is invalidated.
- **Assert:** Query key `['money', 'imports']` (statement batches) is not invalidated.
- **Assert:** Query key `['money', 'history-batches', 'acct-456']` (different account) is not invalidated.

### TC-24: Retry invalidates imports and audit caches
- **Precondition:** A batch in `Failed` state exists.
- **Steps:**
	1. Call `useRetryImportBatch().mutateAsync(batchId)`.
- **Expected:** `onSuccess` invalidates `['money', 'imports']` and `['money', 'audit']`.
- **Assert:** Retried batch immediately appears with `Received` status in import list.

## Suite 8: Attention Badge on Money Page

### TC-25: Attention badge counts Failed statement batch
- **Precondition:** One import batch with status `'Failed'`.
- **Steps:**
	1. Load Money page.
- **Expected:** Badge on "Statement Imports" tab shows count 1; badge variant is destructive.
- **Assert:** `isAttentionStatus('Failed') === true`.

### TC-26: Attention badge counts Failed history batch
- **Precondition:** One transaction history batch with `parseStatus: 'Failed'`; no failed statement batches.
- **Steps:**
	1. Load Money page.
- **Expected:** Badge count is 1; variant is destructive.
- **Assert:** `useAllTransactionHistoryBatches()` (no account filter) fires and returns the failed batch; history failures are now counted (regression guard for the bug we fixed; previously `useTransactionHistoryBatches()` without `financialAccountId` was `enabled: false`).

### TC-27: Attention badge is absent when all batches are in terminal success state
- **Precondition:** All statement batches `Posted`; all history batches `Parsed`.
- **Steps:**
	1. Load Money page.
- **Expected:** No badge rendered on "Statement Imports" tab.
- **Assert:** `attentionCount === 0`.

### TC-28: Attention badge uses secondary variant for non-Failed attention statuses
- **Precondition:** One batch with status `'Parsing'` (in-progress).
- **Steps:**
	1. Load Money page.
- **Expected:** Badge shown; variant is secondary (not destructive) since no batch is `Failed`.
- **Assert:** `hasFailed` computed inside badge IIFE is false; `attentionCount > 0` triggers badge.

## Suite 9: Parse State Machine Integrity

### TC-29: Statement batch cannot transition backwards
- **Precondition:** Batch is in `Parsed` state.
- **Steps:**
	1. Attempt to set `status = 'Received'` directly.
- **Expected:** Domain throws invariant violation (immutable transition).
- **Assert:** State machine only allows `Received → Parsing → Parsed → Posted` and `* → Failed`; no backward transitions.

### TC-30: History batch Received status is correctly labelled in UI
- **Precondition:** Queued history import just submitted; `parseStatus: 'Received'`.
- **Steps:**
	1. Check `BatchProgressCard` or status badge.
- **Expected:** Label reads "Queued..." (not blank or "Unknown").
- **Assert:** `STATUS_LABELS['Received'] === 'Queued...'` in `BatchProgressCard`.

## Suite 10: Reconciliation Convergence

### TC-31: Both import paths produce Unmatched LedgerTransactions
- **Precondition:** Statement import and history import both complete for the same account.
- **Steps:**
	1. Query all `LedgerTransaction` records for the account.
- **Expected:** All newly created transactions have `reconciliationStatus: 'Unmatched'`.
- **Assert:** No transaction is pre-assigned `Matched` before a `ReconciliationRun` executes.

### TC-32: ReconciliationRun consumes transactions from both import types
- **Precondition:** 50 `LedgerTransactions` from statement import plus 50 from history import, all `Unmatched`.
- **Steps:**
	1. Trigger `ReconciliationRun` for the account.
- **Expected:** Run processes all 100 transactions regardless of lineage source.
- **Assert:** `ReconciliationRun.TotalCount === 100`; matched transactions reference specific `LedgerTransaction.Id` values.

### TC-33: Duplicate file cannot produce duplicate LedgerTransactions
- **Precondition:** Statement `jan.pdf` already imported and `Posted`.
- **Steps:**
	1. Upload `jan.pdf` again.
- **Expected:** Backend rejects at SHA-256 check; no second set of `LedgerTransaction` records created.
- **Assert:** `LedgerTransaction` count for the account does not change; reconciliation results are not skewed.

## Suite 11: getStatementLines Pagination (Regression for Fixed Bug)

### TC-34: getStatementLines sends skipCount and maxResultCount, not skip and take
- **Steps:**
	1. Call `useStatementLines(batchId, 0, 100)`.
	2. Intercept the `GET` to `/statements/imports/:batchId/lines`.
- **Expected:** Query string contains `?skipCount=0&maxResultCount=100`.
- **Assert:** Parameters `skip` and `take` are absent from the request.

### TC-35: Page 2 of statement lines loads correct offset
- **Steps:**
	1. Call `useStatementLines(batchId, 100, 100)`.
- **Expected:** Query string `?skipCount=100&maxResultCount=100`.
- **Assert:** Query key includes `[..., 100, 100, '']`; TanStack Query treats this as a separate cache entry from page 1.

## Suite 12: getAuditLog Paged Response (Regression for Fixed Bug)

### TC-36: Import audit log widget receives array of entries, not a paged object
- **Precondition:** Real API returns `{ items: AuditLogEntry[], total: N, page: 1, pageSize: 10 }`.
- **Steps:**
	1. Load Money page and open Audit Log tab.
- **Expected:** `ImportAuditLog` component receives entries as an array, not a paged result object.
- **Assert:** `getAuditLog` extracts `.items` from `result.value` before returning; `Array.isArray(entries) === true` in `ImportAuditLog`.

### TC-37: Audit log widget sends correct pagination params
- **Steps:**
	1. Intercept `GET` to `/business/audit` from the Money page Audit Log tab.
- **Expected:** Query string `?subjectType=FinancialStatement&skipCount=0&maxResultCount=10`.
- **Assert:** Without `skipCount` and `maxResultCount`, backend might return a default large page or reject the request.

## Summary Table

| Suite | TCs | What It Guards |
| --- | --- | --- |
| SHA-256 fingerprinting | TC-01 to TC-03 | File identity, duplicate prevention |
| Statement import sync | TC-04 to TC-07 | Happy path, failures, duplicates, wrong account |
| Statement import queued | TC-08 to TC-12 | Polling lifecycle, terminal statuses, cache hygiene |
| History import sync | TC-13 to TC-16 | CSV and OFX parsing, account hash, dedup counting |
| History import queued | TC-17 to TC-18 | Terminal status set difference vs statement |
| LedgerTransaction lineage | TC-19 to TC-21 | StatementId vs SourceImportId, no orphan transactions |
| Cache invalidation | TC-22 to TC-24 | Correct query key scope per import type |
| Attention badge | TC-25 to TC-28 | History failures visible (regression for fixed bug) |
| Parse state machine | TC-29 to TC-30 | Immutability, Received label |
| Reconciliation convergence | TC-31 to TC-33 | Both import types feed the same pool, no duplicate reconciliation data |
| Statement lines pagination | TC-34 to TC-35 | skipCount and maxResultCount regression |
| Audit log paged shape | TC-36 to TC-37 | `.items` extraction regression |

