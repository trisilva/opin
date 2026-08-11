# Module 12: Premium and Receipts

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class Receipt {
        +UUID id
        +ReceiptType receiptType
        +Date receiptDate
        +int paymentAmount
        +ReceiptCalculation receiptCalculation
        +PaymentMethod premiumPaymentMethod
        +create() Receipt
        +retrieve(id) Receipt
        +refund(id) Receipt
    }
    class PremiumBordereau {
        +UUID id
        +string treatyReference
        +string policyholder
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +int indemnityLimitPolicy
        +int grossWrittenPremium
        +int netPremium
        +ReceiptType transactionType
    }
    Receipt --> PremiumBordereau : aggregated in
```

`[OPIN concern]`: OPIN's Receipt schema has no foreign-key field linking the receipt to the policy or claim it settles. Without a linkage, reconciliation is impossible. This standard exposes `policyNumber` and `claimNumber` as collection filter parameters on `/receipt` so receipts can be located by their settled obligation, but the underlying schema gap remains and an OPIN v1.1 should resolve it.

### Endpoints

`[added]`: OPIN publishes the Receipt, PremiumBordereau, and ClaimsBordereau (Module 11) schemas but no endpoints.

- `POST /receipt` (admin)
- `GET /receipt` (developer), filter by `policyNumber`, `claimNumber`, `receiptType`, `receiptDate` range
- `GET /receipt/{id}` (developer)
- `PUT /receipt/{id}` (admin)
- `POST /receipt/{id}:refund` (admin), issues a reversing receipt rather than mutating the original
- `POST /premiumBordereau` (admin)
- `GET /premiumBordereau` (developer), filter by `treatyReference`
- `GET /premiumBordereau/{id}` (developer)
- `PUT /premiumBordereau/{id}` (admin)

### Primary flow: Record a premium payment and aggregate to bordereau

```mermaid
sequenceDiagram
    participant Payment as Payment Provider
    participant Gateway as API Gateway
    participant Receipt as Receipt Service
    participant Coverage as Coverage Service
    participant Bordereau as Bordereau Service
    Payment->>Gateway: POST /receipt {receiptType: new policy, policyNumber, paymentAmount, premiumPaymentMethod}
    Gateway->>Receipt: createReceipt
    Receipt->>Coverage: validate policyNumber resolves to a coverage record
    Receipt->>Receipt: persist receipt
    Receipt->>Bordereau: aggregate to current premiumBordereau period
    Receipt-->>Gateway: 201 {receiptId}
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Recorded : POST /receipt
    Recorded --> Reversed : :refund issues reversing receipt
    Recorded --> [*] : retained
    Reversed --> [*]
```

`[OPIN concern]`: OPIN does not declare a Receipt lifecycle. This standard keeps it conservative: a receipt is a recorded financial event, immutable once recorded, and a refund is itself a new receipt with `receiptType` set to a reversing value. Reconciliation status, dispute status, and ledger postings are out of scope and live in implementer-specific extensions.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /receipt] --> B{policyNumber OR claimNumber present?}
    B -->|No| E1[400 - reference required]
    B -->|Yes| C{receiptType in receiptType enum?}
    C -->|No| E2[400 - unknown receipt type]
    C -->|Yes| D{paymentAmount &gt; 0?}
    D -->|No| E3[400 - invalid payment amount]
    D -->|Yes| F{Idempotency-Key seen recently?}
    F -->|Yes| R1[200 OK - idempotent replay]
    F -->|No| G[Persist Recorded]
    G --> H[201 Created]
```
