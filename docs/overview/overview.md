**Change log**

- Jan 8th, 2026: Initial content creation for collaboration
- Jan 21st, 2026: 
    - Added additional requirements found in the specification.
    - Added tasks per persona to track during the planning rounds.
- Feb 16th, 2026: Updated summary and added more general information.
- Oct 13th, 2026: Aligned conformance content with the PR3 specification snapshot (v1.0.0-rc.3).
    - Generated persona requirement tables from the specification.
    - Added Identity and Authorization, Extensibility, Problem Types, and Version Management content.
    - Replaced device roles with device capabilities in the persona outlines.
    - Surfaced 167 candidate statements (lowercase, OpenAPI-only, informative, open items, non-product) for review.

# Specification Baseline
The current conformance content tracks the Margo specification **v1.0.0-rc.3 (PR3)**, tag `plugfest-2026-10`, published at [docs.margo.org](https://docs.margo.org/). See [Specification Version Management](https://docs.margo.org/specification/specification-version-management) for the versioning scheme.

| Persona | Requirements |
|---|---|
| Application Supplier | [app_supplier_conf_requirements.md](../persona_based_content/app_supplier_conf_requirements.md) |
| Device Supplier | [device_supplier_conf_requirements.md](../persona_based_content/device_supplier_conf_requirements.md) |
| Workload Fleet Manager Supplier | [WFM_supplier_conf_requirements.md](../persona_based_content/WFM_supplier_conf_requirements.md) |
| Non-persona (operator, deployment, process) | [non_persona_requirement_candidates.md](../persona_based_content/non_persona_requirement_candidates.md) |

Each persona table ends with a **Candidate Requirements (for review)** section listing lowercase, OpenAPI-only, informative, and open-item statements that are not yet CR-IDs.

# Margo Conformance Model
The conformance model that drives Margo is more of a progression from deliverables via various groups in Margo. 

1. User Stories/Vision slides
2. Specification Content
3. Conformance Tests
    - Conformance tests should directly reflect the user stories/items described in the Vision Slides

## Focus of Testing

- Testing is not about ease of use or usability of a publicly available product
- Testing is strictly about **conformance to the normative specification**

## When products fail
Failures typically trace back to one of two causes:

- The specification is insufficient(not granular enough or missing critical guidance)
- **The implementation is incorrect**

> Our focus should be on ensuring the normative specification is granular enough to remove ambiguity via the results. 

## Conformance testing requirements
MUST BE:

- Automatable or largely automatable
- interface-driven

AVOID:

- Manual usability assessments
- Subjective UX evaluation

# Conformance infrastructure for an open‑source, industrial interoperability standard (Margo)
**What “conformance” means in Margo right now**

Persona‑based profiles: The TWG’s direction is to define conformance buckets by persona and value, with clear MUST vs OPTIONAL requirements that are verified via the Conformance Test Toolkit (CTT).

## Overarching Tasks

- [ ] Establish requirement numbering scheme and documentation standard.
- [ ] Implement requirement numbering scheme standard within specification, ensuring consistency and importance in each persona based section of specification.
- [ ] Build traceability matrix: requirements --> Test cases --> Result tracking
> Note: Traceability: Every MUST/SHALL item should map to a test case in the conformance suite.

- [ ] Scope out work required to build conformance test suite and required infrastructure
    - [ ] Work with Steering Committee to ensure work is funded
    - [ ] Build schedule to achieve Test and Conformance strategy by GA release.



# Governance (including revocation) (DRAFT)
Key policies

- Approval: CTT‑verified supplier product at GA1.
- Revocation triggers:
    - Proven non‑conformance after updates (spec or product).
    - Misrepresentation of capabilities.
    - Security vulnerability disclosed where mitigation breaks required behavior.
    - License/compliance violations (e.g., redistribution constraints).
- Process:
    - Open Case: create a public issue linked to the registry entry (traceability).
    - Notification & Window: vendor has a defined remediation window.
    - Decision: Suspend (temporarily hide badges) or Revoke (mark status and keep evidence available).
    - Appeal: vendor may appeal with new evidence; decision logged.
- Transparency: every decision logged with rationale; changelog syndicated into docs.
- Anti‑“Level” framing: keep persona profiles; avoid value judgements beyond MUST/OPTIONAL compliance, consistent with TWG direction.