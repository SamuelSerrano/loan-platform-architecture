# Canonical Business Rules

**Product:** Loan Onboarding & Credit Decisioning Platform
**Status:** Baseline 0.1
**Date:** 2026-08-12
**Canonical language:** English
**Scope:** Transversal business rules and aggregate invariants for the consumer-loan origination MVP

## 1. Purpose and authority

This document is the canonical transversal catalog of business rules and aggregate invariants. Stable IDs are the references used by operational models, events, tests, and future contracts. The owning bounded context remains responsible for enforcing each rule.

This catalog does not redefine calculations, thresholds, matrices, policy parameters, or the internal design of Credit Decisioning. [`PRODUCT_AND_CREDIT_POLICY.md`](PRODUCT_AND_CREDIT_POLICY.md) explains policy intent, [`CREDIT_DECISION_TABLE.md`](CREDIT_DECISION_TABLE.md) is authoritative for deterministic Credit Decisioning behavior, and [`CREDIT_DECISIONING_DESIGN.md`](CREDIT_DECISIONING_DESIGN.md) is authoritative for that context's internal design. If a summary here conflicts with a specialized source, the specialized source takes precedence.

## 2. Reading conventions

- IDs follow `BR-<context>-NNN`; IDs remain stable and are never reused for another rule.
- Context codes are `AP`, `CI`, `CD`, `DP`, `ES`, `CO`, `LB`, `DS`, and `XS`.
- `Owner` names the bounded context that enforces the rule, not a repository or transport component.
- `Inputs` lists the minimum business information needed to evaluate the rule; it is not a schema.
- `Reason code` is `—` when the source defines an invariant but no canonical failure code.
- Source links target the narrowest authoritative section and deliberately reference, rather than copy, specialized calculations and matrices.

## 3. Status model

| Status | Meaning |
| --- | --- |
| `Confirmed` | Explicitly established by an existing canonical source. |
| `Derived` | Necessary implication of established sources; the inference is explained after its table. |
| `Proposed` | Candidate rule requiring explicit approval before implementation or contract publication. |
| `Deferred` | Recognized rule intentionally outside the current MVP or baseline. |

## 4. Application Process (`AP`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-AP-001 | Submission requires complete application data and consent references | Confirmed | Application Process | Credit Application, customer-data references, Required Consent Set references | Submission is allowed only when all required customer data and consent references are present | Freeze submission inputs and move to `IdentityPending`; otherwise remain in `Draft` | `REQUIRED_CONSENT_MISSING` when consent is incomplete | [Aggregate invariants — CreditApplication](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#creditapplication) |
| BR-AP-002 | One onboarding process per application | Confirmed | Application Process | Application ID, active Onboarding Process | No second active Onboarding Process may exist for the same application | Reuse or reject the duplicate process request | — | [Aggregate invariants — CreditApplication](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#creditapplication) |
| BR-AP-003 | Terminal progression requires an explicit operation | Confirmed | Application Process | Current Application Stage, requested transition | A Terminal State cannot advance through retry or message redelivery | Require an explicit `Reopen` or recovery operation | — | [Application Process vocabulary](UBIQUITOUS_LANGUAGE.md#5-application-process-vocabulary) |
| BR-AP-004 | Applicant cancellation ends at signing | Confirmed | Application Process | Current Application Stage, cancellation authority | Applicant cancellation is permitted only before signature in MVP | Cancel the process before the boundary; require an operations workflow afterward | — | [Hotspots and decisions required](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#14-hotspots-and-decisions-required) |
| BR-AP-005 | Application stores cross-context references, not foreign mutable entities | Confirmed | Application Process | Cross-context identifiers and permitted snapshots | The process may retain traceability references and minimum snapshots but not copies of all mutable entities owned elsewhere | Maintain coarse journey state without taking ownership of another context's state | — | [Aggregate invariants — CreditApplication](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#creditapplication) |
| BR-AP-006 | Only an active unexpired offer may be accepted | Confirmed | Application Process | Offer reference and immutable snapshot, current instant, Terms Hash | Expired, superseded, rejected, or previously accepted offers cannot be accepted | Record one acceptance of the exact immutable terms or reject the command | — | [CreditOffer aggregate root](CREDIT_DECISIONING_DESIGN.md#52-creditoffer-aggregate-root) |

## 5. Customer & Identity (`CI`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-CI-001 | Consent is explicit, purpose-bound evidence | Confirmed | Customer & Identity | Customer, purpose, text/version, timestamp | Consent cannot be inferred from application submission | Create an immutable Consent Record for the defined purpose and version | `REQUIRED_CONSENT_MISSING` when absent | [Customer & Identity vocabulary](UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |
| BR-CI-002 | Consent history is append-only | Confirmed | Customer & Identity | Existing Consent Record, revocation fact | Revocation does not erase prior evidence | Append the revocation fact and preserve history | — | [Customer & Identity vocabulary](UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |
| BR-CI-003 | Identity outcome must be valid for decisioning | Confirmed | Customer & Identity | Verification outcome, verification level, validity instant | A successful outcome is usable only at the required level and before expiry | Publish a provider-neutral verified outcome or stop/request evidence with a safe reason | `IDENTITY_NOT_VERIFIED`; `IDENTITY_EVIDENCE_INSUFFICIENT` | [Hard eligibility rules](PRODUCT_AND_CREDIT_POLICY.md#5-hard-eligibility-rules) |
| BR-CI-004 | Provider evidence remains behind the context boundary | Confirmed | Customer & Identity | Provider result, Identity Evidence Reference | Provider-specific codes and raw identity evidence cannot cross the anti-corruption boundary | Expose provider-neutral outcome, level, safe reason codes, and protected references only | — | [Customer & Identity vocabulary](UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |

## 6. Credit Decisioning (`CD`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-CD-001 | Assessment inputs and policy versions are immutable | Confirmed | Credit Decisioning | Assessment Input Snapshot, Rule Set, formula and policy versions | One assessment evaluates one immutable snapshot under exact immutable versions | Persist a reproducible assessment; material changes create a new assessment version | — | [CreditAssessment invariants](CREDIT_DECISIONING_DESIGN.md#51-creditassessment-aggregate-root) |
| BR-CD-002 | Operational uncertainty is not a credit decline | Confirmed | Credit Decisioning | Prerequisite and provider results | Missing/remediable evidence, retryable provider failure, or inconsistent results cannot produce `Unfavorable` | Return the applicable operational disposition without a Credit Decision | `REQUIRED_CONSENT_MISSING`; `IDENTITY_EVIDENCE_INSUFFICIENT`; `INCOME_EVIDENCE_INSUFFICIENT`; `EXTERNAL_ASSESSMENT_TEMPORARILY_UNAVAILABLE`; `ASSESSMENT_RESULT_INCONSISTENT` | [Input completeness and operational disposition table](CREDIT_DECISION_TABLE.md#4-input-completeness-and-operational-disposition-table) |
| BR-CD-003 | Deterministic rule precedence governs the outcome | Confirmed | Credit Decisioning | Normalized snapshot, DecisionPolicyVersion | Rules execute by defined priority; the highest-priority terminal rule governs while only permitted reasons are exposed | Produce exactly one operational disposition or completed credit outcome according to the decision table | See normative rule row | [Evaluation semantics](CREDIT_DECISION_TABLE.md#3-evaluation-semantics) |
| BR-CD-004 | A favorable decision requires eligible alternatives | Confirmed | Credit Decisioning | Completed guard evaluations, calculated alternatives | All guards must pass and at least one distinct eligible alternative must exist | Record `Favorable` with one to three alternatives; otherwise apply the relevant terminal rule | `ELIGIBLE_AMOUNT_BELOW_PRODUCT_MINIMUM` when none qualifies | [Terminal guard decision table](CREDIT_DECISION_TABLE.md#5-terminal-guard-decision-table) |
| BR-CD-005 | Offer construction obeys all governing limits | Confirmed | Credit Decisioning | Requested amount, affordability result, risk cap, segment cap, product limit, permitted terms | Every alternative must remain within every applicable constraint and installment capacity | Return at most three distinct alternatives under the normative calculation and treatment matrices | See normative guard or limiting-cap reason | [Affordability and offer calculation](CREDIT_DECISION_TABLE.md#8-affordability-and-offer-calculation) |
| BR-CD-006 | One active immutable offer per application | Confirmed | Credit Decisioning | Favorable decision, selected owned alternative, existing active offer | An offer is created only from an alternative owned by a favorable decision; client-supplied replacement terms are prohibited | Create one active offer whose terms equal the selected alternative | — | [CreditOffer invariants](CREDIT_DECISIONING_DESIGN.md#52-creditoffer-aggregate-root) |
| BR-CD-007 | Published decisions are never silently recalculated | Confirmed | Credit Decisioning | Published assessment, changed input or configuration | Any material input or version change requires reevaluation; existing decisions remain immutable | Create a linked assessment and Decision ID under the applicable versions | — | [Governance rules](CREDIT_DECISION_TABLE.md#14-governance-rules) |

## 7. Document Preparation (`DP`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-DP-001 | A package binds one accepted-offer snapshot and template versions | Confirmed | Document Preparation | Accepted Offer snapshot, Document Template versions | Generated artifacts must derive from exactly one accepted-offer snapshot and immutable published templates | Create a versioned Document Package with artifact references and hashes | — | [Aggregate invariants — DocumentPackage and SignatureEnvelope](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#documentpackage-and-signatureenvelope) |
| BR-DP-002 | Approved package contents are immutable | Confirmed | Document Preparation | Approved Package Version, requested content or term change | Package contents cannot change after Document Approval | Reject mutation and require Document Correction/regeneration | — | [Document Preparation vocabulary](UBIQUITOUS_LANGUAGE.md#8-document-preparation-vocabulary) |
| BR-DP-003 | Correction creates a new package version | Confirmed | Document Preparation | Existing Package Version, correction request | A correction cannot overwrite the prior package | Generate a monotonic new Package Version and retain prior audit evidence | — | [Document Preparation vocabulary](UBIQUITOUS_LANGUAGE.md#8-document-preparation-vocabulary) |
| BR-DP-004 | Package bytes remain in protected storage | Confirmed | Document Preparation | Document Artifact bytes, metadata, content hash | File bytes cannot be embedded in public events or ordinary cross-context state | Store bytes in protected object storage and share references plus hashes | — | [Document Preparation vocabulary](UBIQUITOUS_LANGUAGE.md#8-document-preparation-vocabulary) |

## 8. Electronic Signature (`ES`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-ES-001 | An envelope is bound to one package version and signer | Confirmed | Electronic Signature | Package Version, Package Hash, signer, signing purpose | An existing Signature Envelope cannot silently switch package versions, signer, or purpose | Create a new controlled envelope when the binding changes | — | [Electronic Signature vocabulary](UBIQUITOUS_LANGUAGE.md#9-electronic-signature-vocabulary) |
| BR-ES-002 | OTP validation only authorizes signing | Confirmed | Electronic Signature | Valid Signing Authorization, Signature Envelope | OTP validation is not an Electronic Signature and cannot mark documents signed by itself | Permit the signing command for the bound purpose; require separate signature completion evidence | — | [Aggregate invariants — DocumentPackage and SignatureEnvelope](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#documentpackage-and-signatureenvelope) |
| BR-ES-003 | Signature evidence binds the reviewed documents | Confirmed | Electronic Signature | Approved Package Hash, signer, authorization/provider reference, signed instant | Completion is valid only when evidence identifies exactly what was reviewed and signed, by whom, and under which authorization | Record tamper-evident Signature Evidence and a Signed Package | — | [Electronic Signature vocabulary](UBIQUITOUS_LANGUAGE.md#9-electronic-signature-vocabulary) |
| BR-ES-004 | Expired envelopes require a new signing flow | Confirmed | Electronic Signature | Envelope state, expiry instant, current instant | A Signature Envelope cannot be signed after Signature Expiry | End the envelope and require a new controlled signing flow | — | [Electronic Signature vocabulary](UBIQUITOUS_LANGUAGE.md#9-electronic-signature-vocabulary) |

## 9. Communications (`CO`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-CO-001 | OTP challenges are time-bound, attempt-limited, and single-use | Confirmed | Communications | OTP Challenge, submitted value, current instant, attempt count | Validation is allowed only while active, unexpired, unused, and below the configured attempt limit | Authorize the bound purpose once or reject/block validation | — | [Communications and OTP vocabulary](UBIQUITOUS_LANGUAGE.md#10-communications-and-otp-vocabulary) |
| BR-CO-002 | OTP challenges bind signer, purpose, and envelope | Confirmed | Communications | Signer reference, purpose, Signature Envelope ID | A challenge cannot authorize another signer, purpose, or envelope | Return a purpose-specific Signing Authorization only for the matching binding | — | [Aggregate invariants — OtpChallenge](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#otpchallenge) |
| BR-CO-003 | OTP secrets never leave protected representation | Confirmed | Communications | OTP secret, persisted challenge, logs and events | Plaintext OTP values cannot be persisted or emitted; logs cannot contain the secret | Persist a salted hash and expose no secret in logs/events | — | [Aggregate invariants — OtpChallenge](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#otpchallenge) |
| BR-CO-004 | Delivery is not proof of receipt or validation | Confirmed | Communications | Notification Request, delivery attempt/outcome | A request does not guarantee delivery, and delivery does not prove reading or OTP validation | Track delivery independently from the business action it supports | — | [Communications and OTP vocabulary](UBIQUITOUS_LANGUAGE.md#10-communications-and-otp-vocabulary) |
| BR-CO-005 | Reissue makes the prior challenge unusable | Confirmed | Communications | Prior OTP Challenge, reissue eligibility | A new controlled challenge cannot leave the prior challenge usable | Issue a new challenge according to policy and invalidate or retain the prior one as unusable | — | [Communications and OTP vocabulary](UBIQUITOUS_LANGUAGE.md#10-communications-and-otp-vocabulary) |

## 10. Loan Booking (`LB`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-LB-001 | One active loan account per application | Confirmed | Loan Booking | Application ID, existing Loan Accounts | An application can produce at most one Active Loan | Reject or idempotently return the existing active account | — | [Aggregate invariants — LoanAccount and DisbursementOrder](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#loanaccount-and-disbursementorder) |
| BR-LB-002 | Reservation is not activation | Confirmed | Loan Booking | Signed Package, accepted Contractual Terms | A Loan Reservation remains `PendingDisbursement`; it is not an active receivable and cannot accrue interest | Reserve the account without making the obligation effective | — | [Loan Booking vocabulary](UBIQUITOUS_LANGUAGE.md#11-loan-booking-vocabulary) |
| BR-LB-003 | Contractual terms match the accepted offer | Confirmed | Loan Booking | Accepted Offer snapshot, signed package, proposed Contractual Terms | Principal, currency, rate, term, and other contractual terms cannot diverge from the accepted-offer snapshot | Reserve the loan with matching immutable terms or reject booking | — | [Loan Booking vocabulary](UBIQUITOUS_LANGUAGE.md#11-loan-booking-vocabulary) |
| BR-LB-004 | Activation requires matching confirmed funds | Confirmed | Loan Booking | Loan Reservation, Disbursement Confirmation | Activation is allowed only when amount, currency, and reservation reference match the pending loan | Idempotently activate the Loan Account and make its schedule effective | — | [Aggregate invariants — LoanAccount and DisbursementOrder](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#loanaccount-and-disbursementorder) |
| BR-LB-005 | Failed activation after disbursement requires reconciliation | Confirmed | Loan Booking | Confirmed disbursement, activation failure | A confirmed transfer cannot be reinterpreted as a decline or automatically reversed | Enter critical reconciliation and retry activation until an explicit recovery decision | — | [Loan booking decisions](PRODUCT_AND_CREDIT_POLICY.md#14-loan-booking-decisions) |

## 11. Disbursement (`DS`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-DS-001 | One successful transfer per reservation | Confirmed | Disbursement | Loan Reservation, Disbursement Order, prior attempts | A reservation can produce at most one successful Disbursement | Reject or idempotently return the existing successful result | — | [Aggregate invariants — LoanAccount and DisbursementOrder](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#loanaccount-and-disbursementorder) |
| BR-DS-002 | Retries preserve business identity | Confirmed | Disbursement | Disbursement Order, retryable failure, idempotency key | Retried provider attempts cannot create a new business order or use a new idempotency key | Retry the same intended transfer under the stable key | — | [Disbursement vocabulary](UBIQUITOUS_LANGUAGE.md#12-disbursement-vocabulary) |
| BR-DS-003 | Unknown outcomes are reconciled before action | Confirmed | Disbursement | Ambiguous provider response, provider reference, order identifiers | The process cannot retry or declare failure while the submitted transfer outcome is unknown | Reconcile the authoritative provider result first | — | [Disbursement vocabulary](UBIQUITOUS_LANGUAGE.md#12-disbursement-vocabulary) |
| BR-DS-004 | Destination data is tokenized or simulated | Confirmed | Disbursement | Disbursement Destination | Raw bank credentials cannot be stored or propagated | Use a validated token or masked/simulated destination reference | — | [Hotspots and decisions required](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#14-hotspots-and-decisions-required) |
| BR-DS-005 | Terminal failure prevents activation | Confirmed | Disbursement | Terminal Failure, pending Loan Reservation | A non-retryable order cannot activate its reserved loan | Stop activation and trigger explicit reservation recovery; create no Active Loan | — | [Loan booking decisions](PRODUCT_AND_CREDIT_POLICY.md#14-loan-booking-decisions) |

## 12. Cross-cutting (`XS`)

| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BR-XS-001 | Monetary values include amount and currency | Confirmed | Cross-cutting | Monetary value | An amount without an ISO 4217 currency is incomplete | Reject incomplete monetary data or represent it as a Money value | — | [Language rules](UBIQUITOUS_LANGUAGE.md#2-language-rules) |
| BR-XS-002 | Public integration events minimize sensitive data | Confirmed | Cross-cutting | Candidate Integration Event payload | Events may contain references and minimum necessary snapshots, never raw OTPs, document bytes, raw identity evidence, or unnecessary PII | Publish an allowlisted, sanitized payload | — | [Language rules](UBIQUITOUS_LANGUAGE.md#2-language-rules) |
| BR-XS-003 | Producers persist business change and outbox atomically | Confirmed | Cross-cutting | Aggregate change, Integration Event | A public fact cannot be staged independently from its committed business change | Persist the change and Outbox record in one local transaction | — | [Cross-cutting integration vocabulary](UBIQUITOUS_LANGUAGE.md#13-cross-cutting-integration-vocabulary) |
| BR-XS-004 | Consumers handle at-least-once delivery idempotently | Derived | Cross-cutting | Event ID, aggregate version, Inbox record | Duplicate or out-of-order delivery cannot repeat the business effect or regress state | Ignore safely, retain deduplication evidence, and record an operational metric | — | [Failure and compensation paths](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#10-failure-and-compensation-paths) |
| BR-XS-005 | Audit cannot control the synchronous workflow | Confirmed | Cross-cutting | Business/security fact, Audit Trail projection | Audit consumption cannot be a synchronous dependency of a business transition | Append or project the sanitized fact asynchronously | — | [Context relationships](BOUNDED_CONTEXTS_AND_EVENT_STORMING.md#5-context-relationships) |
| BR-XS-006 | Compensation is an explicit business action | Confirmed | Cross-cutting | Completed action, compensating intent | Cross-context failure cannot be handled as a distributed database rollback | Execute an owned, traceable compensation or recovery command | — | [Cross-cutting integration vocabulary](UBIQUITOUS_LANGUAGE.md#13-cross-cutting-integration-vocabulary) |

**Derivation note:** The fourth cross-cutting entry combines the documented duplicate/out-of-order-event recovery with the platform definitions of Inbox and idempotency. Together they require consumers to make repeated delivery equivalent to a single business effect; this is a necessary integration invariant, not a new transport contract.

## 13. Deferred rules

The following areas are acknowledged but intentionally have no active rule IDs in this MVP catalog:

- country-specific regulation and legal disclosures;
- real KYC, AML, sanctions, bureau, and open-finance provider policy;
- analyst override and manual underwriting;
- multiple simultaneous products, revolving credit, guarantors, collateral, and co-borrowers;
- dynamic fees, insurance, tax, collections, delinquency, refinancing, and machine-learning model governance;
- automatic financial reversal after a confirmed transfer.

Their current lifecycle status is `Deferred`. Introducing any of them requires approved policy, ownership, acceptance criteria, and new stable IDs; existing IDs must not be repurposed. See [Decisions deferred beyond MVP](PRODUCT_AND_CREDIT_POLICY.md#16-decisions-deferred-beyond-mvp).

## 14. Governance

1. The owning bounded context approves and enforces each rule; process policies cannot bypass aggregate invariants.
2. IDs remain stable when wording improves and are never reused after retirement.
3. A status change requires evidence in an authoritative source and review by the owning context.
4. Credit Decisioning thresholds, priorities, formulas, matrices, reason-code disclosure, and test scenarios are changed only in their specialized canonical documents and referenced here.
5. Public contracts use these semantics but their versioned schemas belong in the future `loan-platform-contracts` repository.
6. Every implemented rule must be traceable from tests, commands, outcomes, and events without exposing protected evidence.
