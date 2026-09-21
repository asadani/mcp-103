# 04 · Retry only safe work

Read `src/resilience.ts` and `src/policy.ts`. Retry bounded, transient failures only. Writes require an operation key; changed arguments with the same key are rejected. The operation claim, product mutation and stored result commit in one transaction, so a crash before commit rolls all three back. Backoff must fit inside the caller's deadline.

**Failure to induce:** move the product write outside the transaction and inject a failure before the result is recorded. Then restore the transaction and verify neither the write nor the claim survives.

**Checkpoint:** `npm test -- --test-name-pattern="idempotency"`
