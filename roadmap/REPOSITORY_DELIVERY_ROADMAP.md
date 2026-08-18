# Repository Delivery Roadmap

## 1. Purpose and authority

This roadmap is the canonical source for the controlled creation sequence, repository entry gates, first vertical slices, portfolio milestones, delivery dependencies, and acceptance evidence for the Loan Onboarding Platform repositories.

It complements rather than replaces the sources that govern specialized decisions:

- [Discovery](../docs/discovery/DISCOVERY.md), [scope](../docs/discovery/SCOPE.md), and the [assumptions register](../docs/discovery/ASSUMPTIONS.md) govern product intent, constraints, assumptions, and open questions.
- The [ubiquitous language](../docs/domain/UBIQUITOUS_LANGUAGE.md), [bounded-context map](../docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md), business rules, event catalogs, and domain designs govern domain language and capability authority.
- The [system context](../docs/architecture/SYSTEM_CONTEXT.md), [container architecture](../docs/architecture/CONTAINER_DIAGRAM.md), [component catalog](../docs/architecture/COMPONENT_CATALOG.md), [data ownership](../docs/architecture/DATA_OWNERSHIP.md), and [security model](../docs/architecture/SECURITY_MODEL.md) govern architecture, data, and security boundaries.
- The four [workflows](../docs/workflows/LOAN_ONBOARDING.md) govern cross-context progression and recovery.
- [ADRs](../docs/adr/) govern accepted architecture decisions.

The GitHub Project is the transversal operating board. It does not replace this roadmap or any canonical source above.

Names in this roadmap identify planned delivery boundaries. Except for this architecture repository, they do not assert that future repositories, contracts, services, pipelines, deployments, or capabilities exist. A delivery state records planning or demonstrated evidence only; it never creates implementation automatically.

## 2. Delivery principles

- Repository creation is incremental, controlled, and justified by an approved, verifiable slice.
- Empty placeholder repositories are prohibited. A name, README, or initial folder structure alone does not justify creation.
- Every bounded context retains its own domain model and capability language.
- Every service retains its own persistence boundary; no database or persistence schema is shared across services.
- Every repository retains an independent pipeline and deployment lifecycle appropriate to its responsibility.
- C# domain models are never shared between services. Public contracts are language-neutral.
- Cross-context integration uses explicit, minimized contracts rather than another service's model or store.
- `Local Zero AWS Cost` is the default execution profile and is the first validation target.
- `AWS Demo` is ephemeral, bounded, and Free-Tier-aware. It does not guarantee zero cost.
- Production architecture, production controls, production data, production readiness, and production operating decisions remain out of scope.

## 3. Repository lifecycle states

Only the following states are valid:

| State | Definition |
| --- | --- |
| `Defined` | Purpose, boundary, dependencies, and intended evidence are documented, but the applicable creation gate is not yet satisfied. The repository may not be created. |
| `Ready` | All applicable universal and repository-specific gate evidence is reviewed and available. Creation is authorized but implementation has not necessarily started. |
| `In progress` | The repository exists after authorization and work on its approved slice is underway. This state makes no claim that the slice works end to end. |
| `Demonstrated` | The milestone-specific exit evidence is reproducible, linked, and verified. Documentation evidence demonstrates only documentation; runtime claims require runtime evidence. |
| `Deferred` | Delivery is intentionally postponed because prerequisites, priority, or scope do not justify creation or continuation. Deferred does not mean rejected. |

`Demonstrated` always requires verifiable evidence for the relevant milestone. A documentary milestone cannot be presented as runtime implementation. A future repository cannot be `Ready` while any applicable gate item is unmet. At this baseline, every implementation capability remains planned; only the architecture baseline has documentary evidence.

## 4. Universal repository creation gate

Before any future repository can move from `Defined` to `Ready`, its owner must link evidence for every applicable item:

- confirmed bounded context or clearly delimited responsibility;
- identified accountable owner;
- explicit purpose and exclusions;
- canonical sources and decision authority;
- authoritative data ownership and prohibited data access;
- relevant end-to-end and specialized workflows;
- identified inbound, outbound, and public contract needs without prematurely defining schemas;
- security, authorization, sensitive-data, retention, and minimization constraints;
- one bounded first vertical slice;
- dependencies that are available or deliberately simulated with deterministic substitutes;
- testing strategy, including unit, integration, contract, and end-to-end responsibilities as applicable;
- understood independent pipeline;
- independent deployment and rollback/teardown strategy;
- target milestone and acceptance evidence.

Passing only a repository-name check, creating a README, or generating an initial structure does not satisfy this gate. Missing or unresolved evidence keeps the repository `Defined`; it must not be created to reserve a name.

## 5. Repository catalog

For each microservice in this catalog, repository ownership includes its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. This ownership statement is part of every microservice row and prohibits direct dependency on another service's domain model or persistence schema.

| Repository | Type | Purpose | Excluded responsibilities | Canonical sources | Dependencies | Repository-specific entry criteria | First vertical slice | Milestone | Current state | Evidence needed to advance |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `loan-platform-architecture` | Architecture and governance | Govern discovery, domain language, boundaries, architecture, security, workflows, ADRs, roadmap, and repository gates. | Runtime code, executable contracts, service persistence, deployments, and service-owned operations. | This repository's canonical discovery, domain, architecture, workflow, ADR, and roadmap documents. | Reviewed product and architecture decisions. | Existing repository; architecture baseline sources are internally consistent, linked, and reviewed. | Documentary architecture baseline and controlled delivery roadmap. | M0 | `Demonstrated` | Documentary review, link validation, accepted ADRs, and merged canonical baselines; no runtime claim. |
| `loan-platform-contracts` | Language-neutral contracts | Own future public OpenAPI, AsyncAPI, JSON Schema, compatibility assets, and contract validation. | Shared C# domain models, service internals, business ownership, implementation, and deployment. | This roadmap, ADR-001, domain/event catalogs, workflows, security model, and issue #8. | M0; issue #8; owner-reviewed contract needs from skeleton contexts. | Issue #8 documents and satisfies the contract repository gate, including ownership, minimization, governance, compatibility, validation, and unresolved `Q-004` handling. | Baseline contracts needed by the walking skeleton, as selected and specified only by issue #8. | M1 | `Defined` | Approved issue #8 deliverable and gate checklist, traceability, contract-validation strategy, and confirmation that no shared domain models are included. |
| `loan-application-service` | Microservice — Application Process | Own the application journey, persisted coarse process manager, stage history, correlation/causation, offer acceptance, and operational dispositions. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Identity or credit decisions, immutable offer terms, documents, OTP, signatures, booking, transfers, or another service's data. | Bounded-context map, Application Process domain sources, master/credit workflows, ADR-003, ADR-004, security model. | M1 contract baseline; deterministic CI/CD substitutes; local event and persistence adapters. | AP owner and data boundary confirmed; skeleton contracts available; process checkpoints, idempotency, Inbox/Outbox, tests, and local dependency simulation specified. | Accept a synchronous application submission, coordinate fake identity and deterministic decision facts, create the applicant view of an offer, and record exact offer acceptance. | M2 | `Defined` | Passing unit/integration/contract tests plus local evidence for persistence, restart-safe progression, correlation/causation, idempotency, favorable and unfavorable paths. |
| `customer-identity-service` | Microservice — Customer & Identity | Own consent, customer profile, verification case/evidence, and provider-neutral identity outcomes. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Credit outcomes, application orchestration, raw evidence publication, documents, OTP/signature, booking, or transfers. | Bounded-context map, identity domain/event sources, master workflow, data ownership, security model, ADR-004. | M1 contract baseline; application submission fact; deterministic fake identity provider. | CI owner and sensitive-data boundary confirmed; skeleton contract needs available; fake provider behavior, evidence protection, idempotency, tests, and local persistence specified. | Consume a submitted application and produce a deterministic provider-neutral verified or rejected identity result without exposing raw evidence. | M2 | `Defined` | Contract tests and reproducible local verified/rejected scenarios proving minimization, ownership, Inbox/Outbox, idempotency, and technical-failure separation. |
| `credit-decisioning-service` | Microservice — Credit Decisioning | Own immutable assessments, ruleset version, completed `Favorable` or `Unfavorable` outcomes, reason codes, alternatives, and immutable offers. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Identity verification, application orchestration, offer acceptance, document/signature processing, booking, transfers, or translating technical failures into credit outcomes. | Credit Decisioning design, business rules, decision table, product policy, domain/event catalogs, credit workflow, security model. | M1 contract baseline; verified identity fact; deterministic policy and provider inputs. | CD owner/data boundary confirmed; required skeleton contracts available; deterministic ruleset, reason handling, immutable snapshot, failure dispositions, tests, and local persistence specified. | Consume a verified identity assessment request and deterministically record a favorable offer or unfavorable decision with safe reason codes and ruleset version. | M2 | `Defined` | Passing decision-table, unit, integration, and contract tests for favorable/unfavorable and operational-failure scenarios, including immutable offer evidence. |
| `loan-platform-infrastructure` | Infrastructure and observability | Compose approved Local and ephemeral AWS Demo capabilities, delivery topology, security controls, observability, cost controls, and verified teardown. Initially owns the non-authoritative Audit & Compliance Projection boundary. | Domain decisions, service business logic, shared service persistence, production topology, or an independent Audit microservice. | ADR-002 through ADR-004, container/component architecture, data ownership, security model, this roadmap. | Demonstrated M2 local integration; applicable AWS blockers resolved; deployable artifacts from authorized service repositories. | Local end-to-end evidence exists; AWS runtime/pricing revalidated; `Q-002`, `H-004`, and `H-005` blockers applicable to the demo are addressed by evidence or explicit constraints; deployment inventory and teardown acceptance are defined. | Provision and tear down only the three-service ephemeral AWS skeleton with edge/auth, event routing, consumer queues/DLQs, owner-scoped stores, logs, audit projection, and cost controls. | M3 | `Defined` | Reproducible provision/test/teardown record, zero residual governed resources, security checks, cost estimate/observation, and hosted skeleton trace evidence. |
| `loan-platform-demo-ui` | Presentation | Demonstrate the approved applicant journey through stable public APIs while keeping business rules in owning services. | Domain authority, direct service-store access, provider administration, operational recovery, or production UX claims. | Scope, system context, container architecture, security model, stable public API contracts, `Q-001`. | Stable applicant APIs from M2; authentication approach; resolved `Q-001`; UI decision supported by validated `H-006`. | `Q-001` resolved; required APIs stable; accessibility/security/test approach and deployment boundary approved; no business logic duplication. | Submit an application, observe fake identity and deterministic decision progress, review an offer, and accept it. | M6 | `Defined` | UI/API contract tests and a reproducible portfolio journey for favorable and unfavorable cases without direct data access or runtime overclaims. |
| `document-service` | Microservice — Document Preparation | Own immutable document packages, versions, protected artifacts/hashes, approval, correction, and invalidation. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Offer ownership, OTP, signature evidence, loan booking, transfer, or public storage access. | Document-signing workflow, bounded-context map, domain/event catalogs, data ownership, security model. | Accepted-offer contract; protected-object adapter; stable offer identity and terms hash. | Owner/data boundary confirmed; protected-object and retention controls specified; contract candidates governed; package version/hash, correction, authorization, and tests defined. | Generate, protect, expose for authorized review, approve, and correct one immutable package for an accepted offer. | M4 | `Defined` | Tests and trace evidence proving immutable versioning, protected access, correction/invalidation, minimization, and contract conformance. |
| `communications-service` | Microservice — Communications | Own notification delivery and OTP issue, protected representation, attempts, expiry, block, validation, and controlled reissue. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Signature completion, document contents, credit decisions, or treating delivery as OTP validation. | Document-signing workflow, bounded-context map, communications rules/events, data ownership, security model. | Approved signing purpose and envelope references; deterministic notification provider; governed private/public boundary. | Owner/data boundary confirmed; OTP secrecy, attempts/expiry/reissue, provider ambiguity, authorization, audit, and tests specified; applicable `Q-008`/provider blockers retained. | Issue and deliver a fake OTP, validate it once for an exact signer/purpose/envelope, and exercise expiry/reissue without exposing plaintext. | M4 | `Defined` | Security, unit, integration, and failure tests proving secrecy, single use, bounded attempts, delivery-versus-validation separation, and controlled reissue. |
| `signature-service` | Microservice — Electronic Signature | Own signature envelopes, signer/purpose/package binding, authorization consumption, provider interaction, reconciliation, evidence, and signed result. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Package generation, OTP validation, loan reservation, credit outcomes, or inferred signature success. | Document-signing workflow, bounded-context map, signature rules/events, data ownership, security model. | Demonstrated document and communications slices; approved package and authorization references; fake signature provider. | Owner/data boundary confirmed; exact bindings, expiry/invalidation, provider reconciliation, evidence protection, contracts, and tests specified. | Create an envelope for one approved package, consume a valid single-use authorization, reconcile the fake provider, and record the exact package as signed. | M4 | `Defined` | Contract and end-to-end tests proving bindings, expiry, invalidation, idempotency, ambiguous-provider reconciliation, and protected evidence. |
| `loan-account-service` | Microservice — Loan Booking | Own loan reservation, contractual terms, repayment schedule, `PendingDisbursement`, and activation after matching confirmed funds. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Signature evidence ownership, transfer execution, application completion, or silent reservation release/cancellation. | Disbursement/master workflows, bounded-context map, booking rules/events, data ownership, security model. | Confirmed signed-package fact; later confirmed-disbursement fact; `Q-007`; governed reservation lifecycle. | Owner/data boundary confirmed; `Q-007` resolved for schedule creation; reservation/activation invariants and tests specified; reservation cancellation/release remains explicitly open. | Reserve one signed package as `PendingDisbursement`, then activate only after an exact matching confirmed-funds fact. | M5 | `Defined` | Tests and traces proving one reservation, schedule consistency, no activation before funds, idempotent activation, and isolation of terminal operational failures. |
| `disbursement-service` | Microservice — Disbursement | Own one disbursement order, provider attempts, idempotency key, outcome reconciliation, and confirmed money-movement result. Retains its local domain model, application layer, persistence mappings, adapters, tests, pipeline, deployment lifecycle, and operational documentation. | Loan activation, reservation mutation, credit outcomes, automatic reversal, or invented post-failure disposition. | Disbursement/master workflows, bounded-context map, disbursement rules/events, data ownership, security model. | Valid `PendingDisbursement` reservation; tokenized destination; deterministic transfer provider; governed recovery path. | Owner/data boundary confirmed; irreversible-operation reconciliation, idempotency, tokenization, manual recovery, contracts, and tests specified; terminal disposition and reservation release remain open. | Create and execute one transfer order for a valid reservation, reconcile ambiguity, and emit confirmed funds without activating the loan. | M5 | `Defined` | Tests and traces for duplicate delivery, retry, ambiguity, confirmed/no-transfer outcomes, manual recovery controls, and prohibition of credit reinterpretation. |

## 6. First walking skeleton

The exact functional boundary is:

`Submit application → Fake identity verified → Deterministic decision → Offer created → Offer accepted`

It includes only Application Process, Customer & Identity, and Credit Decisioning. It must exercise:

- one synchronous applicant API at the Application Process boundary;
- asynchronous integration between the three contexts;
- explicit correlation and causation across the trace;
- transactional producer Outbox and consumer-owned Inbox behavior;
- idempotent handling of duplicate delivery and repeated applicant actions;
- both `Favorable` and `Unfavorable` completed credit outcomes;
- applicant-safe reason codes and traceable ruleset version;
- deterministic fake identity and decision dependencies;
- reproducible Local Zero AWS Cost end-to-end execution;
- contract tests against the issue #8 baseline.

Documents, Communications/OTP, electronic signature, loan reservation, disbursement, loan activation, and application completion are excluded from this skeleton.

## 7. Cross-repository creation order

The controlled order for the first walking skeleton is:

1. `loan-platform-contracts`;
2. `loan-application-service`;
3. `customer-identity-service`;
4. `credit-decisioning-service`;
5. `loan-platform-infrastructure`;
6. `loan-platform-demo-ui`.

This order means:

- The contracts boundary precedes every implementation that requires shared integration contracts.
- The contracts repository cannot be created until issue #8 documents and satisfies its gate.
- The three skeleton services are demonstrated first under Local Zero AWS Cost.
- Infrastructure is enabled only after local integration is demonstrated and applicable AWS Demo blockers are resolved.
- The demo UI is enabled only after its required APIs stabilize and `Q-001` is resolved.
- The order governs creation gates and dependencies; it does not authorize empty repositories to reserve names.

## 8. Subsequent capability sequence

After the first walking skeleton, capability repositories are enabled in this order:

1. `document-service`;
2. `communications-service`;
3. `signature-service`;
4. `loan-account-service`;
5. `disbursement-service`.

Documents precede signing because an exact immutable package must exist before an envelope can bind to it. Communications and OTP must be available before signing can be completed. Confirmed signing must precede Loan Booking. Loan Booking creates a reservation in `PendingDisbursement` before any transfer. Confirmed funds must precede Loan Activation.

Technical, provider, delivery, or persistence failures remain operational dispositions and can never become credit outcomes. Reservation cancellation or release and the business disposition after terminal failures remain open; this roadmap does not invent either decision.

## 9. Infrastructure and Audit boundary

`Audit & Compliance Projection` remains initially inside the infrastructure and observability boundary. It is asynchronous, sanitized, and non-authoritative, and it is not a ninth microservice.

An independent Audit repository may be considered only if Audit acquires at least one of these independently governed characteristics:

- its own API;
- an independent retention lifecycle;
- independent compliance ownership;
- a genuinely autonomous pipeline and deployment lifecycle.

Meeting one trigger permits reconsideration; it does not automatically authorize repository creation. The universal gate and an explicit architecture decision would still apply.

## 10. Portfolio milestones

| Milestone | Objective | Repositories involved | Dependencies | Entry evidence | Exit evidence | State | Explicitly out of scope |
| --- | --- | --- | --- | --- | --- | --- | --- |
| M0 — Architecture Baseline | Establish canonical discovery, domain, architecture, security, workflows, ADRs, and controlled roadmap. | Architecture governance repository. | Completed discovery/domain inputs and reviewed issue baselines. | Canonical source inventory and issue traceability. | Merged, linked, internally consistent documentary baseline and accepted ADRs. | `Demonstrated` | Runtime, executable contracts, service repositories, deployments, production readiness. Documentary evidence is not runtime evidence. |
| M1 — Contracts Baseline | Govern the minimum language-neutral contracts required by the walking skeleton. | Contracts boundary plus architecture governance. | M0; issue #8; preserved `Q-004`. | Approved contract scope, ownership, security constraints, and repository gate. | Versioned contract artifacts and validation evidence as authorized exclusively by issue #8. | `Defined` | Service implementation, shared C# models, production compatibility claims, contracts beyond the approved slice. |
| M2 — Local Walking Skeleton | Demonstrate the exact skeleton end to end at zero AWS cost. | Contracts, Application Process, Customer & Identity, and Credit Decisioning boundaries. | M1; three service gates; deterministic local adapters. | Contract baseline and approved service specifications/test strategies. | Reproducible local favorable/unfavorable traces, contract tests, correlation/causation, Inbox/Outbox, and idempotency evidence. | `Defined` | AWS deployment, UI, documents, OTP/signing, booking, disbursement, activation, production performance. |
| M3 — Ephemeral AWS Demo Skeleton | Demonstrate only the approved skeleton on bounded, ephemeral AWS resources. | M2 repositories plus infrastructure/observability. | Demonstrated M2; applicable `Q-002`, `H-004`, and `H-005` evidence; current pricing/runtime review. | Local trace, deployable artifacts, cost estimate, security review, inventory, teardown plan. | Hosted trace plus provisioning and verified teardown evidence, residual-resource check, and observed cost record. | `Defined` | Persistent environments, guaranteed zero cost, production topology, full platform deployment, load validation. |
| M4 — Documents and Signing | Demonstrate immutable documents, OTP authorization, and exact-package signing. | Contracts, Application Process, Document Preparation, Communications, Electronic Signature, and supporting infrastructure as authorized. | Stable accepted-offer boundary; individual repository gates; M2; relevant contract extensions. | Approved package, OTP, signature, security, provider-fake, and test specifications. | Reproducible package-to-signature happy/recovery traces with protected evidence and contract tests. | `Defined` | Loan reservation, transfer, activation, production legal-signature selection, real OTP or customer data. |
| M5 — Safe Fulfillment | Demonstrate reservation before transfer and activation only after confirmed funds. | Contracts, Application Process, Electronic Signature, Loan Booking, Disbursement, and supporting infrastructure as authorized. | M4; `Q-007` resolution where required; fulfillment contracts and repository gates. | Confirmed signed-package evidence, booking/disbursement invariants, provider fake, recovery test design. | Traces and tests for `PendingDisbursement`, idempotent transfer/reconciliation, confirmed funds, and activation ordering. | `Defined` | Invented reservation release, automatic reversal, post-terminal-failure disposition, production payments or servicing. |
| M6 — Portfolio Demo | Present the governed journey and evidence across approved local and optional ephemeral profiles. | Demonstrated prior milestone repositories plus demo UI when its gate passes. | Required prior milestones; stable APIs; resolved `Q-001`; validated UI value. | Linked milestone evidence and approved presentation/security boundary. | Reproducible portfolio script with evidence links and explicit limitations. | `Defined` | Production readiness, real applicants/funds, load certification, regulatory claims, persistent AWS environment. |

No `Defined` milestone represents implemented, deployed, load-validated, or production-ready capability.

## 11. GitHub Project operating model

The GitHub Project provides transversal operational tracking across issues, repositories, milestones, and pull requests. It displays work status and makes dependencies visible. It must link the acceptance evidence required by each roadmap milestone.

The Project does not replace domain documents, architecture views, security policy, workflows, ADRs, contract governance, or this roadmap. Marking a Project item `Done` does not automatically constitute architecture evidence or runtime evidence; the linked artifact and its verification determine whether a milestone can become `Demonstrated`.

## 12. Open questions and blockers

No question or assumption below is resolved by this roadmap.

| Item | Blocks or constrains |
| --- | --- |
| `Q-001` | Demo UI creation and M6 presentation scope. |
| `Q-002` | Infrastructure AWS account/Region/control choices and M3. |
| `Q-004` | Contract field allowlists, the issue #8 baseline, and every repository publishing or consuming those contracts. |
| `Q-006` | Identity renewal/reassessment behavior beyond the first skeleton; Customer & Identity and later Application Process evolution. |
| `Q-007` | Loan Booking schedule details and M5. |
| `Q-008` | Applicant-safe reason translation in affected APIs/UI, Communications, and M6. |
| `HS-006` | Unapproved private event names for identity-provider unavailability and activation failure; affected service specifications only. |
| `H-004` | EventBridge/SQS routing, retry isolation, and cost assumptions before M3/load claims. |
| `H-005` | DynamoDB transaction and Inbox/Outbox assumptions before M3 and service persistence claims. |
| Reservation cancellation or release | Loan Booking, Disbursement recovery, and any M5 scenario requiring reservation disposition. |
| Post-terminal-failure disposition | Application Process, Loan Booking, Disbursement, and M5 recovery completion. |
| Production decisions | Every production deployment, security, data, compliance, scale, cost, provider, and operating claim. |

Issue #8 retains exclusive responsibility for executable contract fields, payloads, schemas, detailed envelopes and metadata, normative names beyond existing canonical sources, versioning, compatibility, deprecation, and field-by-field allowlists. This roadmap defines only the contracts repository's purpose, scope, dependencies, entry criteria, first slice, milestone, creation order, and required evidence.
