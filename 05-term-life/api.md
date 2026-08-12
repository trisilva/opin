# Module 5: Term Life

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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

`termLifeCoverage` carries a multi-valued `beneficiary` reference. The beneficiary records
themselves are created and maintained through [core parties](../01-core-parties/), because a
beneficiary is a party like any other and can be named on more than one policy.

## Endpoints

Beneficiaries are created and maintained through [core parties](../01-core-parties/), not here.

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /termLifeCoverage` | admin | Bind a policy against an existing life insured |
| `GET /termLifeCoverage` | developer | List, filterable by `policyNumber` |
| `GET /termLifeCoverage/{id}` | developer | Retrieve one policy |
| `PUT /termLifeCoverage/{id}` | admin | Replace a policy |
| `POST /termLifeCoverage/{id}:endorse` | admin | Amend a policy in force, including adding or removing a rider |
| `POST /termLifeCoverage/{id}:cancel` | admin | Surrender the policy |
| `POST /termLifeCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |
| `POST /lifeInsured` | admin | Create a life insured |
| `GET /lifeInsured` | developer | List |
| `GET /lifeInsured/{id}` | developer | Retrieve one |
| `PUT /lifeInsured/{id}` | admin | Replace one |

A death claim goes to `POST /claim` like any other claim. See [Claims](../11-claims/).

## Primary flow: bind a policy with riders

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

Two things this lifecycle does not contain, and both surprise people.

**There is no underwriting.** No quote, no decision, no declined state, no awaiting-medicals state.
The record begins at in force, so everything between an application and a bound policy happens
before the standard sees it. This is a genuine gap rather than a scope decision, and nothing in the
standard closes it today.

**There is no settled state.** A death claim does not move the policy. It runs through
[Claims](../11-claims/) on the shared claim lifecycle, exactly as a motor or travel claim does.

## Errors

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
