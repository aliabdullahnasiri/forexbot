# Operations and Deployment

## Topology
- **Ubuntu VPS:** FastAPI API, Telethon listener, parser/risk/analytics workers, web frontend. Deploy services under least-privileged OS account.
- **Windows MT5 node:** Official local MT5 terminal plus private Python gateway, supervised service, auto-restart. Keep broker terminal profile separate from Linux app.
- **Neon:** managed PostgreSQL with SSL, pooled application endpoint when compatible, direct connection for migrations/operations needing session-level features.

## Startup dependencies
1. Secrets/settings loaded outside Git; clocks NTP synced.
2. Neon reachable, migrations at expected revision; listener and gateway credentials valid.
3. Workers start with execution disabled until account/quote/reconciliation health checks pass.
4. Enable **demo** orders explicitly via authenticated operator workflow only after release gates.

## Required observability
- Ingestion heartbeat, last Telegram message processed, queue age, processing errors.
- Neon pool/connection health and migration revision.
- Gateway heartbeat, MT5 connected account ID and DEMO mode, last broker quote timestamp.
- Order submit/ack/fill latency, UNKNOWN orders, unmatched broker history, daily lockout state.
- Signal counts, filter rejection reasons, operator-triggered pauses and resume events.

## Outage playbooks
**Telegram disconnected:** stop new source-derived orders; reconnect, backfill, mark historical data ineligible for execution after TTL.
**Neon unreachable:** block all new order admission; existing broker-side protective stops remain active; reconcile on recovery.
**MT5 gateway unavailable:** block new order admission; do not automatically replay queued orders after reconnect.
**Order result uncertain:** mark UNKNOWN, reconcile broker orders/deals/positions and broker-side IDs, escalate manual review if unresolved.
**Kill switch activated:** block new admission immediately and persist state. Handling already-open demo positions requires explicit playbook; do not blanket-close as a default.

## Backup and recovery
Neon backup/PITR capability depends on service plan; verify actual account features. Export operational configuration separately, test DB restoration in disposable environment. Never store Telethon sessions, Telegram tokens, Neon URLs or MT5 passwords in backups without encryption and access restrictions.

## Deploy checklist
- [ ] Secret inventory and rotation procedure
- [ ] TLS + private gateway network
- [ ] Demo account verification proven
- [ ] App/database migrations reviewed
- [ ] Paper and risk failure drills pass
- [ ] Logs/alerts/health probes working
- [ ] Restart/reconciliation drill
- [ ] Emergency stop tested
