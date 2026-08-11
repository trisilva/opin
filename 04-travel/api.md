# Module 4: Travel

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class TravelCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +bool isAnnualPolicy
        +bool isGroup
        +int length
        +int destination
        +float emergencyMedical
        +float tripCancellation
        +create() TravelCoverage
        +retrieve(id) TravelCoverage
        +cancel(id) TravelCoverage
    }
    class Traveller {
        +UUID id
        +string firstName
        +string lastName
        +string nationality
        +Date dob
        +create() Traveller
        +retrieve(id) Traveller
        +update(id) Traveller
    }
    TravelCoverage --> Traveller : covers
```

### Endpoints

`[added]`: OPIN publishes the travelCoverage and traveller schemas but no endpoints. CRUD is added here, modelled on the motor pattern.

- `POST /travelCoverage` (admin)
- `GET /travelCoverage` (developer)
- `GET /travelCoverage/{id}` (developer)
- `PUT /travelCoverage/{id}` (admin)
- `POST /travelCoverage/{id}:cancel` (admin)
- `POST /traveller` (admin)
- `GET /traveller` (developer)
- `GET /traveller/{id}` (developer)
- `PUT /traveller/{id}` (admin)

### Primary flow: Bind a single-trip travel policy

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Gateway as API Gateway
    participant Travel as Travel Service
    participant Party as Party Service
    Client->>Gateway: POST /traveller {firstName, lastName, dob, nationality, idType}
    Gateway->>Travel: createTraveller(payload)
    Travel-->>Gateway: 201 {travellerId}
    Client->>Gateway: POST /travelCoverage {policyholderId, travellerIds, destination, length, isAnnualPolicy: false}
    Gateway->>Travel: createTravelCoverage(payload)
    Travel->>Party: validate policyholderId
    Travel->>Travel: validate length consistent with isAnnualPolicy
    Travel->>Travel: Persist with policyStatus=in force
    Travel-->>Gateway: 201 {policyNumber, status: in force}
    Gateway-->>Client: 201 Created
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /travelCoverage
    InForce --> Cancelled : :cancel
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    Cancelled --> [*]
    Lapsed --> [*]
```

States are exactly OPIN `policyStatus`. Trip expiry is a derived property from `expiryDate` and is not a separate lifecycle state.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /travelCoverage] --> B{isGroup consistent with travellerIds count?}
    B -->|isGroup false but multiple travellers| E1[400 - group flag required]
    B -->|consistent| C{Destination valid country reference?}
    C -->|No| E2[400 - invalid destination]
    C -->|Yes| D{length consistent with isAnnualPolicy?}
    D -->|No| E3[400 - length out of range for policy type]
    D -->|Yes| F[Persist with policyStatus=in force]
    F --> G[201 Created]
```
