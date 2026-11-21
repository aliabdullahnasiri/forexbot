# ForexBot 2.0 — Coding Agent Pack

This is a **documentation/instructions overlay** for the existing ForexBot repository. It is not a fully implemented trading application. Copy its contents into your repository root (preserving hidden directories). Existing application source files are intentionally not overwritten.

## What each coding agent reads
- **Codex**: `AGENTS.md` at the repository root. Additional reusable Codex skill: `.agents/skills/forexbot-task/SKILL.md`.
- **Claude Code**: `CLAUDE.md` at the repository root, optional path-based rules under `.claude/rules/`, and reusable skills under `.claude/skills/`.
- **Both**: canonical specifications in `PROJECT_CONTEXT.md`, `docs/`, and task tracking under `tasks/`.

## Start developing
1. Read `PROJECT_CONTEXT.md`, `docs/PRD.md`, and `tasks/NEXT.md`.
2. Inspect the **real** repository. Do not assume the proposed directory structure exists yet.
3. Start the earliest incomplete milestone. Do not skip testing or execute demo trades until all gates are passed.
4. Use `uv` for Python work; follow `docs/ENGINEERING_STANDARDS.md`.
5. After each task, update `tasks/NEXT.md` and the relevant backlog checkbox with factual status.

## Suggested prompts
For Codex:
> Read AGENTS.md, PROJECT_CONTEXT.md, and tasks/NEXT.md. Inspect the current repository. Implement only the first unchecked Phase 1 task with tests. Do not activate broker execution. Report changes, checks run, and remaining risks.

For Claude Code:
> Read CLAUDE.md, PROJECT_CONTEXT.md, and tasks/NEXT.md. Check the repository before making changes. Implement the next Phase 1 task and tests while keeping all trading actions disabled. Summarize verified results.

## Merge caution
If your repo already has an `AGENTS.md`, `CLAUDE.md`, `.claude/`, or `docs/`, review differences before overwriting those paths. The risk rules in `docs/RISK_POLICY.md` are safety requirements and must not be weakened.

## Current state
Documentation package only. MT5 gateway, Telegram monitoring, risk engine, dashboard, backtesting and demo execution remain **unimplemented** unless your repository has separately added them.
