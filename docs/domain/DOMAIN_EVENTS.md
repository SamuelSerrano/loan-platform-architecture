# Domain and Integration Event Catalog

**Product:** Loan Onboarding & Credit Decisioning Platform
**Status:** Domain documentation baseline
**Canonical language:** English

## 1. Purpose and authority

This document is the canonical transversal semantic catalog of events for the loan-platform domain. It assigns stable documentation IDs, separates private domain facts from public cross-context contracts, and records the minimum information required to interpret each fact.

This catalog does not define transport schemas. Future AsyncAPI and JSON Schema contracts belong to `loan-platform-contracts`. The [bounded-context map](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#3-bounded-context-map) remains authoritative for ownership and relationships, while the [Credit Decisioning design](CREDIT_DECISIONING_DESIGN.md#11-domain-events) remains authoritative for that context's internal implementation design.

## 2. Classification

- A **Domain Event** is a completed fact meaningful inside one bounded context. It may remain private and may carry domain-native types.
- An **Integration Event** is a sanitized public fact deliberately published for another context. Its contract name is versioned and its payload is an allowlist, not a serialized aggregate.
- A domain event and an integration event may describe the same business transition, but they are distinct messages with distinct responsibilities. Publication occurs only after an explicit mapping at the producer boundary.
- Commands and provider callbacks are not events. They may be rejected, whereas every event in this catalog records something that already happened.

## 3. Status and versioning

| Status | Meaning |
| --- | --- |
| `Confirmed` | The event name and meaning are explicitly established by an existing source. |
| `Derived` | The private fact is a necessary implication of established state or behavior; its derivation is explained. |
| `Proposed` | The event ownership, name, or semantics require explicit approval before implementation or contract publication. |
| `Deferred` | The event is recognized but intentionally excluded from the current baseline. |

Stable `DE-*` and `IE-*` IDs identify catalog entries and are never reused. Internal domain events are unversioned implementation facts. Public integration-event contract names always end in `.v1`. Additive optional fields are permitted within a major version only under the compatibility policy defined in `loan-platform-contracts`; removing a field or changing its type or semantics requires a new major contract version. Deprecation overlap and retirement windows are deferred to that contracts repository.

## 4. Domain events

For every entry in this section, consumers are internal handlers or projections and remain an implementation detail of the producing context. Private payload exclusions are governed by that context boundary and its protected-data rules. Domain events are unversioned; only their stable documentation IDs are governed here.

### 4.1 Application Process (`AP`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-AP-001 | `CreditApplicationStarted` | Confirmed | `CreditApplication` | A valid start command creates the application in `Draft` | Application ID, application number, selected product reference, started instant | [BR-AP-002](BUSINESS_RULES.md#4-application-process-ap) | [Phase A](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-a--application-and-identity) |
| DE-AP-002 | `CreditApplicationSubmitted` | Confirmed | `CreditApplication` | Complete inputs and consent references are frozen for evaluation | Application ID, customer ID, product ID, consent references, submission instant | [BR-AP-001](BUSINESS_RULES.md#4-application-process-ap), [BR-AP-003](BUSINESS_RULES.md#4-application-process-ap) | [Application stages](UBIQUITOUS_LANGUAGE.md#canonical-application-stages) |
| DE-AP-003 | `CreditOfferAccepted` | Proposed | `CreditApplication` | Applicant accepts the active, unexpired offer and exact terms hash | Application ID, offer ID, accepted terms hash, accepted instant | [BR-AP-006](BUSINESS_RULES.md#4-application-process-ap) | [Phase B](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-b--decision-and-offer) |
| DE-AP-004 | `CreditOfferRejected` | Proposed | `CreditApplication` | Applicant explicitly rejects the active offer | Application ID, offer ID, rejected instant | — | [Application stages](UBIQUITOUS_LANGUAGE.md#canonical-application-stages) |
| DE-AP-005 | `CreditApplicationCompleted` | Confirmed | `CreditApplication` | The matching loan account has activated and the journey completes | Application ID, customer ID, loan account ID, completed instant | — | [Phase E](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-e--loan-booking-and-disbursement) |
| DE-AP-006 | `CreditApplicationCancelled` | Confirmed | `CreditApplication` | An authorized cancellation succeeds before the signing boundary | Application ID, prior stage, cancelled instant, safe reason code | [BR-AP-004](BUSINESS_RULES.md#4-application-process-ap) | [Application stages](UBIQUITOUS_LANGUAGE.md#canonical-application-stages) |

### 4.2 Customer & Identity (`CI`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-CI-001 | `CustomerProfileRegistered` | Confirmed | `Customer` | A valid customer profile is registered | Customer ID, profile version, registered instant | — | [Phase A](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-a--application-and-identity) |
| DE-CI-002 | `CustomerConsentGranted` | Confirmed | `ConsentRecord` | Customer grants one explicit purpose and text version | Customer ID, consent record ID, purpose, text version, granted instant | [BR-CI-001](BUSINESS_RULES.md#5-customer--identity-ci) | [Phase A](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-a--application-and-identity) |
| DE-CI-003 | `IdentityVerificationStarted` | Confirmed | `IdentityVerification` | A verification case starts for the required level | Customer ID, application ID, verification ID, required level, started instant | — | [Phase A](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-a--application-and-identity) |
| DE-CI-004 | `CustomerIdentityVerified` | Confirmed | `IdentityVerification` | Provider-neutral evidence satisfies the required verification level | Customer ID, application ID, verification ID, verification level, verified instant, valid-until | [BR-CI-003](BUSINESS_RULES.md#5-customer--identity-ci), [BR-CI-004](BUSINESS_RULES.md#5-customer--identity-ci) | [Identity vocabulary](UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |
| DE-CI-005 | `CustomerIdentityRejected` | Confirmed | `IdentityVerification` | Completed verification does not satisfy identity requirements | Customer ID, application ID, verification ID, safe reason codes, rejected instant | [BR-CI-003](BUSINESS_RULES.md#5-customer--identity-ci), [BR-CI-004](BUSINESS_RULES.md#5-customer--identity-ci) | [Identity vocabulary](UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |

### 4.3 Credit Decisioning (`CD`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-CD-001 | `CreditAssessmentStarted` | Confirmed | `CreditAssessment` | A valid immutable assessment snapshot and policy versions are accepted | Assessment ID, application ID, assessment version, snapshot hash, policy and formula versions | [BR-CD-001](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Domain events](CREDIT_DECISIONING_DESIGN.md#11-domain-events) |
| DE-CD-002 | `AssessmentDispositionRecorded` | Confirmed | `CreditAssessment` | A prerequisite or operational condition prevents safe completion | Assessment ID, disposition, safe reason codes, recorded instant | [BR-CD-002](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Domain events](CREDIT_DECISIONING_DESIGN.md#11-domain-events) |
| DE-CD-003 | `CreditDecisionRecorded` | Confirmed | `CreditAssessment` | Deterministic evaluation completes with exactly one credit outcome | Assessment ID, decision ID, outcome, reason codes, policy versions, decided instant | [BR-CD-001](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-CD-003](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Domain events](CREDIT_DECISIONING_DESIGN.md#11-domain-events) |
| DE-CD-004 | `OfferAlternativesCalculated` | Confirmed | `CreditAssessment` | A favorable evaluation produces eligible distinct alternatives | Assessment ID, decision ID, alternative IDs, terms hashes | [BR-CD-004](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-CD-005](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Domain events](CREDIT_DECISIONING_DESIGN.md#11-domain-events) |
| DE-CD-005 | `CreditOfferCreated` | Confirmed | `CreditOffer` | A selected alternative from a favorable decision becomes the active offer | Offer ID, application ID, decision and alternative IDs, terms hash, expiry instant | [BR-CD-006](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Domain events](CREDIT_DECISIONING_DESIGN.md#11-domain-events) |
| DE-CD-006 | `CreditOfferExpired` | Confirmed | `CreditOffer` | Current time reaches an active offer's expiry | Offer ID, application ID, expiry instant | [BR-CD-006](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Expire credit offer](CREDIT_DECISIONING_DESIGN.md#uc-cd-005--expire-credit-offer) |
| DE-CD-007 | `CreditOfferSuperseded` | Confirmed | `CreditOffer` | A valid reassessment creates a replacement for an unaccepted offer | Prior offer ID, replacement offer ID, application ID, superseded instant | [BR-CD-007](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Supersede credit offer](CREDIT_DECISIONING_DESIGN.md#uc-cd-006--supersede-credit-offer) |

### 4.4 Document Preparation (`DP`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-DP-001 | `DocumentPackageGenerated` | Confirmed | `DocumentPackage` | Artifacts are generated from one accepted-offer snapshot and template set | Application ID, offer ID, package ID, package version, artifact references and hashes | [BR-DP-001](BUSINESS_RULES.md#7-document-preparation-dp), [BR-DP-004](BUSINESS_RULES.md#7-document-preparation-dp) | [Phase C](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-c--documents-and-approval) |
| DE-DP-002 | `DocumentPackageApproved` | Confirmed | `DocumentPackage` | Applicant approves the reviewed immutable package | Application ID, package ID, package version, package hash, approved instant | [BR-DP-002](BUSINESS_RULES.md#7-document-preparation-dp) | [Document Preparation vocabulary](UBIQUITOUS_LANGUAGE.md#8-document-preparation-vocabulary) |
| DE-DP-003 | `DocumentCorrectionRequested` | Confirmed | `DocumentPackage` | Applicant requests correction of package data or contents | Application ID, package ID, package version, requested instant, safe correction category | [BR-DP-003](BUSINESS_RULES.md#7-document-preparation-dp), [BR-DP-005](BUSINESS_RULES.md#7-document-preparation-dp) | [Phase C](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-c--documents-and-approval) |

### 4.5 Electronic Signature (`ES`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-ES-001 | `SignatureEnvelopeCreated` | Confirmed | `SignatureEnvelope` | An envelope is created for one approved package, signer, and purpose | Envelope ID, application ID, package ID and version, package hash, signer reference, purpose, expiry | [BR-ES-001](BUSINESS_RULES.md#8-electronic-signature-es) | [Phase C](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-c--documents-and-approval) |
| DE-ES-002 | `DocumentPackageSigned` | Confirmed | `SignatureEnvelope` | All required signatures and tamper-evident evidence are recorded | Envelope ID, application ID, package ID and version, signer references, evidence reference, signed instant | [BR-ES-002](BUSINESS_RULES.md#8-electronic-signature-es), [BR-ES-003](BUSINESS_RULES.md#8-electronic-signature-es) | [Electronic Signature vocabulary](UBIQUITOUS_LANGUAGE.md#9-electronic-signature-vocabulary) |
| DE-ES-003 | `SignatureEnvelopeExpired` | Derived | `SignatureEnvelope` | The envelope reaches its terminal signature expiry before completion | Envelope ID, package ID and version, expired instant | [BR-ES-004](BUSINESS_RULES.md#8-electronic-signature-es) | [Electronic Signature vocabulary](UBIQUITOUS_LANGUAGE.md#9-electronic-signature-vocabulary) |
| DE-ES-004 | `SignatureEnvelopeInvalidated` | Derived | `SignatureEnvelope` | Its bound package is corrected, superseded, or otherwise invalidated | Envelope ID, package ID and version, invalidated instant, safe reason | [BR-ES-005](BUSINESS_RULES.md#8-electronic-signature-es) | [Document Preparation vocabulary](UBIQUITOUS_LANGUAGE.md#8-document-preparation-vocabulary) |

**Derivation note:** `SignatureEnvelopeExpired` and `SignatureEnvelopeInvalidated` name the private facts required to represent the explicitly established terminal expiry and package-invalidation transitions. No public contract is implied.

### 4.6 Communications (`CO`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-CO-001 | `OtpChallengeIssued` | Confirmed | `OtpChallenge` | A time-bound challenge is created and bound to signer, purpose, and envelope | Challenge ID, signer reference, envelope ID, purpose, masked destination, issued and expiry instants | [BR-CO-001](BUSINESS_RULES.md#9-communications-co), [BR-CO-002](BUSINESS_RULES.md#9-communications-co), [BR-CO-003](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |
| DE-CO-002 | `OtpDelivered` | Confirmed | `NotificationDelivery` | A channel adapter records successful delivery | Challenge ID, delivery ID, channel, masked destination, delivered instant | [BR-CO-004](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |
| DE-CO-003 | `OtpDeliveryFailed` | Confirmed | `NotificationDelivery` | A delivery attempt fails | Challenge ID, delivery ID, channel, retryability, safe reason code, failed instant | [BR-CO-004](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |
| DE-CO-004 | `OtpValidated` | Confirmed | `OtpChallenge` | Submitted value matches an active eligible challenge | Challenge ID, signer reference, envelope ID, purpose, validated instant, authorization reference | [BR-CO-001](BUSINESS_RULES.md#9-communications-co), [BR-CO-002](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |
| DE-CO-005 | `OtpValidationFailed` | Confirmed | `OtpChallenge` | Submitted value fails validation and the attempt is recorded | Challenge ID, attempt count, attempts remaining or blocked indicator, failed instant | [BR-CO-001](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |
| DE-CO-006 | `OtpExpired` | Confirmed | `OtpChallenge` | Current time reaches the unused challenge's expiry | Challenge ID, envelope ID, expired instant | [BR-CO-001](BUSINESS_RULES.md#9-communications-co) | [Phase D](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-d--otp-and-signature) |

### 4.7 Loan Booking (`LB`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-LB-001 | `LoanAccountReserved` | Confirmed | `LoanAccount` | A valid signed package and matching contractual terms reserve the pending loan | Application ID, loan account ID, offer ID, signed package reference, amount, currency, destination token/reference | [BR-LB-001](BUSINESS_RULES.md#10-loan-booking-lb), [BR-LB-002](BUSINESS_RULES.md#10-loan-booking-lb), [BR-LB-003](BUSINESS_RULES.md#10-loan-booking-lb) | [Phase E](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-e--loan-booking-and-disbursement) |
| DE-LB-002 | `LoanAccountActivated` | Confirmed | `LoanAccount` | Matching confirmed funds make the reserved obligation effective | Application ID, loan account ID, account number/token, schedule version, activated instant | [BR-LB-004](BUSINESS_RULES.md#10-loan-booking-lb) | [Loan Booking vocabulary](UBIQUITOUS_LANGUAGE.md#11-loan-booking-vocabulary) |

### 4.8 Disbursement (`DS`)

| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DE-DS-001 | `DisbursementOrderCreated` | Confirmed | `DisbursementOrder` | A validated loan reservation authorizes one transfer order | Order ID, application ID, loan account ID, amount, currency, destination token/reference, idempotency key reference | [BR-DS-001](BUSINESS_RULES.md#11-disbursement-ds), [BR-DS-004](BUSINESS_RULES.md#11-disbursement-ds) | [Phase E](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-e--loan-booking-and-disbursement) |
| DE-DS-002 | `DisbursementRequested` | Confirmed | `DisbursementOrder` | The order submits an attempt using its stable idempotency identity | Order ID, attempt ID, loan account ID, amount, currency, submitted instant | [BR-DS-002](BUSINESS_RULES.md#11-disbursement-ds) | [Phase E](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-e--loan-booking-and-disbursement) |
| DE-DS-003 | `CreditDisbursed` | Confirmed | `DisbursementOrder` | Provider confirmation matches the intended transfer | Order ID, application ID, loan account ID, amount, currency, provider reference, confirmed instant | [BR-DS-001](BUSINESS_RULES.md#11-disbursement-ds), [BR-LB-004](BUSINESS_RULES.md#10-loan-booking-lb) | [Disbursement vocabulary](UBIQUITOUS_LANGUAGE.md#12-disbursement-vocabulary) |
| DE-DS-004 | `CreditDisbursementFailed` | Confirmed | `DisbursementOrder` | The order reaches a classified transfer failure | Order ID, application ID, loan account ID, retryability, safe reason code, failed instant | [BR-DS-002](BUSINESS_RULES.md#11-disbursement-ds), [BR-DS-005](BUSINESS_RULES.md#11-disbursement-ds) | [Phase E](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#phase-e--loan-booking-and-disbursement) |
| DE-DS-005 | `DisbursementOutcomeMarkedUnknown` | Derived | `DisbursementOrder` | A submitted provider interaction has an ambiguous or unavailable result | Order ID, attempt ID, provider reference when available, recorded instant | [BR-DS-003](BUSINESS_RULES.md#11-disbursement-ds) | [Disbursement vocabulary](UBIQUITOUS_LANGUAGE.md#12-disbursement-vocabulary) |

**Derivation note:** `DisbursementOutcomeMarkedUnknown` names the private fact needed to retain the explicitly defined `Unknown Outcome` before reconciliation. It does not authorize retry or declare transfer failure.

### 4.9 Cross-cutting (`XS`)

Cross-cutting integration mechanics do not own a business aggregate and therefore define no domain events. Outbox, Inbox, Retry, DLQ, and Audit Trail records are operational mechanisms or projections, not new business facts.

**Ownership note:** The legacy Event Storming table places offer acceptance and rejection commands on `CreditOffer`, whose lifecycle is owned by Credit Decisioning. The specialized Credit Decisioning design instead excludes acceptance from that aggregate and states that Application Process records the applicant action. Therefore the private acceptance/rejection facts and their AP-produced public candidates remain `Proposed` until this discrepancy is explicitly resolved; this catalog does not silently choose a new owner.

## 5. Integration events

Each `Proposed` entry below preserves a name already present in the transversal candidate catalog and adds the required `.v1` suffix. Its producer and consumers are also proposed metadata unless a narrower source explicitly establishes them. Credit Decisioning contracts are `Confirmed` because their `.v1` names and semantics are explicitly defined in its specialized design.

### 5.1 Application Process (`AP`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-AP-001 | `CreditApplicationSubmitted.v1` | Proposed | Application Process | Customer & Identity; Audit Trail projection | `CreditApplicationSubmitted` commits | Event envelope; application ID; customer ID; product ID; consent references; submission instant | Full PII; consent text; raw identity evidence; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-AP-001](BUSINESS_RULES.md#4-application-process-ap), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-AP-002 | `CreditOfferAccepted.v1` | Proposed | Application Process | Document Preparation; Audit Trail projection | `CreditOfferAccepted` commits | Event envelope; application ID; offer ID; accepted instant; accepted terms hash | Full PII; document bytes; raw financial evidence; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-AP-006](BUSINESS_RULES.md#4-application-process-ap), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-AP-003 | `CreditOfferRejected.v1` | Proposed | Application Process | Credit Decisioning; Audit Trail projection | `CreditOfferRejected` commits | Event envelope; application ID; offer ID; rejected instant | Full PII; raw financial evidence; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-AP-004 | `CreditApplicationCompleted.v1` | Proposed | Application Process | Communications; Audit Trail projection | `CreditApplicationCompleted` commits | Event envelope; application ID; customer ID; loan account ID; completion instant | Full PII; document bytes; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.2 Customer & Identity (`CI`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-CI-001 | `CustomerIdentityVerified.v1` | Proposed | Customer & Identity | Application Process; Audit Trail projection | `CustomerIdentityVerified` commits | Event envelope; application and customer IDs; verification ID; provider-neutral level; verified instant; valid-until | Raw identity evidence; full PII; provider-specific payloads; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-CI-003](BUSINESS_RULES.md#5-customer--identity-ci), [BR-CI-004](BUSINESS_RULES.md#5-customer--identity-ci), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-CI-002 | `CustomerIdentityRejected.v1` | Proposed | Customer & Identity | Application Process; Communications; Audit Trail projection | `CustomerIdentityRejected` commits | Event envelope; application and customer IDs; verification ID; safe reason codes; rejected instant | Raw identity evidence; full PII; restricted fraud details; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-CI-003](BUSINESS_RULES.md#5-customer--identity-ci), [BR-CI-004](BUSINESS_RULES.md#5-customer--identity-ci), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.3 Credit Decisioning (`CD`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-CD-001 | `CreditAssessmentPendingEvidence.v1` | Confirmed | Credit Decisioning | Application Process | Assessment records `PendingEvidence` disposition | Event envelope; assessment and application IDs; safe reason codes; required evidence categories | Raw income or identity evidence; full PII; provider payloads; provider credentials | Existing `.v1` semantic baseline | [BR-CD-002](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Integration events](CREDIT_DECISIONING_DESIGN.md#12-integration-events) |
| IE-CD-002 | `CreditAssessmentPendingRetry.v1` | Confirmed | Credit Decisioning | Application Process | Assessment records `PendingRetry` disposition | Event envelope; assessment and application IDs; generic code; retry-after hint | Provider error body; raw evidence; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-002](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Integration events](CREDIT_DECISIONING_DESIGN.md#12-integration-events) |
| IE-CD-003 | `CreditAssessmentOperationalExceptionRecorded.v1` | Confirmed | Credit Decisioning | Application Process | Assessment records `OperationalException` disposition | Event envelope; assessment and application IDs; generic code; recovery reference | Internal rule trace; raw evidence; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-002](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Integration events](CREDIT_DECISIONING_DESIGN.md#12-integration-events) |
| IE-CD-004 | `FavorableCreditDecisionRecorded.v1` | Confirmed | Credit Decisioning | Application Process | `CreditDecisionRecorded` commits a favorable outcome | Event envelope; assessment, application and decision IDs; risk band; segment; sanitized alternatives; policy versions | Raw evidence; internal rule trace; restricted fraud details; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-003](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-CD-004](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Integration events](CREDIT_DECISIONING_DESIGN.md#12-integration-events) |
| IE-CD-005 | `UnfavorableCreditDecisionRecorded.v1` | Confirmed | Credit Decisioning | Application Process | `CreditDecisionRecorded` commits an unfavorable outcome | Event envelope; assessment, application and decision IDs; applicant-safe reason codes; policy version | Raw evidence; internal rule trace; restricted fraud details; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-003](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Integration events](CREDIT_DECISIONING_DESIGN.md#12-integration-events) |
| IE-CD-006 | `CreditOfferCreated.v1` | Confirmed | Credit Decisioning | Application Process | `CreditOfferCreated` and outbox commit atomically | Event envelope; application, assessment, decision, alternative and offer IDs; immutable terms snapshot and hash; expiry; policy versions | Raw evidence; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-006](BUSINESS_RULES.md#6-credit-decisioning-cd), [BR-XS-003](BUSINESS_RULES.md#12-cross-cutting-xs) | [Create credit offer](CREDIT_DECISIONING_DESIGN.md#uc-cd-004--create-credit-offer) |
| IE-CD-007 | `CreditOfferExpired.v1` | Confirmed | Credit Decisioning | Application Process | `CreditOfferExpired` commits | Event envelope; application and offer IDs; expiry instant | Raw evidence; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-006](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Expire credit offer](CREDIT_DECISIONING_DESIGN.md#uc-cd-005--expire-credit-offer) |
| IE-CD-008 | `CreditOfferSuperseded.v1` | Confirmed | Credit Decisioning | Application Process | `CreditOfferSuperseded` commits | Event envelope; application ID; prior and replacement offer IDs; superseded instant | Raw evidence; full PII; provider credentials | Existing `.v1` semantic baseline | [BR-CD-007](BUSINESS_RULES.md#6-credit-decisioning-cd) | [Supersede credit offer](CREDIT_DECISIONING_DESIGN.md#uc-cd-006--supersede-credit-offer) |

### 5.4 Document Preparation (`DP`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-DP-001 | `DocumentPackageGenerated.v1` | Proposed | Document Preparation | Application Process; Audit Trail projection | `DocumentPackageGenerated` commits | Event envelope; application, offer, and package IDs; package version; artifact references and hashes | Document bytes; full PII; template contents; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-DP-001](BUSINESS_RULES.md#7-document-preparation-dp), [BR-DP-004](BUSINESS_RULES.md#7-document-preparation-dp), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-DP-002 | `DocumentPackageApproved.v1` | Proposed | Document Preparation | Electronic Signature; Application Process; Audit Trail projection | `DocumentPackageApproved` commits | Event envelope; application and package IDs; package version; package hash; approval instant | Document bytes; full PII; signature evidence; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-DP-002](BUSINESS_RULES.md#7-document-preparation-dp), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.5 Electronic Signature (`ES`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-ES-001 | `DocumentPackageSigned.v1` | Proposed | Electronic Signature | Loan Booking; Application Process; Audit Trail projection | `DocumentPackageSigned` commits | Event envelope; application, package and envelope IDs; package version; signer references; evidence reference; signed instant | OTP secrets; document bytes; full PII; signature-provider payload; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-ES-002](BUSINESS_RULES.md#8-electronic-signature-es), [BR-ES-003](BUSINESS_RULES.md#8-electronic-signature-es), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.6 Communications (`CO`)

OTP and notification facts remain private for the baseline. No `IE-CO-*` contract is authorized until another bounded context demonstrates a business need that cannot be met through the established signing outcome or a targeted API.

### 5.7 Loan Booking (`LB`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-LB-001 | `LoanAccountReserved.v1` | Proposed | Loan Booking | Disbursement; Application Process; Audit Trail projection | `LoanAccountReserved` commits | Event envelope; application, loan account, and offer IDs; amount and currency; destination token/reference | Raw destination credentials; document bytes; full PII; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-LB-001](BUSINESS_RULES.md#10-loan-booking-lb), [BR-LB-002](BUSINESS_RULES.md#10-loan-booking-lb), [BR-LB-003](BUSINESS_RULES.md#10-loan-booking-lb), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-LB-002 | `LoanAccountActivated.v1` | Proposed | Loan Booking | Application Process; Communications; Audit Trail projection | `LoanAccountActivated` commits | Event envelope; application and loan account IDs; account number/token; schedule version; activation instant | Full repayment schedule; full PII; document bytes; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-LB-004](BUSINESS_RULES.md#10-loan-booking-lb), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.8 Disbursement (`DS`)

| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IE-DS-001 | `CreditDisbursed.v1` | Proposed | Disbursement | Loan Booking; Application Process; Communications; Audit Trail projection | `CreditDisbursed` commits | Event envelope; application, loan account, and order IDs; amount; currency; safe provider reference; confirmed instant | Raw destination credentials; full PII; provider response; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-DS-001](BUSINESS_RULES.md#11-disbursement-ds), [BR-LB-004](BUSINESS_RULES.md#10-loan-booking-lb), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |
| IE-DS-002 | `CreditDisbursementFailed.v1` | Proposed | Disbursement | Loan Booking; Application Process; operations projection; Audit Trail projection | `CreditDisbursementFailed` reaches terminal failure | Event envelope; application, loan account, and order IDs; retryability; safe reason code; failed instant | Provider error body; raw destination credentials; full PII; provider credentials | Initial public contract; incompatible change requires `.v2` | [BR-DS-005](BUSINESS_RULES.md#11-disbursement-ds), [BR-XS-002](BUSINESS_RULES.md#12-cross-cutting-xs) | [Public candidate catalog](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#12-public-integration-event-catalog-v0) |

### 5.9 Cross-cutting (`XS`)

Cross-cutting components route and project events but do not publish a generic business integration event. Audit Trail remains a downstream sanitized projection; DLQ movement, retries, and outbox delivery are operational telemetry rather than public domain contracts.

## 6. Envelope and sensitive-data constraints

Every integration event carries an Event Envelope with, at minimum: `eventId`, `eventType`, `eventVersion`, `occurredAt`, `aggregateId`, `correlationId`, `causationId`, `producer`, and `traceId`. Payload entries listed above are additional business data. Producers persist the business change and Outbox record atomically; consumers use Inbox/idempotency controls for at-least-once delivery.

Payloads must never contain OTP secrets, raw identity evidence, full PII, document bytes, or provider credentials. They also exclude raw provider payloads, raw bank credentials, unrestricted fraud details, internal rule traces, and secrets embedded in logs or metadata. Opaque references and minimum immutable snapshots are permitted only when required by the consumer's business action.

## 7. Governance

1. The producing bounded context owns the event semantics and maps private facts to public allowlisted payloads.
2. A new consumer does not automatically justify expanding a public event; prefer references, projections, or an owning API when they reduce sensitive-data propagation.
3. Every published event is staged through the transactional Outbox, and every consumer handles duplicate or out-of-order delivery idempotently.
4. Documentation IDs remain stable through wording improvements and contract-version changes; retired IDs are never reassigned.
5. `Derived` facts require a continuing documented invariant. `Proposed` integration events are not implementation commitments and cannot be treated as published contracts.
6. Domain events are not sent directly to EventBridge. Public contracts require explicit mapping, security review, compatibility policy, and schemas in `loan-platform-contracts`.
7. Operational failures, identity rejection, and unfavorable credit decisions remain distinct semantic outcomes and must never be translated into one another.
