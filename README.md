# MCP 103 · Operate the server

MCP 103 turns the Teamspace server from MCP 102 into a production architecture
lab. The product is still a small task tracker and knowledge host, so each
operational decision can be followed from MCP call to product API and database.

The course targets MCP `2026-07-28`, `@modelcontextprotocol/*` v2 and Node 22+.
It runs locally without a model key or cloud account.

## Interactive tutorial

Read the hosted course at **[tech.anujsadani.in/mcp-103](https://tech.anujsadani.in/mcp-103/)**. The repository root is the tutorial page; the runnable operator lab remains available locally at `http://127.0.0.1:5173/app.html`.

## Run

```powershell
npm install
npm run dev       # API + MCP at http://127.0.0.1:3102
npm run ui        # tutorial at /, product UI at /app.html
npm test
npm run lab
```

The development identity switcher has users from two organizations, two teams,
three roles and two plan tiers. It exists to make isolation failures visible; it
is not a production login system.

## What this repo demonstrates

- per-request identity, organization and team isolation;
- role authorization kept separate from plan entitlement;
- actor token buckets plus organization concurrency limits;
- atomic budget reservation and actual-cost reconciliation;
- idempotent writes, optimistic versions, bounded retries and a circuit breaker;
- exact-operation approval for consequential writes;
- durable actor-scoped jobs with leases and expiry;
- SSRF-resistant, size-bounded URL ingestion whose output stays untrusted;
- audience-specific downstream tokens instead of bearer-token forwarding;
- traceable MCP-to-MCP composition with deadlines, budgets and cycle limits;
- redacted audit records and a small operator view.

Read the ten guided checkpoints in [`course/`](course), starting with
[`01-trust-boundaries.md`](course/01-trust-boundaries.md). Each lesson names a
failure to induce and the code that prevents it.

## Architecture

```text
host ── MCP ── policy / fairness / budget ── Teamspace MCP
                                                │
                                      audience-bound tokens
                                        ┌───────┴────────┐
                                     Tasks API      Knowledge API
                                        └───────┬────────┘
                                           PostgreSQL
                                      jobs / usage / audit
```

PGlite supplies embedded PostgreSQL for the default labs. Set
`DATABASE_URL=postgres://...` for regular PostgreSQL. `docker compose up` starts
that configuration. [`infra/aws/README.md`](infra/aws/README.md) and the sample
template document an optional AWS container deployment and its cleanup.

## Production decisions

The in-process limiter is intentionally observable teaching code. Multiple
replicas need a shared limiter such as Redis or an API-gateway quota, while
business budgets stay transactionally close to the usage ledger. URL import
also needs network-level egress controls in addition to application validation.

The server returns structured, retryable errors and preserves a trace ID. It
does not silently downgrade freshness, retry permanent errors, expose hidden
tools as an authorization mechanism, or let nested calls mint broader identity.

## Verification

```powershell
npm run typecheck
npm test
npm run build
npm run test:ui
```

See [`PRODUCT.md`](PRODUCT.md), [`DESIGN.md`](DESIGN.md), and
[`UX-CONTRACT.md`](UX-CONTRACT.md) for product and interface decisions.

Protocol-sensitive lessons use the MCP
[`2026-07-28` release notes](https://blog.modelcontextprotocol.io/posts/2026-07-28/),
the [Tasks extension](https://tasks.extensions.modelcontextprotocol.io/specification/draft/tasks),
and the [TypeScript SDK v2 migration guide](https://ts.sdk.modelcontextprotocol.io/v2/migration/upgrade-to-v2).
