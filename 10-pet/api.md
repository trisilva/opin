# Module 10: Pet

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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

## Endpoints

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /petCoverage` | admin | Bind a policy against an existing pet |
| `GET /petCoverage` | developer | List, filterable by `policyNumber` |
| `GET /petCoverage/{id}` | developer | Retrieve one policy |
| `PUT /petCoverage/{id}` | admin | Replace a policy |
| `POST /petCoverage/{id}:endorse` | admin | Amend a policy in force |
| `POST /petCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /petCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |
| `POST /pet` | admin | Create a pet |
| `GET /pet` | developer | List pets |
| `GET /pet/{id}` | developer | Retrieve one pet |
| `PUT /pet/{id}` | admin | Replace a pet |

## Primary flow: bind a pet policy

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

There is no waiting state, although `waitingPeriod` is a field on the record. The policy is in force
from binding, and the waiting period is calculated against `inceptionDate` when a claim is
evaluated. [Trade credit](../09-trade-credit/) handles its own waiting period the same way.

## Errors

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
