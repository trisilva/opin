# Module 12: Premium and Receipts

Entities, fields, enumerated values and relationships. The API surface for this module is in [`api.md`](api.md).

Covers payment lifecycle: Receipt for transactions of any type (new policy, renewal, MTA, claim payment, brokerage, profit share); PremiumBordereau for reinsurance premium reporting; receiptType, receiptCalculation enums.

OPIN sources: `Receipt`, `PremiumBordereau`, `receiptType`, `receiptCalculation`, `paymentMethod`.

```mermaid
erDiagram
    RECEIPT {
        enum receiptType
        date receiptDate
        int paymentAmount
        enum receiptCalculation "pro rata or flat"
        enum premiumPaymentMethod
    }
    PREMIUM_BORDEREAU {
        string treatyReference
        string policyholder
        string policyNumber "cedant"
        datetime inceptionDate
        datetime expiryDate
        int indemnityLimitPolicy
        int grossWrittenPremium
        int netPremium "net of brokerage"
        enum transactionType "receiptType ref"
    }
    RECEIPT_TYPE {
        int code "0-6"
        string description
    }
    RECEIPT_CALCULATION {
        int code "0-1"
        string description "pro rata or flat"
    }

    RECEIPT ||--|| RECEIPT_TYPE : "of type"
    RECEIPT ||--|| RECEIPT_CALCULATION : "calculated as"
    PREMIUM_BORDEREAU ||--|| RECEIPT_TYPE : "transaction"
```

### Field annotations (Module 12)

| Entity | Field | Type | OPIN source | Notes |
|---|---|---|---|---|
| Receipt | receiptType | enum (receiptType) | sheet `receiptType` | new / renewal / MTA / claim payment / brokerage / profit share / other |
| Receipt | receiptDate | Date | sheet `Receipt` |  |
| Receipt | paymentAmount | Number/integer | sheet `Receipt` |  |
| Receipt | receiptCalculation | enum (receiptCalculation) | sheet `receiptCalculation` | Pro rata or flat |
| Receipt | premiumPaymentMethod | enum (paymentMethod) | sheet `paymentMethod` |  |
| PremiumBordereau | treatyReference | Text | sheet `PremiumBordereau` | Reinsurance treaty ref |
| PremiumBordereau | grossWrittenPremium | Number/integer | sheet `PremiumBordereau` | Pre-tax |
| PremiumBordereau | netPremium | Number/integer | sheet `PremiumBordereau` | After brokerage |
| PremiumBordereau | transactionType | enum (receiptType) | sheet `PremiumBordereau` | New / renewal / mid-term adjustment |

`[added]`: the inherited `Receipt` entity declares no linkage to a policy or a claim, so cash could not be reconciled to what produced it using inherited fields alone. This version reconciles through the same rule the rest of the model uses: `policyNumber` is globally unique across the namespace and `claimNumber` is unique within it, and `/receipt` exposes both as collection filters. A receipt is located by the obligation it settles. See [`../SCOPE.md`](../SCOPE.md).

`[OPIN concern]`: OPIN publishes `PremiumBordereau` and `ClaimsBordereau` for reinsurance reporting but does not publish a corresponding direct-insurance premium ledger entity. Reconciling premium accruals, collections, and remittances at the direct-insurance level is not supported by OPIN. Filed as a change proposal.

Out-of-scope for OPIN: commission ledgers, payout schedules to distribution partners, and microinsurance distribution-channel revenue splits. These are operational concerns not in OPIN's scope.
