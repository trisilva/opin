# Module 1: Core Parties and Entities

The endpoints, the flow that onboards a party, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

```mermaid
classDiagram
    class InsuranceEntity {
        +UUID id
        +string name
        +string tradeName
        +EntityType type
        +EntityClassification classification
        +string registrationNumber
        +Address address
        +create() InsuranceEntity
        +retrieve(id) InsuranceEntity
        +update(id) InsuranceEntity
        +list(filters) InsuranceEntity[]
    }
    class Personal {
        +UUID id
        +string firstName
        +string lastName
        +Date dob
        +Address address
        +create() Personal
        +retrieve(id) Personal
        +update(id) Personal
        +list(filters) Personal[]
    }
    class Commercial {
        +UUID id
        +string name
        +Address registeredAddress
        +string registrationNumber
        +create() Commercial
        +retrieve(id) Commercial
        +update(id) Commercial
        +list(filters) Commercial[]
    }
    class Beneficiary {
        +UUID id
        +string name
        +Address address
        +float share
        +create() Beneficiary
        +retrieve(id) Beneficiary
        +update(id) Beneficiary
    }
    class Address {
        +string building
        +string streetName
        +string city
        +string state
        +string country
        +string postalCode
        +string threeWordAddress
    }
    InsuranceEntity --> Address : has
    Personal --> Address : has
    Commercial --> Address : has
    Beneficiary --> Address : has
```

Address has no class of its own here because it is not a resource. It is embedded in whichever
entity owns it, and it travels inline on every request and response that carries one.

## Endpoints

Four resources, all the same shape: create and list on the collection, retrieve and replace on the
item. `opin-vn.admin` is the write scope and `opin-vn.developer` is read-only.

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /insuranceEntity` | admin | Create an insurer, broker or agent |
| `GET /insuranceEntity` | developer | List, searchable by name or registration number |
| `GET /insuranceEntity/{id}` | developer | Retrieve one |
| `PUT /insuranceEntity/{id}` | admin | Replace one |
| `POST /personal` | admin | Create an individual party |
| `GET /personal` | developer | List, searchable |
| `GET /personal/{id}` | developer | Retrieve one |
| `PUT /personal/{id}` | admin | Replace one |
| `POST /commercial` | admin | Create a corporate party |
| `GET /commercial` | developer | List, searchable |
| `GET /commercial/{id}` | developer | Retrieve one |
| `PUT /commercial/{id}` | admin | Replace one |
| `POST /beneficiary` | admin | Create a beneficiary |
| `GET /beneficiary` | developer | List |
| `GET /beneficiary/{id}` | developer | Retrieve one |
| `PUT /beneficiary/{id}` | admin | Replace one |

There is no `/address`. See the note above.

## Primary flow: onboard an individual party

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Gateway as API Gateway
    participant Service as Party Service
    participant Store as Party Store
    Client->>Gateway: POST /personal {firstName, lastName, dob, address, idType, idNumber}
    Gateway->>Gateway: Validate OAuth scope opin-vn.admin
    Gateway->>Service: createPersonal(payload)
    Service->>Service: Validate ISO 3166-1 country, ISO 639-2 language refs
    Service->>Service: De-duplicate by idNumber+idType
    Service->>Store: Persist
    Store-->>Service: id
    Service-->>Gateway: 201 Created {id, ...}
    Gateway-->>Client: 201 Created
```

Two things in that flow are worth pulling out. A party is deduplicated on `idType` plus `idNumber`
together, not on name, because names repeat and are spelled inconsistently. And an
`Idempotency-Key` replay returns the original record with a 200 rather than creating a second party,
which is what makes a retried onboarding safe.

## Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Active : POST creates record
    Active --> Active : PUT updates fields
    Active --> [*] : record retained, never hard-deleted
```

The lifecycle is deliberately minimal: a party record exists or it does not, and it is never
deleted. There is no suspended state, no closed state and no identity-check status. Those describe
where a party sits in one operator's process rather than what the party is, so they belong in your
own extension fields.

## Errors

```mermaid
flowchart TD
    A[Inbound POST /personal] --> B{Valid OAuth scope?}
    B -->|No| E1[401 Unauthorized]
    B -->|Yes| C{Required fields present?}
    C -->|No| E2[400 Bad Request - RFC 7807]
    C -->|Yes| D{idNumber+idType unique?}
    D -->|No| E3[409 Conflict - existing party id]
    D -->|Yes| F{Idempotency-Key replay?}
    F -->|Yes| R1[200 OK - idempotent replay]
    F -->|No| G[Persist record]
    G --> H{Persistence success?}
    H -->|No| E4[503 Service Unavailable - retry]
    H -->|Yes| I[201 Created]
```
