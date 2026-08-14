# Assumptions, Constraints, and Open Questions

**Status:** Active register

**Last updated:** 2026-08-14

## How to use this register

- **Decision**: agreed and treated as a baseline; architectural decisions will also receive an ADR.
- **Assumption**: believed true for the MVP but must be validated if the project approaches production use.
- **Constraint**: intentionally limits the solution.
- **Open question**: unresolved and not to be silently encoded in implementation.

## Confirmed product decisions

| ID | Type | Statement | Consequence |
| --- | --- | --- | --- |
| P-001 | Decision | The MVP offers one `Quick Personal Loan` to one adult individual. | No product catalog, joint application, guarantor, or secured-loan model is required. |
| P-002 | Decision | The domain is financially neutral and canonical documentation is in English. | Amounts use `MU`; country-specific regulation remains outside scope. |
| P-003 | Decision | Entry is progressive and requires little information to start. | Deeper identity, consent, income, obligation, and risk controls occur before an offer. |
| P-004 | Decision | Credit outcomes are `Favorable` or `Unfavorable`. | Pending evidence, retry, and technical exception are operational dispositions, not declines. |
| P-005 | Decision | A favorable decision produces alternatives; selecting one creates a single immutable `CreditOffer`. Application Process owns the applicant's acceptance or explicit rejection of the active offer, including the referenced `offerId`, exact canonical `termsHash`, action timestamp, stage history, and idempotency. | Credit Decisioning retains ownership of the immutable offer, canonical terms, creation, expiry, and supersession. Application Process records the journey action without mutating or redefining offer terms. |
| P-006 | Decision | A person becomes a `Borrower` only after the loan account is activated. | `Applicant`, `Customer`, `Signer`, and `Borrower` are not interchangeable terms. |

## Confirmed architecture decisions awaiting ADRs

| ID | Type | Statement | Planned ADR |
| --- | --- | --- | --- |
| A-001 | Decision | Use a multi-repository model with a central architecture repository and a separate contracts repository. | ADR-001 |
| A-002 | Decision | Use .NET 10 on AWS serverless services for the demo environment. | ADR-002 |
| A-003 | Decision | Coordinate the onboarding as an event-driven saga owned by Application Process. | ADR-003 |
| A-004 | Decision | Route integration events through EventBridge and deliver them through SQS queues with a DLQ per consumer. | ADR-003 |
| A-005 | Decision | Each microservice owns its persistence; DynamoDB is the default operational store. | ADR-004 |
| A-006 | Decision | Public contracts live as OpenAPI, AsyncAPI, and JSON Schema—not as shared C# domain types. | Future contract-governance ADR |
| A-007 | Decision | Reserve the loan before disbursement and activate it only after confirmed disbursement. | Future fulfillment-consistency ADR |
| A-008 | Decision | Audit consumes events asynchronously and is not a synchronous dependency. | Future audit ADR |

## Delivery constraints

| ID | Type | Constraint |
| --- | --- | --- |
| C-001 | Constraint | Local execution is the default; AWS is an on-demand demonstration environment. |
| C-002 | Constraint | Avoid persistent-cost infrastructure such as EKS, NAT Gateway, always-on RDS, MSK, and OpenSearch in the MVP. |
| C-003 | Constraint | Use fictitious data and deterministic fake providers by default. |
| C-004 | Constraint | No AWS credentials, secrets, real OTPs, or production PII may be committed or logged. |
| C-005 | Constraint | Each repository must be independently understandable, testable, and deployable. |
| C-006 | Constraint | Domain code must not depend directly on Lambda, DynamoDB, EventBridge, or HTTP frameworks. |

## Working assumptions requiring later validation

| ID | Type | Assumption | Validation trigger |
| --- | --- | --- | --- |
| H-001 | Assumption | Monthly installments and terms of 3, 6, 9, or 12 are sufficient to demonstrate the product. | Before adding a second product or repayment frequency. |
| H-002 | Assumption | A deterministic score between 0 and 1,000 is adequate for an explainable portfolio demo. | Before integrating a real risk provider or model. |
| H-003 | Assumption | One active application and one active loan per customer simplifies the MVP without hiding the core architecture. | Before repeat-borrower or parallel-application scenarios. |
| H-004 | Assumption | EventBridge plus SQS provides adequate routing, retry isolation, and cost characteristics for demo load. | During load testing or cost review. |
| H-005 | Assumption | DynamoDB transactions and an outbox/inbox pattern can satisfy aggregate and event consistency needs. | During detailed persistence design of each service. |
| H-006 | Assumption | A minimal React portal plus API documentation will communicate the journey better than API-only demonstration. | Before implementing the demo UI. |
| H-007 | Assumption | Simulated providers can reproduce success, decline, timeout, duplicate, and inconsistent-response scenarios. | Before the walking skeleton acceptance test. |

## Risks and mitigations

| Risk | Impact | Current mitigation |
| --- | --- | --- |
| Too many repositories before boundaries stabilize | Empty or artificial services and high maintenance overhead. | Create repositories incrementally after context and contract readiness. |
| Application Process becomes a god service | Domain logic and provider details leak into the saga coordinator. | Keep it responsible for process state and policies only; capabilities own their decisions. |
| Shared contracts become a shared domain model | Services become coupled through common C# types. | Publish language-neutral contracts and let each service map them locally. |
| Technical failure is interpreted as credit decline | Unfair outcome and incorrect analytics. | Preserve operational dispositions separately from immutable credit decisions. |
| Money moves without a recorded obligation | Critical financial inconsistency. | Reserve first, disburse second, activate after confirmation. |
| Portfolio demo creates uncontrolled AWS charges | Avoidable cost. | Local-first execution, budgets, concurrency caps, short retention, and automated teardown. |
| Demo policy is mistaken for real underwriting | Misleading or unsafe reuse. | Label parameters as fictitious and versioned; require independent legal/risk validation for production. |

## Open questions

These questions are intentionally deferred and must not be guessed in code:

| ID | Question | Needed before |
| --- | --- | --- |
| Q-001 | Which exact UI scope proves the journey: React portal plus Swagger, or React plus a dedicated operator view? | Demo UI specification. |
| Q-002 | Which AWS region and account safeguards will be used for the on-demand demo? | Infrastructure implementation. |
| Q-003 | What retention periods apply to demo applications, logs, documents, OTP records, audit entries, and DLQs? | Security model and IaC. |
| Q-004 | Which event payload fields contain PII, and what tokenization or minimization rules apply? | Contract publication. |
| Q-005 | Should Application Process use pure event choreography with process policies or persist explicit saga steps and timers? | Application Process design ADR. |
| Q-006 | What is the exact expiration and renewal policy for identity verification across reassessment? | Customer & Identity specification. |
| Q-007 | What are the first-installment date and rounding conventions for the repayment schedule? | Loan Booking specification. |
| Q-008 | Which applicant-facing reason codes may be shown directly and which require a customer-safe translation? | API/UI contract and communications design. |
| Q-009 | What manual recovery action is permitted after repeated disbursement failure? | Disbursement workflow specification. |

## Review rule

Review this register at each architecture baseline, before publishing contracts, and before starting a new service repository. Close an open question by updating its owning source document and recording an ADR when the answer has architectural consequences.
