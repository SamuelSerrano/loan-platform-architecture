# MVP Scope

**Status:** Accepted baseline

**Last updated:** 2026-08-06

## MVP objective

Deliver a demonstrable vertical journey for one low-value unsecured personal-loan product, from application submission through loan activation, with explainable decisions and safe distributed processing.

## In scope

| Capability | MVP behavior |
| --- | --- |
| Application Process | Create and submit an application, expose its stage, coordinate the saga, and complete or terminate it. |
| Customer & Identity | Capture identity data and consent; verify identity through a deterministic fake provider. |
| Credit Decisioning | Evaluate eligibility, fraud signals, affordability, risk, and segment using versioned deterministic rules. |
| Offer | Calculate up to three alternatives, create one immutable selected offer, and communicate acceptance to Application Process. |
| Document Preparation | Generate a versioned document package from templates and record applicant approval. |
| Electronic Signature | Authorize signature after a valid OTP result and preserve evidence through a fake provider. |
| Communications | Issue and validate single-use OTP challenges and simulate notifications. |
| Loan Booking | Reserve a loan before disbursement, create a basic schedule, and activate it after confirmed disbursement. |
| Disbursement | Execute a simulated, idempotent disbursement and preserve the external reference. |
| Audit and observability | Consume relevant events, use structured logs, correlation identifiers, metrics, retry, and DLQ. |
| Delivery | Run locally by default and deploy an on-demand demo to AWS using IaC and CI/CD. |

## Demonstration product

- Product: `Quick Personal Loan`.
- Borrower: one adult individual.
- Type: unsecured consumer credit.
- Installments: monthly.
- Terms: 3, 6, 9, or 12 installments.
- One active application and one active loan per customer in the MVP.
- Amounts use neutral monetary units (`MU`).
- Policy parameters are illustrative and versioned.

The detailed product, affordability, risk, pricing, and offer rules are owned by [PRODUCT_AND_CREDIT_POLICY.md](../domain/PRODUCT_AND_CREDIT_POLICY.md) and [CREDIT_DECISION_TABLE.md](../domain/CREDIT_DECISION_TABLE.md).

## Initial user journeys

### Happy path

```text
Submit application
→ verify identity
→ record favorable decision
→ select and create offer
→ accept offer
→ generate and approve documents
→ validate OTP and sign
→ reserve loan
→ disburse
→ activate loan
→ complete application
```

### Required alternative paths

- Identity rejection.
- Unfavorable credit decision with reason codes.
- Evidence pending without a credit rejection.
- External-provider timeout and safe retry.
- Offer rejection or expiry.
- Signature expiry.
- Disbursement failure without activating the loan.
- Duplicate command or message replay without duplicate effects.

## Out of scope

- Real credit-bureau, biometric, core-banking, signature, SMS, or disbursement integrations.
- Real money movement or production customer data.
- Machine-learning scoring or automated model training.
- Country-specific regulatory certification or legal documents.
- Joint borrowers, guarantors, secured loans, or business lending.
- Multiple products, currencies, repayment frequencies, or dynamic product administration.
- Collections, repayments, refinancing, restructuring, delinquency, or loan cancellation after activation.
- Full case-management or back-office administration portal.
- Production multitenancy, high availability across regions, or disaster-recovery certification.
- Permanent public production environment.

## Quality scope

### Required for the MVP

- Domain logic independent from AWS SDKs and web frameworks.
- Unit, integration, contract, architecture, and selected end-to-end tests.
- Idempotent consumers and optimistic concurrency for aggregates.
- Transactional outbox/inbox or an equivalent atomic design for critical events.
- Retry with backoff, consumer-specific SQS queues, and DLQs.
- JWT authorization, least-privilege IAM, encryption in transit and at rest, and PII/OTP log redaction.
- Structured logs with `correlationId`, `causationId`, `applicationId`, and the appropriate aggregate identifiers.
- Reproducible local setup and AWS infrastructure as code.

### Deferred production qualities

- Formal penetration test, compliance audit, and threat-model sign-off.
- Multi-region failover and production recovery objectives.
- Production-grade manual-review operations.
- Live provider service-level objectives and commercial contracts.
- Advanced fraud detection and model governance.

## Walking skeleton boundary

The first executable slice stops after offer creation and covers:

```text
Submit application
→ fake identity verified
→ deterministic credit assessment
→ favorable offer or unfavorable outcome
```

It must also prove pending evidence, provider timeout, correlation, contract validation, and idempotent event consumption.

## Success criteria

- A developer can clone the relevant repositories and execute the walking skeleton locally from documented commands.
- The end-to-end happy path is reproducible using fictitious data.
- Each service owns its persistence and exposes versioned contracts.
- Favorable and unfavorable results are traceable to a policy version and evaluated rules.
- Operational dispositions never masquerade as credit declines.
- Message replay does not duplicate offers, document packages, signatures, disbursements, or loan accounts.
- The loan is reserved before disbursement and activated only after disbursement confirmation.
- AWS resources can be deployed and torn down on demand with budget safeguards.
- The architecture repository communicates the problem, trade-offs, component status, and implementation roadmap to a prospective client.

## Scope-change rule

A proposed capability enters the MVP only when it is necessary to demonstrate the selected journey or mitigate a material correctness, security, or recoverability risk. Every accepted scope change must update this document, the affected domain source, and any applicable ADR or contract.
