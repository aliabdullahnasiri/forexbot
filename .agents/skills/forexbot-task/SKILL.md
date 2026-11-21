---
name: forexbot-task
description: Use for implementing, auditing or reviewing a scoped ForexBot feature using the shared project context, phase backlog, safety contract and uv test workflow.
---

# ForexBot Scoped Task Workflow

1. Read `PROJECT_CONTEXT.md`, root `AGENTS.md`, `tasks/NEXT.md` and relevant `docs/` spec.
2. Inspect current repository before deciding whether task is already implemented.
3. Define minimal edits, failure cases and proof of safety. For execution changes, read `docs/RISK_POLICY.md` and `docs/SECURITY.md`.
4. Keep implementation incremental, typed and testable. Never enable real trading or default demo auto execution.
5. Run applicable uv lint/test commands. Do not invent tool results.
6. Check diff for secrets, schema drift, hidden broker calls and untested retry paths.
7. Update verified backlog status and next task handoff.
8. Return concise summary with exact check results and any blockers.
