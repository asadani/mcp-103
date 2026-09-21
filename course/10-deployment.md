# 10 · Deployment and shutdown

The default PGlite path makes the course runnable. Docker Compose swaps in PostgreSQL. `infra/aws/template.yaml` shows an optional container deployment baseline, not a deployed reference environment. Run migrations before serving traffic, keep secrets outside images, use health checks, drain in-flight work and make workers tolerate duplicate delivery. Test the image and template in the target account before treating the IAM, secret, networking, and KMS settings as production-ready.

**Checkpoint:** `npm run typecheck && npm test && npm run build`
