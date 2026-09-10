# Service Provider Integration Checklist

Use this checklist to scope and estimate a Service Provider integration with Seamless Connect. The goal is to keep Service Provider integration as small as possible: Seamless Connect handles the cross-provider complexity required to turn a Service Provider request into a verified domain operation.

Two integration paths are supported:

1. **Domain Connect path** — for existing standardized DNS configuration flows.
2. **Seamless Connect Operation API** — for operations that are not covered by an existing standard or require more advanced application behavior.

Service Providers may support either or both.

## Terminology and roles

This document names the actor responsible for each action. Do not use the unqualified term "provider" in integration requirements or implementation notes.

- **Service Provider (SP):** the application or service that requests a domain operation for its customer and receives status from Seamless Connect.
- **DNS Provider:** the operator of the authoritative DNS service and DNS management API used to read or change DNS records.
- **Registrar:** the organization sponsoring the domain registration. A Registrar may also act as the DNS Provider, but registration and DNS-hosting responsibilities remain distinct.
- **Domain Owner:** the registrant or authorized user approving access or changes.
- **Seamless Connect:** the open orchestration layer between Service Providers and domain infrastructure providers.

One organization may perform more than one role. Name the role being performed rather than relying on the organization's name or the word "provider."

## What Seamless Connect handles

All integrations use one consistent, openly documented integration contract. Within that contract, Seamless Connect owns the complexity between the Service Provider request and the infrastructure provider executing it:

- DNS Provider discovery;
- Domain Connect template retrieval, validation, and approval;
- DNS Provider capability and policy evaluation;
- Domain Owner authorization handoff;
- translation of requests into DNS Provider-specific API operations;
- synchronous and asynchronous execution;
- operation state;
- bounded retries and backoff;
- provider-specific error normalization;
- read-back and outcome verification;
- status and completion delivery; and
- shared conformance and integration observability.

The Service Provider should not need to implement separate integrations for each supported DNS Provider.

Provider-specific capabilities and constraints are documented openly in this repository and handled through the common contract, not through private bilateral exceptions.

## Minimum integration

For many Service Providers, the practical minimum should be:

### Existing Domain Connect implementation

```text
Existing Domain Connect request
              │
              │ change target
              ▼
       Seamless Connect
              │
              ▼
          DNS Provider
```

**Service Provider work:**

1. Point existing Domain Connect requests at Seamless Connect.
2. Publish or identify the existing template.
3. Handle Domain Owner return/authorization.
4. Receive completion status.

### Advanced application integration

```text
Service Provider
       │
       │ defined API operation
       ▼
Seamless Connect
       │
       │ discovery
       │ authorization
       │ provider translation
       │ sync / async execution
       │ retries
       │ verification
       ▼
DNS / Domain Infrastructure
```

**Service Provider work:**

1. Call a published Seamless Connect operation.
2. Supply the required inputs.
3. Send the Domain Owner through authorization when requested.
4. Track the operation ID.
5. Receive the normalized result.

## Integration principle

A Service Provider should describe **the standardized request or defined operation it wants performed**, not implement the provider-specific machinery required to perform it.

**Use existing open standards wherever they cover the use case.**

Where no applicable standard exists, Seamless Connect may provide an open, deterministic operation interface to prove the use case, with the goal of developing or adopting interoperable standards as the ecosystem matures.

The complexity of provider discovery, provider differences, authorization routing, execution timing, retries, and verification belongs in Seamless Connect—not in every Service Provider integration.

## Integration path A: Domain Connect

### 1. Reuse the existing Domain Connect implementation

For Service Providers that already generate Domain Connect requests, Seamless Connect should require the smallest practical integration change.

<details>
<summary>Checklist items</summary>

- [ ] Continue using the existing Domain Connect template and parameter model.
- [ ] Continue generating requests according to the applicable Domain Connect standard.
- [ ] Change the request target from a DNS Provider-specific Domain Connect endpoint to the Seamless Connect endpoint.
- [ ] Identify the Service Provider/application making the request.
- [ ] Provide the domain and standard Domain Connect parameters required by the template.
- [ ] Handle the return or completion response from Seamless Connect.

</details>

**Service Provider deliverable:** an existing or new Domain Connect request successfully submitted through Seamless Connect.

For an existing Domain Connect implementation, integration with Seamless Connect should ideally amount to:

**same request → new target**

Seamless Connect then handles DNS Provider discovery, template validation, authorization routing, execution, and verification.

### 2. Publish the Domain Connect template

The Service Provider should be able to maintain its template as part of its normal development workflow.

<details>
<summary>Checklist items</summary>

- [ ] Publish the Domain Connect template at a stable public location or supported source repository.
- [ ] Version template changes using the Service Provider's normal source-control process.
- [ ] Ensure required parameters and records conform to the Domain Connect specification.
- [ ] Provide Seamless Connect with the template location or identifier.

</details>

Seamless Connect is responsible for analyzing the template, applying conformance checks and DNS Provider policies, and determining whether it can be executed by the selected DNS Provider.

**Service Provider deliverable:** a publicly retrievable Domain Connect template and example parameter set.

## Integration path B: Seamless Connect Operation API

Use this path for domain operations that are not adequately represented by an existing standard or where an application needs more direct API-style interaction.

This interface should remain deterministic and explicit. It is not intended to allow an AI system or Seamless Connect to invent privileged DNS operations.

### 3. Choose the operation

The Service Provider identifies a defined Seamless Connect operation and supplies its required inputs.

Examples may eventually include:

- DNS record read;
- DNS record create;
- DNS record update;
- DNS record delete;
- domain connection;
- DNSSEC enablement;
- nameserver changes;
- registration;
- transfer; and
- other operations adopted by the Seamless Connect project.

<details>
<summary>Checklist items</summary>

- [ ] Select the defined operation required by the application.
- [ ] Supply the domain or domain resource being acted upon.
- [ ] Supply the minimum operation-specific inputs.
- [ ] Identify the Domain Owner or application session initiating the operation.
- [ ] Do not encode DNS Provider-specific behavior unless explicitly required by the operation.

</details>

**Service Provider deliverable:** a valid operation request using the published Seamless Connect API schema.

Example:

```http
POST /v1/operations
```

```json
{
  "type": "dns.records.create",
  "domain": "example.com",
  "records": [
    {
      "type": "CNAME",
      "name": "www",
      "value": "app.example-service.com"
    }
  ]
}
```

The Service Provider specifies the operation. Seamless Connect determines how that operation is safely executed against the authoritative DNS Provider.

### 4. Support normal application CRUD patterns where applicable

Advanced Service Providers should be able to integrate Seamless Connect similarly to a conventional API rather than learning every DNS Provider's API.

Where supported, the Service Provider may use defined read and mutation operations such as:

- **GET** — inspect domain/DNS state exposed through Seamless Connect;
- **POST** — create a resource or start an operation;
- **PUT** — replace a supported resource or desired configuration;
- **PATCH** — modify supported properties; and
- **DELETE** — remove an explicitly identified resource.

The Seamless Connect API defines the semantics of these operations. DNS Provider-specific API behavior remains hidden behind Seamless Connect adapters.

<details>
<summary>Checklist items</summary>

- [ ] Use published Seamless Connect resource and operation schemas.
- [ ] Supply stable identifiers when modifying or deleting existing resources.
- [ ] Handle validation or conflict responses returned by Seamless Connect.
- [ ] Use idempotency controls for mutations where supported.
- [ ] Do not assume the underlying DNS Provider implements the same HTTP method or resource model.

</details>

**Service Provider deliverable:** request/response integration against the Seamless Connect API rather than individual DNS Provider APIs.

## Shared requirements

### 5. Start the Domain Owner authorization flow

The Service Provider initiates the operation. Seamless Connect and the DNS Provider handle the infrastructure authorization path.

<details>
<summary>Checklist items</summary>

- [ ] Associate the operation with the correct Domain Owner session.
- [ ] Redirect or present the Domain Owner authorization URL supplied by Seamless Connect when authorization is required.
- [ ] Preserve application return context.
- [ ] Handle approval, denial, cancellation, and expiration.
- [ ] Do not collect or store DNS Provider credentials unless a separate architecture explicitly requires it.

</details>

The DNS Provider remains authoritative for access to its account and DNS resources.

**Service Provider deliverable:** the ability to send the Domain Owner into authorization and resume the application flow afterward.

### 6. Handle the Seamless Connect operation lifecycle

Every Seamless Connect request that requires execution may be represented as an operation. An operation can finish immediately or continue asynchronously.

For example:

```text
accepted
→ awaiting_authorization
→ executing
→ verifying
→ completed
```

or:

```text
accepted
→ executing
→ failed
```

<details>
<summary>Checklist items</summary>

- [ ] Store or associate the Seamless Connect operation ID with the application/customer workflow.
- [ ] Handle an immediately completed operation.
- [ ] Handle an operation that remains pending after the original request returns.
- [ ] Display appropriate progress or pending state to the user.
- [ ] Treat Seamless Connect's terminal status as the application-level completion signal.

</details>

The Service Provider does not need to reproduce DNS Provider polling, retries, propagation handling, or provider-specific asynchronous behavior.

**Service Provider deliverable:** basic application handling for pending, completed, failed, cancelled, or Domain Owner-action-required operations.

### 7. Choose a completion mechanism

Service Providers should be able to use whichever integration style best fits their application.

Supported patterns may include:

- immediate response when the operation completes synchronously;
- polling an operation status endpoint;
- webhook notification; or
- event-based delivery as supported by Seamless Connect.

<details>
<summary>Checklist items</summary>

- [ ] Choose polling, webhook/event delivery, or both for asynchronous completion.
- [ ] Authenticate and validate completion notifications.
- [ ] Handle duplicate notifications idempotently.
- [ ] Retrieve the final operation result when required.

</details>

**Service Provider deliverable:** one supported mechanism for receiving final operation status.

### 8. Handle normalized errors

The Service Provider should not need to understand every DNS Provider's native error model. Seamless Connect normalizes provider responses into stable application-facing error classes.

Examples may include:

- invalid request;
- unsupported operation;
- authorization required;
- authorization denied;
- domain or zone not found;
- conflict;
- provider unavailable;
- retry in progress;
- operation failed; and
- verification failed.

<details>
<summary>Checklist items</summary>

- [ ] Map Seamless Connect error classes to the Service Provider's application UX.
- [ ] Preserve the Seamless Connect operation ID for support/debugging.
- [ ] Present Domain Owner-actionable errors when appropriate.
- [ ] Do not depend on provider-specific error text.

</details>

**Service Provider deliverable:** application handling for the published Seamless Connect error model.

### 9. Test against Seamless Connect conformance fixtures

Service Providers should be able to validate their integration without building test accounts across every supported DNS Provider.

<details>
<summary>Checklist items</summary>

- [ ] Test a successful operation.
- [ ] Test authorization approval and denial.
- [ ] Test synchronous completion.
- [ ] Test asynchronous completion.
- [ ] Test conflict and validation failures.
- [ ] Test retries and delayed provider execution from the Service Provider's perspective.
- [ ] Test duplicate webhook/event delivery if asynchronous callbacks are used.
- [ ] Confirm that an operation ID can be correlated through application logs.

</details>

**Service Provider deliverable:** passing Service Provider integration/conformance tests.

### 10. Run a limited production pilot

Start with a narrow use case and expand after measuring results.

<details>
<summary>Checklist items</summary>

- [ ] Choose one initial domain workflow.
- [ ] Enable the integration for a limited set of customers or traffic if appropriate.
- [ ] Track operation attempts, authorization completion, success rate, failures, and completion time.
- [ ] Compare support burden and onboarding completion with the previous workflow.
- [ ] Review unsupported or problematic operations with the Seamless Connect community.
- [ ] Expand after reliability and user experience meet agreed criteria.

</details>

**Service Provider deliverable:** a production pilot with measurable onboarding and reliability results.
