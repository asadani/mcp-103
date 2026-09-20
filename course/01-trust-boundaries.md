# 01 · Threat model before tools

**Failure to induce:** treat tool discovery as authorization and accept `org` from arguments.

Inspect `src/policy.ts` and `src/product.ts`. Identity supplies organization, team, role and tier. The caller never selects a tenant. Every invocation repeats authorization because hiding a tool is only a usability decision.

**Checkpoint:** `npm test -- --test-name-pattern="tenant"`
