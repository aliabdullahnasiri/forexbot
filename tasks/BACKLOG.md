# Implementation Backlog

Checkboxes are **not** assertions of repository state. Agents must inspect code and tests before marking items complete. No trade execution before phases 1–6 gates pass.

## Phase 1 — Foundation
- [ ] F1.1 Audit current repository layout and dependencies; document baseline.
- [ ] F1.2 Set up uv package metadata, Python pin and reproducible lockfile.
- [ ] F1.3 Add FastAPI app factory and GET `/health`.
- [ ] F1.4 Define safe env config with `ENABLE_DEMO_ORDERS=false` and `MODE=paper`.
- [ ] F1.5 Add pytest, Ruff, CI workflow and sample tests.
- [ ] F1.6 Write local setup instructions and `.env.example`.

## Phase 2 — Postgres + domain
- [ ] F2.1 Configure Neon TLS async database access and direct migration connection.
- [ ] F2.2 Design baseline tables and constraints per `docs/DATABASE_SCHEMA.md`.
- [ ] F2.3 Add initial Alembic revision and upgrade/downgrade tests.
- [ ] F2.4 Add repositories and transaction-safe message inbox.
- [ ] F2.5 Implement internal event log, correlation IDs and reason codes.

## Phase 3 — Telegram listener
- [ ] F3.1 Connect authorized Telethon account to one allowlisted channel.
- [ ] F3.2 Persist original posts, edits and content hashes.
- [ ] F3.3 Add reconnect and backfill with dedup.
- [ ] F3.4 Record listener health and age-of-message metrics.

## Phase 4 — Parser + AI
- [ ] F4.1 Implement message classification and strict Pydantic signal schemas.
- [ ] F4.2 Add regex/template parser with fixture-based tests.
- [ ] F4.3 Add semantic validation, quote/entry policy checks (no execution).
- [ ] F4.4 Add optional, disabled-by-default structured LLM fallback.
- [ ] F4.5 Build review/rejection queue and audit evidence.

## Phase 5 — MT5 gateway
- [ ] F5.1 Windows service reads DEMO account metadata and symbol details.
- [ ] F5.2 Private authenticated gateway for market snapshots and positions.
- [ ] F5.3 Build dry-run order validation and safe test-double broker.
- [ ] F5.4 Implement gateway-side demo verification and duplicate ID ledger.
- [ ] F5.5 Read-only integration tests on authorized demo terminal.

## Phase 6 — Risk + execution
- [ ] F6.1 Deterministic size/risk calculator using broker profit calculations.
- [ ] F6.2 Spread, entry deviation, quote-age and calendar gates.
- [ ] F6.3 Durable order state machine and reconciliation for UNKNOWN.
- [ ] F6.4 Partial fills, TP legs, hedging/netting behaviors.
- [ ] F6.5 Daily lockout, single setup constraint, persistent kill switch.
- [ ] F6.6 Paper-trading tests, then opt-in guarded DEMO-only execution.

## Phase 7 — Analytics
- [ ] F7.1 Authenticated dashboard skeleton and navigation.
- [ ] F7.2 Signals/trade/order detail pages with broker/source links.
- [ ] F7.3 Equity/P&L, risk, win rate and drawdown analytics.
- [ ] F7.4 Reconciliation-driven report tests and CSV export.

## Phase 8 — Replay
- [ ] F8.1 Historical signal import/replay with message timestamps.
- [ ] F8.2 Realistic broker bid/ask cost model, no look-ahead.
- [ ] F8.3 Provider scorecard and out-of-sample reports.

## Phase 9 — Deployment
- [ ] F9.1 VPS Docker/systemd, health, graceful shutdown and backups.
- [ ] F9.2 Windows gateway supervision and private networking.
- [ ] F9.3 Test simulated outages, restarts, missed messages, dup orders.
- [ ] F9.4 Complete incident response and secrets audit.

## Phase 10 — Validation
- [ ] F10.1 Forward demo testing with no real-money exposure.
- [ ] F10.2 Weekly quality and profitability reports with verified costs.
- [ ] F10.3 Backlog improvements driven by observed failure modes.
