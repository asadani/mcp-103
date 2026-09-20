# 08 · MCP-to-MCP composition

Composite tools are useful when they collapse a stable business transaction. Carry the original subject, tenant, trace ID, deadline and remaining budget through each hop. `nextHop` also tracks visited servers and caps depth to stop cycles.

Avoid opaque agent-inside-agent chains. They hide cost, latency, policy decisions and the source of failures.
