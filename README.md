# OPIN

**An open data and API standard for insurance, and the market profiles that make it operable in one
country.**

OPIN gives insurers, brokers, distributors and the systems between them a shared way to describe
parties, products, coverage, claims, premium and receipts. It was created by the Open Insurance
Initiative and published between 2018 and 2022. This repository continues it.

**Current version: v1.5.0-draft.** Additive over the inherited baseline, and the version
implementations are being built against.

<!-- repo-only -->
**Read it as a site.** The standard is published at
[trisilva.ai/docs/standard](https://trisilva.ai/docs/standard), rendered from this repository with
search and working cross-links between the modules, and the Vietnam market profile is at
[trisilva.ai/docs/markets/vietnam](https://trisilva.ai/docs/markets/vietnam). This repository stays
the canonical source: every page there carries an edit link back to the file it came from. Trisilva
maintains the standard and publishes it open under MPL 2.0.
<!-- /repo-only -->

## The twelve modules

The standard is the twelve directories at the root of this repository. Each holds the data model and
the API surface for one module side by side, plus a README naming what to check before implementing
it.

| | Module | What it covers |
| :--- | :--- | :--- |
| 1 | [Core Parties and Entities](01-core-parties/) | The insurer, the broker, the personal and commercial policyholder, beneficiaries, addresses |
| 2 | [Products and Catalog](02-products-catalog/) | The product record and the catalogue it sits in |
| 3 | [Motor](03-motor/) | Motor coverage, vehicle, driver. The only module the inherited API covers |
| 4 | [Travel](04-travel/) | Travel coverage and the traveller |
| 5 | [Term Life](05-term-life/) | Term life coverage, the life insured, riders |
| 6 | [Property](06-property/) | Property coverage, buildings and contents |
| 7 | [Cyber Liability](07-cyber-liability/) | Cyber liability coverage and the insured business |
| 8 | [Business Interruption](08-business-interruption/) | Business interruption, usually written alongside property |
| 9 | [Trade Credit](09-trade-credit/) | Trade credit coverage, the debtor and the credit limit |
| 10 | [Pet](10-pet/) | Pet coverage and the insured animal |
| 11 | [Claims](11-claims/) | The claim, from first notification through settlement |
| 12 | [Premium and Receipts](12-premium-receipts/) | Premium, receipts, and the bordereaux that report to reinsurers |

Modules 1, 2, 11 and 12 are cross-cutting and every coverage type uses them. Modules 3 to 10 are the
eight coverage lines.

Four pages sit alongside the modules and apply to all of them.

| Page | What it holds |
| :--- | :--- |
| [`concepts.md`](concepts.md) | The insurance vocabulary the modules assume. Read first if the domain is new to you |
| [`conventions.md`](conventions.md) | Base URL, authentication, error model, pagination, idempotency, action endpoints and extensions. Read once |
| [`cross-module.md`](cross-module.md) | How the entities relate across modules, and the claim flow that spans every coverage type |
| [`SCOPE.md`](SCOPE.md) | How the entities link, what is settled by convention, and what the standard leaves to the platforms above it |

## How to read this

**New to insurance.** Start at [`concepts.md`](concepts.md). It defines the vocabulary every module
assumes: premium, deductible, peril, indemnity limit, endorsement, reserve, bordereau and the rest.
An hour there saves a day everywhere else.

**Building one coverage line.** Open that module's directory. Its README says what the line of
business is and what to watch, then `data-model.md` says what the fields mean and `api.md` gives you
the surface. Read [`conventions.md`](conventions.md) once, because it applies everywhere. Read
[Motor](03-motor/) first even if you are not building motor, because every other module's endpoints
mirror the shape declared there.

**Building in one market.** Read the module, then the
[market profile](project/markets/README.md) for your market. A profile constrains and adds; it
never changes what a field means.

**Fixing a defect.** [`project/inherited/`](project/inherited/) catalogues twenty defects in the
inherited data standard and fourteen in the inherited API specification, each with the position this
version takes on it. Four of the structural ones are closed here and a fifth is partly closed. The
rest are compatibility breaks held for one major version so an implementer absorbs them together.

## The second layer: market profiles

Everything a market shares lives in the standard. Only what a market does not share lives in a
profile, at [`project/markets/`](project/markets/). That line is the whole design, and it is
stricter than it sounds: authentication, error handling, pagination and record lifecycle are the
same problem in Hanoi and in Manila, so they belong to the standard even though a single market
surfaced them.

Vietnam is the first profile, at [`project/markets/vn/`](project/markets/vn/), and it is v0.2. It
is deliberately thin. Most of what was once filed there turned out to be base-standard work and
has moved into the modules, where every market gets it. A thin profile is the design working:
the more a profile carries, the less the standard settled.

## Where this came from

The Open Insurance Initiative published the OPIN Data Standard, reaching v1.2.1 in December 2021,
and the OPIN API Specification v1.0 in May 2022. Both are open: the data standard under MPL 2.0 and
the API specification under Apache 2.0. Neither has been revised since.

That is a problem for anyone building on them, because twenty structural defects in the published
standard have no route to a fix. A claim carries no foreign key to the coverage it belongs to. A
receipt cannot be reconciled back to the policy that produced it. Trade credit is missing the
lifecycle fields every other coverage type has. Those are catalogued in
[`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md), with fourteen more
against the API specification in
[`project/inherited/concerns-api-v1.0.md`](project/inherited/concerns-api-v1.0.md), and until now
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
waits for a major version. See [`project/VERSIONING.md`](project/VERSIONING.md).

## How the project runs

Everything that is not the standard itself is in [`project/`](project/).

| | |
| :--- | :--- |
| [`project/GOVERNANCE.md`](project/GOVERNANCE.md) | Who decides, how, and what a contributor can rely on. Includes the editor list and the conflict of interest this project carries |
| [`project/VERSIONING.md`](project/VERSIONING.md) | What makes a change major, minor or patch, and why this version is 1.5.0 |
| [`project/CONFORMANCE.md`](project/CONFORMANCE.md) | The three conformance levels, how a claim is stated, and the artefacts that make each level checkable |
| [`project/inherited/`](project/inherited/) | What was received from the Open Insurance Initiative, kept so it can be told apart from what was changed |
| [`project/markets/`](project/markets/) | The market profiles and the test for what belongs in one |
| [`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md) | What a useful issue looks like. The most valuable contribution is a defect in the standard itself |

Trisilva builds insurance operations software in Southeast Asia and does the current editorial
work. The standard is not a Trisilva product and carries no Trisilva product behaviour:
distribution mechanics, commission ledgers, workflow sub-states and operational service-level
measurement all sit above this line, in whatever platform an implementer builds. That scope line is
how the conflict of interest is managed, and
[`project/GOVERNANCE.md`](project/GOVERNANCE.md) sets out the rest.

## Licence and attribution

MPL 2.0. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

OPIN, Open Insurance Initiative and the Open Insurance logo are marks of the Open Insurance
Initiative. Used here to identify the standard this work continues.
