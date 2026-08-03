# Application Supplier Conformance Testing
## Focus Area Drawing
![Drawing](../assets/app-conformance.drawio.svg)

## Focus: Application Package + Application Description → hosting components → telemetry.
Major test themes from this persona:

- App Description Conformance — schema validity, required fields, compatibility metadata.
- Packaging & Distribution — App description/Helm chart/Compose archive checks, OCI registry references, signature policies.
- Deployment & Lifecycle (with a reference WFM) — install, upgrade, remove, configuration capture/validation.
- [optional]  Observability (OTEL) — Application follows the prescribed pattern (TWG noted “optional but standardized”).

### Current specification requirements
For a list of the latest requirements listed in the current pre-draft specification check [App Supplier Conformance Requirements](../persona_based_content/app_supplier_conf_requirements.md).

## Test Suite element 1: Margo Application Package
File(s) to enable the application description package

### Test Items(in scope):
- Margo application description file content:
	- Test suite would only test mandatory elements
	- Configurable attribute schema validation
- Verify artifact location that the description file is pointing to.
	- What the file is pointing to, is it valid/actionable?
- Verify Margo application description package format
	- Specification assumes the file(s) Must be packaged in an OCI format. 
- Signature of this artifact(Authenticity from vendor)
	- cosign currently outlined in our specification 

## Test Suite element 2: Application Description Package Interface

### Test Items(in scope):
Application Description Package Interface
- Testing the App Suppliers interface conformance to the OCI distribution specification
- Testing the implementation of the oci artifacts in the registry
- Testing Authentication/access to repo via WFM
	- (NOT MANDATED IN THE SPECIFICATION TODAY)
	- https://docs.margo.org/specification/applications/application-registry#authentication-authorization--security

## Test Suite element 3: Application component verification

**Meta Note**
- Feb/11th: Was decided we should not take on scope that other test infrastructure/strategies already exist. 
- I(Armand) wanted to include these items in our write up, not to declare this testing in scope, but to ensure completeness.
	- If these items are not going to be tested in Margo conformance suite, we should enable users to find where information exists to assist.
	- Additionally, we should explicitly state what is NOT in scope of our conformance suite. 

### Test Items(in scope):
- Artifact signing standard for deployment artifacts
	- OCI artifacts/containers - Covers the application description package and OCI containers
		- cosign is described in the following writeup.
			- https://docs.margo.org/specification/applications/application-registry#authentication-authorization--security
	- Helm/Compose packages
		- Helm packages can be wrapped as an OCI artifact, therefore we could sign that artifact
			- However, this is not mandated in the specification
	- Compose packages
		- "When digitally signing the package PGP encryption MUST be used."
		- https://docs.margo.org/concepts/applications/application-package 
			- 
	

### Test Items(deemed out of scope):
- Test of Helm Charts to specification
	- https://helm.sh/docs/helm/helm_lint/
- Test of Compose file/package to specification
	- Doesn't seem to be great tooling readily available for verifying Compose implementations
		- https://github.com/compose-spec/conformance-tests 
- Testing of container images linked via the associated manifest
	- https://github.com/opencontainers/oci-conformance/blob/main/README.md

## Test Suite element 4: Application Components interface and deployment
Within Margo, Application componentsthe device's capabilities enables the WFM to make conscious decisions regarding where to place the applications. It is assumed that when assigning an application to the device, the device itself should have all required services to pull and deploy the artifacts assigned. 
- These items the device(s) need to pull include the following: Helm Packages, Compose Packages, and OCI Containers. 

To ensure vendor interoperability, we need to ensure where the devices house the services to pull their associated application artifacts. 

### Test Items(in scope of Margo??):
- Application Supplier Component interface
	- Helm
		- Local helm operator will pull helm packages defined via the deployment specification
		- Typically stored in an OCI registry
			- Not specified in specification at this time
	- Compose
		- Local compose service will pull compose artifacts
		- No outlined strategy for where the compose archive is stored / what service device is required to host(vendor interoperability focus.
	- OCI Containers
		- Pulled and verified via the OCI Container runtime(containerd / CRI-O)
