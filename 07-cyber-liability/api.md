# Module 7: Cyber Liability

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

```mermaid
classDiagram
    class CyberLiabilityCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +int indemnityLimitPolicy
        +ClaimsOccurrence claimsOccurrence
        +CyberCoverageCategory[] scope
        +create() CyberLiabilityCoverage
        +retrieve(id) CyberLiabilityCoverage
        +endorse(id) CyberLiabilityCoverage
    }
    class Business {
        +UUID id
        +string businessSector
        +DataAsset[] dataAssets
        +bool dataSharing
        +float grossAnnualTurnover
        +float numberOfEmployees
        +float itStaff
    }
    CyberLiabilityCoverage --> Business : covers
```

## Endpoints

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /cyberLiabilityCoverage` | admin | Bind a policy against an existing business |
| `GET /cyberLiabilityCoverage` | developer | List, filterable by `policyNumber` |
| `GET /cyberLiabilityCoverage/{id}` | developer | Retrieve one policy |
| `PUT /cyberLiabilityCoverage/{id}` | admin | Replace a policy |
| `POST /cyberLiabilityCoverage/{id}:endorse` | admin | Amend a policy in force |
| `POST /cyberLiabilityCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /cyberLiabilityCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |
| `POST /business` | admin | Create a business |
| `GET /business` | developer | List businesses |
| `GET /business/{id}` | developer | Retrieve one business |
| `PUT /business/{id}` | admin | Replace a business |

There is no incident resource. A cyber loss is filed through `POST /claim` like every other coverage
type. Breach-notification and regulator-notification deadlines sit above the standard, because they
are operational service-level measurement. See [`../SCOPE.md`](../SCOPE.md).

## Primary flow: bind a cyber liability policy

```mermaid
sequenceDiagram
    participant Client as Broker
    participant Gateway as API Gateway
    participant Cyber as Cyber Service
    participant Party as Party Service
    Client->>Gateway: POST /business {businessSector, dataAssets, grossAnnualTurnover, numberOfEmployees}
    Gateway->>Cyber: createBusiness(payload)
    Cyber-->>Gateway: 201 {businessId}
    Client->>Gateway: POST /cyberLiabilityCoverage {policyholderId, businessId, scope, indemnityLimitPolicy}
    Gateway->>Cyber: createCyberLiabilityCoverage(payload)
    Cyber->>Party: validate policyholderId
    Cyber->>Cyber: validate scope against cyberCoverageCategories enum
    Cyber->>Cyber: Persist with policyStatus=in force
    Cyber-->>Gateway: 201 {policyNumber, status: in force}
    Gateway-->>Client: 201 Created
```

## Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /cyberLiabilityCoverage
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
    A[POST /cyberLiabilityCoverage] --> B{Business exists?}
    B -->|No| E1[404 - business not found]
    B -->|Yes| C{All scope categories in cyberCoverageCategories?}
    C -->|No| E2[400 - unknown category]
    C -->|Yes| D{indemnityLimitPolicy &gt; 0?}
    D -->|No| E3[400 - invalid indemnity limit]
    D -->|Yes| F{Inception &lt; expiry?}
    F -->|No| E4[400 - invalid policy term]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
