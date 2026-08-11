# Module 6: Property

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class PropertyCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +int indemnityLimitPolicy
        +bool isAgreedValue
        +PropertyPeril[] perils
        +int deductibleAmountBuilding
        +int deductibleAmountContent
        +ClaimsOccurrence claimsOccurrence
        +create() PropertyCoverage
        +retrieve(id) PropertyCoverage
        +endorse(id) PropertyCoverage
    }
    class Property {
        +UUID id
        +Address address
        +string latitude
        +string longitude
        +PropertyType propertyType
        +int sumInsuredBuilding
        +int sumInsuredContent
        +WallConstruction wall
        +RoofConstruction roof
    }
    PropertyCoverage --> Property : insures
```

### Endpoints

`[added]`: OPIN publishes the propertyCoverage and property schemas but no endpoints.

- `POST /propertyCoverage` (admin)
- `GET /propertyCoverage` (developer)
- `GET /propertyCoverage/{id}` (developer)
- `PUT /propertyCoverage/{id}` (admin)
- `POST /propertyCoverage/{id}:endorse` (admin)
- `POST /propertyCoverage/{id}:cancel` (admin)
- `POST /propertyCoverage/{id}:renew` (admin)
- `POST /property` (admin)
- `GET /property` (developer)
- `GET /property/{id}` (developer)
- `PUT /property/{id}` (admin)

### Primary flow: Bind a home and contents policy

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Gateway as API Gateway
    participant Property as Property Service
    Client->>Gateway: POST /property {address, propertyType, wall, roof, sumInsuredBuilding}
    Gateway->>Property: createProperty(payload)
    Property->>Property: validate propertyType, wall, roof against OPIN enums
    Property-->>Gateway: 201 {propertyId}
    Client->>Gateway: POST /propertyCoverage {policyholderId, propertyId, perils, deductibles}
    Gateway->>Property: createPropertyCoverage(payload)
    Property->>Property: persist with policyStatus=in force
    Property-->>Gateway: 201 {policyNumber}
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /propertyCoverage
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
    A[POST /propertyCoverage] --> B{Property exists?}
    B -->|No| E1[404 - property not found]
    B -->|Yes| C{propertyType in OPIN propertyType enum?}
    C -->|No| E2[400 - unknown propertyType]
    C -->|Yes| D{All perils in propertyPeril enum?}
    D -->|No| E3[400 - unknown peril]
    D -->|Yes| F{Inception &lt; expiry?}
    F -->|No| E4[400 - invalid policy term]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
