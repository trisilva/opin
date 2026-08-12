# Module 04: Travel

Travel insurance covers what goes wrong on a trip: cancellation before you leave, medical treatment
while you are away, lost or delayed baggage, and getting you home if something serious happens. It
is sold two ways, and the difference shapes the model. A **single-trip** policy covers one journey
between fixed dates. An **annual multi-trip** policy covers every journey in a year, each up to a
maximum length.

Travel differs from the other coverage lines in one structural way, and it is worth knowing before
you read the model. Most lines have one indemnity limit. Travel has a dozen, one per benefit, and
each is a separate field.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

Two entities. **`travelCoverage`** is the policy and **`traveller`** is each person covered by it.
One policy covers many travellers, which is how a family or a corporate group is written.

The benefit limits on `travelCoverage` are the substance of the product. `tripCancellation`,
`tripInterruption`, `travelDelay`, `baggageDamage`, `baggageDelay`, `emergencyMedical`,
`emergencyDental`, `emergencyEvacuation`, `repatriationOfRemains` and `rentalCarCollision` each cap
what the policy pays for that kind of loss. They are independent of one another, so exhausting the
baggage limit does not touch the medical one.

Two flags decide the shape of the policy. `isAnnualPolicy` distinguishes annual multi-trip from
single trip, and `isGroup` says whether more than one traveller is covered. `length` means trip days,
and it means something different depending on `isAnnualPolicy`: on a single-trip policy it is the
length of the trip, and on an annual policy it is the maximum length of any one trip.

## What to watch

**`destination` is typed as an integer with no enumeration behind it.** Nothing in the standard says
what the integer means. It is most likely a country or region code, and it is not declared as one.
Agree the meaning with your counterparty explicitly rather than assuming a shared reading, and do
not assume it is an ISO country code.

**The benefit-limit fields carry no declared type.** All ten are typed here as `Number/Float` for
consistency with the other coverage modules, and the underlying data standard leaves them
undeclared. Treat them as monetary amounts.

**`traveller` has no address and no contact details.** Travel cover normally needs an emergency
contact and somewhere to send documents, and neither has a home in this entity. Carry them as
extension fields.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
