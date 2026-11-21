# System Architecture

## Component view

```text
Telegram approved channel
  -> Telethon listener (dedicated process)
  -> PostgreSQL raw-message inbox + revisions
  -> parsing worker (regex/templates; optional structured AI fallback)
  -> deterministic validation + decision engine
  -> risk engine (account + symbol + quote + policy)
  -> execution orchestrator (idempotency key + state machine)
  -> private Windows MT5 gateway -> local MT5 terminal -> DEMO broker
  -> reconciliation worker -> PostgreSQL orders/fills/positions
  -> analytics projection -> FastAPI REST/WS -> React dashboard
  -> operator notification worker -> Telegram Bot API
```

## Boundaries
- **FastAPI backend:** HTTPS API, auth, read models, operator-controlled flags; no direct MT5 order calls in HTTP handlers.
- **Telegram listener:** Telethon session and allowlisted channel ID; persists first and never executes directly.
- **Parser worker:** pure rule-based parsing; optional AI fallback returns structured data only; stores evidence and versions.
- **Risk service:** deterministic, side-effect-free pretrade evaluation from versioned policies and a coherent account/quote snapshot.
- **Execution orchestrator:** durable state machine, concurrency control, idempotency, order lifecycle, no automatic retries after unknown status.
- **Windows gateway:** network-isolated service with local MT5 desktop terminal; independent demo-only/account/symbol/risk validation for each mutation; private authenticated API.
- **Neon PostgreSQL:** authoritative events and normalized facts, accurate order/fill ledger; async application pool, direct migrations.
- **Analytics worker:** builds derived aggregates from fills/signal outcomes; never reports estimates as broker facts.

## Recommended architecture for MVP
Start as a modular monolith plus **dedicated** Telethon/worker processes and a Windows gateway. Use PostgreSQL transactionally for message inbox and idempotency. Add Redis or a full queue only after volume/operational need is demonstrated. No remote public MT5 command endpoint.

## Event and state models
**Message:** RECEIVED -> PARSED | IGNORED | REJECTED | REVIEW_REQUIRED. Revision events preserve earlier raw text.
**Signal:** NEW -> VALIDATED -> SKIPPED | WAITING_ENTRY | APPROVED -> SUBMITTING -> OPEN | REJECTED | UNKNOWN -> CLOSED/CANCELED.
**Order:** PLANNED -> SUBMITTING -> ACKNOWLEDGED/PARTIALLY_FILLED/FILLED/REJECTED/UNKNOWN -> RECONCILED.
**Position:** OPEN -> PARTIALLY_CLOSED -> CLOSED; the gateway is source of truth.

Every transition must be stored with `occurred_at`, `correlation_id`, actor/source and reason. `UNKNOWN` forbids blind retry. Map broker order/deal/position identifiers explicitly; no assumption that ticket numbers have identical semantics on every broker/account mode.

## Idempotency and delivery
- Raw messages unique on `(channel_id, telegram_message_id)`; revisions unique on message/revision or content hash + timestamp.
- Trading intent unique on `(channel_id, telegram_message_id, trading_account_id)` even when text is edited. Multiple TP legs must have distinct stable child IDs under one intent.
- Create durable intent and planned order in database before dispatch. Requests carry immutable `client_order_id`/idempotency key; gateway retains a durable dedup record.
- On timeouts, reconcile using gateway/broker order/deal history and comment/magic mapping **before** retrying; if uncertain, require operator review.

## Trust boundaries
Telegram post -> untrusted content; AI output -> untrusted interpretation; browser -> untrusted client input; Linux orchestrator -> untrusted request until validated by Windows gateway; broker execution report -> authoritative after reconciliation. Never treat a post as operational instruction.

## Deployment and failure model
Ubuntu VPS (FastAPI + workers + web UI) communicates with Neon via TLS and Windows gateway through VPN/private network and service auth. Both OS environments require supervised restart, clocks synchronized, secrets stored outside Git. A Neon outage, stale quote, Telegram disconnection, missing risk inputs or unreachable gateway immediately **pauses new trade admission**. Existing broker-side stops must already be in place.

## Future extensions
Additional strategies/providers, stronger analytics, ML quality ranking, and distributed work queues only after measurable need and new acceptance tests.
