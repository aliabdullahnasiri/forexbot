# ADR-0002 — Windows MT5 Terminal Gateway

**Status:** Proposed default · **Date:** 2026-11-21

## Context
Official MetaTrader 5 Python integration communicates with an installed terminal. A headless Ubuntu VPS should not be assumed to run the native terminal reliably or directly speak to brokers over a generic REST interface.

## Decision
Run the main application on Ubuntu and a restricted Windows gateway next to an installed MT5 DEMO terminal. Use private authenticated transport with allowlisted clients and independent gateway risk/account validation. The gateway exposes read-only broker state and explicitly guarded demo mutation endpoints.

## Alternatives considered
- All-in-one Windows host: simpler integration but less convenient for Linux-based backend operations.
- MT5 through Wine on Ubuntu: possible in some environments, but broker/terminal compatibility and supervision are higher risk; not the default deployment.

## Consequences
Additional Windows host and private connectivity are required, but the trading boundary is explicit and easier to audit.
