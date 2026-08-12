# Module 11: Claims

A claim is a request for payment under a policy. This module carries one claim entity that serves
every coverage type, so a motor claim, a travel claim and a pet claim are the same record with
different values in it.

That is a deliberate design and it is the reason `POST /claim` is a single endpoint rather than
eight. It works because `policyNumber` is globally unique across every coverage type, so a claim
carrying one resolves to exactly one policy without the caller having to say which kind it is.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow from first notification to settlement, the lifecycle and the error paths |

## How the model is shaped

**`Claim`** is the claim. **`ClaimsBordereau`** is the periodic report sent to a reinsurer listing
what has been claimed, which is a report rather than a transaction and is why some figures appear in
both places.

Four ideas in the claim record are worth understanding before you read the fields.

**FNOL is not the loss date.** `fnol` is when the insurer was told. `lossDate` is when it happened.
The gap between them matters legally, because notifying too late can prejudice a claim, and it
matters operationally, because it is the first sign of a claim that will be difficult.

**The reserve is an estimate, and it moves.** `reserve` is money set aside for a claim that is open
and not yet settled. It is revised as the claim is understood. It is how an insurer knows what it
owes before it knows exactly what it owes, and in aggregate it is the largest liability on most
insurers' books.

**Liability can be shared.** `liabilityShare` is the percentage of the loss this insurer is
responsible for, which matters wherever fault is split between parties.

**A closed claim can reopen.** Settlement is not final. Something further emerges, the claim
reopens, and a further payment is recorded. This is why the lifecycle has three states rather than
two.

## What to watch

**`lossCause` does not say which peril list applies.** It references a generic set of perils without
naming `motorPeril`, `propertyPeril` or `tradeCreditPeril`. The resolution is evidently polymorphic
by coverage type, and it is not declared. Resolve it from the coverage the `policyNumber` points at,
and be aware another implementation may have resolved it differently.

**There is no cumulative paid figure on the claim.** `Claim` carries `reserve` and no `paid`.
`ClaimsBordereau` carries `paid` for reinsurance reporting, so the direct-insurance record cannot be
reconciled from reserve to what actually went out the door. Carry a paid total yourself.

**`GrosslLossReserve` on the bordereau is misspelled** with an extra `l` between `Gross` and `Loss`.
It is on the wire, so send it as written.

**Operational sub-states are out of scope by design.** Triage, investigation, awaiting documents,
awaiting payment and fraud review do not appear and will not. A claim's business status belongs to
the standard; its position in one operator's workflow does not. See [`../SCOPE.md`](../SCOPE.md).

## Market profiles

Vietnam's statutory claim handling under Decree 67/2023/ND-CP lands on this module. It is not
written, and it is gated on Vietnamese counsel. See [`../project/markets/vn/`](../project/markets/vn/).
