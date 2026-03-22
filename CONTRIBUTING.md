# Contributing to 0r1g1n

Thank you for your interest in contributing to the 0r1g1n Open Provenance Verification Protocol. This document explains how the project works and how you can get involved.

0r1g1n is a community-governed open specification. It improves through the contributions of people who understand the problem — cryptographers, security engineers, journalists, media technologists, blockchain developers, and domain experts from the industries the framework serves. All of these perspectives are valuable and welcome.

---

## Table of Contents

- [Ways to Contribute](#ways-to-contribute)
- [Before You Start](#before-you-start)
- [Specification Feedback](#specification-feedback)
- [Code Contributions](#code-contributions)
- [Conformance Test Vectors](#conformance-test-vectors)
- [Documentation](#documentation)
- [Pilot Deployments](#pilot-deployments)
- [Pull Request Process](#pull-request-process)
- [Style Guide](#style-guide)
- [Licensing](#licensing)

---

## Ways to Contribute

### If you are a security researcher or cryptographer
- Review the threat model (Section 3 of the specification) and open an Issue if you identify gaps or incorrect assumptions
- Audit the ZKP circuit specification (Section 10) and the oracle consensus algorithm (Section 7.3)
- Contribute adversarial test vectors to the conformance suite

### If you are a blockchain or Web3 developer
- Implement the on-chain verifier contract (Layer 4) and submit it as a reference implementation
- Build a conforming Chainlink oracle node and C2PA validator bridge
- Contribute to the Space and Time VDR integration

### If you are a media technologist or journalist
- Trial the framework in a real workflow and document the experience as a pilot report
- Identify usability gaps that the specification does not address
- Help translate the specification or documentation into other languages

### If you are a C2PA or standards expert
- Review the Layer 1 specification for compliance with C2PA requirements
- Help identify integration points with the CAI ecosystem
- Contribute to the conformance test suite for hardware signing

### If you have general software development skills
- Build the reference implementation components listed in the open Issues
- Improve documentation clarity
- Convert grid tables in the specification to clean GFM pipe tables
- Write integration examples and getting-started guides

---

## Before You Start

1. **Read the specification.** The [SPECIFICATION.md](./specification/SPECIFICATION.md) is the authoritative document. Familiarise yourself with the layer you intend to work on before opening a pull request.

2. **Check existing Issues.** Your idea or question may already be under discussion. Search open and closed Issues before creating a new one.

3. **Open an Issue before significant work.** For anything beyond a small correction, open an Issue first to discuss the approach. This avoids situations where substantial work is submitted that conflicts with the project's direction.

4. **Sign the CLA.** Before your first pull request is merged, you will be asked to sign the Contributor Licence Agreement. This grants the project the right to use your contribution under the Apache 2.0 licence and includes a patent licence. The CLA is lightweight — it does not transfer copyright.

---

## Specification Feedback

The specification is a living document in draft status. Feedback is actively sought.

**For errors, ambiguities, or omissions:**
Open an Issue with the label `specification`. Include:
- The section number and heading
- What the current text says
- What you believe it should say, and why
- Any relevant references (RFCs, academic papers, existing implementations)

**For proposed new sections or significant structural changes:**
Open an Issue with the label `specification-rfc` and describe the problem the change addresses before proposing the solution. Significant changes require community discussion before a pull request.

**For editorial corrections** (typos, formatting, broken links):
A pull request without a prior Issue is acceptable for purely editorial changes.

---

## Code Contributions

The project maintains a `/reference` directory for reference implementations of each layer. These are illustrative, not production-ready. They demonstrate that the specification is implementable and provide a starting point for adopters.

**Reference implementation contributions should:**
- Implement a single, well-defined layer or interface
- Include a README explaining what is implemented and what is not
- Include tests demonstrating the happy path and at least one failure case
- Be written in a widely-used language (TypeScript, Rust, Python, or Solidity)
- Not introduce dependencies that would prevent the code from running without significant setup

**Reference implementations are not:**
- Production systems — they do not need to be optimised, hardened, or complete
- Normative — the specification is the authority, not the code

If your implementation diverges from the specification, the specification is correct. Open an Issue to discuss whether the specification needs amendment.

---

## Conformance Test Vectors

The conformance test suite (described in Section 12.2 of the specification) is critical for interoperability. Test vector contributions are among the most valuable things you can contribute.

**Test vectors should be submitted as:**
- A directory under `/conformance-tests/{category}/`
- A JSON descriptor file describing the vector, expected result, and rationale
- The test asset itself (image, manifest, or mock data as appropriate)
- A note on which layer and section of the specification the test covers

**Categories:**
- `positive/` — known-good content that MUST return VERIFIED
- `negative/` — content with specific defects that MUST return defined error codes
- `adversarial/` — content designed to defeat naive implementations
- `revocation/` — content involving device certificate revocation scenarios
- `failure-mode/` — simulated infrastructure failure scenarios
- `zkp/` — ZKP proof validation vectors (Level 3 conformance only)

---

## Documentation

Documentation contributions are welcome without prior Issue discussion. This includes:

- Improving the clarity of existing specification language
- Adding examples to illustrate abstract requirements
- Writing integration guides for specific platforms or frameworks
- Correcting factual errors in the whitepaper
- Translating documents into other languages

If you are translating the specification, please open an Issue first so we can coordinate and avoid duplicate effort.

---

## Pilot Deployments

If you have deployed or are planning to deploy 0r1g1n in a real context, please share your experience. Pilot deployment reports help the community understand where the specification works well and where it needs improvement.

Pilot reports can be submitted as:
- A pull request adding a file to `/pilots/` describing your deployment
- A GitHub Discussion post
- An Issue tagged `pilot-report`

A useful pilot report includes: the use case, which layers were implemented, what worked, what didn't, and what changes to the specification would have helped.

---

## Pull Request Process

1. Fork the repository and create a branch from `main`.
2. Make your changes. Ensure any specification changes are reflected consistently across the document (headings, section references, appendices, changelog).
3. Update `SPECIFICATION.md` or `WHITEPAPER.md` as appropriate. Update the changelog in Appendix C with a brief description of the change.
4. Open a pull request against `main` with a clear description of what the PR does and why.
5. A maintainer will review the PR. For specification changes, community review may be requested before merging.
6. Once approved, a maintainer will merge the PR.

**Commit message format:**

```
type(scope): short description

Longer description if needed.

Refs: #issue-number
```

Types: `spec` (specification change), `ref` (reference implementation), `test` (conformance tests), `docs` (documentation), `fix` (correction).

---

## Style Guide

**Specification language:**
- Use RFC 2119 keywords (MUST, SHOULD, MAY, etc.) precisely and only where normative requirements are intended
- Prefer "provides cryptographic evidence that" over "ensures" or "guarantees"
- Prefer "mitigates" over "prevents"
- State what the system does not do as clearly as what it does

**Markdown:**
- Use ATX headings (`#`, `##`, `###`) — not underline-style headings
- Use GFM pipe tables, not grid tables
- Use fenced code blocks with language identifiers (` ```json `, ` ```solidity `, etc.)
- Wrap prose at 100 characters where practical

**Code:**
- Follow existing style in the file you are editing
- Include comments explaining non-obvious decisions
- Reference the relevant specification section in comments where applicable

---

## Licensing

By contributing to 0r1g1n, you agree that your contributions will be licensed under the [Apache 2.0 Licence](./LICENSE). You also agree that the contribution does not infringe any third-party intellectual property rights.

The Contributor Licence Agreement will be presented the first time you open a pull request.

---

*0r1g1n Open Provenance Verification Protocol | Apache 2.0*
