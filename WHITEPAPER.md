# 0r1g1n — Technical Whitepaper

**A Hybrid Architecture for Decentralised Content Verification**

> C2PA Hardware Signing · Chainlink Oracle Network · Blockchain Ledger · Zero-Knowledge Proofs

**Version:** 1.1 | **Date:** March 2026 | **Status:** DRAFT  
**Licence:** Apache 2.0 | **Repository:** github.com/0r1g1n-protocol

---

## Abstract

We present 0r1g1n, a hybrid content verification architecture that combines four proven technologies to provide cryptographically verifiable provenance for digital media from the moment of capture through distribution and consumption. The architecture addresses the systemic failure of existing single-component approaches by assigning each technology the function it was specifically designed for: C2PA hardware signing provides a root of trust at the point of physical capture; a Verifiable Data Registry (VDR) abstract interface provides platform-independent provenance storage with ZK proof of query correctness (Space and Time is the reference implementation); the Chainlink Decentralised Oracle Network supersedes the need for single-authority certificate infrastructure with distributed consensus; an immutable blockchain ledger provides platform-independent records that survive metadata stripping; and DECO-based Zero-Knowledge Proofs enable privacy-preserving verification that protects content creators without sacrificing authenticity guarantees.

The architecture is designed for production deployment using tools and APIs available in 2026. No component requires novel research — all four layers are deployed at institutional scale in adjacent domains. The contribution of this paper is the integration specification that combines them into a coherent system that addresses all identified architectural weaknesses.

---

## Contents

1. [Introduction](#1-introduction)
2. [Threat Model](#2-threat-model)
3. [Architecture](#3-architecture)
4. [Inter-Layer Protocol Specification](#4-inter-layer-protocol-specification)
5. [Implementation Guidance](#5-implementation-guidance)
6. [Security Analysis](#6-security-analysis)
7. [Acknowledged Limitations](#7-acknowledged-limitations)
8. [Related Work](#8-related-work)
9. [Conclusion](#9-conclusion)
- [Appendix A — Technology References](#appendix-a--technology-references)
- [Appendix B — Glossary](#appendix-b--glossary)

---

## 1. Introduction

### 1.1 The Authenticity Problem

Digital media authenticity has broken down as a reliable social institution. The convergence of generative AI, optimised distribution infrastructure, and geopolitical information warfare has produced an environment in which the cost of creating convincing synthetic media has dropped to near zero while the cost of verifying authentic media remains high.

Existing detection approaches — AI classifiers, human review, watermark scanning — share a structural weakness: they are reactive. They attempt to identify the fake after it has been created and circulated. Against an adversary generating content at machine speed, reactive detection cannot win at scale. The correct frame is not detecting the fake but certifying the real — establishing infrastructure in which authentic content is verifiably, cryptographically, and immutably itself regardless of what happens to it in distribution.

### 1.2 The Gap in Existing Solutions

C2PA (Coalition for Content Provenance and Authenticity) is the most mature existing standard and represents the correct instinct — hardware-level signing at the point of capture, embedded credentials, a standardised manifest format. Its weaknesses are well-documented: credentials are stripped by every major social platform on upload; the Certificate Authority trust model creates single points of failure; the metadata carries sensitive information that endangers journalists and whistleblowers.

Blockchain-based approaches address the stripping problem by anchoring records independently of the file. But without a hardware signing layer they have no root of trust. Oracle networks provide decentralised verification but require the other components to have anything to verify. Zero-Knowledge Proofs provide privacy guarantees but require an existing verification infrastructure to apply them to.

0r1g1n's contribution is not any individual component — all four exist and are deployed at scale — but their integration into a system in which each component's weaknesses are closed by the others.

### 1.3 Scope of This Paper

Section 2 defines the threat model. Section 3 describes each component layer. Section 4 specifies the inter-layer protocol. Section 5 provides implementation guidance. Section 6 addresses security analysis. Section 7 acknowledges residual limitations. Section 8 discusses related work.

---

## 2. Threat Model

### 2.1 Adversary Classes

0r1g1n is designed against three adversary classes of increasing capability.

| Adversary Class | Capabilities |
|----------------|-------------|
| **Class I — Opportunistic** | Consumer-accessible tools. Fabricated content shared without attempting to forge provenance. Largest by volume. |
| **Class II — Motivated** | Technical capability to forge or strip credentials. May attempt metadata manipulation, watermark removal, or replay attacks. |
| **Class III — State-level** | Resources to compromise Certificate Authorities, operate fake verification nodes, or conduct cryptographic attacks. Able to stage events and produce genuinely captured but deliberately deceptive content. |

0r1g1n provides strong security properties against Class I and II adversaries. Against Class III, the architecture raises the cost of attack significantly but cannot provide absolute guarantees.

### 2.2 Attack Surface

| Attack Vector | Mitigation |
|--------------|------------|
| Metadata stripping | Blockchain ledger independence and pixel watermark soft binding. |
| Certificate compromise | Chainlink distributed consensus — no single CA in the trust path. |
| Replay attack | Content hash binding — credential is cryptographically bound to specific content bytes. |
| Oracle manipulation | Chainlink consensus requiring majority of nodes to agree. |
| Privacy extraction | DECO/ZKP — sensitive fields never revealed to oracle or on-chain. |
| First-mile manipulation | Hardware integration at the sensor chipset level. |
| Staged capture | Not addressable by any provenance system — acknowledged residual limitation. |

---

## 3. Architecture

### 3.1 Layer Overview

| Layer | Name | Function | Trust Basis |
|-------|------|----------|-------------|
| 1 | C2PA Hardware Signing | Hardware signing at capture — root of trust | Hardware certificate |
| 2 | Verifiable Data Registry | Provenance storage with ZK proof of query correctness | VDR proof mechanism |
| 3 | Chainlink Oracle | Decentralised consensus verification | Distributed consensus |
| 4 | On-Chain Verifier | Immutable event log | Cryptographic immutability |
| 5 | Soft Binding | Watermark-based credential recovery | — |
| 6 *(optional)* | Privacy Tier / ZKP | Privacy-preserving proof for high-risk contexts | Mathematical proof |

### 3.2 Layer 1 — C2PA Hardware Signing

The C2PA layer provides the system's root of trust. Signing occurs at the sensor chipset level, before image data is accessible to any software layer.

#### 3.2.1 Signing Process

1. Raw sensor data captured and buffered in the secure enclave
2. SHA-256 hash computed over raw content bytes
3. C2PA manifest assembled: content hash, device certificate reference, ISO 8601 timestamp, `c2pa.created` action assertion
4. Manifest signed using the device's X.509 private key (hardware security module — never extractable)
5. Signed manifest embedded in file using JUMBF container
6. Content hash made available via API endpoint for Layer 2 VDR ingest

#### 3.2.2 Certificate Chain

```
C2PA Root CA (Trust List anchor)
└── Camera Manufacturer Intermediate CA
     └── Device Certificate (per-device, hardware-bound)
          └── Content Manifest Signature
```

```json
{
  "claim_generator": "ManufacturerName/DeviceModel/1.0",
  "title": "<SHA-256 content hash>",
  "assertions": [
    { "label": "c2pa.actions",
      "data": { "actions": [{ "action": "c2pa.created" }] } },
    { "label": "stds.exif",
      "data": {
        "Exif:DateTimeOriginal": "<ISO8601>",
        "Exif:GPSLatitude": "<REDACTED_IN_ZKP>"
      }
    }
  ],
  "cryptographic_algorithm": "ES256"
}
```

### 3.3 Layer 3 — Chainlink Oracle Verification

The Chainlink oracle layer supersedes the need for C2PA's Certificate Authority trust model with distributed consensus verification.

#### 3.3.1 Job Execution Flow

1. Order-Matching Contract selects oracle nodes based on reputation scores
2. Each node independently fetches the C2PA manifest
3. Each node runs the C2PA Validator Bridge
4. Each node submits its result to the Aggregating Contract
5. Aggregating Contract computes weighted median of confidence scores
6. Consensus result written to the on-chain verifier contract

#### 3.3.2 C2PA Validator Bridge

```json
// Input
{
  "contentHash": "string (SHA-256 hex)",
  "manifestURI": "string (HTTPS URI)"
}

// Output
{
  "confidenceScore": "uint8 (0-100)",
  "signatureValid": "bool",
  "deviceTrusted": "bool",
  "hashMatch": "bool",
  "timestampValid": "bool",
  "revocationStatus": "valid | revoked | unknown"
}
```

### 3.4 Layer 4 — On-Chain Verifier

Provides permanent, platform-independent provenance records. Content hashes anchored on-chain cannot be altered, deleted, or stripped.

```solidity
struct ContentRecord {
    bytes32 contentHash;
    address verifiedBy;
    uint256 timestamp;
    bool    zkpValid;
    uint8   confidenceScore;
    bool    exists;
}

mapping(bytes32 => ContentRecord) public registry;
mapping(bytes32 => uint256[])     public recirculationHistory;
```

Chainlink's Cross-Chain Interoperability Protocol (CCIP) enables the registry to be queried across any supported blockchain without re-registration.

### 3.5 Layer 6 — Privacy Tier and Zero-Knowledge Proofs

Allows the Chainlink oracle to verify provenance claims without those claims being revealed to the oracle or appearing on-chain.

#### 3.5.1 DECO Protocol

DECO enables proofs about TLS-authenticated data without revealing the data itself or requiring server-side modification. Three phases:

1. **Three-party handshake:** Prover and Verifier jointly establish a TLS session in secret-shared form
2. **Query execution:** Prover queries the server and commits to the response before key reveal
3. **Proof generation:** VOLE-based ZKP proves claims about the manifest without revealing sensitive fields

#### 3.5.2 Selective Disclosure Policy

| Field | Disclosure | Proven Condition |
|-------|-----------|-----------------|
| Content hash | REVEAL | Matches registry request |
| Device on Trust List | REVEAL | Certificate validates to Trust List root |
| Timestamp range | REVEAL | Within [min, max] bounds |
| Action type | REVEAL | Equals `c2pa.created` |
| GPS coordinates | HIDDEN | Within declared region (optional) |
| Device serial number | HIDDEN | Is valid C2PA device |
| Photographer identity | HIDDEN | Holds valid credential |
| Exact timestamp | HIDDEN | Within range bounds |

---

## 4. Inter-Layer Protocol Specification

### 4.1 Registration Protocol

```
1. CAPTURE
   Device signs content at hardware level
   Output: signedFile, manifestURI, contentHash

2. WATERMARK EMBEDDING
   Embed contentHash + registryEndpoint in pixel data

3. VDR INGEST
   VDR.registerProvenance(contentHash, manifestData)
   Status: PENDING

4. VERIFICATION REQUEST
   registry.requestVerification(contentHash, manifestURI)

5. ORACLE EXECUTION
   Chainlink DON: fetch → validate → score → consensus

6. ZKP GENERATION (optional — Layer 6 only)
   DECO handshake → proof generation → on-chain submission

7. ON-CHAIN REGISTRATION
   Emit ContentVerified event
   Status: VERIFIED
```

### 4.2 Verification Protocol

```
1. HASH EXTRACTION
   C2PA metadata present → extract from manifest
   Watermark present     → extract from watermark, resolve full hash
   Neither               → compute SHA-256 (likely UNVERIFIED)

2. REVOCATION CHECK
   Query on-chain RegistryStatus for device certificate

3. VDR QUERY
   record, proof = VDR.getProvenanceWithProof(contentHash)
   Verify VDR proof

4. TRUST LEVEL ASSIGNMENT
   HIGH:        c2paPresent AND verified AND score >= 80 AND not COMPROMISED
   MEDIUM:      verified AND score >= 60
   LOW:         verified AND score < 60
   PROVISIONAL: verified AND VDR unavailable (cached result)
   UNVERIFIED:  no registry record found
   COMPROMISED: device certificate flagged as compromised
```

---

## 5. Implementation Guidance

### 5.1 Technology Stack

| Component | Technology |
|-----------|-----------|
| C2PA signing service | Rust, `c2pa` crate (min 1.88.0). C2PA-conformant CA certificate. |
| Smart contract | Solidity 0.8.19+. Foundry. OpenZeppelin. |
| Chainlink node | Docker + PostgreSQL, Ubuntu 22.04. Min 4 cores / 8GB RAM. |
| C2PA validator bridge | Node.js external adapter. `c2pa` npm package. |
| ZKP circuit | Circom 2.0. SnarkJS. Groth16. DECO Sandbox for initial deployment. |
| Soft binding | Python, `imwatermark` library (dwtDctSvd). |
| Verification SDK | TypeScript. Ethers.js. `c2pa-js`. |

### 5.2 Deployment Sequence

| Phase | Timeline | Deliverables |
|-------|----------|-------------|
| 1 | Weeks 1–6 | C2PA signing service. Smart contract on Sepolia testnet. |
| 2 | Weeks 7–12 | Chainlink node. C2PA validator bridge. Automated oracle job. |
| 3 | Weeks 13–16 | DECO Sandbox. Soft Binding API. Browser extension alpha. Pilot. |
| 4 | Weeks 17–24 | ZKP circuit. Full proof generation. Mainnet. CCIP. |
| 5 | Weeks 25–36 | Mobile SDK. AI content labelling. Production SLA. |

### 5.3 Testing Requirements

- **Unit tests:** C2PA validator bridge must achieve >98% accuracy on C2PA conformance suite
- **Integration tests:** End-to-end under 30 seconds at 95th percentile
- **Adversarial tests:** Metadata stripping, replay attacks, tampered manifests — all must fail
- **Smart contract audit:** Independent Web3 security firm — mandatory before mainnet
- **ZKP circuit audit:** Cryptographic correctness audit — mandatory before Level 3 conformance claim
- **Load testing:** 10x expected peak load before production launch

---

## 6. Security Analysis

### 6.1 Cryptographic Foundations

| Primitive | Security Basis |
|-----------|---------------|
| ECDSA (P-256 / secp256k1) | Elliptic curve discrete logarithm. No known classical attack below O(2¹²⁸). |
| SHA-256 | Collision resistance. No known practical collision attacks. |
| Groth16 zk-SNARK | Knowledge-of-exponent assumption. Requires MPC trusted setup. |

### 6.2 Quantum Resistance

All current primitives are vulnerable to CRQCs via Shor's algorithm. CRQCs at this scale are not expected within the deployment horizon of this version. NIST post-quantum standards (ML-DSA, SLH-DSA) provide migration paths. All 0r1g1n data structures include a `cryptographic_algorithm` field to support future backwards-compatible migration without breaking changes.

### 6.3 Smart Contract Security

| Attack Vector | Mitigation |
|--------------|------------|
| Reentrancy | Checks-effects-interactions pattern. OpenZeppelin ReentrancyGuard. |
| Oracle manipulation | Distributed consensus. Reputation system. |
| Front-running | Commit-reveal scheme. Chainlink VRF where applicable. |
| Access control | OpenZeppelin Ownable. Multisig for production. Time-lock on changes. |
| Gas limit | Node count bounded. Maximum set size parameter. |

---

## 7. Acknowledged Limitations

### 7.1 Cryptographically Irresolvable

> **The First-Mile Problem:** 0r1g1n verifies the chain of custody from the moment of signing. It cannot verify the truthfulness of what the camera was pointed at. A C2PA-enabled camera capturing staged or fabricated events produces a perfectly authenticated record of a lie. This is a permanent and irresolvable limitation of all provenance systems.

> **The Caption Problem:** A validly credentialled photograph can be distributed with a false caption. 0r1g1n records what was captured and that it has not been altered. What it depicts remains a human judgement.

### 7.2 Practical Limitations

| Limitation | Nature |
|-----------|--------|
| Opt-in architecture | Malicious actors will not participate. The system certifies the authentic, not the total. |
| Oracle data feed security | Compromised data sources can cause false attestations. Mitigated by node diversity — not eliminated. |
| Trusted setup | Groth16 requires MPC ceremony. Compromise produces undetectable false proofs. |
| ZKP computational cost | 3–8 seconds on modern hardware for 720p images. Hardware acceleration required for real-time video. |
| Adoption asymmetry | Value scales non-linearly with adoption. Early deployment has limited coverage. |

---

## 8. Related Work

**Content Provenance Standards:** C2PA specification (2021–2026), Content Authenticity Initiative (Adobe, 2019), Numbers Protocol production deployment.

**Decentralised Oracle Networks:** Chainlink (Nazarov and Ellis, 2017), DECO protocol (Zhang et al., 2020), Chainlink CCIP (2023).

**ZKP Applied to Media:** PhotoProof (Naveh and Tromer, 2016), zk-REAL system (2025), Halo2 circuit design improvements.

**Blockchain Media Provenance:** Starling Lab (USC/Stanford, 2020–), Fact Protocol (2022).

---

## 9. Conclusion

0r1g1n presents a complete, deployable architecture for digital content verification. By combining C2PA hardware signing, Chainlink oracle verification, blockchain immutable records, and DECO-based ZKPs, the architecture achieves properties that no individual component provides.

The residual limitations are acknowledged. No provenance system addresses the first-mile problem. No technical architecture resolves the human adoption problem. 0r1g1n is the technical foundation on which the other necessary components — media literacy, regulatory enforcement, editorial standards — can operate more effectively.

The code is buildable today. The tools exist. The moment to build this infrastructure is now.

---

## Appendix A — Technology References

| Resource | URL |
|----------|-----|
| C2PA Specification | https://c2pa.org/specifications/specifications/ |
| C2PA Rust SDK | https://crates.io/crates/c2pa |
| C2PA JavaScript SDK | https://www.npmjs.com/package/c2pa |
| Content Authenticity Initiative | https://contentauthenticity.org |
| Chainlink Documentation | https://docs.chain.link |
| Chainlink DECO | https://research.chain.link/deco.pdf |
| Chainlink CCIP | https://chain.link/cross-chain |
| Numbers Protocol | https://www.numbersprotocol.io |
| Circom | https://docs.circom.io |
| SnarkJS | https://github.com/iden3/snarkjs |
| OpenZeppelin Contracts | https://openzeppelin.com/contracts |
| Foundry | https://getfoundry.sh |
| C2PA Verify Tool | https://verify.contentauthenticity.org |
| Soft Binding Resolution API | https://c2pa.org/specifications/specifications/2.1/ |

---

## Appendix B — Glossary

| Term | Definition |
|------|-----------|
| C2PA | Coalition for Content Provenance and Authenticity. Standard for signing and verifying digital content provenance. |
| CCIP | Cross-Chain Interoperability Protocol. Chainlink standard for communication between blockchains. |
| Content Credential | A C2PA manifest — the cryptographically signed record of content origin and history. |
| DECO | Decentralised Oracle Commitment. Cornell/Chainlink protocol for ZKPs about TLS-authenticated data. |
| DON | Decentralised Oracle Network. Chainlink's network of independent oracle nodes. |
| Groth16 | A zk-SNARK proving system producing constant-size proofs. Requires trusted setup. |
| JUMBF | JPEG Universal Metadata Box Format. Container used by C2PA for embedding manifests. |
| Soft Binding | C2PA 2.1 mechanism for recovering credentials from a pixel watermark. |
| TLS | Transport Layer Security. The cryptographic protocol underlying HTTPS. |
| VDR | Verifiable Data Registry. Abstract Layer 2 interface for provenance storage. |
| VOLE-ZKP | Vector Oblivious Linear Evaluation ZKP. Interactive ZKP used by DECO. |
| ZKP | Zero-Knowledge Proof. Proves a statement is true without revealing underlying data. |
| zk-SNARK | Zero-Knowledge Succinct Non-Interactive Argument of Knowledge. |

---

*0r1g1n Technical Whitepaper v1.1 | March 2026 | Apache 2.0*
