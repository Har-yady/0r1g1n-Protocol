# 0r1g1n Framework Specification

**Version:** v0.2.0-draft | **Date:** March 2026 | **Status:** DRAFT  
**Licence:** Apache 2.0 | **Repository:** github.com/0r1g1n-protocol

---

## Status of This Document
This document is version 0.2.0-draft of the 0r1g1n Open Provenance Verification Protocol specification. It supersedes version 0.1.0-draft and incorporates feedback from two independent technical reviews. Key additions in this version: formal threat model (Section 3), Verifiable Data Layer abstraction (Section 5), oracle node admission and economic requirements (Section 6), device revocation protocol (Section 7), failure modes and recovery (Section 13), governance and trust onboarding (Section 14), and a conformance test suite structure (Section 12.2).

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in RFC 2119.

## Contents
1\. Introduction

2\. Definitions and Terminology

3\. Threat Model (Normative)

4\. System Architecture

5\. Layer 1 — C2PA Capture and Signing

6\. Layer 2 — Verifiable Data Registry

7\. Layer 3 — Chainlink Oracle Verification

8\. Layer 4 — On-Chain Verifier Contract

9\. Layer 5 — Soft Binding and Watermark Recovery

10\. Layer 6 — Privacy Tier (Optional)

11\. Verification SDK Interface

12\. Conformance

13\. Failure Modes and Recovery

14\. Governance and Trust Onboarding

15\. Security Considerations

16\. Acknowledgements

Appendix A — Data Schemas

Appendix B — Error Codes

Appendix C — Changelog

## 1. Introduction
### 1.1 Purpose
0r1g1n is an open framework specification for verifiable content provenance. It defines the interfaces, data formats, and protocol behaviours required to establish cryptographically verifiable chains of custody for digital media from the moment of capture through distribution and consumption.

The framework is designed to be implemented by content creators, media organisations, platforms, and application developers. A conforming 0r1g1n implementation enables any consumer of digital content to verify its origin, authenticity, and transformation history — regardless of what platforms have done to the content during distribution.

### 1.2 Design Principles
-   Certify the real, not detect the fake. The framework establishes verifiable provenance for authentic content rather than attempting to identify inauthentic content.

-   No single point of failure. Trust is distributed across hardware, decentralised oracle networks, and immutable ledgers. No single party can compromise the system unilaterally.

-   Privacy by design. The optional privacy tier allows content creators to prove provenance without revealing sensitive metadata.

-   Platform independence. Provenance records exist independently of any platform's metadata handling. Records survive stripping, recompression, and recirculation.

-   Open and neutral. The framework is an open specification. No entity controls it. Implementations are free and unrestricted under Apache 2.0.

-   Vendor independence. The Verifiable Data Registry layer (Layer 2) is defined by an abstract interface. No single vendor is mandatory. Space and Time is the reference implementation.

-   Composable. Each layer exposes a well-defined interface and may be implemented independently.

### 1.3 Scope and Non-Goals
This specification covers protocol interfaces, data formats, conformance requirements, security properties, and governance requirements for the 0r1g1n framework.

+——————————————————————————————————————————————————————————————————————————————————————————————-+
| **Non-Goals — 0r1g1n explicitly does NOT attempt to:**                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                           |
| Verify the factual accuracy of captured content. Verify contextual truth or the intent of capture. Determine whether content depicts real events. Replace human editorial judgement or fact-checking. Provide absolute security against state-level adversaries with unlimited resources. |
+——————————————————————————————————————————————————————————————————————————————————————————————-+

### 1.4 Relationship to Other Standards
  ————————— ——————————————————————————————————————————————————————————————————————————–
  **Standard**                **Relationship**

  **C2PA Specification**      0r1g1n uses C2PA for hardware-level content signing. C2PA manifests are the primary input to the registration protocol. 0r1g1n extends C2PA by providing platform-independent registry and decentralised trust infrastructure.

  **Chainlink Protocol**      0r1g1n uses Chainlink Functions for oracle-based verification. The Chainlink DON provides consensus verification of C2PA manifest validity.

  **Space and Time**          Space and Time is the reference implementation of the Verifiable Data Registry interface (Section 6). Other implementations satisfying the VDR interface are permitted.

  **C2PA 2.1 Soft Binding**   0r1g1n's watermark recovery layer is compatible with and extends the C2PA 2.1 Soft Binding Resolution API specification.

  **ERC-7053**                0r1g1n's on-chain verifier contract is compatible with the ERC-7053 on-chain media provenance standard.
  ————————— ——————————————————————————————————————————————————————————————————————————–

## 2. Definitions and Terminology
  ——————————- ———————————————————————————————————————————————————
  **Term**                        **Definition**

  **Attestation**                 A signed record produced by the 0r1g1n pipeline asserting that a specific piece of content has verified provenance.

  **C2PA Manifest**               A cryptographically signed record produced by a C2PA-conformant device at capture, containing the content hash, device certificate, and action history.

  **Confidence Score**            An integer from 0 to 100 expressing the oracle network's consensus assessment of provenance validity.

  **Content Hash**                A SHA-256 digest of raw content bytes at capture. Primary key for all 0r1g1n registry operations.

  **Conforming Implementation**   An implementation satisfying all MUST and REQUIRED requirements of this specification for the layers it implements.

  **DECO**                        Decentralised Oracle Commitment. A protocol enabling zero-knowledge proofs about TLS-authenticated data without server-side modification.

  **Manifest URI**                A dereferenceable HTTPS URI at which the C2PA manifest for a given piece of content can be retrieved.

  **Oracle Node**                 A Chainlink network participant that independently validates content provenance claims and contributes to consensus.

  **Proof of SQL**                Space and Time's ZK proof system guaranteeing correctness of SQL query results. The reference implementation of the VDR proof mechanism.

  **Provenance Record**           The complete provenance data for a piece of content stored in the Verifiable Data Registry, including all fields defined in Section 6.2.

  **Registration**                The act of submitting a content hash and manifest URI to the 0r1g1n protocol for verification and on-chain recording.

  **Soft Binding**                A pixel-embedded watermark referencing the provenance record, allowing recovery when file metadata has been stripped.

  **Trust Level**                 The overall authenticity assessment produced by the SDK. One of: HIGH, MEDIUM, LOW, PROVISIONAL, or UNVERIFIED.

  **VDR**                         Verifiable Data Registry. The abstract Layer 2 interface for provenance data storage with cryptographic proof of query correctness.

  **Verifier**                    Any party querying the 0r1g1n registry to assess the provenance of a piece of content.

  **ZKP Tier**                    The optional privacy-preserving Layer 6, activated for high-risk use cases such as conflict journalism.

  **Watcher**                     An optional third-party agent that monitors distributed content for recirculation events and reports context changes to the registry.
  ——————————- ———————————————————————————————————————————————————

## 3. Threat Model (Normative)
### 3.1 Adversary Classes
The 0r1g1n framework is designed against four adversary classes. Security claims in this specification are scoped to the adversary class indicated. Claims are not made beyond Class A3 unless explicitly stated.

  —————– ————————————————————————————————————– —————————————————————————————— —————————————————————————————–
  Class             Capability                                                                                                     Example                                                                                    0r1g1n Resistance

  A1                Metadata manipulation, file editing, social distribution without credentials                                   Social media repost with false caption, basic image editing                                Strong — provenance chain cryptographically bound to content bytes

  A2                Credential replay, malicious editing, moderate technical capability                                            Malicious editor reattaching credentials to different content, replay of valid manifests   Strong — content hash binding detects any byte-level alteration

  A3                Infrastructure compromise, rogue platforms, coordinated oracle subversion below quorum                         Rogue platform stripping all credentials, minority oracle node collusion                   Moderate — distributed consensus and watermark recovery mitigate but do not eliminate

  A4                Nation-state level: CA compromise, majority oracle collusion, cryptographic attacks, hardware key extraction   State-level adversary with \$500M+ resources targeting the protocol                        Limited — architecture raises cost significantly but provides no absolute guarantee
  —————– ————————————————————————————————————– —————————————————————————————— —————————————————————————————–

### 3.2 Trust Assumptions
The following assumptions are made by the 0r1g1n protocol. Violation of any assumption reduces the security properties of the corresponding layer.

  ——————— ———————————————————————————————————————————————————————————————————————————————-
  **ID**                **Assumption**

  **TA-1**              SHA-256 collision resistance holds. No practical collision attack exists against SHA-256.

  **TA-2**              ECDSA security holds on the curves used by C2PA (P-256 or secp256k1). No practical private key recovery attack exists.

  **TA-3**              At least ⌊N/2⌋+1 oracle nodes in any DON deployment are honest and non-colluding at any given time.

  **TA-4**              Device private keys remain hardware-protected within the secure enclave of C2PA-conformant devices and cannot be extracted.

  **TA-5**              The C2PA Trust List is maintained by a process resistant to unilateral manipulation by any single party.

  **TA-6**              The Verifiable Data Registry implementation correctly computes and returns ZK proofs of query correctness.

  **TA-7**              Cryptographically Relevant Quantum Computers (CRQCs) capable of breaking TA-1 or TA-2 are not available within the deployment horizon of this version of the specification (estimated 5–10 years). See Section 15.3 for migration planning.
  ——————— ———————————————————————————————————————————————————————————————————————————————-

### 3.3 Attack Surface Summary
  —————————————— ———————– ——————————————————————–
  Attack Vector                              Adversary Class         Primary Mitigation

  Metadata stripping by platform             A1                      Pixel watermark (Layer 5) + blockchain event log

  Credential replay on altered content       A2                      Content hash binding — byte-level alteration detected

  Certificate Authority compromise           A3                      Chainlink consensus independent of any single CA

  Oracle node collusion (minority)           A3                      Weighted median consensus + reputation system

  Oracle node collusion (majority)           A4                      Economically irrational at DON scale; not eliminated

  Device key extraction / firmware exploit   A3-A4                   Revocation protocol (Section 8.3) + validity epoch

  Privacy extraction (ZKP tier)              A2-A3                   DECO protocol — sensitive fields never revealed

  Smart contract exploit                     A2-A3                   Minimal contract scope + mandatory independent audit

  Staged capture (fabricated reality)        A1-A4                   Not addressable by any provenance system — acknowledged non-goal
  —————————————— ———————– ——————————————————————–

## 4. System Architecture
### 4.1 Layer Overview
The 0r1g1n framework comprises six layers plus two optional extension roles. Layers 1 through 5 form the core protocol. Layer 6 is an optional privacy extension. The Watcher role is an optional ecosystem extension.

  ———————– ————————– ————————————————————————————————————————-
  Layer                   Name                       Function

  1                       C2PA Capture               Hardware signing at point of capture — root of trust

  2                       Verifiable Data Registry   Abstract interface for provenance storage with ZK proof of query correctness. Reference implementation: Space and Time.

  3                       Chainlink Oracle           Decentralised consensus verification of C2PA manifests

  4                       On-Chain Verifier          Immutable verification event log + device registry status

  5                       Soft Binding               Watermark-based credential recovery after metadata stripping

  6 (optional)            Privacy Tier               ZKP selective disclosure for high-risk contexts

  Extension               Watcher                    Third-party recirculation monitoring agents
  ———————– ————————– ————————————————————————————————————————-

### 4.2 Data Flow — Registration
  ———————————————————————–
  REGISTRATION PROTOCOL

  1\. CAPTURE

  Input: Raw sensor data

  Action: C2PA device signs content at hardware/firmware level

  Output: signedFile, contentHash (SHA-256), manifestURI

  2\. WATERMARK

  Input: signedFile, contentHash

  Action: Embed soft binding watermark in pixel data

  Payload: { truncatedHash (index only), registryEndpoint }

  NOTE: Full 256-bit hash used for all crypto operations.

  Truncated 64-bit value is index key only — not security-critical.

  Output: watermarkedFile

  3\. VDR INGEST

  Input: contentHash, manifestURI

  Action: Call VDR.registerProvenance(contentHash, manifestData)

  Record status: PENDING

  Output: vdrRecordId

  4\. VERIFICATION REQUEST

  Input: contentHash, manifestURI

  Action: Call registry contract requestVerification()

  Output: chainlinkRequestId

  5\. ORACLE EXECUTION

  Input: chainlinkRequestId

  Action: Minimum 5 DON nodes independently:

  \- Fetch manifest from manifestURI

  \- Validate C2PA certificate chain and revocation status

  \- Verify content hash match

  \- Compute confidence score

  DON aggregates to weighted median consensus

  If score variance > 20: status = DISPUTED

  Output: confidenceScore (uint8), validationFlags (bitmask)

  6\. VDR UPDATE

  Input: confidenceScore, validationFlags

  Action: VDR.updateProvenance(contentHash, status=VERIFIED, score)

  Output: Updated provenance record with VDR proof

  7\. ON-CHAIN RECORD

  Input: contentHash, confidenceScore, vdrProof

  Action: Verify VDR proof on-chain

  Emit ContentVerified event

  Record in validity epoch registry

  Output: Immutable on-chain verification event
  ———————————————————————–

### 4.3 Data Flow — Verification Query
  ———————————————————————–
  VERIFICATION QUERY PROTOCOL

  1\. HASH EXTRACTION

  If C2PA metadata present: extract contentHash from manifest

  Else if watermark present: extract truncated hash, resolve full

  hash via Soft Binding Resolution API

  Else: compute SHA-256 of content bytes (likely UNVERIFIED)

  2\. REVOCATION CHECK

  Query on-chain RegistryStatus for device certificate

  If COMPROMISED: apply Validity Epoch logic (Section 8.3)

  3\. VDR QUERY

  record, proof = VDR.getProvenanceWithProof(contentHash)

  Verify VDR proof — confirms tamperproof query result

  4\. TRUST LEVEL ASSIGNMENT

  HIGH: c2paPresent AND verified AND score >= 80

  AND device NOT COMPROMISED

  MEDIUM: verified AND score >= 60

  LOW: verified AND score < 60

  PROVISIONAL: verified AND vdr_unavailable (cached result)

  UNVERIFIED: no registry record found

  5\. RESULT

  Return: TrustLevel, ProvenanceRecord, vdrProof, auditTrail
  ———————————————————————–

## 5. Layer 1 — C2PA Capture and Signing
### 5.1 Requirements
1.  The implementation MUST use a C2PA-conformant device as listed on the C2PA Conforming Products List.

2.  Signing MUST occur at or below the device firmware layer, before content data is accessible to application software.

3.  The C2PA manifest MUST include: content hash (SHA-256), device certificate reference, ISO 8601 timestamp, and a c2pa.created action assertion.

4.  The content hash MUST be computed over raw content bytes before any processing, compression, or metadata embedding.

5.  The device certificate MUST be issued by a Certificate Authority on the C2PA Trust List.

6.  The manifest MUST be available at a dereferenceable HTTPS URI (Manifest URI) for the content's expected verification lifetime.

7.  The Manifest URI MUST be included in the soft binding watermark payload.

8.  The C2PA manifest MUST include a cryptographic_algorithm field identifying the signing algorithm used, to support future migration to post-quantum primitives. See Section 15.3.

### 5.2 Manifest URI Endpoint
  ———————————————————————–
  GET {manifestURI}

  Accept: application/json

  Response 200:

  {

  \"claim_generator\": \"string\",

  \"title\": \"{contentHash}\",

  \"assertions\": [\...],

  \"claim_certificate\": \"base64(X.509 DER)\",

  \"signature\": \"base64(ECDSA signature)\",

  \"cryptographic_algorithm\": \"ES256 | ES384 | ES512\"

  }
  ———————————————————————–

### 5.3 Content Hash Computation
  ———————————————————————–
  contentHash = SHA-256(rawContentBytes)

  Encoding: lowercase hexadecimal, 64 characters

  Example: a3f8c2d1e4b7091f6d2a\...
  ———————————————————————–

## 6. Layer 2 — Verifiable Data Registry
### 6.1 Abstract Interface
Layer 2 is defined by an abstract Verifiable Data Registry (VDR) interface. Implementations MUST satisfy this interface. No specific vendor or technology is mandated. Space and Time (SXT) with Proof of SQL is the reference implementation.

+——————————————————————————————————————————————————————————————————————————————————————————————+
| **Vendor Independence**                                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                          |
| The VDR abstraction ensures the protocol survives changes to any single vendor's infrastructure. Alternative implementations satisfying this interface — including IPFS with Merkle proofs, ZK-rollup storage, or other verifiable database systems — are permitted and encouraged. |
+——————————————————————————————————————————————————————————————————————————————————————————————+

### 6.2 VDR Interface Specification
  ————————————————————————
  // Required VDR interface — all Layer 2 implementations MUST satisfy

  interface VerifiableDataRegistry {

  /\*\*

  \* Register a new provenance record.

  \* \@param contentHash SHA-256 hex string (primary key)

  \* \@param data ProvenanceRecord (Section 6.3)

  \* \@returns vdrRecordId

  \*/

  registerProvenance(contentHash: string, data: ProvenanceRecord)

  : Promise<string>;

  /\*\*

  \* Retrieve a provenance record with cryptographic proof.

  \* \@param contentHash SHA-256 hex string

  \* \@returns { data: ProvenanceRecord, proof: VDRProof }

  \* The proof MUST cryptographically attest that data has not been

  \* tampered with since registration.

  \*/

  getProvenanceWithProof(contentHash: string)

  : Promise<{ data: ProvenanceRecord, proof: VDRProof }>;

  /\*\*

  \* Update status and score after oracle verification.

  \*/

  updateProvenance(contentHash: string, update: ProvenanceUpdate)

  : Promise<void>;

  /\*\*

  \* Append a recirculation event.

  \*/

  appendRecirculation(contentHash: string, event: RecirculationEvent)

  : Promise<void>;

  }

  // VDRProof is implementation-specific.

  // For SXT: Proof of SQL proof bytes.

  // For IPFS/Merkle: Merkle inclusion proof.

  // MUST be verifiable by any party with the public verification key.

  type VDRProof = bytes;
  ————————————————————————

### 6.3 Provenance Record Schema
  ———————————————————————–
  \– Reference schema (Space and Time SQL)

  \– Other VDR implementations MUST store equivalent fields

  CREATE TABLE origin_provenance (

  content_hash VARCHAR(64) NOT NULL PRIMARY KEY,

  manifest_uri VARCHAR(2048) NOT NULL,

  registration_time TIMESTAMP NOT NULL,

  verification_time TIMESTAMP,

  confidence_score TINYINT,

  status VARCHAR(16) NOT NULL,

  \– PENDING | VERIFIED | FAILED | DISPUTED | COMPROMISED

  c2pa_device_class VARCHAR(128),

  c2pa_device_cert_id VARCHAR(256), \– for revocation lookup

  c2pa_timestamp TIMESTAMP,

  c2pa_action VARCHAR(64),

  cryptographic_algorithm VARCHAR(32), \– e.g. ES256, for migration

  validation_flags INTEGER,

  chain_id INTEGER,

  tx_hash VARCHAR(66),

  zkp_verified BOOLEAN DEFAULT FALSE,

  device_revoked_at TIMESTAMP, \– populated on revocation

  revocation_status VARCHAR(16), \– NONE | REVOKED | COMPROMISED

  revocation_reason VARCHAR(256),

  registrar_id VARCHAR(128),

  created_at TIMESTAMP DEFAULT NOW()

  );

  CREATE TABLE origin_recirculation (

  id BIGINT AUTO_INCREMENT PRIMARY KEY,

  content_hash VARCHAR(64) NOT NULL REFERENCES origin_provenance,

  detected_at TIMESTAMP NOT NULL,

  platform VARCHAR(128),

  context_url VARCHAR(2048),

  context_change_score TINYINT,

  reported_by VARCHAR(128) \– watcher agent ID (optional)

  );
  ———————————————————————–

### 6.4 Provenance Query API
All external query responses MUST include a valid VDR proof in the response envelope.

**GET /api/v1/provenance**

  ————————————————————————-
  Request:

  GET /api/v1/provenance?hash={contentHash}

  Response 200:

  {

  \"schema_version\": \"0.2.0\",

  \"content_hash\": \"string\",

  \"status\": \"PENDING|VERIFIED|FAILED|DISPUTED|COMPROMISED\",

  \"confidence_score\": 0-100,

  \"c2pa_device_class\": \"string|null\",

  \"c2pa_timestamp\": \"ISO8601|null\",

  \"c2pa_action\": \"string|null\",

  \"cryptographic_algorithm\": \"string|null\",

  \"validation_flags\": integer,

  \"zkp_verified\": boolean,

  \"revocation_status\": \"NONE|REVOKED|COMPROMISED\",

  \"device_revoked_at\": \"ISO8601|null\",

  \"registration_time\": \"ISO8601\",

  \"verification_time\": \"ISO8601|null\",

  \"on_chain\": { \"chain_id\": integer, \"tx_hash\": \"string|null\" },

  \"recirculation_count\": integer,

  \"vdr_proof\": \"base64(VDRProof)\"

  }
  ————————————————————————-

### 6.5 Validation Flags Bitmask
  ——————— —————————————————
  **Bit**               **Meaning**

  **Bit 0 (0x01)**      C2PA signature valid

  **Bit 1 (0x02)**      Device certificate valid and on Trust List

  **Bit 2 (0x04)**      Certificate not revoked at time of signing (OCSP)

  **Bit 3 (0x08)**      Content hash matches manifest claim

  **Bit 4 (0x10)**      Timestamp within acceptable range

  **Bit 5 (0x20)**      Action type is c2pa.created (not c2pa.edited)

  **Bit 6 (0x40)**      ZKP proof validated (Layer 6 only)

  **Bit 7 (0x80)**      Reserved — MUST be 0 in this version
  ——————— —————————————————

## 7. Layer 3 — Chainlink Oracle Verification
### 7.1 Requirements
9.  The implementation MUST use a Chainlink Decentralised Oracle Network with a minimum of 5 independent nodes for consensus. Implementations with fewer than 5 nodes MUST NOT claim conformance.

10. Each oracle node MUST independently fetch and validate the C2PA manifest before submitting a result.

11. The consensus MUST be computed as a weighted median of individual node confidence scores, weighted by each node's reputation score normalised to [0,1].

12. Oracle nodes MUST implement the C2PA Validator Interface (Section 7.2).

13. The oracle job MUST be delivered via Chainlink Functions or equivalent DON job type.

14. The minimum consensus threshold is 60% weighted agreement. If fewer than 60% of nodes agree within a 20-point confidence band, the result MUST be set to DISPUTED.

15. Oracle nodes MUST satisfy the admission requirements defined in Section 7.4.

### 7.2 C2PA Validator Interface
  ———————————————————————–
  // Input

  { \"contentHash\": \"string (SHA-256 hex)\",

  \"manifestURI\": \"string (HTTPS URI)\" }

  // Output

  { \"confidenceScore\": integer, // 0-100

  \"validationFlags\": integer, // bitmask per Section 6.5

  \"error\": \"string|null\" }

  // Confidence score computation (MUST follow exactly):

  // Base: 0

  // +40 validationFlags bit 0 (signature valid)

  // +25 validationFlags bit 1 (device trusted)

  // +15 validationFlags bit 3 (hash match)

  // +10 validationFlags bit 4 (timestamp valid)

  // +10 validationFlags bit 5 (action is c2pa.created)

  // Max: 100
  ———————————————————————–

### 7.3 Consensus and Reputation Algorithm
  ———————————————————————–
  // Weighted median consensus

  function weightedMedian(nodeResults) {

  sorted = nodeResults.sort((a,b) => a.score - b.score);

  cumulativeWeight = 0;

  totalWeight = sum(node.weight for node in sorted);

  for (node in sorted) {

  cumulativeWeight += node.weight;

  if (cumulativeWeight >= totalWeight / 2) return node.score;

  }

  }

  // Reputation update after each consensus round

  // alpha: decay factor [0.9 recommended]

  // beta: alignment reward factor [0.1 recommended]

  // consensus_alignment: 1 if node score within 10pts of consensus,

  // 0 otherwise

  rep_new = alpha \* rep_old + beta \* consensus_alignment

  // Slashing condition

  // Node stake is slashed by slash_rate if:

  // |node_score - final_consensus| > SLASH_THRESHOLD (default: 25)

  // for three consecutive requests

  // Minimum stake requirement: defined by DON operator (MUST be > 0)
  ———————————————————————–

### 7.4 Node Admission Requirements
Oracle nodes operating in a conforming 0r1g1n DON MUST satisfy the following admission requirements before being included in consensus.

16. Node MUST stake a minimum amount defined by the DON governance (MUST NOT be zero).

17. Node MUST pass the 0r1g1n oracle conformance test suite (Section 12.2) before admission.

18. Node MUST run a conforming C2PA Validator service (Section 7.2).

19. Node MUST maintain an uptime record of at least 99% over a rolling 30-day window. Nodes falling below this threshold are suspended pending review.

20. Node MUST have a reputation score of at least 0.5 on a [0,1] scale. Newly admitted nodes begin at 0.5. Nodes falling below 0.3 are suspended.

21. A node whose stake falls below the minimum due to slashing MUST be automatically removed from the active set.

### 7.5 Disagreement Handling
  ———————————————————————–
  If score variance across nodes > 20:

  status = DISPUTED

  human_review_required = true

  emit DisputedContent(contentHash, nodeScores)

  If nodes_responding < minimum_quorum:

  status = FAILED_PENDING_RETRY

  retry after: exponential backoff (1min, 5min, 30min, 24h)

  after 24h without quorum: status = FAILED
  ———————————————————————–

## 8. Layer 4 — On-Chain Verifier Contract
### 8.1 Requirements
22. The on-chain verifier MUST emit a ContentVerified event for each successful verification.

23. The contract MUST verify the VDR proof before emitting the event.

24. The contract MUST be deployed on an EVM-compatible chain.

25. The contract MUST NOT store full provenance records on-chain. It is an event log and device registry only.

26. The contract MUST be open source and audited by an independent Web3 security firm before mainnet deployment.

27. The contract MUST maintain a RegistryStatus mapping for device certificates to support the revocation protocol.

### 8.2 Contract Interface
  —————————————————————————–
  // SPDX-License-Identifier: Apache-2.0

  pragma solidity \^0.8.19;

  interface IOriginVerifier {

  event ContentVerified(

  bytes32 indexed contentHash,

  uint8 confidenceScore,

  bool zkpVerified,

  uint256 timestamp

  );

  event DeviceRevoked(

  bytes32 indexed deviceCertId,

  uint256 revokedAt,

  string reason

  );

  event ContentCompromised(

  bytes32 indexed contentHash,

  bytes32 indexed deviceCertId

  );

  // Record a verification (Chainlink oracle only)

  function recordVerification(

  bytes32 contentHash,

  uint8 confidenceScore,

  bytes calldata vdrProof

  ) external;

  // Query verification status

  function isVerified(bytes32 contentHash)

  external view

  returns (

  bool verified,

  uint256 timestamp,

  uint8 confidenceScore,

  RegistryStatus deviceStatus

  );

  // Governance: flag device as compromised (multisig required)

  function revokeDevice(

  bytes32 deviceCertId,

  string calldata reason

  ) external;

  // Returns trust-adjusted status per Validity Epoch logic

  function getEffectiveTrustLevel(bytes32 contentHash)

  external view returns (TrustLevel);

  }

  enum RegistryStatus { ACTIVE, REVOKED, COMPROMISED }

  enum TrustLevel { HIGH, MEDIUM, LOW, PROVISIONAL, UNVERIFIED, COMPROMISED }
  —————————————————————————–

### 8.3 Device Revocation Protocol — Validity Epoch
Device revocation introduces a temporal dimension to trust. Records signed before a device was compromised may remain valid; records signed after must be treated as potentially fraudulent. This section defines the normative revocation lifecycle.

  ———————————————————————–
  REVOCATION LIFECYCLE

  1\. Compromise reported to DON governance

  2\. Governance multisig calls revokeDevice(deviceCertId, reason)

  3\. Contract records: deviceCertId -> COMPROMISED, revokedAt = now()

  4\. Contract emits DeviceRevoked event

  5\. Oracle nodes update local trust lists within 1 hour

  6\. Future validations from this device: automatic FAILED

  VALIDITY EPOCH LOGIC for existing records:

  If record.c2pa_timestamp < device.revokedAt:

  // Signed before compromise — may be valid

  effectiveTrustLevel = MEDIUM // downgraded from original

  revocation_status = REVOKED // flagged but not invalidated

  If record.c2pa_timestamp >= device.revokedAt:

  // Signed after or at time of compromise

  effectiveTrustLevel = COMPROMISED

  status = COMPROMISED

  contract emits ContentCompromised(contentHash, deviceCertId)

  If device.revokedAt is unknown (compromise window uncertain):

  effectiveTrustLevel = LOW

  revocation_status = COMPROMISED

  // Conservative: treat as potentially compromised
  ———————————————————————–

## 9. Layer 5 — Soft Binding and Watermark Recovery
### 9.1 Requirements
28. A conforming implementation SHOULD embed a soft binding watermark in all content registered with the 0r1g1n framework.

29. The watermark MUST be imperceptible to human observers under normal viewing conditions.

30. The watermark MUST survive JPEG compression at quality 70 or above, uniform cropping of up to 20% of image dimensions, and brightness/contrast adjustments within ±20%.

31. The watermark payload MUST contain at minimum a truncated content hash (index key only) and a dereferenceable registry endpoint URI.

32. Implementations MUST make clear in documentation that the truncated hash in the watermark is a 64-bit index key only. All cryptographic operations and chain lookups MUST use the full 256-bit SHA-256 hash.

33. A conforming Soft Binding Resolution endpoint MUST implement the API defined in Section 9.3.

### 9.2 Watermark Payload
  ———————————————————————–
  // Watermark payload structure

  {

  \"v\": \"1\",

  \"h\": \"{contentHash[0:16]}\", // 64-bit index key ONLY

  // NOT used for crypto operations

  \"e\": \"https://registry.example\"

  }

  // IMPORTANT: The full 256-bit contentHash is used for:

  // - VDR primary key

  // - On-chain event indexing

  // - All cryptographic verification

  // The truncated value in field 'h' is solely for efficient

  // watermark payload compression. The full hash is resolved

  // via the Soft Binding Resolution API.
  ———————————————————————–

### 9.3 Soft Binding Resolution API
  ———————————————————————–
  GET /api/v1/resolve?hash={truncatedHash}&endpoint={registryEndpoint}

  Response 200:

  {

  \"content_hash\": \"string (full 64-char SHA-256 hex)\",

  \"provenance\": { /\* ProvenanceRecord per Section 6.3 \*/ },

  \"vdr_proof\": \"base64(VDRProof)\",

  \"recovered_via\": \"watermark\"

  }
  ———————————————————————–

## 10. Layer 6 — Privacy Tier (Optional)
### 10.1 Overview
The Privacy Tier is an OPTIONAL extension activated for use cases where content creator identity or location must be protected. Typical use cases: conflict zone journalism, whistleblowing, human rights documentation, and any context where metadata disclosure creates physical safety risk.

+———————————————————————————————————————————————————————————-+
| **OPTIONAL LAYER**                                                                                                                                                               |
|                                                                                                                                                                                  |
| Implementations that do not implement Layer 6 MUST NOT claim ZKP or privacy-preserving verification capability. Absence of Layer 6 does not affect conformance with Layers 1–5. |
+———————————————————————————————————————————————————————————-+

### 10.2 Requirements
34. A conforming Layer 6 implementation MUST use the DECO protocol or an equivalent TLS-based ZKP system.

35. The implementation MUST enable selective disclosure per Section 10.3.

36. Sensitive fields MUST NOT appear in the VDR record, on-chain event, or any oracle node response.

37. The ZKP proof MUST be verifiable by any party with access to the public verification key.

38. Before any Level 3 conformance claim, the Groth16 circuit MUST undergo a multi-party computation (MPC) trusted setup ceremony. The final audited circuit hash MUST be published in the conformance test suite.

### 10.3 Selective Disclosure Policy
  ————————– ———————– ————————————
  Field                      Disclosure              Condition Proven (if HIDDEN)

  Content hash               REVEAL                  —

  Device on Trust List       REVEAL                  —

  Timestamp range            REVEAL                  —

  Action type                REVEAL                  —

  Exact timestamp            HIDDEN                  Within declared [min, max] range

  GPS coordinates            HIDDEN                  Within declared region (optional)

  Device serial number       HIDDEN                  Is valid C2PA-conformant device

  Photographer identity      HIDDEN                  Holds valid registrar credential

  Manifest server identity   HIDDEN                  Is on approved server list
  ————————– ———————– ————————————

### 10.4 Standard and Deferred Registration
Generating ZKP proofs on mobile devices at the moment of capture may impose unacceptable latency and battery drain, particularly in field conditions. The following registration modes are therefore defined.

  ————————————— ————————————————————————————————————————————————————————————————————————–
  **Mode**                                **Description**

  **Standard Registration (immediate)**   Registers content using direct C2PA validation path. No ZKP generated. Returns MEDIUM or HIGH trust level. Suitable for immediate publication requirements. privacyTier: false (default).

  **Privacy Registration (immediate)**    Generates ZKP proof at registration time. Full selective disclosure. Returns MEDIUM, HIGH, or privacy-tier trust level. Suitable when device has sufficient compute resources. privacyTier: true.

  **Deferred Privacy Registration**       Content first registered as Standard. Privacy upgrade submitted later (e.g. when device is charging or on Wi-Fi). RECOMMENDED for mobile field capture. privacyTier: deferred. Upgrade via POST /api/v1/upgrade-privacy.
  ————————————— ————————————————————————————————————————————————————————————————————————–

  ———————————————————————–
  // Deferred privacy upgrade endpoint

  POST /api/v1/upgrade-privacy

  {

  \"contentHash\": \"string\",

  \"zkpProof\": \"base64(ZKPProof)\",

  \"publicInputs\": { /\* per Section 10.3 REVEAL fields \*/ }

  }

  // On success: VDR record updated, zkp_verified = true

  // Trust level upgrades to privacy-tier equivalent

  // Original Standard registration record preserved
  ———————————————————————–

## 11. Verification SDK Interface
### 11.1 Required Methods
  ———————————————————————————————-
  interface OriginVerifier {

  verify(input: string | ArrayBuffer): Promise<VerificationResult>;

  getHistory(contentHash: string): Promise<ProvenanceRecord[]>;

  }

  interface VerificationResult {

  trustLevel: 'HIGH'|'MEDIUM'|'LOW'|'PROVISIONAL'|'UNVERIFIED'|'COMPROMISED';

  contentHash: string | null;

  confidenceScore: number | null;

  c2paPresent: boolean;

  blockchainVerified: boolean;

  zkpVerified: boolean;

  revocationStatus: 'NONE'|'REVOKED'|'COMPROMISED';

  timestamp: string | null;

  recoveredVia: 'c2pa'|'watermark'|'hash'|null;

  auditTrail: AuditEvent[];

  rawRecord: ProvenanceRecord | null;

  }
  ———————————————————————————————-

### 11.2 Trust Level Computation
  ———————————————————————–
  // Trust level MUST be computed as follows:

  if (revocationStatus === 'COMPROMISED') return 'COMPROMISED';

  if (!blockchainVerified && cachedResult) return 'PROVISIONAL';

  if (c2paPresent && blockchainVerified

  && confidenceScore >= 80

  && revocationStatus === 'NONE') return 'HIGH';

  if (blockchainVerified && confidenceScore >= 60) return 'MEDIUM';

  if (blockchainVerified) return 'LOW';

  return 'UNVERIFIED';
  ———————————————————————–

## 12. Conformance
### 12.1 Conformance Levels
  ————————- —————————————————————————————————————————————————–
  **Level**                 **Requirements**

  **Level 1 — Core**      Implements Layers 2, 3, 4. Suitable for platforms and verifiers receiving already-signed content.

  **Level 2 — Full**      Implements Layers 1, 2, 3, 4, 5. Suitable for content creation devices and services requiring full provenance chain.

  **Level 3 — Privacy**   Implements all layers including Layer 6. MPC ceremony for ZKP circuit MUST be completed before Level 3 claim. Suitable for high-risk documentation.
  ————————- —————————————————————————————————————————————————–

### 12.2 Conformance Test Suite Structure
The conformance test suite is provided in the project repository. All tests are grouped into the following categories. Implementations claiming a given level MUST pass all applicable tests.

  ——————————– ————————————————————————————————————————————————————————————————————————————
  **Category**                     **Description**

  **Positive Vectors**             Known-good C2PA files from certified devices. Tests MUST return VERIFIED with confidence score >= 80 and appropriate validation flags set.

  **Negative Vectors**             Files with: broken signatures, expired certificates, hash mismatches, revoked device certificates, missing manifests, and stripped metadata. Tests MUST return correct error codes per Appendix B.

  **Adversarial Vectors**          Files with valid C2PA manifests but altered pixel data. Watermarks subjected to cropping and compression beyond spec claims. Replayed credentials on different content. Tests MUST detect tampering and return appropriate status.

  **Revocation Vectors**           Registrations from a device certificate subsequently revoked. Tests MUST correctly apply Validity Epoch logic per Section 8.3 based on signing timestamp relative to revocation timestamp.

  **Failure Mode Vectors**         Simulated VDR unavailability, oracle quorum failure, and blockchain congestion. Tests MUST return correct degraded status per Section 13.

  **ZKP Vectors (Level 3 only)**   Valid and invalid ZKP proofs. Circuit hash MUST match published MPC ceremony output. Tests MUST verify selective disclosure policy per Section 10.3.
  ——————————– ————————————————————————————————————————————————————————————————————————————

## 13. Failure Modes and Recovery
Real systems fail. This section defines normative protocol behaviour during partial failure conditions. Implementations MUST conform to the failure behaviours specified here.

### 13.1 Failure Behaviour Table
  ———————————————————- —————————————————————————————————————————————————————————————————————————————————–
  **Failure Condition**                                      **Required Behaviour**

  **Oracle node(s) offline**                                 If quorum maintained: proceed with available nodes. If below quorum: queue request; retry with exponential backoff (1min, 5min, 30min, 24h). After 24h: status = FAILED. Emit OracleQuorumFailed event.

  **VDR unavailable**                                        SDK MUST check local cache of recently verified hashes. If cached: return PROVISIONAL trust level with cache timestamp. If not cached: return UNVERIFIED with error ORIG_014. Do NOT return stale HIGH or MEDIUM without indicating degraded state.

  **Blockchain congestion**                                  Delay on-chain anchoring. Verification may proceed using VDR proof alone. Return MEDIUM trust level until on-chain confirmation. Include pending_chain_confirmation: true in response.

  **Manifest URI unreachable**                               If manifest URI returns non-200 for more than 24 consecutive hours: status MUST transition to FAILED_PENDING_RETRY. Oracle nodes MUST attempt alternative manifest retrieval if fallback URI is specified in manifest. If no fallback: FAILED.

  **VDR proof verification failure**                         Return error ORIG_009. Do NOT return a trust level. Log failure for governance review.

  **Partial oracle results (between quorum and full set)**   Proceed with available results above quorum. Note in response: partial_oracle_set: true. Confidence score SHOULD be discounted by 5 points per missing node above minimum.
  ———————————————————- —————————————————————————————————————————————————————————————————————————————————–

### 13.2 Degraded Verification Trust Levels
  ———————————————————————–
  // Degraded states — MUST be communicated to SDK callers

  PROVISIONAL: VDR unavailable, cached result returned

  -> include cache_age_seconds in response

  PENDING: On-chain anchoring delayed due to congestion

  -> include pending_chain_confirmation: true

  DISPUTED: Oracle node disagreement > 20 points variance

  -> human review required

  -> include disputed_at timestamp
  ———————————————————————–

## 14. Governance and Trust Onboarding
### 14.1 Roles
  ————————— ——————————————————————————————————————————————————————————————————————————————————
  **Role**                    **Description and Constraints**

  **Trust List Maintainer**   Responsible for maintaining the list of trusted C2PA Certificate Authorities, device manufacturers, and registrar organisations. MUST be governed by a multi-party process. No single entity MAY have unilateral authority to modify the Trust List.

  **Oracle Operator**         Operates one or more Chainlink oracle nodes participating in 0r1g1n consensus. Subject to admission requirements (Section 7.4) and ongoing performance monitoring.

  **Registrar**               An organisation or individual authorised to submit content for registration. May act on behalf of content creators. Identified by registrar_id in the provenance record.

  **Verifier**                Any party querying the 0r1g1n registry. No admission requirements. Verification is public and permissionless.

  **Watcher (optional)**      Third-party agent monitoring distributed content for recirculation events. Reports context changes to the registry. Watchers are permissionless but their reports are weighted by reputation.
  ————————— ——————————————————————————————————————————————————————————————————————————————————

### 14.2 Device Manufacturer Onboarding
Hardware manufacturers wishing to have their devices listed on the 0r1g1n Trust List MUST satisfy the following onboarding requirements.

39. Submit a hardware security audit from an approved third-party security firm demonstrating that private keys are hardware-protected and cannot be extracted.

40. Provide a hardware attestation certificate demonstrating C2PA conformance.

41. Submit the complete certificate chain for the manufacturer's CA.

42. Approval requires a multisig vote of at least 3 of 5 Trust List Maintainer keyholders.

43. Manufacturers MUST commit to publishing Certificate Revocation List (CRL) updates within 24 hours of any device compromise report.

### 14.3 Governance Neutrality
+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————+
| **Neutrality Requirement**                                                                                                                                                                                                                                                                                                                                                               |
|                                                                                                                                                                                                                                                                                                                                                                                          |
| No single entity — including the 0r1g1n project maintainers — MAY have unilateral authority to modify the Trust List, add or remove oracle nodes, or alter the protocol specification without a defined multi-party governance process. This requirement is essential for institutional adoption. Large organisations will not build on infrastructure controlled by a single party. |
+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————+

### 14.4 Dispute Resolution
When a content record enters DISPUTED status, the following process applies.

44. Disputed records are flagged in the VDR and on-chain with a disputed_at timestamp.

45. The registrar and any challenging parties are notified via the dispute API.

46. Human reviewers with access to full oracle node submissions assess the disagreement.

47. Resolution may result in: VERIFIED (consensus reached), FAILED (content rejected), or DISPUTED_PERMANENT (unresolvable disagreement, flagged indefinitely).

48. Dispute resolution decisions are recorded on-chain as governance transactions.

## 15. Security Considerations
### 15.1 Scope of Security Guarantees
+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————-+
| **What 0r1g1n provides cryptographic evidence of:**                                                                                                                                                                                                                                                                                                               |
|                                                                                                                                                                                                                                                                                                                                                                   |
| \(a\) A specific piece of content was signed by a C2PA-conformant device at a specific time. (b) The content has not been altered since signing. (c) The provenance record has not been tampered with. (d) Oracle consensus verified the C2PA manifest against the declared content hash. (e) A device certificate was or was not revoked at the time of signing. |
+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————-+

+———————————————————————————————————————————————————————————————————————————————–+
| **What 0r1g1n does NOT provide:**                                                                                                                                                                                                             |
|                                                                                                                                                                                                                                               |
| Verification that content accurately depicts reality. Verification of contextual truth or intent of capture. Absolute security against Class A4 (nation-state) adversaries. Detection of staged or fabricated events at the point of capture. |
+———————————————————————————————————————————————————————————————————————————————–+

### 15.2 Threat Mitigations
  ———————————- ————————————————————————————————————————————————————-
  **Threat**                         **Mitigation**

  **Metadata stripping**             Pixel watermark (Layer 5) provides recovery. Blockchain event log is platform-independent.

  **CA compromise**                  Chainlink consensus is independent of any single CA. A compromised CA elevates risk but does not unilaterally break the system.

  **Oracle manipulation**            Weighted median consensus with reputation weighting mitigates minority collusion. Majority collusion is economically irrational at Chainlink mainnet scale.

  **Replay attack**                  Content hash binding: reattaching credentials to different content produces a hash mismatch detected by oracle validation.

  **Device compromise**              Validity Epoch logic (Section 8.3) ensures records signed before compromise retain qualified validity while records signed after are flagged COMPROMISED.

  **Smart contract exploit**         Minimal contract scope. Mandatory independent audit. Proxy upgrade pattern recommended.

  **Privacy extraction (Layer 6)**   DECO prevents oracle nodes from learning sensitive fields. ZKP reveals only claims in Section 10.3.
  ———————————- ————————————————————————————————————————————————————-

### 15.3 Quantum Resistance and Migration Path
All current cryptographic primitives (ECDSA on P-256, SHA-256) are vulnerable to Cryptographically Relevant Quantum Computers (CRQCs) using Shor's algorithm. CRQCs at this scale are not expected within the deployment horizon of this specification version (estimated 5–10 years from 2026).

To support future migration without breaking changes, all data structures in this specification include a cryptographic_algorithm field. When post-quantum standards (ML-DSA, SLH-DSA per NIST FIPS 204/205) are supported by C2PA device firmware, conforming implementations MUST support algorithm negotiation via this field. A migration specification will be published as a separate document prior to the estimated CRQC availability window.

### 15.4 Residual Risks
-   First-mile fabrication: Content staged at capture is not detectable by any provenance system. This is an acknowledged and permanent non-goal.

-   Oracle collusion at majority: Economically irrational at DON scale but theoretically possible. Not eliminated by design.

-   VDR dependency: The registry depends on VDR network availability. Local caching (Section 13) mitigates but does not eliminate.

-   Quantum: Addressed by migration path in Section 15.3.

## 16. Acknowledgements
The 0r1g1n framework builds on the work of the Coalition for Content Provenance and Authenticity (C2PA), the Chainlink Foundation, Space and Time, the Cornell DECO research group, and the broader open source content authenticity community including Numbers Protocol, Starling Lab, and the Content Authenticity Initiative.

Technical reviewers who contributed feedback incorporated in version 0.2.0-draft are acknowledged with gratitude. All feedback was provided independently and without compensation.

## Appendix A — Data Schemas
### A.1 ProvenanceRecord (JSON)
  ———————————————————————————–
  { \"\$schema\": \"https://0r1g1n.protocol/schemas/provenance/0.2.0\",

  \"content_hash\": \"string (64-char hex)\",

  \"manifest_uri\": \"string (HTTPS URI)\",

  \"status\": \"PENDING|VERIFIED|FAILED|DISPUTED|COMPROMISED\",

  \"confidence_score\": \"integer 0-100|null\",

  \"c2pa\": {

  \"device_class\": \"string|null\",

  \"device_cert_id\": \"string|null\",

  \"timestamp\": \"ISO8601|null\",

  \"action\": \"string|null\",

  \"cryptographic_algorithm\": \"string|null\"

  },

  \"validation_flags\": \"integer\",

  \"zkp_verified\": \"boolean\",

  \"revocation_status\": \"NONE|REVOKED|COMPROMISED\",

  \"device_revoked_at\": \"ISO8601|null\",

  \"revocation_reason\": \"string|null\",

  \"registration_time\": \"ISO8601\",

  \"verification_time\": \"ISO8601|null\",

  \"on_chain\": { \"chain_id\": \"integer|null\", \"tx_hash\": \"string|null\" },

  \"vdr_proof\": \"base64 string\"

  }
  ———————————————————————————–

### A.2 VerificationResult (JSON)
  —————————————————————————————————–
  { \"\$schema\": \"https://0r1g1n.protocol/schemas/verification/0.2.0\",

  \"trust_level\": \"HIGH|MEDIUM|LOW|PROVISIONAL|UNVERIFIED|COMPROMISED\",

  \"content_hash\": \"string|null\",

  \"confidence_score\": \"integer 0-100|null\",

  \"c2pa_present\": \"boolean\",

  \"blockchain_verified\": \"boolean\",

  \"zkp_verified\": \"boolean\",

  \"revocation_status\": \"NONE|REVOKED|COMPROMISED\",

  \"pending_chain_confirmation\": \"boolean\",

  \"timestamp\": \"ISO8601|null\",

  \"recovered_via\": \"c2pa|watermark|hash|null\",

  \"cache_age_seconds\": \"integer|null\",

  \"audit_trail\": [{ \"event\": \"string\", \"timestamp\": \"ISO8601\", \"detail\": \"string\" }],

  \"raw_record\": \"ProvenanceRecord|null\"

  }
  —————————————————————————————————–

## Appendix B — Error Codes
  ——————— ——————————————————————
  **Code**              **Meaning**

  **ORIG_001**          Content hash not found in registry

  **ORIG_002**          Manifest URI unreachable or returned non-200

  **ORIG_003**          C2PA manifest signature invalid

  **ORIG_004**          Device certificate not on Trust List

  **ORIG_005**          Device certificate revoked (OCSP)

  **ORIG_006**          Content hash mismatch between request and manifest

  **ORIG_007**          Timestamp outside acceptable range

  **ORIG_008**          Oracle consensus below minimum node threshold

  **ORIG_009**          VDR proof verification failed

  **ORIG_010**          Watermark not detected or payload unreadable

  **ORIG_011**          ZKP proof verification failed (Layer 6)

  **ORIG_012**          Smart contract call failed or reverted

  **ORIG_013**          Registry record in DISPUTED status

  **ORIG_014**          VDR unavailable — cached result returned (PROVISIONAL)

  **ORIG_015**          Device certificate marked COMPROMISED — Validity Epoch applied

  **ORIG_016**          Oracle quorum failure — request queued for retry

  **ORIG_017**          On-chain anchoring pending (blockchain congestion)

  **ORIG_018**          Privacy upgrade rejected — original record not found

  **ORIG_019**          ZKP circuit hash does not match published MPC ceremony output

  **ORIG_099**          Internal error — see detail field
  ——————— ——————————————————————

## Appendix C — Changelog
  —————————— ————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
  **Version**                    **Changes**

  **0.2.0-draft (March 2026)**   Added: formal threat model (Section 3), VDR abstract interface (Section 6.1-6.2), oracle node admission requirements (Section 7.4), reputation and slashing algorithm (Section 7.3), device revocation protocol and Validity Epoch (Section 8.3), failure modes and recovery (Section 13), governance and trust onboarding (Section 14), deferred privacy registration (Section 10.4), watermark truncation clarification (Section 9.1), cryptographic_algorithm field (Sections 5, 6.3), expanded conformance test suite structure (Section 12.2), PROVISIONAL and COMPROMISED trust levels, new error codes ORIG_014 through ORIG_019. Changed: Section 5 renamed Verifiable Data Registry (formerly Space and Time Registry). Fixed: layer numbering alignment with whitepaper.

  **0.1.0-draft (March 2026)**   Initial draft published for community review.
  —————————— ————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————

0r1g1n Framework Specification 0.2.0-draft | March 2026 | Apache 2.0
