# Across the modules

What only makes sense above a single module: how the entities relate to each other, and the one
flow that spans every coverage type.

## Cross-module relationships (composite view)

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

The composite view shows OPIN's coverage-centric model: Product instantiates one of the eight coverage types, Personal or Commercial parties act as policyholders, Beneficiary attaches at policy level (canonically term life), Claim arises from any coverage, Receipt records the cash leg, and PremiumBordereau/ClaimsBordereau are reinsurance-side reports. Claim and Receipt reach their coverage through `policyNumber`, which this standard declares globally unique across the namespace. That rule is what carries the two associations the inherited schema left implicit, and it is set out in [`SCOPE.md`](SCOPE.md).

---

## Cross-module primary flow

This standard exposes one cross-module primary flow: the universal claim submission flow. It is OPIN-aligned because OPIN's `POST /claim` is itself polymorphic across coverages.

### Flow A: Submit claim (universal, polymorphic across coverages)

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

`[OPIN]`: the `POST /claim` entry point is exactly OPIN's. The internal coverage-routing step is implementer-side, hidden from the wire contract. From the caller's perspective, one endpoint serves all coverage types.

`[added]`: the polymorphism works because `policyNumber` resolves deterministically to a single coverage record across all eight coverage types. The inherited standard assumed that constraint without declaring it. This version declares it: `policyNumber` is globally unique across the namespace, and an implementation assigns policy numbers from one sequence across every coverage type it writes. See [`SCOPE.md`](SCOPE.md).
