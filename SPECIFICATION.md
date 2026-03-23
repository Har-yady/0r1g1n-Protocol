# 0r1g1n Framework Specification

**Open Provenance Verification Protocol**

**Version:** 0.2.0-draft | **Date:** March 2026 | **Status:** DRAFT — not for production use  
**Licence:** Apache 2.0 | **Repository:** github.com/0r1g1n-protocol | **Supersedes:** 0.1.0-draft

> This is version 0.2.0-draft published for community review and comment. Interfaces, data formats,
> and protocol behaviours are subject to change before a stable release.
> Do not implement against this specification in production systems.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted
as described in RFC 2119.

---

## Contents

1. [Introduction](#1-introduction)
2. [Definitions and Terminology](#2-definitions-and-terminology)
3. [Threat Model (Normative)](#3-threat-model-normative)
4. [System Architecture](#4-system-architecture)
5. [Layer 1 — C2PA Capture and Signing](#5-layer-1--c2pa-capture-and-signing)
6. [Layer 2 — Verifiable Data Registry](#6-layer-2--verifiable-data-registry)
7. [Layer 3 — Chainlink Oracle Verification](#7-layer-3--chainlink-oracle-verification)
8. [Layer 4 — On-Chain Verifier Contract](#8-layer-4--on-chain-verifier-contract)
9. [Layer 5 — Soft Binding and Watermark Recovery](#9-layer-5--soft-binding-and-watermark-recovery)
10. [Layer 6 — Privacy Tier (Optional)](#10-layer-6--privacy-tier-optional)
11. [Verification SDK Interface](#11-verification-sdk-interface)
12. [Conformance](#12-conformance)
13. [Failure Modes and Recovery](#13-failure-modes-and-recovery)
14. [Governance and Trust Onboarding](#14-governance-and-trust-onboarding)
15. [Security Considerations](#15-security-considerations)
16. [Acknowledgements](#16-acknowledgements)
- [Appendix A — Data Schemas](#appendix-a--data-schemas)
- [Appendix B — Error Codes](#appendix-b--error-codes)
- [Appendix C — Changelog](#appendix-c--changelog)

---

## 1. Introduction

### 1.1 Purpose

0r1g1n is an open framework specification for verifiable content provenance. It defines the
interfaces, data formats, and protocol behaviours required to establish cryptographically verifiable
chains of custody for digital media from the moment of capture through distribution and consumption.

### 1.2 Design Principles

- **Certify the real, not detect the fake.** The framework establishes verifiable provenance for
  authentic content rather than attempting to identify inauthentic content.
- **No single point of failure.** Trust is distributed across hardware, decentralised oracle
  networks, and immutable ledgers. No single party can compromise the system unilaterally.
- **Privacy by design.** The optional privacy tier allows content creators to prove provenance
  without revealing sensitive metadata.
- **Platform independence.** Provenance records exist independently of any platform's metadata
  handling. Records survive stripping, recompression, and recirculation.
- **Open and neutral.** The framework is an open specification. No entity controls it.
  Implementations are free and unrestricted under Apache 2.0.
- **Vendor independence.** The Verifiable Data Registry layer is defined by an abstract interface.
  No specific vendor or technology is mandated. Space and Time is the reference implementation.
- **Composable.** Each layer exposes a well-defined interface and may be implemented independently.

### 1.3 Scope and Non-Goals

This specification covers protocol interfaces, data formats, conformance requirements, security
properties, and governance requirements.

> **Non-Goals — 0r1g1n explicitly does NOT attempt to:**
> - Verify the factual accuracy of captured content
> - Verify contextual truth or the intent of capture
> - Determine whether content depicts real events
> - Replace human editorial judgement or fact-checking
> - Provide absolute security against state-level adversaries with unlimited resources

### 1.4 Relationship to Other Standards

| Standard | Relationship |
|----------|-------------|
| C2PA Specification | 0r1g1n uses C2PA for hardware-level signing. C2PA manifests are the primary input to the registration protocol. 0r1g1n extends C2PA with platform-independent registry and decentralised trust infrastructure. |
| Chainlink Protocol | 0r1g1n uses Chainlink Functions for oracle-based verification. The DON provides consensus verification of C2PA manifest validity. |
| Space and Time | Reference implementation of the Verifiable Data Registry interface. Other implementations satisfying the VDR interface are permitted. |
| C2PA 2.1 Soft Binding | 0r1g1n's watermark recovery layer is compatible with and extends the C2PA 2.1 Soft Binding Resolution API. |
| ERC-7053 | 0r1g1n's on-chain verifier contract is compatible with the ERC-7053 on-chain media provenance standard. |

---

## 2. Definitions and Terminology

| Term | Definition |
|------|-----------|
| Attestation | A signed record produced by the 0r1g1n pipeline asserting that a specific piece of content has verified provenance. |
| C2PA Manifest | A cryptographically signed record produced by a C2PA-conformant device at capture, containing the content hash, device certificate, and action history. |
| Confidence Score | An integer from 0 to 100 expressing the oracle network's consensus assessment of provenance validity. |
| Content Hash | A SHA-256 digest of raw content bytes at capture. Primary key for all 0r1g1n registry operations. |
| Conforming Implementation | An implementation satisfying all MUST and REQUIRED requirements for the layers it implements. |
| DECO | Decentralised Oracle Commitment. A protocol enabling ZKPs about TLS-authenticated data without server-side modification. |
| Manifest URI | A dereferenceable HTTPS URI at which the C2PA manifest can be retrieved. |
| Oracle Node | A Chainlink network participant that independently validates content provenance claims. |
| Proof of SQL | Space and Time's ZK proof system guaranteeing correctness of SQL query results. Reference implementation of the VDR proof mechanism. |
| Provenance Record | The complete provenance data for a piece of content stored in the VDR. |
| Registration | The act of submitting a content hash and manifest URI to the 0r1g1n protocol for verification and on-chain recording. |
| Soft Binding | A pixel-embedded watermark referencing the provenance record, allowing recovery when file metadata has been stripped. |
| Trust Level | The overall authenticity assessment: HIGH, MEDIUM, LOW, PROVISIONAL, UNVERIFIED, or COMPROMISED. |
| VDR | Verifiable Data Registry. The abstract Layer 2 interface for provenance storage with cryptographic proof of query correctness. |
| Verifier | Any party querying the 0r1g1n registry to assess the provenance of a piece of content. |
| Watcher | An optional third-party agent that monitors distributed content for recirculation events. |
| ZKP Tier | The optional privacy-preserving Layer 6, activated for high-risk use cases such as conflict journalism. |

---

## 3. Threat Model (Normative)

### 3.1 Adversary Classes

| Class | Capability | Example | 0r1g1n Resistance |
|-------|-----------|---------|------------------|
| **A1** | Metadata manipulation, basic image editing, social distribution | Social media repost with false caption | Strong |
| **A2** | Credential replay, malicious editing, moderate technical capability | Reattaching credentials to different content | Strong |
| **A3** | Infrastructure compromise, minority oracle collusion, rogue platforms | Minority node collusion | Moderate |
| **A4** | Nation-state: CA compromise, majority oracle collusion, cryptographic attacks | State-level adversary with $500M+ resources | Limited |

### 3.2 Trust Assumptions

| ID | Assumption |
|----|-----------|
| TA-1 | SHA-256 collision resistance holds. |
| TA-2 | ECDSA security holds on the curves used by C2PA. |
| TA-3 | At least ⌊N/2⌋+1 oracle nodes in any DON deployment are honest and non-colluding. |
| TA-4 | Device private keys remain hardware-protected and cannot be extracted. |
| TA-5 | The C2PA Trust List is maintained by a process resistant to unilateral manipulation. |
| TA-6 | The VDR implementation correctly computes and returns ZK proofs of query correctness. |
| TA-7 | CRQCs capable of breaking TA-1 or TA-2 are not available within this specification's deployment horizon (estimated 5–10 years). |

### 3.3 Attack Surface Summary

| Attack Vector | Adversary Class | Primary Mitigation |
|--------------|----------------|-------------------|
| Metadata stripping by platform | A1 | Pixel watermark + blockchain event log |
| Credential replay on altered content | A2 | Content hash binding |
| Certificate Authority compromise | A3 | Chainlink consensus independent of any single CA |
| Oracle node collusion (minority) | A3 | Weighted median consensus + reputation system |
| Oracle node collusion (majority) | A4 | Economically irrational at DON scale |
| Device key extraction / firmware exploit | A3–A4 | Revocation protocol (Section 8.3) + Validity Epoch |
| Privacy extraction (ZKP tier) | A2–A3 | DECO — sensitive fields never revealed |
| Smart contract exploit | A2–A3 | Minimal scope + mandatory independent audit |
| Staged capture | A1–A4 | Not addressable — acknowledged non-goal |

---

## 4. System Architecture

### 4.1 Layer Overview

| Layer | Name | Function |
|-------|------|----------|
| 1 | C2PA Capture | Hardware signing at point of capture — root of trust |
| 2 | Verifiable Data Registry | Provenance storage with ZK proof of query correctness. Reference: Space and Time. |
| 3 | Chainlink Oracle | Decentralised consensus verification of C2PA manifests |
| 4 | On-Chain Verifier | Immutable verification event log + device registry status |
| 5 | Soft Binding | Watermark-based credential recovery after metadata stripping |
| 6 *(optional)* | Privacy Tier | ZKP selective disclosure for high-risk contexts |
| Extension | Watcher | Third-party recirculation monitoring agents |

### 4.2 Registration Protocol

```
1. CAPTURE
   Input:  Raw sensor data
   Action: C2PA device signs content at hardware/firmware level
   Output: signedFile, contentHash (SHA-256), manifestURI

2. WATERMARK
   Action: Embed soft binding watermark in pixel data
   Note:   Payload uses truncated hash as index key only.
           Full 256-bit hash used for all crypto operations.

3. VDR INGEST
   Action: VDR.registerProvenance(contentHash, manifestData)
   Status: PENDING

4. VERIFICATION REQUEST
   Action: registry.requestVerification(contentHash, manifestURI)

5. ORACLE EXECUTION
   Action: Min 5 DON nodes independently fetch, validate, score
           Weighted median consensus
           If score variance > 20: status = DISPUTED

6. VDR UPDATE
   Action: VDR.updateProvenance(contentHash, status=VERIFIED, score)

7. ON-CHAIN RECORD
   Action: Verify VDR proof on-chain
           Emit ContentVerified event
           Record in validity epoch registry
```

### 4.3 Verification Query Protocol

```
1. HASH EXTRACTION
   C2PA metadata present → extract from manifest
   Watermark present     → resolve full hash via Soft Binding API
   Neither               → compute SHA-256 (likely UNVERIFIED)

2. REVOCATION CHECK
   Query on-chain RegistryStatus for device certificate
   Apply Validity Epoch logic if COMPROMISED (Section 8.3)

3. VDR QUERY
   record, proof = VDR.getProvenanceWithProof(contentHash)
   Verify VDR proof

4. TRUST LEVEL ASSIGNMENT
   HIGH:        c2paPresent AND verified AND score >= 80
                AND device NOT COMPROMISED
   MEDIUM:      verified AND score >= 60
   LOW:         verified AND score < 60
   PROVISIONAL: verified AND VDR unavailable (cached result)
   UNVERIFIED:  no registry record found
   COMPROMISED: device certificate flagged COMPROMISED
```

---

## 5. Layer 1 — C2PA Capture and Signing

### 5.1 Requirements

1. The implementation **MUST** use a C2PA-conformant device as listed on the C2PA Conforming Products List.
2. Signing **MUST** occur at or below the device firmware layer, before content data is accessible to application software.
3. The C2PA manifest **MUST** include: content hash (SHA-256), device certificate reference, ISO 8601 timestamp, and a `c2pa.created` action assertion.
4. The content hash **MUST** be computed over raw content bytes before any processing or compression.
5. The device certificate **MUST** be issued by a Certificate Authority on the C2PA Trust List.
6. The manifest **MUST** be available at a dereferenceable HTTPS URI for the content's expected verification lifetime.
7. The Manifest URI **MUST** be included in the soft binding watermark payload.
8. The C2PA manifest **MUST** include a `cryptographic_algorithm` field to support future migration to post-quantum primitives.

### 5.2 Manifest URI Endpoint

```http
GET {manifestURI}
Accept: application/json

Response 200:
{
  "claim_generator": "string",
  "title": "{contentHash}",
  "assertions": [...],
  "claim_certificate": "base64(X.509 DER)",
  "signature": "base64(ECDSA signature)",
  "cryptographic_algorithm": "ES256 | ES384 | ES512"
}
```

### 5.3 Content Hash Computation

```
contentHash = SHA-256(rawContentBytes)
Encoding: lowercase hexadecimal, 64 characters
```

---

## 6. Layer 2 — Verifiable Data Registry

### 6.1 Abstract Interface

Layer 2 is defined by an abstract VDR interface. Implementations **MUST** satisfy this interface.
No specific vendor or technology is mandated. Space and Time with Proof of SQL is the reference
implementation.

> **Vendor Independence:** The VDR abstraction ensures the protocol survives changes to any single
> vendor's infrastructure. Alternative implementations — IPFS with Merkle proofs, ZK-rollup storage,
> or other verifiable database systems — are permitted and encouraged.

### 6.2 VDR Interface

```typescript
interface VerifiableDataRegistry {
  /**
   * Register a new provenance record.
   * @param contentHash  SHA-256 hex string (primary key)
   * @param data         ProvenanceRecord
   * @returns            vdrRecordId
   */
  registerProvenance(contentHash: string, data: ProvenanceRecord): Promise<string>;

  /**
   * Retrieve a provenance record with cryptographic proof.
   * The proof MUST cryptographically attest that data has not been
   * tampered with since registration.
   */
  getProvenanceWithProof(contentHash: string): Promise<{
    data: ProvenanceRecord;
    proof: VDRProof;      // Implementation-specific. For SXT: Proof of SQL.
  }>;

  /** Update status and score after oracle verification. */
  updateProvenance(contentHash: string, update: ProvenanceUpdate): Promise<void>;

  /** Append a recirculation event. */
  appendRecirculation(contentHash: string, event: RecirculationEvent): Promise<void>;
}
```

### 6.3 Provenance Record Schema

```sql
-- Reference schema (Space and Time SQL)
-- Other VDR implementations MUST store equivalent fields

CREATE TABLE origin_provenance (
  content_hash            VARCHAR(64)   NOT NULL PRIMARY KEY,
  manifest_uri            VARCHAR(2048) NOT NULL,
  registration_time       TIMESTAMP     NOT NULL,
  verification_time       TIMESTAMP,
  confidence_score        TINYINT,          -- 0-100
  status                  VARCHAR(16)   NOT NULL,
    -- PENDING | VERIFIED | FAILED | DISPUTED | COMPROMISED
  c2pa_device_class       VARCHAR(128),
  c2pa_device_cert_id     VARCHAR(256),     -- for revocation lookup
  c2pa_timestamp          TIMESTAMP,
  c2pa_action             VARCHAR(64),
  cryptographic_algorithm VARCHAR(32),      -- e.g. ES256 — for future migration
  validation_flags        INTEGER,          -- bitmask (Section 6.5)
  chain_id                INTEGER,
  tx_hash                 VARCHAR(66),
  zkp_verified            BOOLEAN DEFAULT FALSE,
  device_revoked_at       TIMESTAMP,        -- populated on revocation
  revocation_status       VARCHAR(16),      -- NONE | REVOKED | COMPROMISED
  revocation_reason       VARCHAR(256),
  registrar_id            VARCHAR(128),
  created_at              TIMESTAMP DEFAULT NOW()
);

CREATE TABLE origin_recirculation (
  id                      BIGINT AUTO_INCREMENT PRIMARY KEY,
  content_hash            VARCHAR(64) NOT NULL REFERENCES origin_provenance,
  detected_at             TIMESTAMP NOT NULL,
  platform                VARCHAR(128),
  context_url             VARCHAR(2048),
  context_change_score    TINYINT,
  reported_by             VARCHAR(128)      -- watcher agent ID (optional)
);
```

### 6.4 Provenance Query API

All external query responses **MUST** include a valid VDR proof.

```http
GET /api/v1/provenance?hash={contentHash}

Response 200:
{
  "schema_version": "0.2.0",
  "content_hash": "string",
  "status": "PENDING | VERIFIED | FAILED | DISPUTED | COMPROMISED",
  "confidence_score": 0-100,
  "c2pa_device_class": "string | null",
  "c2pa_timestamp": "ISO8601 | null",
  "c2pa_action": "string | null",
  "cryptographic_algorithm": "string | null",
  "validation_flags": integer,
  "zkp_verified": boolean,
  "revocation_status": "NONE | REVOKED | COMPROMISED",
  "device_revoked_at": "ISO8601 | null",
  "registration_time": "ISO8601",
  "verification_time": "ISO8601 | null",
  "on_chain": { "chain_id": integer, "tx_hash": "string | null" },
  "recirculation_count": integer,
  "vdr_proof": "base64(VDRProof)"
}

Response 404:
{ "error": "NOT_FOUND", "content_hash": "string" }
```

### 6.5 Validation Flags Bitmask

| Bit | Meaning |
|-----|---------|
| Bit 0 (0x01) | C2PA signature valid |
| Bit 1 (0x02) | Device certificate valid and on Trust List |
| Bit 2 (0x04) | Certificate not revoked at time of signing (OCSP) |
| Bit 3 (0x08) | Content hash matches manifest claim |
| Bit 4 (0x10) | Timestamp within acceptable range |
| Bit 5 (0x20) | Action type is `c2pa.created` |
| Bit 6 (0x40) | ZKP proof validated (Layer 6 only) |
| Bit 7 (0x80) | Reserved — MUST be 0 in this version |

---

## 7. Layer 3 — Chainlink Oracle Verification

### 7.1 Requirements

1. The implementation **MUST** use a Chainlink DON with a minimum of **5** independent nodes. Fewer than 5 nodes **MUST NOT** claim conformance.
2. Each oracle node **MUST** independently fetch and validate the C2PA manifest before submitting a result.
3. The consensus **MUST** be computed as a weighted median of node confidence scores.
4. Oracle nodes **MUST** implement the C2PA Validator Interface (Section 7.2).
5. The minimum consensus threshold is **60% weighted agreement**.
6. If fewer than 60% of nodes agree within a 20-point band, the result **MUST** be set to DISPUTED.
7. Oracle nodes **MUST** satisfy the admission requirements in Section 7.4.

### 7.2 C2PA Validator Interface

```json
// Input
{ "contentHash": "string (SHA-256 hex)", "manifestURI": "string (HTTPS URI)" }

// Output
{ "confidenceScore": "integer 0-100", "validationFlags": "integer", "error": "string | null" }

// Confidence score computation (MUST follow exactly):
// Base:   0
// +40     bit 0 set (signature valid)
// +25     bit 1 set (device trusted)
// +15     bit 3 set (hash match)
// +10     bit 4 set (timestamp valid)
// +10     bit 5 set (action is c2pa.created)
// Max:    100
```

### 7.3 Consensus and Reputation Algorithm

```javascript
// Weighted median consensus
function weightedMedian(nodeResults) {
  const sorted = nodeResults.sort((a, b) => a.score - b.score);
  const totalWeight = sorted.reduce((sum, n) => sum + n.weight, 0);
  let cumulative = 0;
  for (const node of sorted) {
    cumulative += node.weight;
    if (cumulative >= totalWeight / 2) return node.score;
  }
}

// Reputation update after each consensus round
// alpha: decay factor [recommended: 0.9]
// beta:  alignment reward [recommended: 0.1]
// consensus_alignment: 1 if node score within 10pts of consensus, else 0
rep_new = alpha * rep_old + beta * consensus_alignment

// Slashing: node stake slashed if |node_score - consensus| > 25
// for three consecutive requests
```

### 7.4 Node Admission Requirements

1. Node **MUST** stake a minimum amount defined by the DON governance (MUST NOT be zero).
2. Node **MUST** pass the 0r1g1n oracle conformance test suite before admission.
3. Node **MUST** run a conforming C2PA Validator service.
4. Node **MUST** maintain ≥99% uptime over a rolling 30-day window.
5. Node **MUST** maintain a reputation score ≥0.5. Nodes below 0.3 are suspended.
6. Nodes whose stake falls below minimum due to slashing **MUST** be automatically removed.

### 7.5 Disagreement Handling

```
If score variance across nodes > 20:
  status = DISPUTED
  human_review_required = true
  emit DisputedContent(contentHash, nodeScores)

If nodes_responding < minimum_quorum:
  status = FAILED_PENDING_RETRY
  retry: exponential backoff (1min → 5min → 30min → 24h)
  after 24h: status = FAILED
```

---

## 8. Layer 4 — On-Chain Verifier Contract

### 8.1 Requirements

1. The verifier **MUST** emit a `ContentVerified` event for each successful verification.
2. The contract **MUST** verify the VDR proof before emitting the event.
3. The contract **MUST** be deployed on an EVM-compatible chain.
4. The contract **MUST NOT** store full provenance records on-chain. It is an event log and device registry only.
5. The contract **MUST** be open source and audited by an independent security firm before mainnet deployment.
6. The contract **MUST** maintain a `RegistryStatus` mapping for device certificates to support the revocation protocol.

### 8.2 Contract Interface

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity ^0.8.19;

interface IOriginVerifier {
    event ContentVerified(
        bytes32 indexed contentHash,
        uint8   confidenceScore,
        bool    zkpVerified,
        uint256 timestamp
    );
    event DeviceRevoked(
        bytes32 indexed deviceCertId,
        uint256 revokedAt,
        string  reason
    );
    event ContentCompromised(
        bytes32 indexed contentHash,
        bytes32 indexed deviceCertId
    );

    function recordVerification(
        bytes32 contentHash,
        uint8   confidenceScore,
        bytes   calldata vdrProof
    ) external;

    function isVerified(bytes32 contentHash)
        external view
        returns (
            bool           verified,
            uint256        timestamp,
            uint8          confidenceScore,
            RegistryStatus deviceStatus
        );

    // Governance: multisig required
    function revokeDevice(bytes32 deviceCertId, string calldata reason) external;

    function getEffectiveTrustLevel(bytes32 contentHash)
        external view returns (TrustLevel);
}

enum RegistryStatus { ACTIVE, REVOKED, COMPROMISED }
enum TrustLevel     { HIGH, MEDIUM, LOW, PROVISIONAL, UNVERIFIED, COMPROMISED }
```

### 8.3 Device Revocation Protocol — Validity Epoch

```
REVOCATION LIFECYCLE

1. Compromise reported to DON governance
2. Governance multisig calls revokeDevice(deviceCertId, reason)
3. Contract records deviceCertId → COMPROMISED, revokedAt = now()
4. Contract emits DeviceRevoked event
5. Oracle nodes update local trust lists within 1 hour
6. Future validations from this device: automatic FAILED

VALIDITY EPOCH LOGIC for existing records:

record.c2pa_timestamp < device.revokedAt:
  → Signed before compromise — may be valid
  → effectiveTrustLevel = MEDIUM (downgraded)
  → revocation_status = REVOKED

record.c2pa_timestamp >= device.revokedAt:
  → Signed at or after compromise
  → effectiveTrustLevel = COMPROMISED
  → emit ContentCompromised(contentHash, deviceCertId)

device.revokedAt unknown (compromise window uncertain):
  → effectiveTrustLevel = LOW (conservative)
  → revocation_status = COMPROMISED
```

---

## 9. Layer 5 — Soft Binding and Watermark Recovery

### 9.1 Requirements

1. A conforming implementation **SHOULD** embed a soft binding watermark in all registered content.
2. The watermark **MUST** be imperceptible to human observers under normal viewing conditions.
3. The watermark **MUST** survive JPEG compression ≥70, uniform cropping ≤20%, and brightness/contrast adjustments ±20%.
4. The watermark payload **MUST** contain at minimum a truncated content hash (index key only) and a dereferenceable registry endpoint URI.
5. Implementations **MUST** document clearly that the truncated hash in the watermark is a 64-bit index key only. All cryptographic operations **MUST** use the full 256-bit SHA-256 hash.

### 9.2 Watermark Payload

```json
{
  "v": "1",
  "h": "{contentHash[0:16]}",  // 64-bit index key ONLY — NOT for crypto operations
  "e": "https://registry.example"
}
// Minimum 128-bit payload capacity required
// RECOMMENDED algorithm: dwtDctSvd (imwatermark library)
```

### 9.3 Soft Binding Resolution API

```http
GET /api/v1/resolve?hash={truncatedHash}&endpoint={registryEndpoint}

Response 200:
{
  "content_hash": "string (full 64-char SHA-256 hex)",
  "provenance": { /* ProvenanceRecord per Section 6.3 */ },
  "vdr_proof": "base64(VDRProof)",
  "recovered_via": "watermark"
}
```

---

## 10. Layer 6 — Privacy Tier (Optional)

### 10.1 Overview

The Privacy Tier is an **OPTIONAL** extension activated for use cases where content creator
identity or location must be protected — conflict zone journalism, whistleblowing, human rights
documentation.

> **Implementations that do not implement Layer 6 MUST NOT claim ZKP or privacy-preserving
> verification capability. Absence of Layer 6 does not affect conformance with Layers 1–5.**

### 10.2 Requirements

1. A conforming Layer 6 implementation **MUST** use the DECO protocol or equivalent TLS-based ZKP system.
2. Selective disclosure **MUST** follow Section 10.3.
3. Sensitive fields **MUST NOT** appear in the VDR record, on-chain event, or oracle node response.
4. The ZKP proof **MUST** be verifiable by any party with the public verification key.
5. Before any Level 3 conformance claim, the Groth16 circuit **MUST** undergo an MPC trusted setup ceremony. The final audited circuit hash **MUST** be published in the conformance test suite.

### 10.3 Selective Disclosure Policy

| Field | Disclosure | Condition Proven (if HIDDEN) |
|-------|-----------|------------------------------|
| Content hash | REVEAL | — |
| Device on Trust List | REVEAL | — |
| Timestamp range | REVEAL | — |
| Action type | REVEAL | — |
| Exact timestamp | HIDDEN | Within declared [min, max] range |
| GPS coordinates | HIDDEN | Within declared region (optional) |
| Device serial number | HIDDEN | Is valid C2PA-conformant device |
| Photographer identity | HIDDEN | Holds valid registrar credential |
| Manifest server identity | HIDDEN | Is on approved server list |

### 10.4 Standard and Deferred Registration

| Mode | Description |
|------|-------------|
| **Standard** (default) | Direct C2PA validation. No ZKP. `privacyTier: false`. |
| **Privacy** (immediate) | ZKP generated at registration time. `privacyTier: true`. |
| **Deferred Privacy** | Registered as Standard immediately, upgraded to Privacy later when device is charging or on Wi-Fi. **RECOMMENDED for mobile field capture.** `privacyTier: "deferred"` |

```http
POST /api/v1/upgrade-privacy
{
  "contentHash": "string",
  "zkpProof": "base64(ZKPProof)",
  "publicInputs": { /* Section 10.3 REVEAL fields */ }
}
```

---

## 11. Verification SDK Interface

### 11.1 Required Methods

```typescript
interface OriginVerifier {
  verify(input: string | ArrayBuffer): Promise<VerificationResult>;
  getHistory(contentHash: string): Promise<ProvenanceRecord[]>;
}

interface VerificationResult {
  trustLevel:          "HIGH" | "MEDIUM" | "LOW" | "PROVISIONAL" | "UNVERIFIED" | "COMPROMISED";
  contentHash:         string | null;
  confidenceScore:     number | null;    // 0-100
  c2paPresent:         boolean;
  blockchainVerified:  boolean;
  zkpVerified:         boolean;
  revocationStatus:    "NONE" | "REVOKED" | "COMPROMISED";
  timestamp:           string | null;    // ISO8601
  recoveredVia:        "c2pa" | "watermark" | "hash" | null;
  auditTrail:          AuditEvent[];
  rawRecord:           ProvenanceRecord | null;
}
```

### 11.2 Trust Level Computation

```javascript
// MUST be computed as follows:

if (revocationStatus === "COMPROMISED")
  return "COMPROMISED";

if (!blockchainVerified && cachedResult)
  return "PROVISIONAL";

if (c2paPresent && blockchainVerified
    && confidenceScore >= 80
    && revocationStatus === "NONE")
  return "HIGH";

if (blockchainVerified && confidenceScore >= 60)
  return "MEDIUM";

if (blockchainVerified)
  return "LOW";

return "UNVERIFIED";
```

---

## 12. Conformance

### 12.1 Conformance Levels

| Level | Layers Required | Suitable For |
|-------|----------------|-------------|
| **Level 1 — Core** | 2, 3, 4 | Platforms and verifiers receiving already-signed content |
| **Level 2 — Full** | 1, 2, 3, 4, 5 | Content creation devices and publishing services |
| **Level 3 — Privacy** | 1, 2, 3, 4, 5, 6 | High-risk documentation. MPC ceremony required before claim. |

### 12.2 Conformance Test Suite Structure

| Category | Description |
|----------|-------------|
| **Positive Vectors** | Known-good C2PA files. MUST return VERIFIED with score ≥80. |
| **Negative Vectors** | Broken signatures, expired certs, hash mismatches, revoked devices. MUST return correct error codes. |
| **Adversarial Vectors** | Valid C2PA manifests with altered pixel data. Watermarks compressed beyond spec claims. MUST detect tampering. |
| **Revocation Vectors** | Registrations from subsequently revoked devices. MUST apply Validity Epoch logic per Section 8.3. |
| **Failure Mode Vectors** | Simulated VDR unavailability, oracle quorum failure, blockchain congestion. MUST return correct degraded status. |
| **ZKP Vectors** *(Level 3)* | Valid and invalid ZKP proofs. Circuit hash MUST match published MPC ceremony output. |

---

## 13. Failure Modes and Recovery

Implementations **MUST** conform to the failure behaviours specified in this section.

| Failure Condition | Required Behaviour |
|------------------|-------------------|
| Oracle node(s) offline (below quorum) | Queue request. Retry with exponential backoff (1min, 5min, 30min, 24h). After 24h: status = FAILED. |
| VDR unavailable | Return PROVISIONAL with cached result and `cache_age_seconds`. If no cache: return UNVERIFIED with ORIG_014. |
| Blockchain congestion | Delay on-chain anchoring. Return MEDIUM with `pending_chain_confirmation: true`. |
| Manifest URI unreachable >24h | Status transitions to FAILED\_PENDING\_RETRY. Attempt fallback URI if specified. |
| VDR proof verification failure | Return ORIG_009. Do NOT return a trust level. Log for governance review. |
| Partial oracle results (above quorum, below full set) | Proceed. Include `partial_oracle_set: true`. Discount confidence score 5 points per missing node above minimum. |

---

## 14. Governance and Trust Onboarding

### 14.1 Roles

| Role | Description and Constraints |
|------|------------------------------|
| Trust List Maintainer | Maintains trusted C2PA CAs and device manufacturers. **MUST** be governed by a multi-party process. No single entity MAY have unilateral authority. |
| Oracle Operator | Operates Chainlink oracle nodes. Subject to admission requirements (Section 7.4). |
| Registrar | Authorised to submit content for registration. Identified by `registrar_id`. |
| Verifier | Any party querying the registry. No admission requirements — permissionless. |
| Watcher *(optional)* | Third-party recirculation monitoring agent. Permissionless. Reports weighted by reputation. |

### 14.2 Device Manufacturer Onboarding

Manufacturers wishing to join the Trust List **MUST**:

1. Submit a hardware security audit from an approved third-party firm demonstrating hardware-protected private keys.
2. Provide a hardware attestation certificate demonstrating C2PA conformance.
3. Submit the complete manufacturer CA certificate chain.
4. Obtain approval via multisig vote of at least 3 of 5 Trust List Maintainer keyholders.
5. Commit to publishing CRL updates within 24 hours of any device compromise report.

### 14.3 Governance Neutrality

> **No single entity — including the 0r1g1n project maintainers — MAY have unilateral authority
> to modify the Trust List, add or remove oracle nodes, or alter the protocol specification
> without a defined multi-party governance process.** This requirement is essential for
> institutional adoption.

---

## 15. Security Considerations

### 15.1 Scope of Security Guarantees

> **0r1g1n provides cryptographic evidence that:**
> - A specific piece of content was signed by a C2PA-conformant device at a specific time
> - The content has not been altered since signing
> - The provenance record has not been tampered with
> - Oracle consensus verified the C2PA manifest against the declared content hash
> - A device certificate was or was not revoked at the time of signing

> **0r1g1n does NOT provide:**
> - Verification that content accurately depicts reality
> - Verification of contextual truth or intent of capture
> - Absolute security against Class A4 adversaries
> - Detection of staged or fabricated events at point of capture

### 15.2 Threat Mitigations

| Threat | Mitigation |
|--------|-----------|
| Metadata stripping | Pixel watermark (Layer 5) + blockchain event log |
| CA compromise | Chainlink consensus independent of any single CA |
| Oracle manipulation | Weighted median + reputation weighting. Majority collusion economically irrational at mainnet scale. |
| Replay attack | Content hash binding — byte-level alteration detected |
| Device compromise | Validity Epoch logic (Section 8.3) |
| Smart contract exploit | Minimal scope. Mandatory audit. Proxy upgrade pattern recommended. |
| Privacy extraction (Layer 6) | DECO — sensitive fields never revealed |

### 15.3 Quantum Resistance and Migration Path

Current primitives (ECDSA, SHA-256) are vulnerable to CRQCs. CRQCs at this scale are not expected within this specification's deployment horizon. All data structures include a `cryptographic_algorithm` field. When ML-DSA or SLH-DSA (NIST FIPS 204/205) are supported by C2PA device firmware, conforming implementations **MUST** support algorithm negotiation via this field. A migration specification will be published prior to the estimated CRQC availability window.

### 15.4 Residual Risks

- **First-mile fabrication:** Content staged at capture is not detectable. Permanent non-goal.
- **Oracle majority collusion:** Economically irrational at DON scale but not eliminated.
- **VDR dependency:** Local caching (Section 13) mitigates unavailability but does not eliminate it.
- **Quantum:** Migration path defined in Section 15.3.

---

## 16. Acknowledgements

The 0r1g1n framework builds on the work of the Coalition for Content Provenance and Authenticity (C2PA), the Chainlink Foundation, Space and Time, the Cornell DECO research group, Numbers Protocol, Starling Lab, and the Content Authenticity Initiative.

Technical reviewers who contributed feedback incorporated in version 0.2.0-draft are acknowledged with gratitude.

---

## Appendix A — Data Schemas

### A.1 ProvenanceRecord (JSON)

```json
{
  "$schema": "https://0r1g1n.protocol/schemas/provenance/0.2.0",
  "content_hash": "string (64-char hex)",
  "manifest_uri": "string (HTTPS URI)",
  "status": "PENDING | VERIFIED | FAILED | DISPUTED | COMPROMISED",
  "confidence_score": "integer 0-100 | null",
  "c2pa": {
    "device_class": "string | null",
    "device_cert_id": "string | null",
    "timestamp": "ISO8601 | null",
    "action": "string | null",
    "cryptographic_algorithm": "string | null"
  },
  "validation_flags": "integer",
  "zkp_verified": "boolean",
  "revocation_status": "NONE | REVOKED | COMPROMISED",
  "device_revoked_at": "ISO8601 | null",
  "revocation_reason": "string | null",
  "registration_time": "ISO8601",
  "verification_time": "ISO8601 | null",
  "on_chain": {
    "chain_id": "integer | null",
    "tx_hash": "string | null"
  },
  "vdr_proof": "base64 string"
}
```

### A.2 VerificationResult (JSON)

```json
{
  "$schema": "https://0r1g1n.protocol/schemas/verification/0.2.0",
  "trust_level": "HIGH | MEDIUM | LOW | PROVISIONAL | UNVERIFIED | COMPROMISED",
  "content_hash": "string | null",
  "confidence_score": "integer 0-100 | null",
  "c2pa_present": "boolean",
  "blockchain_verified": "boolean",
  "zkp_verified": "boolean",
  "revocation_status": "NONE | REVOKED | COMPROMISED",
  "pending_chain_confirmation": "boolean",
  "timestamp": "ISO8601 | null",
  "recovered_via": "c2pa | watermark | hash | null",
  "cache_age_seconds": "integer | null",
  "audit_trail": [
    { "event": "string", "timestamp": "ISO8601", "detail": "string" }
  ],
  "raw_record": "ProvenanceRecord | null"
}
```

---

## Appendix B — Error Codes

| Code | Meaning |
|------|---------|
| ORIG_001 | Content hash not found in registry |
| ORIG_002 | Manifest URI unreachable or returned non-200 |
| ORIG_003 | C2PA manifest signature invalid |
| ORIG_004 | Device certificate not on Trust List |
| ORIG_005 | Device certificate revoked (OCSP) |
| ORIG_006 | Content hash mismatch between request and manifest |
| ORIG_007 | Timestamp outside acceptable range |
| ORIG_008 | Oracle consensus below minimum node threshold |
| ORIG_009 | VDR proof verification failed |
| ORIG_010 | Watermark not detected or payload unreadable |
| ORIG_011 | ZKP proof verification failed (Layer 6) |
| ORIG_012 | Smart contract call failed or reverted |
| ORIG_013 | Registry record in DISPUTED status |
| ORIG_014 | VDR unavailable — cached result returned (PROVISIONAL) |
| ORIG_015 | Device certificate COMPROMISED — Validity Epoch applied |
| ORIG_016 | Oracle quorum failure — request queued for retry |
| ORIG_017 | On-chain anchoring pending (blockchain congestion) |
| ORIG_018 | Privacy upgrade rejected — original record not found |
| ORIG_019 | ZKP circuit hash does not match published MPC ceremony output |
| ORIG_099 | Internal error — see detail field |

---

## Appendix C — Changelog

| Version | Changes |
|---------|---------|
| **0.2.0-draft** (March 2026) | Added: formal threat model (Section 3), VDR abstract interface (Section 6.1–6.2), oracle node admission requirements (Section 7.4), reputation and slashing algorithm (Section 7.3), device revocation protocol and Validity Epoch (Section 8.3), failure modes and recovery (Section 13), governance and trust onboarding (Section 14), deferred privacy registration (Section 10.4), watermark truncation clarification (Section 9.1), `cryptographic_algorithm` field (Sections 5, 6.3), expanded conformance test suite structure (Section 12.2), PROVISIONAL and COMPROMISED trust levels, error codes ORIG_014–ORIG_019. Changed: Layer 2 renamed Verifiable Data Registry (abstract interface). |
| **0.1.0-draft** (March 2026) | Initial draft published for community review. |

---

*0r1g1n Framework Specification v0.2.0-draft | March 2026 | Apache 2.0*
