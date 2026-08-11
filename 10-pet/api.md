# Module 10: Pet

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class PetCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +int annualReimbursementLimit
        +int waitingPeriod
        +bool preexistingConditions
        +PetBenefit[] benefits
        +create() PetCoverage
        +retrieve(id) PetCoverage
        +renew(id) PetCoverage
    }
    class Pet {
        +UUID id
        +string petName
        +PetKind petKind
        +float age
        +bool purebred
        +PetBreed petBreed
        +float reimbursement
    }
    PetCoverage --> Pet : covers
```

### Endpoints

`[added]`: OPIN publishes both the petCoverage and pet schemas but no endpoints.

- `POST /petCoverage` (admin)
- `GET /petCoverage` (developer)
- `GET /petCoverage/{id}` (developer)
- `PUT /petCoverage/{id}` (admin)
- `POST /petCoverage/{id}:endorse` (admin)
- `POST /petCoverage/{id}:cancel` (admin)
- `POST /petCoverage/{id}:renew` (admin)
- `POST /pet` (admin)
- `GET /pet` (developer)
- `GET /pet/{id}` (developer)
- `PUT /pet/{id}` (admin)

### Primary flow: Bind a pet policy

```mermaid
sequenceDiagram
    participant Client as Pet Owner App
    participant Gateway as API Gateway
    participant Pet as Pet Service
    Client->>Gateway: POST /pet {petName, petKind, age, breed if dog}
    Gateway->>Pet: createPet
    Pet-->>Gateway: 201 {petId}
    Client->>Gateway: POST /petCoverage {policyholderId, petId, benefits, annualReimbursementLimit, waitingPeriod}
    Gateway->>Pet: createPetCoverage
    Pet->>Pet: validate benefits against petBenefits enum
    Pet->>Pet: Persist with policyStatus=in force
    Pet-->>Gateway: 201 {policyNumber, status: in force}
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /petCoverage
    InForce --> InForce : :endorse (endorsementType applied)
    InForce --> Cancelled : :cancel
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    Cancelled --> [*]
    Lapsed --> [*]
```

`[OPIN concern]`: OPIN ships a `waitingPeriod` field on petCoverage but, as with tradeCreditCoverage, no lifecycle state for it. This standard treats the waiting period as a derived calculation against `inceptionDate` when a claim is evaluated; the policy itself is `in force` from binding.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /petCoverage] --> B{Pet exists?}
    B -->|No| E1[404 - pet not found]
    B -->|Yes| C{petKind in petKind enum?}
    C -->|No| E2[400 - unknown petKind]
    C -->|Yes| D{petKind=dog implies petBreed set?}
    D -->|No| E3[400 - dog breed required]
    D -->|Yes| F{Benefits all in petBenefits enum?}
    F -->|No| E4[400 - unknown benefit]
    F -->|Yes| G[Persist with policyStatus=in force]
    G --> H[201 Created]
```
