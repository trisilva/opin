# Module 4: Travel

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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
        +endorse(id) TravelCoverage
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

## Endpoints

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /travelCoverage` | admin | Bind a policy against one or more existing travellers |
| `GET /travelCoverage` | developer | List, filterable by `policyNumber` |
| `GET /travelCoverage/{id}` | developer | Retrieve one policy |
| `PUT /travelCoverage/{id}` | admin | Replace a policy |
| `POST /travelCoverage/{id}:endorse` | admin | Amend a policy in force, including extending the term |
| `POST /travelCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /traveller` | admin | Create a traveller |
| `GET /traveller` | developer | List travellers |
| `GET /traveller/{id}` | developer | Retrieve one traveller |
| `PUT /traveller/{id}` | admin | Replace a traveller |

Extending a trip is an endorsement rather than a new policy, which matters on an annual multi-trip
policy where the customer is still mid-journey.

## Primary flow: bind a single-trip policy

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

The four states are the shared `policyStatus` set. A trip simply ending is not one of them: expiry
is derived from `expiryDate`, so a policy past its expiry date is still recorded as in force rather
than moving to a state of its own.

## Errors

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
