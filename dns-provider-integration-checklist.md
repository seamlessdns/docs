# DNS Provider Integration Checklist

Use this checklist to scope and estimate a DNS Provider integration with Seamless Connect. It separates DNS Provider-owned work from responsibilities handled by Seamless Connect and is intended for a joint product and engineering review.

The working pilot is intentionally narrow:

**support registrar-originated DNSSEC enablement by preparing and signing the child zone, publishing CDS/CDNSKEY, and exposing enough state for Seamless Connect to coordinate parent-side completion.**

The remainder of the checklist also covers the broader DNS record automation capabilities that a DNS Provider integration may support beyond the initial DNSSEC pilot.

## Terminology and roles

This document names the actor responsible for each action. Do not use the unqualified term "provider" in integration requirements or implementation notes.

- **Service Provider (SP):** the application or service that initiates a domain configuration request for its customer and receives status from Seamless Connect.
- **DNS Provider:** the operator of the authoritative DNS service and DNS management API used to read or change the domain's DNS records.
- **Registrar:** the organization sponsoring the domain registration. A Registrar may also act as the DNS Provider, but registration and DNS-hosting responsibilities remain distinct.
- **Domain Owner:** the registrant or authorized user who approves access and DNS changes.
- **Registry:** the operator of the parent zone where DS records ultimately become authoritative.
- **Parental Agent:** the system authorized to act on the parent side of DNSSEC delegation automation, whether operated by the Registrar, Registry, Seamless Connect, or another delegated component.
- **Seamless Connect:** the open coordination layer connecting the Domain Owner, DNS Provider, Registrar, and Service Provider where one participates.

One organization may perform more than one role. Name the role being performed—for example, "Registrar discovery" or "DNS Provider API"—rather than relying on the organization's name or the word "provider."

## Seamless Connect responsibilities

Unless an integration requires a different written agreement, Seamless Connect owns coordination across the administrative boundaries involved in an operation. Depending on the use case, this includes the Domain Owner, DNS Provider, Registrar, Parental Agent, Registry, and Service Provider.

- accepting registrar-originated DNSSEC intent and coordinating child-side readiness;
- observing CDS/CDNSKEY signals and coordinating Parental Agent or Registrar processing;
- verifying DS publication and the resulting DNSSEC state;
- analyzing Domain Connect templates and translating them into executable DNS changes where record automation is in scope;
- accepting standard-based and intent-based operations from authorized initiators and maintaining operation state;
- normalizing synchronous and asynchronous DNS Provider behavior behind one lifecycle;
- applying bounded retries and backoff to DNS Provider-classified retryable failures;
- performing DNS read-back verification and recording the result;
- normalizing DNS Provider statuses and safe error details;
- delivering current and final status to the initiating actor; and
- providing shared conformance fixtures and integration observability.

The DNS Provider remains the authority for child-zone access, DNS data, DNS Provider API behavior, DNS Provider policy, safety constraints, and the truth of DNS Provider-side execution. The Registrar carries or enforces the Domain Owner's authority over the domain delegation and remains the authority for registration-side changes. Seamless Connect must enforce each participant's published rules and must not claim capabilities absent from reviewed capability metadata.

## Initial DNSSEC pilot: registrar-originated

The initial DNSSEC lifecycle begins with the Domain Owner exercising domain-level authority through the Registrar. Seamless Connect then coordinates the DNS Provider and parent-side processing.

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
     │ request child-side readiness
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

The DNS Provider controls whether and how the child zone is signed. It does not decide independently that the Domain Owner wants the parent delegation changed. For the working pilot, the Registrar establishes that intent and Seamless Connect carries the workflow across the two control planes.

**DNS Provider work for the pilot:**

1. Accept or expose a supported request to make the child zone DNSSEC-ready, with appropriate Domain Owner authorization where direct DNS Provider access is required.
2. Sign the zone and publish standards-compliant CDS/CDNSKEY signaling.
3. Expose sufficient child-side status for Seamless Connect to distinguish pending, ready, failed, and action-required states.
4. Participate in conformance testing and a controlled production pilot.

The same child-side standards should also support a DNS-provider-originated lifecycle where the Domain Owner enables DNSSEC at the DNS Provider and an existing Parental Agent detects or reacts to CDS/CDNSKEY. Both initiation paths should converge on the same signing, signaling, policy, and verification behavior.

## Definition of ready for a pilot estimate

A DNS Provider integration is ready for a credible engineering estimate when:

- the initial use case, record types, actions, and operation mode are fixed;
- for DNSSEC, signing behavior, CDS/CDNSKEY support, and child-readiness semantics are known;
- Domain Owner authorization scopes and the DNS Provider sandbox flow are known;
- DNS Provider endpoints, error behavior, limits, and asynchronous semantics are documented;
- DNS Provider capability metadata and policy constraints have repository owners;
- DNS Provider discovery and account/zone mapping rules are testable;
- Registrar dependencies are identified and separated from DNS Provider responsibilities;
- operation, status, and verification mappings have acceptance criteria; and
- conformance, security, rollout, support, and pilot exit requirements are agreed.

## DNS Provider integration checklist

### 1. Choose the initial operation scope

<details>
<summary>Checklist items</summary>

- [ ] Select the smallest end-to-end use case for the pilot.
- [ ] For the working DNSSEC pilot, include registrar-originated enablement from authorized intent through verified DS publication.
- [ ] List the DNS record types and actions required by that use case.
- [ ] Decide whether each action is supported through a Domain Connect standard-based operation, a Seamless Connect intent-based operation, or both.
- [ ] Identify excluded operations explicitly so Service Providers do not infer support.
- [ ] Define DNS Provider account, zone, reseller, and domain-state restrictions that affect the scope.
- [ ] Identify whether the DNS Provider is also the Registrar and document any registration-specific dependencies separately from DNS-hosting dependencies.
- [ ] Document any supported DNS-provider-originated DNSSEC path without making it a prerequisite for the registrar-originated pilot.

</details>

**DNS Provider deliverable:** a reviewed capability declaration covering supported records, actions, operation modes, limits, and exclusions.

### 2. Provide delegated, scoped authorization where required

Where Seamless Connect directly invokes DNS Provider APIs, it needs narrow authority for the approved child-zone operation. A DNS Provider may require less direct Seamless API authority if an existing authorized workflow can prepare the zone and expose its state.

<details>
<summary>Checklist items</summary>

- [ ] Provide OAuth or an equivalent delegated authorization mechanism for the Domain Owner where direct API access is required.
- [ ] Define the minimum scopes needed for Seamless Connect to discover a zone, read records, and perform each supported mutation.
- [ ] For DNSSEC, limit mutation authority to the requested signing or child-readiness operation wherever the DNS Provider platform permits.
- [ ] Confirm token lifetime, refresh, revocation, and consent behavior.
- [ ] Avoid credentials that grant access beyond the authorized DNS Provider account, zone, or operation wherever the DNS Provider platform permits.
- [ ] Document DNS Provider authentication errors and reauthorization triggers.
- [ ] Document cases where an existing authorized DNS Provider workflow can proceed without direct Seamless API access.

</details>

**DNS Provider deliverable:** authorization configuration, scopes, test credentials or sandbox access, and documented token lifecycle behavior.

### 3. Confirm existing DNS Provider API endpoints and behavior

<details>
<summary>Checklist items</summary>

- [ ] Identify DNS Provider production and sandbox endpoints for zone lookup and DNS record read, create, update, and delete operations in scope.
- [ ] For DNSSEC, identify the existing API or workflow used to enable signing, read child-side status, and retrieve relevant DNSKEY, CDS, or CDNSKEY state.
- [ ] Document request and response formats, identifiers, pagination, rate limits, timeouts, idempotency behavior, and relevant headers.
- [ ] Provide the DNS Provider error model, including retryable errors, terminal errors, conflicts, authorization failures, and validation failures.
- [ ] Confirm DNS-specific semantics such as name normalization, apex representation, TTL defaults and limits, duplicate handling, and record-set replacement behavior.
- [ ] Confirm whether a successful DNS Provider API response means accepted, persisted, published, or globally observable.
- [ ] Distinguish acceptance of a DNSSEC-readiness request from completion of zone signing and publication of CDS/CDNSKEY.

</details>

**DNS Provider deliverable:** an API mapping package with endpoint references, examples, sandbox details, and known behavioral edge cases.

### 4. Publish DNS Provider policy and safety constraints

DNS Provider-specific behavior must be visible and reviewable rather than embedded as undocumented adapter logic.

<details>
<summary>Checklist items</summary>

- [ ] Document protected or DNS Provider-managed records.
- [ ] Document CNAME/apex rules, conflicting record rules, TTL limits, restricted record types, DNSSEC interactions, and any approval or DNS Provider account-state requirements.
- [ ] For DNSSEC, document supported signing and digest algorithms, key-management constraints, CDS/CDNSKEY publication behavior, rollover behavior, and disablement policy.
- [ ] Define whether Seamless Connect may replace, merge, or delete pre-existing records.
- [ ] Define the DNS Provider's required Domain Owner warnings, confirmations, and audit information.
- [ ] Identify constraints that Seamless Connect can enforce automatically and those that require Domain Owner or DNS Provider review.

</details>

**DNS Provider deliverable:** open, version-controlled policy files in this repository. DNS Provider capability metadata and DNS Provider-specific constraints should be maintained as separate, reviewable files, for example:

```text
dns-providers/<dns-provider-id>/
  capabilities.yaml
  constraints.md
```

Changes to these files should use the normal pull request review process so DNS Provider integrations and Service Providers can see when DNS Provider behavior changes. Do not place secrets, private endpoint details, or customer data in the repository.

### 5. Implement DNS Provider discovery

<details>
<summary>Checklist items</summary>

- [ ] Define how Seamless Connect determines which DNS Provider manages a domain.
- [ ] Prefer existing Domain Connect discovery when it applies; document any additional authoritative DNS signals or DNS Provider API lookup required.
- [ ] Define how the discovered domain maps to the correct DNS Provider account and zone.
- [ ] Handle delegated subdomains, reseller relationships, aliases, multiple matching zones, and domains not present in the authenticated DNS Provider account.
- [ ] Document any separate Registrar discovery needed when the Registrar and DNS Provider are different organizations.
- [ ] For registrar-originated DNSSEC, define how an authorized request maps to the correct DNS Provider account and child zone without conflating Registrar authority with DNS Provider account access.
- [ ] Return a clear outcome for found, not found, ambiguous, unauthorized, and temporarily unavailable.

</details>

**DNS Provider deliverable:** DNS Provider discovery rules, account/zone resolution behavior, Registrar dependencies if any, and test fixtures for positive and negative cases.

### 6. Support the Domain Owner authorization flow

<details>
<summary>Checklist items</summary>

- [ ] Define the handoff from the initiating actor through Seamless Connect to DNS Provider consent where separate DNS Provider authorization is required.
- [ ] For registrar-originated DNSSEC, define how the Registrar-established intent is correlated with any required DNS Provider authorization.
- [ ] Display the domain, requested changes, initiating actor, DNS Provider, and requested scopes before Domain Owner approval.
- [ ] Define redirect URI registration, state/nonce handling, session expiry, cancellation, and safe return behavior.
- [ ] Confirm reauthorization behavior for expired grants or materially changed requests.
- [ ] Ensure the Domain Owner's authorization decision is auditable by the DNS Provider and Seamless Connect as appropriate.

</details>

**DNS Provider deliverable:** a sandbox authorization journey with approved, denied, cancelled, expired, and DNS Provider account-mismatch cases.

### 7. Map Seamless Connect operations to DNS Provider APIs

<details>
<summary>Checklist items</summary>

- [ ] Map every supported standard-based or intent-based Seamless Connect operation to one or more DNS Provider API calls or documented workflows.
- [ ] For DNSSEC, map the child-readiness operation through signing and CDS/CDNSKEY publication without treating it as an immediate parent-side DS change.
- [ ] Define input normalization, validation, conflict detection, and idempotency rules.
- [ ] Define compensation or safe-stop behavior when a multi-call DNS Provider operation partially succeeds.
- [ ] Preserve enough DNS Provider response detail for diagnosis without exposing sensitive data to the initiating actor.
- [ ] Add mapping tests for each supported record type and action.

</details>

**DNS Provider deliverable:** reviewed Seamless Connect-to-DNS Provider operation mapping and test cases, including partial-failure behavior.

### 8. Return normalized execution status

<details>
<summary>Checklist items</summary>

- [ ] Map DNS Provider responses to the Seamless Connect status model.
- [ ] Distinguish at minimum: pending, in progress, succeeded, failed, needs authorization, needs Domain Owner action, conflict, and cancelled.
- [ ] For DNSSEC, distinguish request accepted, signing in progress, child ready, signaling published, and child-side failure; Seamless Connect separately tracks parental processing and DS publication.
- [ ] Provide stable DNS Provider operation or request identifiers for support and audit use.
- [ ] Classify DNS Provider errors as retryable or terminal and include a safe, actionable reason.
- [ ] Define which DNS Provider details Seamless Connect may return to the initiating actor.

</details>

**DNS Provider deliverable:** a DNS Provider-to-Seamless Connect status and error mapping with example responses.

### 9. Support read-back verification

<details>
<summary>Checklist items</summary>

- [ ] Provide a reliable DNS Provider API read path for the affected record set.
- [ ] For DNSSEC, permit verification that the zone is signed and that expected CDS/CDNSKEY records are authoritative.
- [ ] Define when the DNS Provider considers a change readable after mutation.
- [ ] Identify DNS Provider publication or DNS propagation delays and the signals that distinguish delay from failure.
- [ ] Supply fixtures for normalized names, default TTLs, DNS Provider-added values, and record-set ordering.
- [ ] Agree on the Seamless Connect verification result for exact match, acceptable normalized match, mismatch, and inconclusive state.
- [ ] Agree that child-side readiness and final parent DS publication are separate verification stages.

</details>

**DNS Provider deliverable:** verification behavior and expected results for every operation in the initial scope.

### 10. Define synchronous and asynchronous behavior

<details>
<summary>Checklist items</summary>

- [ ] Mark which DNS Provider API calls complete synchronously and which return accepted or pending work.
- [ ] For asynchronous DNS Provider work, provide polling, callback, or webhook behavior and authentication.
- [ ] Document expected completion times, polling limits, expiry, duplicate notifications, and out-of-order events.
- [ ] Define DNS Provider cancellation support and behavior after a Seamless Connect operation times out.
- [ ] Identify DNS Provider maintenance or queue conditions that change normal timing.

</details>

**DNS Provider deliverable:** asynchronous lifecycle documentation and test cases for completion, delay, failure, duplicate delivery, and timeout.

### 11. Pass conformance and security tests

<details>
<summary>Checklist items</summary>

- [ ] Run contract tests for DNS Provider discovery, Domain Owner authorization, operation mapping, status normalization, and verification.
- [ ] Test registrar-originated DNSSEC enablement, delayed signing, CDS/CDNSKEY publication, unacceptable signaling, and final DS verification.
- [ ] Test idempotent replay, partial failure, DNS Provider rate limiting, token expiry, revoked access, and concurrent DNS changes.
- [ ] Validate least-privilege scopes, redirect and callback security, secret handling, logging redaction, and audit events.
- [ ] Complete threat modeling for account/zone confusion, DNS takeover, record replacement, callback spoofing, and replay.
- [ ] Agree on DNS Provider and Seamless Connect operational contacts and severity/escalation handling before production access.

</details>

**DNS Provider deliverable:** passing conformance results, resolved security findings, and named DNS Provider operational contacts.

### 12. Run a limited pilot

The first DNSSEC pilot should use one Registrar, one or more DNS Providers, a controlled set of domains, and DNSSEC delegation operations only.

<details>
<summary>Checklist items</summary>

- [ ] Select the participating Registrar, DNS Providers, Domain Owners, domains, DNS Provider accounts, and DNSSEC operation types.
- [ ] Confirm registrar-originated initiation as the working pilot path and identify any existing DNS-provider-originated path that should remain active or be tested.
- [ ] Define success metrics, support coverage, rollback or disable controls, and pilot exit criteria.
- [ ] Instrument Domain Owner authorization completion, child readiness, CDS/CDNSKEY publication, parental processing, DS publication, verification, latency, retries, and Domain Owner-action rates.
- [ ] Review failures jointly and update DNS Provider capability or constraint files when observed DNS Provider behavior differs from documented behavior.
- [ ] Approve expansion only after conformance, security, reliability, and support criteria are met.

</details>

**DNS Provider deliverable:** pilot plan, dashboards or reports, issue review cadence, and a recorded go/no-go decision.
