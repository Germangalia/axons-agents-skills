---
name: axons-transport-websocket
description: WebSocket binding, connection lifecycle, subprotocol negotiation, ping/pong keep-alive, and reconnection strategies for AXONS. Use when implementing WebSocket handlers, connection state machines, or transport-level resilience.
keywords:
  - axons
tags:
  - axons
  - iot
  - protocol
---

# AXONS WebSocket Transport

## When to Load

Activate this skill when the task involves:
- WebSocket connection establishment and handshake
- Subprotocol version negotiation
- Connection state machine implementation
- Ping/pong keep-alive mechanism
- Reconnection with exponential backoff
- Message framing and queuing during disconnection

## Key References

- `content/docs/axons-specification/03-transport/03-01-websocket-binding.md`
- `content/docs/axons-specification/03-transport/03-02-access-tokens.md`

## Connection Requirements

| Requirement | Level |
|-------------|-------|
| TLS 1.3 or higher | MUST |
| Mutual TLS (mTLS) | MUST |
| Subprotocol `axons-{version}` | MUST |
| JSON-RPC 2.0 in text frames only | MUST |
| Ping/Pong keep-alive | MUST |
| Exponential backoff reconnection | MUST |
| FIFO order preserved during reconnection | MUST |

## WebSocket Handshake

**Request:**
```
GET /{base-path} HTTP/1.1
Host: {host-domain}
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: <base64-nonce>
Sec-WebSocket-Protocol: axons-0.1
Sec-WebSocket-Version: 13
```

**Response:**
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: <hash>
Sec-WebSocket-Protocol: axons-0.1
```

## Subprotocol Format

```
axons-{MAJOR}.{MINOR}
```

- Protocol and subprotocol versions MUST be strictly equal
- Entity 1 offers list of supported versions
- Entity 2 selects highest compatible version
- No match → connection rejected

## Frame Requirements

| Property | Value |
|----------|-------|
| Opcode | `0x01` (Text) |
| FIN | 1 (single frame) |
| Masked | Yes (client-to-server per RFC 6455) |
| Encoding | UTF-8 JSON |
| Max frame size | 2^63 bytes (theoretical), 1 MB (recommended) |

## Connection State Machine

```
Disconnected → Connecting → Connected → Authenticated → Authorized
                                                              ↓
Disconnected ← Closing ←───────────────────────────────── Suspended
```

| State | Description |
|-------|-------------|
| Disconnected | No active connection |
| Connecting | TCP + TLS + WS handshake in progress |
| Connected | WebSocket established, awaiting InitRequest |
| Authenticated | InitRequest sent, awaiting response |
| Authorized | Ready for normal operation |
| Suspended | Temporarily paused |
| Closing | Graceful shutdown |

## Ping/Pong Configuration

| Parameter | Default | Level |
|-----------|---------|-------|
| pingInterval | 30s | MUST |
| pongTimeout | 5s | MUST |
| maxMissedPongs | 3 | MUST |
| autoReconnect | true | MUST |

**Failure detection:**
- 1 missed pong → log warning
- 2 missed pongs → prepare reconnection
- 3+ missed pongs → close + reconnect

## Reconnection Parameters

| Parameter | Value | Level |
|-----------|-------|-------|
| Initial delay | 1s | MUST |
| Max delay | 10s | MUST |
| Backoff multiplier | 2x | MUST |
| Jitter | ±25% | MUST |
| Max retries | Unlimited | MUST |
| Reset on success | true | MUST |

**Formula:**
```
delay = min(initialDelay * 2^(attempt - 1), maxDelay)
delay_with_jitter = delay * (1 + random(-jitterFactor, jitterFactor))
```

## Topology-Specific Recovery

| Topology | Recovery Behavior |
|----------|------------------|
| Client-to-Server | Reconnect to same CMS; state resync via TriggerMessage |
| Server-to-Server | Reconnect to all peers; federation state resynchronized |
| Peer-to-Peer | Try alternative neighbors; routing table updated |

## Message Queuing During Disconnection

- All outgoing messages SHOULD be queued
- Queued messages sent when connection is restored (FIFO)
- Optional: expire messages older than configurable threshold

## Common Pitfalls

- Using binary frames instead of text frames → breaks WSS-002
- Not masking client-to-server frames → breaks RFC 6455 compliance
- Fragmented messages across multiple frames → breaks WSS-002
- Not implementing exponential backoff with jitter → thundering herd problem
- Hardcoding the subprotocol version without negotiation → breaks WSS-006
- Not handling all connection states (especially Suspended and Syncing)
- Missing message queuing during disconnection → message loss
