# MCP 103 · Operate the server

MCP 103 turns the Teamspace server from MCP 102 into a production architecture
lab. The product is still a small task tracker and knowledge host, so each
operational decision can be followed from MCP call to product API and database.

The course targets MCP `2026-07-28`, `@modelcontextprotocol/*` v2 and Node 22+.
It runs locally without a model key or cloud account.

## Interactive tutorial

Read the hosted course at **[tech.anujsadani.in/mcp-103](https://tech.anujsadani.in/mcp-103/)**. The repository root is the tutorial page, a single self-contained `index.html` in the same format as MCP 101 (no build step; every JSON-RPC frame on it was captured from this repository’s running server, and every code excerpt is pulled from `src/`); the runnable operator lab remains available locally at `http://127.0.0.1:5173/app.html`.

Every chapter can also be listened to: about 27 minutes of narration, one track per chapter, in an AI voice (Kokoro-82M) by default. A switch beside *Listen straight through* flips to Anuj's own voice (about 31 minutes) on `author.html`, and the page remembers the choice as you move between the courses. The scripts are in `narration/`, and `tools/README.md` explains how the audio is made.

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

## Operational notes

- **`GET /healthz`** answers 200 after a `SELECT 1`, or 503 if the database is unreachable. It needs no token, so a container or load-balancer health check can use it.
- **`/mcp` challenges instead of failing.** A missing, invalid, expired or wrong-audience token gets `401` with `WWW-Authenticate: Bearer resource_metadata="…/.well-known/oauth-protected-resource"`. The check (`requireToken` in `src/auth.ts`) runs in Express *before* the MCP handler: an error thrown inside the handler factory is reported by the SDK as a 500, which would hide the challenge from the client.
- **Demo data is development-only.** Members and tasks are seeded unless `NODE_ENV=production`; set `SEED_DEMO_DATA=1` to opt in deliberately.
- **Writes and operation results commit atomically.** `once()` runs the idempotency claim, product write and stored result in one database transaction. A crash before commit rolls everything back; a lost response after commit is replayed by the same key.
- **One pipeline for every tool.** `guarded()` in `src/mcp.ts` identifies the caller, takes a rate-limit permit, runs the tool, writes an audit row (outcome `success`, `failure` or `throttled`) and answers in one structured shape. The job tools and `import_url` use it too, and creating a job needs write permission *and* the Pro plan.
- **The URL importer pins the validated address.** IPv4-mapped IPv6 is unwrapped first, IPv4 ranges include `100.64.0.0/10`, and IPv6 is an allow-list. The HTTPS connection uses the already validated address while preserving the hostname for TLS. An egress proxy or network policy remains useful defence in depth.
- **A refused resource read is a structured not-found.** `resources/read` for a page that is missing, in another team or in another organisation answers with the SDK’s `ResourceNotFoundError` (JSON-RPC `-32602`, with the URI in `data`), never a bare internal error.
- **The circuit breaker guards the product APIs.** `Downstream` keeps one breaker per API. It counts only outages (unreachable, 5xx, blown deadlines), never a definite refusal such as `NOT_FOUND`.
- **Production needs its settings.** With `NODE_ENV=production` the server refuses to start unless `DATABASE_URL`, `TOKEN_SECRET` and `PUBLIC_ORIGIN` are set.
- **Rate limiting is shared.** `SharedRate` keeps the per-member token bucket in PostgreSQL (`rate_buckets`), so the allowance is one number for every replica. The concurrency caps (three per organisation, two per member) stay per process on purpose.
- **Budget holds expire.** A reservation lives five minutes and is swept by the next reservation. Settlement never charges more than was reserved, and `import_url` names its size-and-budget ceiling `maxKilobytes`.
- **Operations data is admin-only.** `/api/ops` rejects members and viewers even when they belong to the same organization.
- **Operational rows have retention.** A maintenance sweep expires approvals and removes old operation, audit, rate, reservation and terminal-job rows.
- **Jobs have a worker.** `src/worker.ts` runs in the process (`WORKER=off` disables it) or on its own with `npm run worker` against a shared PostgreSQL. A lease records its owner, only the owner may complete the job, and a job is dead-lettered after three attempts.
- **A composite tool uses `nextHop()`.** `draft_release_note` reads the completed tasks and the checklist page as two hops under one deadline, one budget and one route, and stops at a draft.
- **Deployment files are written, not proven.** `Dockerfile` and `infra/aws/template.yaml` were not built, linted or deployed where they were written. See `infra/aws/README.md`.
