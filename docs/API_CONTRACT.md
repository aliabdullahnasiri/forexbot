# API Contract — Initial REST and Gateway Interfaces

**Prefix:** `/api/v1` for FastAPI public/admin API. All non-health routes require authentication/authorization. Responses use UTC ISO-8601 timestamps and decimal values as **strings**.

## Operator/API routes
| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Liveness, no sensitive details |
| GET | `/ready` | Dependency readiness, restricted diagnostic detail |
| GET | `/api/v1/channels` | Approved channel configurations |
| GET | `/api/v1/messages` | Ingested messages, filters |
| GET | `/api/v1/signals` | Paginated signal/decision list |
| GET | `/api/v1/signals/{id}` | Parsed fields, revisions and decisions |
| POST | `/api/v1/signals/{id}/review` | Admin review (approve/reject) |
| GET | `/api/v1/accounts` | Masked configured account metadata |
| GET | `/api/v1/accounts/{id}/snapshots` | Equity/balance snapshots |
| GET | `/api/v1/orders` | Order ledger with state |
| GET | `/api/v1/positions` | Reconciled positions |
| GET | `/api/v1/analytics/overview` | Totals, status, summary P&L |
| GET | `/api/v1/analytics/equity` | Time series; demo and simulated distinct |
| GET | `/api/v1/analytics/signals` | Win rate, rejects, provider quality |
| GET | `/api/v1/analytics/risk` | Exposure, budget and lockouts |
| POST | `/api/v1/trading/pause` | Persist and audit kill switch |
| POST | `/api/v1/trading/resume` | Auth, preflight and demo-only gates |
| GET | `/api/v1/events` | Filtered audit/system events |

## Example normalized signal response
```json
{
  "id": "b24a3fc8-280e-4fd1-96ed-b263490c4689",
  "source_message_id": "1234",
  "symbol": "XAUUSD",
  "side": "SELL",
  "entry_price": "3523.00",
  "stop_loss": "3540.00",
  "targets": ["3519.00", "3518.00", "3500.00"],
  "state": "VALIDATED",
  "execution_eligible": false,
  "rejection_reasons": [],
  "created_at": "2026-11-21T00:00:00Z"
}
```
This is an illustrative fixture, not a real Telegram message or broker execution.

## Windows private gateway contract (proposed)
- `GET /gateway/v1/account`: masked account ID/reference, account trade mode DEMO/REAL, currency, equity, balance, free margin, server time.
- `GET /gateway/v1/symbols/{symbol}`: broker symbol spec and trading conditions.
- `GET /gateway/v1/quotes/{symbol}`: bid/ask and broker timestamp.
- `GET /gateway/v1/positions`: current broker positions and orders.
- `POST /gateway/v1/orders/check`: deterministic dry-run validation (does not place).
- `POST /gateway/v1/orders/demo`: **disabled by default**, auth required, gateway verifies account type, policy and idempotency.
- `GET /gateway/v1/orders/{client_order_id}`: reconcile result, never rely on an HTTP request outcome alone.
- Any modify/close operation must pass the same demo and authorization checks.

## Cross-cutting rules
- Stable `X-Correlation-ID` and idempotency token for writes; server must store unique request IDs.
- Explicit `409` for conflicts/duplicate incompatible intent; `422` validation; `503` unavailable dependency. Never interpret timeout as not executed.
- API never returns credentials or secrets. User-provided limits may only tighten hard defaults for v1.
- Require account-scoped authorization and policy-version match for every mutation.
