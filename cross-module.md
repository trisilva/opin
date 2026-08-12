# Across the modules

What only makes sense above a single module: how the entities relate to each other, and the one flow
that spans every coverage type.

If the vocabulary here is unfamiliar, read [Insurance concepts](concepts.md) first.

## How the entities relate

```mermaid
erDiagram
    INSURANCE_ENTITY ||--o{ PRODUCT : "issues"
    PRODUCT ||--o| MOTOR_COVERAGE : "instance"
    PRODUCT ||--o| TRAVEL_COVERAGE : "instance"
    PRODUCT ||--o| TERM_LIFE_COVERAGE : "instance"
    PRODUCT ||--o| PROPERTY_COVERAGE : "instance"
    PRODUCT ||--o| CYBER_COVERAGE : "instance"
    PRODUCT ||--o| BI_COVERAGE : "instance"
    PRODUCT ||--o| TRADE_CREDIT_COVERAGE : "instance"
    PRODUCT ||--o| PET_COVERAGE : "instance"
    PERSONAL ||--o{ MOTOR_COVERAGE : "policyholder"
    PERSONAL ||--o{ TRAVEL_COVERAGE : "policyholder"
    PERSONAL ||--o{ TERM_LIFE_COVERAGE : "policyholder"
    PERSONAL ||--o{ PROPERTY_COVERAGE : "policyholder"
    PERSONAL ||--o{ PET_COVERAGE : "policyholder"
    COMMERCIAL ||--o{ PROPERTY_COVERAGE : "policyholder"
    COMMERCIAL ||--o{ CYBER_COVERAGE : "policyholder"
    COMMERCIAL ||--o{ BI_COVERAGE : "policyholder"
    COMMERCIAL ||--o{ TRADE_CREDIT_COVERAGE : "policyholder"
    BENEFICIARY ||--o{ TERM_LIFE_COVERAGE : "named in"
    MOTOR_COVERAGE ||--o{ CLAIM : "produces"
    TRAVEL_COVERAGE ||--o{ CLAIM : "produces"
    TERM_LIFE_COVERAGE ||--o{ CLAIM : "produces"
    PROPERTY_COVERAGE ||--o{ CLAIM : "produces"
    CYBER_COVERAGE ||--o{ CLAIM : "produces"
    BI_COVERAGE ||--o{ CLAIM : "produces"
    TRADE_CREDIT_COVERAGE ||--o{ CLAIM : "produces"
    PET_COVERAGE ||--o{ CLAIM : "produces"
    MOTOR_COVERAGE ||--o{ RECEIPT : "generates"
    CLAIM ||--o{ RECEIPT : "settles via"
    MOTOR_COVERAGE ||--o{ PREMIUM_BORDEREAU : "ceded in"
    CLAIM ||--o{ CLAIMS_BORDEREAU : "ceded in"
```

Read that diagram in five steps.

**A product becomes a coverage.** An insurer publishes products, and a product instantiates exactly
one of the eight coverage types. The product is what is for sale; the coverage record is what one
customer holds.

**A party holds the coverage.** `Personal` or `Commercial` is the policyholder. Which of the two can
hold which coverage is not arbitrary: cyber, business interruption and trade credit are commercial
lines and have no personal policyholder, while pet and term life are the reverse. Property takes
both.

**A beneficiary attaches at policy level.** Canonically on [term life](05-term-life/), where the
insured cannot collect their own death benefit.

**A claim arises from any coverage.** One `Claim` entity serves all eight. A receipt records the
cash, in either direction: premium coming in, settlement going out.

**Bordereaux report to reinsurers.** `PremiumBordereau` and `ClaimsBordereau` are periodic reports
of what has been ceded. They are reports rather than transactions, which is why figures in them also
exist elsewhere.

**Everything hangs on one key.** Claim and Receipt reach their coverage through `policyNumber`,
which is globally unique across the namespace. No foreign key field carries that relationship; the
uniqueness rule does. It is set out in [`SCOPE.md`](SCOPE.md).

## The one flow that spans every coverage type

Submitting a claim is the only flow that crosses modules, and it works the same way whichever of the
eight coverage types is involved.

```mermaid
sequenceDiagram
    participant Insured as Insured Portal
    participant Gateway as API Gateway
    participant Claim as Claim Service
    participant CoverageRouter as Coverage Router
    participant Cov as Coverage Service (motor/travel/life/property/cyber/BI/tradeCredit/pet)
    Insured->>Gateway: POST /claim {policyNumber, fnol, lossDate, lossCause, description, documents}
    Gateway->>Claim: createClaim
    Claim->>CoverageRouter: lookupCoverage(policyNumber)
    CoverageRouter->>Cov: route to the coverage type that owns policyNumber
    Cov-->>Claim: coverage details, perils, indemnity, deductibles
    Claim->>Claim: validate lossCause within covered perils
    Claim->>Claim: persist with claimStatus=open, set initial reserve
    Claim-->>Gateway: 201 {claimNumber, status: open}
    Gateway-->>Insured: 201 Created
```

**One endpoint serves every coverage type.** The caller does not say what kind of policy it is
claiming against, and does not need to. The coverage-routing step in the middle of that diagram is
internal to the implementation and never appears on the wire.

**It works because of one rule.** `policyNumber` is globally unique across the namespace, so it
resolves to exactly one coverage record among all eight types. An implementation therefore assigns
policy numbers from **one sequence across every coverage type it writes**. Scope them per line of
business and this endpoint stops working, because a number would no longer identify a single policy.
See [`SCOPE.md`](SCOPE.md).
