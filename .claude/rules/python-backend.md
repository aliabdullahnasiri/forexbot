---
paths:
  - "**/*.py"
  - "**/pyproject.toml"
  - "**/alembic/**"
---

# Python Backend Rules

- Use Python>=3.12 and uv, not an alternate project manager.
- Keep FastAPI routes thin; dependency-inject DB sessions and integration clients.
- Use SQLAlchemy 2 async and Pydantic v2; make financial computations with Decimal.
- UTC-aware timestamps; no shared async sessions across concurrent tasks.
- Add tests for meaningful logic, error cases, migration changes and idempotency.
- Before declaring success run available `uv run ruff check .` and `uv run pytest`; say what was actually run.
