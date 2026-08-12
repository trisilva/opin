# Module 8: Business Interruption

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

```mermaid
classDiagram
    class BusinessInterruptionCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +TypeBusinessInterruption type
        +PropertyPeril[] perils
        +int grossAnnualProfit
        +int icwLimit
        +int contingencyLossLimit
        +int IndemnityPeriod
        +int deductible
        +create() BusinessInterruptionCoverage
        +retrieve(id) BusinessInterruptionCoverage
    }
    class Property {
        +UUID id
        +Address address
    }
    BusinessInterruptionCoverage --> Property : attached to
```

`businessInterruptionCoverage` carries a `propertyRef`. The premises it points at is created through
[property](../06-property/), which is why the flow below writes the property first.

## Endpoints

`Property` is created through [property](../06-property/). There is no property endpoint here.

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /businessInterruptionCoverage` | admin | Bind a policy |
| `GET /businessInterruptionCoverage` | developer | List, filterable by `policyNumber` |
| `GET /businessInterruptionCoverage/{id}` | developer | Retrieve one policy |
| `PUT /businessInterruptionCoverage/{id}` | admin | Replace a policy |
| `POST /businessInterruptionCoverage/{id}:endorse` | admin | Amend a policy in force |
| `POST /businessInterruptionCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /businessInterruptionCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |

## Primary flow: bind alongside a property policy

Three writes, in order, because this cover attaches to a premises that has to exist first and is
normally sold with the property policy over the same premises.

```mermaid
sequenceDiagram
    participant Client as Broker
    participant Gateway as API Gateway
    participant BI as BI Service
    participant Property as Property Service
    Client->>Gateway: POST /property {address, propertyType, sumInsuredBuilding}
    Gateway->>Property: createProperty
    Property-->>Gateway: 201 {propertyId}
    Client->>Gateway: POST /propertyCoverage {policyholderId, propertyId, perils}
    Gateway-->>Client: 201 {propertyPolicyNumber}
    Client->>Gateway: POST /businessInterruptionCoverage {propertyId, type, grossAnnualProfit, IndemnityPeriod, perils}
    Gateway->>BI: createBusinessInterruptionCoverage
    BI->>Property: validate underlying property exists
    BI->>BI: validate type in typeBusinessInterruption enum
    BI->>BI: Persist with policyStatus=in force
    BI-->>Gateway: 201 {policyNumber}
```

## Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /businessInterruptionCoverage
    InForce --> InForce : :endorse (endorsementType applied)
    InForce --> Cancelled : :cancel
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    Cancelled --> [*]
    Lapsed --> [*]
```

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

## Errors

```mermaid
flowchart TD
    A[POST /businessInterruptionCoverage] --> B{Property exists?}
    B -->|No| E1[404 - underlying property not found]
    B -->|Yes| C{Type in typeBusinessInterruption enum?}
    C -->|No| E2[400 - unknown BI type]
    C -->|Yes| D{IndemnityPeriod &gt; 0?}
    D -->|No| E3[400 - invalid indemnity period]
    D -->|Yes| F{All perils in propertyPeril enum?}
    F -->|No| E4[400 - unknown peril]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
