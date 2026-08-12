# Module 09: Trade Credit

Most businesses sell on credit: goods now, payment in 30, 60 or 90 days. Trade credit insurance
covers the seller when the buyer does not pay, because they went bankrupt, became insolvent, simply
never paid, or were prevented from paying by their government.

This module has a shape the other seven coverage lines do not, and it is worth grasping before you
read the model. The insured is the **seller**. The risk being underwritten is the **buyer**, who is
not a party to the policy and may not know it exists. So the record describes a company that never
signed anything, which is why `Debtor` carries financial statements, a credit rating and a parent
company. The insurer is underwriting a stranger's balance sheet.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

**`tradeCreditCoverage`** is the policy and **`debtor`** is the buyer whose failure to pay is the
insured event.

**`tradeCreditType`** says how much of the seller's book is covered. Whole turnover covers every
customer. Key accounts covers the largest few. Single buyer covers one. Transactional covers one
shipment. The choice drives everything about how the policy is administered.

**`tradeCreditPeril`** is the four causes of non-payment that count: bankruptcy, insolvency,
protracted default (the buyer has simply not paid for long enough that the debt is treated as bad),
and political risk.

Three numbers do the work. `creditLimit` is the most the insurer will cover against that buyer.
`creditLimitUtilized` is how much of it is currently in use, so the headroom is the difference.
`maxCreditPeriod` is the longest payment term the cover allows, in days, and invoicing a buyer on
terms longer than that puts the debt outside the policy.

`waitingPeriod` is how long an unpaid debt must stay unpaid before it can be claimed. Protracted
default is a matter of time passing rather than of an event happening, so the policy needs a
threshold.

## What to watch

**This module now carries the standard policy lifecycle.** `inceptionDate`, `expiryDate`, `status`
and the premium, brokerage and endorsement fields are all present, with the same names, types and
value sets they carry on every other coverage type. An implementation that already handles motor or
property handles these unchanged.

**Two names are misspelled on the wire and stay that way.** The type enumeration is `tradeCreditTpe`
and the utilisation field is `creditLimitUtiilized` with a doubled `i`. Both are in every existing
implementation, so send them as written. A third, `entitytype` with a lowercase `t`, is shown
corrected here.

**Political risk overlaps with a separate product.** `tradeCreditPeril` code 3 is political risk,
and the product catalogue also carries political risk insurance as a line of its own. The boundary
between the two is not drawn. Decide which one you are selling before you rely on the peril.

**There is no waiting-period state.** The field exists and nothing signals entering or leaving it.
Treat it as a calculation against the loss date when a claim is filed rather than as something the
policy record tracks.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
