# Module 1: Core Parties and Entities

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

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

`[OPIN concern]`: Address is an embedded value object in the OPIN data standard. It is not a top-level resource and this standard does not expose it as one. Address fields are only ever sent and received inline within their owning entity.

### Endpoints

`[added]`: OPIN publishes the InsuranceEntity, Personal, Commercial, Beneficiary, and address schemas but no endpoints. CRUD endpoints are added here, modelled on the motor `/vehicle`, `/driver` pattern.

- `POST /insuranceEntity` (admin)
- `GET /insuranceEntity` (developer), search by name or registration number
- `GET /insuranceEntity/{id}` (developer)
- `PUT /insuranceEntity/{id}` (admin)
- `POST /personal` (admin)
- `GET /personal` (developer), search
- `GET /personal/{id}` (developer)
- `PUT /personal/{id}` (admin)
- `POST /commercial` (admin)
- `GET /commercial` (developer), search
- `GET /commercial/{id}` (developer)
- `PUT /commercial/{id}` (admin)
- `POST /beneficiary` (admin)
- `GET /beneficiary` (developer)
- `GET /beneficiary/{id}` (developer)
- `PUT /beneficiary/{id}` (admin)

### Primary flow: Onboard a Personal party

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

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Active : POST creates record
    Active --> Active : PUT updates fields
    Active --> [*] : record retained, never hard-deleted
```

`[OPIN concern]`: OPIN does not declare a party lifecycle. This standard keeps the lifecycle conservative: a party record exists or it does not. Status flags such as KYC status, suspension, or closure are out of scope at this version and live in implementer-specific extensions.

### Routing and error handling

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
