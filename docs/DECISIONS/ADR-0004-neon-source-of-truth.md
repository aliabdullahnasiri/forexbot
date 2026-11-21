# ADR-0004 — Neon PostgreSQL as System of Record

**Status:** Accepted for initial architecture · **Date:** 2026-11-21

## Decision
Use Neon PostgreSQL as authoritative storage for source messages, revisions, trading decisions, order states, fills and audit events. Apply SQLAlchemy async + Alembic. Use transactional inbox, uniqueness and durable order intent to prevent replayed signals from becoming duplicate orders. Derived dashboard metrics are computed from reconciled fills and account snapshots.

## Consequences
Database availability becomes a precondition for admitting new orders: loss of database connectivity **pauses new trading**, while protective SL must exist broker-side. Use SSL, least privilege, direct migration connection where required, and verified backups.
