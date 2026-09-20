# 10 · Deployment and shutdown

The default PGlite path makes the course runnable. Docker Compose swaps in PostgreSQL. `infra/aws/template.yaml` shows an optional container deployment baseline. Run migrations before serving traffic, keep secrets outside images, use health checks, drain in-flight work and make workers tolerate duplicate delivery.

**Checkpoint:** `npm run typecheck && npm test && npm run build`
