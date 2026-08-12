# Loan Onboarding Platform — Discovery

**Status:** Baseline 0.2

**Last updated:** 2026-08-06

**Owner repository:** `loan-platform-architecture`

## Purpose

This directory is the discovery baseline for the Loan Onboarding & Credit Decisioning Platform. It records why the product exists, what the first release includes, and which assumptions constrain the design. Detailed domain and architecture documents remain separate sources of truth.

## Discovery package

| Document | Responsibility |
| --- | --- |
| [PRODUCT_VISION.md](PRODUCT_VISION.md) | Problem, users, value proposition, outcomes, and portfolio intent. |
| [SCOPE.md](SCOPE.md) | MVP boundaries, capabilities, quality expectations, and success criteria. |
| [ASSUMPTIONS.md](ASSUMPTIONS.md) | Validated decisions, hypotheses, constraints, risks, and unresolved questions. |

## Product summary

The platform demonstrates the end-to-end origination of a low-value unsecured personal loan through a progressive digital onboarding experience:

```text
Application
→ Identity verification
→ Credit assessment
→ Offer selection and acceptance
→ Document preparation
→ Electronic signature
→ Loan reservation
→ Disbursement
→ Loan activation
```

The product begins with minimal customer information, but identity, consent, affordability, and credit-risk controls must be completed before a favorable decision and offer can be produced.

## Current domain baseline

Eight business bounded contexts have been identified:

1. Application Process
2. Customer & Identity
3. Credit Decisioning
4. Document Preparation
5. Electronic Signature
6. Communications
7. Loan Booking
8. Disbursement

Audit is a cross-cutting event consumer, not a synchronous dependency or a ninth business context.

The canonical domain sources are:

- [Bounded Context Map and Event Storming](../domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md)
- [Ubiquitous Language](../domain/UBIQUITOUS_LANGUAGE.md)
- [Product and Credit Policy](../domain/PRODUCT_AND_CREDIT_POLICY.md)
- [Formal Credit Decision Table](../domain/CREDIT_DECISION_TABLE.md)
- [Credit Decisioning Design](../domain/CREDIT_DECISIONING_DESIGN.md)

## Confirmed solution direction

- Multi-repository delivery governed by a central architecture repository.
- Independent repository, model, data ownership, pipeline, and deployment lifecycle per microservice.
- Separate contracts repository for OpenAPI, AsyncAPI, and JSON Schema; no shared domain-model package.
- .NET 10 and AWS serverless services for the demonstration environment.
- Event-driven saga coordinated by Application Process.
- EventBridge for routing and SQS plus DLQ per consumer.
- DynamoDB ownership per service and S3 for document binaries.
- Local-first execution and an on-demand AWS demo environment.
- Spec-Driven Development with traceable contracts, rules, tests, and ADRs.

## Critical consistency decision

Money must not be transferred before the platform knows that the loan can be recorded. The fulfillment sequence is therefore:

```text
Signed document package
→ reserve Loan Account as PendingDisbursement
→ execute disbursement
→ confirm disbursement
→ activate Loan Account
→ complete Credit Application
```

## Current position

Product discovery is baselined and domain discovery is advanced. Credit Decisioning has a formal policy, decision table, and internal bounded-context design. The next repository-level work is the cross-cutting architecture baseline, followed by platform contracts and the first walking skeleton.

## Next deliverables

1. Create `BUSINESS_RULES.md` and `DOMAIN_EVENTS.md` as cross-context indexes.
2. Produce system context, container view, component catalog, data ownership, and security model.
3. Formalize the initial architecture decisions as ADRs.
4. Create the platform `README.md` and implementation roadmap.
5. Design the first versioned Credit Decisioning contracts.

## Governance

- English is the canonical language for code, contracts, events, and repository documentation.
- Domain terms must follow the Ubiquitous Language.
- A decision changes this baseline only through an updated source document and, when architectural, an ADR.
- Credit thresholds and prices are illustrative, versioned demo policy—not production or jurisdiction-specific financial advice.
