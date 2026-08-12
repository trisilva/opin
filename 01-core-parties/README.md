# Module 01: Core Parties and Entities

Insurance involves more distinct roles than most systems expect, and one person can hold several of
them at once. The customer who buys a policy may not be the person it protects, and neither may be
the one who gets paid. This module defines every party type once, and the other eleven refer back to
it rather than redefining a customer each time.

If the difference between a policyholder, an insured and a beneficiary is not yet second nature,
read [Insurance concepts](../concepts.md) first. This is the module where that distinction becomes
concrete.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that onboards a party, the lifecycle and the error paths |

## How the model is shaped

Five entities, and the first thing to understand is that a person and a company are modelled
separately rather than as one party record with a type flag.

**`Personal`** is a human being: name, date of birth, nationality, identity document, occupation.
**`Commercial`** is an organisation: registered name, date founded, registration number, VAT number.
They are kept apart because almost none of their fields overlap, and one record carrying both sets
would be mostly empty whichever kind it held.

**`insuranceEntity`** is the other side of the contract: the insurer, reinsurer, broker, agent or
managing general agent. One entity type covers all of them, told apart by an `entityType` value,
because a broker and a carrier are described by the same facts even though they do very different
things.

**`Beneficiary`** is whoever receives a payout. It carries a `share`, so a payout can be split
between several, and it is used canonically by [term life](../05-term-life/).

**`address`** works differently from the other four. It is embedded inside whichever entity owns it
and never stored or fetched on its own, so there is no `/address` endpoint and there will not be
one.

## What to watch

**`Personal.gender` carries three values only: `m`, `f` and `o`.** Several jurisdictions require
more values, a different representation, or that the field be optional. Check your market before you
make it mandatory.

**`Commercial` does not record legal form.** No field says whether an organisation is a limited
company, a partnership or a sole trader, even though the standard defines a `legalEntity` enum and
[trade credit](../09-trade-credit/) uses it on its `Debtor` entity. If you need legal form on a
commercial policyholder, carry it as an extension field under the rules in
[`../conventions.md`](../conventions.md).

**Party records are never deleted.** A party exists or it does not, and the lifecycle offers no
closure or suspension state. Status of that kind, including anything about identity checks, is
implementer-side.

## Market profiles

Vietnam's identity document types and its personal data obligations both land on this module.
Neither is written yet. See [`../project/markets/vn/`](../project/markets/vn/).
