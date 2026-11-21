# Product Requirements Document — ForexBot 2.0

**Status:** Proposed · **Market:** XAUUSD · **Trading:** MT5 demo-only · **Backend:** Python/FastAPI + uv · **Database:** Neon PostgreSQL

## Problem and goal
A signal provider publishes trading ideas on Telegram. Manually monitoring them is slow, and actual performance is difficult to audit. ForexBot must record new messages, interpret supported trade signals, evaluate them against account and market restrictions, optionally copy eligible signals to an MT5 **demo** account, and provide accurate end-to-end analytics. Claims of 90% success are not accepted without independently reconciled data.

## Primary users
- Owner/operator: configure one allowed source, inspect signal quality, change conservative risk settings, pause and resume **demo** execution.
- Reviewer/auditor: inspect decisions, costs, rejection reasons, and performance evidence.

## In-scope v1
1. Ingest messages/edits from one legally accessible Telegram channel, with reconnect, backfill and dedup.
2. Parse XAUUSD BUY/SELL, entry, numbered TPs, SL and relevant close/cancel statements.
3. Normalize with structured types and validate with deterministic rules; optional AI fallback for formatting variations.
4. Calculate broker-accurate allowable size from current demo equity, stop distance, contract metadata, fees and worst-case slippage allowances.
5. Manage safe demo orders, fills, positions, TP legs and SL changes through an authenticated Windows MT5 gateway.
6. Log complete decision and fill lineage, including provider message version and execution request correlation IDs.
7. Provide authenticated web dashboard: overview, signal history, rejection analysis, order/position log, account curve, realized P&L, provider statistics, health.
8. Send operator Telegram alerts for signals, rejections, trades, connection changes and lockouts.
9. Support replay testing with realistic costs, plus forward demo tracking and operational monitoring.

## Out of scope for v1
- Any real-money trading or guarantee of profitability.
- Multiple broker accounts, multiple channel copy configurations, multi-asset optimization.
- Blind LLM-generated execution parameters or automated strategy invention.
- Public multi-tenant SaaS, mobile native app or high-frequency trading.

## Core use cases
**UC1 — Receive:** Telegram post arrives -> persist raw message and metadata -> acknowledge processing -> parse -> classify.
**UC2 — Evaluate:** Valid signal -> freshness, quote, account and exposure checks -> deterministic sizing -> accepted/rejected with reasons.
**UC3 — Execute:** Accepted signal -> prepared idempotent request -> gateway verifies demo and broker constraints -> submits/checks order -> reconcile -> log fills/position.
**UC4 — Manage:** Position open -> monitor SL/TP -> partial exit if supported; edit/cancel messages are reviewed according to explicit policy, not blindly applied.
**UC5 — Analyze:** Reconcile trades and account snapshots, aggregate performance by **signal** and **execution leg**, display cost-adjusted results.
**UC6 — Recover:** Service restarts, channel/network/DB/broker unavailable -> no duplicate orders; reconcile uncertain states before proceeding.

## MVP priorities
**P0:** Config, secure ingest, parser, deterministic safety engine, data lineage, manual/paper review, tests.
**P1:** Demo gateway, idempotent execution, position management, basic dashboard.
**P2:** Full analytics, replay/backtest, advanced alerts, AI fallback, optimization reports.

## Non-functional requirements
- Never allow an unverified real-money account. Trading disabled by default.
- Event ingestion should persist before processing; retry idempotently; use UTC for storage and explicit market/session time zones.
- Database loss/disconnection => pause new trades; do not silently queue live actions for later.
- Encrypted links; least privilege; API auth; no public MT5 gateway; secret rotation; no secret logging.
- Metrics and audit trail permit every broker order to be traced to a specific message version and approved policy version.
- Database money/price types use exact numeric representation; no financial floats in persistence.
- Dashboard metrics must clearly distinguish realized/unrealized P&L, gross/net P&L, and simulations/actual demo fills.

## Key metrics
Signal ingestion coverage; parsing accept/reject rate; time from provider post to broker decision; expected versus actual entry; blocked trades by rule; total signals; opened positions; closed signals; win rate **by signal**; profit factor; expectancy in R; drawdown; daily return; equity curve; slippage; fees; provider claim versus independently measured outcomes.

## Acceptance criteria
- Replay of a message/version cannot create a second broker order.
- Missing stop or invalid side/SL/TP is rejected.
- Under-minimum lot or insufficient equity is rejected, never rounded up.
- Broker account is verified demo on every state-changing call.
- Disconnect and timeout recovery reconciles broker before retry.
- End-to-end audit trail supports trade and analytics reconstruction.
- Unit, integration, contract and demo-only acceptance tests pass before enabling auto demo mode.

## Unresolved product choices (require explicit configuration before demo orders)
- Telegram source channel identifier/permissions; Telegram session operating procedure.
- Entry trigger policy and price deviation maximum; pending order expiration.
- Hedging vs netting account behavior; multi-TP split/partial-close fallback.
- Broker/server time zone, trading calendar, rollover behavior, spread/slippage thresholds.
- Whether edited/cancelled signals may close/modify active orders (default: do not automatically act).

See `RISK_POLICY.md`, `SIGNAL_SPEC.md`, and `DEVELOPMENT_PLAN.md` for implementation detail.
