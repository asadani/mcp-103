# 09 · Logs, traces and safe audit records

An audit record answers who, which tenant, what operation, outcome, trace and time. It deliberately omits tokens, arguments and content. Metrics need tenant-safe dimensions; high-cardinality task IDs and user text do not belong in metric labels.

The UI's operations panel reads the same usage and audit data that an operator would inspect. The backing endpoint is restricted to organization administrators. A maintenance sweep removes expired approvals and jobs, stale rate and reservation rows, idempotency records after seven days, and audit records after thirty days; retention must match the product's replay and compliance promises.
