name: axons-authorization
description: OAuth 2.0 simplified authorization model for AXONS session-based authentication, opaque access tokens, per-message validation, and identity extraction from mTLS certificates.

# AXONS Authorization

## When to Load

Activate this skill when the task involves:
- Session establishment and token exchange
- Access token generation and validation
- Authorization decision flow (mTLS → InitRequest → token issuance)
- Identity extraction from X.509 certificates
- Per-message token validation

## Key Concepts

AXONS authorization operates at the transport/session boundary:

- Identity is derived from mTLS X.509 certificates
- Sessions are bound to opaque access tokens
- Every message (except InitRequest) requires token validation

## Authorization Flow

1. mTLS handshake establishes secure identity
2. InitRequest is sent with entityId and protocol capabilities
3. Server issues sessionId + opaque accessToken
4. All subsequent messages include accessToken
5. Token is revoked on disconnect or explicit termination

## Access Token Rules

- MUST be opaque (not JWT)
- MUST be cryptographically random
- MUST be session-bound
- MUST be validated per message
- MUST be revoked on session end

## Identity Model

- Identity is extracted ONLY from X.509 certificates
- No self-declared identity is allowed
- Each identity is bound to a session lifecycle

## Scope Rules

AXONS authorization defines ONLY:

- Transport-level authentication
- Session-level authorization

It does NOT define:

- Roles
- Permissions
- ACLs
- Application-level access control

## Conformance Rules

- Default deny all requests (unless authenticated)
- Validate access token on every message
- Terminate session on token revocation
- Bind token strictly to session context

## Common Mistakes

- Using JWT instead of opaque tokens
- Skipping per-message validation
- Allowing cross-session token reuse
- Implementing role systems inside AXONS layer