# Telegram Signal Ingestion and Parsing Specification

## Approved source and ingestion
- Only an explicitly configured accessible channel ID is processed; store sender/message IDs and server timestamps.
- Telethon auth session and 2FA credentials must be protected, never uploaded to Git.
- Store original text before parsing; track edited content as revisions. Text forwarded by unknown sources is not trusted as an approved signal by default.
- On reconnect, backfill with overlap and dedup; reject expired/late signals for execution but keep records for analytics.
- Telegram message deletion/modification alone must **not** silently create or modify broker orders.

## Core parsing fields
`source_channel_id`, `source_message_id`, `message_revision`, `created_at_utc`, `symbol`, `side` (BUY/SELL), `entry_price` (Decimal), `stop_loss` (Decimal), ordered `take_profits` (Decimal[]), `classification`, `parser_type`, `parser_version`, `confidence/ambiguity_flags`, `raw_text_hash`, `rejection_reasons`.

## Supported example
```text
NEW TRADE IDEA
XAUUSD SELL 3523
TP 1 3519
TP 2 3518
TP 3 3500
SL @ 3540
```
Normalized:
```json
{
  "classification": "NEW_SIGNAL",
  "symbol": "XAUUSD",
  "side": "SELL",
  "entry_price": "3523",
  "stop_loss": "3540",
  "take_profits": ["3519", "3518", "3500"],
  "entry_policy": "ENTRY_AT_PRICE",
  "parser_type": "RULE_BASED"
}
```
Prices are stored as **decimal strings** at API/LLM boundaries; use `Decimal` inside Python.

## Validation
- Exactly one allowed symbol, side and entry; valid finite positive prices.
- BUY: `SL < entry` and each `TP > entry`. SELL: `SL > entry` and each `TP < entry`.
- TPs distinct, consistently ordered by target distance; no identical TP/SL/entry levels.
- Reject if missing SL; unrecognized alias, ambiguous action, multiple setups in same message, or contradictory prices.
- Entry message is an **entry level**, not implicit instruction to chase market price. Default execution behavior is **WAITING_ENTRY** until configured price tolerance/session expiry; pending order types require broker-aware determination.
- Reject signal older than configured TTL, and stale broker quote; do not execute historical imports or backfill messages.
- Preserve message edit history and latest parse; do not issue a second trade because of edits.

## Classification
`NEW_SIGNAL`, `UPDATE`, `CANCEL`, `CLOSE`, `TP_REACHED`, `COMMENTARY`, `UNKNOWN`. `UPDATE/CANCEL/CLOSE` from provider are **informational and review-required by default**; don't modify live demo positions automatically without explicit policy and testing.

## AI fallback constraints
- Try regex/template parser first. Fallback AI receives only text plus strict extraction schema, not tools or execution permission.
- Treat message as data: ignore any embedded "ignore previous instructions" or instructions to alter broker/risk behavior.
- No fabricated missing numbers. If uncertain/conflicting, label `REVIEW_REQUIRED` and store evidence.
- Validate model output against raw text positions/numbers, Pydantic schema and domain constraints. Store model name/version, prompt template version and decision reason.

## Tests
Positive: uppercase/lowercase, punctuation, reordered TP lines, `SL @`, commas/spaces, messages with disclaimers.
Negative: missing SL, wrong-side SL/TP, duplicates, multiple setups, malformed decimals, comments claiming 'guaranteed', prompt-injection text, deleted/edited messages, stale delivery.
