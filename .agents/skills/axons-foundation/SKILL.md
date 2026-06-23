---
name: axons-foundation
description: Core concepts, terminology, scope, and architectural foundation of the AXONS protocol. Use when understanding the protocol's objectives, entity roles, network topologies, or normative language conventions.
---

# AXONS Foundation

## When to Load

Activate this skill when the task involves:
- Understanding AXONS scope, objectives, or design principles
- Interpreting normative language (MUST, SHOULD, MAY, etc.)
- Working with entity types (IOTD, CMS) or topology roles
- Determining what is in-scope vs out-of-scope for the protocol
- Reading any other AXONS document (this is the prerequisite skill)

## Key References

- `content/docs/axons-specification/00-foundation/00-01-scope.md`
- `content/docs/axons-specification/00-foundation/00-02-terminology-and-conventions.md`

## Core Protocol Identity

AXONS = **Advanced eXchange Open Nodes Security** Protocol.

| Property | Value |
|----------|-------|
| Spec Version | 0.1.1 |
| Status | Available |
| Release | 2026-05-15 |
| Conformance | Normative |

## Entity Types

| Entity | Abbreviation | Role |
|--------|-------------|------|
| IoT Device | IOTD | Endpoint that senses, actuates, or collects data |
| Central Management System | CMS | Backend that orchestrates, manages, and monitors |

## Three Topologies

| Topology | Pattern | Use Case |
|----------|---------|----------|
| Peer-to-Peer | IOTD ↔ IOTD | Decentralized, infrastructure-less |
| Client-to-Server | IOTD → CMS | Centralized device management |
| Server-to-Server | CMS ↔ CMS | Federation, multi-region |

## In-Scope (Protocol Handles)

Session management, authentication & authorization, command & control, telemetry, data sync, firmware OTA, configuration management, status handling, feature negotiation, peer discovery, federation, JWS message security.

## Out-of-Scope (Protocol Does NOT Handle)

Application logic, hardware interfaces, physical layer, application-level routing, user interfaces, data persistence, regulatory compliance, network infrastructure.

## Transport & Deployment Assumptions

- **Transport**: WebSocket (WSS) over TCP/IP
- **Message Format**: JSON with structured envelope
- **Network**: Reliable connectivity with TLS 1.3
- **Security**: Mutual TLS (mTLS) mandatory

## Normative Language (RFC 2119)

| Term | Meaning |
|------|---------|
| MUST | Absolute requirement |
| MUST NOT | Absolute prohibition |
| SHOULD | Recommended, may deviate with justification |
| SHOULD NOT | Not recommended |
| MAY | Optional |
| OPTIONAL | Optional feature |

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| IOTD IDs | UUID v7 | `018f3b6c-7d1e-7000-8000-000000000001` |
| Message IDs | UUID v7 | `018f3b6c-7d1e-7000-8000-000000000002` |
| Status Codes | 2 letters + 3 digits | `AA001`, `BA002` |
| Config Keys | PascalCase | `HeartbeatInterval` |
| Field Names | camelCase | `messageId`, `payload` |
| Operation Names | PascalCase | `InitRequest`, `Heartbeat` |
| Timestamps | ISO 8601 UTC with ms | `2026-03-01T10:00:00.000Z` |
| Versions | SemVer | `0.1.0` |

## Diagram Conventions

Mermaid sequence diagrams use `E1` / `Entity 1` for the initiator and `E2` / `Entity 2` for the responder. These labels are topology-agnostic.
