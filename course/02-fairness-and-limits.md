# 02 · Fairness before global rate limits

`FairLimiter` combines an actor-within-organization token bucket with organization concurrency. A single noisy tenant cannot consume every in-flight slot. Limits belong at both the edge and the expensive dependency.

**Failure to induce:** omit `leave()` and observe how a leaked concurrency permit denies later work.

**Checkpoint:** `npm test -- --test-name-pattern="rate limits"`
