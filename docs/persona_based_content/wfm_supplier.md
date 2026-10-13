# Workload Fleet Manager Supplier Testing Outline
## Focus Area Drawing
![Drawing](../assets/wfm-conformance.drawio.svg)

## Focus: Orchestration workflows across mixed devices/apps.
Test groups:

1. Repository Enrollment — application registry + component registries (Helm charts and Compose Archives via OCI); credential handling.
2. [optional]  Device Enrollment — clients authenticate with mutual TLS using X.509-SVIDs; enrollment is out-of-band operator provisioning. Devices are matched by reported capabilities (deployment types and runtimes).
3. [optional]  Deployment Workflows — schedule/execute/rollback; configuration capture; compatibility checks (device eligibility rules).
4. [optional] Telemetry Surfaces — application OTEL + infrastructure metrics visibility.
5. Support for Gateway functionality (opaque and see-thru gateways, directed and autonomous placement).
6. Identity and Authorization — WFM identity profile, client authorization by SPIFFE ID, trust bundle endpoints when the WFM acts as identity issuer.
7. Extensibility — `custom` runtimes/deployment types and specification extensions pass-through.


### Current specification requirements
Specification baseline: **v1.0.0-rc.3 (PR3)**. For the full list of requirements check [WFM Supplier Conformance Requirements](../persona_based_content/WFM_supplier_conf_requirements.md).


## Test Suite element 1: Application Description retrieval / Interpretation

<img width="254" height="279" alt="Image" src="https://github.com/user-attachments/assets/6f77314f-c08e-4583-8ab9-ba10be816bd3" />

Notes from meeting:
App description being pulled via the WFM

- OCI standard for storing and distributing
- follows notes of the OCI containers getting to the edge in previous.

- Can be driven via OCI verification no need for Postman collections here. 
	- App registry is following our expected format. 
	- We can use a OCI Client (Test the APP Supplier)

- Client side: OCI conformance tests are for the "Server" API. RESTful api. 

Is this a candidate for end to end testing?

- Pull
- verify the application pacakge was pulled.  
- create deployment specification


## Test Suite element 2: Workload Management Interface (interface to device)

<img width="254" height="279" alt="Image" src="https://github.com/user-attachments/assets/211fddc3-309d-4d46-901f-6d0c644d94e8" />

Notes from meeting:
End to End workflow verification(Margo means)

- Pulling the application package into WFM
- Ensure clients authenticate with mutual TLS (X.509-SVID) and are authorized by SPIFFE ID; requests without a valid client certificate are rejected
- Ensuring it can then create an instance deployment speciciation
- Ensuring edge client can pull from RESTFull API endpoint(deployment specification)
	- State manifest: `manifestVersion` starts at 1 and strictly increases; ETag, `Cache-Control`, and `If-None-Match`/304 behavior
	- Bundle: present and `null` when there are no deployments; never an empty archive
- Workload deployment status (including `adoptedManifestVersion`)
- Device capabilities: `201`/`200`/`204` success codes, child-before-gateway `404`
- Errors returned as RFC 9457 problem details using the Margo problem type catalog
- WE WOULD OWN THIS TEST INFRASTRUCTURE
- Postman collections
- Need to verify the artifacts being trasnported to ensure conformance to API artifacts. 

- we would need a mimicd device to enable this testing workflow. 



## Test Suite element 5: Gateway support

<img width="366" height="364" alt="Image" src="https://github.com/user-attachments/assets/cbd6a8f5-fb54-4d7d-8f58-5ec715bcf46b" />

Test Items:

- Infer gateway relationships from `deviceId` paths; reject a child manifest received before its gateway with `404`
- Directed placement (`targetName: gateway/child`) and autonomous placement (`gateway/*`)
- https://docs.margo.org/specification/margo-management-interface/device-capabilities#gateways-considerations

## Test Suite element 6: Identity and Authorization

Test Items:

- Requires mutual TLS on every management request and presents its own `spiffe://<trust-domain>/margo/wfm/<wfm-id>` SVID
- Rejects clients of another WFM, non-client paths, and foreign trust domains; enforces its accepted-client policy
- TLS and session lifetime limits; TLS-terminating proxies strip incoming `Client-Cert` headers
- If acting as identity issuer: serves the discovery document and trust bundle endpoints
- https://docs.margo.org/specification/identity/wfm-identity-profile

## Test Suite element 7: Extensibility

Test Items:

- Unknown runtimes and deployment types default to `custom`
- Unknown specification extensions do not cause failures; WFM-targeted extensions are not passed to the device; device-targeted extensions are passed through in the `ApplicationDeployment`
- https://docs.margo.org/specification/extensibility/specification-extensions

