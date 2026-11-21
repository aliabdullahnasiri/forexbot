# Next Task — Agent Handoff

**Current target:** Phase 1 — Foundation; task **F1.1**.

## Goal
Inspect the actual GitHub repository and its uv project setup; identify which files already exist and which are missing. Do not overwrite existing application work with assumptions from the documentation overlay.

## Steps
1. Read `PROJECT_CONTEXT.md`, `AGENTS.md` or `CLAUDE.md`, and `docs/ENGINEERING_STANDARDS.md`.
2. Inspect `git status`, root files, `pyproject.toml`, Python package path, tests and `.gitignore`.
3. Write a concise `docs/CURRENT_STATE.md` based **only on verified repository facts**: current runtime commands, dependencies, tests, and gaps.
4. If the repository is already usable, recommend the smallest next implementation step F1.2; do not quietly perform later phases.
5. Run commands only where appropriate; record actual output and failures.

## Acceptance
- [ ] Existing code mapped without destructive changes.
- [ ] Current state document reports facts and unknowns.
- [ ] Next task identified and status updated.
- [ ] No broker or Telegram credentials used or requested.

## Agent notes
Keep this file updated between Codex and Claude sessions. On task completion: change `Current target`, update backlog checkbox only when proven, and summarize changed files and tests. Do **not** claim work is done if only plans/specs exist.
