# Module 6: Property

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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

## Endpoints

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /propertyCoverage` | admin | Bind a policy against one or more existing properties |
| `GET /propertyCoverage` | developer | List, filterable by `policyNumber` |
| `GET /propertyCoverage/{id}` | developer | Retrieve one policy |
| `PUT /propertyCoverage/{id}` | admin | Replace a policy |
| `POST /propertyCoverage/{id}:endorse` | admin | Amend a policy in force |
| `POST /propertyCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /propertyCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |
| `POST /property` | admin | Create a property |
| `GET /property` | developer | List properties |
| `GET /property/{id}` | developer | Retrieve one property |
| `PUT /property/{id}` | admin | Replace a property |

[Business interruption](../08-business-interruption/) is normally written alongside this cover on
the same premises, and it is a separate policy record.

## Primary flow: bind a buildings and contents policy

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

Endorsing keeps the policy in force. Renewing writes a second record rather than moving this one
forward.

## Errors

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
