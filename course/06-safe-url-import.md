# 06 · URL ingestion is a security boundary

`SafeImporter` requires HTTPS, rejects URL credentials, resolves DNS, blocks private and link-local addresses, validates every redirect, limits bytes and applies a deadline. Extracted text is still marked untrusted; stripping HTML does not make instructions safe.

Production deployments should also use an egress proxy, re-check the connected IP, validate media types and run malware/content scanning appropriate to the product.

**Checkpoint:** `npm test -- --test-name-pattern="URL validation"`
