# Insurance concepts

The vocabulary the twelve modules assume. If you are a good engineer and new to insurance, read
this once and the modules stop being opaque.

Terms are grouped by what they describe rather than alphabetically, because most of them only make
sense next to the two or three terms they sit with.

## Who the parties are

Insurance involves more roles than it first appears, and the standard keeps them apart because a
single person can hold several of them at once.

**Insurer**, also called the carrier. The company that takes the risk and pays the claim. In
[Core parties](01-core-parties/) this is the `insuranceEntity`.

**Broker.** An intermediary who arranges cover between a customer and an insurer. A broker does not
carry risk. They are paid a **brokerage**, which is a percentage of the premium, and it appears on
the coverage record as `brokeragePercentage` and `brokerageAmount`.

**Policyholder.** Whoever owns the policy and pays for it. The standard splits this into `Personal`
and `Commercial` because a person and a company are described by different fields.

**Insured.** Whoever or whatever the policy actually protects. Often the same as the policyholder
and often not. A company insuring its fleet is the policyholder; the driver is the insured. In
[Term life](05-term-life/) the person whose death triggers the payout is the `lifeInsured`, and they
may not be the one paying.

**Beneficiary.** Whoever receives the payout. Named on the policy, and canonically on term life,
where the insured cannot collect their own death benefit.

**Third party.** Someone who suffers a loss you caused and who has no contract with your insurer.
Third-party motor cover pays them rather than you, which is why it is the cover most countries make
compulsory.

## The policy and what it covers

**Coverage.** The unit this standard is built around. A coverage record is one line of protection
on one thing, and each of the eight coverage modules defines its own. There is deliberately no
separate `Policy` entity above it. The coverage record carries the policy fields directly, and
`policyNumber` is what ties everything together.

**Peril.** A specific cause of loss that the policy will pay for. Fire is a peril. Theft is a peril.
A claim is admitted when its stated cause is a peril the coverage names, and refused when it is not.

**Exclusion.** A cause of loss the policy will not pay for, stated explicitly so the boundary is not
left to argument.

**Sum insured.** The value the thing is insured for. It is what the insurer would pay if the thing
were a total loss, and it is the number the premium is calculated from.

**Indemnity limit.** The most the insurer will pay. `indemnityLimitPolicy` caps what the policy pays
in total across its whole term. `indemnityLimitAccident` caps what it pays for any single incident.
A policy can exhaust the first while never approaching the second, and the two are not
interchangeable.

**Inception and expiry.** The dates cover starts and ends. A claim is only admitted if the loss
happened while the policy was in force between them, which is why the loss date matters more than
the date the claim was filed.

**In force, lapsed, cancelled, extended.** The four states a policy can be in. Lapsed means it
stopped because the premium was not paid. Cancelled means someone ended it deliberately. The legal
transitions between these are drawn in each module's API page, and a transition that is not drawn is
not one an implementation may make.

**Endorsement.** A change to a policy that is already in force: adding a driver, changing an
address, raising a limit. It amends the existing record rather than creating a new one, and the
policy stays in force throughout.

**Renewal.** Cover continuing into a new term. This issues a **new** coverage record rather than
extending the old one, because the terms and the premium are renegotiated.

**Waiting period.** A stretch at the start of a policy during which cover is not yet active. Common
in [pet](10-pet/) and health cover, where it exists to stop someone buying a policy for a problem
their animal already has.

**Pre-existing condition.** A problem that was already there when the policy was bought. Usually
excluded, for the same reason.

## The money

**Premium.** What the customer pays for the cover.

**Gross written premium**, written `grossWrittenPremium`. The full premium for the policy term
before any deduction. "Written" means contracted rather than collected, so a policy contributes its
gross written premium the moment it is issued, whether or not the money has arrived.

**Premium rate.** The rate the premium is calculated from, usually applied against the sum insured.

**Premium payment frequency.** How often the customer pays: annually, monthly, in one instalment.

**Deductible.** The portion of every claim the customer pays before the insurer pays anything. A
policy with a 5,000,000 VND deductible on a 20,000,000 VND loss pays out 15,000,000. It exists to
remove small claims, which cost more to process than they are worth.

A **compulsory** deductible is set by the insurer and cannot be declined. A **voluntary** deductible
is one the customer chooses to take on in exchange for a lower premium. Both appear on the coverage
record, as an amount or as a percentage of the claim.

**Excess.** The same idea as a deductible, under the name used in British-influenced markets. The
standard carries both: the deductible fields sit on the coverage record, and `excessAmount` sits on
the [claim](11-claims/) as the amount actually withheld from that settlement.

**Loading.** An increase to the premium for an identified extra risk. A young or newly licensed
driver attracts a loading.

**No-claims discount**, `noClaimsDiscount`. A reduction earned by not claiming, usually stated as a
percentage and usually growing year on year. It is the main reason a customer will pay a small loss
themselves rather than claim for it.

**Discount amount** and **sales tax.** What was taken off the premium, and what was added to it in
tax. Both sit on the coverage record so a premium figure can be reconciled.

**Reimbursement.** In [pet](10-pet/) cover, the percentage of an admitted claim the insurer pays
back after the deductible. An 80% reimbursement pays 80% of what remains.

**Receipt.** The record of money actually moving, in either direction. Premium coming in and a
settlement going out are both receipts. See [Premium and receipts](12-premium-receipts/).

## When something goes wrong

**Claim.** A request for payment under a policy. One `Claim` entity serves all eight coverage types,
and it reaches its coverage through `policyNumber`.

**FNOL**, first notification of loss. The moment the insurer is first told something happened, which
is usually before anyone knows what it will cost. It is a timestamp on the claim, and it is separate
from the loss date because the gap between them matters: notify too late and the claim can be
prejudiced.

**Loss date and loss cause.** When it happened and what caused it. The loss cause is checked against
the perils the coverage names, and that check is what admits or refuses the claim.

**Reserve.** Money the insurer sets aside for a claim that is open but not yet settled. It is an
estimate, revised as the claim is understood, and it is how an insurer knows what it owes before it
knows exactly what it owes. Reserves are the largest single number on most insurers' balance sheets.

**Liability share.** The percentage of the loss this insurer is responsible for, where fault is
split. A driver found 70% at fault produces a claim with a liability share to match.

**Settlement.** Paying the claim and closing it. A closed claim can be **reopened** if something
further emerges, which is why the claim lifecycle has a reopened state rather than treating closed
as final.

**Subrogation.** The insurer paying you, then pursuing whoever actually caused the loss to get it
back. It happens after settlement and it is why an insurer will pay a claim quickly even when
another party is clearly at fault.

## Reinsurance

Insurers insure themselves. These three terms are why [module 12](12-premium-receipts/) exists.

**Cede.** To pass a share of a risk, and the premium that goes with it, to a reinsurer. An insurer
cedes so that one catastrophic year cannot end the company.

**Treaty.** The standing agreement that governs what gets ceded, identified by `treatyReference`.

**Bordereau**, plural bordereaux. The periodic report an insurer sends its reinsurer listing what
was ceded. A **premium bordereau** reports premium; a **claims bordereau** reports claims and
reserves. They are reports rather than transactions, which is why they carry a snapshot of figures
that also exist elsewhere.

## Abbreviations used in the modules

| | |
| :--- | :--- |
| **ADAS** | Advanced driver assistance systems. The sensor and automation package on a vehicle: lane keeping, automatic braking, adaptive cruise. It affects motor pricing, which is why the vehicle record captures it |
| **OEM** | Original equipment manufacturer. The company that built the vehicle, as against whoever supplied a replacement part |
| **VIN** | Vehicle identification number. The globally unique 17-character identifier stamped on a vehicle |
| **ISCO-08** | The International Standard Classification of Occupations, 2008 revision. A published code list for occupations, used so that "driver" means the same thing in two systems |
| **KYC** | Know your customer. The identity checks a regulated firm runs before it takes someone on |
| **PII** | Personally identifiable information. Anything that identifies a specific person, and the class of data that residency rules apply to |

## Where the standard stops

Two things a reader often expects here and will not find.

The standard carries no **pricing** or **rating**. How an insurer turns a risk into a premium is
where insurers compete, and a standard that settled it would be a standard nobody could adopt. The
standard carries the premium figure, not the calculation behind it.

The standard carries no **operational workflow**. A claim's business status is open, closed or
reopened. Whether it is in triage, awaiting documents or with a fraud reviewer belongs to whichever
platform is running the claim. See [`SCOPE.md`](SCOPE.md).
