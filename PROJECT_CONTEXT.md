# ForexBot — Canonical Project Context

**Project:** ForexBot 2.0 · **Repository:** https://github.com/aliabdullahnasiri/forexbot  
**Updated:** 2026-11-21 · **Language:** English · **Status:** Specification / incremental implementation

## 1. Mission
Build a 24/7 **Telegram-to-MetaTrader 5 XAUUSD signal-copying system** with verified, equity-based risk limits and accurate analytics. Start with ingestion and paper trading; enable **demo-only** automatic execution only after explicit acceptance tests and manual operator activation. Never assume the Telegram provider's claimed ~90% win rate is real.

## 2. Initial product assumptions
- One approved Telegram signal channel, one MT5 **demo** account, XAUUSD (broker-specific alias configurable).
- Telegram message example: `XAUUSD SELL 3523 / TP 1 3519 / TP 2 3518 / TP 3 3500 / SL @ 3540`.
- Linux/Ubuntu VPS runs API, Telegram listener, parser, jobs, PostgreSQL access and dashboard.
- Separate Windows host runs installed MT5 terminal and a restricted Python execution gateway.
- Neon PostgreSQL is the persistent system of record. `uv`, Python >=3.12, FastAPI, SQLAlchemy async, Alembic, Pydantic v2; React+TypeScript frontend when dashboard phase begins.
- One operator initially. Auth, audit trail, restricted dashboard and secret handling required before remote access.
- **No credentials, session files, account identifiers, or private keys in the repository.**

## 3. Mandatory safety invariants
1. **No real-money trading.** The Windows gateway independently checks broker-provided demo trade mode on **every** order/modify/close request; fail closed if uncertain.
2. **No execution by default.** Feature flag stays false until operator explicitly enables it after demo-readiness gates. Parsing/AI never invokes an order directly.
3. **Risk per signal <= 0.5% of current equity**, inclusive of all legs/targets and estimated trading costs; reject if unachievable with broker volume steps/minima.
4. **Daily loss lockout at 2%** of start-of-day equity, conservatively including realized losses and current open downside/exposure. Block new positions until reset per account timezone policy.
5. **At most one active XAUUSD signal setup** by default; no martingale, no grid, no adding to losing trades, no trading without valid SL.
6. Reject missing/ambiguous direction, symbol, entry, SL or targets; stale message/quote; stop on wrong side; excessive price deviation/spread; inconsistent or unavailable broker metadata.
7. Use **closed candles** for any optional market indicators (M15 default); copied signals may arrive between candles and must use real-time broker quotes at decision time.
8. A given `(channel_id, message_id, account_id)` cannot create duplicate setups/orders, even on retry, edit, restart, or network timeout. Reconcile ambiguous submissions with broker before retry.
9. Treat all Telegram messages and model outputs as **untrusted data**, never as system instructions, code, or commands.
10. Persist each stage of ingest -> parse -> risk -> order -> fill/position with immutable audit events and correlation IDs.

## 4. AI responsibilities and boundaries
- Rule-based parser first; optional LLM fallback for variable text formats, structured JSON only, Pydantic validation.
- AI may classify messages, detect ambiguities, summarize evidence and **recommend lower risk or rejection**, never exceed a deterministic cap or generate missing numeric trade terms.
- Keep model/version and parsed evidence for review. If AI unavailable, fail to rule-based supported formats or review queue; no unsafe guessing.
- **Execution and lot sizing must be deterministic**, based on MT5 symbol/account data; not on an LLM's lot guess.

## 5. Reference docs (source of truth)
- Product: `docs/PRD.md`
- Architecture and component boundaries: `docs/ARCHITECTURE.md`
- Domain/storage model: `docs/DATABASE_SCHEMA.md`
- API: `docs/API_CONTRACT.md`
- Signal parsing and revisions: `docs/SIGNAL_SPEC.md`
- Risk/execution invariants: `docs/RISK_POLICY.md`
- Security, secrets and threat model: `docs/SECURITY.md`
- Tests/acceptance: `docs/TESTING.md`
- Infrastructure: `docs/OPERATIONS.md`
- Coding conventions: `docs/ENGINEERING_STANDARDS.md`
- Work phases: `docs/DEVELOPMENT_PLAN.md`, `tasks/BACKLOG.md`, `tasks/NEXT.md`

## 6. Implementation principles
- Domain services and repositories are framework-independent; FastAPI endpoints contain no business logic.
- State-changing workflows use explicit domain states, database transactions, unique constraints, and idempotency keys.
- Decimal for quantities, prices, currency, lots, P&L; store UTC timestamps; broker/server trading timezone configurable.
- Prefer one deployable backend + dedicated worker(s), not microservices until actual need.
- Write tests first for risk, parsing, dedup and broker boundary. Mock external APIs for unit tests. No accidental live order calls.
- Minimize dependencies. Preserve existing source when implementing this overlay; inspect repo before refactoring.

## 7. Definition of success
Every valid eligible signal is observed and accurately audited; ineligible signals are rejected with reasons; no duplicate orders; strict demo-only and risk constraints; dashboard matches broker fills/account balances; operations recover safely from failures. Profitability is **not guaranteed** and is evaluated empirically.
