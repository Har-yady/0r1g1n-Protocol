# 0r1g1n — Technical Whitepaper

**Version:** v1.1 | **Date:** March 2026 | **Status:** DRAFT  
**Licence:** Apache 2.0 | **Repository:** github.com/0r1g1n-protocol

---

## Abstract

We present 0r1g1n, a hybrid content verification architecture that combines four proven technologies to provide cryptographically verifiable provenance for digital media from the moment of capture through distribution and consumption. The architecture addresses the systemic failure of existing single-component approaches by assigning each technology the function it was specifically designed for: C2PA hardware signing provides a root of trust at the point of physical capture; a Verifiable Data Registry (VDR) abstract interface provides platform-independent provenance storage with ZK proof of query correctness (Space and Time is the reference implementation); the Chainlink Decentralised Oracle Network supersedes the need for single-authority certificate infrastructure with distributed consensus; an immutable blockchain ledger provides platform-independent records that survive metadata stripping; and DECO-based Zero-Knowledge Proofs enable privacy-preserving verification that protects content creators without sacrificing authenticity guarantees.

The architecture is designed for production deployment using tools and APIs available in 2026. No component requires novel research — all four layers are deployed at institutional scale in adjacent domains. The contribution of this paper is the integration specification that combines them into a coherent, system that addresses all identified architectural weaknesses. We describe the full technical architecture, the protocol specifications for inter-layer communication, implementation guidance for each component, and the residual limitations that no cryptographic system can resolve.

## 1. Introduction
### 1.1 The Authenticity Problem
Digital media authenticity has broken down as a reliable social institution. The convergence of generative AI, optimised distribution infrastructure, and geopolitical information warfare has produced an environment in which the cost of creating convincing synthetic media has dropped to near zero while the cost of verifying authentic media remains high.

Existing detection approaches — AI classifiers, human review, watermark scanning — share a structural weakness: they are reactive. They attempt to identify the fake after it has been created and circulated. Against an adversary generating content at machine speed, reactive detection cannot win at scale. The correct frame is not detecting the fake but certifying the real — establishing infrastructure in which authentic content is verifiably, cryptographically, and immutably itself regardless of what happens to it in distribution.

### 1.2 The Gap in Existing Solutions
C2PA (Coalition for Content Provenance and Authenticity) is the most mature existing standard and represents the correct instinct — hardware-level signing at the point of capture, embedded credentials, a standardised manifest format. Its weaknesses are well-documented: credentials are stripped by every major social platform on upload; the Certificate Authority trust model creates single points of failure; the metadata carries sensitive information that endangers journalists and whistleblowers.

Blockchain-based approaches address the stripping problem by anchoring records independently of the file. But without a hardware signing layer they have no root of trust — content can be registered on-chain after manipulation as easily as before. Oracle networks provide decentralised verification but require the other components to have anything to verify. Zero-Knowledge Proofs provide privacy guarantees but require an existing verification infrastructure to apply them to.

0r1g1n's contribution is not any individual component — all four exist and are deployed at scale — but their integration into a system in which each component's weaknesses are closed by the others.

### 1.3 Scope of This Paper
This paper specifies the 0r1g1n architecture in sufficient technical detail to enable independent implementation. Section 2 defines the threat model. Section 3 describes each component layer. Section 4 specifies the inter-layer protocol. Section 5 provides implementation guidance. Section 6 addresses security analysis. Section 7 acknowledges residual limitations. Section 8 discusses related work.

## 2. Threat Model
### 2.1 Adversary Classes
0r1g1n is designed against three adversary classes of increasing capability.

  ——————————- ————————————————————————————————————————————————————————————————————————-
  **Adversary Class**             **Capabilities**

  **Class I — Opportunistic**   Actors using consumer-accessible tools to create and distribute synthetic media without technical sophistication. Share fabricated content without attempting to forge provenance. Largest by volume.

  **Class II — Motivated**      Actors with technical capability who attempt to forge or strip credentials. May attempt metadata manipulation, watermark removal, or replay attacks using previously valid credentials on new content.

  **Class III — State-level**   Actors with resources to compromise Certificate Authorities, operate fake verification nodes, or conduct cryptographic attacks. Able to stage events and produce genuinely captured but deliberately deceptive content.
  ——————————- ————————————————————————————————————————————————————————————————————————-

0r1g1n provides strong security properties against Class I and II adversaries. Against Class III, the architecture raises the cost of attack significantly but cannot provide absolute guarantees — no provenance system can, since a sufficiently resourced adversary can stage reality at the point of capture.

### 2.2 Attack Surface
  —————————– ————————————————————————————————————————————————————————————
  **Attack Vector**             **Mitigation**

  **Metadata stripping**        Platforms process uploaded content in ways that destroy embedded C2PA manifests. Addressed by blockchain ledger independence and pixel watermark soft binding.

  **Certificate compromise**    A compromised CA retroactively invalidates all credentials it issued. Addressed by Chainlink distributed consensus — no single CA in the trust path.

  **Replay attack**             Valid credentials from authentic content reattached to manipulated content. Addressed by content hash binding — credential is cryptographically bound to specific content bytes.

  **Oracle manipulation**       A malicious actor operates a fake Chainlink node providing false attestations. Addressed by Chainlink consensus requiring majority of nodes to agree.

  **Privacy extraction**        Verification reveals sensitive metadata — location, device identity, photographer. Addressed by DECO/ZKP — sensitive fields never revealed to oracle or on-chain.

  **First-mile manipulation**   Content manipulated before signing at the hardware layer. Beyond cryptographic mitigation — addressed by hardware integration at the sensor chipset level.

  **Staged capture**            Authentic capture of staged or fabricated reality. Not addressable by any provenance system — acknowledged as a residual limitation.
  —————————– ————————————————————————————————————————————————————————————

## 3. Architecture
### 3.1 Layer Overview
0r1g1n comprises four layers operating in sequence. Each layer has a precisely defined input, function, and output. Layers communicate through standardised interfaces specified in Section 4.

  ———————————– ———————————— —————————————– ———————————

  Layer 1 — C2PA                    Layer 3 — Chainlink                Layer 4 — On-Chain Verifier             Layer 6 — Privacy Tier/ZKP

  Hardware signing at capture         Decentralised oracle verification    Immutable on-chain record                 Privacy-preserving proof

  Input: raw content                  Input: content hash + manifest URI   Input: consensus attestation              Input: TLS manifest data

  Output: signed manifest + hash      Output: verified attestation         Output: permanent record                  Output: ZKP proof

  Trust basis: hardware certificate   Trust basis: distributed consensus   Trust basis: cryptographic immutability   Trust basis: mathematical proof
  ———————————– ———————————— —————————————– ———————————

### 3.2 Layer 1 — C2PA Hardware Signing
The C2PA layer provides the system's root of trust. Signing occurs at the sensor chipset level, before image data is accessible to any software layer. This is the only point in the pipeline where physical reality and digital record are simultaneously present.

#### 3.2.1 Signing Process
When a C2PA-enabled device captures content, the following operations occur in sequence within the secure hardware enclave:

1.  Raw sensor data is captured and buffered in the secure enclave

2.  SHA-256 hash computed over the raw content bytes

3.  C2PA manifest assembled containing: content hash, device certificate reference, ISO 8601 timestamp, action record (c2pa.created), and optional fields (geolocation if permitted)

4.  Manifest signed using the device's X.509 private key — stored in hardware security module, never extractable

5.  Signed manifest embedded in file using JUMBF (JPEG Universal Metadata Box Format) container

6.  Content hash made available via standardised API endpoint for Layer 2 VDR ingest

#### 3.2.2 Certificate Chain
The C2PA certificate chain follows X.509 PKI conventions with extensions specific to the C2PA use case. The device certificate includes the C2PA Extended Key Usage extension (OID 1.3.6.1.4.1.57737.2.1.1) and is issued by a Certificate Authority on the C2PA Trust List. Validation requires the full chain: device certificate, intermediate CA, root CA on Trust List.

  ——————————————————————————————-
  Certificate Chain:

  C2PA Root CA (Trust List anchor)

  └── Camera Manufacturer Intermediate CA

  └── Device Certificate (per-device, hardware-bound)

  └── Content Manifest Signature

  C2PA Manifest Structure (JSON-LD):

  {

  \"claim_generator\": \"ManufacturerName/DeviceModel/1.0\",

  \"title\": \"<SHA-256 content hash>\",

  \"assertions\": [

  { \"label\": \"c2pa.actions\",

  \"data\": { \"actions\": [{\"action\": \"c2pa.created\",

  \"digitalSourceType\": \"http://cv.iptc.org/newscodes/digitalsourcetype/digitalCapture\",

  \"softwareAgent\": \"DeviceFirmware/x.x\"}] } },

  { \"label\": \"stds.exif\",

  \"data\": { \"Exif:DateTimeOriginal\": \"<ISO8601>\",

  \"Exif:GPSLatitude\": \"<REDACTED_IN_ZKP>\" } }

  ]

  }
  ——————————————————————————————-

### 3.3 Layer 3 — Chainlink Oracle Verification
The Chainlink oracle layer (Layer 3) supersedes the need for C2PA's Certificate Authority trust model with distributed consensus verification. Multiple independent oracle nodes independently validate the same content and must reach consensus before any attestation is written on-chain. No single node can determine the outcome.

#### 3.3.1 Job Execution Flow
When a verification request is submitted to the smart contract, the Chainlink job executes the following pipeline:

7.  Order-Matching Contract selects a set of oracle nodes based on reputation scores

8.  Each node independently fetches the C2PA manifest from the URI provided in the request

9.  Each node executes the C2PA Validator Bridge — an external adapter that validates manifest signature, checks device certificate against Trust List, verifies content hash match, and scores confidence

10. Each node submits its result to the Aggregating Contract

11. Aggregating Contract computes weighted median of confidence scores, weighting by each node's historical accuracy via the Reputation Contract

12. Consensus result written to the ContentVerificationRegistry smart contract

#### 3.3.2 Reputation System
Oracle nodes accumulate reputation scores based on historical accuracy. A node whose submissions consistently match consensus gains reputation and receives more job assignments. A node whose submissions deviate from consensus loses reputation and eventually exits the active set. This creates a self-correcting system that improves over time without central administration.

#### 3.3.3 C2PA Validator Bridge Specification
The external adapter that each Chainlink node calls to validate a C2PA manifest. This is the critical integration point between the C2PA layer and the Chainlink oracle layer.

  ———————————————————————–
  Input schema:

  {

  \"contentHash\": \"string (SHA-256, hex-encoded)\",

  \"manifestURI\": \"string (HTTPS URI of C2PA manifest)\",

  \"requestId\": \"string (Chainlink request identifier)\"

  }

  Validation steps:

  1\. Fetch manifest from manifestURI

  2\. Verify JUMBF container integrity

  3\. Validate X.509 certificate chain to Trust List root

  4\. Verify manifest signature against device public key

  5\. Compare claimed content hash to registry request

  6\. Validate timestamp is within acceptable range

  7\. Check device certificate for revocation (OCSP)

  Output schema:

  {

  \"confidenceScore\": \"uint8 (0-100)\",

  \"signatureValid\": \"bool\",

  \"deviceTrusted\": \"bool\",

  \"hashMatch\": \"bool\",

  \"timestampValid\": \"bool\",

  \"revocationStatus\": \"enum(valid|revoked|unknown)\"

  }
  ———————————————————————–

### 3.4 Layer 4 — On-Chain Verifier
The blockchain layer provides permanent, platform-independent provenance records. Content hashes anchored on-chain are immutable — they cannot be altered, deleted, or stripped regardless of what platforms do to the file itself.

#### 3.4.1 ContentVerificationRegistry Contract
The core smart contract maintains the on-chain provenance registry. Key design decisions:

-   Content hash as primary key — SHA-256 of content bytes, not filename or metadata

-   Attestation is append-only — records can be added but not modified or deleted

-   Recirculation history tracked — each time content is reverified, the event is recorded

-   ZKP validation status stored separately — allows ZKP to be added post-registration

  ———————————————————————–
  struct ContentRecord {

  bytes32 contentHash;

  address verifiedBy; // Chainlink oracle address

  uint256 timestamp; // Block timestamp

  bool zkpValid; // ZKP proof validated

  uint8 confidenceScore; // 0-100 consensus score

  string c2paManifestURI; // IPFS URI of full manifest

  bool exists;

  }

  mapping(bytes32 => ContentRecord) public registry;

  mapping(bytes32 => uint256[]) public recirculationHistory;
  ———————————————————————–

#### 3.4.2 Cross-Chain via CCIP
Chainlink's Cross-Chain Interoperability Protocol enables the registry to be queried across any supported blockchain. A record anchored on Ethereum Mainnet is queryable from Polygon, Arbitrum, Base, or any CCIP-supported chain without re-registration. This is essential for ecosystem interoperability — a news organisation using one chain's infrastructure can verify content from a source using a different chain.

### 3.5 Layer 6 — Privacy Tier and Zero-Knowledge Proofs
The DECO/ZKP layer provides privacy-preserving verification. It allows the Chainlink oracle to verify that a C2PA manifest contains valid provenance claims without those claims being revealed to the oracle or appearing on-chain.

#### 3.5.1 DECO Protocol
DECO (Decentralised Oracle Commitment) is a three-phase protocol developed at Cornell University and integrated into Chainlink that enables proofs about TLS-authenticated data without revealing the data itself.

The three phases:

13. Three-party handshake: The Prover (content creator's device) and Verifier (Chainlink node) jointly establish a TLS session with the Server (C2PA manifest server) in secret-shared form. Neither party holds the complete session key. The server is unaware of DECO's participation.

14. Query execution: The Prover queries the server for the content's manifest record. The server responds with the full metadata — including sensitive fields. The Prover commits to this response cryptographically before the Verifier reveals their key share, preventing post-hoc alteration.

15. Proof generation: Using VOLE-based ZKP (Vector Oblivious Linear Evaluation), the Prover generates a proof demonstrating that specific claims about the manifest are true, revealing only the non-sensitive fields to the Verifier.

#### 3.5.2 Selective Disclosure Specification
0r1g1n defines the following disclosure policy. Fields marked REVEALED appear in the ZKP public inputs and may appear on-chain. Fields marked HIDDEN are proven to satisfy conditions without being revealed.

  ———————– —————– —————————————— —————————-
  Field                   Disclosure        Proven Condition                           Rationale

  Content hash            REVEALED          Matches registry request                   Required for chain linking

  Device on Trust List    REVEALED          Certificate validates to Trust List root   Core authenticity claim

  Timestamp range         REVEALED          Within [min, max] bounds                 Temporal authenticity

  Action type             REVEALED          Equals c2pa.created                        Proves original capture

  GPS coordinates         HIDDEN            Within declared region (optional)          Journalist safety

  Device serial number    HIDDEN            Is valid C2PA device                       Source protection

  Photographer identity   HIDDEN            Holds valid credential                     Source protection

  Exact timestamp         HIDDEN            Within range bounds                        Operational security
  ———————– —————– —————————————— —————————-

#### 3.5.3 ZKP Circuit Specification
For production deployment, 0r1g1n uses a Groth16 zk-SNARK circuit compiled using Circom 2.0. The circuit proves the following statement: given a C2PA manifest M retrieved from an authenticated TLS session with server S, the content hash H is valid, the device certificate C is on the Trust List T, and the timestamp t satisfies min <= t <= max — without revealing GPS coordinates, device serial number, or exact timestamp.

  ———————————————————————–
  // Circuit public inputs (revealed to verifier and on-chain)

  signal input contentHash[256];

  signal input trustListRootHash[256];

  signal input timestampMin;

  signal input timestampMax;

  // Circuit private inputs (proven without revealing)

  signal input deviceSerialNumber[256];

  signal input gpsLatitude;

  signal input gpsLongitude;

  signal input exactTimestamp;

  signal input deviceCertificate[2048];

  signal input manifestSignature[512];

  // Verification constraints

  // 1. Timestamp in range

  exactTimestamp >= timestampMin;

  exactTimestamp <= timestampMax;

  // 2. Certificate validates to Trust List root

  // (certificate chain verification sub-circuit)

  // 3. Manifest signature valid over content hash

  // (ECDSA verification sub-circuit)

  // Output

  signal output verified <==

  timestampValid \* certValid \* sigValid;
  ———————————————————————–

## 4. Inter-Layer Protocol Specification
### 4.1 Registration Protocol
The sequence of operations when new content is captured and registered in 0r1g1n.

  ———————————————————————–
  1\. CAPTURE

  Device signs content at hardware level

  Output: signedFile, manifestURI, contentHash

  2\. WATERMARK EMBEDDING

  Pixel watermark embedding contentHash + softBindingURI

  into image data (survives platform recompression)

  3\. VERIFICATION REQUEST

  Client calls: registry.requestVerification(

  contentHash,

  manifestURI

  )

  Query fee transferred to Chainlink oracle

  4\. ORACLE EXECUTION

  Chainlink DON executes C2PA validation job

  Each node: fetch -> validate -> score

  Aggregating contract: weighted median consensus

  5\. ZKP GENERATION (parallel or sequential)

  DECO three-party handshake with manifest server

  ZKP proof generated for selective disclosure

  Proof submitted to on-chain ZKP verifier

  6\. ON-CHAIN REGISTRATION

  registry.fulfillVerification(

  contentHash, zkpValid, confidenceScore

  )

  Immutable record created — timestamp, score, ZKP status
  ———————————————————————–

### 4.2 Verification Protocol
The sequence of operations when a consumer queries the provenance of a piece of content.

  ———————————————————————–
  1\. CONTENT RECEIVED (image/video with or without metadata)

  2\. HASH EXTRACTION

  If C2PA metadata present: extract hash from manifest

  Else if watermark present: extract hash from pixel watermark

  Else: compute hash from content bytes (unregistered content)

  3\. BLOCKCHAIN QUERY

  registry.verifyContent(contentHash)

  Returns: verified, timestamp, confidenceScore, zkpValid

  4\. SOFT BINDING RECOVERY (if metadata stripped)

  GET /api/v1/manifest?contentHash={hash}

  Returns: full attestation record in C2PA manifest format

  5\. TRUST LEVEL ASSIGNMENT

  HIGH: zkpValid AND blockchainVerified AND c2paPresent

  MEDIUM: blockchainVerified AND c2paPresent

  LOW: blockchainVerified only (metadata stripped)

  UNVERIFIED: no record found

  6\. RESULT DISPLAY

  Verification badge with trust level and audit trail
  ———————————————————————–

### 4.3 Recirculation Detection Protocol
Chainlink Automation monitors on-chain records and triggers reverification when content is detected in a new context. Smart contract logic determines whether the new context constitutes a material change requiring human review.

  ———————————————————————–
  // Chainlink Automation condition

  function checkUpkeep(bytes calldata) external view

  returns (bool upkeepNeeded, bytes memory performData) {

  upkeepNeeded = recirculationQueue.length > 0;

  performData = abi.encode(recirculationQueue[0]);

  }

  function performUpkeep(bytes calldata performData) external {

  (bytes32 contentHash, string memory newContext) =

  abi.decode(performData, (bytes32, string));

  // Record recirculation event

  recirculationHistory[contentHash].push(block.timestamp);

  // Flag for human review if context change is significant

  if (contextChangeScore(contentHash, newContext) > THRESHOLD) {

  emit RecirculationAlert(contentHash, newContext);

  }

  }
  ———————————————————————–

## 5. Implementation Guidance
### 5.1 Technology Stack
  ————————— ————————————————————————————————————————————————-
  **Component**               **Technology**

  **C2PA signing service**    Rust with c2pa crate (minimum version 1.88.0). Certificate from DigiCert or SSL.com C2PA-conformant CA.

  **Smart contract**          Solidity 0.8.19+. Foundry for development and testing. OpenZeppelin for access control. Chainlink contracts for oracle integration.

  **Chainlink node**          Docker + PostgreSQL on Ubuntu 22.04. Minimum 4 CPU cores, 8GB RAM, 50GB SSD. WebSocket-capable Ethereum RPC endpoint.

  **C2PA validator bridge**   Node.js external adapter. c2pa npm package for manifest validation. Exposed as HTTP service on node infrastructure.

  **ZKP circuit**             Circom 2.0 for circuit definition. SnarkJS for trusted setup and proof generation. Groth16 proving system. DECO Sandbox for initial deployment.

  **Soft binding**            Python watermarking service using imwatermark library (dwtDctSvd algorithm). Express.js for Soft Binding Resolution API.

  **Verification SDK**        JavaScript/TypeScript. Ethers.js for blockchain queries. c2pa-js for manifest validation. Distributed as NPM package and browser extension.
  ————————— ————————————————————————————————————————————————-

### 5.2 Deployment Sequence
Components must be deployed in dependency order. The following sequence minimises integration risk.

  —————————- —————————————————————————————————————————————————————–
  **Phase**                    **Deliverables**

  **Phase 1 (Weeks 1–6)**     C2PA signing service on testnet. Smart contract deployed to Sepolia. Manual verification flow proving end-to-end hash registration works before any automation.

  **Phase 2 (Weeks 7–12)**    Chainlink node operational. C2PA validator bridge deployed and registered. Automated oracle job executing on testnet with real camera hardware.

  **Phase 3 (Weeks 13–16)**   DECO Sandbox integration. Soft Binding Resolution API live. Browser extension alpha release. Pilot programme with first news organisation.

  **Phase 4 (Weeks 17–24)**   ZKP circuit deployed to testnet. Full ZKP proof generation integrated. Mainnet deployment. Cross-chain via CCIP.

  **Phase 5 (Weeks 25–36)**   Mobile SDK. AI content labelling pipeline. Production SLA. Series A preparation.
  —————————- —————————————————————————————————————————————————————–

### 5.3 Testing Requirements
The following test categories are mandatory before mainnet deployment.

-   Unit tests: Each component in isolation. C2PA validator bridge must achieve >98% accuracy on the C2PA conformance test suite.

-   Integration tests: Full pipeline from camera capture to blockchain record. Must complete end-to-end in under 30 seconds for 95th percentile.

-   Adversarial tests: Attempt metadata stripping, replay attacks, and tampered manifest submission. All must fail to produce false positive attestations.

-   Smart contract audit: Independent security audit by a recognised Web3 security firm before mainnet deployment. Mandatory — not optional.

-   ZKP circuit audit: Separate cryptographic audit of circuit correctness. Circuit soundness failure would allow false proofs — this is a critical security property.

-   Load testing: Verify oracle network throughput at 10x expected peak load before production launch.

## 6. Security Analysis
### 6.1 Cryptographic Foundations
0r1g1n's security rests on three cryptographic foundations whose hardness assumptions are well-established and widely deployed.

  —————————– ————————————————————————————————————————————————————————
  **Primitive**                 **Security Basis**

  **ECDSA (secp256k1/P-256)**   Used for C2PA manifest signing and blockchain transactions. Security relies on elliptic curve discrete logarithm problem. No known classical attack below O(2\^128).

  **SHA-256**                   Used for content hashing. Security relies on collision resistance. No known practical collision attacks.

  **Groth16 zk-SNARK**          Used for ZKP circuit. Security relies on knowledge-of-exponent assumption in bilinear groups. Requires trusted setup — multi-party computation ceremony recommended.
  —————————– ————————————————————————————————————————————————————————

### 6.2 Quantum Resistance
All three cryptographic foundations are vulnerable to Cryptographically Relevant Quantum Computers (CRQCs) using Shor's algorithm. Current consensus among cryptographers is that CRQCs capable of breaking these primitives are 10–20 years from deployment. NIST has finalised post-quantum standards (ML-KEM, ML-DSA, SLH-DSA) that provide migration paths. 0r1g1n's architecture is modular — cryptographic primitives can be upgraded without redesigning the protocol. A migration roadmap to post-quantum primitives should be planned for the 5-year horizon. All data structures include a cryptographic_algorithm field to support future backwards-compatible migration to post-quantum primitives (ML-DSA, SLH-DSA) when supported by C2PA device firmware.

### 6.3 Smart Contract Security
The ContentVerificationRegistry contract presents the following attack surfaces requiring explicit mitigation.

  ————————- ———————————————————————————————————————————————————-
  **Attack Vector**         **Mitigation**

  **Reentrancy**            Mitigated by checks-effects-interactions pattern and OpenZeppelin ReentrancyGuard. All state changes before external calls.

  **Oracle manipulation**   Mitigated by Chainlink distributed consensus — requires majority of nodes to collude. Node reputation system increases cost of sustained manipulation.

  **Front-running**         Registration transactions visible in mempool. Commit-reveal scheme for sensitive registrations. Chainlink VRF for randomness-dependent operations.

  **Access control**        Owner-only functions protected by OpenZeppelin Ownable. Multisig recommended for production owner key. Time-lock on parameter changes.

  **Gas limit**             Aggregation computation bounded by node count. Maximum node set size parameter mitigates unbounded loops.
  ————————- ———————————————————————————————————————————————————-

## 7. Acknowledged Limitations
### 7.1 Cryptographically Irresolvable Limitations
+———————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————–+
| **The First-Mile Problem**                                                                                                                                                                                                                                                                                                                                                                                            |
|                                                                                                                                                                                                                                                                                                                                                                                                                       |
| 0r1g1n verifies the chain of custody from the moment of signing. It cannot verify the truthfulness of what the camera was pointed at. A C2PA-enabled camera capturing staged or fabricated events produces a perfectly authenticated record of a lie. Provenance systems certify custody chains, not ground truth. This is a fundamental limitation of the approach and is not resolvable by any cryptographic means. |
+———————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————–+

+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————-+
| **The Caption Problem**                                                                                                                                                                                                                                                                                                                                                       |
|                                                                                                                                                                                                                                                                                                                                                                               |
| A validly credentialled photograph can be distributed with a false caption or in a misleading context. 0r1g1n records what a device captured and that it has not been altered. It does not record what the caption says or whether the context is accurate. This requires complementary approaches: metadata standards for caption provenance, and human editorial judgement. |
+——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————-+

### 7.2 Practical Limitations
  ——————————- ——————————————————————————————————————————————————————————————————————————————
  **Limitation**                  **Nature and Mitigation**

  **Opt-in architecture**         Malicious actors will not use C2PA-enabled hardware or register content. The system certifies the authentic; it cannot compel the inauthentic to identify itself. Value comes from the authenticated population, not universal coverage.

  **Oracle data feed security**   Chainlink oracle nodes fetch external data. Compromised data sources could cause nodes to attest false information. Mitigated by node diversity and source cross-referencing, not eliminated.

  **Trusted setup requirement**   Groth16 ZKP circuits require a trusted setup ceremony. Compromise of the setup produces false proofs that are indistinguishable from valid ones. Multi-party computation ceremony with public verifiability is mandatory.

  **Computational cost of ZKP**   Proof generation is computationally intensive. Current benchmarks on 720p images: approximately 3–8 seconds on modern hardware. Real-time video verification at capture requires hardware acceleration not yet widely deployed.

  **Adoption asymmetry**          System value scales with adoption. Early deployment has limited coverage. Value increases non-linearly as more content creators, platforms, and verifiers participate.
  ——————————- ——————————————————————————————————————————————————————————————————————————————

## 8. Related Work
0r1g1n builds on and integrates work from four distinct research and deployment traditions.

### 8.1 Content Provenance Standards
The C2PA specification (Coalition for Content Provenance and Authenticity, 2021–2026) provides the hardware signing and manifest format foundations. The Content Authenticity Initiative (Adobe, 2019) contributed the original content credentials concept. Numbers Protocol has demonstrated C2PA-blockchain hybrid deployment at production scale, providing validated architecture patterns this specification extends.

### 8.2 Decentralised Oracle Networks
Chainlink (Nazarov and Ellis, 2017) established the decentralised oracle network architecture. The DECO protocol (Zhang et al., 2020) introduced ZKP-based TLS proofs without server modification, later integrated into the Chainlink platform. The Chainlink CCIP specification (2023) provides the cross-chain interoperability foundation.

### 8.3 Zero-Knowledge Proofs Applied to Media
PhotoProof (Naveh and Tromer, 2016) first proposed ZKP-based image authentication. The zk-REAL system (2025) demonstrated practical ZKP verification for C2PA manifests with hardware-compatible performance characteristics. Halo2 circuit design improvements have significantly reduced proving time for image transformation verification.

### 8.4 Blockchain Media Provenance
Starling Lab (USC/Stanford, 2020–) has demonstrated blockchain-based archival of conflict documentation for legal proceedings, establishing the evidentiary use case. The Fact Protocol (2022) introduced economic incentive mechanisms for decentralised fact-checking, informing 0r1g1n's governance design.

## 9. Conclusion
0r1g1n presents a complete, deployable architecture for digital content verification that addresses the systemic failures of existing single-component approaches. By combining C2PA hardware signing, Chainlink decentralised oracle verification, blockchain immutable records, and DECO-based Zero-Knowledge Proofs, the architecture achieves properties that no individual component provides: a root of trust at the point of physical capture; decentralised consensus replacing single-authority certification; platform-independent records that survive any distribution pipeline; and privacy-preserving verification that protects content creators.

All four component technologies are deployed at institutional scale in 2026. The contribution of this specification is their integration — the inter-layer protocol that connects them into a system where each component's weaknesses are closed by the others. The result is an architecture in which the question shifts from can we detect the fake to can we certify the real. The former is a race that cannot be won at machine speed. The latter is infrastructure that can be built.

The residual limitations are acknowledged and important. No provenance system can address the first-mile problem — a camera pointed at a staged scene produces authentic records of fabricated reality. No technical architecture resolves the human adoption problem — infrastructure helps only those who choose to use it. 0r1g1n is necessary but not sufficient. It is the technical foundation on which the other necessary components — media literacy, regulatory enforcement, editorial standards — can operate more effectively.

The code is buildable today. The tools exist. The moment to build this infrastructure is now.

## Appendix A — Technology References
  ————————————- —————————————————–
  **Resource**                          **Reference**

  **C2PA Specification**                https://c2pa.org/specifications/specifications/

  **C2PA Rust SDK (c2pa crate)**        https://crates.io/crates/c2pa

  **C2PA JavaScript SDK**               https://www.npmjs.com/package/c2pa

  **Content Authenticity Initiative**   https://contentauthenticity.org

  **Chainlink Documentation**           https://docs.chain.link

  **Chainlink DECO**                    https://research.chain.link/deco.pdf

  **Chainlink CCIP**                    https://chain.link/cross-chain

  **Numbers Protocol**                  https://www.numbersprotocol.io

  **Circom (ZKP circuit language)**     https://docs.circom.io

  **SnarkJS**                           https://github.com/iden3/snarkjs

  **OpenZeppelin Contracts**            https://openzeppelin.com/contracts

  **Foundry (Solidity framework)**      https://getfoundry.sh

  **C2PA Verify Tool**                  https://verify.contentauthenticity.org

  **Soft Binding Resolution API**       https://c2pa.org/specifications/specifications/2.1/
  ————————————- —————————————————–

## Appendix B — Glossary
  ———————— ————————————————————————————————————————–
  **Term**                 **Definition**

  **C2PA**                 Coalition for Content Provenance and Authenticity. Standard for signing and verifying the provenance of digital content.

  **CCIP**                 Cross-Chain Interoperability Protocol. Chainlink standard for communication between blockchain networks.

  **Content Credential**   A C2PA manifest — the cryptographically signed record of a piece of content's origin and history.

  **DECO**                 Decentralised Oracle Commitment. Cornell/Chainlink protocol for zero-knowledge proofs about TLS-authenticated data.

  **DON**                  Decentralised Oracle Network. Chainlink's network of independent oracle nodes.

  **Groth16**              A zk-SNARK proving system producing constant-size proofs. Requires trusted setup.

  **JUMBF**                JPEG Universal Metadata Box Format. Container format used by C2PA for embedding manifests.

  **LINK**                 Chainlink's native token. Used to pay for oracle services and stake by node operators.

  **Soft Binding**         C2PA 2.1 mechanism for recovering credentials from a pixel watermark when embedded metadata has been stripped.

  **TLS**                  Transport Layer Security. The cryptographic protocol underlying HTTPS.

  **VOLE-ZKP**             Vector Oblivious Linear Evaluation Zero-Knowledge Proof. Interactive ZKP used by DECO for efficiency.

  **ZKP**                  Zero-Knowledge Proof. A cryptographic method to prove a statement is true without revealing the underlying data.

  **zk-SNARK**             Zero-Knowledge Succinct Non-Interactive Argument of Knowledge. A form of ZKP producing short, quickly verifiable proofs.
  ———————— ————————————————————————————————————————–

0r1g1n Technical Whitepaper v1.1 | March 2026
