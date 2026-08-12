# Module 06: Property

Property insurance covers physical things in a fixed place: the building itself, what is inside it,
and the fixtures between the two. It covers them against named perils, mostly fire, flood, storm,
theft and impact.

The thing to understand before reading the model is that a property policy is not one sum of money.
Building, contents and fixtures are insured separately, each with its own sum insured and often its
own deductible. A fire that destroys a kitchen touches all three, and the claim is settled against
each in turn.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that binds a policy, the lifecycle and the error paths |

## How the model is shaped

**`propertyCoverage`** is the policy and **`property`** is the building. One policy can insure
several properties, which is how a portfolio is written.

Four enumerations do the descriptive work, and they are large. `propertyType` carries roughly 250
values from the RESO real-estate standard. `propertyPeril` carries 37. `wallConstruction` carries
about 40 and `roofConstruction` about 30. Construction materials are enumerated in that much detail
because what a building is made of is most of what decides whether it burns down.

Two ideas in this module are worth understanding before you meet the fields.

**Agreed value against market value.** `isAgreedValue` decides how a total loss is paid. Under an
agreed value policy the insurer pays the sum insured, full stop. Under a market value policy it pays
what the property was worth at the moment it was destroyed, which can be less than the sum insured
and is argued about far more often.

**Claims occurring against claims made.** This decides which policy year a claim belongs to. A
claims-occurring policy covers losses that happened during its term, whenever they are reported. A
claims-made policy covers claims reported during its term, whenever the loss happened. The
difference decides who pays when a loss surfaces years later, and it is the single most consequential
field on the record.

## What to watch

**`claimsOccurrence` is declared two ways.** It is a Boolean on the coverage entity and a two-value
enumeration (`Claims occurring`, `Claims made`) on its own. Use the enumeration. A Boolean cannot
say which of the two bases applies without an out-of-band convention, and getting this wrong
misassigns claims between policy years.

**`numberOfBedrooms` is declared twice on the property entity.** The first of the two describes
itself as the number of bathrooms, so it is read here as `numberOfBathrooms`. Expect data from other
implementations to have resolved this differently.

**The roof construction enumeration is spelled two ways.** `roofConstruction` in the data standard
and `rootConstruction` in the API specification. The data standard is authoritative and this module
uses `roofConstruction`.

**`alternativeAccomodation` is misspelled and stays that way.** One `m`. It is on the wire, so send
it as written.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
