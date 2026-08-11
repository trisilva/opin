# Module 5: Term Life

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class TermLifeCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +int freeCoverLimit
        +int totalSumInsured
        +TermLifeType termLifeType
        +TermLifeRider[] coverRiders
        +create() TermLifeCoverage
        +retrieve(id) TermLifeCoverage
        +endorse(id) TermLifeCoverage
    }
    class LifeInsured {
        +UUID id
        +string firstName
        +string lastName
        +Date dob
        +int annualSalary
        +int sumInsured
    }
    class Beneficiary {
        +UUID id
        +string name
        +float share
    }
    TermLifeCoverage --> LifeInsured : covers
    TermLifeCoverage --> Beneficiary : pays
```

### Endpoints

`[added]`: OPIN publishes the termLifeCoverage and lifeInsured schemas but no endpoints. Beneficiary CRUD lives in Module 1.

- `POST /termLifeCoverage` (admin)
- `GET /termLifeCoverage` (developer)
- `GET /termLifeCoverage/{id}` (developer)
- `PUT /termLifeCoverage/{id}` (admin)
- `POST /termLifeCoverage/{id}:endorse` (admin)
- `POST /termLifeCoverage/{id}:cancel` (admin)
- `POST /termLifeCoverage/{id}:renew` (admin)
- `POST /lifeInsured` (admin)
- `GET /lifeInsured` (developer)
- `GET /lifeInsured/{id}` (developer)
- `PUT /lifeInsured/{id}` (admin)

### Primary flow: Bind a term life policy with riders

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Gateway as API Gateway
    participant Life as Term Life Service
    participant Party as Party Service
    Client->>Gateway: POST /lifeInsured {firstName, dob, occupation, annualSalary}
    Gateway->>Life: createLifeInsured(payload)
    Life-->>Gateway: 201 {lifeInsuredId}
    Client->>Gateway: POST /termLifeCoverage {policyholderId, lifeInsuredId, sumInsured, riders, termLifeType}
    Gateway->>Life: createTermLifeCoverage(payload)
    Life->>Party: validate policyholderId
    Life->>Life: validate riders against termLifeRiders enum
    Life->>Life: Persist with policyStatus=in force
    Life-->>Gateway: 201 {policyNumber, status: in force}
    Gateway-->>Client: 201 Created
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /termLifeCoverage
    InForce --> InForce : :endorse (rider added/removed under endorsementType)
    InForce --> Cancelled : :cancel (surrender)
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    Cancelled --> [*]
    Lapsed --> [*]
```

States are exactly OPIN `policyStatus`. Underwriting outcomes (declined, awaiting medicals) and claim settlement are out of scope here. Underwriting belongs to a future OPIN module not in v1.0; settlement of a death claim is the same flow as any other claim and lives in Module 11.

`[OPIN concern]`: OPIN does not model underwriting (quote, decision, decline). An OPIN v1.1 underwriting module would close this gap.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /termLifeCoverage] --> B{LifeInsured exists?}
    B -->|No| E1[404 - lifeInsured not found]
    B -->|Yes| C{Riders all in termLifeRiders enum?}
    C -->|No| E2[400 - unknown rider]
    C -->|Yes| D{termLifeType in termLifeType enum?}
    D -->|No| E3[400 - unknown term life type]
    D -->|Yes| F{Inception &lt; expiry?}
    F -->|No| E4[400 - invalid policy term]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
