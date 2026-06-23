---
name: axons-monitoring-logging
description: Structured JSON logging, hash-chained audit trails, periodic log signing, rate limiting, connection monitoring, and graceful degradation for AXONS. Use when implementing observability, security audit, or operational monitoring.
---

# AXONS Monitoring & Logging

## When to Load

Activate this skill when the task involves:
- Implementing audit logging with hash chain integrity
- Setting up rate limiting (per-entity, tiered, adaptive)
- Connection health monitoring and heartbeats
- Graceful degradation under load or failure
- Security event alerting

## Key References

- `content/docs/axons-specification/06-services/06-03-monitoring.md`
- `content/docs/axons-specification/06-services/06-04-logging.md`

## Audit Log Requirements

| ID | Requirement | Level |
|----|-------------|-------|
| MON-002 | Log all Authentication and Authorization events | MUST |
| MON-003 | Implement tamper-evident log integrity | SHOULD |
| MON-006 | Redact PII and credentials from all logs | MUST |
| AUD-001 | All security events MUST be recorded | MUST |
| AUD-002 | Audit logs MUST be append-only and immutable | MUST |
| AUD-003 | Include cryptographically verifiable chain | MUST |
| AUD-004 | Accurate, synchronized timestamps | MUST |

## Audit Log Entry Schema

| Field | Type | Description |
|-------|------|-------------|
| entryId | UUID v7 | Unique entry identifier |
| timestamp | dateTime | ISO 8601 UTC |
| eventType | string | Type of event |
| source | string | Component generating event |
| actor | string | Identity of acting entity |
| action | string | Action performed |
| resource | string | Resource affected |
| outcome | enum | success, failure, denied |
| hash | string | Hash of current entry |
| previousHash | string | Hash of previous entry |

## Hash Chain Integrity

Each new entry includes:
- `hash`: SHA-256 of (previousHash + entry data)
- `previousHash`: hash from prior entry

This creates a tamper-evident chain. Modifying any entry invalidates all subsequent hashes.

## Periodic Log Signing

- Hash chaining detects internal tampering
- Periodic signing creates non-repudiable checkpoints
- Sign final hash of batch using Ed25519 private key in secure hardware
- Once signed and stored externally, history cannot be rewritten

## Rate Limiting

### Tiered Rate Limiting

| Message Tier | Priority | Limit Treatment |
|-------------|----------|----------------|
| Critical | Highest | Most generous limits |
| Standard | High | Moderate limits |
| Telemetry | Medium | Reduced limits |
| Bulk | Lowest | Most restrictive |

### Token Bucket Algorithm

- Allows temporary bursts up to bucket size
- Enforces long-term rate via token replenishment
- Burst tokens consumed first, then base tokens

### Adaptive Rate Limiting

Rate limits dynamically adjust based on system resource utilization (CPU, memory, queue depth). When thresholds exceeded, throughput limits reduced proportionally.

## Connection Health Monitoring

| Mechanism | Description |
|-----------|-------------|
| WebSocket Ping/Pong | Native keep-alive (30s interval) |
| Heartbeat messages | Application-level liveness |
| Missed pong detection | 3 missed pongs → disconnect + reconnect |

## Graceful Degradation Levels

| Level | Description |
|-------|-------------|
| Full Service | All features operational |
| Reduced Features | Non-critical features disabled |
| Essential Only | Only critical operations |
| Offline Mode | Local operation, no network |

## Conformance Requirements

| ID | Requirement | Level |
|----|-------------|-------|
| MON-001 | WebSocket liveness monitoring (Heartbeat) | MUST |
| MON-002 | Log all AuthN/AuthZ events | MUST |
| MON-003 | Tamper-evident log integrity | SHOULD |
| MON-004 | Rate limiting per-entity | MUST |
| MON-005 | Alert on security-critical events | MUST |
| MON-006 | Redact PII from logs | MUST |

## Common Pitfalls

- Logging access tokens or credentials → breaks MON-006
- Not implementing hash chaining → logs can be tampered without detection
- Using mutable log storage → breaks AUD-002
- Not rate limiting per-entity → DoS vulnerability
- Applying same rate limit to all message tiers → critical ops may be throttled
- Not alerting on certificate expiry events → unexpected disconnections
- Not synchronizing log timestamps → broken audit trail
