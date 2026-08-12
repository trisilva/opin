# Module 05: Term Life

Term life insurance pays a lump sum if the insured person dies within a fixed term. If they survive
the term, it pays nothing and there is no cash value. That is the whole product, and its simplicity
is why it is the cheapest life cover sold.

Three roles come apart here more visibly than anywhere else in the standard, so hold them clearly.
The **policyholder** buys the policy and pays for it. The **life insured** is the person whose death
triggers the payout. The **beneficiary** receives the money. The life insured cannot collect their
own death benefit, which is exactly why the beneficiary is a separate party.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

**`termLifeCoverage`** is the policy. **`lifeInsured`** is the person covered, carrying occupation
and annual salary because both are underwriting inputs: what someone does for a living affects how
likely they are to die, and what they earn caps how much cover they can justify.

**`termLifeType`** says which variant of term life this is. Level term pays the same sum throughout.
Decreasing term pays less as time passes, which is how a policy is matched to a shrinking mortgage.
Renewable term can be extended without new medical evidence. Convertible term can be swapped for a
whole-of-life policy later.

**Riders** are optional add-ons that extend cover beyond death alone: accidental death benefit,
total permanent disability, total and partial permanent disability, and critical illness. A policy
can carry several.

`freeCoverLimit` is worth knowing about. It is the amount of cover available without medical
underwriting, and it is what makes group schemes practical: below the limit, nobody has to be
examined.

## What to watch

**The two enumerations here are the clearest disagreement in the standard.** `termLifeType` carries
four values in the data standard and three in the API specification. `termLifeRiders` carries four
in one and five in the other, and the extra rider value is `Convertible term`, which is a type of
policy rather than a rider at all.

This module treats the data standard as authoritative: `termLifeType` has four values including
`Convertible term`, and `termLifeRiders` has four without it. Validate against those sets. If you
integrate with something built from the API specification, expect a mismatch on exactly these two
fields.

**Beneficiary shares are not validated for you.** `termLifeCoverage` carries a multi-valued
`beneficiary` reference and each beneficiary carries a `share`. Nothing checks that the shares total
100, and a policy where they do not cannot be settled without someone deciding what happens to the
remainder. Check it on the way in.

**`businessSector` has no declared type.** It is treated here as a UK SIC code, matching
`Commercial.occupation` elsewhere in the standard, and the underlying standard declares neither type
nor reference list.

**There is no underwriting.** No quote, no decision, no decline, no awaiting-medicals state. A
policy record exists once cover is bound, and everything that happens before that sits above the
standard. Death claims settle through [Claims](../11-claims/) like any other claim.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
