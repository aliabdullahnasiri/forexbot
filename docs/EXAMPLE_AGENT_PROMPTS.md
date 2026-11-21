# Tested-by-Design Agent Prompts (Templates)

These are task prompts, not evidence that features already exist. Use them inside the cloned repository.

## Codex — First task
```text
Read AGENTS.md, PROJECT_CONTEXT.md, tasks/NEXT.md and docs/ENGINEERING_STANDARDS.md.
Inspect all existing files and git status. Perform only F1.1. Record verified facts in docs/CURRENT_STATE.md.
Do not enable trading or send any broker orders. Report exact checks and next task.
```

## Claude Code — Implement an approved scope
```text
Read CLAUDE.md and PROJECT_CONTEXT.md; follow tasks/NEXT.md and the relevant spec.
Implement only the next verified backlog item with tests. Use uv. Keep all broker order mutations disabled.
Run available checks and give me a concise diff summary, remaining risks, and task handoff update.
```

## Either agent — Parser test expansion
```text
Read docs/SIGNAL_SPEC.md and docs/TESTING.md. Add table-driven parser fixtures for XAUUSD BUY/SELL, 1–3 TPs, malformed SL,
edited messages, duplicate post IDs, and prompt injection. Don't connect to real Telegram or MT5.
```

## Either agent — Risk review
```text
Review code against docs/RISK_POLICY.md and docs/SECURITY.md. Prioritize paths allowing real accounts, lot rounding up,
>0.5% combined risk, missing SL, UNKNOWN order retries, stale quotes and double execution. Do not execute orders.
Provide actionable findings with tests that reproduce each issue.
```

## Either agent — Database schema
```text
Read docs/DATABASE_SCHEMA.md and docs/ARCHITECTURE.md. Implement the smallest migration and typed models for the first
message inbox entities. Use UUIDs, TIMESTAMPTZ, NUMERIC, uniqueness. Test migrations on disposable PostgreSQL only.
```

## Either agent — Analytics
```text
Read docs/DATABASE_SCHEMA.md and docs/PRD.md. Design signal-level and execution-leg metrics separately. Validate
win rate, net P&L, profit factor and drawdown against deterministic fixtures. No fabricated market performance.
```
