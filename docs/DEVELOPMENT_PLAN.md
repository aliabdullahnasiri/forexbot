# Phased Development Plan

**Delivery sequence is dependency-driven.** Each phase requires tested acceptance criteria before advancing toward demo execution. Estimates are planning ranges, not promises.

| Phase | Focus | Estimated effort | Unlocks |
|---|---|---|---|
| 1 | uv Python foundation, CI, health endpoint, config | Week 1 | Safe engineering base |
| 2 | Neon Postgres + SQLAlchemy models/migrations | Week 2 | Durable event/ledger storage |
| 3 | Telegram listener and replay-safe inbox | Week 3 | Reliable signals |
| 4 | Deterministic parser and optional LLM fallback | Week 4 | Validated trade intents |
| 5 | Windows MT5 gateway (read-only first) | Weeks 5–6 | Broker-specific facts |
| 6 | Deterministic risk and execution state machine | Weeks 7–8 | Paper -> guarded demo |
| 7 | Dashboard and analytics | Weeks 9–10 | Operational observability |
| 8 | Historical signal replay and provider evaluation | Week 11 | Quality measurement |
| 9 | Security/failure testing and deployment | Weeks 12–13 | VPS reliability |
| 10 | Sustained forward demo validation | Week 14+ | Evidence-based decisions |

## Phase 1 exit criteria
`uv sync` works, FastAPI `/health` returns expected response, config validates safe defaults, Ruff and pytest pass, README and env templates describe the project. No gateway order endpoint enabled.

## Phase 2 exit criteria
Async Neon DB connectivity via TLS; models for channels/messages/revisions/signals/accounts/order intents/positions/fills/events; Alembic migration tested; duplicate message constraints and transactional tests.

## Phase 3 exit criteria
Accessible channel monitor persists new/edited messages; reconnect/backfill covered; secret Telegram session protected; no duplicate raw-message records.

## Phase 4 exit criteria
Typed parsing of known formats; explicit ambiguity rejection; golden sample tests and adversarial/prompt-injection tests; no missing price inference; no live trading.

## Phase 5 exit criteria
Windows gateway reads account info/quotes/symbol metadata/positions; broker DEMO mode tested; private auth and narrow allowlist; mock order endpoints tested but disabled by default.

## Phase 6 exit criteria
Decimal risk calculations and broker-based profit checks; concurrency/idempotency; fail-closed policy; SL/TP plans; ambiguous timeout reconciliation; paper simulations; manual operator demo activation gate.

## Phase 7 exit criteria
Dashboard auth; overview; trade and signal ledger; costs; daily P&L, signal-level win rate, max drawdown and equity; reconciliation tests.

## Phase 8 exit criteria
Historical channel replay without look-ahead bias; spread/fees/slippage; provider-level signal outcomes; distinguish simulated results from verified demo fills.

## Phase 9 exit criteria
Ubuntu services and Windows gateway supervised; private networking; restore/backup procedures; alerts; deliberate outage, restart and duplicate-replay tests; runbooks verified.

## Phase 10 exit criteria
Controlled forward demo monitoring and periodic independently audited performance summaries. Real-money support remains out of scope.

## Release bundles
- **v0.1 Monitoring MVP:** Phases 1–4; review-only, execution off.
- **v0.2 Demo automation:** Phases 5–6; broker/risk gates and staged enablement.
- **v1.0 Observability:** Phases 7–10; dashboard, replay, deployment and validation.

Use `tasks/BACKLOG.md` for implementation-level items and `tasks/NEXT.md` as the handoff point between Codex and Claude.
