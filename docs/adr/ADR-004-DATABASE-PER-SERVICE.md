# ADR-004: Database per Service and Data Isolation

- **Status:** Accepted
- **Decision date:** 2026-08-14
- **Decision authority:** Platform architecture and bounded-context owners

## Context

The platform assigns authoritative data to bounded contexts and depends on independent service delivery. Shared persistence would bypass owner rules, broaden access, couple schemas, and undermine event-driven integration. Local convenience must not erase logical isolation.

`Accepted` formalizes `A-005`; it does not select table designs, keys, indexes, libraries, capacity, IaC, or repository code.

## Decision drivers

- Enforce authoritative capability ownership and least privilege.
- Permit independent schema evolution, testing, deployment, and lifecycle.
- Prevent persistence from becoming an integration contract.
- Preserve isolation consistently in Local and AWS Demo.
- Support local atomic aggregate/Outbox changes and eventual cross-context consistency.

## Decision

Every service and projection owns an isolated logical persistence boundary.

### AWS Demo

- DynamoDB is the default operational store.
- Every deployed service owns its table or tables.
- Table names, IAM access, schema, keys, indexes, TTL configuration, backups, exports, and lifecycle are owner-scoped.
- No DynamoDB table is shared across bounded contexts.
- Audit Projection owns its own non-authoritative store.

### Local

Implementations may share a database engine or process for convenience, but each service uses a separate schema, database, file, or explicitly isolated logical store. Local convenience cannot introduce a shared domain schema or universal data-access credential.

### Transactions and integration

- Transactions are local to one owning service.
- Aggregate changes and that service's Outbox record commit atomically.
- Consumers persist Inbox/idempotency records inside their own boundary.
- Cross-context consistency is eventual and governed through events and workflow policies.
- Authoritative reads use the owning API or capability.
- Cross-context views use owner-approved replicas or projections.
- Copies retain source references, provenance, minimization, and retention.
- A projection cannot become an authoritative business source.

## Scope

This ADR governs logical data isolation, transactional boundaries, authorized reads, and replicas/projections. It does not decide single-table versus multi-table design, DynamoDB keys/indexes/capacity, libraries, repository code structure, or physical deployment topology.

## Consequences

### Positive

- Business invariants and access policy remain at the authoritative owner.
- Services can evolve and deploy persistence independently.
- A compromised service identity has a smaller data-access scope.
- Event and API contracts are explicit instead of hidden in shared queries.

### Negative

- Cross-context views require projections or API composition.
- Data is deliberately duplicated with provenance and retention obligations.
- Eventual consistency and reconciliation increase design and testing effort.

### Neutral and operational

- Local stores may share infrastructure while remaining logically isolated.
- DynamoDB is a default for AWS Demo, not a universal production database claim.
- Every copy has its own owner-controlled lifecycle without extending the source maximum.

## Alternatives considered

| Alternative | Benefits | Drawbacks for this platform | Why not selected now |
| --- | --- | --- | --- |
| One shared relational database | Simple joins and transactions | Couples schemas, credentials, releases, and authority | Conflicts with independent bounded-context ownership. |
| Schema per service in one shared database | Some namespace isolation | Shared engine administration and credential/transaction pressure remain | Local may use logical equivalents, but AWS service ownership requires stronger isolation. |
| One shared DynamoDB table for all contexts | Potential operational consolidation | Shared keys, IAM surface, lifecycle, and schema evolution | Owner-scoped tables better express authority and least privilege. |
| Central data service | One controlled query surface | Moves domain authority and creates a platform bottleneck | Capability owners must remain authoritative. |
| Distributed transactions | Immediate cross-context atomicity | Tight coordination, failure coupling, and infrastructure complexity | Local transactions plus events and process policies match the workflow. |
| Event sourcing by default | Complete event-derived history | Adds model, storage, replay, and migration complexity for every service | Existing requirements need events and audit, not universal event-sourced persistence. |

## Constraints and guardrails

- Prohibit cross-service table access, joins, shared schemas, shared domain persistence models, universal repository layers, universal database credentials, direct reporting queries, table-based integration, and distributed transactions across bounded contexts.
- Apply least-privilege service identities, encryption at rest, and owner-scoped access.
- For AWS Demo, do not enable PITR, backups, exports, retained snapshots, or storage replication. Owner-approved minimized projections remain permitted and retain their own provenance and lifecycle.
- Apply approved TTL/retention categories and explicit verified teardown from the Security Model.
- Production retention and database topology remain out of scope.

## Validation and revisit triggers

Revisit when a service specification defines physical storage, a cross-context query cannot be satisfied by an approved API/projection, persistence tests challenge `H-005`, or production recovery requirements are introduced. Revisit must preserve authority and cannot silently authorize shared schemas or credentials.

## References

- [Assumption A-005 and H-005](../discovery/ASSUMPTIONS.md)
- [Bounded-context ownership](../domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#5-context-responsibilities-and-ownership)
- [Container Architecture](../architecture/CONTAINER_DIAGRAM.md)
- [Data Ownership](../architecture/DATA_OWNERSHIP.md)
- [Security Model retention and deletion](../architecture/SECURITY_MODEL.md#10-aws-demo-retention-and-verified-deletion)
- [Cross-cutting business rules](../domain/BUSINESS_RULES.md#12-cross-cutting-xs)
