# Module 03: Motor

Motor insurance covers a vehicle, the people who drive it, and the losses that come from using it on
a road. It is the largest line of general insurance in most markets and the one most often made
compulsory, because a car can injure someone who never agreed to take that risk.

Read this module first even if you are not building motor. Every other module's API surface mirrors
the shape declared here, so the endpoints, the lifecycle and the error paths will already be
familiar when you get to travel or pet.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

Three entities carry motor, and they are separate records rather than one nested document.

**`motorCoverage`** is the policy. It holds the term, the status, the premium, the perils covered
and the deductibles. It is the record `policyNumber` belongs to, and therefore the record a claim
resolves back to.

**`Vehicle`** is the thing insured. **`Driver`** is the person whose history prices the risk, which
is why the driver record carries a licence, convictions and a no-claims discount rather than just a
name.

The split matters when you build. A vehicle and a driver are created before the coverage that binds
them, and the coverage refers to both by identifier. One vehicle can appear on successive policies
and one driver can appear on several vehicles at once.

Two entities sit underneath `Driver` and are worth knowing about before you read the model.
`drivingLicence` carries the licence itself, and `conviction` carries motoring offences, because an
insurer prices a licence held for twenty years differently from one held for six months.

If any of premium, peril, deductible, indemnity limit, endorsement or no-claims discount is
unfamiliar, read [Insurance concepts](../concepts.md) first.

## What to watch

**`Vehicle` is very large.** It carries more than a hundred fields spanning registration,
manufacturer specification, telematics, driver assistance, physical condition and consent flags.
That is a lot for one record and it has practical consequences: expect to make most of it optional,
expect your validation to be the slow part of a write, and do not assume a caller will populate more
than a fraction of it. Treat the field groups as separable rather than as one payload that arrives
whole.

**Perils are validated against an enumerated set.** A coverage names its perils from `motorPeril`,
and a claim is admitted by matching its stated cause against that set. A peril code outside the enum
is rejected at write time, not discovered later.

**Renewal issues a new record.** `:renew` does not move the existing coverage into a new term. It
creates a second `motorCoverage` with its own policy number. Reporting that assumes one record per
customer per vehicle will double-count across a renewal boundary.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
