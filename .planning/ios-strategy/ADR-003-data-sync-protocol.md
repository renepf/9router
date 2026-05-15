# ADR-003: Data Sync Protocol (iOS <-> Relay)

## Status: Accepted

## Context
The iOS app must push routing configuration to the cloud relay and receive usage stats back.

## Decision
**WebSocket with JSON messages** using a simple command/event protocol.

## Protocol
```
iOS -> Relay (commands):
  { type: "config:update", payload: { providers, combos, rules } }
  { type: "config:replace", payload: { ...fullConfig } }
  { type: "ping" }

Relay -> iOS (events):
  { type: "config:ack", payload: { version } }
  { type: "usage:report", payload: { provider, model, tokens, cost, timestamp } }
  { type: "error", payload: { code, message } }
  { type: "pong" }
```

## Rationale
- WebSocket keeps persistent connection for instant config pushes
- JSON is simple, debuggable, and native to both Swift and Node.js
- Stateless relay: config lives in memory, iOS is source of truth
- On reconnect, iOS sends full config:replace

## Alternatives Considered
- **REST polling**: Higher latency, more battery usage on iOS
- **gRPC**: Overkill, adds protobuf complexity
- **MQTT**: Good for IoT, unnecessary here
