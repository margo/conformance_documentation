# Margo Device Supplier — Conformance Suite: Complete Technical and Demo Guide

---

## Table of Contents

1. [What Is This?](#1-what-is-this)
2. [The Big Picture — How Everything Fits Together](#2-the-big-picture--how-everything-fits-together)
3. [File and Folder Structure Explained](#3-file-and-folder-structure-explained)
4. [Module Deep-Dive: The Three Pillars](#4-module-deep-dive-the-three-pillars)
   - [Pillar 1: The Mock Server (server.go)](#pillar-1-the-mock-server-servergo)
   - [Pillar 2: The Test Runner (run_tests.go)](#pillar-2-the-test-runner-run_testsgo)
   - [Pillar 3: The JSON Data Files](#pillar-3-the-json-data-files)
5. [All API Endpoints Explained](#5-all-api-endpoints-explained)
6. [How the Data Flows End-to-End](#6-how-the-data-flows-end-to-end)
7. [RFC 9421 HTTP Message Signatures — Explained Simply](#7-rfc-9421-http-message-signatures--explained-simply)
8. [Server-Side Validation Engine — How Assertions Work](#8-server-side-validation-engine--how-assertions-work)
9. [How to Add a New Test (Step by Step, No Code)](#9-how-to-add-a-new-test-step-by-step-no-code)
10. [How to Add a New Assertion Rule](#10-how-to-add-a-new-assertion-rule)
11. [Running the Suite — All Commands Explained](#11-running-the-suite--all-commands-explained)
12. [State Management and Data Lifecycle](#12-state-management-and-data-lifecycle)
13. [TLS and Certificate Architecture](#13-tls-and-certificate-architecture)
14. [Go Dependencies Explained](#14-go-dependencies-explained)
15. [What the Conformance Test Report Shows](#15-what-the-conformance-test-report-shows)
16. [Connecting to a Real Device Agent](#16-connecting-to-a-real-device-agent)
17. [Common Questions and Answers](#17-common-questions-and-answers)

---

## 1. What Is This?

The **Margo Device Supplier Conformance Suite** is a self-contained testing framework that verifies whether a Workload Fleet Manager (WFM) server correctly implements the [Margo Management Interface API v1.0.0](https://raw.githubusercontent.com/margo/specification/pre-draft/system-design/specification/margo-management-interface/workload-management-api-1.0.0.yaml).

In plain English:
- **Margo** is an open standard for managing workloads (software applications) on edge devices (e.g. industrial computers, IoT gateways).
- A **WFM server** is the cloud/server side that tells devices what to run.
- A **device agent** runs on the device and talks to the WFM server.
- This suite contains a **mock WFM server** and a **test runner** that acts like a device agent, running 36 automated tests to confirm everything works according to the spec.

**Why does this matter?**
Any vendor who builds a WFM server or a device agent can use this suite to confirm their implementation is correct before deploying to real devices. It removes the need to run real hardware for basic conformance checking.

**Current status: 36/36 tests PASSING (100% conformance)**

---

## 2. The Big Picture — How Everything Fits Together

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONFORMANCE SUITE                            │
│                                                                 │
│  ┌──────────────────────────┐   ┌───────────────────────────┐  │
│  │   Test Runner            │   │   Mock WFM Server         │  │
│  │   (run_tests.go)         │◄──►   (server.go)             │  │
│  │                          │   │                           │  │
│  │  Reads:                  │   │  Reads:                   │  │
│  │  test-scenarios.json     │   │  assertions.json          │  │
│  │  (WHAT to test)          │   │  (HOW to validate)        │  │
│  │                          │   │                           │  │
│  │  Signs requests with     │   │  Verifies signatures      │  │
│  │  device private key      │   │  Validates request bodies │  │
│  │                          │   │  Returns spec responses   │  │
│  └──────────────────────────┘   └───────────────────────────┘  │
│           │                               │                     │
│           ▼                               ▼                     │
│    HTML Report                    data/ (JSON files)            │
│    (reports/)                     (temporary state)             │
└─────────────────────────────────────────────────────────────────┘
```

### One-Sentence Summary of Each Component

| Component | File | What It Does |
|-----------|------|--------------|
| **Mock Server** | `server.go` | Pretends to be a real WFM server, enforcing all spec rules |
| **Test Runner** | `run_tests.go` | Pretends to be a device agent, sending every kind of request |
| **Test Scenarios** | `device-scenarios/test-scenarios.json` | Defines what HTTP calls to make and what responses to expect |
| **Assertions** | `manifests/assertions.json` | Defines every validation rule the server enforces on incoming requests |
| **Reports** | `reports/*.html` | The test results in a human-readable format |
| **Certificates** | `certs/` | TLS and signing certificates used for secure communication |
| **Data State** | `data/*.json` | Temporary per-run storage of clients and deployments |

---

## 3. File and Folder Structure Explained

```
device_supplier/
│
├── server.go                          ← Mock WFM server (the thing being tested)
├── run_tests.go                       ← Conformance test runner (the tester)
├── go.mod / go.sum                    ← Go dependency declarations
├── Makefile                           ← Convenience commands (build, run, demo)
│
├── manifests/
│   └── assertions.json                ← ALL validation rules (server reads this on startup)
│
├── device-scenarios/
│   └── test-scenarios.json            ← ALL test cases (runner reads this on startup)
│
├── certs/
│   ├── ca-cert.pem                    ← Root CA certificate (who we trust)
│   ├── ca-key.pem                     ← Root CA private key (to sign server cert)
│   ├── server-cert.pem                ← Server TLS certificate
│   └── server-key.pem                 ← Server TLS private key
│
├── data/                              ← Runtime state (auto-deleted on server startup)
│   ├── clients.json                   ← Registered devices from current run
│   └── deployments.json               ← Deployment state from current run
│
├── bin/
│   ├── server                         ← Compiled server binary
│   └── run_tests                      ← Compiled test runner binary
│
└── reports/
    └── conformance-report-<timestamp>.html ← Test results (one per run)
```

---

## 4. Module Deep-Dive: The Three Pillars

### Pillar 1: The Mock Server (`server.go`)

**Written in:** Go  
**Runs on:** `https://localhost:3001`  
**URL prefix:** `/v1alpha2/margo`

The server is a complete implementation of the Margo Management Interface API. When it starts:

1. **Clears old data** — deletes `data/clients.json` and `data/deployments.json` for a clean run (unless `KEEP_DATA=true` is set)
2. **Loads assertions** — reads `manifests/assertions.json` into memory; this is the validation rulebook
3. **Sets up TLS** — loads or generates HTTPS certificates from `certs/`
4. **Starts listening** — opens HTTPS on port 3001

**Key internal components:**

| Function | What It Does |
|----------|-------------|
| `loadAssertions()` | Reads `assertions.json` into memory at startup |
| `validateRequest()` | Applies loaded rules against an incoming request body |
| `verifyRFC9421Signature()` | Checks the HTTP request signature (proves the sender is who they say) |
| `validateContentDigest()` | Verifies the body hash header matches the actual body |
| `buildDeploymentYAML()` | Creates a Margo ApplicationDeployment YAML on the fly |
| `buildBundleArchive()` | Creates a `.tar.gz` containing all deployment YAMLs |
| `buildStateManifest()` | Creates the JSON manifest listing all deployments with content-addressed URLs |
| `saveClientsToFile()` | Persists registered clients to `data/clients.json` |
| `ensureDefaultDeployment()` | When a new device registers, automatically assigns a default workload |

**Important design choice — why does the server read JSON for validation instead of having rules in code?**

If validation rules were hardcoded in Go, every time someone added a new field or changed an enum value, a developer would have to edit Go code, recompile, and redeploy. With `assertions.json`, a non-developer can change validation rules, drop the new file, and restart the server — no code changes needed. This is called a **data-driven architecture**.

---

### Pillar 2: The Test Runner (`run_tests.go`)

**Written in:** Go  
**Acts as:** A device agent (the client side)

On startup, the runner:

1. **Loads the device certificate and private key** from `/home/margo/sandbox/poc/device/agent/config/`
2. **Loads test scenarios** from `device-scenarios/test-scenarios.json`
3. **Waits for the server** to be ready (polls `/health` endpoint)
4. **Runs each scenario** in sequence, step by step

**For every test step, the runner:**

1. Builds the HTTP request body from the scenario JSON
2. **Injects the real device certificate** into the request body where needed (for onboarding steps)
3. **Signs the request** using the `htmsig` library (RFC 9421 signatures)
4. Sends the request to the server
5. Checks the HTTP status code matches the expected value
6. Runs field-level validations on the response body
7. Saves named values from the response (e.g., `clientId`) for use in later steps

**Context sharing between steps:**

Some tests are multi-step. For example, step 3.0 onboards a device and saves the `clientId`. Step 3.1 then uses that `clientId` in the URL. This is done via `extract_context` and `{placeholders}`:

```json
"extract_context": { "clientId": "clientId" }   ← save from response
"endpoint": "/api/v1/clients/{clientId}/..."     ← use in next step
```

The runner maintains a `TestContext` struct with a `Data` map that carries these values across steps.

---

### Pillar 3: The JSON Data Files

There are two JSON files that define the entire behaviour of the suite without touching Go code:

#### `manifests/assertions.json` — The Server's Rulebook

This file is loaded by the **server** at startup. It defines:

- **What fields are required** in each request body
- **What type** each field must be (string, number, object, array)
- **What values are allowed** (enums like `"installed"`, `"failed"`)
- **What error format** to use when validation fails (400 with `error` string, or 422 with `errors` array)
- **Which certificate values are blocklisted** (for testing rejection scenarios)

Structure overview:

```json
{
  "rejected_certificates": ["MIID-rejected-device-cert"],
  "endpoints": {
    "POST_onboarding": {
      "path": "/api/v1/onboarding",
      "method": "POST",
      "status_code": 201,
      "validation_error_key": "badRequest",
      "validations": [
        { "rule_id": "onboarding-001", "field": "apiVersion", "type": "string",
          "required": true, "value": "onboarding.margo.org/v1alpha1" }
      ]
    }
  },
  "error_responses": {
    "badRequest":    { "status_code": 400, "format": "error_string" },
    "unprocessable": { "status_code": 422, "format": "validation_errors" }
  }
}
```

#### `device-scenarios/test-scenarios.json` — The Test Runner's Script

This file is loaded by the **test runner** at startup. It defines:

- **Which scenarios** exist (e.g. onboarding, capabilities, deployments, error handling)
- **Each step** within a scenario: what HTTP method, what URL, what body, what headers
- **What the expected HTTP status code** is
- **What fields to validate** in the response body (field name + operation like `equals`, `not_empty`, `is_string`)
- **Special flags** like `skip_signing: true` (to test what happens when no signature is provided)
- **What to extract** from the response for later steps

---

## 5. All API Endpoints Explained

The server implements 8 endpoints according to the Margo Management Interface API v1.0.0. All are HTTPS under the base path `/v1alpha2/margo`.

### 1. `GET /api/v1/onboarding/certificate`
**Purpose:** A new device downloads the WFM's root CA certificate to trust for future secure connections.  
**Auth:** None — the device doesn't have an identity yet.  
**Returns:** `{ "certificate": "<PEM string>" }`

---

### 2. `POST /api/v1/onboarding`
**Purpose:** A device registers itself with the WFM server by sending its X.509 public certificate.  
**Auth:** No signature required — this is the bootstrapping step. The device doesn't have a registered key yet.  
**Body must include:** `apiVersion`, `kind: "OnboardingRequest"`, `certificate`  
**On success:** Returns `{ "clientId": "<uuid>" }` — the device's permanent identifier  
**On failure:** 400 (missing/wrong fields), 403 (blocklisted certificate)

---

### 3. `POST /api/v1/clients/{clientId}/capabilities`
**Purpose:** The device reports its hardware capabilities (CPU, memory, storage, interfaces).  
**Auth:** RFC 9421 signature + Content-Digest header required  
**Body must include:** `apiVersion`, `kind: "DeviceCapabilitiesManifest"`, `properties` with id, vendor, modelNumber, serialNumber, roles, resources  
**Returns:** `{ "status": "capabilities_received" }`

---

### 4. `PUT /api/v1/clients/{clientId}/capabilities`
**Purpose:** Same as POST capabilities but used for updating existing capabilities.  
**Auth:** Same as POST capabilities  
**Handled by:** Same server code as POST (identical validation)

---

### 5. `GET /api/v1/clients/{clientId}/deployments`
**Purpose:** The device polls to see what workloads it should be running.  
**Auth:** Client must be registered. No signature required for GET.  
**Headers:** `Accept: application/vnd.margo.manifest.v1+json` — if an unsupported type is sent, the server returns 406.  
**Caching:** Supports `If-None-Match` / `ETag` — if nothing has changed, returns HTTP 304 (Not Modified) with no body, saving bandwidth.  
**Returns:** A JSON manifest with:
- `manifestVersion` — monotonically increasing number (prevents rollback attacks)
- `bundle` — reference to the `.tar.gz` bundle containing all deployment YAMLs
- `deployments` — list of individual deployment references, each with a content-addressed URL

---

### 6. `GET /api/v1/clients/{clientId}/bundles/{digest}`
**Purpose:** Downloads the full `tar.gz` bundle containing all deployment YAML files.  
**Auth:** Client must be registered. No signature required for GET.  
**Content-addressing:** The `{digest}` in the URL is `sha256:<hex>`. The server only returns the file if the digest matches the actual content, guaranteeing the device gets exactly what the server intended.  
**Caching:** `Cache-Control: public, max-age=31536000, immutable` — once downloaded, this URL never changes.

---

### 7. `GET /api/v1/clients/{clientId}/deployments/{deploymentId}/{digest}`
**Purpose:** Downloads the YAML file for a single specific deployment.  
**Auth:** Client must be registered. No signature required for GET.  
**Content-addressing:** Same digest guarantee as bundles — the server returns 404 if the digest doesn't match.  
**Response headers:** `Content-Type: application/yaml`, `Vary: Accept-Encoding` (compression negotiation), `Cache-Control: immutable`

---

### 8. `POST /api/v1/clients/{clientId}/deployments/{deploymentId}/status`
**Purpose:** The device reports what happened when it tried to deploy the workload.  
**Auth:** RFC 9421 signature + Content-Digest header required  
**Body must include:** `apiVersion`, `kind: "DeploymentStatusManifest"`, `deploymentId`, `status.state`, `components` array  
**Valid states:** `pending`, `installing`, `installed`, `failed`, `removing`, `removed`  
**Returns:** `{ "acknowledgement": "received" }`

---

## 6. How the Data Flows End-to-End

This is the complete lifecycle in order:

```
Step 1: Device boots up
        ↓
Step 2: Device calls GET /onboarding/certificate
        ← Server returns WFM root CA cert
        ↓
Step 3: Device calls POST /onboarding with its own certificate
        ← Server validates body via assertions.json
        ← Server stores device certificate (for future sig verification)
        ← Server assigns default deployment
        ← Server returns { "clientId": "uuid" }
        ↓
Step 4: Device calls POST /capabilities with hardware info
        ← Server enforces RFC 9421 signature check using stored cert
        ← Server validates body via assertions.json
        ← Server stores capabilities
        ← Server returns { "status": "capabilities_received" }
        ↓
Step 5: Device calls GET /deployments
        ← Server builds manifest: lists all deployment IDs + digests + URLs
        ← Server returns JSON with ETag header
        ↓
Step 6: Device calls GET /bundles/{digest} (to get all YAMLs at once)
        ← Server builds tar.gz containing all ApplicationDeployment YAMLs
        ← Server verifies digest matches URL parameter
        ← Server returns binary archive
        ↓
        (OR: Device calls GET /deployments/{id}/{digest} for individual YAML)
        ← Server returns single YAML file + digest verification
        ↓
Step 7: Device deploys the workload (runs docker compose, helm, etc.)
        ↓
Step 8: Device calls POST /deployments/{id}/status with result
        ← Server enforces RFC 9421 signature check
        ← Server validates body via assertions.json
        ← Server stores status history
        ← Server returns { "acknowledgement": "received" }
```

---

## 7. RFC 9421 HTTP Message Signatures — Explained Simply

**What is it?**  
RFC 9421 is an internet standard that lets an HTTP client prove that a request came from a specific key holder. It's like a digital wax seal on a letter — it cannot be faked without the private key.

**Why is it needed here?**  
When a device sends its status to the WFM server, the server needs to be absolutely sure this status came from the actual device, not from someone pretending to be that device (an impersonation attack). The server verifies the message using the public certificate the device registered with during onboarding.

**What gets signed?**  
For every POST/PUT request, the device signs:
1. `@method` — the HTTP verb (`POST`)
2. `@target-uri` — the full URL
3. `content-digest` — a SHA-256 hash of the request body

**What headers does this produce?**  
```
Content-Digest: sha-256=:base64encodedSHA256hashOfBody:
Signature-Input: sig1=("@method" "@target-uri" "content-digest");created=1713541234;keyid="device-key"
Signature: sig1=:base64encodedSignatureBytes:
```

**Verification logic on the server (in order):**
1. If `Signature-Input` header is missing → **401 Unauthorized** (no signature = rejected)
2. If `Content-Digest` is empty or doesn't match the body → **400 Bad Request** (body tampered)
3. Extract public key from device's stored certificate
4. Run `htmsig` library verification against the signature → fails → **401 Unauthorized**
5. All checks pass → continue to business logic

**Which endpoints enforce signatures?**
- POST /onboarding → **No** (device has no registered key yet)
- POST/PUT /capabilities → **Yes**
- GET /deployments → **No** (read-only, low risk)
- GET /bundles → **No** (read-only)
- GET individual deployment → **No** (read-only)
- POST /status → **Yes**

**Library used:** `github.com/lestrrat-go/htmsig v1.0.0` — the same library the real device agent uses for signing. Using the same library on both sides guarantees format compatibility.

---

## 8. Server-Side Validation Engine — How Assertions Work

The server's validation engine is the most powerful non-code feature of this suite. Here is exactly how it works:

### Step 1 — Server startup loads assertions.json

```go
assertionsConfig, err = loadAssertions("manifests/assertions.json")
```

The entire JSON file is parsed into a Go struct called `AssertionsConfig` and held in memory. This happens once. If you change the file, you need to restart the server.

### Step 2 — Request arrives at an endpoint

When a POST arrives at `/capabilities`, the handler calls:

```go
errors := validateRequest("POST_capabilities", body)
```

The string `"POST_capabilities"` is the key that selects which endpoint's rules to apply from the loaded config.

### Step 3 — Rules are applied one by one

For each rule in `assertions.json` under that endpoint key, the engine checks:

| Rule type | What happens |
|-----------|-------------|
| `"required": true` and field missing | Adds error: `"<field> is required"` |
| `"type": "string"` but value is not | Adds error: `"<field> must be a string"` |
| `"value": "SomeExactValue"` but mismatched | Adds error: `"<field> must be '<value>'"` |
| `"enum": ["a","b"]` but value not in list | Adds error: `"<field> must be one of [a, b]"` |
| `"minLength": 1` but empty string | Adds error: `"<field> must be at least 1 characters"` |
| `"itemsEnum": ["a"]` for array items | Each item is checked against the enum |

The field path supports dot notation (`properties.resources.cpu.cores`) and wildcard (`interfaces.*.type`).

### Step 4 — Error response format depends on the endpoint's `validation_error_key`

Different endpoints return errors differently:
- `"badRequest"` → HTTP 400 with `{ "error": "first error message" }` (single string)
- `"unprocessable"` → HTTP 422 with `{ "status": "validation_failed", "errors": [...] }` (array)

This is also configured in `assertions.json` under `"error_responses"`.

### Step 5 — If no errors, business logic continues

If all rules pass, the handler proceeds to store data, build responses, etc.

---

## 9. How to Add a New Test (Step by Step, No Code)

Adding a new test requires editing only one file: `device-scenarios/test-scenarios.json`. No Go code changes are needed.

### Example: Add a test that verifies a device can be rejected during onboarding

**Open** `device-scenarios/test-scenarios.json` and find the appropriate scenario (e.g. `scenario-onboarding-errors`). Add a new step object inside its `steps` array:

```json
{
  "id": "step-4.5",
  "name": "Reject Onboarding With Unknown Kind",
  "method": "POST",
  "endpoint": "/api/v1/onboarding",
  "request_body": {
    "apiVersion": "onboarding.margo.org/v1alpha1",
    "kind": "SomethingWrong",
    "certificate": "MIID-some-cert"
  },
  "headers": {},
  "expected_status": 400,
  "validations": [
    {
      "field": "error",
      "operation": "contains",
      "value": "kind"
    }
  ],
  "extract_context": {}
}
```

### Test step fields explained:

| Field | Required | Meaning |
|-------|----------|---------|
| `id` | Yes | Unique identifier like `step-4.5` |
| `name` | Yes | Human-readable description shown in the report |
| `method` | Yes | `GET`, `POST`, or `PUT` |
| `endpoint` | Yes | URL path, may use `{placeholders}` filled from context |
| `request_body` | No | JSON body to send (omit for GET requests) |
| `headers` | No | Extra headers; set `"Content-Digest": ""` to force empty digest |
| `expected_status` | Yes | HTTP status code you expect back (200, 201, 400, 401, etc.) |
| `validations` | No | List of checks on the response body |
| `extract_context` | No | Save response fields as named variables for later steps |
| `skip_signing` | No | `true` = don't add signature headers (tests 401 responses) |
| `skip_certificate_injection` | No | `true` = don't replace `certificate` field with real device cert |

### Validation operations available:

| Operation | Meaning |
|-----------|---------|
| `equals` | Field value matches exactly |
| `contains` | Field value contains the string |
| `not_empty` | Field exists and is non-empty |
| `is_string` | Field is a string (any value) |
| `is_number` | Field is a number |
| `is_array` | Field is an array |
| `exists` | Field is present in the response |

### Multi-step scenarios using context:

To create a test that depends on a previous step's result (e.g. register first, then test):

```json
{
  "id": "step-7.0",
  "name": "Onboard Device (Setup)",
  "method": "POST",
  "endpoint": "/api/v1/onboarding",
  "request_body": { "apiVersion": "onboarding.margo.org/v1alpha1", "kind": "OnboardingRequest", "certificate": "placeholder" },
  "expected_status": 201,
  "extract_context": { "clientId": "clientId" }    ← saves the returned clientId
},
{
  "id": "step-7.1",
  "name": "Do Something With That Client",
  "method": "GET",
  "endpoint": "/api/v1/clients/{clientId}/deployments",  ← uses saved clientId
  "expected_status": 200,
  "headers": { "Accept": "application/vnd.margo.manifest.v1+json" }
}
```

---

## 10. How to Add a New Assertion Rule

Assertion rules live in `manifests/assertions.json`. The server loads them **at startup**. To change them, edit the file and restart the server.

### Example: Add a validation that `properties.vendor` cannot exceed 100 characters

Find the `POST_capabilities` endpoint section in `assertions.json` and add a new rule object to its `validations` array:

```json
{
  "rule_id": "capabilities-019",
  "field": "properties.vendor",
  "type": "string",
  "required": false,
  "maxLength": 100,
  "description": "properties.vendor must not exceed 100 characters"
}
```

> Note: If you add a new validation type (like `maxLength`) that the Go engine doesn't currently handle, you would need to add a few lines of Go code in the `applyRule()` function in `server.go`. However, all the types currently used (string, number, object, array, enum, minLength, itemsEnum) are already implemented.

### Example: Add a new endpoint's validation rules

To add server-side rules for a brand new endpoint, add a new key to the `"endpoints"` object:

```json
"POST_new_endpoint": {
  "path": "/api/v1/clients/{clientId}/some-new-thing",
  "method": "POST",
  "status_code": 201,
  "validation_error_key": "unprocessable",
  "validations": [
    {
      "rule_id": "new-001",
      "field": "kind",
      "type": "string",
      "required": true,
      "value": "NewKind"
    }
  ],
  "response_structure": {}
}
```

Then in `server.go`, in your new handler, call:

```go
errors := validateRequest("POST_new_endpoint", body)
```

### Example: Add a certificate to the rejected list

To make a new certificate value trigger a 403 response during onboarding:

```json
"rejected_certificates": [
  "MIID-rejected-device-cert",
  "MIID-another-blocked-cert"
]
```

---

## 11. Running the Suite — All Commands Explained

### Prerequisites

- Go 1.24+ installed
- Device agent certificates at: `/home/margo/sandbox/poc/device/agent/config/`
  - `device-public.crt` — device's X.509 certificate (public)
  - `device-private.key` — device's private key (for signing)
- Working directory: `/home/margo/nitin/margo/personas/device_supplier`

---

### Build the Server

```bash
go build -o bin/server server.go
```

**What this does:** Compiles `server.go` into a standalone binary at `bin/server`. Must be re-run any time `server.go` is modified.

---

### Build the Test Runner

```bash
go build -o bin/run_tests run_tests.go
```

**What this does:** Compiles `run_tests.go` into a standalone binary at `bin/run_tests`. Must be re-run any time `run_tests.go` is modified.

---

### Build Both (shortcut)

```bash
make build
```

**What this does:** Runs both build commands above in sequence.

---

### Start the Server

```bash
# Option A: Run in foreground (you see all logs)
./bin/server

# Option B: Run in background (log to file)
./bin/server &> /tmp/server.log &
```

**What you see on the console:**
```
🧹 Clearing persisted data for a clean test run (set KEEP_DATA=true to skip)
ℹ No persisted clients found (first startup)
✓ Loaded assertions from: manifests/assertions.json
✓ Using existing server TLS certificates
🚀 Mock WFM Server starting on https://localhost:3001
```

**Environment variable options:**
```bash
# Keep data from previous run (useful when testing with real device agent)
KEEP_DATA=true ./bin/server

# Use make
make run-server
```

---

### Run the Conformance Tests

```bash
# Option A: Run directly with go run (no compile step needed)
go run run_tests.go

# Option B: Run pre-built binary
./bin/run_tests

# Option C: Using make
make run-tests
```

**The server must already be running** before executing the tests.

**What you see:**
```
✅ WFM Server is ready

▶ Running Scenario: Device Onboarding (scenario-onboarding)
  → Step: Get Root CA Certificate
    ✅ PASS (HTTP 200)
  → Step: Onboard Trusted Device
    ✅ PASS (HTTP 201)
...

╔══════════════════════════════════════════════════════════════════╗
║  Test Results: 36 PASSED, 0 FAILED (Total: 36)
╚══════════════════════════════════════════════════════════════════╝
📊 Test report saved: reports/conformance-report-2026-04-19T16-17-59-000Z.html
```

---

### Run Everything in One Command (Full Demo)

```bash
make demo
```

**What this does:**
1. Builds both server and test runner
2. Starts the server in the background
3. Waits 2 seconds for startup
4. Runs all conformance tests
5. Saves the HTML report

---

### Stop the Server

```bash
# Using make
make kill-server

# Or directly
pkill -f "bin/server"
```

---

### View Test Report

After running tests, open the HTML report in a browser:

```bash
# Show the latest report file path
ls -t reports/*.html | head -1

# Open it (on Linux with a browser)
xdg-open reports/conformance-report-*.html
```

The report shows a table: step name, PASS/FAIL status, HTTP code, and failure reason.

---

### Install or Update Dependencies

```bash
go mod tidy
```

**What this does:** Downloads missing packages and removes unused ones. Run this after modifying `go.mod`.

---

### Clean All Build Artifacts

```bash
make clean
```

**What this does:** Deletes the `bin/` folder and `go.sum` file.

---

## 12. State Management and Data Lifecycle

### How state is stored

The server stores two kinds of runtime state in the `data/` directory:

- `data/clients.json` — all registered devices (their UUID, certificate, capabilities, and assigned deployment IDs)
- `data/deployments.json` — deployment status history

These files are plain, human-readable JSON.

### Clean startup (default behaviour)

By default, every time the server starts, it **deletes both files**:

```
🧹 Clearing persisted data for a clean test run (set KEEP_DATA=true to skip)
```

This is intentional: the test runner creates fresh clients on every run, and clients from a previous run have no meaning in a new run. Accumulating thousands of stale clients would slow down the server.

### Preserve state across restarts

```bash
KEEP_DATA=true ./bin/server
```

Use this when you want the server to remember previously registered clients — for example, when testing with a real device agent that already went through onboarding.

### Thread safety

The server uses a `sync.RWMutex` (`mu`) to protect all reads and writes to the `clients` and `deployments` maps. Multiple requests arriving at the same time cannot corrupt each other's data.

---

## 13. TLS and Certificate Architecture

All communication between the test runner and server is over **HTTPS**. Here is how the certificates are organised:

```
Root CA (ca-cert.pem + ca-key.pem)
    │
    ├── Signs → Server TLS certificate (server-cert.pem)
    │           Used by the mock WFM server for HTTPS
    │
    └── Counter-party reference for device agent
              Device has its own cert issued by same CA
```

- **`ca-cert.pem`**: The root authority that issued both the server and device certificates. Copied from `~/certs/ca-cert.pem` at server startup if not already present.
- **`server-cert.pem`**: Used for TLS (HTTPS). The test runner skips TLS verification (`InsecureSkipVerify: true`) because this is a known self-signed cert in a test environment.
- **Device certificate** (from `/home/margo/sandbox/poc/device/agent/config/device-public.crt`): Sent by the device during onboarding. Stored by the server and used later to verify RFC 9421 signatures.

### Generating Certificates

If certificates are missing, run:

```bash
bash generate-certs.sh
```

See `CERTIFICATE_GENERATION.md` for detailed certificate generation documentation.

---

## 14. Go Dependencies Explained

| Package | What It Does |
|---------|-------------|
| `github.com/gorilla/mux` | HTTP router — maps URL patterns like `/clients/{clientId}/...` to handler functions |
| `github.com/google/uuid` | Generates UUIDs (like `a3e2f5dc-912e-494f-8395-52cf3769bc06`) for client IDs |
| `github.com/lestrrat-go/htmsig` | RFC 9421 HTTP Message Signatures — both signing (in test runner) and verifying (in server). **Critical:** same library used by both sides guarantees format compatibility. |
| `github.com/lestrrat-go/sfv` | Structured Field Values (RFC 8941) — used internally by htmsig for parsing signature headers |
| `github.com/lestrrat-go/dsig` | Digital signatures support — used internally by htmsig |
| Standard library: `crypto/*` | X.509 certificates, RSA key generation, SHA-256 hashing |
| Standard library: `archive/tar`, `compress/gzip` | Building the `.tar.gz` bundle archive |
| Standard library: `encoding/json` | JSON parsing and serialisation |
| Standard library: `sync` | Thread-safety mutex for concurrent request handling |

---

## 15. What the Conformance Test Report Shows

Each HTML report saved to `reports/` includes:

| Column | Meaning |
|--------|---------|
| Step | Name of the test step |
| Status | ✅ PASS or ❌ FAIL |
| HTTP Code | The actual HTTP status code returned by the server |
| Details | For failures: the specific reason (e.g. `Expected HTTP 400, got 201`) |

### The 6 Test Scenarios (36 steps total)

| Scenario | ID | Steps | What It Covers |
|----------|----|-------|---------------|
| Device Onboarding | scenario-onboarding | 3 | Certificate download, successful onboarding, blocked certificate |
| Capabilities Reporting | scenario-capabilities | 3 | POST and PUT capability manifests |
| Deployment Retrieval and Status | scenario-deployments | 7 | Get manifest, ETag caching, bundle download, individual YAML, status reporting |
| Onboarding Error Handling | scenario-onboarding-errors | 4 | Wrong apiVersion, missing cert, empty cert, no signature (should succeed) |
| Capabilities Error Handling | scenario-capabilities-errors | 7 | Missing properties, wrong role, bad interface type, bad CPU arch, missing digest, no signature, unknown client |
| Status and Retrieval Errors | scenario-status-and-retrieval-errors | 9 | Unsupported Accept, wrong digest, bad state, missing component name, ID mismatch, missing digest, no signature, valid end-to-end |

---

## 16. Connecting to a Real Device Agent

The real device agent lives at `/home/margo/sandbox/agent` and reads config from `/home/margo/sandbox/poc/device/agent/config/config.yaml`.

To test with the real agent instead of the conformance test runner:

### Step 1: Start the mock server preserving data between restarts

```bash
cd /home/margo/nitin/margo/personas/device_supplier
KEEP_DATA=true ./bin/server
```

The `KEEP_DATA=true` flag prevents the server from wiping the registered clients each time.

### Step 2: Run the device agent

```bash
cd /home/margo/sandbox/poc/device/agent
/home/margo/sandbox/agent --config config/config.yaml
```

### Step 3: Watch the server logs

The server logs all incoming requests with their summary:

```
[Onboarding] ✅ Device accepted: a165aad5-1a9b-4b51-a04c-cc00e00748fc
[Capabilities] Accepted for client: a165aad5-1a9b-4b51-a04c-cc00e00748fc
[Deployments] GET request received for client: a165aad5-1a9b-4b51-a04c-cc00e00748fc
[Status] Update for deployment a3e2f5dc-912e-494f-8395-52cf3769bc06 under client a165aad5-...
```

### Step 4: Reset for a fresh cycle

To make the agent go through the full onboarding → deployment cycle again:

```bash
# Remove the agent's stored state
rm /home/margo/sandbox/poc/device/agent/data/agent.database.json

# Remove any deployed containers
docker rm -f $(docker ps -aq --filter "name=compose-sample") 2>/dev/null

# Stop and restart the server without KEEP_DATA (clean slate)
pkill -f "bin/server"
./bin/server
```

---

## 17. Common Questions and Answers

**Q: Do I need to recompile after changing `assertions.json` or `test-scenarios.json`?**  
No. Both files are read at runtime. Just restart the server (for assertions) or re-run the test runner (for scenarios).

**Q: Can I run the test runner without the server running?**  
No. The runner polls the `/health` endpoint and waits up to 5 seconds. If the server is not running, it exits with `❌ WFM Server not responding`.

**Q: What is the default deployment that every device gets?**  
Every newly registered client is automatically assigned one deployment:
- ID: `a3e2f5dc-912e-494f-8395-52cf3769bc06`  
- Workload: `nginx-proxy` docker-compose from GitHub  
- This is the Margo spec's example UUID and matches the real device agent's expected format.

**Q: What happens if a request has a valid signature but the body is tampered?**  
The `Content-Digest` header contains the SHA-256 hash of the original body. If the body is changed after signing, the hash won't match and the server returns HTTP 400. If there's no signature at all, the server returns HTTP 401.

**Q: Why does onboarding not need a signature?**  
Because the device hasn't registered its key with the server yet. Signatures are only possible after onboarding, when the server has stored the device's public certificate to verify against. Onboarding is the trust bootstrapping step.

**Q: Can I add a completely new scenario (not just a new step)?**  
Yes. Add a new top-level object to the array in `test-scenarios.json` with a unique `id`, a `name`, a `description`, and a `steps` array. The runner processes all scenarios in order.

**Q: How do I test with a different device certificate?**  
The test runner loads the certificate from `/home/margo/sandbox/poc/device/agent/config/device-public.crt` at startup. To use a different cert, either:
1. Replace that file before running the tests, or
2. Set `skip_certificate_injection: true` in specific test steps and put a custom certificate string directly in the `request_body.certificate` field.

**Q: Why does the URL contain the digest? (e.g. `/bundles/sha256:abc123`)**  
This is called **content-addressing**. The digest in the URL is a promise: the server guarantees that anyone who fetches `sha256:abc123` always gets the exact same bytes. This makes deployments safe from time-of-check/time-of-use races (the YAML can't change between when the manifest listed it and when the device actually fetches it).

**Q: What is the `manifestVersion` field in the deployment manifest?**  
It is a monotonically increasing integer (starts at 1, goes up). The device agent remembers the latest version it saw. If the server ever returns a lower version, the agent rejects it as a potential rollback attack.

**Q: Where do I find the HTML report after a test run?**  
```
reports/conformance-report-<YYYY-MM-DDTHH-MM-SS-000Z>.html
```
The filename includes the UTC timestamp of when the run started.

---

*This document covers the full conformance suite as of April 2026. All 36 tests pass against Margo Management Interface API v1.0.0.*
