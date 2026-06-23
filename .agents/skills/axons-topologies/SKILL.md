---
name: axons-topologies
description: AXONS network topology specifications including Peer-to-Peer (P2P), Client-to-Server (C2S), Server-to-Server (S2S) federation, and gateway patterns. Use when designing deployment architecture or implementing topology-specific behavior.
---

# AXONS Topologies

## When to Load

Activate this skill when the task involves:
- Selecting or designing a network topology for AXONS deployment
- Implementing topology-specific discovery, connection, or recovery logic
- Configuring gateways for NAT traversal or protocol translation
- Peer-to-Peer network formation and maintenance
- Server-to-Server federation setup

## Key References

- `content/docs/axons-specification/01-architecture/01-01-topologies.md`
- `content/docs/axons-specification/01-architecture/01-02-protocol-stack.md`

## Topology Comparison

| Factor | Peer-to-Peer | Client-to-Server | Server-to-Server |
|--------|-------------|------------------|------------------|
| Central infra required | No | Yes | Yes |
| SPOF tolerance | High | Low | Medium |
| Latency | Lowest | Medium | Higher |
| Management complexity | High | Low | Medium |
| Scalability | Horizontal | Vertical + Horizontal | Horizontal |
| Network | Local/P2P net | Internet/Intranet | Internet/Intranet |
| Discovery | Distributed | Server-managed | Federation protocol |

## Selection Logic

```
Central infrastructure available?
├── No  → Peer-to-Peer
└── Yes → Multiple independent servers needed?
           ├── No  → Client-to-Server
           └── Yes → Server-to-Server
```

## Peer-to-Peer (P2P)

**Pattern**: IOTD ↔ IOTD (direct, symmetric, mesh)

**Key Characteristics**:
- Decentralization: no single point of control or failure
- Self-organization: adapts to node joins/leaves
- Resilience: remains functional with partial connectivity
- Low latency: direct communication paths

**Use Cases**: Infrastructure-less environments, real-time coordination, resilient networks, edge computing, ad-hoc networks.

**Discovery**: Uses Wi-Fi Direct for local peer discovery. No central infrastructure required.

## Client-to-Server (C2S)

**Pattern**: IOTD → CMS (initiator-responder, hub-and-spoke)

**Key Characteristics**:
- Centralized control: CMS manages all connections
- Simplified management: single point for config and monitoring
- Audit trail: complete visibility of all communications

**Use Cases**: Centralized management, device fleet management, data aggregation, command and control, audit and compliance.

**Constraint**: Each IOTD MUST connect to exactly one CMS at a time (prevents split-brain).

## Server-to-Server (S2S)

**Pattern**: CMS ↔ CMS (federation)

**Key Characteristics**:
- Federation: independent servers cooperate, maintain autonomy
- Interoperability: cross-domain communication
- Scalability: horizontal scaling through server distribution
- Resilience: geographic and organizational redundancy

**Use Cases**: Multi-region operations, cross-organization collaboration, horizontal scaling, high availability, data sovereignty.

## Gateway Role

A Gateway is an intermediary service that forwards messages between entities that cannot directly connect.

### Gateway Types

| Type | Function |
|------|----------|
| Message Routing | Forward messages without modification, load balancing |
| Protocol Translation | Convert between AXONS and external protocols |
| Aggregation | Combine data from multiple sources, compression, batching |

### Gateway Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| GatewayType | enum | MessageRouting | Type of gateway |
| BackendEntities | string[] | — | Backend entity endpoints |
| ConnectionPoolSize | integer | 10 | Connections per backend |
| TimeoutMs | integer | 30000 | Request timeout |
| RetryAttempts | integer | 3 | Retry attempts on failure |

### Topology-Specific Gateway Behavior

- **C2S with Gateway**: Gateway terminates IOTD connections, forwards to CMS, may aggregate data
- **S2S with Gateway**: Federates connections between CMS instances in different domains
- **P2P with Gateway**: Relays P2P messages when direct connections impossible

## Certificate Requirements Per Topology

| Topology | Certificates | mTLS |
|----------|-------------|------|
| Client-to-Server | serverAuth (server), clientAuth (client) | enabled |
| Peer-to-Peer | clientAuth + serverAuth (each peer) | enabled |
| Server-to-Server | clientAuth + serverAuth (each server) | enabled |

## Common Pitfalls

- Applying Client-to-Server single-connection rule to Peer-to-Peer topology
- Expecting central discovery in Peer-to-Peer → uses distributed discovery
- Not implementing federation handshake for Server-to-Server connections
- Mixing topology-specific state synchronization strategies
- Forgetting that IOTDs in P2P need both clientAuth and serverAuth EKU
- Not configuring gateways with proper NAT traversal
- Assuming all topologies support the same reconnection strategy