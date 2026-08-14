# Credit Decision Workflow

## 1. Purpose and authority

This workflow describes the logical path from a valid identity result to a completed credit outcome, eligible alternatives, one immutable offer, and Application Process-owned acceptance or rejection. The [Credit Decisioning design](../domain/CREDIT_DECISIONING_DESIGN.md), [decision table](../domain/CREDIT_DECISION_TABLE.md), [product policy](../domain/PRODUCT_AND_CREDIT_POLICY.md), [business rules](../domain/BUSINESS_RULES.md), and [event catalog](../domain/DOMAIN_EVENTS.md) remain authoritative.

## 2. Scope and non-goals

It covers assessment dispositions, deterministic evaluation, offer lifecycle, acceptance/rejection, expiry, supersession, and reassessment. Logical orchestration uses the persisted Application Process process manager established by [ADR-003](../adr/ADR-003-EVENT-DRIVEN.md). It does not define executable contracts, new rules, provider behavior, persistence schemas, scheduler technology, safe-reason allowlists, or numeric retry/timer defaults.

## 3. Trigger

Application Process observes a valid provider-neutral identity result and requests `CMD-CD-001 RequestCreditAssessment` with one immutable input snapshot and exact policy/formula versions.

## 4. Preconditions

- Identity is verified, unexpired for the assessment instant, and traceable to Customer & Identity; renewal across reassessment remains `Q-006`.
- Required consents and assessment fields exist.
- Snapshot hash, assessment identity/version, application/customer references, correlation, and governing versions are stable.
- Credit Decisioning, not Application Process, owns evaluation and immutable offer terms.

## 5. Participants and ownership

| Participant | Responsibility | Does not own |
| --- | --- | --- |
| Application Process | Requests assessment, projects coarse state, records applicant acceptance/rejection | Evaluation, outcome, alternatives, offer terms/expiry |
| Customer & Identity | Supplies provider-neutral verification fact/reference | Credit outcome |
| Credit Decisioning | Snapshot, assessment, disposition, decision, alternatives, offer, expiry/supersession | Applicant acceptance/rejection journey action |
| Applicant | Selects an eligible alternative and accepts/rejects exact active offer | Replacement terms or authoritative decision fields |
| Audit | Sanitized asynchronous projection | Workflow control |

## 6. Happy path

| Step | Trigger / actor | Command | Owner / aggregate | Completed fact | Public mapping | Consumer / reaction | Idempotency | Outcome | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Valid identity reaction | `CMD-CD-001 RequestCreditAssessment` | CD / `CreditAssessment` | `DE-CD-001 CreditAssessmentStarted` | — | Deterministic engine | Assessment ID/version + snapshot hash | `DecisionPending` | Confirmed | [Phase B](../domain/EVENT_STORMING.md#5-phase-b--decision-and-offer) |
| 2 | Engine completes guards | `CMD-CD-002 RecordCreditDecision` | CD / `CreditAssessment` | `DE-CD-003 CreditDecisionRecorded` | `IE-CD-004 FavorableCreditDecisionRecorded.v1` | Calculate alternatives | Same assessment cannot produce another decision | `Favorable` | Confirmed | [Decision table](../domain/CREDIT_DECISION_TABLE.md#5-terminal-guard-decision-table) |
| 3 | Favorable reaction | `CMD-CD-003 CalculateOfferAlternatives` | CD / `CreditAssessment` | `DE-CD-004 OfferAlternativesCalculated` | — | Applicant options view | Deterministic snapshot/versions | 1..3 eligible alternatives | Confirmed | [Offer calculation](../domain/CREDIT_DECISION_TABLE.md#8-affordability-and-offer-calculation) |
| 4 | Applicant selects owned alternative | `CMD-CD-004 CreateCreditOffer` | CD / `CreditOffer` | `DE-CD-005 CreditOfferCreated` | `IE-CD-006 CreditOfferCreated.v1` | AP records active reference/snapshot | Conditional one-active-offer identity | Active immutable offer | Confirmed | [CreditOffer](../domain/CREDIT_DECISIONING_DESIGN.md#52-creditoffer-aggregate-root) |
| 5 | Applicant | `CMD-AP-003 AcceptCreditOffer` | AP / `CreditApplication` | `DE-AP-003 CreditOfferAccepted` | `IE-AP-002 CreditOfferAccepted.v1` | DP may generate documents | Applicant-action key; exact `offerId` + `termsHash` | `DocumentsPending` | Proposed | [HS-001](../domain/EVENT_STORMING.md#11-hotspots) |

Only `Favorable` and `Unfavorable` are completed credit outcomes. Alternative calculation occurs only after `Favorable`; one selected owned alternative creates one immutable active offer.

## 7. Alternate paths

| Condition | Detection owner | Permitted reaction | Forbidden shortcut | Retry / idempotency | Resulting state | Manual action | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Missing/remediable evidence | CD | Record `DE-CD-002 AssessmentDispositionRecorded` as `PendingEvidence`; request named categories | Create `Unfavorable` | New material evidence follows governed version progression | `DecisionPending` | Evidence collection only | Confirmed | [IE-CD-001](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Retryable provider unavailable | CD | Record `PendingRetry`; bounded retry same snapshot/versions/assessment ID | Decline or create new assessment implicitly | Same assessment and idempotency identity | `DecisionPending` | Escalate after exhaustion | Confirmed | [IE-CD-002](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Inconsistent/unsafe result | CD | Record `OperationalException`; stop automated evaluation | Fabricate/mutate decision | Duplicate disposition harmless | `OperationalException` / `DecisionPending` | Controlled recovery reference | Confirmed | [IE-CD-003](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Valid guard fails | CD | Record completed `Unfavorable`; emit safe permitted reasons | Treat as technical failure or produce offer | Published decision immutable; reassessment is new version | `CreditDeclined` | No silent override | Confirmed | [IE-CD-005](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Applicant rejects offer | AP | `CMD-AP-005 RejectCreditOffer`; record `DE-AP-004` | Mutate CD offer or generate documents | Stable applicant action + Inbox | `OfferClosed` | Explicit governed new path | Proposed | [IE-AP-003](../domain/DOMAIN_EVENTS.md#51-application-process-ap) |
| Offer expires | CD | Apply `DE-CD-006 CreditOfferExpired`; AP prevents acceptance | Accept stale terms or silently recalculate | Clock/redelivery idempotent | `OfferClosed` | Reassessment/new offer policy | Confirmed | [IE-CD-007](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Offer superseded | CD | Apply `DE-CD-007`; replace active AP reference from `IE-CD-008` | Accept prior offer or mutate terms | Prior/replacement identities stable | New active offer or closed path | None beyond governed reassessment | Confirmed | [IE-CD-008](../domain/DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Expired/superseded/rejected/unaccepted offer reaches documents | AP / DP | Reject progression | Generate package | Replay returns existing rejection/state | No document generation | Owner-controlled investigation | Confirmed | [BR-AP-006](../domain/BUSINESS_RULES.md#4-application-process-ap) |

## 8. Timeouts and expiry

Credit Decisioning owns offer expiry and uses an injected clock; expiry is idempotent. Assessment dependency timeouts yield `PendingRetry`, never a decision. Numeric backoff and retry budget are deferred. Identity reuse/renewal across reassessment remains `Q-006`.

## 9. Retry and idempotency

The same request key and snapshot hash return the existing assessment result. `PendingRetry` reuses the original assessment ID, snapshot, and versions. A material change or authorized reassessment creates a linked new assessment/version; it never rewrites a published decision. Offer creation enforces one active offer and exact selected-alternative terms. Applicant acceptance/rejection uses a stable action identity.

## 10. Compensation and recovery

There is no rollback of a completed decision. Reassessment is a new governed fact; supersession changes the separate offer lifecycle while preserving history. Operations may recover an `OperationalException` only through owner-controlled commands and evidence, never database edits or an invented analyst outcome.

## 11. Outcomes

| Class | Values |
| --- | --- |
| Completed credit | `Favorable`, `Unfavorable` |
| Operational disposition | `PendingEvidence`, `PendingRetry`, `OperationalException` |
| Journey projection | `DecisionPending`, `CreditDeclined`, active offer, `OfferClosed`, `DocumentsPending` |
| Not credit outcomes | Identity rejection, missing evidence, provider timeout, technical inconsistency, DLQ, duplicate/out-of-order delivery |

## 12. Security and data minimization

Public facts carry approved IDs, safe codes, policy versions, and minimum immutable offer snapshots/hashes. They exclude raw identity/financial evidence, full PII, provider payloads/credentials, unrestricted fraud details, and internal rule traces. Applicant explanations use safe codes without resolving `Q-008`; exact field allowlists remain `Q-004`. Authorization prevents client-supplied replacement terms and cross-customer access.

## 13. Open questions

- `Q-004`: contract field allowlists.
- `Q-006`: identity validity/renewal across reassessment.
- `Q-008`: applicant-facing reason-code disclosure/translation.

## 14. Traceability and navigation

- [Master Loan Onboarding](LOAN_ONBOARDING.md)
- [Document Signing](DOCUMENT_SIGNING.md)
- [Decision table](../domain/CREDIT_DECISION_TABLE.md)
- [Credit Decisioning design](../domain/CREDIT_DECISIONING_DESIGN.md)
- [Domain Events](../domain/DOMAIN_EVENTS.md)
