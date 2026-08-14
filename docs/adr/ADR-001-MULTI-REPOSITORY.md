# ADR-001: Controlled Multi-Repository Delivery Model

- **Status:** Accepted
- **Decision date:** 2026-08-14
- **Decision authority:** Platform architecture and repository governance

## Context

The platform separates eight business bounded contexts, cross-cutting projections, language-neutral contracts, infrastructure, and presentation concerns. Repository boundaries must preserve capability ownership and independent delivery without creating empty repositories before a useful slice exists.

`Accepted` approves this architecture direction. It does not assert that future repositories, contracts, pipelines, services, or deployments exist.

## Decision drivers

- Preserve bounded-context language, authority, data ownership, and deployment autonomy.
- Keep architecture governance independent from executable contracts and runtime code.
- Permit incremental delivery without premature repository proliferation.
- Prevent shared C# models or persistence schemas from coupling services.
- Make every created repository independently understandable, testable, and operable.

## Decision

Adopt a controlled, incremental multi-repository model:

- `loan-platform-architecture` is the canonical architecture and governance repository.
- A future `loan-platform-contracts` repository owns language-neutral public contracts.
- Service repositories are independently owned and are created only after satisfying their entry criteria.
- Infrastructure and UI repositories are created only after their boundaries, first slices, and delivery value are approved.
- The GitHub Project is the transversal delivery board, not a source of domain truth.

Issue #7 owns the repository creation sequence and portfolio milestones. This ADR does not finalize them.

### Architecture repository boundary

This repository owns Discovery, domain language and rules, bounded-context boundaries, architecture views, the security baseline, workflows, ADRs, the delivery roadmap, and repository entry criteria.

It is not a runtime monorepo, shared domain-library repository, executable-contract repository, deployment repository, or substitute for service-owned operational documentation.

### Contracts repository boundary

The future contracts repository owns OpenAPI, AsyncAPI, JSON Schema, event envelopes, compatibility and deprecation policy, and contract-validation assets. It does not contain or distribute shared C# domain models. `Q-004` remains open until field-by-field contract allowlists are approved.

### Service repository boundary

Each future service repository owns its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. It cannot depend directly on another service's domain model or persistence schema.

### Repository creation gate

A repository may be created only when all applicable evidence exists:

- confirmed bounded context and owner;
- documented responsibility and exclusions;
- documented authoritative data ownership;
- relevant documented workflows;
- understood inbound and outbound contract needs;
- identified security and sensitive-data constraints;
- defined first vertical slice;
- available or deliberately simulated dependencies;
- understood independent testing and deployment approach;
- defined milestone and acceptance evidence.

Empty placeholder repositories are prohibited.

## Scope

This decision governs repository responsibilities and creation gates for the architecture, future contracts, services, infrastructure, and UI. It does not create repositories or select their delivery order.

## Consequences

### Positive

- Ownership, release cadence, access, and operational responsibility can follow bounded contexts.
- Contracts remain language-neutral and independently governed.
- Repositories appear only when they can deliver and verify useful behavior.

### Negative

- Cross-repository changes require compatibility discipline and coordinated automation.
- More repositories increase discovery, dependency, and pipeline overhead as delivery expands.
- Shared improvements may need deliberate replication or tooling rather than a common domain package.

### Neutral and operational

- GitHub Project status complements but never replaces canonical documentation.
- Local end-to-end execution will eventually compose independently owned repositories.
- Repository catalogs and milestones remain roadmap artifacts governed by issue #7.

## Alternatives considered

| Alternative | Benefits | Drawbacks for this platform | Why not selected now |
| --- | --- | --- | --- |
| One monorepo for all services | Simple atomic changes and initial discovery | Encourages shared models, broad pipelines, and blurred ownership | The established bounded contexts require stronger independent ownership and delivery boundaries. |
| Create one repository per bounded context immediately | Makes the target topology visible early | Produces empty maintenance surfaces before slices and contracts are ready | Controlled creation provides evidence before operational overhead. |
| Combine architecture and contracts | Fewer repositories and simpler navigation | Mixes decision records with executable compatibility assets and release cadence | Contracts require independent versioning and validation. |
| Shared C# contract/domain packages | Convenient reuse for .NET consumers | Couples internal models, releases, and language choices | Public contracts remain language-neutral; each service maps locally. |
| Delay separation until after implementation | Low setup overhead initially | Makes later extraction depend on boundaries encoded in runtime code | Boundaries are already established and can govern incremental repository creation. |

## Constraints and guardrails

- No implementation repository is created before its entry criteria are satisfied.
- No repository directly accesses another service's domain model or persistence schema.
- Public compatibility is governed through language-neutral contracts, not shared domain assemblies.
- Architecture decisions remain canonical here; implementation details remain service-owned.
- Repository sequencing must remain open until issue #7 approves it.

## Validation and revisit triggers

Revisit when issue #7 defines delivery milestones, a repository repeatedly cannot deploy independently, a bounded-context boundary changes, contract governance proves insufficient, or operational overhead outweighs the demonstrated autonomy. Revalidation must not silently merge domain or persistence ownership.

## References

- [Discovery](../discovery/DISCOVERY.md)
- [Assumptions A-001 and A-006](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-and-adr-tracking)
- [Bounded-context repository boundaries](../domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#11-repository-boundaries-derived-from-the-model)
- [MVP delivery sequence](../../README.md#delivery-sequence)
- [Data Ownership](../architecture/DATA_OWNERSHIP.md)
