name: axons-message-format
description: JSON-RPC 2.0 message envelope, header fields, payload structure, JWS compact serialization signing, and subprotocol message extension for AXONS. Use when constructing or parsing AXONS messages or implementing JWS signing.

# AXONS Message Format

## When to Load

Activate this skill when the task involves:
- Constructing or parsing AXONS messages
- Implementing JWS signing or signature verification
- Understanding header fields and their purposes
- Extending the message format for subprotocols
- Message canonicalization for deterministic signing

## Key References

- `content/docs/axons-specification/05-application/05-01-message-format.md`
- `content/docs/axons-specification/05-application/05-02-jws-signing.md`

## Envelope Structure

```
{
  "data": {
    "header": { ... },
    "payload": { ... }
  },
  "jws": "BASE64URL(header).BASE64URL(payload).BASE64URL(signature)"  // optional
}
```

| Field | Cardinality | Description |
|-------|-------------|-------------|
| `data` | 1 | Contains header and payload |
| `jws` | 0..1 | JWS compact serialization string |

## Header Fields

| Field | Type | Cardinality | Description |
|-------|------|-------------|-------------|
| `messageId` | UUID v7 | 1 | Unique message identifier |
| `action` | string | 1 | Operation name (e.g., `InitRequest`) |
| `timestamp` | dateTime | 1 | ISO 8601 UTC with milliseconds |
| `version` | string | 1 | AXONS protocol version (e.g., `axons-0.1`) |
| `protocol` | string | 1 | Negotiated subprotocol (e.g., `abcde-1.0`) |
| `accessToken` | string | 0..1 | Bearer credential (all messages except InitRequest) |
| `correlationId` | UUID v7 | 0..1 | Links response to request (mirrors request messageId) |

## Access Token Rule

- Present in every message EXCEPT `InitRequest`
- Obtained during session establishment (InitRequest/InitResponse)
- Validated on every message by the receiving entity

## Request vs Response Headers

| Aspect | Request | Response |
|--------|---------|----------|
| `messageId` | New UUID v7 | New UUID v7 (different) |
| `action` | Operation name | Same as request |
| `accessToken` | Sender token | Sender token |
| `correlationId` | Not included | MUST match request `messageId` |
| `payload` | Operation params | Operation result |

## JWS Signing

### Mandatory Properties

| Property | Value | Level |
|----------|-------|-------|
| Algorithm | EdDSA (Ed25519) | MUST |
| Key size | 256 bits | MUST |
| Header fields | `alg`, `kid`, `typ` | MUST |
| Payload | Full message | MUST |
| Encoding | Base64URL | MUST |
| Format | Compact serialization | MUST |

### JWS Header Example

```json
{
  "alg": "EdDSA",
  "kid": "0194a21c-7d1e-7000-8000-000000000001",
  "typ": "JWS"
}
```

### JWS Compact Serialization

```
BASE64URL(header) '.' BASE64URL(payload) '.' BASE64URL(signature)
```

## Message Categories & JWS Requirements

| Category | JWS Required | Examples |
|----------|-------------|----------|
| Session | No | InitRequest, Heartbeat |
| Policy | Recommended | SecurityPolicyExchange |
| Subprotocol | Defined by subprotocol | Domain-specific messages |
| Configuration | Recommended | ChangeConfiguration |
| Critical Operations | Recommended | MissionAuthorize, RemoteCommand |
| Firmware | Recommended | UpdateFirmware |

## JSON Canonicalization for Signing

Because JSON does not define a canonical representation, use these rules before signing:
- Alphabetical key ordering
- No whitespace
- UTF-8 encoding
- Normalized number formats

## Conformance Requirements

| ID | Requirement | Level |
|----|-------------|-------|
| MSG-001 | JWS Compact Serialization | MUST |
| MSG-002 | EdDSA with Ed25519 | MUST |
| MSG-003 | Protected header: `alg`, `kid`, `iat` | MUST |
| MSG-004 | UUID v7 for `kid` | MUST |
| MSG-005 | Replay protection (messageId + timestamp, 300s window) | MUST |
| MSG-006 | Mandatory signing for safety-critical commands | SHOULD |
| MSG-007 | Key rotation with grace period | SHOULD |

## Common Pitfalls

- Omitting `accessToken` in non-InitRequest messages → breaks session validation
- Including `accessToken` in InitRequest → token doesn't exist yet
- Forgetting `correlationId` in responses → breaks request-response matching
- Skipping JSON canonicalization before JWS signing → signature verification fails
- Using JWS JSON Serialization instead of Compact → breaks MSG-001
- Using `alg: "HS256"` or other HMAC → must use EdDSA
- Not including `kid` in JWS header → key lookup fails
- Setting invalid `timestamp` format (missing milliseconds or Z suffix)
