name: axons-operations
description: AXONS core operations including session establishment (InitRequest/InitResponse), security policy exchange, subprotocol operations, and their request-response patterns. Use when implementing session flows, operation handlers, or subprotocol integration.

# AXONS Operations

## When to Load

Activate this skill when the task involves:
- Implementing session establishment (InitRequest/InitResponse)
- Security policy negotiation (PolicyExchange)
- Understanding request-response patterns and correlation
- Extending AXONS with subprotocol operations
- Error handling and status codes

## Key References

- `content/docs/axons-specification/05-application/05-04-operations.md`
- `content/docs/axons-specification/05-application/05-01-message-format.md`

## Session Establishment Flow

```
Entity 1                  Entity 2
   |                         |
   |--- InitRequest -------->|  (entityId, protocols, supportedFeatures)
   |                         |  Validate mTLS identity
   |                         |  Generate access token
   |                         |  Select highest compatible protocol
   |<-- InitResponse --------|  (sessionId, accessToken, protocol, endpoints)
   |                         |
   |--- SecurityPolicyExchangeReq ->|  (proposed security policies)
   |                         |  Apply "Most Restrictive" rule
   |<-- SecurityPolicyExchangeRes -|  (negotiated policies)
   |                         |
   |--- PolicyAcknowledgeReq ->|  (acceptedPolicies)
   |<-- PolicyAcknowledgeRes --|  (status: accepted)
   |                         |
   Session Established
```

## Core Operations

| Operation | Direction | Description |
|-----------|-----------|-------------|
| InitRequest | E1 → E2 | Session initiation with entityId, protocols |
| InitResponse | E2 → E1 | Session acceptance with token, selected protocol |
| SecurityPolicyExchangeRequest | E1 → E2 (or E2 → E1) | Propose security policy parameters |
| SecurityPolicyExchangeResponse | E2 → E1 (or E1 → E2) | Negotiated policy result |
| PolicyAcknowledgeRequest | E1 → E2 | Confirm acceptance of negotiated policy |
| PolicyAcknowledgeResponse | E2 → E1 | Policy acceptance confirmation |

## Request-Response Pattern

- Every request MUST receive a response
- Response `correlationId` MUST mirror request `messageId`
- Error responses follow JSON-RPC 2.0 error conventions (`code`, `message`, `data`)

## Subprotocol Operations

Subprotocols register their actions during negotiation. All subprotocol actions follow this naming pattern:

```
{ActionName}Request → {ActionName}Response
```

Examples:
- `TelemetryReportRequest` → `TelemetryReportResponse`
- `EventNotificationRequest` → `EventNotificationResponse`
- `ExecuteCommandRequest` → `ExecuteCommandResponse`

Subprotocol messages MUST include `accessToken` in the header.
Subprotocol messages MAY use JWS signing per their own security requirements.

## Security Policy Negotiation Rules

| Policy Key | Negotiation Rule |
|------------|-----------------|
| AccessTokenLengthBytes | Maximum Value |
| CertValidityMaxDays | Minimum Value |
| JWSKeyValidityDays | Minimum Value |
| JWSReplayWindowSeconds | Minimum Value |
| LogIntegrityEnabled | AND (Must be true) |
| LogRetentionDays | Maximum Value |
| RateLimitThreshold | Minimum Value |
| SessionTimeoutSeconds | Minimum Value |

**"Most Restrictive" Rule**: When entities exchange policies, the most restrictive value is applied to both. This prevents downgrade attacks.

## Error Handling

- All error responses use JSON-RPC 2.0 format: `{"code": <int>, "message": <string>, "data": <object>}`
- Status codes follow pattern: 2 letters + 3 digits (e.g., `AA001`)
- Unknown/enum errors MUST be rejected

## Common Pitfalls

- Sending InitRequest AFTER the session is already established → duplicate session
- Not including all required fields in InitRequest (entityId, protocols)
- Skipping PolicyExchange → non-compliant session establishment
- Applying wrong negotiation rule (using Minimum instead of Maximum or vice versa)
- Not matching response `correlationId` to request `messageId` → broken correlation
- Implementing operations in wrong order (must be: Init → PolicyExchange → Ack → Ready)
