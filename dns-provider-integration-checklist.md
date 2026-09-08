# DNS Provider Integration Checklist

Use this checklist to scope and estimate a DNS Provider integration with Seamless Connect. It separates DNS Provider-owned work from responsibilities handled by Seamless Connect and is intended for a joint product and engineering review.

## Terminology and roles

This document names the actor responsible for each action. Do not use the unqualified term "provider" in integration requirements or implementation notes.

- **Service Provider (SP):** the application or service that initiates a domain configuration request for its customer and receives status from Seamless Connect.
- **DNS Provider:** the operator of the authoritative DNS service and DNS management API used to read or change the domain's DNS records.
- **Registrar:** the organization sponsoring the domain registration. A Registrar may also act as the DNS Provider, but registration and DNS-hosting responsibilities remain distinct.
- **Domain Owner:** the registrant or authorized user who approves access and DNS changes.
- **Seamless Connect:** the open orchestration layer between a Service Provider and a DNS Provider.

One organization may perform more than one role. Name the role being performed—for example, "Registrar discovery" or "DNS Provider API"—rather than relying on the organization's name or the word "provider."

## Estimation summary

For each checklist item, record an owner, size, dependencies, risks, and acceptance criteria. Estimate the initial pilot scope first; defer record types and flows that are not required for that pilot.

| Workstream | DNS Provider owner | Estimate | Dependencies or risks | Acceptance criteria agreed |
| --- | --- | --- | --- | --- |
| Operation scope and capabilities |  |  |  |  |
| Authorization |  |  |  |  |
| DNS API validation |  |  |  |  |
| Policy and safety constraints |  |  |  |  |
| DNS Provider discovery |  |  |  |  |
| Operation mapping and status |  |  |  |  |
| Verification and async behavior |  |  |  |  |
| Testing and pilot |  |  |  |  |

## DNS Provider integration checklist

### 1. Choose the initial operation scope

- [ ] Select the smallest end-to-end use case for the pilot.
- [ ] List the DNS record types and actions required by that use case.
- [ ] Decide whether each action is supported through a Domain Connect standard-based operation, a Seamless Connect intent-based operation, or both.
- [ ] Identify excluded operations explicitly so Service Providers do not infer support.
- [ ] Define DNS Provider account, zone, reseller, and domain-state restrictions that affect the scope.
- [ ] Identify whether the DNS Provider is also the Registrar and document any registration-specific dependencies separately from DNS-hosting dependencies.

**DNS Provider deliverable:** a reviewed capability declaration covering supported records, actions, operation modes, limits, and exclusions.

### 2. Provide delegated, scoped authorization

- [ ] Provide OAuth or an equivalent delegated authorization mechanism for the Domain Owner.
- [ ] Define the minimum scopes needed for Seamless Connect to discover a zone, read records, and perform each supported mutation.
- [ ] Confirm token lifetime, refresh, revocation, and consent behavior.
- [ ] Avoid credentials that grant access beyond the authorized DNS Provider account, zone, or operation wherever the DNS Provider platform permits.
- [ ] Document DNS Provider authentication errors and reauthorization triggers.

**DNS Provider deliverable:** authorization configuration, scopes, test credentials or sandbox access, and documented token lifecycle behavior.

### 3. Confirm existing DNS Provider API endpoints and behavior

- [ ] Identify DNS Provider production and sandbox endpoints for zone lookup and DNS record read, create, update, and delete operations in scope.
- [ ] Document request and response formats, identifiers, pagination, rate limits, timeouts, idempotency behavior, and relevant headers.
- [ ] Provide the DNS Provider error model, including retryable errors, terminal errors, conflicts, authorization failures, and validation failures.
- [ ] Confirm DNS-specific semantics such as name normalization, apex representation, TTL defaults and limits, duplicate handling, and record-set replacement behavior.
- [ ] Confirm whether a successful DNS Provider API response means accepted, persisted, published, or globally observable.

**DNS Provider deliverable:** an API mapping package with endpoint references, examples, sandbox details, and known behavioral edge cases.

### 4. Publish DNS Provider policy and safety constraints

DNS Provider-specific behavior must be visible and reviewable rather than embedded as undocumented adapter logic.

- [ ] Document protected or DNS Provider-managed records.
- [ ] Document CNAME/apex rules, conflicting record rules, TTL limits, restricted record types, DNSSEC interactions, and any approval or DNS Provider account-state requirements.
- [ ] Define whether Seamless Connect may replace, merge, or delete pre-existing records.
- [ ] Define the DNS Provider's required Domain Owner warnings, confirmations, and audit information.
- [ ] Identify constraints that Seamless Connect can enforce automatically and those that require Domain Owner or DNS Provider review.

**DNS Provider deliverable:** open, version-controlled policy files in this repository. DNS Provider capability metadata and DNS Provider-specific constraints should be maintained as separate, reviewable files, for example:

```text
dns-providers/<dns-provider-id>/
  capabilities.yaml
  constraints.md
```

Changes to these files should use the normal pull request review process so DNS Provider integrations and Service Providers can see when DNS Provider behavior changes. Do not place secrets, private endpoint details, or customer data in the repository.

### 5. Implement DNS Provider discovery

- [ ] Define how Seamless Connect determines which DNS Provider manages a domain.
- [ ] Prefer existing Domain Connect discovery when it applies; document any additional authoritative DNS signals or DNS Provider API lookup required.
- [ ] Define how the discovered domain maps to the correct DNS Provider account and zone.
- [ ] Handle delegated subdomains, reseller relationships, aliases, multiple matching zones, and domains not present in the authenticated DNS Provider account.
- [ ] Document any separate Registrar discovery needed when the Registrar and DNS Provider are different organizations.
- [ ] Return a clear outcome for found, not found, ambiguous, unauthorized, and temporarily unavailable.

**DNS Provider deliverable:** DNS Provider discovery rules, account/zone resolution behavior, Registrar dependencies if any, and test fixtures for positive and negative cases.

### 6. Support the Domain Owner authorization flow

- [ ] Define the handoff from the Service Provider through Seamless Connect to DNS Provider consent.
- [ ] Display the domain, requested changes, requesting Service Provider, DNS Provider, and requested scopes before Domain Owner approval.
- [ ] Define redirect URI registration, state/nonce handling, session expiry, cancellation, and safe return behavior.
- [ ] Confirm reauthorization behavior for expired grants or materially changed requests.
- [ ] Ensure the Domain Owner's authorization decision is auditable by the DNS Provider and Seamless Connect as appropriate.

**DNS Provider deliverable:** a sandbox authorization journey with approved, denied, cancelled, expired, and DNS Provider account-mismatch cases.

### 7. Map Seamless Connect operations to DNS Provider APIs

- [ ] Map every supported standard-based or intent-based Seamless Connect operation to one or more DNS Provider API calls.
- [ ] Define input normalization, validation, conflict detection, and idempotency rules.
- [ ] Define compensation or safe-stop behavior when a multi-call DNS Provider operation partially succeeds.
- [ ] Preserve enough DNS Provider response detail for diagnosis without exposing sensitive data to Service Providers.
- [ ] Add mapping tests for each supported record type and action.

**DNS Provider deliverable:** reviewed Seamless Connect-to-DNS Provider operation mapping and test cases, including partial-failure behavior.

### 8. Return normalized execution status

- [ ] Map DNS Provider responses to the Seamless Connect status model.
- [ ] Distinguish at minimum: pending, in progress, succeeded, failed, needs authorization, needs Domain Owner action, conflict, and cancelled.
- [ ] Provide stable DNS Provider operation or request identifiers for support and audit use.
- [ ] Classify DNS Provider errors as retryable or terminal and include a safe, actionable reason.
- [ ] Define which DNS Provider details Seamless Connect may return to the Service Provider.

**DNS Provider deliverable:** a DNS Provider-to-Seamless Connect status and error mapping with example responses.

### 9. Support read-back verification

- [ ] Provide a reliable DNS Provider API read path for the affected record set.
- [ ] Define when the DNS Provider considers a change readable after mutation.
- [ ] Identify DNS Provider publication or DNS propagation delays and the signals that distinguish delay from failure.
- [ ] Supply fixtures for normalized names, default TTLs, DNS Provider-added values, and record-set ordering.
- [ ] Agree on the Seamless Connect verification result for exact match, acceptable normalized match, mismatch, and inconclusive state.

**DNS Provider deliverable:** verification behavior and expected results for every operation in the initial scope.

### 10. Define synchronous and asynchronous behavior

- [ ] Mark which DNS Provider API calls complete synchronously and which return accepted or pending work.
- [ ] For asynchronous DNS Provider work, provide polling, callback, or webhook behavior and authentication.
- [ ] Document expected completion times, polling limits, expiry, duplicate notifications, and out-of-order events.
- [ ] Define DNS Provider cancellation support and behavior after a Seamless Connect operation times out.
- [ ] Identify DNS Provider maintenance or queue conditions that change normal timing.

**DNS Provider deliverable:** asynchronous lifecycle documentation and test cases for completion, delay, failure, duplicate delivery, and timeout.

### 11. Pass conformance and security tests

- [ ] Run contract tests for DNS Provider discovery, Domain Owner authorization, operation mapping, status normalization, and verification.
- [ ] Test idempotent replay, partial failure, DNS Provider rate limiting, token expiry, revoked access, and concurrent DNS changes.
- [ ] Validate least-privilege scopes, redirect and callback security, secret handling, logging redaction, and audit events.
- [ ] Complete threat modeling for account/zone confusion, DNS takeover, record replacement, callback spoofing, and replay.
- [ ] Agree on DNS Provider and Seamless Connect operational contacts and severity/escalation handling before production access.

**DNS Provider deliverable:** passing conformance results, resolved security findings, and named DNS Provider operational contacts.

### 12. Run a limited pilot

- [ ] Select a small number of Service Providers, Domain Owners, domains, DNS Provider accounts, and operation types.
- [ ] Define success metrics, support coverage, rollback or disable controls, and pilot exit criteria.
- [ ] Instrument Domain Owner authorization completion, DNS Provider operation success, verification success, latency, retries, and Domain Owner-action rates.
- [ ] Review failures jointly and update DNS Provider capability or constraint files when observed DNS Provider behavior differs from documented behavior.
- [ ] Approve expansion only after conformance, security, reliability, and support criteria are met.

**DNS Provider deliverable:** pilot plan, dashboards or reports, issue review cadence, and a recorded go/no-go decision.

## Seamless Connect responsibilities

Unless an integration requires a different written agreement, Seamless Connect owns orchestration across Service Providers and DNS Providers:

- analyzing Domain Connect templates and translating them into executable DNS changes;
- accepting standard-based and intent-based operations from Service Providers and maintaining operation state;
- normalizing synchronous and asynchronous DNS Provider behavior behind one lifecycle;
- applying bounded retries and backoff to DNS Provider-classified retryable failures;
- performing DNS read-back verification and recording the result;
- normalizing DNS Provider statuses and safe error details;
- delivering current and final status to the initiating Service Provider; and
- providing shared conformance fixtures and integration observability.

The DNS Provider remains the authority for Domain Owner authorization, DNS data, DNS Provider API behavior, DNS Provider policy, safety constraints, and the truth of DNS Provider-side execution. The Registrar remains the authority for domain registration data and registration operations when those are in scope. Seamless Connect must enforce the DNS Provider's published rules and must not claim capabilities absent from the DNS Provider's reviewed capability metadata.

## Definition of ready for a pilot estimate

A DNS Provider integration is ready for a credible engineering estimate when:

- the initial use case, record types, actions, and operation mode are fixed;
- Domain Owner authorization scopes and the DNS Provider sandbox flow are known;
- DNS Provider endpoints, error behavior, limits, and asynchronous semantics are documented;
- DNS Provider capability metadata and policy constraints have repository owners;
- DNS Provider discovery and account/zone mapping rules are testable;
- Registrar dependencies are identified and separated from DNS Provider responsibilities;
- operation, status, and verification mappings have acceptance criteria; and
- conformance, security, rollout, support, and pilot exit requirements are agreed.
