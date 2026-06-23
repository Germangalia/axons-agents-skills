# AXONS Agents & Skills

A structured knowledge base for AI coding agents building, deploying, and testing the **AXONS protocol** (Advanced eXchange Open Nodes Security) — a secure communication protocol for IoT systems.

## Overview

This repository is a **capability registry** for AI-assisted development environments. It defines:

- **Skills** — Markdown reference documents that teach an AI agent the AXONS protocol specification, domain by domain.
- **Agents** — Specialized AI agent personas that compose skills into focused roles (architect, security implementer, test engineer, etc.).

No executable code lives here. This is pure knowledge, structured for agent consumption.

## Skills

| Skill | Covers |
|-------|--------|
| **axons-foundation** | Core concepts, terminology, network entity types, scope |
| **axons-topologies** | P2P, C2S, S2S, gateway patterns, deployment blueprints |
| **axons-transport-websocket** | WebSocket binding, subprotocol negotiation, reconnection |
| **axons-security-tls** | TLS 1.3 ciphers, handshake, session management |
| **axons-pki-certificates** | X.509, Ed25519, CA chains, OCSP/CRL, mTLS |
| **axons-authorization** | OAuth 2.0 simplified model, opaque tokens, per-message validation |
| **axons-message-format** | JSON-RPC 2.0, JWS compact serialization, replay protection |
| **axons-operations** | Session flows (InitRequest/InitResponse), policy exchange, error handling |
| **axons-data-types** | UUID v7, CiString, Base64URL, ISO 8601, enumerations |
| **axons-services-resilience** | At-least-once queuing, FIFO, dedup, dead letter handling |
| **axons-monitoring-logging** | Structured JSON logging, hash-chained audit trails, rate limiting |
| **axons-compliance** | Certification levels, requirement IDs, conformance testing |
| **neon-postgres** | Neon Serverless Postgres — connection methods, SDKs, CLI |

## Agents

| Agent | Specialty |
|-------|-----------|
| **protocol-architect** | System architecture, topology selection, deployment blueprints |
| **application-developer** | JSON-RPC envelopes, JWS signing, session establishment |
| **security-implementer** | TLS configs, certificate management, authorization code |
| **transport-developer** | WebSocket handlers, connection state machines, reconnection |
| **service-developer** | Message queuing, audit logging, rate limiting |
| **test-engineer** | Conformance test suites, certification procedures |
| **integration-specialist** | Gateway configs, protocol translation, cross-domain federation |
| **documentation-writer** | Tutorials, API references, deployment guides |

## Usage

This repository is consumed by agent platforms (e.g., OpenCode) that support skill-based knowledge loading. Skills are loaded on demand when an agent encounters a matching task. The `skills-lock.json` manifest records integrity hashes for each skill.

To use locally:

```bash
git clone https://github.com/Germangalia/axons-agents-skills.git
# Point your agent platform's skill directory at .agents/skills/
```

## About the AXONS Protocol

AXONS (**A**dvanced e**X**change **O**pen **N**odes **S**ecurity) is a specification for secure IoT communications over WebSocket (WSS) with mandatory mTLS, Ed25519/JWS message signing, and OAuth 2.0-based authorization. Version **0.1.1** (released 2026-05-15). See `axons-foundation` for the full introduction.

## License

Not yet declared.
