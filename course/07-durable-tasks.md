# 07 · Durable asynchronous work

Stateless MCP means the request carries its context. It does not mean every operation must finish quickly. Durable jobs store actor, tenant, input, attempts, lease and expiry. `start_release_job` returns a handle; `get_job` exposes only the creating actor's job.

This is the application pattern behind long-running work. The MCP Tasks extension standardizes the protocol-facing lifecycle with `tasks/get`, `tasks/update`, and `tasks/cancel`; the database or queue still owns durability. The extension does not define `tasks/list` or a separate `tasks/result` method.
