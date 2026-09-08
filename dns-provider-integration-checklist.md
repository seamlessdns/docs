# DNS Provider Integration Checklist

Use this checklist to scope and estimate a DNS Provider integration with Seamless Connect. It separates provider-owned work from responsibilities handled by Seamless Connect and is intended for a joint product and engineering review.

## Estimation summary

For each checklist item, record an owner, size, dependencies, risks, and acceptance criteria. Estimate the initial pilot scope first; defer record types and flows that are not required for that pilot.

| Workstream | Provider owner | Estimate | Dependencies or risks | Acceptance criteria agreed |
| --- | --- | --- | --- | --- |
| Operation scope and capabilities |  |  |  |  |
| Authorization |  |  |  |  |
| DNS API validation |  |  |  |  |
| Policy and safety constraints |  |  |  |  |
| Provider discovery |  |  |  |  |
| Operation mapping and status |  |  |  |  |
| Verification and async behavior |  |  |  |  |
| Testing and pilot |  |  |  |  |

## Provider integration checklist

### 1. Choose the initial operation scope

- [ ] Select the smallest end-to-end use case for the pilot.
- [ ] List the DNS record types and actions required by that use case.
- [ ] Decide whether each action is supported through a Domain Connect standard-based operation, a Seamless intent-based operation, or both.
- [ ] Identify excluded operations explicitly so Service Providers do not infer support.
- [ ] Define account, zone, reseller, and domain-state restrictions that affect the scope.

**Provider deliverable:** a reviewed capability declaration covering supported records, actions, operation modes, limits, and exclusions.

### 2. Provide delegated, scoped authorization

- [ ] Provide OAuth or an equivalent delegated authorization mechanism.
- [ ] Define the minimum scopes needed to discover a zone, read records, and perform each supported mutation.
- [ ] Confirm token lifetime, refresh, revocation, and consent behavior.
- [ ] Avoid credentials that grant access beyond the authorized account, zone, or operation wherever the provider platform permits.
- [ ] Document authentication errors and reauthorization triggers.

**Provider deliverable:** authorization configuration, scopes, test credentials or sandbox access, and documented token lifecycle behavior.

### 3. Confirm existing DNS API endpoints and behavior

- [ ] Identify production and sandbox endpoints for zone lookup and DNS record read, create, update, and delete operations in scope.
- [ ] Document request and response formats, identifiers, pagination, rate limits, timeouts, idempotency behavior, and relevant headers.
- [ ] Provide the error model, including retryable errors, terminal errors, conflicts, authorization failures, and validation failures.
- [ ] Confirm DNS-specific semantics such as name normalization, apex representation, TTL defaults and limits, duplicate handling, and record-set replacement behavior.
- [ ] Confirm whether a successful API response means accepted, persisted, published, or globally observable.

**Provider deliverable:** an API mapping package with endpoint references, examples, sandbox details, and known behavioral edge cases.

### 4. Publish provider policy and safety constraints

Provider-specific behavior must be visible and reviewable rather than embedded as undocumented adapter logic.

- [ ] Document protected or provider-managed records.
- [ ] Document CNAME/apex rules, conflicting record rules, TTL limits, restricted record types, DNSSEC interactions, and any approval or account-state requirements.
- [ ] Define whether Seamless may replace, merge, or delete pre-existing records.
- [ ] Define the provider's required user warnings, confirmations, and audit information.
- [ ] Identify constraints that can be machine-enforced and those that require human review.

**Provider deliverable:** open, version-controlled policy files in this repository. Capability metadata and provider-specific constraints should be maintained as separate, reviewable files, for example:

```text
providers/<provider>/
  capabilities.yaml
  constraints.md
```

Changes to these files should use the normal pull request review process so integrations and Service Providers can see when provider behavior changes. Do not place secrets, private endpoint details, or customer data in the repository.

### 5. Implement provider discovery

- [ ] Define how Seamless determines that the provider manages a domain.
- [ ] Prefer existing Domain Connect discovery when it applies; document any additional authoritative DNS signals or API lookup required.
- [ ] Define how the discovered domain maps to the correct account and zone.
- [ ] Handle delegated subdomains, resellers, aliases, multiple matching zones, and domains not present in the authenticated account.
- [ ] Return a clear outcome for found, not found, ambiguous, unauthorized, and temporarily unavailable.

**Provider deliverable:** discovery rules, account/zone resolution behavior, and test fixtures for positive and negative cases.

### 6. Support the domain-owner authorization flow

- [ ] Define the handoff from the Service Provider through Seamless to provider consent.
- [ ] Display the domain, requested changes, requesting Service Provider, and requested scopes before approval.
- [ ] Define redirect URI registration, state/nonce handling, session expiry, cancellation, and safe return behavior.
- [ ] Confirm reauthorization behavior for expired grants or materially changed requests.
- [ ] Ensure the authorization decision is auditable.

**Provider deliverable:** a sandbox authorization journey with approved, denied, cancelled, expired, and account-mismatch cases.

### 7. Map Seamless operations to provider APIs

- [ ] Map every supported standard-based or intent-based Seamless operation to one or more provider API calls.
- [ ] Define input normalization, validation, conflict detection, and idempotency rules.
- [ ] Define compensation or safe-stop behavior when a multi-call operation partially succeeds.
- [ ] Preserve enough provider response detail for diagnosis without exposing sensitive data to Service Providers.
- [ ] Add mapping tests for each supported record type and action.

**Provider deliverable:** reviewed operation mapping and test cases, including partial-failure behavior.

### 8. Return normalized execution status

- [ ] Map provider responses to the Seamless status model.
- [ ] Distinguish at minimum: pending, in progress, succeeded, failed, needs authorization, needs user action, conflict, and cancelled.
- [ ] Provide stable provider operation or request identifiers for support and audit use.
- [ ] Classify errors as retryable or terminal and include a safe, actionable reason.
- [ ] Define which provider details may be returned to the Service Provider.

**Provider deliverable:** a provider-to-Seamless status and error mapping with example responses.

### 9. Support read-back verification

- [ ] Provide a reliable API read path for the affected record set.
- [ ] Define when the provider considers a change readable after mutation.
- [ ] Identify propagation or publication delays and the signals that distinguish delay from failure.
- [ ] Supply fixtures for normalized names, default TTLs, provider-added values, and record-set ordering.
- [ ] Agree on the verification result for exact match, acceptable normalized match, mismatch, and inconclusive state.

**Provider deliverable:** verification behavior and expected results for every operation in the initial scope.

### 10. Define synchronous and asynchronous behavior

- [ ] Mark which provider calls complete synchronously and which return accepted/pending work.
- [ ] For asynchronous work, provide polling, callback, or webhook behavior and authentication.
- [ ] Document expected completion times, polling limits, expiry, duplicate notifications, and out-of-order events.
- [ ] Define cancellation support and behavior after an operation times out.
- [ ] Identify provider maintenance or queue conditions that change normal timing.

**Provider deliverable:** async lifecycle documentation and test cases for completion, delay, failure, duplicate delivery, and timeout.

### 11. Pass conformance and security tests

- [ ] Run contract tests for discovery, authorization, operation mapping, status normalization, and verification.
- [ ] Test idempotent replay, partial failure, rate limiting, token expiry, revoked access, and concurrent changes.
- [ ] Validate least-privilege scopes, redirect and callback security, secret handling, logging redaction, and audit events.
- [ ] Complete threat modeling for account/zone confusion, DNS takeover, record replacement, callback spoofing, and replay.
- [ ] Agree on operational contacts and severity/escalation handling before production access.

**Provider deliverable:** passing conformance results, resolved security findings, and named operational contacts.

### 12. Run a limited pilot

- [ ] Select a small number of Service Providers, domains, accounts, and operation types.
- [ ] Define success metrics, support coverage, rollback or disable controls, and pilot exit criteria.
- [ ] Instrument authorization completion, operation success, verification success, latency, retries, and user-action rates.
- [ ] Review failures jointly and update capability or constraint files when the observed provider behavior differs from the documented behavior.
- [ ] Approve expansion only after conformance, security, reliability, and support criteria are met.

**Provider deliverable:** pilot plan, dashboards or reports, issue review cadence, and a recorded go/no-go decision.

## Seamless Connect responsibilities

Unless an integration requires a different written agreement, Seamless Connect owns the cross-provider orchestration layer:

- analyzing Domain Connect templates and translating them into executable changes;
- accepting standard-based and intent-based operations and maintaining operation state;
- normalizing synchronous and asynchronous provider behavior behind one lifecycle;
- applying bounded retries and backoff to provider-classified retryable failures;
- performing read-back verification and recording the result;
- normalizing provider statuses and safe error details;
- delivering current and final status to the initiating Service Provider; and
- providing shared conformance fixtures and integration observability.

The DNS Provider remains the authority for authorization, DNS data, API behavior, provider policy, safety constraints, and the truth of provider-side execution. Seamless must enforce the provider's published rules and must not claim capabilities that are absent from the provider's reviewed capability metadata.

## Definition of ready for a pilot estimate

A provider integration is ready for a credible engineering estimate when:

- the initial use case, record types, actions, and operation mode are fixed;
- authorization scopes and the sandbox flow are known;
- endpoints, error behavior, limits, and async semantics are documented;
- capability metadata and policy constraints have repository owners;
- discovery and account/zone mapping rules are testable;
- operation, status, and verification mappings have acceptance criteria; and
- conformance, security, rollout, support, and pilot exit requirements are agreed.
