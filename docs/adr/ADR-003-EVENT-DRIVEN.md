# ADR-003: Event-Driven Communication and Persisted Application Process Saga

- **Status:** Accepted
- **Decision date:** 2026-08-14
- **Decision authority:** Application Process and platform architecture

## Context

Loan onboarding is a long-running workflow across independently authoritative capabilities. Duplicate, delayed, and out-of-order delivery, external-provider ambiguity, deadlines, retries, and manual recovery require durable coordination without moving capability decisions into a central service.

This ADR formalizes `A-003`, `A-004`, and resolves `Q-005`. `Accepted` does not imply that the process manager, contracts, stores, queues, scheduler, or infrastructure are implemented.

## Decision drivers

- Preserve capability ownership while making end-to-end progress observable and recoverable.
- Support long-running deadlines and restart-safe coordination.
- Tolerate at-least-once, duplicate, and out-of-order delivery.
- Keep Local and AWS Demo behavior portable through replaceable adapters.
- Avoid distributed transactions and irreversible-operation retries based on ambiguous outcomes.

## Decision

Use event-driven communication and an explicitly persisted saga/process manager owned by Application Process.

Application Process persists only coarse orchestration information:

- application/process identity and current coarse stage;
- completed checkpoints and expected next facts;
- correlation and causation references;
- consumed event/version references needed for progression;
- pending operational disposition and recovery reference;
- workflow deadlines and timer metadata;
- idempotency information.

It does not persist or decide identity evidence/verification details, credit rules/calculations/immutable offer terms, document bytes/package ownership, OTP values/validation ownership, signature evidence, loan-booking rules, or transfer-provider outcomes. Capability contexts remain authoritative.

### Messaging topology

- Private domain events remain inside their owner.
- Explicit mappings produce versioned integration events; domain events are never published directly to EventBridge.
- Proposed integration events remain contract candidates until schemas are published in `loan-platform-contracts`.
- AWS Demo uses EventBridge for routing, one SQS queue and DLQ per deployed consumer, a transactional producer Outbox, consumer-owned Inbox/idempotency records, and an asynchronous sanitized Audit Projection.

### Delivery semantics

- Delivery is at least once; duplicates are expected and consumers are idempotent.
- Global ordering is not guaranteed. Per-aggregate version regression is rejected or safely deferred.
- Producer business state and its Outbox record commit atomically.
- Retries are bounded and preserve business identity.
- Poison messages are isolated; DLQ redrive follows cause correction and compatibility review.
- Queued business facts are never edited.
- Unknown irreversible provider outcomes are reconciled before retry.
- Manual recovery uses authorized, owner-controlled commands.
- Audit remains asynchronous and non-authoritative.

### Persisted deadlines and timers — Q-005 resolution

- Application Process persists workflow deadlines and timer metadata.
- A replaceable scheduling port activates due workflow checks.
- Local uses a deterministic clock/scheduler adapter.
- The AWS scheduling implementation remains an IaC/runtime decision.
- DynamoDB TTL is asynchronous expiry/deletion, not a precise workflow scheduler, and cannot guarantee deadline execution.
- Timer handling is idempotent; delayed or duplicate timers cannot bypass current aggregate state.
- Numeric deadline values remain in domain/service specifications unless already canonically approved.
- AWS Step Functions is not selected as the domain saga coordinator.

The exact scheduler technology, persistence schema, storage keys, and numeric timers remain implementation decisions.

## Scope

This decision governs cross-context event delivery and coarse Application Process coordination. It does not define executable contracts, schemas, commands/events beyond the existing catalogs, provider configurations, scheduler technology, or numeric policies.

## Consequences

### Positive

- Progress, deadlines, checkpoints, and recovery survive restarts.
- Capabilities retain authority while Application Process provides journey visibility.
- Per-consumer isolation supports independent retries, back-pressure, and redrive.
- The same logical coordination can run with deterministic local adapters or AWS services.

### Negative

- Eventual consistency and duplicate handling add implementation and testing complexity.
- Process state, Inbox, Outbox, queues, and timers require lifecycle management and observability.
- Debugging causal chains requires reliable correlation, version, and checkpoint data.

### Neutral and operational

- Process state is authoritative only for coarse journey progression, not capability facts.
- Consumer projections may be stale and must retain provenance.
- `H-004` and `H-005` remain assumptions requiring load, cost, persistence, and failure testing.

## Alternatives considered

| Alternative | Benefits | Drawbacks for this platform | Why not selected now |
| --- | --- | --- | --- |
| Pure choreography without persisted coordinator | Fewer central process records | Weak deadline visibility and restart-safe end-to-end recovery | Long-running onboarding needs explicit coarse checkpoints and timers. |
| AWS Step Functions as workflow owner | Managed execution history and scheduling | Couples domain coordination to AWS workflow semantics and the hosted profile | Application Process remains the portable domain process owner. |
| Synchronous service-call chain | Immediate request flow | Temporal coupling and fragile long-running recovery | Capabilities must progress independently and tolerate provider delays. |
| Direct broker-to-service delivery | Fewer queue resources | Shared retry/back-pressure and weaker consumer isolation | One queue/DLQ per consumer isolates delivery policy. |
| Shared workflow database | Easy centralized inspection | Violates service ownership and creates cross-context persistence coupling | Application Process owns only its process state; each consumer owns Inbox records. |
| Globally ordered event stream | Simplified ordering assumptions | Higher coupling and unnecessary global serialization | Per-aggregate progression checks are sufficient. |
| Exactly-once delivery assumption | Simpler handlers | Unsupported under the selected delivery semantics and failure model | Idempotent effects make duplicate delivery safe. |

## Constraints and guardrails

- Integration events require explicit mapping, minimization, versioning, and later contract publication.
- No consumer reads or writes another service's store.
- Process policies cannot bypass receiving aggregate invariants.
- Capability data cannot become authoritative in Application Process.
- Timers re-check current state and are idempotent.
- DLQ recovery preserves the original fact and identity.
- `HS-003` remains `Derived`; `H-004` and `H-005` are not treated as validated facts.

## Validation and revisit triggers

Revisit after load/cost testing, persistence failure tests, timer/restart tests, Inbox/Outbox design, or material workflow expansion. Revisit the AWS scheduler at IaC design without changing Application Process ownership. Any proposal for Step Functions or a different coordinator requires a new ADR that preserves domain authority.

## References

- [Assumptions A-003, A-004, H-004, H-005, and resolved Q-005](../discovery/ASSUMPTIONS.md)
- [Event Storming](../domain/EVENT_STORMING.md)
- [Domain Events](../domain/DOMAIN_EVENTS.md)
- [Container Architecture](../architecture/CONTAINER_DIAGRAM.md)
- [Component Catalog](../architecture/COMPONENT_CATALOG.md)
- [Data Ownership](../architecture/DATA_OWNERSHIP.md)
- [Loan Onboarding Workflow](../workflows/LOAN_ONBOARDING.md)
- [Amazon SQS at-least-once delivery](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues-at-least-once-delivery.html)
- [DynamoDB TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
