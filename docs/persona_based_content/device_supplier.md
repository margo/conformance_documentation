# Device Supplier Testing Outline
## Focus Area Drawing
![Drawing](../assets/device-conformance.drawio.svg)

## Focus: Edge compute device hosting compliant workloads and communication patterns.
Major test themes from this persona:

1. Workload Management Interface Client — Ensure enrollment, desired state interpretation, and status updates.
2. Runtime Capability — provider support(Helm + Compose) container engine/orchestrator baseline, resource reporting, hardware profiles.
3. Compatibility Reporting — device advertises capabilities used by WFM to match apps.
4. Workload Observability checks — lifecycle signals, resource quotas, isolation policies.
(Per PR1 FAQ, device management—firmware/OS—is out of scope until GA; keep those tests gated behind GA1.)

### Current specification requirements
For a list of the latest requirements listed in the current pre-draft specification check [Device Supplier Conformance Requirements](../persona_based_content/device_supplier_conf_requirements.md).

## Test Suite element 1: Workload Management Interface - Client

<img width="211" height="238" alt="Image" src="https://github.com/user-attachments/assets/dc8024d4-0139-431c-9e65-b4a45503b726" />

End to End workflow verification(Margo means)

- Pulling the application package into WFM
- Ensure enrollment of clients
- Ensuring it can then create an instance deployment specification
- Ensuring edge client can pull from RESTFull API endpoint(deployment specification)
- Workload deployment status
- Device capabilities. 
- WE WOULD OWN THIS TEST INFRASTRUCTURE
- Postman collections
- Need to verify the artifacts being transported to ensure conformance to API artifacts. 

- we would need a mimicked WFN to enable this testing workflow. 


Other ways to verify API contracts. 
.RUST 
.HTTP files

## Test Suite element 2: Workload Deployment and runtime of defined artifacts
Within Margo, Application components the device's capabilities enables the WFM to make conscious decisions regarding where to place the applications. It is assumed that when assigning an application to the device, the device itself should have all required services to pull and deploy the artifacts assigned. 
- These items the device(s) need to pull include the following: Helm Packages, Compose Packages, and OCI Containers. 

To ensure vendor interoperability, we need to ensure where the devices house the services to pull their associated application artifacts. 

Test Items:
1 and 2 can inherit the OCI endpoint verification
- Can pull the helm outlined in deployment specification
- Can then pull the OCI artifacts
- Can then "RUN" the workload

Would need to implement a example application being deployed to ensure it can achieve all necessary steps. 

Testing of the Hosting capabilities that are reported in the other test suite/bucket. 

Still working through how compose Tar files are pulled. 
- Will integrate into this testing. 


## Test Suite element 3: Application/Platform Observability Support

<img width="409" height="306" alt="Image" src="https://github.com/user-attachments/assets/7cd650cf-743c-46ee-814b-82d887107b09" />

Test Items:
Deploy example app that is instrumented via OTEL
- Can that example app find the OTEL endpoints via environment variables
- does the Collector properly store the OTEL information
- Ensure the Collector can be configured to export to a example observability platform. 
- Verify the required container runtime metrics are being provided to the local OTEL collector. 


# Current SUPs that have an impact on this persona
## Test Suite element 4: Gateway Support

<img width="491" height="290" alt="Image" src="https://github.com/user-attachments/assets/aba1a25d-d5dc-490f-bd23-b7263b0e033c" />

## Test Suite element 5: Support for deployment artifacts if WFM ingests artifacts

