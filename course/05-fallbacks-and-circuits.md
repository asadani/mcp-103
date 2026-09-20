# 05 · Honest fallbacks and circuit breakers

A fallback must declare reduced freshness or capability. Returning stale data as if it were fresh corrupts agent decisions. The circuit breaker in `src/resilience.ts` stops repeated calls to a failing dependency and probes recovery after a cooldown.

Do not use retries and fallbacks to hide permanent authorization, validation or business-rule failures.
