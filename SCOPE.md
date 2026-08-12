# OPIN v1.5.0-draft: scope and design

What this version settles, how the entities link to one another, and what the standard leaves
to the platforms built on it.

A standard is defined as much by what it refuses to decide as by what it declares. This page
holds both, so an implementer can see the whole boundary in one place rather than inferring it
from twelve modules.

## How the model links

`policyNumber` is the linkage key. It is globally unique across the namespace, and it is what
ties a coverage record to the claims and receipts that arise from it.

That single rule is what makes the polymorphic entry point work. `POST /claim` accepts a claim
against any of the eight coverage types, and it can do that because `policyNumber` resolves
deterministically to exactly one coverage record. An implementation that scoped policy numbers
per line of business would break the one endpoint every caller uses.

Three things follow from it, and all three are normative:

- Collection endpoints expose `?policyNumber=` and `?claimNumber=` filters, so a caller can
  reach the related records without a join the standard does not own.
- A request body carries `policyNumber` wherever the linkage matters, rather than relying on
  the path alone.
- An implementation assigns policy numbers from one sequence across every coverage type it
  writes.

The inherited baseline left this implicit. It declared no foreign key from `Claim` to coverage
and none from `Receipt` to the policy or claim that produced it, and it inferred the
relationship from correlation between `claimNumber` and `policyNumber`. Correlation is not a
contract, so this version states the constraint that the model was already assuming. The
catalogue entries behind that decision are in
[`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md).

## Lifecycle is declared, not implied

`policyStatus` and `claimStatus` carry their value sets in the data model, and the state
diagrams in each module's API page walk the legal transitions. Those diagrams are normative. A
transition they do not draw is not a transition an implementation may make.

This is stricter than the inherited baseline, which published the value sets and left the
transitions to the reader. Two implementations that agreed on every field could still disagree
about whether a policy may move from lapsed back to active, which is the kind of disagreement
that surfaces in production rather than in review.

## Field naming

The API design is authoritative for anything that travels on the wire. The data model is
authoritative for what a field means.

Where the inherited standard misspelled a field name, the wire keeps the misspelling and the
data model records the corrected name beside it as a `[normalisation]`. So `creditLimitUtiilized`,
`issueDtae`, `GrosslLossReserve`, `countryOfRegisteration` and `medicalConditon` are what an
implementation sends, and the normalisation tells a reader what the field was meant to be
called.

The rule reads oddly until you see what it protects. These names are already on the wire in
every existing implementation. Correcting them would break working integrations to make a
document tidier, which is a trade the standard does not make. The corrections ship with the
other compatibility breaks, in one major version, so an implementer absorbs them together.

## Compatibility breaks, and when they ship

Three wire identifiers still carry a market profile name that no longer owns them. The base URL
is `https://api.opin-vn.{tld}/v1`, and the two OAuth scopes are `opin-vn.admin` and
`opin-vn.developer`. This material is base-standard work that applies in every market, so the
names are wrong.

Use them as written. Changing a base URL or a scope name breaks every caller, which makes it a
major change however small the edit looks, and this version is additive. See
[`VERSIONING.md`](project/VERSIONING.md) for what that means and when the held breaks ship.

These three are the only place the retired profile name survives on the wire. The annotation
markers that also carried it were prose rather than contract, so they were swept:
`[OPIN-VN extension to API; OPIN schema reused]` is now `[added]`, and `[OPIN-VN normalisation]`
is now `[normalisation]`. A reader meeting an `opin-vn` string has one question to ask rather
than two. If it travels on the wire it is held, and if it does not it is already gone.

## Out of scope by design

These sit above the standard, in whatever platform an implementer builds. Pulling them in would
turn a shared contract into one vendor's product specification, and a standard that encodes one
vendor's product is a standard nobody else can adopt.

- Distribution channels, commission ledgers and payout schedules to distribution partners
- Telematics ingestion and event streams
- Parametric and index-linked triggers, and the oracles behind them
- Breach-notification timelines and operational service-level measurement
- Operational claim sub-states such as triage, awaiting documents, awaiting payment and fraud
  review

The last one is worth stating plainly, because it is the one most often assumed to belong here.
A claim's business status belongs to the standard. A claim's position in one operator's
workflow does not.

The standard will also never carry a pricing model or a rating engine. Pricing is where
insurers compete, and a standard that settled it would be a standard nobody could adopt. That
exclusion is permanent and it is not a gap.

## Coverage of the API surface

The inherited API specification covered one module. Motor carried endpoints, and everywhere
else the baseline published entity schemas with no surface over them.

This version carries endpoints for all twelve, mirroring the CRUD and lifecycle pattern motor
declared. Those additions are annotated inline as `[added]`, so a reader can tell what came
from the baseline and what this version supplies. No new entity schemas are introduced. The
vocabulary is the baseline's, and the surface over it is this version's work.
