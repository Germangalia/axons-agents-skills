name: axons-compliance
description: AXONS conformance framework, certification levels, requirement ID system, and compliance testing procedures. Use when validating implementation compliance or preparing for certification.

# AXONS Compliance & Conformance

## When to Load

Activate this skill when the task involves:
- Validating an AXONS implementation against the specification
- Preparing for certification (Basic, Standard, Advanced)
- Running conformance tests
- Understanding requirement levels (MUST, SHOULD, MAY)
- Auditing security or operational compliance

## Key References

- `content/docs/axons-specification/07-compliance/07-01-conformance.md`
- `content/docs/axons-specification/07-compliance/07-02-best-practices.md`
- `content/docs/axons-specification/07-compliance/07-03-technical-decisions.md`

## Compliance Levels

| Symbol | Level | Description |
|--------|-------|-------------|
| [M] | MANDATORY | Must implement for any compliance |
| [S] | RECOMMENDED | Should implement unless documented justification |
| [O] | OPTIONAL | May implement based on use case |

## Certification Levels

| Level | Requirements | Verification |
|-------|-------------|-------------|
| Basic | All [M] | Self-assessment |
| Standard | All [M] + [S] | Third-party audit |
| Advanced | All [M] + [S] + [O] | Third-party audit + Penetration test |

## Requirement ID System

| Prefix | Domain |
|--------|--------|
| STK | Protocol Stack & Architecture |
| WSS | WebSocket Transport & Handshake |
| SEC | Transport Security (TLS, Ciphers) |
| ATH | Authentication (mTLS, Certificates) |
| AZN | Authorization & Access Control |
| MSG | Message Format & JWS Integrity |
| CFG | Configuration & Policy Negotiation |
| MON | Monitoring, Logging & Operations |
| TOP | Topology-Specific Requirements |
| TST | Compliance Testing Procedures |
| AUD | Audit and Compliance Requirements |

## Protocol Stack [STK]

| ID | Requirement | Level |
|----|-------------|-------|
| STK-001 | Implement four-layer protocol stack | [M] |
| STK-002 | Support at least one topology | [M] |
| STK-003 | Use JSON-RPC 2.0 envelope | [M] |
| STK-004 | UTF-8 encoding | [M] |
| STK-005 | UUID v7 for identifiers | [M] |

## WebSocket Transport [WSS]

| ID | Requirement | Level |
|----|-------------|-------|
| WSS-001 | WSS exclusively (no ws://) | [M] |
| WSS-002 | Text frames only | [M] |
| WSS-003 | Ping/Pong keep-alive | [M] |
| WSS-004 | Exponential backoff reconnection | [M] |
| WSS-005 | Offline message queuing | [M] |
| WSS-006 | Subprotocol negotiation | [S] |
| WSS-007 | Connection migration / session resumption | [S] |

## Transport Security [SEC]

| ID | Requirement | Level | Constraint |
|----|-------------|-------|------------|
| SEC-001 | TLS 1.3 exclusively | [M] | Disable older versions |
| SEC-002 | Ed25519 for TLS signatures | [M] | Fast, IoT-ready |
| SEC-003 | ECDHE with X25519 | [M] | Forward secrecy |
| SEC-004 | ChaCha20-Poly1305 | [S] | Recommended for IoT |
| SEC-005 | AES-256-GCM | [M] | Mandatory for servers |
| SEC-006 | Disable renegotiation + compression | [M] | Attack hardening |
| SEC-007 | Disable 0-RTT | [M] | Replay prevention |

## Authentication [ATH]

| ID | Requirement | Level |
|----|-------------|-------|
| ATH-001 | mTLS for all connections | [M] |
| ATH-002 | Validate X.509 v3 chain | [M] |
| ATH-003 | Revocation checking (OCSP, CRL fallback) | [M] |
| ATH-004 | Ed25519 for certificate keys | [M] |
| ATH-005 | Certificate validity per best practices | [M] |
| ATH-006 | OCSP Stapling | [S] |
| ATH-007 | Bind tokens to entity certificate | [M] |

## Authorization [AZN]

| ID | Requirement | Level |
|----|-------------|-------|
| AZN-001 | Default Deny policy | [M] |
| AZN-002 | Opaque access tokens (no JWT) | [M] |
| AZN-003 | RBAC | [M] |
| AZN-004 | Per-request scope validation | [M] |
| AZN-005 | Immediate termination on revocation | [M] |
| AZN-006 | Dynamic scope updates | [S] |

## Message Integrity [MSG]

| ID | Requirement | Level |
|----|-------------|-------|
| MSG-001 | JWS Compact Serialization | [M] |
| MSG-002 | EdDSA with Ed25519 | [M] |
| MSG-003 | Protected header: `alg`, `kid`, `iat` | [M] |
| MSG-004 | UUID v7 for `kid` | [M] |
| MSG-005 | Replay protection (messageId + timestamp) | [M] |
| MSG-006 | Mandatory signing for safety-critical | [S] |
| MSG-007 | Key rotation with grace period | [S] |

## Configuration [CFG]

| ID | Requirement | Level |
|----|-------------|-------|
| CFG-001 | PolicyExchange during session | [M] |
| CFG-002 | Most Restrictive negotiation | [M] |
| CFG-003 | Persist config across restarts | [M] |
| CFG-004 | Validate schema before applying | [M] |

## Monitoring [MON]

| ID | Requirement | Level |
|----|-------------|-------|
| MON-001 | WebSocket liveness monitoring | [M] |
| MON-002 | Log all AuthN/AuthZ events | [M] |
| MON-003 | Tamper-evident log integrity | [S] |
| MON-004 | Per-entity rate limiting | [M] |
| MON-005 | Alert on security-critical events | [M] |
| MON-006 | Redact PII from logs | [M] |

## Key Compliance Tests

| Test ID | Validates | Procedure | Expected |
|---------|-----------|-----------|----------|
| TST-SEC-001 | SEC-001 | Attempt TLS 1.2 | REJECTED |
| TST-ATH-001 | ATH-001 | Connect without client cert | REJECTED |
| TST-ATH-002 | ATH-005 | Present over-max-validity cert | REJECTED |
| TST-AZN-001 | AZN-002 | Present JWT instead of opaque | FAILED |
| TST-MSG-001 | MSG-005 | Replay captured signed message | REJECTED |
| TST-CFG-001 | CFG-002 | Negotiate conflicting policies | Most restrictive chosen |
| TST-MON-001 | MON-004 | Exceed RateLimitThreshold | 429 Too Many Requests |

## Audit Requirements [AUD]

| ID | Requirement | Level |
|----|-------------|-------|
| AUD-001 | Record all security events | [M] |
| AUD-002 | Append-only, immutable logs | [M] |
| AUD-003 | Cryptographically verifiable chain | [M] |
| AUD-004 | Synchronized timestamps | [M] |
| AUD-005 | Compliance reporting on demand | [S] |

## Common Pitfalls

- Claiming compliance without implementing all [M] requirements
- Using self-assessment when third-party audit is needed (Standard certification)
- Not testing TLS version fallback → connection accepted on TLS 1.2
- Missing revocation checking → accepting revoked certificates
- Not understanding the difference between [S] and [O] requirements
- Skipping PolicyExchange → CFG-001 violation
- Not persisting security configurations → CFG-003 violation
