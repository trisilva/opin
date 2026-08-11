# Module 7: Cyber Liability

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class CyberLiabilityCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +int indemnityLimitPolicy
        +bool claimsOccurrence
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

### Endpoints

`[added]`: OPIN publishes the cyberLiabilityCoverage and business schemas but no endpoints.

- `POST /cyberLiabilityCoverage` (admin)
- `GET /cyberLiabilityCoverage` (developer)
- `GET /cyberLiabilityCoverage/{id}` (developer)
- `PUT /cyberLiabilityCoverage/{id}` (admin)
- `POST /cyberLiabilityCoverage/{id}:endorse` (admin)
- `POST /cyberLiabilityCoverage/{id}:cancel` (admin)
- `POST /cyberLiabilityCoverage/{id}:renew` (admin)
- `POST /business` (admin)
- `GET /business` (developer)
- `GET /business/{id}` (developer)
- `PUT /business/{id}` (admin)

`[OPIN concern]`: OPIN does not publish a `cyberIncident` schema or a breach-notification model. Incident reporting and regulator notification timelines are insurance-adjacent operational concerns that an OPIN v1.1 publication should consider, but they are out of scope for this version.

### Primary flow: Bind a cyber liability policy

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

### Lifecycle

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

### Routing and error handling

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
