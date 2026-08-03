# Conformance Artifacts

## Conformance Card (human‑readable)
Vendor, persona, product, profile(s) passed, specification version, conformance status, known limits, and a stable reference to the signed conformance result.

## Conformance Result (machine‑readable)
A signed JSON document produced by the CTT containing test outcomes, requirement trace links (CR‑IDs), and evidence references. This document is the authoritative artifact validated and published by the Public Conformance Registry

### Conformance Result JSON(Draft)
``` json
{
  "standard": "margo",
  "persona": "workload_supplier | device_supplier | wfm_supplier",
  "profile": "GA1-applications | GA1-device-mgmt | ...",

  "spec": {
    "version": "GA1",
    "commit": "bc073106..."
  },

  "product": {
    "name": "Acme Vision Application",
    "vendor": "Acme Inc.",
    "version": "3.2.1"
  },

  "results": {
    "tests_passed": ["APP-DESC-001", "PKG-HELM-005", "DEPLOY-013"],
    "tests_skipped": ["OBS-OTEL-002"],
    "tests_failed": []
  },

  "traceability": {
    "method": "cr-id",
    "requirements_satisfied": [
      {
        "cr_id": "APP-DESC-001",
        "test_id": "app-desc-validation",
        "spec_section": "4.2.1"
      },
      {
        "cr_id": "PKG-HELM-005",
        "test_id": "helm-lint",
        "spec_section": "5.1.3"
      }
    ]
  },

  "evidence_references": [
    {
      "type": "log",
      "uri": "oci://registry.margo.org/acme/vision/evidence/logs@sha256:...",
      "hash": "sha256:..."
    },
    {
      "type": "manifest",
      "uri": "oci://registry.margo.org/acme/vision/manifests@sha256:...",
      "hash": "sha256:..."
    }
  ],

  "signature": {
    "algorithm": "sha256",
    "signed_by": "vendor",
    "timestamp": "2026-01-07T17:45:00Z"
  },

  "status": "valid | revoked | suspended",

  "validity": {
    "from": "2026-01-07",
    "tested_against": "GA1",
    "expires_on_spec_change": true
  }
}
```