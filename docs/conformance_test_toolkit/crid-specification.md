# Margo Conformance Requirement Identifier (CR-ID) Specification

## Purpose

This document defines the **Conformance Requirement Identifier (CR-ID)** format used by the Margo specification and explains **how CR-IDs are derived** from persona-based normative requirements. CR-IDs provide a stable, implementation-independent mechanism to trace requirements across:

- The normative specification text
- Conformance Test Toolkit (CTT) test cases
- Spec Traceability Matrices (STM)
- Conformance claims

CR-IDs are **authoritative identifiers**. They are assigned once and MUST NOT change, even if requirement wording, strength, or test implementation evolves.

---

## CR-ID Format

All CR-IDs follow this canonical structure:

```
MARGO-<PERSONA>-<PORTION>-<NNN>
```

### Field Definitions

| Segment | Description |
|------|-------------|
| `MARGO` | Fixed prefix identifying the Margo standard |
| `<PERSONA>` | Responsibility boundary to which the requirement applies |
| `<PORTION>` | Functional portion of the specification |
| `<NNN>` | Zero-padded sequential number (per persona + portion) |

---

## Persona Codes

Persona codes are derived directly from the persona segmentation used in the specification and spreadsheet.

| Persona | Code |
|------|------|
| Application Supplier | `APP` |
| Device Supplier | `DEV` |
| Workload Fleet Manager | `WFM` |

Each CR-ID is scoped to **exactly one persona**.

---

## Portion Codes

Portion codes represent the functional concern area of the specification to which the requirement belongs. Portions are derived deterministically from the specification section (in practice, inferred from the Source URL).

| Portion | Code |
|------|------|
| Application Description | `APPDESC` |
| Application Registry | `REG` |
| Observability | `OBS` |
| Management Interface | `MGMT` |
| Security | `SEC` |
| Deployment / Device Capabilities | `DEPLOY` |
| General / Cross-cutting | `GEN` |

Portion codes allow conformance to be claimed **selectively and transparently**.

---

## Example CR-IDs

```
MARGO-APP-APPDESC-007
MARGO-DEV-OBS-014
MARGO-WFM-MGMT-023
```

These examples indicate:

- The responsible persona
- The specification portion
- The stable ordinal within that scope

## Stability Rules

CR-IDs are intentionally decoupled from requirement wording and strength.

A CR-ID:
- ✅ **Does NOT encode** MUST / SHOULD / MAY
- ✅ **Does NOT change** if text is clarified
- ✅ **Does NOT change** if tests are refactored

A CR-ID **MUST ONLY change** if a requirement is explicitly removed from the specification.

---

## Relationship to Conformance Testing

CR-IDs act as the **join key** between specification and conformance artifacts:

```
Specification Requirement
  → CR-ID
     → One or more CTT Test Cases
        → Evidence
           → Conformance Claim
```

This enables:
- Partial conformance by persona
- Profile-based conformance (PR1, GA1, etc.)
- Transparent reporting during plugfests

---

## Summary

CR-IDs provide a stable, persona-scoped, and portion-aware mechanism to govern conformance across the Margo ecosystem. Their deterministic derivation ensures that conformance remains objective, automatable, and evolvable without re-litigating identifiers.
