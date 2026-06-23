---
name: axons-pki-certificates
description: X.509 certificate management, Ed25519 keys, CA chain validation, OCSP/CRL revocation checking, and mTLS authentication for AXONS. Use when generating certificates, configuring trust stores, or validating certificate chains.
keywords:
  - axons
tags:
  - axons
  - iot
  - protocol
---

# AXONS PKI & Certificates

## When to Load

Activate this skill when the task involves:
- Generating or managing X.509 certificates for AXONS entities
- Configuring Certificate Authorities (CAs) or certificate chains
- Implementing mTLS authentication
- Setting up OCSP stapling or CRL distribution
- Certificate lifecycle management (rotation, renewal, revocation)

## Key References

- `content/docs/axons-specification/02-security/02-02-certificate-management.md`
- `content/docs/axons-specification/02-security/02-01-tls-security.md`
- `content/docs/axons-specification/01-architecture/01-01-topologies.md` §7

## Certificate Requirements

| Property | Value | Level |
|----------|-------|-------|
| Format | X.509 v3 | MUST |
| Public key algorithm | Ed25519 | MUST |
| Signature algorithm | EdDSA | MUST |
| Key size | 256 bits | MUST |
| Max validity | 398 days | MUST |
| Subject CN | UUID v7 format | MUST |

## Key Usage

| Usage | End-Entity | CA |
|-------|-----------|-----|
| digitalSignature | MUST | MUST |
| keyEncipherment | NOT USED (TLS 1.3) | NOT USED |
| keyCertSign | MUST NOT | MUST |
| cRLSign | MUST NOT | MUST |

## Extended Key Usage

| Usage | Client Role | Server Role |
|-------|-------------|-------------|
| clientAuth | MUST | MUST (P2P, S2S) |
| serverAuth | N/A | MUST (C2S) |

## mTLS Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| enabled | boolean | true | Mutual TLS on |
| verifyPeer | boolean | true | Validate peer cert |
| failIfNoPeerCert | boolean | true | Reject without cert |
| verifyDepth | integer | 3 | Max chain depth |

## Certificate Validation Order

1. **Expiration** → reject if expired
2. **Revocation** → OCSP stapling → OCSP query → CRL download
3. **Key Usage** → digitalSignature must be present
4. **Extended Key Usage** → clientAuth / serverAuth must be present
5. **Chain Validation** → verify chain to trusted CA
6. **Subject CN** → must be valid UUID v7

All checks MUST pass. If any fails, the connection is rejected.

## Validation Configuration

| Field | Type | Default | Level |
|-------|------|---------|-------|
| checkExpiration | boolean | true | MUST |
| checkRevocation | boolean | true | MUST |
| requireOcspStapling | boolean | false | MAY |
| allowWildcards | boolean | true | MAY |
| maxChainDepth | integer | 3 | MUST |

## Certificate Chain Structure

```
Root CA (offline, self-signed)
  └── Intermediate CA (online, signs end-entity certs)
       └── End-Entity Certificate (IOTD or CMS)
```

- Root CA: `digitalSignature + keyCertSign + cRLSign`
- Intermediate CA: `digitalSignature + keyCertSign + cRLSign`
- End-Entity: `digitalSignature` only

## Topology-Specific Certificate Requirements

| Topology | Certificate | Key Usage | mTLS |
|----------|------------|-----------|------|
| Client-to-Server | required | serverAuth (server), clientAuth (client) | enabled |
| Peer-to-Peer | required | clientAuth, serverAuth (both peers) | enabled |
| Server-to-Server | required | clientAuth, serverAuth (both servers) | enabled |

## Revocation Checking Fallback

OCSP Stapling (SHOULD) → OCSP Online Query → CRL Download.

If all methods fail and `checkRevocation = true`, the certificate MUST be treated as revoked.

## Common Pitfalls

- Using RSA or ECDSA keys instead of Ed25519 → breaks ATH-004
- Certificate validity > 398 days → breaks ATH-005
- Allowing self-signed certificates in production → breaks ATH-002
- Omitting OCSP/CRL check → breaks ATH-003
- Setting verifyDepth too low (<3) → breaks chain validation for deep hierarchies
- Not rotating certificates before expiry → connection failures
- Using JWT instead of opaque tokens → breaks AZN-002
