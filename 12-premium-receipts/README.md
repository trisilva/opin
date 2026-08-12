# Module 12: Premium and Receipts

This module is where money is recorded moving, in both directions. A customer paying a premium and
an insurer paying a settlement are both receipts, described by the same entity with a different
`receiptType`.

It also carries the reinsurance side. An insurer buys cover for itself, ceding a share of its risks
and the premium that goes with them, and it reports what it ceded to its reinsurer periodically. That
report is a bordereau. Premium is reported here and claims are reported from
[Claims](../11-claims/).

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that records a payment, the lifecycle and the error paths |

## How the model is shaped

**`Receipt`** is one financial event. `receiptType` says which kind: a new policy, a renewal, a
mid-term adjustment, a claim payment, brokerage, a profit share, or other. One entity covers money
in and money out, which is why a settlement and a premium look the same in the ledger.

**`PremiumBordereau`** is the periodic premium report to a reinsurer.

Two ideas worth holding before you read the fields.

**A receipt is immutable.** Once recorded it does not change. A refund is not an edit; it is a
second receipt that reverses the first. This is normal ledger discipline and it is what makes the
record auditable.

**Pro rata against flat.** `receiptCalculation` says how an amount was worked out. Pro rata means
proportioned to the time actually covered, which is what a mid-term cancellation produces. Flat
means the whole amount regardless. The distinction decides what a customer is owed when a policy
ends early.

`netPremium` on the bordereau is premium after brokerage has come out, where `grossWrittenPremium`
is before. The reinsurer sees both because it is paying commission on one and taking risk against
the other.

## What to watch

**A receipt carries no field pointing at what it settles.** There is no policy reference and no
claim reference on the entity. Reconciliation runs through the collection filters instead:
`/receipt` accepts `policyNumber` and `claimNumber`, and `policyNumber` is globally unique across
every coverage type, which is what makes that lookup deterministic. A receipt is found by the
obligation it settles rather than by a foreign key it carries.

**There is no direct-insurance premium ledger.** Both bordereaux report to reinsurers. Nothing in
the standard reconciles premium accrued, premium collected and premium remitted at the direct
level, so if you need that ledger you are building it yourself.

**Commission ledgers are out of scope by design.** Payout schedules to distribution partners and
channel revenue splits sit above the standard, in whatever platform you build. See
[`../SCOPE.md`](../SCOPE.md).

## Market profiles

Vietnamese dong is unit-denominated and carries no minor units, unlike the two-decimal assumption
most money handling makes. That has a wire consequence on every amount in this module. It is not
written yet. See [`../project/markets/vn/`](../project/markets/vn/).
