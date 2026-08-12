# Domain Documentation Baseline Design

**Issue:** [#1 Complete domain documentation baseline](https://github.com/SamuelSerrano/loan-platform-architecture/issues/1)

**Status:** Approved design

**Date:** 2026-08-12

**Canonical language:** English

## 1. Objective

Complete the transversal domain baseline with three explicit canonical documents while preserving the established bounded-context map and the authority of specialized Credit Decisioning documents.

The design must make business rules, domain events, integration events, commands, policies, aggregates, external systems, hotspots, and unresolved questions traceable without maintaining competing sources of truth.

## 2. Deliverables

Create:

- `docs/domain/BUSINESS_RULES.md`
- `docs/domain/DOMAIN_EVENTS.md`
- `docs/domain/EVENT_STORMING.md`

Update:

- `README.md`
- `docs/discovery/DISCOVERY.md`
- `docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`

No file will be deleted. No API or integration contract will be introduced or changed by this work.

## 3. Document authority

### 3.1 `BUSINESS_RULES.md`

This document is the canonical transversal catalog of business rules and aggregate invariants. Each entry identifies its stable ID, name, lifecycle status, owning bounded context, inputs, condition or constraint, outcome, applicable reason code, and source references.

Detailed formulas, pricing matrices, thresholds, and policy parameters remain canonical in `PRODUCT_AND_CREDIT_POLICY.md` and `CREDIT_DECISION_TABLE.md`. The catalog references those documents instead of copying their detail.

### 3.2 `DOMAIN_EVENTS.md`

This document is the canonical transversal event catalog. It separates domain events from integration events and records, for each event, its stable documentation ID, lifecycle status, producer, consumers, trigger, minimum payload, excluded sensitive data, and versioning expectations.

An integration event has both a stable documentation ID and a versioned contract name. For example:

```text
IE-CD-002 -> FavorableCreditDecisionRecorded.v1
```

This document defines semantic expectations only. Schemas remain a future responsibility of `loan-platform-contracts`.

### 3.3 `EVENT_STORMING.md`

This document is the canonical operational Event Storming model. It presents the end-to-end flow by phase and references catalog IDs rather than repeating rule and event definitions.

Each modeled sequence follows:

```text
Actor/Trigger -> Command -> Aggregate -> Domain Event -> Policy -> Integration Event -> Consumer
```

The document also catalogs external systems, alternate paths, operational recovery, hotspots, and unresolved questions.

### 3.4 Existing documents

`BOUNDED_CONTEXTS_AND_EVENT_STORMING.md` remains the canonical bounded-context classification, context map, relationship map, ownership summary, and executive journey overview. Detailed rule, event, and Event Storming catalogs will be replaced there by concise summaries and links to the new canonical documents.

`CREDIT_DECISIONING_DESIGN.md` remains canonical for the internal design of the Credit Decisioning bounded context. `PRODUCT_AND_CREDIT_POLICY.md` and `CREDIT_DECISION_TABLE.md` remain canonical for policy semantics, calculations, thresholds, matrices, reason codes, and test scenarios.

`README.md` and `DISCOVERY.md` will expose the new navigation and authority boundaries.

## 4. Organization

The catalogs are organized primarily by bounded context so ownership remains explicit and the content can later inform independently owned repositories. `EVENT_STORMING.md` supplies the journey-oriented view across those contexts.

Bounded-context abbreviations are:

| Code | Bounded context |
| --- | --- |
| `AP` | Application Process |
| `CI` | Customer & Identity |
| `CD` | Credit Decisioning |
| `DP` | Document Preparation |
| `ES` | Electronic Signature |
| `CO` | Communications |
| `LB` | Loan Booking |
| `DS` | Disbursement |
| `XS` | Cross-cutting |

## 5. Stable identifiers

| Pattern | Element |
| --- | --- |
| `BR-<context>-NNN` | Business rule or invariant |
| `DE-<context>-NNN` | Domain event |
| `IE-<context>-NNN` | Integration event |
| `CMD-<context>-NNN` | Command in the Event Storming model |
| `POL-<context>-NNN` | Policy or reaction in the Event Storming model |
| `HS-NNN` | Hotspot |
| `OQ-NNN` | Open question |

Identifiers remain stable when descriptive names change. IDs are never reused for a different concept.

## 6. Lifecycle status

Every cataloged element has one of these statuses:

| Status | Meaning |
| --- | --- |
| `Confirmed` | Explicitly established by an existing canonical source. |
| `Derived` | Necessary implication of established sources; the inference is documented. |
| `Proposed` | Candidate design requiring explicit approval before implementation or contract publication. |
| `Deferred` | Recognized concept intentionally outside the current MVP or baseline. |

Derived elements include a concise derivation note. Proposed elements are never presented as adopted decisions.

## 7. Event Storming phases

The operational model follows the established phases:

1. Application and identity
2. Decision and offer
3. Documents and approval
4. OTP and signature
5. Loan booking and disbursement

The happy path is accompanied by explicit alternate and recovery paths for:

- failed validation or missing consent;
- rejected or temporarily unavailable identity verification;
- incomplete, pending, or technically failed credit assessment;
- unfavorable decisions and rejected or expired offers;
- document correction and regeneration;
- invalid, exhausted, or expired OTP challenges;
- signature expiry;
- transient, terminal, or unknown disbursement outcomes;
- retry, idempotency, dead-letter handling, reconciliation, and manual recovery.

Operational failures remain distinct from unfavorable credit decisions. Compensation is modeled as an explicit business action rather than a distributed rollback.

## 8. Hotspots and open questions

Hotspots and unresolved questions are consolidated in `EVENT_STORMING.md` with stable IDs, an owning or deciding context, impact, and required decision. Existing entries in `ASSUMPTIONS.md` are linked rather than duplicated.

## 9. Validation strategy

The completed documentation must pass these checks:

1. All relative Markdown links resolve.
2. Every stable ID is unique and every ID reference resolves.
3. Every business rule has an owner, inputs, outcome, status, and source reference.
4. Every event has a producer, consumers, trigger, minimum payload, versioning expectation, and status.
5. Domain events and integration events are clearly separated.
6. Terms and bounded-context names match `UBIQUITOUS_LANGUAGE.md`.
7. Public events remain consistent with the existing context-map summary and Credit Decisioning design.
8. The existing combined document retains the context map without a competing detailed catalog.
9. Substantial literal duplication between canonical documents is avoided.
10. `git diff --check` passes or any intentional Markdown hard breaks are reviewed explicitly.
11. No secrets, temporary files, or unrelated changes are introduced.

## 10. Completion boundary

This work completes the documentation deliverables and acceptance criteria of issue #1. It does not publish schemas, create service repositories, implement runtime components, close the issue, commit implementation changes, or push changes without a separate review and authorization.
