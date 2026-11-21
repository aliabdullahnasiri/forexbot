# ADR-0003 — AI Parses Signals; Code Controls Risk

**Status:** Accepted for initial releases · **Date:** 2026-11-21

## Context
Provider messages can vary in formatting and may be ambiguous, manipulated or malicious. LLMs cannot guarantee extraction accuracy or reliably enforce numeric hard limits.

## Decision
Use deterministic parsers first; optional structured-output LLM as a fallback, with Pydantic validation and source-text evidence checks. AI may propose rejection or lower risk **within** policy. Final position sizing, trade admission, order validation and demo-only enforcement are deterministic and gateway-verified.

## Consequences
Higher reliability and auditability at the cost of rejecting some valid but ambiguous messages. No model receives direct broker trading privileges.
