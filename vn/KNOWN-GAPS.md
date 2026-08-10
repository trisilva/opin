# OPIN-VN v0.2: known gaps

Everything on this page is a reason to be careful with v0.2. It is published because an implementer
needs it before committing, not after.

Gaps fall into three kinds, and the distinction matters when deciding what to do about one. Some
are inherited from OPIN and should be fixed upstream. Some are things v0.2 handles by convention
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
brokerage or endorsement fields. The module is present in this track for completeness of the
vocabulary and is not usable as a wire contract at v0.2. This is an OPIN defect, and closing it at
the country layer would mean inventing fields OPIN has not defined, which is exactly the kind of
quiet fork this track exists to avoid.

**Data standard and API specification disagree in places.** `termLifeType` and `termLifeRiders`
carry different value sets in the two documents. The track treats the data standard as
authoritative and flags each divergence inline as `[OPIN concern]`.

The full list of twenty is at [`../upstream/opin-concerns.md`](../upstream/opin-concerns.md).

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

The data schema applies an `[OPIN-VN normalisation]` in these places, rendering the corrected name
and flagging the original for upstream report. The API design preserves them verbatim, on the
grounds that they are already on the wire in every existing implementation.

Both positions are defensible and they cannot both be right in one track. Until this is resolved,
treat the API design document as authoritative for anything that travels on the wire, and read a
normalisation in the data schema as a note about what OPIN should have called the field rather than
what your implementation should send. Resolving the inconsistency is on the list for the next patch.

## Out of scope by design

These are not gaps to be closed in a later version. They sit above the country track, in whatever
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
schemas without endpoints, and this track adds the endpoints by mirroring the motor CRUD pattern.
Those additions are annotated inline as `[OPIN-VN extension to API; OPIN schema reused]`. No new
entity schemas are introduced anywhere in this track.
