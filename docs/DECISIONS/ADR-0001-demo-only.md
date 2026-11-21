# ADR-0001 — Demo-Only Order Execution

**Status:** Accepted for initial releases · **Date:** 2026-11-21

## Context
ForexBot copies third-party Telegram signals of unknown independently verified performance. Automated errors could create material financial losses.

## Decision
For initial releases, ForexBot will only connect order-mutating operations to an MT5 broker-reported **DEMO** account. Execution defaults disabled and requires an audited operator enablement step. The gateway checks broker trade mode on every state-changing request, not just on startup. Backend and gateway both enforce hard per-signal risk and daily loss constraints. No real account support is developed in the initial scope.

## Consequences
- Can test operational correctness without real-money exposure.
- Cannot claim broker performance equals live execution conditions.
- Future expansion to real trading would be a **new product/security decision**, not a hidden flag toggle.

## References
`PROJECT_CONTEXT.md`, `docs/RISK_POLICY.md`, `docs/SECURITY.md`.
