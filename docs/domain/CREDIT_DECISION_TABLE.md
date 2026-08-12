# Formal Credit Decision Table

**Product:** Loan Onboarding & Credit Decisioning Platform  
**Policy:** `QuickPersonalLoanPolicy.v1`  
**Status:** Proposed baseline v0.1  
**Date:** 2026-08-06  
**Canonical language:** English  
**Scope:** Fictitious, deterministic demo policy; not production underwriting or country-specific regulatory guidance

## 1. Purpose

This document is the executable semantic baseline for `credit-decisioning-service`. It defines rule precedence, input requirements, credit outcomes, operational dispositions, exposure caps, permitted terms, pricing and reason codes. Each row must be traceable to a parameterized automated test.

The engine evaluates an immutable `CreditAssessmentInputSnapshot` against one immutable `DecisionPolicyVersion`. It returns either:

- an operational disposition when the assessment cannot safely finish; or
- one credit decision: `Favorable` or `Unfavorable`.

An operational failure must never be converted into an unfavorable credit decision.

## 2. Illustrative policy parameters

Amounts are expressed in fictitious monetary units (`MU`) so the portfolio project remains market-neutral.

| Parameter | Value |
| --- | ---: |
| `ProductMinimumAmount` | 100 MU |
| `ProductMaximumAmount` | 2,000 MU |
| `ProductMinimumVerifiedIncome` | 500 MU/month |
| `LowFrictionThreshold` | 1,000 MU |
| `MinimumAgeAtApplication` | 18 years |
| `MaximumAgeAtMaturity` | 70 years |
| `SupportedTerms` | 3, 6, 9, 12 months |
| `OfferValidity` | 24 hours |
| `MoneyRounding` | 2 decimals, midpoint away from zero |
| `RatePrecision` | 6 decimal places |

These values belong to configuration and are repeated here only to make v0.1 testable.

## 3. Evaluation semantics

Rules execute in ascending priority. Within a group, the engine records every applicable non-sensitive reason, but the highest-priority terminal rule determines the outcome.

| Rule effect | Meaning | Continue evaluation? |
| --- | --- | --- |
| `ValidationError` | Input is syntactically invalid; no assessment is created | No |
| `PendingEvidence` | More evidence is needed | No |
| `PendingRetry` | A required provider is temporarily unavailable | No |
| `OperationalException` | Results are conflicting or unsafe to use | No |
| `Unfavorable` | A valid assessment failed a credit-policy guard | No after its priority group |
| `Favorable` | All guards passed and at least one eligible alternative exists | Complete construction |

When multiple unfavorable rules apply in the same priority group:

1. store all internal reason codes;
2. select one `PrimaryReasonCode` using the lowest rule number;
3. expose only safe applicant-facing reasons;
4. never expose fraud thresholds or raw provider data.

## 4. Input completeness and operational disposition table

These rules run before credit-policy evaluation and do not produce a credit decline.

| ID | Priority | Condition | Disposition | Reason code | Retry/evidence action |
| --- | ---: | --- | --- | --- | --- |
| OP-001 | 10 | Required assessment field is missing or malformed | `ValidationError` | `ASSESSMENT_INPUT_INVALID` | Correct request; do not persist a decision |
| OP-002 | 20 | Required consent has not yet been captured | `PendingEvidence` | `REQUIRED_CONSENT_MISSING` | Return to consent collection |
| OP-003 | 30 | Identity evidence is incomplete or below the evidence threshold, without a confirmed mismatch | `PendingEvidence` | `IDENTITY_EVIDENCE_INSUFFICIENT` | Request additional identity evidence |
| OP-004 | 40 | Income cannot be corroborated, without evidence that it is false | `PendingEvidence` | `INCOME_EVIDENCE_INSUFFICIENT` | Request permitted income evidence |
| OP-005 | 50 | Required provider times out or returns a retryable error | `PendingRetry` | `EXTERNAL_ASSESSMENT_TEMPORARILY_UNAVAILABLE` | Retry with backoff and idempotency key |
| OP-006 | 60 | Provider results conflict or cannot be safely interpreted | `OperationalException` | `ASSESSMENT_RESULT_INCONSISTENT` | Route to controlled recovery; no decision |

## 5. Terminal guard decision table

`Any` means the rule does not depend on that input. Every row assumes all higher-priority operational rules have passed.

| ID | Priority | Condition | Credit outcome | Primary reason code | Applicant disclosure |
| --- | ---: | --- | --- | --- | --- |
| G-001 | 100 | Confirmed blocked-party or high-severity fraud signal | `Unfavorable` | `HIGH_RISK_FRAUD_SIGNAL` | Generic safe stop message; do not disclose fraud logic |
| G-002 | 110 | Identity verification is `Failed`, expired, or confirmed data mismatch | `Unfavorable` | `IDENTITY_NOT_VERIFIED` | Identity could not be verified |
| G-003 | 200 | Applicant age is below minimum at application | `Unfavorable` | `AGE_OUTSIDE_PRODUCT_POLICY` | Product eligibility requirements not met |
| G-004 | 210 | Applicant age at expected maturity exceeds maximum | `Unfavorable` | `AGE_AT_MATURITY_OUTSIDE_POLICY` | Selected terms are not eligible |
| G-005 | 220 | Declared residence is outside a supported market | `Unfavorable` | `MARKET_NOT_SUPPORTED` | Product is unavailable in the declared market |
| G-006 | 230 | Requested amount is outside configured product limits | `Unfavorable` | `REQUESTED_AMOUNT_OUTSIDE_PRODUCT_LIMITS` | Requested amount is outside available limits |
| G-007 | 240 | Requested term is not one of 3, 6, 9 or 12 months | `Unfavorable` | `TERM_NOT_SUPPORTED` | Selected term is unavailable |
| G-008 | 250 | Same customer already has an active application | `Unfavorable` | `ACTIVE_APPLICATION_ALREADY_EXISTS` | Continue the existing application |
| G-009 | 260 | Same customer already has an active loan in MVP v1 | `Unfavorable` | `ACTIVE_LOAN_ALREADY_EXISTS` | A new loan is not currently available |
| G-010 | 300 | Verified net monthly income is below 500 MU | `Unfavorable` | `INCOME_BELOW_PRODUCT_MINIMUM` | Income does not meet product requirements |
| G-011 | 310 | Available monthly installment is less than or equal to zero | `Unfavorable` | `INSUFFICIENT_PAYMENT_CAPACITY` | Current obligations leave no available payment capacity |
| G-012 | 400 | Deterministic risk score is below 500 | `Unfavorable` | `RISK_SCORE_BELOW_THRESHOLD` | Application does not meet current credit criteria |
| G-013 | 500 | Calculated maximum eligible amount is below 100 MU for every permitted term | `Unfavorable` | `ELIGIBLE_AMOUNT_BELOW_PRODUCT_MINIMUM` | No sustainable offer is currently available |
| G-014 | 900 | All preceding guards pass and at least one eligible alternative exists | `Favorable` | None | Present eligible alternatives |

Consent and insufficient-but-remediable evidence are deliberately absent from this table because they are operational prerequisites, not measures of creditworthiness.

## 6. Risk treatment matrix

| Risk score | Risk band | PTI limit | Risk exposure cap | Permitted terms | Base monthly effective rate |
| ---: | --- | ---: | ---: | --- | ---: |
| 750–1,000 | `Low` | 35% | 2,000 MU | 3, 6, 9, 12 | 1.70% |
| 600–749 | `Medium` | 30% | 1,400 MU | 3, 6, 9, 12 | 1.90% |
| 500–599 | `High` | 25% | 700 MU | 3, 6 | 2.10% |
| 0–499 | Not eligible | N/A | 0 MU | None | N/A |

Boundary convention: score intervals are inclusive at both displayed ends. A score outside 0–1,000 produces `ASSESSMENT_RESULT_INCONSISTENT`, not a decline.

## 7. Segment treatment matrix

| Segment | Segment exposure cap | Rate adjustment | Additional effect |
| --- | ---: | ---: | --- |
| `New` | 800 MU | +0.00 pp | Amounts above 1,000 MU require additional evidence, but the segment cap already prevents this in v1 |
| `Standard` | 1,400 MU | +0.00 pp | Standard evidence flow |
| `Preferred` | 2,000 MU | -0.20 pp | May use simplified evidence only when identity, fraud and income evidence remain sufficient |

`pp` means percentage points. The rate adjustment cannot reduce the monthly effective rate below 1.50%. Segment never changes the risk band, PTI limit or a terminal guard.

## 8. Affordability and offer calculation

For each term allowed by the risk matrix:

```text
monthlyRate = max(0.015, baseRiskRate + segmentRateAdjustment)

maximumTotalMonthlyDebtService =
    verifiedNetMonthlyIncome × paymentToIncomeLimit

availableMonthlyInstallment = max(
    0,
    maximumTotalMonthlyDebtService - verifiedMonthlyDebtObligations)

affordablePrincipal =
    availableMonthlyInstallment
    × (1 - (1 + monthlyRate)^(-termMonths))
    / monthlyRate

maximumEligibleAmount = roundMoney(min(
    requestedAmount,
    affordablePrincipal,
    riskExposureCap,
    segmentExposureCap,
    productMaximumAmount))
```

An alternative is eligible when:

- `maximumEligibleAmount >= ProductMinimumAmount`;
- its term is permitted by the risk band;
- its calculated installment does not exceed `availableMonthlyInstallment` after monetary rounding; and
- its amount does not exceed any exposure cap.

The service returns at most three distinct alternatives:

1. `RequestedOrMaximumAmount`: greatest eligible amount;
2. `LowestInstallment`: longest eligible term;
3. `LowestTotalCost`: shortest eligible term.

If two objectives produce identical terms, return only one alternative. Selecting an alternative creates the immutable `CreditOffer`; assessment alone does not create or accept an offer.

## 9. Pricing matrix

| Risk band | New | Standard | Preferred |
| --- | ---: | ---: | ---: |
| `Low` | 1.70% monthly effective | 1.70% | 1.50% |
| `Medium` | 1.90% | 1.90% | 1.70% |
| `High` | 2.10% | 2.10% | 1.90% |

Annual effective rate is disclosed but not used as the calculation source:

```text
AnnualEffectiveRate = (1 + MonthlyEffectiveRate)^12 - 1
```

## 10. Canonical scenario table

These scenarios are acceptance examples. Monetary outputs are assertions over the defined formulas; exact installment and total-repayment values should be generated by the shared calculation test fixture.

| Scenario | Key inputs | Expected result | Expected treatment |
| --- | --- | --- | --- |
| S-001 Happy path | Identity valid; no fraud; income 2,000; obligations 200; score 820; `Preferred`; request 1,500 for 12 months | `Favorable` | PTI 35%; rate 1.50%; cap 1,500; terms 3/6/9/12 evaluated |
| S-002 New customer cap | Identity valid; income 3,000; obligations 0; score 800; `New`; request 1,500 | `Favorable` | Maximum 800 MU due to segment cap; rate 1.70% |
| S-003 Medium risk | Identity valid; income 1,500; obligations 150; score 650; `Standard`; request 1,200 | `Favorable` if affordability supports >=100 | PTI 30%; maximum 1,200; rate 1.90%; all terms permitted |
| S-004 High risk | Identity valid; income 1,500; obligations 100; score 550; `Standard`; request 1,000 | `Favorable` if affordability supports >=100 | Cap 700; only 3/6 months; rate 2.10% |
| S-005 Score boundary 500 | All guards pass; score exactly 500 | Evaluate as `High` | Not declined by score rule |
| S-006 Score boundary 499 | All guards pass; score 499 | `Unfavorable` | `RISK_SCORE_BELOW_THRESHOLD` |
| S-007 No capacity | Income 1,000; obligations 350; score 800 | `Unfavorable` | PTI capacity is 350, available installment is 0; `INSUFFICIENT_PAYMENT_CAPACITY` |
| S-008 Reduced offer | All guards pass; calculated affordability below requested amount but >=100 | `Favorable` | Offer is reduced to affordable principal; do not decline |
| S-009 Below minimum offer | All guards pass; maximum eligible amount for every term is 99.99 | `Unfavorable` | `ELIGIBLE_AMOUNT_BELOW_PRODUCT_MINIMUM` |
| S-010 Missing identity evidence | Identity result not final; no confirmed mismatch | `PendingEvidence` | `IDENTITY_EVIDENCE_INSUFFICIENT`; no credit decision |
| S-011 Provider timeout | Risk provider times out | `PendingRetry` | `EXTERNAL_ASSESSMENT_TEMPORARILY_UNAVAILABLE`; no credit decision |
| S-012 Fraud overrides score | Score 900 and affordability passes, but confirmed high-severity fraud | `Unfavorable` | `HIGH_RISK_FRAUD_SIGNAL`; do not expose detail |
| S-013 Duplicate and low score | Active application exists and score 450 | `Unfavorable` | Primary `ACTIVE_APPLICATION_ALREADY_EXISTS`; risk need not be evaluated after terminal eligibility guard |
| S-014 Preferred cannot override capacity | `Preferred`, score 850, but available installment 0 | `Unfavorable` | `INSUFFICIENT_PAYMENT_CAPACITY` |
| S-015 Invalid score | Risk score 1,001 | `OperationalException` | `ASSESSMENT_RESULT_INCONSISTENT`; no decline |

## 11. Reason-code catalog

| Code | Category | Public? | Terminal? |
| --- | --- | --- | --- |
| `ASSESSMENT_INPUT_INVALID` | Validation | Yes | Yes, before assessment |
| `REQUIRED_CONSENT_MISSING` | Prerequisite | Yes | No credit decision |
| `IDENTITY_EVIDENCE_INSUFFICIENT` | Evidence | Yes | No credit decision |
| `INCOME_EVIDENCE_INSUFFICIENT` | Evidence | Yes | No credit decision |
| `EXTERNAL_ASSESSMENT_TEMPORARILY_UNAVAILABLE` | Operational | Generic only | No credit decision |
| `ASSESSMENT_RESULT_INCONSISTENT` | Operational | Generic only | No credit decision |
| `HIGH_RISK_FRAUD_SIGNAL` | Restricted safety | No; replace with generic message | Yes |
| `IDENTITY_NOT_VERIFIED` | Identity | Yes | Yes |
| `AGE_OUTSIDE_PRODUCT_POLICY` | Eligibility | Generic eligibility wording | Yes |
| `AGE_AT_MATURITY_OUTSIDE_POLICY` | Eligibility | Generic eligibility wording | Yes |
| `MARKET_NOT_SUPPORTED` | Eligibility | Yes | Yes |
| `REQUESTED_AMOUNT_OUTSIDE_PRODUCT_LIMITS` | Product | Yes | Yes |
| `TERM_NOT_SUPPORTED` | Product | Yes | Yes |
| `ACTIVE_APPLICATION_ALREADY_EXISTS` | Exposure | Yes | Yes |
| `ACTIVE_LOAN_ALREADY_EXISTS` | Exposure | Yes | Yes |
| `INCOME_BELOW_PRODUCT_MINIMUM` | Eligibility | Yes | Yes |
| `INSUFFICIENT_PAYMENT_CAPACITY` | Affordability | Yes | Yes |
| `RISK_SCORE_BELOW_THRESHOLD` | Risk | Generic credit-criteria wording | Yes |
| `ELIGIBLE_AMOUNT_BELOW_PRODUCT_MINIMUM` | Offer construction | Yes | Yes |

Localized messages are not embedded in the decision engine. Public events carry only permitted codes; restricted details remain in protected assessment evidence.

## 12. Required decision output

Every completed assessment records:

- `DecisionId`, `ApplicationId`, `CustomerId` and correlation identifiers;
- `CreditOutcome` and permitted `ReasonCodes`;
- `RiskScore`, `RiskBand` and `CustomerSegment`;
- verified income, obligations, PTI limit and available installment;
- evaluated alternatives, including rejected-term calculation reasons;
- selected caps and which cap constrained each alternative;
- rate, term, installment, total repayment and rounding results;
- hashes/references for immutable input evidence;
- every policy and formula version used;
- evaluation timestamp and offer-expiry basis.

## 13. Test traceability

Minimum automated test suites:

| Suite | Required coverage |
| --- | --- |
| Guard tests | One test per `G-*` row plus precedence combinations |
| Operational tests | One test per `OP-*` row proving no unfavorable decision is emitted |
| Boundary tests | Scores 499/500/599/600/749/750/1,000; all amount, income, age and term boundaries |
| Matrix tests | Every risk-band × segment combination |
| Formula tests | Zero capacity, rounding boundaries, rate conversion and present-value calculations |
| Property tests | Offered amount never exceeds request, affordability or any cap; installment never exceeds capacity |
| Determinism tests | Same snapshot + policy versions always produces byte-equivalent business result |
| Contract tests | Public response/events exclude restricted fraud details and raw provider data |

Each test should reference the decision-table rule ID in its display name or trait, for example `G_012_ScoreBelow500_ReturnsUnfavorableDecision`.

## 14. Governance rules

- A published decision table is immutable.
- Any threshold, cap, formula, priority or reason-code change creates a new policy version.
- Existing decisions are never recalculated silently.
- Reevaluation creates a new `DecisionId` linked to the previous assessment.
- Code paths not represented by a rule ID are not permitted to create a credit outcome.
- Examples in this document are architectural fixtures, not lending recommendations.

