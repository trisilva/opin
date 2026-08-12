# Module 08: Business Interruption

Property insurance pays to rebuild the factory. Business interruption pays for the money the
business did not make while the factory was being rebuilt. That is the whole idea, and it is why the
two are almost always sold together against the same premises and the same perils.

The distinction matters because the two policies are settled completely differently. A property
claim is settled against the cost of physical things. A business interruption claim is settled
against accounts: what the business would have earned, minus what it did earn, over a defined
period.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

One entity, **`businessInterruptionCoverage`**, and it looks like a property policy with a financial
exposure base bolted on. It reuses `propertyType` and `propertyPeril` from
[property](../06-property/), because the events that stop a business are the events that damage its
premises.

Three concepts carry this module, and none of them appear in any other.

**The indemnity period.** `IndemnityPeriod` is the number of days the policy will keep paying after
the interruption starts. It is not how long the policy lasts. A twelve-month policy can carry a
24-month indemnity period, because rebuilding takes longer than the policy year. This is the field
most often set wrongly, and setting it too short is how a business is underinsured without anyone
noticing until the claim.

**The exposure base.** `grossAnnualProfit`, `netAnnualProfit`, `grossAnnualTurnover`, `totalPayroll`
and `fixedExpenses` are the accounts the loss is calculated from. The policy pays the shortfall
against these, which is why business interruption is underwritten from financial statements rather
than from a survey.

**Increased cost of working.** `icwLimit` covers money spent specifically to keep trading through
the interruption: renting temporary premises, paying for express delivery, running an extra shift.
It is separate from the lost-profit cover because spending it reduces the eventual claim. The
`typeBusinessInterruption` enumeration distinguishes three forms: standard business interruption,
increased cost of working alone, and contingent loss of profit, which covers an interruption at a
supplier or a customer rather than at the insured's own premises.

`denialOfAccessLimit` and `closureByPublicAuthority` cover the case where nothing is damaged at all
and the business still cannot trade, because access is blocked or an authority has ordered it shut.

## What to watch

**`propertyRef` links this policy to the premises it protects.** The property record itself is
created through [property](../06-property/), and this module only references it. Bind the property
and its own coverage first.

**`deductible` is in days here, not money.** There is a separate `deductibleAmount` for the monetary
excess. A time-based excess means cover starts only once the interruption has run past a waiting
period, which is standard for this line.

**One field name contains spaces at source**, `closure by public authority`. It cannot travel as
written, and this module uses `closureByPublicAuthority`.

**`IndemnityPeriod` is PascalCase** where every other field on the entity is camelCase. It is kept as
written rather than corrected.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
