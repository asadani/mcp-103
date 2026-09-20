# 04 · Retry only safe work

Read `src/resilience.ts` and `src/policy.ts`. Retry bounded, transient failures only. Writes require an operation key; changed arguments with the same key are rejected. Backoff must fit inside the caller's deadline.

**Failure to induce:** remove the idempotency lookup and retry `create_task` after an ambiguous timeout.

**Checkpoint:** `npm test -- --test-name-pattern="idempotency"`
