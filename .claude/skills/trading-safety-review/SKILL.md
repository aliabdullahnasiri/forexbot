---
name: trading-safety-review
description: Review ForexBot parsing, risk, gateway, order, and persistence changes for demo-only safety, duplicate orders, money calculation errors, and test gaps.
---

# Trading Safety Review

Read `PROJECT_CONTEXT.md`, `docs/RISK_POLICY.md`, `docs/SECURITY.md`, `docs/SIGNAL_SPEC.md`, and `docs/TESTING.md`.

Inspect the changed code and tests; do not activate trading. Verify:
- No REAL account access through mutation paths; gateway enforces DEMO independently.
- No execution by default or bypassable feature flag.
- Risk includes all legs and costs, caps and volume rounding enforced.
- Correct stop placement, quote freshness, spread/deviation and broker symbol checks.
- Replays/edits/timeouts do not cause duplicated broker orders.
- UNKNOWN orders are reconciled before retry.
- Message text/LLM cannot issue program commands or direct orders.
- Secrets protected and logs redacted.
- Analytics reconcile fills and distinguish signals from legs.

Produce a prioritized findings report with file/line references, reproducible failure scenarios, suggested fix, and tests. If none found, state remaining blind spots and what was not tested.
