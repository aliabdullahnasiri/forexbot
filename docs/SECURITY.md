# Security and Safety Threat Model

## Protected assets
Telegram auth session; Telegram bot token; MT5 demo account credentials; gateway service credentials; Neon database credentials; operational configs; audit and trade data.

## Threats and controls
1. **Prompt injection in channel messages:** never follow directions in raw post, generated model response or forwarded content. Parse only supported trading fields; model has no execution tools.
2. **Unauthorized broker execution:** restrict gateway to VPN/private network, mutual auth or short-lived tokens, allowlisted clients, DEMO verification per mutation, strict order payload schema and caps.
3. **Duplicated execution from retries:** unique keys in database and durable gateway dedup; `UNKNOWN` state requires reconciliation, not automatic repeat.
4. **Credential leakage:** use host env/secret store, never Git or frontend; redact logs; rotate on exposure; Telethon `.session` files treated as secrets.
5. **Compromised dashboard:** authentication + role authorization, CSRF protections where cookies used, request validation, TLS, secure sessions, rate limits, no public gateway.
6. **Database failure:** pause new trading, preserve broker-side protective stops, reconcile state before restart of admission.
7. **Account switching:** gateway independently reads actual account ID and demo trade mode before **every** mutation and checks configured allowlist. Refuse mismatched or non-demo account.
8. **Unauthorized source:** allowlist Telegram channel by stable ID; verify access rights, avoid scraping prohibited/private content.
9. **Dependency / supply chain:** lock uv dependencies, run CI scanning when configured, review dependency updates and never run unknown downloaded scripts as administrator.

## Secrets handling
- `.env` is local only, excluded from Git. Public `.env.example` contains placeholders.
- Never print complete SQLAlchemy Neon URLs, passwords, connection strings or bot tokens.
- Prefer read-only Neon role for analytics queries where architecture supports it; use least-privileged DB role for app and a separately controlled role for migrations.
- API responses redact broker account identity and private source identifiers for non-admin users.

## Before enabling demo execution
- Gateway reachable only via private authenticated transport.
- MT5 account type and identity verified from terminal API on every order and close.
- Admin explicitly opts in; state stored and reviewed.
- Risk/sizing, idempotency, SL and failure drills pass.
- Monitoring and emergency stop are tested.

## Incident procedure
Immediately disable new executions; capture event IDs without disclosing credentials; check broker-side positions/stops; reconcile database with broker; rotate compromised credentials; investigate root cause; require explicit operator approval before restarting automatic demo orders.
