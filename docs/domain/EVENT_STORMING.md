# Canonical Event Storming Model

**Product:** Loan Onboarding & Credit Decisioning Platform

**Status:** Domain documentation baseline

**Canonical language:** English

## 1. Purpose and authority

This document is the canonical operational Event Storming model for the end-to-end loan-onboarding journey. It connects actors, commands, aggregates, completed domain facts, process reactions, public integration candidates, and consumers without redefining their semantics.

The [business-rule catalog](BUSINESS_RULES.md) is authoritative for invariants, the [event catalog](DOMAIN_EVENTS.md) is authoritative for domain and integration-event semantics, and the [bounded-context map](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md) is authoritative for ownership and context relationships. Credit Decisioning internals remain governed by [its specialized design](CREDIT_DECISIONING_DESIGN.md), [policy](PRODUCT_AND_CREDIT_POLICY.md), and [decision table](CREDIT_DECISION_TABLE.md).

## 2. Conventions

- The model reads `Actor or trigger -> Command -> Aggregate -> Domain Event -> Policy -> Integration Event -> Consumer`.
- Commands express rejectable intent. Domain events record completed facts. Policies react to facts and may request the next context-owned action.
- `Application Process` coordinates the coarse saga; it does not evaluate identity, credit, documents, signatures, booking, or transfers.
- `CMD-*`, `POL-*`, `HS-*`, and `OQ-*` are stable documentation IDs. Statuses are `Confirmed`, `Derived`, `Proposed`, or `Deferred` as defined in the catalogs.
- `—` means that no public integration event is required for that step. Private domain events are never sent directly to EventBridge.
- Every public event reference denotes the versioned contract candidate cataloged in `DOMAIN_EVENTS.md`; it is not a published schema.
- A phase-step status is the least mature status among its command, event, policy, and public mapping. Assigning an ID to behavior already established by a canonical source keeps it `Confirmed`; a necessary but previously unnamed reaction is `Derived`; unresolved ownership or contract mapping makes the whole step `Proposed`.

## 3. Actors and external systems

| Participant | Classification | Role in this model | Source |
| --- | --- | --- | --- |
| Applicant | Human actor | Starts, submits, accepts or rejects, reviews, corrects, authenticates, and signs | [Parties and roles](UBIQUITOUS_LANGUAGE.md#4-parties-and-roles) |
| Customer | Domain role | Identified person whose profile, consent, and verification evidence are owned by Customer & Identity | [Product decision P-006](../discovery/ASSUMPTIONS.md#confirmed-product-decisions) |
| Signer | Domain role | Person authorized to sign one bound package; not yet necessarily a Borrower | [Product decision P-006](../discovery/ASSUMPTIONS.md#confirmed-product-decisions) |
| Credit Analyst | Human actor | Reviews explanations and future manual-review cases; does not override MVP decisions | [Deferred rules](BUSINESS_RULES.md#13-deferred-rules) |
| Platform Operator | Human actor | Requests authorized, evidenced, owner-controlled recovery and reconciliation; has no implicit underwriting authority | [Resolved Q-009](../discovery/ASSUMPTIONS.md#resolved-questions) |
| Identity Provider | External business system | Supplies provider-specific verification evidence through the Customer & Identity anti-corruption layer | [Context relationships](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#6-context-relationships) |
| Disbursement Provider | External business system | Executes or reports the transfer identified by the stable business idempotency key | [Disbursement rules](BUSINESS_RULES.md#11-disbursement-ds) |
| Amazon Cognito | Generic platform system | Authenticates actors and supplies authorization scopes; it owns no business decision | [Domain classification](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#3-domain-classification) |
| Amazon EventBridge | Integration infrastructure | Routes versioned public facts; it does not own business semantics | [Architecture decision A-004](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-awaiting-adrs) |
| Per-consumer SQS queue | Integration infrastructure | Isolates delivery and retry for one consumer with at-least-once semantics | [Architecture decision A-004](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-awaiting-adrs) |
| Per-consumer DLQ | Recovery infrastructure | Retains exhausted deliveries for diagnosis and controlled redrive | [Architecture decision A-004](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-awaiting-adrs) |
| Protected object storage | Security infrastructure | Stores document bytes and protected evidence behind owning APIs | [BR-DP-004](BUSINESS_RULES.md#7-document-preparation-dp) |
| Simulated notification channels | External adapter | Deliver fictitious OTP and transactional messages without owning validation | [Constraints C-003 and C-004](../discovery/ASSUMPTIONS.md#delivery-constraints) |

## 4. Phase A — Application and identity

| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A1 | Applicant | CMD-AP-001 `StartCreditApplication` | `CreditApplication` | [DE-AP-001](DOMAIN_EVENTS.md#41-application-process-ap) | POL-AP-011 capture progressive customer data | — | Applicant Application Tracker | Confirmed |
| A2 | Applicant | CMD-CI-001 `RegisterCustomerProfile` | `Customer` | [DE-CI-001](DOMAIN_EVENTS.md#42-customer--identity-ci) | POL-CI-001 request the required purpose-bound consents | — | Customer profile view | Confirmed |
| A3 | Applicant | CMD-CI-002 `GrantConsent` | `ConsentRecord` | [DE-CI-002](DOMAIN_EVENTS.md#42-customer--identity-ci) | POL-AP-012 allow submission only when [BR-CI-001](BUSINESS_RULES.md#5-customer--identity-ci) and the required consent set hold | — | Consent status | Confirmed |
| A4 | Applicant | CMD-AP-002 `SubmitCreditApplication` | `CreditApplication` | [DE-AP-002](DOMAIN_EVENTS.md#41-application-process-ap) | POL-AP-001 start identity verification after [BR-AP-001](BUSINESS_RULES.md#4-application-process-ap) succeeds | [IE-AP-001](DOMAIN_EVENTS.md#51-application-process-ap) | Customer & Identity; Audit Trail projection | Proposed |
| A5 | Application-submitted reaction | CMD-CI-003 `StartIdentityVerification` | `IdentityVerification` | [DE-CI-003](DOMAIN_EVENTS.md#42-customer--identity-ci) | POL-CI-002 invoke the provider through the owning anti-corruption layer | — | Identity Provider | Confirmed |
| A6 | Provider-neutral successful result | CMD-CI-004 `RecordIdentityVerification` | `IdentityVerification` | [DE-CI-004](DOMAIN_EVENTS.md#42-customer--identity-ci) | POL-AP-002 request assessment only while verification satisfies [BR-CD-008](BUSINESS_RULES.md#6-credit-decisioning-cd) | [IE-CI-001](DOMAIN_EVENTS.md#52-customer--identity-ci) | Application Process; Audit Trail projection | Proposed |

## 5. Phase B — Decision and offer

| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| B1 | Verified-identity reaction | CMD-CD-001 `RequestCreditAssessment` | `CreditAssessment` | [DE-CD-001](DOMAIN_EVENTS.md#43-credit-decisioning-cd) | POL-CD-002 evaluate the immutable snapshot under [BR-CD-001](BUSINESS_RULES.md#6-credit-decisioning-cd) and deterministic precedence | — | Credit Decisioning | Confirmed |
| B2 | Decision engine | CMD-CD-002 `RecordCreditDecision` | `CreditAssessment` | [DE-CD-003](DOMAIN_EVENTS.md#43-credit-decisioning-cd) | POL-CD-001 calculate alternatives only for a completed favorable outcome | [IE-CD-004](DOMAIN_EVENTS.md#53-credit-decisioning-cd) | Application Process | Confirmed |
| B3 | Favorable-decision reaction | CMD-CD-003 `CalculateOfferAlternatives` | `CreditAssessment` | [DE-CD-004](DOMAIN_EVENTS.md#43-credit-decisioning-cd) | POL-CD-003 present eligible alternatives without moving ownership to the coordinator | — | Applicant offer-options view | Confirmed |
| B4 | Applicant selects an owned alternative | CMD-CD-004 `CreateCreditOffer` | `CreditOffer` | [DE-CD-005](DOMAIN_EVENTS.md#43-credit-decisioning-cd) | POL-AP-003 present one immutable active offer with its expiry | [IE-CD-006](DOMAIN_EVENTS.md#53-credit-decisioning-cd) | Application Process | Confirmed |
| B5 | Applicant | CMD-AP-003 `AcceptCreditOffer` | `CreditApplication` | [DE-AP-003](DOMAIN_EVENTS.md#41-application-process-ap) | POL-AP-004 under confirmed Application Process ownership, verify the active offer reference and exact canonical `termsHash`, record the idempotent applicant action and stage transition, then request document generation | [IE-AP-002](DOMAIN_EVENTS.md#51-application-process-ap) | Document Preparation; Audit Trail projection; Credit Decisioning projection if required | Proposed |

`HS-001` was resolved on 2026-08-14: Application Process owns the acceptance action and its idempotency. The command references, but never mutates, Credit Decisioning's immutable `CreditOffer` and canonical terms.

## 6. Phase C — Documents and approval

| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| C1 | Accepted-offer reaction | CMD-DP-001 `GenerateDocumentPackage` | `DocumentPackage` | [DE-DP-001](DOMAIN_EVENTS.md#44-document-preparation-dp) | POL-DP-001 make references and hashes available; bytes remain protected under [BR-DP-004](BUSINESS_RULES.md#7-document-preparation-dp) | [IE-DP-001](DOMAIN_EVENTS.md#54-document-preparation-dp) | Application Process; Audit Trail projection | Proposed |
| C2 | Applicant after review | CMD-DP-002 `ApproveDocumentPackage` | `DocumentPackage` | [DE-DP-002](DOMAIN_EVENTS.md#44-document-preparation-dp) | POL-ES-001 create an envelope for the exact immutable package | [IE-DP-002](DOMAIN_EVENTS.md#54-document-preparation-dp) | Electronic Signature; Application Process; Audit Trail projection | Proposed |
| C3 | Approved-package reaction | CMD-ES-001 `CreateSignatureEnvelope` | `SignatureEnvelope` | [DE-ES-001](DOMAIN_EVENTS.md#45-electronic-signature-es) | POL-ES-004 request a purpose-, signer-, and envelope-bound challenge | — | Electronic Signature | Confirmed |

## 7. Phase D — OTP and signature

| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| D1 | Signature flow | CMD-CO-001 `IssueOtpChallenge` | `OtpChallenge` | [DE-CO-001](DOMAIN_EVENTS.md#46-communications-co) | POL-CO-001 deliver through the configured channel without exposing the secret | — | Simulated notification channel | Confirmed |
| D2 | Channel adapter success | CMD-CO-002 `RecordOtpDelivery` | `NotificationDelivery` | [DE-CO-002](DOMAIN_EVENTS.md#46-communications-co) | POL-CO-004 preserve delivery separately from validation under [BR-CO-004](BUSINESS_RULES.md#9-communications-co) | — | Signature status view | Confirmed |
| D3 | Applicant | CMD-CO-003 `ValidateOtp` | `OtpChallenge` | [DE-CO-004](DOMAIN_EVENTS.md#46-communications-co) | POL-ES-002 authorize, but do not complete, the bound signing action | — | Electronic Signature | Confirmed |
| D4 | Authorized Signer | CMD-ES-002 `SignDocumentPackage` | `SignatureEnvelope` | [DE-ES-002](DOMAIN_EVENTS.md#45-electronic-signature-es) | POL-LB-001 reserve a loan only after valid tamper-evident evidence | [IE-ES-001](DOMAIN_EVENTS.md#55-electronic-signature-es) | Loan Booking; Application Process; Audit Trail projection | Proposed |

OTP and notification facts remain private. No cross-context `IE-CO-*` contract is implied.

## 8. Phase E — Loan booking and disbursement

| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| E1 | Signed-package reaction | CMD-LB-001 `ReserveLoanAccount` | `LoanAccount` | [DE-LB-001](DOMAIN_EVENTS.md#47-loan-booking-lb) | POL-DS-001 create one transfer order for the pending reservation | [IE-LB-001](DOMAIN_EVENTS.md#57-loan-booking-lb) | Disbursement; Application Process; Audit Trail projection | Proposed |
| E2 | Reservation reaction | CMD-DS-001 `CreateDisbursementOrder` | `DisbursementOrder` | [DE-DS-001](DOMAIN_EVENTS.md#48-disbursement-ds) | POL-DS-003 validate the tokenized destination under [BR-DS-004](BUSINESS_RULES.md#11-disbursement-ds) | — | Disbursement | Confirmed |
| E3 | Valid order | CMD-DS-002 `ExecuteDisbursement` | `DisbursementOrder` | [DE-DS-002](DOMAIN_EVENTS.md#48-disbursement-ds) | POL-DS-004 call the provider once under the stable identity required by [BR-DS-002](BUSINESS_RULES.md#11-disbursement-ds) | — | Disbursement Provider | Confirmed |
| E4 | Provider confirmation | CMD-DS-003 `ConfirmDisbursement` | `DisbursementOrder` | [DE-DS-003](DOMAIN_EVENTS.md#48-disbursement-ds) | POL-LB-002 activate only when amount, currency, and reservation reference match | [IE-DS-001](DOMAIN_EVENTS.md#58-disbursement-ds) | Loan Booking; Application Process; Communications; Audit Trail projection | Proposed |
| E5 | Confirmed-funds reaction | CMD-LB-002 `ActivateLoanAccount` | `LoanAccount` | [DE-LB-002](DOMAIN_EVENTS.md#47-loan-booking-lb) | POL-AP-005 complete the matching journey; only now does Customer become Borrower | [IE-LB-002](DOMAIN_EVENTS.md#57-loan-booking-lb) | Application Process; Communications; Audit Trail projection | Proposed |
| E6 | Activated-loan reaction | CMD-AP-004 `CompleteCreditApplication` | `CreditApplication` | [DE-AP-005](DOMAIN_EVENTS.md#41-application-process-ap) | POL-CO-005 send completion notification asynchronously | [IE-AP-004](DOMAIN_EVENTS.md#51-application-process-ap) | Communications; Audit Trail projection | Proposed |

## 9. Alternate and recovery paths

| Path | Owner | Detected fact | Permitted reaction | Forbidden shortcut | Retry / idempotency | Outcome |
| --- | --- | --- | --- | --- | --- | --- |
| Validation or consent failure | Application Process / Customer & Identity | Required data or purpose-bound consent is absent | Reject submission and remain `Draft`; collect missing evidence | Infer consent or enter identity processing | Repeated submission is safe and cannot create a second process under [BR-AP-002](BUSINESS_RULES.md#4-application-process-ap) | Correctable pre-submission state |
| Identity rejection | Customer & Identity | [DE-CI-005](DOMAIN_EVENTS.md#42-customer--identity-ci) | POL-AP-006 stop decisioning, expose only safe reason codes, and require explicit retry/reopen | Convert rejection into an unfavorable credit decision or leak provider evidence | Redelivery is deduplicated; a new verification needs an explicit operation | `IdentityRejected`; [IE-CI-002](DOMAIN_EVENTS.md#52-customer--identity-ci) is the proposed public fact |
| Identity provider unavailable | Customer & Identity | No cataloged domain event; operational fact remains to be named | Retry asynchronously, then isolate exhausted delivery/work for controlled review | Publish verified/rejected by inference or start assessment | Preserve verification identity; query ambiguous provider results before retry | `IdentityPending`; event naming remains a proposed hotspot |
| Assessment incomplete | Credit Decisioning | [DE-CD-002](DOMAIN_EVENTS.md#43-credit-decisioning-cd) records `PendingEvidence` | POL-AP-007 request only the missing evidence categories | Create a decision from incomplete inputs | New material evidence creates the governed assessment/version progression | `DecisionPending`; [IE-CD-001](DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Assessment temporarily unavailable | Credit Decisioning | Operational disposition `PendingRetry` | Retry after the safe hint while retaining immutable input identity | Treat technical unavailability as `Unfavorable` | Retry the same intended assessment; deduplicate delivery | `DecisionPending`; [IE-CD-002](DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Assessment technical inconsistency | Credit Decisioning | Operational disposition `OperationalException` | Stop automated progression and route the recovery reference to operations | Fabricate or mutate a Credit Decision | Recovery must be explicit and auditable | `DecisionPending`; [IE-CD-003](DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Unfavorable decision | Credit Decisioning | [DE-CD-003](DOMAIN_EVENTS.md#43-credit-decisioning-cd) records the completed unfavorable outcome | POL-AP-008 close the journey with applicant-safe reasons | Reclassify it as a provider failure or mutate the published decision | Reevaluation creates a new assessment and decision version | `CreditDeclined`; [IE-CD-005](DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Offer rejected | Application Process | CMD-AP-005 `RejectCreditOffer` records explicit applicant rejection against the active `offerId` | POL-AP-009 idempotently record [DE-AP-004](DOMAIN_EVENTS.md#41-application-process-ap), close the application offer path, and publish the proposed public fact if a consumer needs it | Mutate or redefine Credit Decisioning's immutable offer, or generate documents | Applicant-action identity and Inbox/idempotency prevent repeated effect | `OfferClosed`; [IE-AP-003](DOMAIN_EVENTS.md#51-application-process-ap) remains Proposed pending contract publication |
| Offer expired | Credit Decisioning | [DE-CD-006](DOMAIN_EVENTS.md#43-credit-decisioning-cd) | POL-AP-010 prevent acceptance and require an explicit reassessment/new-offer policy | Accept stale terms or silently recalculate a decision | Clock processing and redelivery are idempotent | `OfferClosed`; [IE-CD-007](DOMAIN_EVENTS.md#53-credit-decisioning-cd) |
| Document correction | Document Preparation | Applicant requests correction; [DE-DP-003](DOMAIN_EVENTS.md#44-document-preparation-dp) | CMD-DP-003 `RequestDocumentCorrection`; create a monotonic replacement package and invalidate the bound envelope | Overwrite approved bytes or reuse an obsolete envelope | Regeneration preserves prior versions and hashes | `DocumentsPending`; envelope may record [DE-ES-004](DOMAIN_EVENTS.md#45-electronic-signature-es) |
| OTP invalid or attempts exhausted | Communications | [DE-CO-005](DOMAIN_EVENTS.md#46-communications-co), including blocked indicator | POL-CO-002 increment attempts, block at limit, and allow only controlled reissue/manual review | Mark the package signed or reset attempts implicitly | Each challenge is single-use; reissue makes the old challenge unusable under [BR-CO-005](BUSINESS_RULES.md#9-communications-co) | `SignatureBlocked` or retryable challenge state |
| OTP expired | Communications | [DE-CO-006](DOMAIN_EVENTS.md#46-communications-co) | CMD-CO-004 `ReissueOtpChallenge` only under controlled policy | Validate or reuse the expired challenge | New challenge identity; old challenge remains unusable | Signature flow remains pending |
| Signature expired | Electronic Signature | [DE-ES-003](DOMAIN_EVENTS.md#45-electronic-signature-es) | POL-ES-003 require a new controlled envelope and challenge | Reopen or sign the expired envelope | Query an ambiguous provider result before deciding it expired | `SignaturePending` with a new flow |
| Disbursement transient failure | Disbursement | Retryable [DE-DS-004](DOMAIN_EVENTS.md#48-disbursement-ds) | POL-DS-002 retry with backoff and the same business idempotency key | Create a second order/key or activate the loan | Inbox/outbox deduplication and stable provider key | `DisbursementPending` |
| Disbursement terminal failure | Disbursement | Non-retryable or retry-exhausted [DE-DS-004](DOMAIN_EVENTS.md#48-disbursement-ds) | Stop automated progression, isolate the case, and permit only the approved `CMD-XS-002` owner-recovery request; reservation disposition remains a later decision | Create an Active Loan, reinterpret failure as decline, silently cancel/release the reservation, or reverse funds automatically | Duplicate terminal facts and recovery requests are harmless under their stable identities | `DisbursementFailed`; proposed [IE-DS-002](DOMAIN_EVENTS.md#58-disbursement-ds); [Q-009 policy](../workflows/DISBURSEMENT.md#10-compensation-and-manual-recovery-q-009-resolution) |
| Disbursement outcome unknown | Disbursement | [DE-DS-005](DOMAIN_EVENTS.md#48-disbursement-ds) after ambiguous submission | CMD-DS-004 `ReconcileDisbursementOutcome` using the original order/provider reference and stable key; retry only after confirmed absence, valid reservation, retryable classification, and available budget | Retry, fail, cancel, replace, or activate before authoritative reconciliation | Query by stable order/key; never create a new order/key | `DisbursementPending` until known; [Q-009 policy](../workflows/DISBURSEMENT.md#10-compensation-and-manual-recovery-q-009-resolution) |
| Activation failure after confirmed funds | Loan Booking | No cataloged domain event; operational fact remains to be named | POL-LB-004 enter critical reconciliation and retry activation | Automatically reverse funds or label the application declined | Activation command is idempotent; preserve matching confirmation | `ActivationPending` critical under [BR-LB-005](BUSINESS_RULES.md#10-loan-booking-lb); event naming remains a proposed hotspot |
| Consumer retry exhausted / DLQ | Owning consumer and platform operations | A versioned event repeatedly fails its consumer handler | CMD-XS-001 `RedriveDeadLetter` only after cause correction and compatibility review | Edit the business fact, bypass invariants, or blindly replay | Original event ID and Inbox evidence are preserved | Controlled redrive or explicit manual recovery |
| Duplicate or out-of-order delivery | Every consumer | Inbox already contains the event or aggregate version would regress | POL-XS-001 ignore safely and record an operational metric | Repeat the business effect or regress aggregate state | [BR-XS-004](BUSINESS_RULES.md#12-cross-cutting-xs) governs deduplication | Business state unchanged |
| Manual recovery | Platform Operator plus owning context | A diagnosed case requires an operation not safely automated | CMD-XS-002 `RequestManualRecovery` identifies authorized actor, owner, application/loan/order/correlation refs, diagnosis, evidence, requested operation, idempotency identity, audit reason, and timestamp; DS reconciles/recovers transfer and LB recovers activation only through existing owner paths | Direct queue/message/database edits, distributed rollback, fabricated success, automatic reversal, silent reservation cancellation/release, activation, decline, or completion | Request and owner operation are unique, traceable, auditable, and preserve original business identity | Existing recovery path or isolated terminal case awaiting a later governed disposition; [approved policy](../workflows/DISBURSEMENT.md#10-compensation-and-manual-recovery-q-009-resolution) |

## 10. Cross-cutting policies

| Policy | Status | Trigger | Required behavior | Governing rule |
| --- | --- | --- | --- | --- |
| POL-XS-002 `StagePublicFactAtomically` | Confirmed | A producer commits a public business fact | Persist aggregate change and Outbox record in the same local transaction | [BR-XS-003](BUSINESS_RULES.md#12-cross-cutting-xs) |
| POL-XS-003 `MinimizePublicPayload` | Confirmed | A domain fact maps to an integration event | Publish only allowlisted identifiers and minimum snapshots; exclude secrets and protected evidence | [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) |
| POL-XS-004 `ProjectAuditAsynchronously` | Confirmed | A sanitized business or security fact is available | Append to Audit Trail without making audit a synchronous workflow dependency | [BR-XS-005](BUSINESS_RULES.md#12-cross-cutting-xs) |
| POL-XS-005 `CompensateExplicitly` | Confirmed | A completed cross-context action requires recovery | Issue an owned, traceable business command rather than simulate distributed rollback | [BR-XS-006](BUSINESS_RULES.md#12-cross-cutting-xs) |

## 11. Hotspots

The source register uses the types `Assumption` and `Open question`, while this catalog uses only its four lifecycle statuses. A hotspot inferred from a working assumption is `Derived`: this records the need for validation, not that the assumption is true. An unresolved question is `Proposed`: it identifies a decision candidate and does not imply approval. Explicitly out-of-scope behavior is `Deferred`.

| ID | Status | Owner / decision maker | Topic | Impact | Decision required | Source |
| --- | --- | --- | --- | --- | --- | --- |
| HS-001 | Confirmed | Application Process and Credit Decisioning | Offer acceptance/rejection ownership | Establishes the command aggregate, private fact producer, public producer, and invariants for progression to documents | Resolved 2026-08-14: Application Process records the applicant's acceptance or explicit rejection, exact `offerId`, accepted canonical `termsHash`, action timestamp, stage history, and idempotency. Credit Decisioning retains the immutable `CreditOffer`, canonical terms, creation, expiry, and supersession. | [Ownership resolution](DOMAIN_EVENTS.md#49-cross-cutting-xs), [P-005](../discovery/ASSUMPTIONS.md#confirmed-product-decisions) |
| HS-002 | Deferred | Credit Decisioning / product governance | Manual credit review | Changes dispositions, analyst authority, process timers, and reopening | Define a future manual-review state and authority before implementation | [Deferred rules](BUSINESS_RULES.md#13-deferred-rules) |
| HS-003 | Derived | Platform architecture | EventBridge, SQS, Outbox, and Inbox adequacy | Affects delivery isolation, cost, ordering, and operational recovery | Validate delivery and persistence assumptions under load and failure | [H-004 and H-005](../discovery/ASSUMPTIONS.md#working-assumptions-requiring-later-validation) |
| HS-004 | Confirmed | Disbursement and operations | Recovery after repeated transfer failure | Defines operator authority while leaving reservation disposition outside the approved action | Resolved 2026-08-14: use `CMD-XS-002`; reconcile ambiguity first; retry only after confirmed absence under the same order/key and valid budget/reservation; otherwise isolate. No fabricated success, silent reservation mutation, activation, decline, completion, or automatic reversal. | [Disbursement Workflow](../workflows/DISBURSEMENT.md#10-compensation-and-manual-recovery-q-009-resolution) |
| HS-005 | Proposed | Security and contract governance | Sensitive-field contract allowlists | Affects public payload schemas and every sensitive copy | AWS Demo classification, minimization, masking, prohibited data, and retention are defined; complete field-by-field event allowlists before contract publication | [Security Model](../architecture/SECURITY_MODEL.md#6-data-classification-minimization-and-prohibited-data), [Q-004](../discovery/ASSUMPTIONS.md#open-questions) |
| HS-006 | Proposed | Customer & Identity and Loan Booking | Unnamed operational failure facts | Provider unavailability and post-disbursement activation failure need traceable semantics without being invented here | Decide whether each fact needs a private domain event and name it in the event catalog | [Event governance](DOMAIN_EVENTS.md#7-governance) |

## 12. Open questions

| ID | Status | Owner / decision maker | Topic | Impact | Decision required | Source |
| --- | --- | --- | --- | --- | --- | --- |
| OQ-001 | Proposed | Application Process architecture | Saga persistence and timers | Determines persisted coordinator state, timeout commands, and replay behavior | Choose choreography-only policies or explicit saga steps/timers | [Q-005](../discovery/ASSUMPTIONS.md#open-questions) |
| OQ-002 | Proposed | Customer & Identity | Identity validity across reassessment | Determines whether a new assessment may reuse a verification | Define exact expiration and renewal policy | [Q-006](../discovery/ASSUMPTIONS.md#open-questions) |
| OQ-003 | Proposed | Loan Booking | Schedule dates and rounding | Blocks normative schedule generation and contract examples | Define first-installment date and rounding conventions | [Q-007](../discovery/ASSUMPTIONS.md#open-questions) |
| OQ-004 | Proposed | Product, Communications, and UI governance | Customer-safe reason disclosure | Affects decline, identity, and failure messages | Classify codes that may be shown directly versus translated | [Q-008](../discovery/ASSUMPTIONS.md#open-questions) |
| OQ-005 | Confirmed | Operations and Disbursement | Repeated-failure manual action | Establishes authorized reconciliation/retry and isolation without inventing a new event or compensation | Resolved 2026-08-14 by the evidenced, idempotent, audited `CMD-XS-002` policy; reservation cancellation/release and later terminal disposition remain separate future decisions | [Disbursement Workflow](../workflows/DISBURSEMENT.md#10-compensation-and-manual-recovery-q-009-resolution) |
| OQ-006 | Confirmed | Security and platform architecture | AWS Demo retention and DLQ treatment | Establishes protected-artifact, OTP, audit, operational-record, log, and DLQ maxima plus teardown | Resolved 2026-08-14: use the category matrix, scheduled mechanisms, explicit teardown, and service-specific verification; production policy remains deferred | [Security Model](../architecture/SECURITY_MODEL.md#10-aws-demo-retention-and-verified-deletion) |

## 13. Governance

1. Commands and policies never bypass the aggregate invariants in `BUSINESS_RULES.md`.
2. Application Process owns coordination and coarse journey state only; capability contexts own their decisions and protected evidence.
3. Operational dispositions, identity rejection, unfavorable credit decisions, and transport failures remain distinct outcomes.
4. A public integration event requires explicit domain-to-contract mapping, payload minimization, compatibility review, and a schema in `loan-platform-contracts` before publication.
5. Retries preserve business identity; unknown provider outcomes are reconciled before another side effect is attempted.
6. Compensation and manual recovery are explicit authorized commands, never database edits or distributed rollback.
7. Stable IDs are never reused. A status or ownership change must update the owning canonical source and any affected ADR.
8. Open questions and hotspots are not implementation defaults. They close only through an approved source update and, where architectural, an ADR.
