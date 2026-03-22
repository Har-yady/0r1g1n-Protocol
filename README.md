# 0r1g1n

**Open Provenance Verification Protocol**

> \*Certify the real. Not detect the fake.\*

0r1g1n is an open framework specification for verifiable digital media provenance. It defines the interfaces, data formats, and protocol behaviours required to establish cryptographically verifiable chains of custody for digital content — from the moment of capture through distribution and consumption.

\---

## The Problem

A photograph taken in a conflict zone. A video of a public official. A document leaked by a whistleblower. Each of these can be fabricated, manipulated, or stripped of context before most people see it.

Existing solutions address parts of this problem. C2PA hardware signing creates a root of trust at capture. Blockchain ledgers create tamper-evident records. Detection tools attempt to identify synthetic content. None of them close all the gaps — and none of them protect the people creating the content while doing it.

0r1g1n is the integration layer that connects these tools into a coherent, gap-free system.

\---

## How It Works

0r1g1n combines four proven technologies, each doing what it was designed for:

|Layer|Technology|Function|
|-|-|-|
|1|C2PA Hardware Signing|Signs content cryptographically at the sensor level — before any software can touch it|
|2|Verifiable Data Registry|Stores provenance records with ZK proof of query correctness. Reference implementation: Space and Time|
|3|Chainlink Oracle Network|Decentralised consensus verification — no single authority can certify or deny|
|4|Blockchain Ledger|Immutable event log that survives metadata stripping by any platform|
|5|Soft Binding Watermark|Pixel-embedded pointer that recovers credentials after platform processing|
|6 *(optional)*|ZKP Privacy Tier|Proves provenance without revealing location, device identity, or photographer — for journalists and whistleblowers|

A piece of content registered with 0r1g1n can be verified by anyone, on any platform, regardless of what that platform has done to the file.

\---

## What 0r1g1n Provides

* ✅ Cryptographic evidence that content was captured by an attested device at a specific time
* ✅ Verification that content has not been altered since capture
* ✅ A tamper-evident provenance record independent of any platform
* ✅ Optional privacy-preserving verification for high-risk contexts
* ✅ Vendor-independent architecture — no single company controls the stack

## What 0r1g1n Does Not Provide

* ❌ Verification that content accurately depicts reality
* ❌ Detection of synthetic or AI-generated content
* ❌ Truth determination or fact-checking
* ❌ Absolute security against nation-state adversaries

Provenance is not truth. A camera pointed at a staged scene produces authentic records of a fabricated event. 0r1g1n tells you where content came from and that it has not been altered. What it depicts is a human judgement.

\---

## Status

**Current version:** `0.2.0-draft`
**Status:** Published for community review. Not for production use.

This is an early-stage open specification seeking community feedback, contributors, and pilot implementations. Interfaces and data formats are subject to change before a stable release.

\---

## Documents

|Document|Description|
|-|-|
|[Specification v0.2.0-draft](./specification/SPECIFICATION.md)|Full technical specification — interfaces, schemas, conformance requirements|
|[Whitepaper v1.1](./whitepaper/WHITEPAPER.md)|Architecture overview, design rationale, threat model|

\---

## Who Is This For

**News organisations and journalists** — verify the provenance of field footage without exposing sources.

**Insurance companies** — cryptographically authenticated claim photographs that cannot be deepfaked.

**Legal and forensic teams** — chain-of-custody records admissible as evidence.

**Platform developers** — integrate provenance verification into content distribution pipelines.

**Camera and device manufacturers** — implement conforming Layer 1 signing and join the verified ecosystem.

**Researchers and contributors** — extend the framework, implement conformance tests, build reference implementations.

\---

## Conformance Levels

|Level|Layers Required|Suitable For|
|-|-|-|
|Level 1 — Core|2, 3, 4|Platforms and verifiers receiving pre-signed content|
|Level 2 — Full|1, 2, 3, 4, 5|Content creation devices and publishing services|
|Level 3 — Privacy|1, 2, 3, 4, 5, 6|High-risk documentation — conflict journalism, whistleblowing|

\---

## Contributing

0r1g1n is an open specification. Contributions are welcome and encouraged.

**Ways to contribute:**

* **Specification feedback** — open an Issue with questions, corrections, or proposed amendments
* **Reference implementation** — build a conforming implementation of any layer
* **Conformance tests** — contribute test vectors to the conformance suite
* **Pilot deployment** — integrate 0r1g1n into a real workflow and document the experience
* **Translation** — help make the specification accessible in other languages

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting a pull request.

All contributors are expected to follow the [Code of Conduct](./CODE_OF_CONDUCT.md).

\---

## Governance

0r1g1n is governed as a community project. No single entity has unilateral authority over the specification. Changes to the specification require community review and a defined acceptance process described in [GOVERNANCE.md](./GOVERNANCE.md).

The framework is published under the **Apache 2.0 licence**. Implementations are free and unrestricted.

\---

## Relationship to Existing Standards

0r1g1n is not a replacement for C2PA. It is an extension — adding decentralised trust infrastructure and platform-independent registry to the C2PA signing layer that already exists in commercial hardware.

0r1g1n is designed to be compatible with:

* [C2PA Specification](https://c2pa.org/specifications/)
* [C2PA 2.1 Soft Binding](https://c2pa.org/specifications/specifications/2.1/)
* [ERC-7053](https://eips.ethereum.org/EIPS/eip-7053) on-chain media provenance
* [Chainlink Functions](https://docs.chain.link/chainlink-functions)

\---

## Contact and Community

* **Issues:** Use GitHub Issues for specification feedback and bug reports
* **Discussions:** Use GitHub Discussions for broader questions and ideas
* **Email:** 0r1g1nprotocol@proton.me

\---

*0r1g1n Framework Specification v0.2.0-draft | March 2026 | Apache 2.0*

