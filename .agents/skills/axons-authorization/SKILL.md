---
name: axons-authorization
description: OAuth 2.0 simplified authorization model, opaque session-bound access tokens, per-message token validation, and identity extraction from mTLS certificates for AXONS. Use when implementing session management, token issuance, or authorization checks.
---

# AXONS Authorization

## When to Load

Activate this skill when the task involves:
- Session establishment and token exchange
- Access token generation and validation
- Authorization decision flow (mTLS → InitRequest → token)
- Identity extraction from X.509 certificates
- Per-message token validation

## Key References

- `content/docs/axons-specification/04-authorization/04-01-authorization-model.md`
- `content/docs/axons-specification/04-authorization/04-02-session-lifecycle.md`

## Scope

AXONS authorization operates **exclusively at the transport layer**. Application-layer authorization (roles, permissions, ACLs) is delegated to subprotocols.

| Layer | Mechanism | Purpose | Scope |
|-------|-----------|---------|-------|
| Transport | mTLS certificate validation | Verify entity identity | Connection-level |
| Session | Access token validation | Authorize session | Session-level |

## AXONS Does NOT Define

- Roles or role hierarchies
- Permission matrices
- Access control lists
- Resource authorization rules
- Application-specific scopes

## Authorization Components

| Component | Responsibility |
|-----------|---------------|
| Certificate Validator | Validates X.509 certs during mTLS |
| Identity Extractor | Extracts identity from cert subject |
| Session Manager | Manages session lifecycle and token validation |
| Authorization Context | Packages identity info for upper layers |

## Access Token Properties

| Property | Value | Level |
|----------|-------|-------|
| Type | Opaque (not JWT) | MUST |
| Length | 32+ bytes (43 chars Base64URL) | MUST |
| Encoding | Base64URL, no padding | MUST |
| Character set | `[A-Za-z0-9_-]` | MUST |
| Generation | CSPRNG | MUST |
| Lifetime | Session duration | MUST |
| Transmission | Message header `accessToken` field | MUST |
| Storage | Server-side token store | MUST |

## Authorization Flow

1. **mTLS Handshake**: Both entities present and validate X.509 certificates
2. **InitRequest**: Entity 1 sends `InitRequest` with entityId, supported protocols
3. **Token Generation**: Entity 2 generates opaque access token bound to session
4. **InitResponse**: Entity 2 responds with `sessionId`, `accessToken`, selected protocol
5. **Per-Message Validation**: Every subsequent message includes `accessToken` in header
6. **Token Revocation**: Token invalidated on disconnect or explicit revocation

## Identity Data Model

### identity

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| subjectId | UUID v7 | From cert Subject CN | `0191a7e0-...` |
| certificateThumbprint | string | SHA-256 hash of cert | `sha256:4rnc...` |
| entityType | enum | Device, Server, Peer | `Device` |
| topology | enum | ClientToServer, PeerToPeer, ServerToServer | `ClientToServer` |

### authorizationContext

| Field | Type | Description |
|-------|------|-------------|
| identity | IdentityContextType | Identity from certificate |
| sessionId | UUID v7 | Unique session identifier |
| accessToken | string | Opaque bearer token |
| authorizedAt | dateTime | When authorization established |
| expiresAt | dateTime | Token expiration |

## Per-Message Validation Rule

Every incoming request MUST include a valid `accessToken` in the header. The receiving entity validates the token before processing. The only exception is `InitRequest` (sent before token issuance).

## Conformance Requirements

| ID | Requirement | Level |
|----|-------------|-------|
| AZN-001 | Default Deny policy for all resources | MUST |
| AZN-002 | Opaque access tokens (no JWT) | MUST |
| AZN-003 | Role-Based Access Control | MUST |
| AZN-004 | Session scope validation on every request | MUST |
| AZN-005 | Immediate session termination on token revocation | MUST |
| AZN-006 | Dynamic scope updates without reconnection | SHOULD |

## Common Pitfalls

- Using JWT instead of opaque tokens → breaks AZN-002 (JWT exposes metadata, cannot be instantly revoked)
- Implementing application-level authorization at the AXONS layer → should be delegated to subprotocol
- Not validating access token on every message → breaks AZN-004
- Allowing tokens to persist after session disconnect → breaks AZN-005
- Using self-reported identity instead of certificate-extracted identity → security risk
- Not binding tokens to specific sessions → token theft across sessions