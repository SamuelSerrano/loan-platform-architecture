# Contracts Governance Baseline

**Status:** Approved governance baseline; M1 evidence published

**Date:** 2026-08-18

## 1. Purpose and authority

This document governs the language-neutral contracts repository for the Loan Onboarding Platform. It defines contract categories, ownership, formats, versioning, compatibility, deprecation, validation, and the repository creation gate. The [initial contract catalog](INITIAL_CONTRACT_CATALOG.md) remains authoritative for the approved M1 field allowlists.

Domain semantics remain authoritative in the domain sources, bounded contexts retain business ownership, the [Security Model](../architecture/SECURITY_MODEL.md) governs data treatment, and workflows govern cross-capability progression. Executable OpenAPI, AsyncAPI, and JSON Schema artifacts live in [`loan-platform-contracts`](https://github.com/SamuelSerrano/loan-platform-contracts), not in this governance document. Producers and receiving capabilities retain their business semantics.

## 2. Contract categories and ownership

### Public synchronous APIs

OpenAPI will describe HTTP operations. The bounded context executing an operation owns its semantics. The edge authenticates, validates transport concerns, and routes; it does not own business rules. Errors use minimized [RFC 9457 Problem Details](https://www.rfc-editor.org/info/rfc9457/) fields and approved safe extensions only.

### Integration events

AsyncAPI will describe operations, channels, producers, and consumers. JSON Schema will describe the event envelope and payload. The producer owns the meaning of the occurred fact. Private domain events are never published directly: an explicit mapping and approved field allowlist are mandatory before producing an integration event.

### Asynchronous policy requests

Policy requests ask another capability to evaluate work and may be rejected. They are neither occurred facts nor domain events and do not receive `IE-*` catalog identifiers. The sender owns publication; the receiver owns acceptance rules and translation to an internal command.

### Versioned policy and configuration contracts

`QuickPersonalLoanPolicy.v1` is fictitious, immutable, versioned configuration. Credit Decisioning owns its meaning and validation. It is not a shared domain model.

### Internal domain models

Aggregates, value objects, commands, and domain events remain private to their owning service. They are excluded from the contracts repository and are not distributed as shared C# assemblies. Every service maps contracts to its own internal model.

## 3. Standards baseline

| Concern | Baseline | Decision |
| --- | --- | --- |
| HTTP APIs | [OpenAPI 3.1.2](https://spec.openapis.org/oas/v3.1.2.html) | Initial published baseline. |
| OpenAPI reassessment | [OpenAPI 3.2.0](https://spec.openapis.org/oas/v3.2.0.html) | Official newer version; tooling support must be reevaluated before a later baseline adopts it. |
| Async APIs | [AsyncAPI 3.1.0](https://www.asyncapi.com/docs/reference/specification/v3.1.0) | Published channel, operation, producer, and consumer description. |
| Payload schemas | [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12) | Published envelope, request, event, and policy validation. |
| Timestamps | [RFC 3339](https://www.rfc-editor.org/info/rfc3339/) UTC | Use an explicit `Z` offset and sufficient precision for ordering evidence. |
| HTTP errors | [RFC 9457](https://www.rfc-editor.org/info/rfc9457/) | Safe Problem Details response with allowlisted extensions. |

Release [`v0.1.0`](https://github.com/SamuelSerrano/loan-platform-contracts/releases/tag/v0.1.0) implements these baselines; later evolution remains governed by this document.

## 4. Repository structure

```text
openapi/              HTTP API descriptions
asyncapi/             Asynchronous operations and channels
schemas/common/       Reusable transport primitives and envelopes
schemas/events/       Integration-event payload schemas
schemas/requests/     Asynchronous request payload schemas
schemas/policies/     Versioned policy/configuration schemas
examples/             Fictitious, minimized positive and negative examples
tests/                Lint, compatibility, example, and contract checks
docs/                 Repository-local governance and usage guidance
CHANGELOG.md           Version and deprecation history
CONTRIBUTING.md        Change and review workflow
CODEOWNERS             Required owner and security reviewers
```

The initial repository materializes this structure for the approved M1 boundary.

## 5. Naming and versioning

- Contract names use business language and an explicit major suffix such as `.v1`.
- Integration-event names describe facts in the past tense; policy-request names express intent and must not imply that a fact occurred.
- Documentation IDs such as `IE-CD-001` are traceability identifiers, not wire names.
- Documentation IDs are stable and retired names, IDs, and versions are never reused with different semantics.
- The repository release version is independent of each contract's visible major version; one release may carry several contract versions.
- HTTP paths and operation identifiers remain stable within a released major version.
- JSON property names use `camelCase`; schema names use `PascalCase`.
- IDs are opaque strings. Money is an amount plus ISO-style currency code; no floating-point representation is allowed.
- Every event, request, and policy instance declares its contract version.
- A new major version is required for removal, type or requiredness changes, semantic changes, enum narrowing, or incompatible validation changes.

## 6. Event envelope

Every initial integration event uses the allowlisted common envelope in the catalog: `eventId`, `eventType`, `eventVersion`, `occurredAt`, `aggregateId`, `correlationId`, `causationId`, `producer`, `traceId`, and `payload`. Timestamps are RFC 3339 UTC and IDs are opaque. `eventId` is the consumer delivery-deduplication identity; `correlationId` groups the journey; `causationId` identifies the immediately causal message or action. `traceId` is sanitized telemetry and never replaces correlation. The envelope prohibits PII, secrets, credentials, and provider payloads. Broker metadata is not silently promoted into the business contract.

No CloudEvents conformance is claimed. A later ADR may reconsider it only if concrete interoperability needs and tooling justify adoption.

## 7. Asynchronous request metadata

`CreditAssessmentRequested.v1` uses `requestId`, `requestType`, `requestVersion`, `requestedAt`, `applicationId`, `assessmentVersion`, `correlationId`, `causationId`, `producer`, `targetCapability`, `traceId`, `idempotencyKey`, `snapshotHash`, and `payload`. It deliberately does not reuse `eventId` or `occurredAt` and has no `IE-*` identifier. The receiver deduplicates by the stable idempotency identity and verifies that a replay has the same snapshot hash.

## 8. Compatibility and evolution

Adding an optional field may be backward compatible only when old consumers ignore it safely and defaults do not alter business meaning. Removing or renaming a field; making an optional field required; changing type, format, meaning, ownership, classification, unit, or monetary semantics; narrowing a range or constraint; or adding enum values without an explicitly extensible-enum strategy is breaking. Published versions are immutable, and a breaking change requires a new visible major version. Consumer-driven and producer validation must cover supported versions. Unknown major versions are rejected or quarantined as technical incompatibility, never translated into a credit outcome.

## 9. Deprecation

A deprecation proposal requires a published replacement, changelog entry, affected-producer/consumer inventory, notice, contract tests, verified migration, and preserved historical documentation. The portfolio/demo support window is at least 30 days and one demonstrable release cycle. A contract remains available while any known consumer is not migrated or replay/DLQ exposure is unresolved. Emergency retirement for a security flaw requires an ADR or incident decision plus consumer coordination; it does not permit silent semantic reuse. The minimum window is a portfolio convention, not a universal production policy.

## 10. Security and field allowlists

Contract publication is deny-by-default. Only fields listed in the [initial catalog](INITIAL_CONTRACT_CATALOG.md) are approved for the initial M1 scope. Permitted classifications are `Public`, `Internal`, `Confidential`, `Restricted`, and `Restricted secret`. Full PII, raw evidence or documents, plaintext OTPs, credentials or tokens, provider payloads, unrestricted fraud details, and internal rule traces are prohibited.

Each future contract or field requires its own owner, purpose, consumer, classification, treatment, permitted/prohibited locations, retention inheritance, rationale, canonical source, and approval. This baseline does not authorize applicant display of reason codes: `Q-008` remains open, and applicant exposure or translation is prohibited until it is resolved.

## 11. Validation and contract testing

The future repository gate requires:

1. structural validation against the selected standards;
2. linting and naming checks;
3. positive and negative fictitious examples;
4. field-allowlist and classification checks;
5. backward-compatibility comparison against released versions;
6. producer serialization and consumer deserialization tests;
7. consumer-driven tests for known consumers;
8. duplicate, unsupported-version, malformed-message, and technical-failure scenarios;
9. security review and secret scanning;
10. traceability from contract to owner command/private event, workflow, rules, and decision scenario.

Contract tests prove transport agreement, not business correctness or production readiness.

## 12. Repository creation gate

The documentary gate checklist is satisfied when:

- [x] ADR-005 is `Accepted`.
- [x] this baseline is approved and the initial catalog is complete;
- [x] `Q-004` is resolved for every initial field;
- [x] ownership, security treatment, compatibility, deprecation, validation, and contract testing are defined;
- [x] traceability and the planned repository structure are documented;
- [x] no domain model or shared C# assembly is included;
- [x] planned pipeline release gates and `CODEOWNERS` responsibilities are understood; and
- [x] all future examples are required to be fictitious.

With this baseline and [ADR-005](../adr/ADR-005-CONTRACT-GOVERNANCE.md), `loan-platform-contracts` moved to `Ready`; issue #18 then governed its creation and initial publication.

PR [#1](https://github.com/SamuelSerrano/loan-platform-contracts/pull/1) was merged as [`e922039`](https://github.com/SamuelSerrano/loan-platform-contracts/commit/e9220390054109f99b9ce62fa87570ea2be3092d). Release [`v0.1.0`](https://github.com/SamuelSerrano/loan-platform-contracts/releases/tag/v0.1.0) publishes exactly 16 contracts and 175 governed field paths. Its .NET 10 Hexagonal Architecture validation pipeline records ten passing gates and zero findings in the [sanitized report](https://github.com/SamuelSerrano/loan-platform-contracts/releases/download/v0.1.0/validation-report.json), with the generated [OpenAPI bundle](https://github.com/SamuelSerrano/loan-platform-contracts/releases/download/v0.1.0/openapi-bundle.yaml) and [SHA-256 checksums](https://github.com/SamuelSerrano/loan-platform-contracts/releases/download/v0.1.0/SHA256SUMS.txt). This evidence makes M1 `Demonstrated`; it does not start service implementation or advance M2 beyond `Defined`. Any new contract or field reopens the same field-level gate. `Q-001`, `Q-002`, `Q-006`, `Q-007`, and `Q-008` remain open.

## References

- [Initial Contract Catalog](INITIAL_CONTRACT_CATALOG.md)
- [ADR-005](../adr/ADR-005-CONTRACT-GOVERNANCE.md)
- [Domain Events](../domain/DOMAIN_EVENTS.md)
- [Security Model](../architecture/SECURITY_MODEL.md)
- [Repository Delivery Roadmap](../../roadmap/REPOSITORY_DELIVERY_ROADMAP.md)
