# ADR-005 — Language-Neutral Contract Governance

**Status:** Accepted

**Date:** 2026-08-18

**Authority:** Issue #8 and the architecture repository governance process

## Context

The platform uses independently owned bounded contexts and planned repositories. Cross-boundary communication needs stable, minimized contracts without sharing domain models or allowing transport artifacts to become the source of business semantics. The initial walking skeleton also needs a field-level publication decision before the future contracts repository can be created.

## Drivers

- Independent service evolution and local domain ownership.
- Explicit producer, consumer, and compatibility responsibilities.
- Deny-by-default handling of sensitive data.
- Traceable correlation, causation, idempotency, versions, and timestamps.
- Deterministic contract validation without claiming implementation readiness.

## Decision

Public HTTP contracts will use OpenAPI; asynchronous operations and channels will use AsyncAPI; payloads will use JSON Schema. Contracts remain language-neutral and each service maps them to private models. The [Contracts Baseline](../contracts/CONTRACTS_BASELINE.md) governs evolution, while the [Initial Contract Catalog](../contracts/INITIAL_CONTRACT_CATALOG.md) approves the initial M1 field allowlists.

This decision formalizes `A-006`. It resolves `Q-004` only for the cataloged initial contracts and fields. A future addition requires its own field-level approval. `Q-008` remains open; no applicant-facing reason-code exposure or translation is authorized.

## Scope

Included: initial HTTP operations, integration events, one asynchronous policy request, `QuickPersonalLoanPolicy.v1`, ownership, planned standards, field allowlists, compatibility, deprecation, validation, and repository readiness.

Excluded: executable specifications, generated code, shared packages, infrastructure, runtime implementation, repository creation, production policy, and contracts beyond the catalog.

## Ownership and separation

- The operation-executing bounded context owns HTTP semantics; the edge owns only authentication, transport validation, and routing.
- The producer owns an integration event's business fact and maps a private domain event explicitly to an allowlisted public payload.
- The sender owns publication of an asynchronous request; the receiver owns acceptance and translation to a private command.
- Credit Decisioning owns versioned policy meaning and validation.
- Aggregates, commands, value objects, and domain events remain private and are never serialized directly.

## Planned formats

The initial planned targets are OpenAPI 3.1.2, AsyncAPI 3.1.0, JSON Schema Draft 2020-12, RFC 3339 UTC timestamps, and RFC 9457 Problem Details. OpenAPI 3.2.0 must be reevaluated against tooling when the future repository is created.

## Compatibility and deprecation

Only demonstrably safe additive optional changes may remain within a major version. Breaking structure, validation, or semantics requires a new major version. Deprecation requires replacement, consumer inventory, migration evidence, replay/DLQ analysis, an announced support window, and owner approval before removal.

## Security and allowlists

Publication is deny-by-default. Every field requires documented ownership/source, purpose, consumer, classification, treatment, locations, retention inheritance, rationale, canonical source, and approval. Full PII, raw evidence/documents, OTP values, secrets, credentials/tokens, provider payloads, unrestricted fraud details, and internal rule traces are prohibited. `Q-008` blocks applicant reason-code disclosure or translation.

## Validation and contract testing

The future repository must automate standards validation, linting, examples, field-policy checks, compatibility comparison, producer/consumer tests, duplicate and unsupported-version cases, technical-failure separation, traceability, and secret scanning. Contract tests do not replace domain tests.

## Consequences

### Positive

- Services evolve behind explicit, reviewable boundaries.
- Sensitive-field publication becomes auditable.
- Compatibility and ownership failures can be detected before deployment.

### Negative

- Each contract change requires additional review and compatibility evidence.
- Mapping between transport and private models adds deliberate implementation work.

### Neutral

- `loan-platform-contracts` becomes `Ready` but remains uncreated.
- M1 remains `Defined`; no executable contract is published by this ADR.

## Alternatives considered

| Alternative | Assessment |
| --- | --- |
| Share C# models | Rejected because it couples language and internal domain evolution. |
| Copy DTOs manually between services | Rejected because copies drift without a governed source. |
| Keep contracts only inside each service | Rejected because cross-repository compatibility and discovery become fragmented. |
| Use Markdown only | Retained for this planning issue but insufficient for future machine validation. |
| Always adopt the newest standard | Rejected as an automatic rule; ecosystem/tooling compatibility must be evaluated. |
| Serialize aggregates or domain events directly | Rejected because it leaks private models and bypasses minimization. |

## Constraints and guardrails

- No shared domain assembly or direct store access.
- No unpublished field, inferred consent, raw provider response, or technical-to-credit translation.
- Stable opaque identities and explicit versions are mandatory.
- A `Ready` repository state is not implementation or milestone evidence.

## Revisit triggers

- Creation of the contracts repository or material change in tooling support.
- A new consumer, contract category, field, product, or major version.
- Resolution of `Q-008` or a change to data classification/retention.
- Regulatory, provider, incident, or replay requirements beyond the demo baseline.

## References

- [Contracts Baseline](../contracts/CONTRACTS_BASELINE.md)
- [Initial Contract Catalog](../contracts/INITIAL_CONTRACT_CATALOG.md)
- [ADR-001](ADR-001-MULTI-REPOSITORY.md)
- [ADR-003](ADR-003-EVENT-DRIVEN.md)
- [Security Model](../architecture/SECURITY_MODEL.md)
- [Domain Events](../domain/DOMAIN_EVENTS.md)
- [Repository Delivery Roadmap](../../roadmap/REPOSITORY_DELIVERY_ROADMAP.md)
