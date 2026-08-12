# Module 10: Pet

Pet insurance pays veterinary bills. It is structurally closer to health insurance than to any other
line in this standard, and it works differently from the rest in one way that shapes everything: it
reimburses a percentage of a bill the customer has already paid, rather than paying a fixed sum on a
defined event.

That is why three fields carry this module. The customer pays the vet. The deductible comes off. The
policy pays back a percentage of what remains, up to an annual cap.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

**`petCoverage`** is the policy and **`pet`** is the animal.

**`petBenefits`** is a 38-value list of what treatment is covered, spanning veterinary, surgical,
diagnostic, preventive and behavioural care. A policy names the subset it pays for, and that subset
is most of what distinguishes a cheap policy from an expensive one.

Two mechanisms exist to stop a customer buying cover for a problem the animal already has, and both
matter more here than in any other line because a pet owner can buy a policy on the drive to the
vet.

**`waitingPeriod`** is a number of days at the start of the policy during which nothing is covered.
**`preexistingConditions`** flags whether conditions the animal already had are covered at all,
which they usually are not.

`reimbursement` sits on the pet rather than on the coverage, and it is the percentage of an admitted
claim the policy pays back. Eighty percent reimbursement on a 10,000,000 VND bill with a
1,000,000 VND deductible pays 7,200,000.

## What to watch

**`petBreed` enumerates dog breeds only.** `petKind` covers five kinds of animal: cat, dog, bird,
exotic and rabbit. The breed list has roughly 320 entries and every one of them is a dog. Four of
the five kinds have no breed values at all, so decide up front how you will record a cat's breed.
The options are to leave breed empty for non-dogs, carry an extension field, or hold it as free
text, and none of them will match what another implementation chose.

**There is no per-incident limit.** Every other coverage type carries `indemnityLimitAccident`, and
this one does not. `annualReimbursementLimit` may be doing that job, and the relationship between
the annual cap and any single claim is not stated. Agree it with your counterparty.

**`waitingPeriod` has no state behind it.** The field exists and nothing signals entering or leaving
it. The policy is in force from binding, and the waiting period is a calculation against
`inceptionDate` when a claim is evaluated.

**`deductible` is typed `Number/fFloat` at source**, with a stray lower-case `f`. Read it as
`Number/Float`. The same typo appears in business interruption, trade credit and on the pet entity.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
