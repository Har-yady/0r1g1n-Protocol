# Governance

This document describes how the 0r1g1n Open Provenance Verification Protocol is governed — how decisions are made, who makes them, and how that changes over time.

The short version: 0r1g1n is governed as a community project. No single person, company, or organisation controls it. Changes to the specification require community review. The project is designed to be adopted by institutions that need to trust it, and those institutions will not trust infrastructure controlled by a single party.

---

## Principles

**Neutrality.** No single entity has unilateral authority over the specification, the trust list, or the protocol direction. This is not just a value — it is a requirement for institutional adoption. Camera manufacturers, news organisations, and platforms evaluating 0r1g1n need confidence that the standard they implement today will not be changed in ways that serve a single party's interests tomorrow.

**Transparency.** All decisions about the specification are made in public, through documented processes, with a record of the reasoning.

**Meritocracy.** Influence is earned through sustained, high-quality contribution — not through affiliation, funding, or seniority.

**Conservative evolution.** The specification changes slowly and deliberately. Breaking changes to interfaces or data formats require strong justification and extended community review. Stability is a feature.

---

## Project Phases

The governance structure evolves as the project matures. This document describes the current phase and the intended evolution.

### Phase 1 — Bootstrap (Current)

During the initial phase, the project is maintained by its founding contributors. Decision-making is informal and consensus-based. The priority is to attract contributors, validate the specification through pilot deployments, and build the community needed for formal governance.

**During Phase 1:**
- The founding maintainers make day-to-day decisions by consensus
- Significant specification changes require public discussion via GitHub Issues before merging
- Any contributor may object to a proposed change and request extended review
- No changes to normative requirements (MUST, MUST NOT) are made without at least 14 days of public comment

### Phase 2 — Steering Committee

When the project reaches meaningful adoption — defined as at least three independent conforming implementations across different organisations — a Steering Committee will be established.

The Steering Committee will:
- Comprise representatives from a diversity of stakeholder groups (news organisations, hardware manufacturers, platform developers, security researchers, civil society)
- Include no more than one member from any single organisation
- Make decisions about specification versions, breaking changes, and Trust List governance by supermajority (two-thirds)
- Meet at least quarterly, with records published publicly

Steering Committee members serve two-year terms, staggered to ensure continuity. The process for electing and removing members will be defined before Phase 2 begins.

### Phase 3 — Foundation

When the project is sufficiently mature, it will seek a permanent home with a neutral non-profit foundation — modelled on the Linux Foundation, Apache Software Foundation, or similar. This provides long-term legal and financial stability without any single party's ownership.

---

## Decision Types

Not all decisions require the same process. The following categories describe the review required for different types of changes.

### Editorial (no review required)
Typo corrections, formatting fixes, broken link repairs, and purely cosmetic changes to documentation. These may be merged by any maintainer after a single review.

### Clarification (7-day review)
Changes that clarify existing requirements without altering their meaning. Examples: adding an example to illustrate an existing rule, improving ambiguous wording where the intent is clear.

### Non-breaking addition (14-day review)
Adding new optional features, new appendix content, new error codes, or new informative sections that do not change existing normative requirements. Must be opened as an Issue before a pull request.

### Breaking change (30-day review + supermajority)
Any change to a normative requirement (MUST, MUST NOT, REQUIRED), any change to a data schema field that is not purely additive, any change to an interface definition. Requires:
- A published RFC-style Issue describing the problem, proposed solution, alternatives considered, and backwards compatibility impact
- 30 days of open comment
- Supermajority approval from active maintainers (or Steering Committee in Phase 2+)
- A migration path for existing implementations

### Version increment
A new specification version is published when: a sufficient number of changes have accumulated, a breaking change has been approved and implemented, or security considerations require an urgent update. Version numbers follow semantic versioning: MAJOR.MINOR.PATCH.

---

## Roles

### Contributor
Anyone who opens an Issue, submits a pull request, participates in discussion, or otherwise engages with the project. No formal status required.

### Maintainer
Contributors who have demonstrated sustained, high-quality contribution and have been granted write access to the repository by existing maintainers. Maintainers review and merge pull requests, manage Issues, and make day-to-day decisions.

Maintainers are added by consensus of existing maintainers. There is no fixed number. A maintainer who has been inactive for six months may be moved to Emeritus status.

### Founding Maintainer
The initial maintainers who established the project. During Phase 1, founding maintainers collectively hold the responsibilities of the Steering Committee. Founding maintainer status does not confer permanent authority — it transitions to the Steering Committee structure in Phase 2.

### Trust List Maintainer
A distinct role responsible for maintaining the list of trusted C2PA Certificate Authorities and device manufacturers. Trust List decisions require multisig approval from at least three of five keyholders drawn from different organisations. No single organisation may hold more than one Trust List Maintainer key.

Trust List Maintainers are appointed by the Steering Committee (or founding maintainers in Phase 1). They are accountable to the Steering Committee and may be removed by supermajority vote.

---

## Forking

0r1g1n is published under Apache 2.0. Anyone may fork the repository and create a derivative specification. However, only implementations that conform to the canonical specification maintained in this repository may use the 0r1g1n name and trademark.

Forks are encouraged as a mechanism for experimentation. If a fork produces improvements that the community supports, those changes are welcome back through the normal pull request process.

---

## Conflict Resolution

Most disagreements are resolved through discussion. The following process applies when discussion does not converge.

1. Any party may request a formal resolution by opening an Issue tagged `resolution-requested`
2. A 14-day comment period follows during which all perspectives are documented
3. The maintainers (or Steering Committee in Phase 2) deliberate and publish a written decision with reasoning
4. The decision is final unless new material information emerges

Maintainers and Steering Committee members with a conflict of interest in a decision must recuse themselves.

---

## Amendments to This Document

This governance document may be amended through the breaking change process described above. Any change to governance requires 30 days of public comment and supermajority approval.

---

## Questions

If you have questions about governance that this document does not answer, open a GitHub Discussion tagged `governance`. Governance questions are always welcome and the answers will improve this document.

---

*0r1g1n Open Provenance Verification Protocol | Apache 2.0*
