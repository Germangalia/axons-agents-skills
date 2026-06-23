AXONS Agents & Skills

A structured knowledge base for AI coding agents building, deploying, and testing the AXONS protocol (Advanced eXchange Open Nodes Security).

AXONS is a secure IoT and distributed systems communication framework designed for machine-to-machine and agent-to-agent communication using WebSocket (WSS), TLS 1.3, mutual TLS (mTLS), and signed message exchange.

This repository is published as a skills.sh skill registry for AI agent runtimes.

🔑 Keywords (for search indexing)

axons, axons agents, axons protocol, skills, iot, internet of things, websocket, wss, tls, tls 1.3, mtls, mutual tls, pki, x509, ed25519, oauth2, json-rpc, secure messaging, distributed systems, event-driven architecture, protocol design, device communication, agent framework, message signing, audit logging, resilience, networking, automation

📌 Repository Overview

This repository is a capability registry for AI agents.

It defines:

Skills — modular knowledge units used by AI agents
Agents — role-based AI personas that compose skills into execution behavior

There is no runtime application code. This repository is purely a structured knowledge system optimized for AI skill loading platforms such as skills.sh.

📁 Repository Structure

All skills and agents are stored under the .agents/ directory:

.agents/
├── skills/
│   ├── axons-foundation/
│   ├── axons-topologies/
│   ├── axons-transport-websocket/
│   ├── axons-security-tls/
│   ├── axons-pki-certificates/
│   ├── axons-authorization/
│   ├── axons-message-format/
│   ├── axons-operations/
│   ├── axons-data-types/
│   ├── axons-services-resilience/
│   ├── axons-monitoring-logging/
│   ├── axons-compliance/
│   └── neon-postgres/
│
└── agents/
    ├── protocol-architect/
    ├── application-developer/
    ├── security-implementer/
    ├── transport-developer/
    ├── service-developer/
    ├── test-engineer/
    ├── integration-specialist/
    └── documentation-writer/
🧠 AXONS Protocol (Core Concept)

AXONS (Advanced eXchange Open Nodes Security) is a secure communication protocol for IoT systems and distributed agents.

It is designed for secure, verifiable, and observable machine-to-machine communication.

Core Features
WebSocket (WSS) transport layer
TLS 1.3 encrypted communication channels
Mutual TLS (mTLS) device authentication
Ed25519 / JWS message signing
OAuth 2.0-style authorization model
JSON-RPC 2.0 structured messaging
Replay protection mechanisms
Audit logging and traceability

Protocol Version: 0.1.1 (2026-05-15)

🧩 Skills

Each skill is a modular knowledge unit designed for AI agent retrieval and composition.

Skill	Domain
axons-foundation	Core concepts, terminology, identity model
axons-topologies	Network architectures (P2P, C2S, S2S, gateways)
axons-transport-websocket	WebSocket/WSS transport, session lifecycle
axons-security-tls	TLS 1.3 configuration and encryption flows
axons-pki-certificates	X.509, CA chains, Ed25519, mTLS lifecycle
axons-authorization	Token validation and message-level auth
axons-message-format	JSON-RPC 2.0 + JWS signing + replay protection
axons-operations	Session lifecycle and protocol state machines
axons-data-types	UUID v7, Base64URL, ISO 8601, canonical formats
axons-services-resilience	Retry logic, queues, deduplication, DLQ
axons-monitoring-logging	Structured logs, audit trails, observability
axons-compliance	Certification rules and conformance testing
neon-postgres	Database integration for AXONS systems
🤖 Agents

Role-based AI agents composed from AXONS skills:

protocol-architect — system design, topology selection, distributed architecture planning
application-developer — JSON-RPC flows, message signing, session handling
security-implementer — TLS, PKI, authentication, secure communication design
transport-developer — WebSocket connection handling and state machines
service-developer — queues, retries, logging, resilience patterns
test-engineer — conformance testing and protocol validation
integration-specialist — gateway and federation design
documentation-writer — tutorials, API references, protocol documentation
⚡ Installation (skills.sh)

Install the full skill set via registry:

npx skills add Germangalia/axons-agents-skills

Search skills:

npx skills search axons
npx skills search websocket
npx skills search tls
npx skills search iot

Note: Search indexing may take time depending on registry propagation, keyword weighting, and install signals.

⚙️ Local Usage

Clone the repository:

git clone https://github.com/Germangalia/axons-agents-skills.git
cd axons-agents-skills

The runtime should load skills from:

.agents/skills/

and agents from:

.agents/agents/
🔍 Discoverability Notes (skills.sh)

If skills do not appear in search results immediately, this may be due to:

delayed indexing of new repositories
keyword weighting and ranking heuristics
low install/usage signals
insufficient explicit keyword mapping inside skills

This repository is optimized for indexing via:

repeated semantic keywords
explicit AXONS protocol references
IoT + security + transport domain tagging
🧠 Design Philosophy

AXONS Skills are designed to be:

modular and composable
agent-readable and retrieval-friendly
protocol-first (not application-first)
secure-by-design
optimized for IoT and distributed systems
compatible with AI orchestration platforms
📌 License

Not yet declared.