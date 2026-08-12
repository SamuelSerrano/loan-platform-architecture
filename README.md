# Loan Onboarding Platform Architecture

Architecture and discovery repository for an event-driven loan onboarding platform built with .NET and AWS.

## Purpose

This repository is the central source of truth for product discovery, domain language, business policy, bounded contexts, architectural decisions, and cross-service workflows. It intentionally contains no deployable service code.

The platform demonstrates a low-value unsecured personal-loan journey with progressive onboarding, explainable credit decisions, electronic signing, safe disbursement, and loan activation.

## Current status

**Discovery baseline:** 0.2 complete  
**Domain design:** in progress  
**Implementation:** not started

The next milestone is to formalize the architecture baseline before creating shared contracts and service repositories.

## Sources of truth

### Discovery

- [Discovery overview](docs/discovery/DISCOVERY.md)
- [Product vision](docs/discovery/PRODUCT_VISION.md)
- [MVP scope](docs/discovery/SCOPE.md)
- [Assumptions, constraints, and open questions](docs/discovery/ASSUMPTIONS.md)

### Domain

- [Ubiquitous language](docs/domain/UBIQUITOUS_LANGUAGE.md)
- [Bounded context map and Event Storming](docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md)
- [Product and credit policy](docs/domain/PRODUCT_AND_CREDIT_POLICY.md)
- [Formal credit decision table](docs/domain/CREDIT_DECISION_TABLE.md)
- [Credit Decisioning bounded-context design](docs/domain/CREDIT_DECISIONING_DESIGN.md)

## Repository strategy

This platform follows a multi-repository strategy. New repositories are created only after their boundaries and contracts are validated here.

Planned repositories include:

- `loan-platform-contracts`
- `loan-application-service`
- `customer-identity-service`
- `credit-decisioning-service`
- `document-service`
- `signature-service`
- `communications-service`
- `loan-account-service`
- `disbursement-service`
- `loan-platform-infrastructure`
- `loan-platform-demo-ui`

## Delivery sequence

1. Complete the domain baseline.
2. Formalize system architecture, data ownership, security, workflows, and ADRs.
3. Publish versioned integration contracts.
4. Create each service repository from an approved specification.
5. Implement vertical slices with Spec-Driven Development and TDD.

## Important notice

The financial parameters, rates, score thresholds, and policies in this repository are fictitious and intended for architecture demonstration only. They are not production underwriting rules or country-specific regulatory guidance.
