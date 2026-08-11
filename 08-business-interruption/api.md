# Module 8: Business Interruption

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

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

### Endpoints

`[added]`: OPIN publishes the businessInterruptionCoverage schema but no endpoints. The Property schema is reused from Module 6.

- `POST /businessInterruptionCoverage` (admin)
- `GET /businessInterruptionCoverage` (developer)
- `GET /businessInterruptionCoverage/{id}` (developer)
- `PUT /businessInterruptionCoverage/{id}` (admin)
- `POST /businessInterruptionCoverage/{id}:endorse` (admin)
- `POST /businessInterruptionCoverage/{id}:cancel` (admin)
- `POST /businessInterruptionCoverage/{id}:renew` (admin)

### Primary flow: Bind a BI policy alongside property

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

### Lifecycle

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

### Routing and error handling

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
