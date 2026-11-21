# Database Schema — Logical Design (Neon PostgreSQL)

Use async SQLAlchemy 2.x and Alembic. **This is a design contract, not a migration.** Column details may evolve through reviewed migrations. All values use UTC timestamps; prices, quantities and monetary values use `NUMERIC`/Python `Decimal`.

## Source and signal tables
- `telegram_channels(id UUID PK, telegram_channel_id BIGINT UNIQUE, title TEXT, is_enabled BOOL, created_at TIMESTAMPTZ)`
- `telegram_messages(id UUID PK, channel_id FK, telegram_message_id BIGINT, published_at TIMESTAMPTZ, first_seen_at TIMESTAMPTZ, status TEXT, UNIQUE(channel_id, telegram_message_id))`
- `telegram_message_revisions(id UUID PK, message_id FK, revision_no INT, text TEXT, text_sha256 TEXT, observed_at TIMESTAMPTZ, UNIQUE(message_id, revision_no))`
- `trading_signals(id UUID PK, message_id FK UNIQUE, latest_revision_id FK, kind TEXT, symbol TEXT, side TEXT, entry_price NUMERIC, stop_loss NUMERIC, state TEXT, parser_version TEXT, created_at TIMESTAMPTZ)`
- `signal_targets(id UUID PK, signal_id FK, position_no INT, price NUMERIC, volume_fraction NUMERIC NULL, UNIQUE(signal_id,position_no))`
- `signal_decisions(id UUID PK, signal_id FK, message_revision_id FK, decision TEXT, reason_codes JSONB, policy_version TEXT, created_at TIMESTAMPTZ)`

## Account and execution tables
- `trading_accounts(id UUID PK, broker TEXT, mt5_login_reference TEXT, symbol_alias TEXT, currency TEXT, demo_verified_at TIMESTAMPTZ, enabled BOOL)` — store reference/masked login, not passwords.
- `account_snapshots(id UUID PK, account_id FK, balance NUMERIC, equity NUMERIC, margin NUMERIC, free_margin NUMERIC, captured_at TIMESTAMPTZ)`
- `risk_policies(id UUID PK, account_id FK, version INT, max_risk_fraction NUMERIC, daily_limit_fraction NUMERIC, max_setups INT, enabled BOOL, changed_by FK NULL, created_at TIMESTAMPTZ)`
- `risk_assessments(id UUID PK, signal_id FK, account_id FK, snapshot_id FK, policy_id FK, side TEXT, assumed_entry NUMERIC, sl NUMERIC, volume NUMERIC, expected_loss NUMERIC, estimated_costs NUMERIC, status TEXT, reasons JSONB, created_at TIMESTAMPTZ)`
- `trade_intents(id UUID PK, signal_id FK, account_id FK, idempotency_key TEXT UNIQUE, state TEXT, created_at TIMESTAMPTZ, UNIQUE(signal_id,account_id))`
- `orders(id UUID PK, intent_id FK, client_order_id TEXT UNIQUE, mt5_order_id TEXT NULL, requested_volume NUMERIC, request_json JSONB, state TEXT, submitted_at TIMESTAMPTZ NULL)`
- `fills(id UUID PK, order_id FK, broker_deal_id TEXT, volume NUMERIC, price NUMERIC, commission NUMERIC, swap NUMERIC, realized_pnl NUMERIC, executed_at TIMESTAMPTZ, UNIQUE(order_id,broker_deal_id))`
- `positions(id UUID PK, intent_id FK, broker_position_id TEXT, open_volume NUMERIC, state TEXT, opened_at TIMESTAMPTZ, closed_at TIMESTAMPTZ NULL)`
- `position_events(id UUID PK, position_id FK, event_type TEXT, details JSONB, occurred_at TIMESTAMPTZ)`

## Operations and analytics
- `system_events(id UUID PK, event_type TEXT, severity TEXT, correlation_id TEXT, details JSONB, occurred_at TIMESTAMPTZ)`
- `outbox_events(id UUID PK, topic TEXT, aggregate_id UUID, payload JSONB, published_at TIMESTAMPTZ NULL, created_at TIMESTAMPTZ)`
- `daily_account_metrics(account_id FK, day DATE, start_equity NUMERIC, end_equity NUMERIC, realized_pnl NUMERIC, fees NUMERIC, max_drawdown NUMERIC, PRIMARY KEY(account_id,day))` (derived projection)

## Required constraints and indexes
- Unique channel + Telegram message ID; unique signal + account intent; unique client order ID; unique broker deal ID scoped appropriately to account/broker.
- Index: messages(channel_id,published_at DESC), signals(state,created_at), orders(state,submitted_at), fills(executed_at), events(correlation_id), account_snapshots(account_id,captured_at).
- Add CHECK constraints for positive entry, SL, target price, volume, risk fraction in [0,0.005], daily limit <=0.02 where persisted as policy. DB validation complements gateway validation.
- `trading_accounts` must never store MT5 password or Telegram session content.
- Audit records append-only where possible; corrections via compensating events, not overwriting financial history.

## Analytics definitions
- **Total signals**: unique provider setup messages, not every message/TP leg.
- **Executed signals**: unique signal/account intents with at least one broker fill.
- **Win rate by signal**: completed executed signals with aggregate **net realized P&L >0** / all fully closed executed signals; breakeven reported separately.
- **Trade-leg win rate**: positive net completed legs / completed legs, separate metric.
- **Profit factor**: gross positive net P&L / absolute gross negative net P&L, undefined if no negative trades.
- **Drawdown**: peak-to-trough decline on time-ordered equity curve. Distinguish balance and equity curves.
- **Provider accuracy**: compare full source population with eligible/executed sample; explicitly show rejected/expired/unfilled signals to avoid selection bias.

## Migration caution
Use direct Neon connection for Alembic when required by pooling semantics. Check PostgreSQL version/connection URI from real deployment. Never migrate production without verifying backup and rollback approach.
