# ADR-002: Cloud Relay Technology Choice

## Status: Accepted

## Context
The cloud relay must proxy OpenAI-compatible API requests with minimal latency, support SSE streaming, and accept config updates from the iOS app via WebSocket.

## Decision
**Node.js + Express + undici** on a lightweight container (Fly.io or Railway).

## Rationale
- Same language as main 9Router — code sharing possible
- undici provides fast HTTP/2 upstream connections with streaming
- Express 5 is stable and well-understood
- WebSocket (ws) for real-time config sync from iOS
- Fly.io provides edge deployment close to users, auto-scaling, $0 idle

## Alternatives Considered
- **Cloudflare Workers**: No WebSocket server support for persistent config sync
- **Bun**: Less mature for production, but could swap later
- **Go**: Better performance, but splits the codebase language
- **Rust (axum)**: Overkill for a thin proxy layer
