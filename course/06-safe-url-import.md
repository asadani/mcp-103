# 06 · URL ingestion is a security boundary

`SafeImporter` requires HTTPS, rejects URL credentials, resolves DNS, blocks private and link-local addresses, pins the checked address for the TLS connection, validates and pins every redirect, accepts only HTML or plain text, limits bytes and applies a deadline. Extracted text is still marked untrusted; stripping HTML does not make instructions safe.

Production deployments should also use an egress proxy or network policy and run malware/content scanning appropriate to the product. Those controls limit the damage if an application-layer check is later weakened.

**Checkpoint:** `npm test -- --test-name-pattern="URL import"`
