# Product and Credit Policy

**Product:** Loan Onboarding & Credit Decisioning Platform  
**Status:** Proposed baseline v0.1  
**Date:** 2026-08-06  
**Canonical language:** English  
**Scope:** Fictitious low-value, unsecured consumer-loan demo; not a production lending policy

## 1. Policy intent

The product offers a low-friction entry experience without removing the controls required before a credit offer is created. The applicant provides only the information needed to start; identity, consent, fraud, affordability and risk controls are applied progressively before decisioning.

The policy follows five principles:

1. **Progressive onboarding:** do not request documents or data before they are needed.
2. **No blind approval:** a short form is not a reduced decision standard.
3. **Explainability:** every decision is reproducible from an immutable input snapshot and a versioned rule set.
4. **Proportionality:** friction, evidence and limits increase with exposure and uncertainty.
5. **Responsible offer construction:** the offered amount is the minimum of what the customer requests, can afford and the product permits.

## 2. Demonstration product

| Parameter | Baseline decision |
| --- | --- |
| Product | `Quick Personal Loan` |
| Borrower | One natural person |
| Purpose | General-purpose consumer credit |
| Security | Unsecured; no guarantor or collateral |
| Currency | Configurable ISO 4217 currency; demo scenarios use fictitious amounts |
| Product limits | Configured by `ProductPolicy.v1`, not hard-coded in domain entities |
| Term | 3, 6, 9 or 12 monthly installments |
| Repayment | Fixed monthly installment |
| Interest | Fixed for the contractual term |
| Fees | Zero in MVP v1 to keep calculations transparent |
| Offer validity | 24 hours |
| Early repayment | Allowed without a simulated penalty |
| Concurrent exposure | One active application and one active loan per customer in MVP |

The MVP has one versioned product definition inside Credit Decisioning. A separate Product Catalog bounded context is deferred until multiple products or independent product administration justify it.

## 3. Progressive information collection

### Stage 1 — Start the application

Only the following data is required to create a `Draft`:

- document type and document number;
- mobile number;
- email address;
- requested amount;
- acceptance of privacy/data-processing notice.

No credit decision occurs at this stage.

### Stage 2 — Submit for verification

Before submission, collect:

- full legal name and date of birth;
- supported residence/market declaration;
- income source and declared net monthly income;
- declared monthly debt obligations;
- bank-account destination token or masked reference;
- required identity, credit-assessment and electronic-communications consents.

### Stage 3 — Evidence on demand

Additional evidence is requested only when a rule requires it, for example:

- identity evidence is insufficient;
- declared income cannot be corroborated by the simulated provider;
- fraud signals conflict with the applicant profile;
- requested exposure exceeds the low-friction threshold.

For MVP v1, an unresolved evidence request does not create a third credit outcome. The application remains pending or is closed with a safe reason code. Future versions may add a formal `Referred` decision and analyst workflow.

## 4. Control layers

| Layer | Purpose | Typical result |
| --- | --- | --- |
| Entry validation | Ensure the request is syntactically usable | Remain in `Draft` and show validation errors |
| Product eligibility | Check non-negotiable product boundaries | Decline with stable reason code |
| Identity and fraud | Establish that the person and request are trustworthy enough to assess | Continue, request evidence or stop |
| Affordability | Calculate sustainable installment capacity | Limit the offer or decline |
| Credit risk | Estimate probability of non-payment | Set risk band, limit and price |
| Commercial segmentation | Adapt experience/benefits without redefining risk | Apply permitted commercial policy |
| Offer construction | Produce terms within all previous constraints | Create one immutable active offer |

## 5. Hard eligibility rules

All hard rules must pass before a favorable decision:

1. Applicant meets the configured minimum and maximum age at the expected maturity date.
2. Applicant declares residence in a supported market.
3. Identity verification is valid and unexpired.
4. The required consent set is complete and valid.
5. Requested amount and selected term are within the product policy.
6. No duplicate active application exists for the same customer.
7. No active loan exists for the customer in MVP v1.
8. No confirmed high-severity fraud or blocked-party signal exists in the simulated checks.
9. Declared income meets the product minimum.
10. Calculated payment capacity is greater than zero.

Age, market and product thresholds are configuration values. They are intentionally not presented as legal or regulatory requirements.

## 6. Affordability policy

The demo uses a transparent payment-to-income calculation:

```text
maximumTotalMonthlyDebtService = verifiedNetMonthlyIncome × paymentToIncomeLimit
availableMonthlyInstallment = max(0,
    maximumTotalMonthlyDebtService - verifiedMonthlyDebtObligations)
```

Baseline `paymentToIncomeLimit` by risk band:

| Risk band | Limit |
| --- | ---: |
| `Low` | 35% |
| `Medium` | 30% |
| `High` | 25% |

The amount that can be supported by the installment is calculated using the fixed-payment present-value formula:

```text
affordablePrincipal = installment × (1 - (1 + monthlyRate)^(-termMonths)) / monthlyRate
```

The calculation records:

- verified income and its evidence source;
- existing obligations;
- applicable payment-to-income limit;
- available installment;
- rate and term evaluated;
- rounding policy;
- formula version.

Zero or negative payment capacity produces `INSUFFICIENT_PAYMENT_CAPACITY`. A lower capacity may reduce the offered amount rather than automatically declining the application.

## 7. Risk assessment

The MVP uses a deterministic, reproducible `RiskScore` from 0 to 1,000. It is a demonstration model, not machine learning and not a real bureau score.

| Factor | Maximum points | Example signals |
| --- | ---: | --- |
| Identity confidence | 200 | verification level, consistency of identity data |
| Income stability | 250 | income source, tenure/stability signal, evidence quality |
| Debt burden | 250 | payment-to-income after proposed installment |
| Simulated credit behavior | 200 | delinquencies, recent inquiries, prior performance |
| Application consistency | 100 | conflicting data and non-severe fraud indicators |
| **Total** | **1,000** | Immutable input snapshot and formula version required |

Baseline bands:

| Score | Risk band | Credit treatment |
| ---: | --- | --- |
| 750–1,000 | `Low` | Highest permitted limit; lowest configured rate tier |
| 600–749 | `Medium` | Moderate limit and rate tier |
| 500–599 | `High` | Smallest limit, shortest permitted term and highest configured rate tier |
| Below 500 | Not eligible | Unfavorable decision with `RISK_SCORE_BELOW_THRESHOLD` |

A hard fraud or identity rule overrides the numeric score. A score never acts as the decision by itself.

## 8. Customer segmentation

Commercial segment and risk band remain separate dimensions.

| Segment | Definition | Permitted effect |
| --- | --- | --- |
| `New` | No activated loan history in the platform | Conservative exposure cap; standard onboarding |
| `Standard` | Prior loan completed without severe delinquency in simulated history | Higher permitted exposure within affordability |
| `Preferred` | Repeated positive simulated behavior and strong identity/income evidence | Best permitted rate tier and simpler evidence requests |

Segmentation may improve terms only within affordability, risk and product limits. It cannot override identity, fraud or responsible-lending guards.

## 9. Offer construction

For each supported term, Credit Decisioning calculates:

```text
maximumEligibleAmount = min(
    requestedAmount,
    affordablePrincipal,
    riskBandLimit,
    customerSegmentLimit,
    productMaximumAmount)
```

An offer is created only when `maximumEligibleAmount` is at or above the product minimum. The applicant receives up to three precomputed alternatives:

- requested or maximum eligible amount at the recommended term;
- a lower installment through a longer permitted term;
- a lower total cost through a shorter permitted term.

The applicant selects exactly one alternative before acceptance. Selection creates the single active `CreditOffer`; it does not mutate a published offer. Every alternative and final offer carries amount, currency, term, payment frequency, installment, canonical rate, disclosed equivalent rate, total repayment, expiry, decision ID and rule-set version.

## 10. Rate and pricing convention

The canonical calculation rate is `MonthlyEffectiveRate`. Contracts and customer views also show a derived `AnnualEffectiveRate` for clarity:

```text
annualEffectiveRate = (1 + monthlyEffectiveRate)^12 - 1
```

The monthly effective rate is the source of truth for installment calculation. Conversion precision and monetary rounding belong to a versioned calculation policy. The MVP uses configured fictional tiers and must not imply compliance with any country's interest-rate ceiling.

| Risk/segment treatment | Illustrative monthly effective tier |
| --- | ---: |
| Low / Preferred | 1.50% |
| Low / New or Standard | 1.70% |
| Medium | 1.90% |
| High | 2.10% |

## 11. Decision outcomes

The credit decision remains binary in MVP v1:

- `Favorable`: permits generation of offer alternatives.
- `Unfavorable`: produces no offer and includes one or more reason codes.

Operational uncertainty—provider timeout, missing evidence or inconsistent external result—is not an unfavorable credit decision. It leaves the process pending, triggers controlled retry/evidence collection, or becomes an operational exception.

Priority for multiple outcomes:

1. identity/fraud safety stop;
2. product eligibility;
3. affordability;
4. risk threshold;
5. limit and pricing calculation.

## 12. Reason-code disclosure

| Audience | Disclosure rule | Examples |
| --- | --- | --- |
| Applicant | Safe, actionable and non-sensitive reason plus localized explanation | `REQUIRED_CONSENT_MISSING`, `IDENTITY_EVIDENCE_INSUFFICIENT`, `INSUFFICIENT_PAYMENT_CAPACITY`, `PRODUCT_NOT_ELIGIBLE` |
| Credit/operations analyst | Detailed policy reason and relevant calculated values | `RISK_SCORE_BELOW_THRESHOLD`, rule ID, score band, failed guard |
| Security/fraud operations | Restricted fraud signal and provider evidence reference | `HIGH_RISK_FRAUD_SIGNAL`, `IDENTITY_DATA_MISMATCH` |

Fraud rules, security thresholds and provider raw responses are never disclosed to the applicant or emitted in public integration events.

## 13. Validity and reassessment

| Artifact | Baseline validity |
| --- | --- |
| Identity verification | 30 days for the same customer, unless risk signals require renewal |
| Financial input snapshot | 24 hours for an unaccepted offer |
| Credit decision | 24 hours |
| Offer | 24 hours |
| OTP challenge | 5 minutes, one use, maximum 5 attempts |
| Signature envelope | 24 hours or the earlier offer/package expiry |

Any material change to income, obligations, requested terms, identity status or applicable rule-set version requires a new assessment. Published decisions remain immutable and linked through a reevaluation sequence.

## 14. Loan booking decisions

- The first installment date is based on the disbursement business date plus one monthly cycle, adjusted by a configured business-day convention.
- The repayment schedule may be calculated at reservation but becomes effective only on activation.
- A terminal disbursement failure expires the pending reservation after the configured recovery window; it does not create an active loan.
- A confirmed disbursement with a failed activation enters critical reconciliation and retries activation. It is never automatically treated as a declined application.

## 15. Configuration and governance

The following artifacts are versioned and immutable after publication:

- `ProductPolicy`;
- `EligibilityRuleSet`;
- `AffordabilityPolicy`;
- `RiskScoreModel`;
- `PricingPolicy`;
- `OfferConstructionPolicy`;
- `ReasonCodeCatalog`.

Every assessment stores the exact versions used. A configuration change applies only to new assessments; it does not silently change a published decision or accepted offer.

## 16. Decisions deferred beyond MVP

- country-specific regulatory policy and legal disclosures;
- real KYC, AML, sanctions, bureau and open-finance providers;
- formal analyst override and manual underwriting;
- multiple simultaneous products or revolving credit;
- guarantors, collateral and co-borrowers;
- dynamic fees, insurance and tax calculations;
- collections, delinquency and refinancing;
- model governance for machine-learning decisions.

## 17. Formal decision table

The executable rule precedence, operational dispositions, guard outcomes, risk/segment matrices, pricing, canonical scenarios and test traceability are defined in [`CREDIT_DECISION_TABLE.md`](CREDIT_DECISION_TABLE.md).

`PRODUCT_AND_CREDIT_POLICY.md` explains policy intent; `CREDIT_DECISION_TABLE.md` is the normative baseline for deterministic implementation and automated tests. If an example appears inconsistent, the versioned decision table takes precedence for `credit-decisioning-service` behavior.
