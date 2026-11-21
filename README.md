# ForexBot

**Telegram signal monitoring, risk-controlled MetaTrader 5 demo trading, and performance analytics.**

> **Project status:** In development. This repository starts with specifications and a Python scaffold. Telegram monitoring, AI parsing, MT5 order execution, and the dashboard are planned features; do not assume they are implemented or operational.
>
> **Trading safety:** Automatic trading is **disabled by default** and must remain **demo-only** until explicit acceptance tests are passed. A signal provider's advertised win rate is unverified. Trading can result in losses.

## What ForexBot is designed to do

ForexBot monitors an authorized Telegram signal channel, parses trade instructions, validates them against live broker quotes and account information, and eventually executes eligible trades on an **MT5 demo account**. It records the complete history of each signal, risk decision, order, fill, and resulting position in **Neon PostgreSQL**, then displays performance in a web dashboard.

Example Telegram message:

```text
NEW TRADE IDEA

XAUUSD SELL 3523

TP 1 3519
TP 2 3518
TP 3 3500

SL @ 3540
```

Expected workflow:

```text
Telegram channel
    -> message listener (Telethon)
    -> raw message + revisions stored in Neon PostgreSQL
    -> rule-based parser; optional structured AI fallback
    -> signal validation
    -> deterministic risk checks (equity, spread, stop loss, exposure)
    -> demo-only MT5 gateway (Windows + MT5 terminal)
    -> orders, fills, position reconciliation
    -> analytics API and dashboard + Telegram alerts
```

**Important:** AI is limited to interpreting messages and assisting review. It must not invent missing prices, choose unchecked lot sizes, bypass limits, or submit orders directly. All position sizing and trade authorization are handled by deterministic, tested code.

## Technology stack

| Component | Planned technology |
| --- | --- |
| Backend API | Python 3.12+, FastAPI, Pydantic v2 |
| Python tooling | `uv`, Ruff, pytest |
| Database | Neon PostgreSQL, SQLAlchemy 2.0 async, Alembic |
| Telegram ingestion | Telethon, with an authorized account/channel |
| Notifications | Telegram Bot API |
| MT5 connectivity | Python MetaTrader5 package + locally installed Windows MT5 terminal |
| Dashboard | React, TypeScript, Tailwind CSS, shadcn/ui, Recharts |
| Deployment | Ubuntu VPS for API/workers; isolated Windows MT5 gateway |

## Initial trading policy

These are **design limits**, not proof of an implemented trading control:

- **Market:** XAUUSD (allow broker-specific aliases such as `XAUUSD.m` when configured).
- **Risk per signal:** Maximum **0.5% of current demo equity**, across all take-profit legs and estimated execution costs.
- **Daily loss lockout:** **2%** of start-of-day equity, using a conservative exposure policy.
- **Active setups:** At most **one** active XAUUSD signal setup by default.
- **Stop loss:** Required for every eligible trade; never expand risk to accommodate a signal.
- **Order execution:** Off by default; independently restricted to demo accounts at the MT5 gateway.
- **No martingale, grid trading, adding to losers, or blind retries after unknown order outcomes.**

Signals with ambiguous instructions, stale prices, missing broker metadata, excessive spread, or an unachievable minimum lot size must be skipped or sent for review.

## Getting started

### Requirements

- Python **3.12+**
- [`uv`](https://docs.astral.sh/uv/)
- Git

Clone the repository:

```bash
git clone https://github.com/aliabdullahnasiri/forexbot.git
cd forexbot
```

For an existing `pyproject.toml`-based scaffold:

```bash
uv sync
uv run pytest
uv run ruff check .
```

If the Python package and `forexbot` CLI entry point from the initial scaffold are present:

```bash
uv run forexbot
```

If the repository contains **only documentation**, initialize the Python package before running the commands above:

```bash
uv init --package forexbot
uv python pin 3.12
uv add fastapi 'uvicorn[standard]' sqlalchemy 'psycopg[binary]' \
  alembic pydantic-settings telethon httpx structlog
uv add --dev pytest pytest-asyncio ruff
```

Do not start a broker connection or enable automatic execution until the implementation and its tests are complete.

## Configuration and secrets

Keep account credentials, Telegram session data, API tokens, and Neon connection strings **out of Git**. Use environment variables or a private secret manager. Copy `.env.example` to `.env` only if that file is present in your checkout, and keep `.env` ignored.

The application must keep `ENABLE_DEMO_ORDERS=false` (or an equivalent explicit flag) by default. A UI toggle alone is not enough to authorize trading: the MT5 gateway must independently verify the account is demo-only before each mutation.

## Development roadmap

| Phase | Outcome |
| --- | --- |
| 1. Foundation | Python project, tests, configuration, CI |
| 2. Database and API | Neon models, SQLAlchemy sessions, migrations |
| 3. Telegram listener | Persistent message ingestion and revisions |
| 4. Parser and AI fallback | Strictly validated, structured signal data |
| 5. MT5 gateway | Demo-account verification and broker connectivity |
| 6. Risk and execution | Broker-accurate sizing, idempotency, partial exits |
| 7. Analytics | Trade ledger, P&L, drawdown, signal win rate |
| 8. Historical evaluation | Replay with spread, fees, slippage assumptions |
| 9. Operations | Deployment, security, monitoring, recovery |
| 10. Demo validation | Forward testing and evidence-based evaluation |

**Analytics must distinguish signal outcomes from individual partial fills**, calculate realized P&L from reconciled broker data, and account for spread, commission, and slippage rather than relying on a channel's reported wins.

## Documentation and coding agents

This repository is intended to be maintained with both **OpenAI Codex** and **Claude Code**. When the agent documentation pack is installed, start with:

- [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) — project purpose, constraints, and safety invariants.
- [`AGENTS.md`](AGENTS.md) — Codex repository instructions.
- [`CLAUDE.md`](CLAUDE.md) — Claude Code instructions.
- [`docs/PRD.md`](docs/PRD.md) — product requirements.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system components and trust boundaries.
- [`docs/RISK_POLICY.md`](docs/RISK_POLICY.md) — position sizing and demo-only safeguards.
- [`docs/DATABASE_SCHEMA.md`](docs/DATABASE_SCHEMA.md) — Neon PostgreSQL schema design.
- [`docs/DEVELOPMENT_PLAN.md`](docs/DEVELOPMENT_PLAN.md) — detailed development phases.
- [`tasks/NEXT.md`](tasks/NEXT.md) and [`tasks/BACKLOG.md`](tasks/BACKLOG.md) — current work and planned tasks.

Agents should inspect existing code, implement a single scoped task, add tests, and update the backlog. They should never weaken risk limits or claim execution is ready without end-to-end verification.

## Deployment design

The Ubuntu VPS runs the API, listener, background workers, and dashboard. A separate restricted **Windows MT5 gateway** communicates with an installed MT5 terminal. Use authenticated private networking (such as a VPN), not a publicly exposed order endpoint. Neon PostgreSQL is the persistent source of truth. On database outages, stale market data, unknown execution outcomes, or loss of gateway connectivity, **new trade admission must stop**.

## License

This project is distributed under the **MIT License**. See [LICENSE.md](LICENSE.md).

Copyright © 2026 Ali Abdullah Nasiri.
