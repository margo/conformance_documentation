# Conformance Artifacts

## Conformance Card (human‑readable)
Vendor, persona, product, profile(s) passed, specification version (SemVer, for example `1.0.0-rc.3`), conformance status, known limits, and a stable reference to the signed conformance result.

## Conformance Result (machine‑readable)
A signed JSON document produced by the CTT containing test outcomes, requirement trace links (CR‑IDs), and evidence references. This document is the authoritative artifact validated and published by the Public Conformance Registry

### Conformance Result JSON(Draft)
The `spec.version` field uses the specification SemVer defined in [Specification Version Management](https://docs.margo.org/specification/specification-version-management). Release previews use `X.Y.Z-rc.N` tags; PR3 is `v1.0.0-rc.3` (also tagged `plugfest-2026-10`).
``` json
{
  "standard": "margo",
  "persona": "app_supplier | device_supplier | wfm_supplier",
  "profile": "PR3-applications | PR3-device-mgmt | ...",

  "spec": {
    "version": "1.0.0-rc.3",
    "tag": "v1.0.0-rc.3",
    "commit": "dc0bf84..."
  },

  "product": {
    "name": "Acme Vision Application",
    "vendor": "Acme Inc.",
    "version": "3.2.1"
  },

  "results": {
    "tests_passed": ["app-desc-validation", "compose-archive-validation"],
    "tests_skipped": ["otel-export"],
    "tests_failed": []
  },

  "traceability": {
    "method": "cr-id",
    "requirements_satisfied": [
      {
        "cr_id": "MARGO-APP-APPDESC-001",
        "test_id": "app-desc-validation",
        "spec_url": "https://docs.margo.org/specification/applications/application-description#top-level-attributes"
      },
      {
        "cr_id": "MARGO-APP-REG-017",
        "test_id": "compose-archive-validation",
        "spec_url": "https://docs.margo.org/specification/applications/application-registry#compose-component-registry"
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
    "tested_against": "1.0.0-rc.3",
    "expires_on_spec_change": true
  }
}
```