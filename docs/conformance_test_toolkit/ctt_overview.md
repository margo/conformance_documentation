# Conformance Suite Components

- **Conformance Test Toolkit (CTT)**: A modular test toolkit that runs within a supplier’s local or cloud environment. The CTT executes persona‑scoped test modules, evaluates results, resolves requirement traceability using the Spec Traceability Matrix, and packages outcomes, trace links, and evidence references into a single signed conformance result for submission to the Public Conformance Registry.
- **Spec Traceability Matrix**: A maintained mapping of specification requirements (e.g., CR‑IDs) to conformance test cases and expected assertions. The STM is used by the CTT to deterministically resolve which requirements are satisfied by executed tests and to embed verifiable trace links into the conformance result. The STM itself does not publish artifacts or results.
- **Public Conformance Registry**: The authoritative publication point for conformance claims. The PCR validates submitted signed conformance results (schema, signature, and traceability) and publishes human‑readable conformance records alongside machine‑readable JSON for discovery and consumption by participant systems.

## Conformance Mapping

``` mermaid

flowchart TD
  STM[Spec Traceability Matrix - STM]
  CTT[Conformance Test Toolkit - CTT]
  PCR[Public Conformance Registry - PCR]

  STM -->|Provides requirement mappings| CTT
  CTT -->|Runs tests using mappings| STM
  CTT -->|Submits signed result including trace links and evidence references| PCR

```
## Publishing flow

- Supplier runs CTT → produces a single signed conformance result that includes test outcomes, requirement trace links, and evidence references.
- Submit to registry.margo.org → validation of schema conformance, signature integrity, and requirement traceability.
- Publish → entry is assigned a unique, immutable ID; a human‑readable Conformance Card is rendered in documentation, and a machine‑readable JSON record is exposed via API for programmatic discovery (e.g., by WFM tooling).
- Update → vendor re‑submits a new signed conformance result when product versions, profiles, or specification versions change; the registry maintains lineage and historical records.


## Conformance Requirement Identification and schema
For details on how requirements are tracked individually, navigate to the following [document](../conformance_test_toolkit/crid-specification.md)

## Conformance Test Toolkit User Guide
...coming soon.



