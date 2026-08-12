# Module 9: Trade Credit

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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

## Endpoints

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /tradeCreditCoverage` | admin | Bind a policy against an existing debtor |
| `GET /tradeCreditCoverage` | developer | List, filterable by `policyNumber` |
| `GET /tradeCreditCoverage/{id}` | developer | Retrieve one policy |
| `PUT /tradeCreditCoverage/{id}` | admin | Replace a policy |
| `POST /tradeCreditCoverage/{id}:endorse` | admin | Amend a policy in force. This is how a credit limit is raised or lowered, under an `endorsementType` of addition or deletion |
| `POST /tradeCreditCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /tradeCreditCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |
| `POST /debtor` | admin | Create a debtor |
| `GET /debtor` | developer | List debtors |
| `GET /debtor/{id}` | developer | Retrieve one debtor |
| `PUT /debtor/{id}` | admin | Replace a debtor |

Credit limits move often, and `:endorse` is the only route. There is no separate limit endpoint, so
a limit change is an endorsement on a policy that stays in force.

## Primary flow: bind a single-buyer policy

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

There is no waiting-period state, although `waitingPeriod` is a field on the record. The waiting
period is calculated against the loss date at the point a claim is filed, and the policy does not
move while it runs.

## Errors

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
