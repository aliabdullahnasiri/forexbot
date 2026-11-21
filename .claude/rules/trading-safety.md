---
paths:
  - "**/risk/**/*.py"
  - "**/trading/**/*.py"
  - "**/mt5*/**"
  - "**/broker*/**"
  - "**/execution/**/*.py"
---

# Trading-Sensitive Code Rules

When changing broker, risk, execution, or position code, read `docs/RISK_POLICY.md`, `docs/SECURITY.md`, and `docs/TESTING.md` first.

- MT5 demo account mode checked independently inside gateway per mutation.
- Aggregate planned loss per signal <=0.5% current equity, with costs included; daily 2% lockout and maximum one active XAUUSD setup.
- Stop loss mandatory. Never force min lot if it breaches cap. Never round volume up.
- No retry of UNKNOWN order state without broker reconciliation.
- Execution feature flag is false by default. Never send real-money trades or bypass gating.
- Include negative and outage tests for new execution paths.
