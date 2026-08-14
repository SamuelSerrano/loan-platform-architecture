# Data Ownership

## 1. Purpose and authority

This document is the canonical transversal catalog for authoritative data ownership, permitted access, copies, sensitivity, retention decisions, and audit expectations across the Loan Onboarding Platform. Each business datum has exactly one authoritative owner. The owner controls its system of record, valid writers, invariants, and authoritative query surface.

Ownership does not transfer when another context holds an identifier, immutable provenance snapshot, cache, replica, or projection. Provider-native data and transport state may be authoritative within an external or infrastructure boundary, but the relevant bounded context owns the provider-neutral business interpretation used by the platform.

This catalog does not define database schemas, final retention periods, event payload schemas, or implementation repositories. Detailed domain semantics remain in the [Ubiquitous Language](../domain/UBIQUITOUS_LANGUAGE.md), [Business Rules](../domain/BUSINESS_RULES.md), and [event catalog](../domain/DOMAIN_EVENTS.md).

## 2. Ownership and access rules

- **Authoritative owner:** the bounded context or external/platform authority that defines meaning and invariants.
- **System of record:** the owner-controlled logical store or external authority from which the datum is recovered.
- **Writer:** only the owner through its application/domain behavior; infrastructure adapters act on the owner's behalf.
- **Consumer access:** owner-controlled command/query interface, minimum versioned integration event, or an explicitly eventual projection.
- **Reference:** opaque identifier that does not copy the owner's business state.
- **Immutable snapshot:** consumer-owned evidence of exactly what was used for a local decision. It has independent provenance but does not replace the source.
- **Projection/replica/cache:** consumer-owned copy that may be stale and is non-authoritative unless this catalog explicitly states otherwise.

All design statuses use `Confirmed`, `Derived`, `Proposed`, or `Deferred`; all delivery statuses are `Planned`. `Retention status: Open (Q-003)` means classification and lifecycle controls are required, but no numeric period is approved.

## 3. Authoritative data ownership matrix

### 3.1 Application Process

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Application ID and Application Number | Application Process | AP-owned store | AP command handling | Authoritative AP query; minimum events | Other contexts may retain identifiers as correlation references | Safe identifiers; application number remains non-sensitive but logs are controlled | Open (`Q-003`) | Creation and correlation trace | Confirmed / Planned | [AP vocabulary](../domain/UBIQUITOUS_LANGUAGE.md#5-application-process-vocabulary) |
| Selected Product reference | Application Process | Credit Application | AP | AP query/events | Product ID/reference only; product definition remains separately governed | Commercial metadata | Open (`Q-003`) | Selection/submission changes | Confirmed / Planned | [AP event](../domain/DOMAIN_EVENTS.md#41-application-process-ap) |
| Credit Application state | Application Process | AP-owned store | AP aggregate | Authoritative AP command/query; AP events | Coarse replicas only | May reveal applicant journey | Open (`Q-003`) | Every valid transition and actor/cause | Confirmed / Planned | [Business Rules](../domain/BUSINESS_RULES.md#4-application-process-ap) |
| Application Stage and Stage History | Application Process | Credit Application plus append-only history | AP aggregate/policies | Authoritative AP query; tracker may be eventual | Rebuildable tracker projection | Journey metadata | Open (`Q-003`) | Append-only stage transition evidence | Confirmed / Planned | [Ubiquitous Language](../domain/UBIQUITOUS_LANGUAGE.md#5-application-process-vocabulary) |
| Onboarding Process/saga state | Application Process | AP-owned process store | AP process manager | Owner operations query only | Operations projection may copy safe state | Internal operational metadata | Open (`Q-003`) | Step, cause, retry, recovery action | Confirmed responsibility; persistence detail Proposed / Planned | [A-003](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-awaiting-adrs) |
| Customer, decision, offer, package, envelope, loan, and disbursement references | Application Process | Credit Application/process state | AP on validated owner fact | AP query/events | Opaque IDs and minimum status cache; source context remains authoritative | Correlation may be sensitive in aggregate | Open (`Q-003`) | Source event ID/version and update time | Confirmed / Planned | [Container Architecture](CONTAINER_DIAGRAM.md#2-architectural-rules) |
| Applicant offer acceptance record | Application Process | Credit Application | AP handling `AcceptCreditOffer` | AP authoritative query/event | DP may retain package-bound snapshot; CD may project awareness | Journey/legal evidence; no mutable terms | Open (`Q-003`) | Applicant/action identity, `offerId`, exact `termsHash`, timestamp, idempotency | Confirmed (`HS-001`) / Planned | [HS-001](../domain/EVENT_STORMING.md#11-hotspots) |
| Applicant offer rejection record | Application Process | Credit Application | AP handling `RejectCreditOffer` | AP authoritative query/event | CD/audit may project awareness | Journey decision metadata | Open (`Q-003`) | Applicant/action identity, `offerId`, timestamp, idempotency | Confirmed (`HS-001`) / Planned | [HS-001](../domain/EVENT_STORMING.md#11-hotspots) |
| Application Inbox, Outbox, and idempotency records | Application Process | AP-owned operational store | AP infrastructure acting for AP | Operations query only | Metrics/checkpoints; DLQ may copy event payload | Operational metadata and potentially sensitive minimized payload | Open (`Q-003`) | Event ID, handler/publish outcome, retry/redrive actor | Confirmed / Planned | [Cross-cutting rules](../domain/BUSINESS_RULES.md#12-cross-cutting-xs) |

Application Process does not own customer PII, identity evidence, credit calculations or original Offer Terms, document bytes, signature evidence, loan balance/schedule, or the disbursement-provider outcome.

### 3.2 Customer & Identity

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Customer Profile | Customer & Identity | CI-owned store | CI profile handling | Owner-controlled authoritative query; minimized events | Other contexts retain `customerId` and approved snapshot fields only | PII; strict minimization and access control | Open (`Q-003`, `Q-004`) | Create/change actor, purpose, version | Confirmed / Planned | [CI vocabulary](../domain/UBIQUITOUS_LANGUAGE.md#6-customer--identity-vocabulary) |
| Purpose-bound Consent Records | Customer & Identity | CI-owned store | CI consent handling | Owner query; consent references in events | Reference and minimum purpose/version evidence | PII/legal evidence | Open (`Q-003`) | Text version, purpose, timestamp, actor | Confirmed / Planned | [BR-CI-001](../domain/BUSINESS_RULES.md#5-customer--identity-ci) |
| Required Consent Set evidence | Customer & Identity | CI-owned store | CI | Owner validation/query; reference to AP | Required-set result/reference | Legal and customer metadata | Open (`Q-003`) | Requirement version and satisfaction evidence | Confirmed / Planned | [Submission event](../domain/DOMAIN_EVENTS.md#41-application-process-ap) |
| Identity Verification cases and normalized result/validity | Customer & Identity | CI-owned store | CI workflow/ACL | Owner query; minimized verified/rejected event | Safe result, verification ID, level, validity | Identity and restricted fraud data | Open (`Q-003`, `Q-004`) | Provider-neutral outcome, evidence refs, timestamps, reason classification | Confirmed / Planned | [CI events](../domain/DOMAIN_EVENTS.md#42-customer--identity-ci) |
| Protected identity-evidence references | Customer & Identity | CI store plus owner-scoped protected objects | CI evidence adapter | Owner-authorized query only | Opaque references; no raw evidence in events | Highly sensitive identity evidence | Open (`Q-003`) | Access, creation, replacement, deletion/lifecycle action | Confirmed / Planned | [System Context](SYSTEM_CONTEXT.md#7-trust-boundary-rules) |

### 3.3 Credit Decisioning

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Credit Assessment | Credit Decisioning | CD-owned store | CD assessment aggregate | Sanitized authoritative query; minimized events | IDs/status projection | Financial and internal decision metadata | Open (`Q-003`, `Q-004`) | Input/version identity, disposition/decision, cause | Confirmed / Planned | [CD aggregate](../domain/CREDIT_DECISIONING_DESIGN.md#51-creditassessment-aggregate-root) |
| Immutable Assessment Input Snapshot | Credit Decisioning | CD-owned store | CD orchestration at assessment creation | Restricted owner query | Hash and minimum evidence references only | Financial evidence and PII-derived facts | Open (`Q-003`, `Q-004`) | Snapshot hash, source refs, captured time, policy versions | Confirmed / Planned | [CD design](../domain/CREDIT_DECISIONING_DESIGN.md#51-creditassessment-aggregate-root) |
| Affordability and risk results | Credit Decisioning | Credit Assessment | Deterministic CD engine | Sanitized query/event | Safe bands/codes; never unrestricted trace | Financial evidence/internal risk detail | Open (`Q-003`, `Q-004`) | Formula/rule versions and result trace | Confirmed / Planned | [CD services](../domain/CREDIT_DECISIONING_DESIGN.md#8-domain-services) |
| Policy and Rule Set versions | Credit Decisioning | Governed CD configuration source | Authorized policy governance | Read by CD engine; version references in events | Immutable versioned artifact/cache | Internal decision policy | Open (`Q-003`) | Publication, checksum, effective version | Confirmed responsibility; storage Proposed / Planned | [CD design](../domain/CREDIT_DECISIONING_DESIGN.md#2-context-responsibility) |
| Credit Decision and reason codes | Credit Decisioning | Credit Assessment | CD aggregate once | Sanitized owner query/event | Applicant-safe codes and outcome snapshot | Internal decision trace; disclosure under `Q-008` | Open (`Q-003`, `Q-004`) | Immutable outcome, rule/policy versions, reason codes | Confirmed / Planned | [CD events](../domain/DOMAIN_EVENTS.md#43-credit-decisioning-cd) |
| Offer Alternatives | Credit Decisioning | Credit Assessment | CD engine | Owner query; sanitized favorable event | AP may present immutable alternative snapshot | Financial terms | Open (`Q-003`) | Calculation inputs/version and term hashes | Confirmed / Planned | [BR-CD-004](../domain/BUSINESS_RULES.md#6-credit-decisioning-cd) |
| Immutable Credit Offer and canonical Offer Terms/hash | Credit Decisioning | CD-owned offer store | CD offer lifecycle only | Authoritative offer query; `CreditOfferCreated.v1` candidate/contract | AP references offer and caches canonical snapshot; DP provenance snapshot | Financial/contractual terms | Open (`Q-003`) | Creation, selection source, canonical hash, expiry, supersession | Confirmed / Planned | [CreditOffer](../domain/CREDIT_DECISIONING_DESIGN.md#52-creditoffer-aggregate-root) |
| Offer creation, expiry, and supersession state | Credit Decisioning | Credit Offer | CD lifecycle commands | Owner query/events | AP/CD consumer projections may copy status | Commercial journey metadata | Open (`Q-003`) | Every terminal transition and replacement link | Confirmed / Planned | [CD events](../domain/DOMAIN_EVENTS.md#43-credit-decisioning-cd) |

Acceptance or rejection never mutates the immutable `CreditOffer`; awareness in Credit Decisioning is a non-authoritative projection of Application Process facts.

### 3.4 Document Preparation and Electronic Signature

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Document Template versions | Document Preparation | DP-owned store/artifact source | Authorized DP template management | DP generation query only | Version/checksum reference | Legal/commercial content | Open (`Q-003`) | Version publication and use | Derived / Planned | [Document rules](../domain/BUSINESS_RULES.md#7-document-preparation-dp) |
| Document Package metadata, versions, approval/correction state | Document Preparation | DP-owned store | DP aggregate | Owner query; minimized events | Checklist/status projections and references | PII-linked legal metadata | Open (`Q-003`) | Generation, approval, correction, invalidation with hashes | Confirmed / Planned | [DP events](../domain/DOMAIN_EVENTS.md#44-document-preparation-dp) |
| Document Artifact references, hashes, and protected bytes | Document Preparation | DP store plus owner-scoped protected object store | DP artifact adapter | Authorized owner-controlled download/query | References/hashes only; no bytes in events | Highly sensitive document artifacts | Open (`Q-003`) | Access, generation, hash verification, lifecycle action | Confirmed / Planned | [BR-DP-004](../domain/BUSINESS_RULES.md#7-document-preparation-dp) |
| Package-bound accepted-offer snapshot | Document Preparation | Document Package provenance | DP at package generation from AP event/owner access | DP package query; audit reference | Immutable exact `offerId`, canonical `termsHash`, minimum accepted terms/action snapshot | Contractual/financial provenance | Open (`Q-003`) | Source event/version, source owners, hashes, generation link | Confirmed / Planned | [HS-001 resolution](../domain/EVENT_STORMING.md#11-hotspots) |
| Signature Envelope, signer references, and signing state | Electronic Signature | ES-owned store | ES aggregate | Owner query; minimized events | Status projection and package/envelope references | Identity/legal metadata | Open (`Q-003`, `Q-004`) | Creation, authorization, state transitions, provider refs | Confirmed / Planned | [ES events](../domain/DOMAIN_EVENTS.md#45-electronic-signature-es) |
| Signing Authorization consumption | Electronic Signature | Signature Envelope/ES operational store | ES signing coordination | Owner operations query | Authorization reference only; OTP remains CO-owned | Security authorization evidence | Open (`Q-003`) | Challenge/authorization reference, purpose, use timestamp | Derived / Planned | [Event Storming](../domain/EVENT_STORMING.md#7-phase-d--otp-and-signature) |
| Signature Evidence and provider-neutral signature references | Electronic Signature | ES store plus protected objects | ES evidence handling/ACL | Strict owner-controlled query; Signed Package event carries reference | Opaque evidence reference only | Highly sensitive legal/signature evidence | Open (`Q-003`) | Signer, package hash/version, evidence hash/ref, provider result | Confirmed / Planned | [BR-ES-002](../domain/BUSINESS_RULES.md#8-electronic-signature-es) |
| Signed Package fact | Electronic Signature | Signature Envelope | ES aggregate | Versioned integration event; owner query | Consumers retain package/envelope/evidence references | Legal journey metadata | Open (`Q-003`) | Exact package version/hash, signer refs, signed instant | Confirmed / Planned | [IE-ES-001](../domain/DOMAIN_EVENTS.md#55-electronic-signature-es) |

### 3.5 Communications, Loan Booking, and Disbursement

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| OTP Challenge and protected OTP representation | Communications | CO-owned store | CO aggregate | CO commands/query only; authorization reference to ES | No plaintext copy; masked/opaque references only | Secret; plaintext must never be persisted, logged, or emitted | Open (`Q-003`) | Issue, expiry, attempts, block, validation; never secret value | Confirmed / Planned | [BR-CO-001](../domain/BUSINESS_RULES.md#9-communications-co) |
| Notification Request, Delivery, retry history, masked destination, and delivery refs | Communications | CO-owned store | CO delivery lifecycle/adapters | Owner operations query; no baseline public event | Masked destination/safe delivery ref | Contact PII and provider metadata | Open (`Q-003`, `Q-004`) | Request, attempt, normalized result, retry | Confirmed / Planned | [CO events](../domain/DOMAIN_EVENTS.md#46-communications-co) |
| Loan Reservation and activation state | Loan Booking | LB-owned store | LB aggregate | Owner query/events | AP references loan/reservation and projects status | Financial/contractual metadata | Open (`Q-003`) | Reservation source, state transition, disbursement match | Confirmed / Planned | [BR-LB](../domain/BUSINESS_RULES.md#10-loan-booking-lb) |
| Loan Account, contractual terms snapshot, initial schedule, account number/reference | Loan Booking | LB-owned store | LB aggregate/schedule component | Authoritative LB query; minimized events | Consumer summary projection; protected account reference | Financial account data | Open (`Q-003`, `Q-004`) | Terms/package provenance, schedule version, activation | Confirmed / Planned | [LB events](../domain/DOMAIN_EVENTS.md#47-loan-booking-lb) |
| Disbursement Order and stable idempotency identity | Disbursement | DS-owned store | DS aggregate | Owner operations query/events | Other contexts retain order ID/status projection | Financial operation metadata | Open (`Q-003`) | Creation source, stable key reference, state transitions | Confirmed / Planned | [BR-DS](../domain/BUSINESS_RULES.md#11-disbursement-ds) |
| Destination token/reference snapshot | Disbursement | Disbursement Order | DS at validated order creation | DS/provider adapter only | Tokenized reference; LB may carry upstream reference | Highly sensitive financial destination; never raw credentials | Open (`Q-003`, `Q-004`) | Source, validation, access/use | Confirmed / Planned | [DE-DS-001](../domain/DOMAIN_EVENTS.md#48-disbursement-ds) |
| Disbursement attempts, provider-neutral result, safe provider ref, terminal failure/reconciliation state | Disbursement | DS-owned store | DS execution/reconciliation | Owner query; minimized events/receipt projection | Safe result/reference only | Financial outcome and provider metadata | Open (`Q-003`) | Every attempt, ambiguous outcome, reconciliation query/result, terminal state | Confirmed / Planned | [DS events](../domain/DOMAIN_EVENTS.md#48-disbursement-ds) |

### 3.6 Platform, audit, and external authorities

| Business datum | Authoritative owner | System of record | Permitted writers | Consumer access | Allowed copy/reference | Sensitive-data consideration | Retention status | Audit requirement | Status | Sources |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Authentication identities and credentials for demo | Amazon Cognito | Cognito user pool | Cognito/admin flows | Claims through authentication integration | Subject ID reference; no Customer Profile inference | Credentials/authentication PII | Open (`Q-003`) | Authentication/security events sanitized | Confirmed direction / Planned | [System Context](SYSTEM_CONTEXT.md#5-external-systems) |
| Provider-native payloads/references | Respective provider | Provider system | Provider | Through owner ACL only | Owner stores minimum safe reference/normalized snapshot | Often highly sensitive; credentials never enter domain/events | Provider policy plus platform decision open (`Q-003`) | Access and normalized interpretation | Confirmed principle / Planned | [Trust rules](SYSTEM_CONTEXT.md#7-trust-boundary-rules) |
| Transport delivery state | EventBridge/SQS/DLQ or local equivalent | Transport service | Transport/adapters | Operations and consuming Inbox | Event payload replica in queues/DLQ | Payload remains sensitive; DLQ is a sensitive replica | Open (`Q-003`) | Publish/delivery/retry/redrive metadata | Confirmed direction / Planned | [Container Architecture](CONTAINER_DIAGRAM.md#2-architectural-rules) |
| Audit projection records and ingestion checkpoints | Audit & Compliance Projection | Audit-owned projection store | Audit consumer only | Eventually consistent authorized audit query | Sanitized copied facts with source IDs | Audit/operational metadata; never unrestricted payload dump | Open (`Q-003`, `Q-004`) | Append/ingestion/checkpoint/export/access | Confirmed / Planned | [A-008](../discovery/ASSUMPTIONS.md#confirmed-architecture-decisions-awaiting-adrs) |
| Public API transport state | Public API | Transient runtime only | Edge adapter | Operational telemetry | Redacted request metadata only | Avoid payload/credential logging | Open (`Q-003`) | Authentication/authorization/transport result with safe IDs | Confirmed principle / Planned | [System Context](SYSTEM_CONTEXT.md#7-trust-boundary-rules) |
| Client/UI presentation state | Client/UI | Browser/client memory | Client | User session only | Ephemeral, minimized cache | User-controlled and potentially PII-bearing | Session-only intent; final policy open | Avoid sensitive telemetry; user actions audited by owner command | Proposed / Planned | [Component Catalog](COMPONENT_CATALOG.md#3-platform-edge-and-shared-infrastructure-components) |

The Public API owns no business data. Cognito is not authoritative for Customer Profile or Identity Verification. EventBridge, SQS, and DLQs own delivery mechanics, not the business facts carried. Audit owns only its projection and cannot be a synchronous dependency or decision source.

## 4. Database and model isolation

1. Every business service owns its persistence. DynamoDB is the default AWS operational store, not a shared platform database.
2. Every deployed service uses a separate logical table/store boundary and owner-scoped credentials. A local adapter preserves the same ownership boundary.
3. No service reads or writes another service's table, file store, or schema.
4. Cross-context information moves through commands handled by the owner, owner-controlled queries, or versioned integration events.
5. Transactions never span service-owned persistence boundaries. Cross-context consistency is explicit and eventual.
6. Outbox records commit atomically with the owning aggregate. Inbox records belong to the consuming service.
7. Caches and projections are disposable and non-authoritative unless explicitly identified otherwise.
8. Shared C# domain entities and persistence models are prohibited. Future `loan-platform-contracts` may share public schemas and generated transport types, never service domain models.

## 5. Replica, cache, snapshot, and projection registry

| Replica/read model | Owning consumer | Authoritative source | Update mechanism | Data subset and purpose | Consistency/freshness | Rebuildable | Authoritative? | Retention status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Applicant Application Tracker | Application Process | AP Application/Stage History plus consumed owner facts | Local domain events and versioned integration events | Coarse journey, safe statuses, correlation refs for applicant visibility | Eventually consistent for external facts; may be stale | Yes, from retained source facts/snapshots | No; AP state query is authoritative for coarse stage | Open (`Q-003`) |
| Operations Case View | Operations/Audit projection | Each owning bounded context | Sanitized asynchronous integration events | Failures, retries, DLQ/recovery refs; supports diagnosis, not direct mutation | Eventually consistent; may omit unpublished/private facts | Yes where source events retained | No; cannot decide outcomes | Open (`Q-003`, `Q-004`) |
| Decision Explanation | Credit Decisioning | Credit Assessment, Decision, rules/policy versions | Owner-local projection or owner-controlled query | Sanitized outcome, safe codes, versions; excludes unrestricted trace | Owner query may be current; projection may lag | Yes from CD state | Only direct CD query is authoritative; replica is not | Open (`Q-003`, `Q-004`) |
| Document Checklist | Document Preparation | Document Package and Artifact metadata | DP domain events | Package/artifact readiness and hashes; no bytes | Eventually consistent | Yes | No | Open (`Q-003`) |
| Signature Status | Electronic Signature | Signature Envelope | ES domain events/provider reconciliation | Envelope state, expiry, safe provider status | Eventually consistent and may lag provider reconciliation | Yes from envelope | No | Open (`Q-003`) |
| Disbursement Receipt | Disbursement | Confirmed Disbursement Order result | DS domain/integration event | Amount/currency, confirmed instant, safe provider reference | Eventually consistent after confirmation | Yes | No; DS order query is authoritative | Open (`Q-003`) |
| Loan Summary and Schedule | Loan Booking | Loan Account and initial schedule | LB local projection/events | Protected account ref, activation, schedule version/entries for display | Eventually consistent when replicated; owner query can be current | Yes from LB state | No when replicated | Open (`Q-003`, `Q-004`) |
| Audit & Compliance Projection | Audit & Compliance Projection | Published sanitized facts from all owners | Per-consumer queue and idempotent ingestion | Immutable audit-oriented copies and source event IDs | Eventually consistent; never blocks business flow | Conditionally, from retained events/source exports | No | Open (`Q-003`, `Q-004`) |
| Application offer/customer reference cache | Application Process | Credit Decisioning offer; Customer & Identity profile/result | Owner query or versioned events | `offerId`, exact canonical terms/hash/expiry; `customerId` and minimum safe status required for coordination | May be stale; expiry/version checked before action | Yes | No; cannot redefine offer/customer facts | Open (`Q-003`) |
| Package-bound accepted-offer snapshot | Document Preparation | AP acceptance record plus CD canonical offer terms | `CreditOfferAccepted.v1` minimum snapshot and/or owner-controlled query during generation | Exact package input/provenance: offer/action refs, terms/hash required to render package | Immutable once package generated; intentionally point-in-time | No; it is preserved provenance, though a package may be regenerated as a new version | Authoritative only for what DP used in that package; not for original offer or acceptance | Open (`Q-003`) |
| Inbox records | Each consuming service | Delivered integration event plus handler result | Atomic consumer transaction | Event ID, consumer, outcome, expiry metadata for deduplication | Current for that consumer; eventual relative to transport | Usually disposable after approved window | No business authority | Open (`Q-003`) |
| Outbox records | Each producing service | Owner aggregate transaction | Atomic producer transaction, asynchronous publication update | Public event payload, publish state, attempts | May await publication; business state can precede delivery | Reconstructability depends on owner policy; do not assume | No; aggregate remains business authority | Open (`Q-003`) |
| DLQ records | Transport/consumer operations | Failed delivered integration event | Transport after retry exhaustion; controlled redrive | Full minimized event payload plus delivery metadata | Stale by definition; quarantined | Not a business projection | No; never make decisions from DLQ content alone | Open (`Q-003`); treat as sensitive |
| Policy/rule cache | Credit Decisioning | Governed immutable policy/rule-set source | Version-addressed load | Immutable policy version used for deterministic evaluation | Must match requested version; stale latest pointer cannot replace explicit version | Yes | No; source artifact/version is authoritative | Open (`Q-003`) |

Every listed copy exists for read performance, coordination, provenance, delivery safety, or audit—not to bypass an owner. A stale copy cannot authorize a financial or legal action unless the owning aggregate validates the referenced version/hash. Deletion and retention follow either the source's approved policy or a separately approved consumer policy; neither exists numerically while `Q-003` remains open.

## 6. Sensitive data, retention, and audit

### 6.1 Provisional categories and controls

| Category | Examples | Minimum control |
| --- | --- | --- |
| PII | Customer Profile, contact destination, signer linkage | Least privilege, encryption, minimized copies, redacted logs, safe IDs. |
| Identity evidence | Provider evidence and protected references | Owner-scoped protected objects; references only across contexts. |
| Financial evidence | Income/obligation inputs, affordability, account/destination refs | Strict access, tokenization/reference, no raw event payload. |
| Internal decision trace | Rule evaluations, restricted fraud details | CD-only protected access; sanitized explanations/events. |
| Document artifacts | Generated package bytes and versions | Protected object access, hashes, explicit lifecycle. |
| Signature evidence | Tamper-evident provider-neutral evidence | Protected access, immutable provenance, audited reads. |
| OTP secrets | Submitted OTP and protected representation | Never plaintext at rest, in logs, events, queues, or audit. |
| Disbursement destination credentials/tokens | Destination secret/token and provider reference | Tokenize, restrict adapter access, emit only safe references. |
| Provider credentials and raw payloads | API secrets, vendor requests/responses | Secret store and ACL boundary; never domain/events/logs. |
| Audit and operational metadata | Event IDs, retries, DLQs, correlation timeline | Sanitize, authorize, redact, and apply explicit lifecycle. |

Integration events use references and the minimum approved immutable snapshot. They never contain OTP plaintext, raw identity evidence, unrestricted PII, document bytes, raw destination credentials, provider credentials, or unrestricted internal rule traces. Logs use redaction and safe identifiers. DLQs inherit the sensitivity of event payloads and require restricted access, audit, and lifecycle controls. Protected objects require explicit access control and lifecycle configuration. Audit consumes sanitized events asynchronously.

### 6.2 Retention decision

Numeric retention periods remain unresolved under `Q-003`; field-level PII/tokenization rules remain unresolved under `Q-004`. Neither question is resolved by this document. The Local Zero AWS Cost profile uses only fictitious data but preserves ownership, minimization, redaction, expiry, and deletion semantics. The eventual AWS Demo profile must choose short explicit retention values, enforce teardown, and verify deletion without claiming that demo defaults are production policy.

## 7. Execution profiles

### Local — Zero AWS Cost

- No AWS account or credentials are required.
- Local stores, object adapters, queues, and provider fakes remain owner-scoped and independently addressable.
- A convenience runtime may co-locate processes physically, but it cannot create a shared schema or shared business model.
- Event semantics, Inbox/Outbox identity, recovery, minimization, audit, and retention decisions remain equivalent to hosted profiles.

### AWS Demo — Ephemeral and Free-Tier-Aware

- Only the walking-skeleton services are initially deployable.
- Each deployed service has a distinct logical DynamoDB boundary and credentials.
- EventBridge/SQS/DLQ and optional S3 are transport/object mechanisms, never shared business authority.
- No persistent-cost infrastructure is introduced; short retention and teardown constraints remain visible.
- A planned component or documented datum is not represented as implemented or deployed.

See [Container Architecture cost governance](CONTAINER_DIAGRAM.md#7-cost-model-and-governance) for the complete cost model.

## 8. Governance checks

1. A new datum must name one authoritative owner and system of record before entering a contract or service specification.
2. A new consumer must document why a reference, query, snapshot, or projection is needed and whether it may be stale.
3. A consumer cannot write copied owner data or use a projection to bypass owner invariants.
4. Event payload expansion requires minimization and `Q-004` review; a new consumer does not automatically justify more PII.
5. Retention and deletion changes must update `Q-003` and all affected source/replica policies.
6. Auditing records the actor/cause, source fact/version, correlation, timestamps, and result without copying prohibited secrets.
7. Direct database integration, shared domain entities, and distributed transactions are architecture violations.

## 9. Navigation

- [Component Catalog](COMPONENT_CATALOG.md)
- [System Context](SYSTEM_CONTEXT.md)
- [Container Architecture](CONTAINER_DIAGRAM.md)
- [Assumptions and Open Questions](../discovery/ASSUMPTIONS.md)
- [Domain and Integration Events](../domain/DOMAIN_EVENTS.md)
- [Event Storming](../domain/EVENT_STORMING.md)
