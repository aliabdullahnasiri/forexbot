# Trading Risk Policy — Hard Safety Contract

**All rules below apply to the demo system and must be enforced deterministically in code.** AI may suggest rejection or lower exposure; it cannot approve an otherwise blocked trade or set size above limits.

## Defaults
| Parameter | Default | Enforcement |
|---|---:|---|
| Execution enabled | `false` | API + orchestrator + Windows gateway |
| Allowed account type | MT5 `DEMO` | Gateway checks broker trade mode each mutation |
| Instrument | XAUUSD alias configured per broker | Gateway allowlist |
| Risk per **signal** | <= 0.005 of current equity (0.5%) | Deterministic sizing |
| Daily loss stop | 0.02 of start-day equity (2%) | New order admission lockout |
| Concurrent active setups | 1 XAUUSD | Transactional locking |
| Mandatory stop loss | True | Before first order submission |
| Martingale/grid/averaging down | Forbidden | Strategy and order validation |
| New orders after unknown submission | Forbidden | Reconcile first |

Risk includes every leg tied to a Telegram signal. If a signal is split into 3 TP legs, **combined worst-case loss** including expected costs must stay within 0.5%; do not allocate 0.5% per leg.

## Position sizing algorithm
1. Read coherent fresh account snapshot: account ID, trade mode, currency, equity, free margin and existing exposure.
2. Read fresh bid/ask quote, execution price expectation and broker symbol data: point/tick size, contract definition, volume min/max/step, stop-level/freeze constraints, margin requirements.
3. Confirm approved SL is on correct side of expected **actual fill price** and passes broker stop restrictions.
4. Compute maximum loss budget `equity * min(requested_risk_fraction, 0.005)` using `Decimal`.
5. Use MT5 `order_calc_profit` to estimate monetary loss at SL for a test volume, with the correct order side and account currency. Include spread/commission and conservative slippage buffers in the budget.
6. Determine allowable volume; quantize **down** to broker step, never round upward to minimum. Reject if below minimum or margin insufficient.
7. For multiple TP legs, allocate stepped sizes whose **total** volume and worst-case risk remain within budget. If splitting is infeasible, apply a preapproved fallback or skip.
8. Check daily loss, open setup count, quote age, spread, price deviation, trading session, gateway account mode and permissions immediately before placing.
9. Gateway independently recalculates and validates broker-side preconditions. Use `order_check` where supported, but do not treat it as a fill guarantee.
10. Persist risk decision, market/account snapshot, policy version, reason codes and idempotency key.

## Additional controls
- SL should be attached broker-side at entry wherever broker permits. If broker cannot establish protective SL safely, reject/cancel rather than leaving an unprotected position as normal operation.
- Account for exchange/currency conversions using broker-provided profit calculation; do not assume $1 gold-price movement equals a fixed amount per lot across symbols/brokers.
- Count planned maximum loss of open setups conservatively in daily admission check. Realized losses and unrealized losses must be tracked separately and displayed clearly.
- Daily reset time is configured for a stated account/reporting timezone. A restart must **not** clear an active daily lockout.
- Consider concurrent orders, hedging/netting differences, other manual positions, spread spikes, slippage, disconnections and market closure.
- Broker/quote/account state missing or stale => **fail closed**; trading paused, logged and notified.
- Manual risk changes require authenticated admin action, versioned policy history, and confirmation; hard caps may only be tightened for v1.

## Entry policy
A signal saying `SELL 3523` specifies an **entry level**, not unconditional market sell. Use configured `ENTRY_AT_PRICE` behavior: wait for a valid near-price quote within max distance, within expiry and spread thresholds. Choose market/pending type based on actual bid/ask and MT5 rules. Never chase a moved market merely because the provider posted earlier.

## Multi-TP and exit accounting
- Support up to three targets initially; define TP allocation policy explicitly (equal risk/volume by default only if broker steps allow).
- Hedging: separate legs where supported. Netting: partial close manager and reconciler.
- Partial fills, partial closes, broker fees and SL changes are first-class events.
- A Telegram signal is **one** signal for signal-win-rate analysis even if three legs close separately.

## Emergency controls
- Operator kill switch blocks new orders instantly; must survive restarts.
- Gateway exposes a guarded 'pause all new orders' state independent of frontend.
- Never automatically re-send an uncertain order; reconcile by broker order/deal/position records.
- Document incident/recovery in `docs/OPERATIONS.md`.

## Boundaries for AI
AI can classify, parse and propose **risk reduction** within policy. No model gets raw broker execution credentials or a direct order tool; gateway must reject any size exceeding deterministic caps.
