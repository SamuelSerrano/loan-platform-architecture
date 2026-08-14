# Loan Onboarding Platform Architecture

Architecture and discovery repository for an event-driven loan onboarding platform built with .NET and AWS.

## Purpose

This repository is the central source of truth for product discovery, domain language, business policy, bounded contexts, architectural decisions, and cross-service workflows. It intentionally contains no deployable service code.

The platform demonstrates a low-value unsecured personal-loan journey with progressive onboarding, explainable credit decisions, electronic signing, safe disbursement, and loan activation.

## Current status

**Discovery baseline:** 0.2 complete

**Domain documentation baseline:** complete

**Architecture baseline:** in progress

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
- [Bounded context map](docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md)
- [Canonical business rules](docs/domain/BUSINESS_RULES.md)
- [Canonical domain and integration events](docs/domain/DOMAIN_EVENTS.md)
- [Canonical Event Storming model](docs/domain/EVENT_STORMING.md)
- [Product and credit policy](docs/domain/PRODUCT_AND_CREDIT_POLICY.md)
- [Formal credit decision table](docs/domain/CREDIT_DECISION_TABLE.md)
- [Credit Decisioning bounded-context design](docs/domain/CREDIT_DECISIONING_DESIGN.md)

### Architecture

- [System context](docs/architecture/SYSTEM_CONTEXT.md)
- [Container architecture and execution profiles](docs/architecture/CONTAINER_DIAGRAM.md)
- [Component catalog](docs/architecture/COMPONENT_CATALOG.md)
- [Data ownership](docs/architecture/DATA_OWNERSHIP.md)
- [Platform security model](docs/architecture/SECURITY_MODEL.md)

### Architecture decision records

- [ADR-001 — Controlled Multi-Repository Delivery Model](docs/adr/ADR-001-MULTI-REPOSITORY.md)
- [ADR-002 — .NET 10 and Serverless AWS Demo Architecture](docs/adr/ADR-002-SERVERLESS-AWS.md)
- [ADR-003 — Event-Driven Communication and Persisted Application Process Saga](docs/adr/ADR-003-EVENT-DRIVEN.md)
- [ADR-004 — Database per Service and Data Isolation](docs/adr/ADR-004-DATABASE-PER-SERVICE.md)

### Workflows

- [Loan onboarding master workflow](docs/workflows/LOAN_ONBOARDING.md)
- [Credit decision workflow](docs/workflows/CREDIT_DECISION.md)
- [Document signing workflow](docs/workflows/DOCUMENT_SIGNING.md)
- [Disbursement workflow](docs/workflows/DISBURSEMENT.md)

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

1. Formalize system architecture, data ownership, security, workflows, and ADRs on top of the completed domain documentation baseline.
2. Publish versioned integration contracts.
3. Create each service repository from an approved specification.
4. Implement vertical slices with Spec-Driven Development and TDD.

## Important notice

The financial parameters, rates, score thresholds, and policies in this repository are fictitious and intended for architecture demonstration only. They are not production underwriting rules or country-specific regulatory guidance.
