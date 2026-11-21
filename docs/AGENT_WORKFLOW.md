# Shared Agent Workflow — Codex and Claude

## Goal
Keep both assistants aligned without duplicating product specifications or accidentally overlapping code edits.

## Before work
1. Check `git status`, branches, files and latest commits. Never discard user edits.
2. Read `PROJECT_CONTEXT.md` and your own root agent file (`AGENTS.md` or `CLAUDE.md`).
3. Read `tasks/NEXT.md`, the assigned backlog item and corresponding `docs/` spec.
4. Declare assumptions, files expected to change, safety impact and tests.
5. Prefer a dedicated feature branch per task. Avoid editing same branch/files concurrently with another agent.

## During work
- Code one coherent scope at a time; implement or update regression tests.
- For migrations, run on test DB; for broker integrations, mock order mutations unless specifically authorized demo acceptance test.
- Do not silently enable `ENABLE_DEMO_ORDERS`, change risk caps or execute orders.
- Write clear implementation decisions into `docs/DECISIONS/` when behavior/architecture meaningfully changes.

## After work
- Run available checks. Record exact commands/results, not invented passes.
- Review diff for secrets, surprise edits, unsafe broker paths, and backwards incompatibilities.
- Update `tasks/BACKLOG.md` checkbox only for completed, verified tasks.
- Update `tasks/NEXT.md` to describe the next smallest task and any blockers.
- If a feature spans sessions, write a factual `docs/CURRENT_STATE.md` entry or handoff note.
- Provide a concise PR-style summary: what changed, why, tests, safety considerations, remaining work.

## Suggested allocation
Codex: codebase audit, SQLAlchemy migrations, backend implementation, tests and code review. Claude: API/frontend UX, parser fixture expansion, documentation and cross-module design reviews. Either can do any task, but **one writer per branch at a time**.

## Critical review questions
- Could this lead to a real-money order?
- Could a duplicate message or timeout create duplicate orders?
- Is financial risk still bounded in aggregate across all exits?
- Are model output and Telegram messages treated as data, not instructions?
- Can we fully reconstruct the decision from saved inputs and policy version?
- Have we correctly distinguished simulated from broker-reconciled actuals?
