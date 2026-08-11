# Module 9: Trade Credit

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class TradeCreditCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +TradeCreditType tradeCreditType
        +TradeCreditPeril[] perils
        +int creditLimit
        +int creditLimitUtilized
        +int maxCreditPeriod
        +int waitingPeriod
        +create() TradeCreditCoverage
        +retrieve(id) TradeCreditCoverage
    }
    class Debtor {
        +UUID id
        +string name
        +string ultimateParentCompany
        +LegalEntity legalForm
        +int netAssets
        +int annualizedTurnover
        +string creditRating
        +create() Debtor
        +retrieve(id) Debtor
        +update(id) Debtor
    }
    TradeCreditCoverage --> Debtor : covers
```

### Endpoints

`[added]`: OPIN publishes the tradeCreditCoverage and debtor schemas but no endpoints.

- `POST /tradeCreditCoverage` (admin)
- `GET /tradeCreditCoverage` (developer)
- `GET /tradeCreditCoverage/{id}` (developer)
- `PUT /tradeCreditCoverage/{id}` (admin)
- `POST /tradeCreditCoverage/{id}:endorse` (admin), supports credit limit changes via OPIN `endorsementType=addition/increase` or `deletion/decrease`
- `POST /tradeCreditCoverage/{id}:cancel` (admin)
- `POST /tradeCreditCoverage/{id}:renew` (admin)
- `POST /debtor` (admin)
- `GET /debtor` (developer)
- `GET /debtor/{id}` (developer)
- `PUT /debtor/{id}` (admin)

### Primary flow: Bind a single-buyer trade credit policy

```mermaid
sequenceDiagram
    participant Client as Broker
    participant Gateway as API Gateway
    participant TC as Trade Credit Service
    Client->>Gateway: POST /debtor {name, registrationNumber, parentCompany, financials}
    Gateway->>TC: createDebtor
    TC-->>Gateway: 201 {debtorId}
    Client->>Gateway: POST /tradeCreditCoverage {policyholderId, debtorId, tradeCreditType, creditLimit, maxCreditPeriod}
    Gateway->>TC: createTradeCreditCoverage
    TC->>TC: validate tradeCreditType against tradeCreditTpe enum
    TC->>TC: validate perils against tradeCreditPeril enum
    TC->>TC: Persist with policyStatus=in force
    TC-->>Gateway: 201 {policyNumber}
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /tradeCreditCoverage
    InForce --> InForce : :endorse (credit limit adjustment under endorsementType)
    InForce --> Cancelled : :cancel
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    Cancelled --> [*]
    Lapsed --> [*]
```

`[OPIN concern]`: OPIN ships a `waitingPeriod` field on tradeCreditCoverage but no operational signal for entering or exiting it. This standard does not invent a waiting-period state; the waiting period is a derived calculation against the loss-event date when a claim is filed.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /tradeCreditCoverage] --> B{Debtor exists?}
    B -->|No| E1[404 - debtor not found]
    B -->|Yes| C{tradeCreditType in tradeCreditTpe enum?}
    C -->|No| E2[400 - unknown trade credit type]
    C -->|Yes| D{Credit limit &gt; 0?}
    D -->|No| E3[400 - invalid credit limit]
    D -->|Yes| F{All perils in tradeCreditPeril enum?}
    F -->|No| E4[400 - unknown peril]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
