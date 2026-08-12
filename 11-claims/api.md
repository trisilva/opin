# Module 11: Claims

The endpoints, the flow from first notification to settlement, the lifecycle and the error paths.
The entities and fields are in [`data-model.md`](data-model.md), and the rules that apply on every
call are in [`../conventions.md`](../conventions.md).

## Resource model

```mermaid
classDiagram
    class Claim {
        +UUID id
        +string claimNumber
        +DateTime fnol
        +DateTime lossDate
        +ClaimType claimType
        +string location
        +string lossCause
        +int liabilityShare
        +int reserve
        +ClaimStatus claimStatus
        +Date lastUpdate
        +Date reopenDate
        +int excessAmount
        +PaymentMethod paymentMethod
        +string[] documents
        +create() Claim
        +retrieve(id) Claim
        +update(id) Claim
        +addDocument(id, url) Claim
        +settle(id, amount) Claim
        +reopen(id) Claim
    }
    class ClaimsBordereau {
        +UUID id
        +string treatyReference
        +string policyNumber
        +DateTime fnol
        +float GrosslLossReserve
        +float paid
        +float expectedRecovery
        +ClaimStatus claimStatus
    }
    Claim --> ClaimsBordereau : ceded into
```

**`policyNumber` is what links a claim to its policy.** It is globally unique across the namespace,
it travels in the claim payload, and `/claim` accepts it as a collection filter. That constraint is
what makes one polymorphic endpoint possible across all eight coverage types, and it is set out in
[`../SCOPE.md`](../SCOPE.md).

## Endpoints

`POST /claim` accepts a claim against any coverage type. The caller does not say which kind of
policy it is, because `policyNumber` already determines that.

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /claim` | admin | File a claim against any coverage type |
| `GET /claim` | developer | List, filterable by `policyNumber` and `claimNumber` |
| `GET /claim/{id}` | developer | Retrieve one claim |
| `PUT /claim/{id}` | admin | Replace a claim, which is how an adjuster revises the reserve and the liability share |
| `POST /claim/{id}/documents` | admin | Append to the claim's documents |
| `POST /claim/{id}:settle` | admin | Pay and close. Records the amount and the payment method |
| `POST /claim/{id}:reopen` | admin | Reopen a closed claim and set `reopenDate` |
| `POST /claimsBordereau` | admin | Create a reinsurance report |
| `GET /claimsBordereau` | developer | List, filterable by `treatyReference` |
| `GET /claimsBordereau/{id}` | developer | Retrieve one report |

## Primary flow: first notification through settlement

```mermaid
sequenceDiagram
    participant Insured as Insured Portal
    participant Gateway as API Gateway
    participant Claim as Claim Service
    participant Coverage as Coverage Service
    participant Adjuster as Adjuster
    Insured->>Gateway: POST /claim {policyNumber, fnol, lossDate, claimType, location, lossCause, description}
    Gateway->>Claim: createClaim(payload)
    Claim->>Coverage: lookup policyNumber, validate active at lossDate
    Coverage-->>Claim: coverage details, perils, indemnity limits
    Claim->>Claim: validate lossCause against coverage perils
    Claim->>Claim: set claimStatus=open, set initial reserve
    Claim-->>Gateway: 201 {claimNumber, status: open}
    Insured->>Gateway: POST /claim/{id}/documents {url, type}
    Gateway->>Claim: addDocument
    Claim-->>Gateway: 200
    Adjuster->>Gateway: PUT /claim/{id} {liabilityShare, reserve}
    Gateway->>Claim: updateClaim, set lastUpdate
    Adjuster->>Gateway: POST /claim/{id}:settle {paymentAmount, paymentMethod}
    Gateway->>Claim: settleClaim
    Claim->>Claim: set claimStatus=closed
    Claim-->>Gateway: 200 {status: closed}
```

## Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Open : POST /claim
    Open --> Closed : :settle (payment recorded)
    Closed --> Reopened : :reopen
    Reopened --> Closed : :settle (additional payment recorded)
    Closed --> [*]
```

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

Three states: open, closed, reopened. Settlement closes a claim and it does not end it, because
something further can always emerge. A reopened claim settles again, and a second payment is
recorded against it.

Triage, investigation, awaiting documents, awaiting payment and fraud review are not states here and
will not become states. They describe where a claim sits in one operator's process, which is not
something two insurers need to agree on to exchange a claim.

## Errors

The check that matters most is the first one. A policy has to have been in force on the **loss
date**, not on the date the claim was filed, so a claim can be admitted against a policy that has
since expired and refused against one that is currently active.

```mermaid
flowchart TD
    A[POST /claim] --> B{Policy active at lossDate?}
    B -->|No| E1[409 - policy not in force at lossDate]
    B -->|Yes| C{lossCause in coverage perils?}
    C -->|No| E2[422 - peril not covered]
    C -->|Yes| D{claimType in claimType enum?}
    D -->|No| E3[400 - unknown claim type]
    D -->|Yes| F{fnol within reporting window?}
    F -->|No| W1[Warn - late notification, may affect claim]
    F -->|Yes| G[Set reserve, claimStatus=open]
    W1 --> G
    G --> H[201 Created]
```
