name: axons-security-tls
description: TLS 1.3 configuration, cipher suite negotiation, and handshake requirements for AXONS. Use when implementing transport security, configuring TLS libraries, or validating cipher suite compliance.

# AXONS TLS Security

## When to Load

Activate this skill when the task involves:
- TLS configuration (versions, cipher suites, key exchange)
- TLS handshake implementation or debugging
- Cipher suite selection for specific hardware (IoT vs server)
- Session management and resumption via tickets

## Key References

- `content/docs/axons-specification/02-security/02-01-tls-security.md`
- `content/docs/axons-specification/01-architecture/01-02-protocol-stack.md`

## Mandatory Constraints

| Rule | Level | Detail |
|------|-------|--------|
| TLS version | MUST | 1.3 only |
| TLS 1.0/1.1/1.2 | MUST NOT | All disabled |
| SSL 2.0/3.0 | MUST NOT | All disabled |
| Key exchange | MUST | ECDHE with X25519 |
| Signature algorithm | MUST | Ed25519 |
| Forward secrecy | MUST | Enabled |
| 0-RTT data | MUST NOT | Disabled (replay attack risk) |
| mTLS | MUST | Mutual certificate authentication |
| Renegotiation | MUST | Disabled |
| Compression | MUST | Disabled |

## Permitted Cipher Suites

| Cipher Suite | Level | Best For |
|-------------|-------|----------|
| TLS_CHACHA20_POLY1305_SHA256 | SHOULD | IoT without AES hardware |
| TLS_AES_256_GCM_SHA384 | MAY | Servers with AES-NI |
| TLS_AES_128_GCM_SHA256 | MAY | High-throughput servers |

## Prohibited Cipher Suites

| Cipher Suite | Reason |
|-------------|--------|
| TLS_AES_128_CCM_SHA256 | Low adoption, less efficient |
| TLS_AES_128_CCM_8_SHA256 | 8-byte tag insufficient |

## Recommended Client Ordering

| Device Type | Order |
|-------------|-------|
| IoT (no AES-NI) | ChaCha20-Poly1305, AES-256-GCM, AES-128-GCM |
| Server (with AES-NI) | AES-256-GCM, AES-128-GCM, ChaCha20-Poly1305 |
| Mixed | ChaCha20-Poly1305, AES-256-GCM, AES-128-GCM |

## Session Management

| Parameter | Value | Level |
|-----------|-------|-------|
| sessionTimeout | 7200s (2h) | MUST |
| sessionTickets | true | MUST |
| sessionTicketLifetime | 7200s | MUST |
| ticket key rotation | 3600s (1h) | MUST |
| sessionCacheSize | 10000 | SHOULD |
| reuseSessions | true | MUST |
| maxEarlyData | 0 | MUST |

## Session Resumption

- Session tickets SHOULD be used (faster reconnection without 0-RTT risks)
- Tickets encrypted with keys rotated every hour
- Maximum ticket validity: 2 hours
- 0-RTT data MUST be disabled

## Handshake Phases

1. **Key Exchange**: ClientHello + KeyShare (X25519) → ServerHello + KeyShare
2. **Server Auth**: EncryptedExtensions → CertificateRequest → Certificate → CertificateVerify → Finished
3. **Client Auth**: Certificate → CertificateVerify → Finished
4. **Application Data**: Encrypted bi-directional communication

## Code Pattern: TLS Config Concept

```
tls_config:
  min_version: TLS 1.3
  max_version: TLS 1.3
  cipher_suites:
    preferred: TLS_CHACHA20_POLY1305_SHA256
    allowed:
      - TLS_AES_256_GCM_SHA384
      - TLS_AES_128_GCM_SHA256
  require_client_cert: true
  client_ca: <trusted CA pool>
  curves: [X25519]
  signatures: [Ed25519]
  session_tickets: true
  session_timeout: 7200
  early_data: disabled
  renegotiation: disabled
  compression: disabled
```

## Common Pitfalls

- Allowing TLS 1.2 as fallback → breaks SEC-001
- Enabling 0-RTT for faster reconnects → replay attack vector
- Using RSA or ECDSA instead of Ed25519 → breaks SEC-002
- Disabling certificate revocation checks → breaks ATH-003
- Not disabling TLS compression → CRIME attack vulnerability
- Setting excessive certificate validity (>398 days) → breaks ATH-005
