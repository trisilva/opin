# OPIN

**An open data and API standard for insurance, and the market profiles that make it operable in one
country.**

OPIN gives insurers, brokers, distributors and the systems between them a shared way to describe
parties, products, coverage, claims, premium and receipts. It was created by the Open Insurance
Initiative and published between 2018 and 2022. This repository continues it.

There are two layers here and they answer different questions.

| Layer | Question it answers | Where |
| :--- | :--- | :--- |
| **The standard** | What is an insurance policy, a claim, a receipt, and how does a caller ask for one? | [`standard/`](standard/) |
| **Market profiles** | What does one national market require that a global standard cannot decide for it? | [`markets/`](markets/) |

Everything a market shares lives in the standard. Only what a market does not share lives in a
profile. That line is the whole design, and it is stricter than it sounds: authentication, error
handling, pagination and record lifecycle are the same problem in Hanoi and in Manila, so they
belong to the standard even though a single market surfaced them.

## Start here

**Building an integration.** Go to the module you need. The standard is organised as twelve
modules, one directory each, and every directory holds the data model and the API surface side by
side.

| | | | |
| :--- | :--- | :--- | :--- |
| [1 Core Parties](standard/01-core-parties/) | [2 Products](standard/02-products-catalog/) | [3 Motor](standard/03-motor/) | [4 Travel](standard/04-travel/) |
| [5 Term Life](standard/05-term-life/) | [6 Property](standard/06-property/) | [7 Cyber Liability](standard/07-cyber-liability/) | [8 Business Interruption](standard/08-business-interruption/) |
| [9 Trade Credit](standard/09-trade-credit/) | [10 Pet](standard/10-pet/) | [11 Claims](standard/11-claims/) | [12 Premium and Receipts](standard/12-premium-receipts/) |

Read [`standard/conventions.md`](standard/conventions.md) once, because authentication, errors,
pagination and idempotency apply to every module. Read
[`standard/KNOWN-GAPS.md`](standard/KNOWN-GAPS.md) before you commit to anything, because it lists
what this version does not settle.

**Working in Vietnam.** [`markets/vn/`](markets/vn/) carries the Vietnam profile.

**Bringing a new market.** [`markets/README.md`](markets/README.md) says what a profile is and what
starting one involves.

**Deciding whether to trust this.** [`GOVERNANCE.md`](GOVERNANCE.md) says who decides and how, and
[`standard/inherited/`](standard/inherited/) says exactly what was inherited and what is being
changed.

## Where this came from

The Open Insurance Initiative published the OPIN Data Standard, reaching v1.2.1 in December 2021,
and the OPIN API Specification v1.0 in May 2022. Both are open: the data standard under MPL 2.0 and
the API specification under Apache 2.0. Neither has been revised since.

That is a problem for anyone building on them, because twenty structural defects in the published
standard have no route to a fix. A claim carries no foreign key to the coverage it belongs to. A
receipt cannot be reconciled back to the policy that produced it. Trade credit is missing the
lifecycle fields every other coverage type has. Those are catalogued in
[`standard/inherited/concerns-v1.2.1.md`](standard/inherited/concerns-v1.2.1.md), with fourteen
more against the API specification in
[`standard/inherited/concerns-api-v1.0.md`](standard/inherited/concerns-api-v1.0.md), and until now
the only thing an implementer could do about them was carry a private patch.

This repository continues the standard so those get fixed at source. The published v1.2.1 and v1.0
are kept intact and unmodified for reference, and every change made on top of them is recorded
against the specific defect it closes.

## What is being changed, and what is not

**Changed.** The twenty defects, worked one at a time, each with the reasoning written down. The
data standard and the API specification are brought onto one version number, because two documents
describing one standard at two versions is how they came to disagree with each other.

**Not changed.** Field names that are misspelled but stable on the wire. `creditLimitUtiilized` and
`GrosslLossReserve` are wrong and they are also what every existing implementation sends. Renaming
them is a compatibility break wearing a tidy-up's clothes, so they are reported and left alone. Any
change that would break a working integration gets the same treatment: it is written down, and it
waits for a major version.

## Status

v1.5.0 is a draft and says so. It is being built against and it is not ratified.

The version is Trisilva's work, published openly on top of the inherited v1.2.1 vocabulary. It
stays on the 1.x line because nothing inherited was removed or renamed, and it skips 1.3 and 1.4 so
that the Open Insurance Initiative keeps room on its own line if it publishes again.
[`VERSIONING.md`](VERSIONING.md) carries the reasoning, including why there is no directory per
version.

The Vietnam profile is at v0.2 and is nearly empty, which is the honest position rather than a gap.
Most of what was previously filed there turned out to be base-standard work and has moved to
[`standard/`](standard/), where every market gets it.

## Who maintains this

Trisilva builds insurance operations software in Southeast Asia and does the current editorial
work. [`GOVERNANCE.md`](GOVERNANCE.md) sets out how a change is decided, how editorship works, and
what happens to a proposal that a maintainer disagrees with.

The standard is not a Trisilva product and carries no Trisilva product behaviour. Distribution
mechanics, commission ledgers, workflow sub-states and operational service-level measurement all
sit above this line, in whatever platform an implementer builds. A proposal that would pull any of
them in is out of scope, and so is a proposal that only makes sense because of one vendor's
software. If you find something here that reads that way, it is a defect and an issue is the right
response.

## Contributing

[`CONTRIBUTING.md`](CONTRIBUTING.md) covers what a useful issue looks like. The most valuable
contribution is a defect in the standard itself, because it compounds for everyone who builds on it.

## Licence and attribution

MPL 2.0. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

OPIN, Open Insurance Initiative and the Open Insurance logo are marks of the Open Insurance
Initiative. Used here to identify the standard this work continues.
