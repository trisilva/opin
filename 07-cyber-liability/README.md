# Module 07: Cyber Liability

Cyber liability insurance covers what a business loses when its systems or its data are attacked:
the cost of investigating a breach, notifying the people whose data was taken, restoring systems,
the revenue lost while they were down, any ransom, and the regulatory fines and legal claims that
follow.

It is unlike the other coverage lines in one way that shapes the whole model. There is no physical
thing to insure. So the record describes the business itself, and specifically the parts of it that
predict how bad a breach would be: what data it holds, how much of it, how many people work there,
how many of them work in IT, and how much of the turnover is online.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

**`cyberLiabilityCoverage`** is the policy and **`business`** is the insured organisation.

**`dataAssets`** is what makes cyber pricing work. It enumerates the five kinds of sensitive data a
business might hold, and the kind matters more than the volume because each carries a different
regulatory consequence:

- **IP**, intellectual property
- **PII**, personally identifiable information
- **PCI**, payment card data, governed by the card schemes' own security standard
- **PHI**, protected health information, which is the most heavily regulated of the five
- **Commercial**, confidential business data

**`cyberCoverageCategories`** is the 21-value list of what the policy actually pays for: data loss,
breach and privacy response, incident management, kidnap and ransom, business interruption,
contingent business interruption, multimedia liability, legal defence, reputation, network failure,
errors and omissions, professional indemnity, fidelity, theft of intellectual property, asset
damage, compensation, terrorism, fines, directors and officers, general liability, and
environmental. A policy names the subset it covers.

Note that `business.deductible` is measured in days here, not money, alongside a separate
`deductibleAmount`. A time-based excess is normal in cyber: the policy starts paying only once an
outage has lasted past a waiting period.

## What to watch

**`claimsOccurrence` decides which policy year pays, and it matters more here than anywhere else.**
A breach is routinely discovered long after the intrusion that caused it, so the gap between the
loss and the notification is often years rather than days. Cyber is usually written claims-made. The
field is the same two-value enumeration used in property and business interruption.

**`business.cyberCoverageCategories` points at the wrong list.** It references the
`cyberLiabilityCoverage` sheet rather than the `cyberCoverageCategories` enumeration it names.
Validate against the enumeration.

**Two field names are misspelled at source.** `claimsOcuurence` on the coverage entity, and
`NumberOfEmployees` in PascalCase where the rest of the standard is camelCase. This module shows the
corrected forms.

**There is no incident model.** No breach detection, no notification timeline, no remediation
tracking. A cyber loss is recorded through the shared `Claim` entity like any other. Breach
notification deadlines in particular are out of scope by design rather than pending: they are
operational service-level measurement, and they sit above the standard in whatever platform you
build. See [`../SCOPE.md`](../SCOPE.md).

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
