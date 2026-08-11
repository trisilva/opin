# Module 11: Claims

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

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

`[OPIN concern]`: OPIN's Claim schema does not carry an explicit foreign-key field linking to the policy or coverage being claimed against. Implementations must associate claims to coverage out-of-band (typically via `policyNumber` carried in the claim payload). An OPIN v1.1 should add an explicit linkage field to Claim.

### Endpoints

OPIN endpoints (kept verbatim, attributed to OPIN, polymorphic across coverages):

- `POST /claim` (admin) `[OPIN]`
- `GET /claim` (developer) `[OPIN]`

Added here, on the same OPIN schemas:

- `GET /claim/{id}` (developer) `[added]`
- `PUT /claim/{id}` (admin) `[added]`
- `POST /claim/{id}/documents` (admin) `[added]`: append to OPIN `documents` array
- `POST /claim/{id}:settle` (admin) `[added]`: transitions claimStatus from open to closed and records payment amount/method
- `POST /claim/{id}:reopen` (admin) `[added]`: transitions claimStatus from closed to reopened, sets reopenDate
- `POST /claimsBordereau` (admin) `[added]`
- `GET /claimsBordereau` (developer) `[added]`: filter by treatyReference
- `GET /claimsBordereau/{id}` (developer) `[added]`

### Primary flow: Submit a claim (FNOL through settlement)

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

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Open : POST /claim
    Open --> Closed : :settle (payment recorded)
    Closed --> Reopened : :reopen
    Reopened --> Closed : :settle (additional payment recorded)
    Closed --> [*]
```

States are exactly OPIN `claimStatus` (open, closed, reopened). All operational sub-states (triage, investigation, awaiting documents, awaiting payment, fraud review) are implementer-side workflow concerns and do not appear on the wire.

### Routing and error handling

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
