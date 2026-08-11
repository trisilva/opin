# OPIN v1.5.0-draft: known gaps

Everything on this page is a reason to be careful with this version. It is published because an implementer
needs it before committing, not after.

Gaps fall into three kinds, and the distinction matters when deciding what to do about one. Some
are inherited from OPIN and are this standard's to fix. Some are things this version handles by convention
rather than by schema. Some are deliberate exclusions that will never be closed here.

## Inherited from OPIN

**Claim carries no foreign key to Coverage or Policy.** OPIN's `Claim` entity infers the
relationship from `claimNumber` and `policyNumber` correlation rather than declaring a reference.
For a claims implementation this is the central entity relationship in the model, and it is the one
OPIN leaves implicit.

**Receipt carries no `policyRef` or `claimRef`.** The same shape, one module over. Reconciling a
payment back to the policy or claim that produced it requires foreign keys the schema does not
declare.

**Trade credit (module 9) is structurally incomplete.** Unlike every other coverage entity,
`tradeCreditCoverage` carries no `inceptionDate`, no `expiryDate`, no `status`, and no premium,
brokerage or endorsement fields. The module is present in the standard for completeness of the
vocabulary and is not usable as a wire contract at this version. This is an OPIN defect, and closing it in a
market profile would mean inventing fields OPIN has not defined, which is exactly the kind of
quiet fork the standard exists to avoid.

**Data standard and API specification disagree in places.** `termLifeType` and `termLifeRiders`
carry different value sets in the two documents. This version treats the data standard as
authoritative and flags each divergence inline as `[OPIN concern]`.

The full list of twenty is at [`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md).

## Handled by convention, not by schema

**`policyNumber` is treated as globally unique across the namespace.** This is the workaround for
the two missing foreign keys above. Collection endpoints expose `?policyNumber=` and
`?claimNumber=` filters, and implementers are expected to carry `policyNumber` in the request body
where the linkage is needed.

This is wire-level fragility and should be read as such. Every implementer has to agree to a
convention the schema does not declare, which means an implementation can be fully conformant and
still fail to interoperate. Tightening this into a normative schema note is the first item on the
next patch.

**Lifecycle transitions are implied rather than declared.** `policyStatus` and `claimStatus` carry
their value sets, and the state diagrams in the API design document walk the transitions
conservatively, but nothing here is normative. Two conformant implementations can disagree about
whether a given transition is legal. A v1.0 has to declare them.

**The two documents disagree about misspelled field names.** OPIN carries spelling errors in field
names and enum values: `creditLimitUtiilized`, `issueDtae`, `GrosslLossReserve`, `countryOfRegisteration`,
`medicalConditon` and others.

The data schema applies a `[normalisation]` in these places, rendering the corrected name
and recording the original beside it. The API design preserves them verbatim, on the
grounds that they are already on the wire in every existing implementation.

Both positions are defensible and they cannot both be right in one standard. Until this is resolved,
treat the API design document as authoritative for anything that travels on the wire, and read a
normalisation in the data schema as a note about what OPIN should have called the field rather than
what your implementation should send. Resolving the inconsistency is on the list for the next patch.

## Breaking changes held for a major version

**Three wire identifiers still name a market profile that no longer owns them.** The base URL is
`https://api.opin-vn.{tld}/v1` and the two OAuth scopes are `opin-vn.admin` and
`opin-vn.developer`. This material is base-standard work and applies to every market, so the names
are wrong.

They are not corrected here. Changing a base URL or a scope name breaks every caller, which makes
this a major change however small the edit looks, and this version is additive. See
[`VERSIONING.md`](project/VERSIONING.md).

Use them as written. They are strings, they identify the right things, and the name being wrong
costs a reader a moment of confusion rather than costing an integration anything. The correction
ships with the other held breaks, together, so an implementer absorbs one change instead of three.

These three are now the only place the retired profile name survives. The annotation markers that
also carried it were prose rather than wire, so they were swept: `[OPIN-VN extension to API; OPIN
schema reused]` is now `[added]` and `[OPIN-VN normalisation]` is now `[normalisation]`. That leaves
a reader one question to ask about any `opin-vn` string they meet, rather than two: if it travels on
the wire it is held, and if it does not it is already gone.

## Out of scope by design

These are not gaps to be closed in a later version. They sit above the standard, in whatever
platform an implementer builds, and pulling them in would turn a shared contract into one vendor's
product specification.

- Distribution channels, commission ledgers and payout schedules to distribution partners
- Telematics ingestion and event streams
- Parametric and index-linked triggers, and the oracles behind them
- Breach-notification timelines and operational service-level measurement
- Operational claim sub-states such as triage, awaiting documents, awaiting payment and fraud review

The last one is worth stating plainly, because it is the one most often assumed to belong here. A
claim's business status is OPIN's. A claim's position in one operator's workflow is not.

## Coverage of the API surface

Only the motor module carries endpoints in OPIN itself. Everywhere else, OPIN publishes entity
schemas without endpoints, and this version adds the endpoints by mirroring the motor CRUD pattern.
Those additions are annotated inline as `[added]`. No new
entity schemas are introduced anywhere in this version.
