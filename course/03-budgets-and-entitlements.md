# 03 · Entitlements and cost budgets

Plan tier answers whether a feature is available. The usage ledger answers whether this organization can afford this invocation. `Budget.reserve` performs one conditional database update so concurrent requests cannot all spend the last allowance. Reconciliation records actual cost and releases the remainder.

**Checkpoint:** `npm run lab`
