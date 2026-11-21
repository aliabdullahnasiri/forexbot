# Test Strategy and Acceptance Plan

## Test layers
- **Unit:** pure parser, Decimal risk sizing, policy constraints, state transitions, metrics formulas.
- **DB integration:** disposable PostgreSQL (not production Neon), migrations, uniqueness, concurrent trade admission, transactional outbox/inbox.
- **Contract:** Telegram adapter mock; Windows gateway API request/response schema; MT5 broker adapter mock.
- **End-to-end paper:** replay frozen channel posts through parser/risk/paper fill simulator, verify audit lineage and dashboard.
- **Demo smoke:** authorized DEMO account, isolated market conditions, human-approved controlled tests only after safety gate.

## High-value acceptance cases
1. `SELL 3523 / SL 3540 / TP1 3519 / TP2 3518 / TP3 3500` parses precisely.
2. Wrong-side SL or target fails with reason.
3. Missing price, mixed BUY+SELL, or prompt-injection text cannot become an order.
4. Duplicate message/edit/reconnect produces at most **one** trade intent.
5. Requested lot below minimum: rejected, never rounded up.
6. Three TP legs together remain <=0.5% planned risk.
7. Account trade mode REAL or unknown: gateway rejects, regardless of Linux setting.
8. Daily loss 2% breached: no new trade until verified reset.
9. Stale quote/unknown symbol/Neon unavailable: no new order.
10. Order POST times out after broker accepts: UNKNOWN -> reconcile, never blind repeat.
11. Existing position + second signal: rejects by one-setup policy.
12. Partial fill and netting partial close are reconciled correctly.
13. Analytics count one signal (not three wins) for three TP legs.
14. All analytics distinguish gross, fees, net and realized versus unrealized.
15. Restart preserves kill switch, lockout, pending intents, audit history.

## Fixtures
Use fake XAUUSD broker symbol metadata and synthetic quotes marked as fixtures; **never claim they are real market prices**. Add golden input/output JSON cases to `tests/fixtures/`. Use test-specific DB credentials.

## Quality commands
```bash
uv sync
uv run ruff check .
uv run ruff format --check .
uv run pytest -q
```
Only claim test success after commands execute. Add type checks/security scanning once configured. Avoid tests that could send real MT5 orders: broker mutations must be mocks except specifically authorized DEMO smoke tests.

## Release gates
No demo auto execution until parser, broker account check, risk limits, duplicate handling, SL protection, timeout reconciliation, permissions, and emergency stop tests pass. Human operator must explicitly enable it.
