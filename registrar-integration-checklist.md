# Registrar Integration Checklist

Use this checklist to scope and estimate a Registrar integration with Seamless Connect.

The initial Registrar integration is intentionally narrow:

**automate DNSSEC delegation management between the Domain Owner, DNS Provider, and Registrar.**

Domain registration, availability, pricing, payment, renewals, transfers, nameserver management, and broad Registrar CRUD APIs are explicitly out of scope for this phase.

Seamless Connect should absorb as much cross-provider coordination complexity as possible while allowing the Registrar to retain control over authorization, policy, and authoritative parent-side DNSSEC delegation changes.

## Terminology and roles

This document names the actor responsible for each action. Do not use the unqualified term "provider" in integration requirements or implementation notes.

- **Service Provider (SP):** the application or service that requests an operation for its customer and receives status from Seamless Connect.
- **Registrar:** the organization sponsoring the domain registration and handling parent-side DNSSEC delegation.
- **DNS Provider:** the operator of the authoritative DNS service that executes DNS-zone operations and prepares child-side DNSSEC state, including signing the zone and publishing DNSSEC signaling records.
- **Domain Owner:** the registrant or authorized user approving DNSSEC-related actions.
- **Registry:** the operator of the parent zone where DS records ultimately become authoritative.
- **Parental Agent:** the system authorized to act on the parent side of DNSSEC delegation automation, whether operated by the Registrar, Registry, Seamless Connect, or another delegated component.
- **Seamless Connect:** the open coordination layer connecting the Domain Owner, DNS Provider, and Registrar.

One organization may perform more than one role. Name the role being performed rather than relying on the organization's name.

## What Seamless Connect handles

All integrations use one consistent, openly documented integration contract. Within that contract, Seamless Connect owns the coordination required to move a DNSSEC operation safely between the DNS Provider and Registrar.

This may include:

- discovering the DNS Provider and Registrar;
- coordinating Domain Owner authorization;
- determining supported DNSSEC automation capabilities;
- accepting registrar-originated or DNS-provider-originated initiation;
- coordinating child-side DNSSEC readiness;
- observing CDS and CDNSKEY signals published by the DNS Provider;
- initiating or triggering Registrar-side processing when appropriate;
- providing or coordinating Parental Agent processing where the Registrar does not already operate it;
- handling synchronous and asynchronous execution;
- maintaining operation state;
- bounded polling, retries, and backoff;
- normalizing Registrar and DNS Provider status;
- coordinating multi-step workflows;
- verifying DS publication and DNSSEC state;
- reporting completion or failure; and
- shared conformance and integration observability.

The Registrar should not need to build separate integrations with every participating DNS Provider.

Registrar-specific capabilities, limitations, and policy constraints must be documented in this open-source repository as reviewable metadata, not established through private bilateral exceptions.

## Minimum integration

The initial integration uses the same standards-based DNSSEC lifecycle regardless of which supported side initiates the operation. Registrar-originated initiation is the working pilot proposal, not a requirement for every integration.

### Working pilot proposal: registrar-originated

```text
Domain Owner
     │
     │ Enable DNSSEC
     ▼
Registrar
     │
     │ authorized DNSSEC request
     ▼
Seamless Connect
     │
     │ coordinate child-side readiness
     ▼
DNS Provider
     │
     │ sign zone
     │ publish CDS / CDNSKEY
     ▼
Seamless Connect / Parental Agent
     │
     │ validate + reconcile
     ▼
Registrar
     │
     │ DS update
     ▼
Registry / Parent
```

### Supported alternative: DNS-provider-originated

```text
Domain Owner
     │
     │ Enable DNSSEC
     ▼
DNS Provider
     │
     │ sign zone
     │ publish CDS / CDNSKEY
     ▼
DNS / Seamless Connect
     │
     │ signal detected
     ▼
Registrar / Parental Agent
     │
     │ validate + reconcile
     ▼
Registry / Parent
```

The second model is especially useful where the Registrar already detects or reacts to CDS/CDNSKEY changes through a Parental Agent process.

**Registrar work:**

1. For the working pilot proposal, authorize or accept registrar-originated DNSSEC intent; otherwise document the supported initiation path.
2. Provide Seamless Connect with scoped authorization where direct Registrar API access is required.
3. Expose or identify the existing path used to read and update DS information.
4. Publish the Registrar's supported DNSSEC automation capabilities and policy constraints.
5. Return sufficient status for Seamless Connect to determine whether an operation succeeded, failed, or remains pending.
6. Participate in conformance testing and a limited production pilot.

Seamless Connect handles discovery, cross-provider coordination, operation state, polling where needed, retries, and verification.

## Integration principle

**Use existing open DNSSEC standards wherever they cover the use case.**

The Registrar carries or enforces the registrant's authority over the domain delegation. The DNS Provider controls the child zone and publishes standardized DNSSEC state through CDS/CDNSKEY.

Seamless Connect coordinates the lifecycle between those administrative boundaries.

The preferred architecture should support both:

- **Registrar-originated initiation**, where the Domain Owner requests DNSSEC through the registrar-side control plane and Seamless Connect coordinates child-side readiness; and
- **DNS-provider-originated initiation**, where the DNS Provider publishes CDS/CDNSKEY and the Registrar or Parental Agent detects and processes those signals according to policy.

These are different initiation patterns for the same standards-based DNSSEC lifecycle.

Seamless Connect does not replace CDS/CDNSKEY or other DNSSEC standards. It provides coordination around them.

Where functionality is not defined by an existing standard, Seamless Connect should use an explicit, deterministic operation and document the gap openly, with the goal of adopting or developing interoperable standards rather than creating a closed protocol.

Service Providers request operations when they participate, subject to Domain Owner authorization. Seamless Connect also supports the documented Registrar- and DNS-provider-originated DNSSEC initiation patterns. Seamless Connect coordinates a published and auditable workflow; it does not invent privileged domain changes.

## Initial integration: DNSSEC

### 1. Support DNSSEC initiation

The first integration decision is how a DNSSEC lifecycle begins.

The Registrar should support at least one interoperable initiation path and may support both.

#### Path A: Registrar-originated initiation

In this model, the Domain Owner exercises domain-level authority through the Registrar or an authorized Seamless Connect flow associated with the Registrar.

Example:

```text
Domain Owner
     │
     │ "Enable DNSSEC"
     ▼
Registrar
     │
     ▼
Seamless Connect
     │
     │ request child-side readiness
     ▼
DNS Provider
```

This path is useful when the Registrar is the natural control plane for domain lifecycle operations.

The Registrar does not necessarily need to build new UI for the initial pilot. Seamless Connect may provide the pilot UI or API while relying on Registrar authorization to establish the Domain Owner's authority.

Under the working pilot proposal, the Registrar would authorize or accept that registrar-originated intent and expose its existing DS-management path. Seamless Connect may provide or coordinate the Parental Agent processing that validates the DNS Provider's child-side signals and uses that path; the Registrar does not need to build a new Parental Agent implementation solely for the pilot.

#### Path B: DNS-provider-originated initiation

In this model, the Domain Owner enables DNSSEC at the DNS Provider.

The DNS Provider signs the child zone and publishes CDS/CDNSKEY. The Registrar or Parental Agent detects the standardized signal and processes it according to its policy.

Example:

```text
Domain Owner
     │
     │ "Enable DNSSEC"
     ▼
DNS Provider
     │
     │ publish CDS / CDNSKEY
     ▼
Registrar / Parental Agent
```

Seamless Connect may assist with discovery, signaling, monitoring, status, and verification without replacing the standards-based state exchange.

<details>
<summary>Checklist items</summary>

- [ ] Confirm whether Registrar-originated DNSSEC initiation is supported.
- [ ] Confirm whether DNS-provider-originated DNSSEC initiation is supported.
- [ ] Confirm whether the Registrar already detects or reacts to CDS/CDNSKEY through a Parental Agent process.
- [ ] Identify how Domain Owner authority is established for each supported path.
- [ ] Identify whether Seamless Connect may initiate Registrar-side processing through an API.
- [ ] Identify whether Seamless Connect may notify or trigger an existing Parental Agent process.
- [ ] Document any initiation path that requires manual intervention.
- [ ] Ensure both initiation paths converge on the same Registrar DNSSEC policy and validation rules.

</details>

**Registrar deliverable:** a documented DNSSEC initiation model identifying which paths are supported and how Domain Owner authority is established.

### 2. Confirm supported DNSSEC operations

The Registrar defines which DNSSEC delegation operations Seamless Connect may coordinate.

The initial scope should align with standardized DS automation.

Examples include:

- initial DNSSEC enablement;
- DS updates associated with key rollover; and
- DNSSEC disablement where supported by the applicable standard and Registrar policy.

<details>
<summary>Checklist items</summary>

- [ ] Confirm support for reading current DS state.
- [ ] Confirm support for adding or replacing DS records.
- [ ] Confirm whether automated initial DNSSEC enablement is supported.
- [ ] Confirm whether automated DS updates / rollover are supported.
- [ ] Confirm whether automated DNSSEC disablement is supported.
- [ ] Identify operations that require additional Domain Owner confirmation.
- [ ] Document unsupported operations.

</details>

**Registrar deliverable:** a published capability profile describing supported DNSSEC delegation operations.

### 3. Provide delegated authorization where required

Where Seamless Connect directly invokes Registrar APIs, it needs narrow authority to perform only the DNSSEC actions approved for the Domain Owner and Registrar integration.

OAuth with granular scopes is preferred where available, but another secure delegated authorization mechanism may be used.

Example conceptual scope:

```text
domain:example.com
permission:dnssec.read
permission:dnssec.update
```

A Registrar that operates a fully standards-driven Parental Agent may require less direct Seamless API authority if it can process CDS/CDNSKEY independently.

<details>
<summary>Checklist items</summary>

- [ ] Provide OAuth or an equivalent delegated authorization mechanism where direct API access is required.
- [ ] Limit authorization to the applicable domain or account resources.
- [ ] Limit permissions to DNSSEC/delegation operations.
- [ ] Support credential expiration or revocation.
- [ ] Document authorization lifetime and refresh behavior.
- [ ] Ensure Seamless Connect does not require unrelated Registrar account privileges.
- [ ] Document cases where standards-based processing can proceed without direct Seamless API access.

</details>

**Registrar deliverable:** a test authorization path appropriate to the supported DNSSEC initiation and execution model.

### 4. Expose or identify the existing DNSSEC API

Where API integration is needed, Seamless Connect should use the Registrar's existing supported API wherever possible. The same requirement may be satisfied by an existing Registrar workflow that a Parental Agent can invoke or trigger.

The Registrar does not need to build a separate Seamless-specific DNSSEC API if existing interfaces provide the required functionality.

At minimum, Seamless Connect should be able to determine current DS state and, for API-mediated flows, request a supported DS change.

<details>
<summary>Checklist items</summary>

- [ ] Identify the API endpoint used to retrieve current DS records or DNSSEC status.
- [ ] Identify the API endpoint used to add, replace, or remove DS records as supported.
- [ ] Document authentication requirements.
- [ ] Document request and response formats.
- [ ] Document rate limits.
- [ ] Document retryable and non-retryable errors.
- [ ] Identify any stable request or transaction identifier returned by the Registrar.
- [ ] Identify which parts of the workflow are handled by an existing Parental Agent rather than an API request.

</details>

**Registrar deliverable:** API and/or Parental Agent documentation sufficient for Seamless Connect to coordinate the DNSSEC workflow.

### 5. Publish Registrar DNSSEC policy

The Registrar remains authoritative for deciding which DNSSEC signals and operations it will accept.

The Registrar's implementation requirements and constraints should be documented openly where possible so Seamless Connect can enforce and test them consistently.

This may include:

- CDS versus CDNSKEY support;
- requirements for accepting initial trust;
- accepted digest algorithms;
- observation or hold-down periods;
- authentication requirements;
- domain or TLD restrictions;
- concurrency behavior;
- conditions under which existing DS records may be replaced;
- DNSSEC removal requirements; and
- cases requiring manual intervention.

<details>
<summary>Checklist items</summary>

- [ ] Document CDS support.
- [ ] Document CDNSKEY support.
- [ ] Document initial-enrollment policy.
- [ ] Document update / rollover policy.
- [ ] Document DNSSEC-removal policy.
- [ ] Document required validation or observation periods.
- [ ] Document supported algorithms and other technical constraints.
- [ ] Document concurrency / locking behavior.
- [ ] Document conditions that require manual review.
- [ ] Publish these constraints in a machine-readable or repository-maintained profile where practical.

</details>

**Registrar deliverable:** a reviewable Registrar DNSSEC policy/capability definition suitable for use by Seamless Connect.

### 6. Coordinate child-side DNSSEC readiness

For registrar-originated workflows, Seamless Connect must be able to move from a Domain Owner's request to a DNS Provider state that is ready for parent-side processing.

The Registrar does not need to understand each DNS Provider's implementation.

Example:

```text
Registrar-authorized request
        ↓
Seamless Connect
        ↓
discover DNS Provider
        ↓
request DNSSEC readiness
        ↓
DNS Provider signs zone
        ↓
CDS / CDNSKEY published
```

For DNS-provider-originated workflows, this step may already have occurred before the Registrar becomes involved.

<details>
<summary>Checklist items</summary>

- [ ] Confirm that Registrar-originated operations may wait for child-side DNSSEC readiness.
- [ ] Preserve operation state while the DNS Provider prepares the zone.
- [ ] Do not treat Registrar initiation as equivalent to immediate DS publication.
- [ ] Allow the same Registrar validation rules to apply regardless of which side initiated the lifecycle.

</details>

**Registrar deliverable:** agreement that child readiness and parent delegation updates are separate stages of the DNSSEC operation.

### 7. Process standardized CDS/CDNSKEY signals

Once the child zone is DNSSEC-ready, the parent-side workflow should rely on standardized DNSSEC signaling wherever possible.

CDS/CDNSKEY remains the standards-based mechanism for communicating desired DS state from the child toward the parent.

This applies regardless of whether the lifecycle began at the Registrar or DNS Provider.

#### Registrar-originated

```text
Domain Owner request
       ↓
Registrar / Seamless
       ↓
DNS Provider prepares zone
       ↓
CDS / CDNSKEY
       ↓
Registrar / Parental Agent
```

#### DNS-provider-originated

```text
DNS Provider prepares zone
       ↓
CDS / CDNSKEY
       ↓
Registrar / Parental Agent
```

The second half of the workflow can therefore be identical.

<details>
<summary>Checklist items</summary>

- [ ] Detect or receive valid CDS/CDNSKEY signaling.
- [ ] Apply the Registrar's required acceptance checks.
- [ ] Process signals consistently regardless of initiation path.
- [ ] Reject signals that do not satisfy published policy.
- [ ] Return or expose clear status when processing cannot proceed.
- [ ] Preserve standardized DNS data as the source of truth rather than replacing it with a Seamless-specific cryptographic representation.

</details>

**Registrar deliverable:** successful standards-based processing of a test DNSSEC delegation change.

### 8. Return usable operation status

DNSSEC coordination may not complete within a single synchronous request.

The Registrar should expose enough information for Seamless Connect to distinguish:

- accepted;
- awaiting child readiness;
- awaiting parental processing;
- processing;
- completed;
- rejected;
- authorization required;
- retryable failure; and
- permanent failure.

A Registrar does not necessarily need to implement a new asynchronous API. Seamless Connect can normalize API responses, DNS observation, or an existing Parental Agent process into its own durable operation model.

<details>
<summary>Checklist items</summary>

- [ ] Return explicit success or failure for synchronous API requests.
- [ ] Return a stable operation or transaction identifier where available.
- [ ] Provide a status endpoint, callback, or readable resource state for long-running operations where available.
- [ ] Allow DNS state to serve as observable progress where appropriate.
- [ ] Identify errors that may safely be retried.
- [ ] Identify errors requiring Domain Owner action.
- [ ] Avoid requiring callers to infer success from undocumented behavior.

</details>

**Registrar deliverable:** documented execution/status behavior that Seamless Connect can map into its operation lifecycle.

### 9. Allow final-state verification

Seamless Connect should be able to verify that the requested DNSSEC delegation state actually became authoritative.

For example:

```text
DNS Provider signs zone
        ↓
CDS / CDNSKEY available
        ↓
Registrar / Parental Agent processes signal
        ↓
Registry publishes DS
        ↓
Seamless Connect verifies
        ↓
COMPLETED
```

<details>
<summary>Checklist items</summary>

- [ ] Allow Seamless Connect to retrieve the Registrar's current DNSSEC/DS state where available.
- [ ] Return the DS values submitted or currently associated with the domain where available.
- [ ] Permit independent verification against authoritative DNS.
- [ ] Define what constitutes Registrar-side completion.
- [ ] Distinguish Registrar acceptance from final parent-zone publication where applicable.
- [ ] Apply the same completion criteria regardless of initiation path.

</details>

**Registrar deliverable:** a reliable way for Seamless Connect to determine that Registrar-side and parent-side processing have completed.

### 10. Test conformance and both initiation paths

Testing should focus on the safety and interoperability properties of DNSSEC automation rather than broad Registrar functionality.

<details>
<summary>Checklist items</summary>

- [ ] Test registrar-originated initial DNSSEC enablement where supported.
- [ ] Test DNS-provider-originated initial DNSSEC enablement where supported.
- [ ] Test CDS/CDNSKEY detection and processing.
- [ ] Test a DS update / key rollover.
- [ ] Test Domain Owner authorization denial.
- [ ] Test invalid or unacceptable CDS/CDNSKEY data.
- [ ] Test existing conflicting DS state.
- [ ] Test duplicate / idempotent requests.
- [ ] Test delayed child-side readiness.
- [ ] Test delayed parent-side processing.
- [ ] Test retryable Registrar failure.
- [ ] Test permanent rejection.
- [ ] Test final DS verification.
- [ ] Confirm every operation can be correlated through Seamless Connect and Registrar logs.

</details>

**Registrar deliverable:** passing DNSSEC integration and conformance tests for the initiation paths the Registrar supports.

### 11. Run a limited DNSSEC pilot

The first production integration should remain deliberately narrow.

A suitable pilot is:

**one Registrar + one or more DNS Providers + a controlled set of domains + DNSSEC only.**

The working proposal is to begin with registrar-originated initiation because it gives the Domain Owner a natural domain-lifecycle control point. This is a pilot hypothesis, not a permanent restriction. The pilot should preserve and evaluate DNS-provider-originated initiation where the Registrar already detects or reacts to CDS/CDNSKEY through a Parental Agent process.

<details>
<summary>Checklist items</summary>

- [ ] Confirm the registrar-originated initiation flow proposed for the first pilot.
- [ ] Identify any existing DNS-provider-originated path that should remain active or be tested.
- [ ] Select participating DNS Providers and a controlled set of domains.
- [ ] Restrict the pilot to DNSSEC delegation operations.
- [ ] Define authorization, rollback, manual-review, and disable controls.
- [ ] Measure initiation, authorization, child-readiness, CDS/CDNSKEY processing, DS publication, verification, retry, failure, and completion timing.
- [ ] Compare the behavior and outcomes of both initiation paths where both are available.
- [ ] Review failures jointly and update Registrar or DNS Provider capability and policy definitions when observed behavior differs from documentation.
- [ ] Record which assumptions were validated before expanding the integration or standardizing an initiation model.

</details>

**Registrar deliverable:** a limited DNSSEC pilot with measurable results and a recorded decision on the next phase.

## Explicitly out of scope

The initial Registrar integration and pilot do not require Seamless Connect to support:

- domain search or availability;
- domain registration;
- domain transfers;
- nameserver management;
- domain pricing or premium-domain pricing;
- shopping-cart or checkout flows;
- payment, billing, or refunds;
- renewals, expiration, or redemption;
- registrant contact management;
- broad Registrar account administration;
- broad or general-purpose Registrar CRUD APIs; or
- Registry operations unrelated to DNSSEC delegation.

These capabilities may be evaluated independently in future work. They should not expand the scope, authorization requirements, or implementation estimate for the initial DNSSEC integration.
