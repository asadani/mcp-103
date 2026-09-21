# 04 · Retry only safe work

Read `src/resilience.ts` and `src/policy.ts`. Retry bounded, transient failures only. Writes require an operation key; changed arguments with the same key are rejected. The operation claim, product mutation and stored result commit in one transaction, so a crash before commit rolls all three back. Backoff must fit inside the caller's deadline.

**Failure to induce:** move the product write outside the transaction and inject a failure before the result is recorded. Then restore the transaction and verify neither the write nor the claim survives.

**Checkpoint:** `npm test -- --test-name-pattern="idempotency"`

An operation key does not last forever. `cleanupOperationalData()` in `src/db.ts` deletes
operation records seven days after they were created (audit rows after thirty days,
rate-limit rows after one day). A retry inside the seven days is replayed; a retry after
them is a new operation and the write runs again, so clients must stop retrying a write
long before a week has passed. `tests/idempotency.test.ts` checks both sides of that line.
