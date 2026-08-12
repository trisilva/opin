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

No foreign key carries this. `Claim` has no field pointing at its coverage and `Receipt` has none
pointing at the policy or claim that produced it. The uniqueness rule is what makes the association
deterministic, which is why it is stated as a constraint rather than left as a convention.

## Lifecycle is declared, not implied

`policyStatus` and `claimStatus` carry their value sets in the data model, and the state
diagrams in each module's API page walk the legal transitions. Those diagrams are normative. A
transition they do not draw is not a transition an implementation may make.

Publishing a value set without the transitions is not enough. Two implementations can agree on every
field and still disagree about whether a policy may move from lapsed back to active, and that is the
kind of disagreement that surfaces in production rather than in review.

## Field naming

The API design is authoritative for anything that travels on the wire. The data model is
authoritative for what a field means.

Several field names are misspelled. The wire keeps the misspelling and the data model shows the
corrected name beside it. So `creditLimitUtiilized`, `issueDtae`, `GrosslLossReserve`,
`countryOfRegisteration` and `medicalConditon` are what an implementation sends, and the corrected
form tells a reader what the field was meant to be called.

The rule reads oddly until you see what it protects. These names are on the wire in every existing
implementation. Correcting them would break working integrations to make a document tidier, which is
a trade the standard does not make. The corrections ship with the other compatibility breaks, in one
major version, so an implementer absorbs them together.

## Compatibility breaks, and when they ship

Three wire identifiers still carry a market profile name that no longer owns them. The base URL
is `https://api.opin-vn.{tld}/v1`, and the two OAuth scopes are `opin-vn.admin` and
`opin-vn.developer`. This material is base-standard work that applies in every market, so the
names are wrong.

Use them as written. Changing a base URL or a scope name breaks every caller, which makes it a
major change however small the edit looks, and this version is additive. See
[`VERSIONING.md`](project/VERSIONING.md) for what that means and when the held breaks ship.

These three are the only place that name survives. Anywhere else you meet an `opin-vn` string, it is
on the wire and it is held for a major version.

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

## The API surface

All twelve modules carry endpoints, and they carry the same ones. Create and list on a collection,
retrieve and replace on an item, plus the lifecycle actions each coverage type needs. Motor declares
the pattern and everything else mirrors it, which is why reading
[Motor](03-motor/) first is worth the time even if you are building something else.

No entity schema is invented anywhere. The vocabulary is inherited unmodified and the surface over
it is this standard's work.

Where the two source documents disagree, the disagreement is recorded on the module page rather than
silently resolved. [Term life](05-term-life/) carries the sharpest case.
