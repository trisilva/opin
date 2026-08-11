# Vietnam profile

**Profile version 0.2. Targets OPIN v1.5.0-draft. Working draft, and nearly empty.**

## Read this first

This profile is close to empty, and that is the accurate state rather than work in progress that has
stalled.

At v0.1 this profile carried the OPIN data model rendered consistently, plus authentication, an error
model, pagination, idempotency, item-level operations, lifecycle endpoints and claim-to-coverage
linkage. None of that is Vietnamese. Every market needs all of it, and it sat here only because the
standard could not be changed at the time, so base-layer work had nowhere else to go.

All of it has moved to [the standard](../../../), where every market gets it. What is left in
this profile is what is genuinely specific to Vietnam, and most of that has not been written yet.

If you are building in Vietnam, the standard is what you implement today. This profile is what will
sit on top of it.

## What belongs here

Named so a reader can see the shape of the work and so the gaps are checkable. None of these are
written. None should be treated as specified.

**Statutory claim handling.** Decree 67/2023/ND-CP governs claims settlement in Vietnam. What it
requires of a response timeframe, and whether that is a fixed national window or a contract-stated
one, has to be confirmed with Vietnamese counsel before anything is written here. It is not
confirmed.

**Personal data.** Law 91/2025 on personal data protection carries residency and handling
obligations that affect what an implementation may transmit and where it may hold it. The profile has
to state what that means for the fields in the standard.

**Identity documents.** The party model needs the document types Vietnam actually uses, and the
enum in the standard is not built around them.

**Currency and money.** Vietnamese dong is unit-denominated and does not carry minor units the way
the standard's money handling assumes. This has a wire consequence and needs stating.

**Addresses.** Vietnam's administrative structure does not map onto the standard's address shape
without a decision about how it is carried.

## What does not belong here

Anything another market would also need. That goes to the standard, and the test is in
[`project/markets/README.md`](../README.md).

Anything that only makes sense because of one vendor's software. Distribution mechanics, commission
ledgers, workflow sub-states and operational service-level measurement all sit above this profile.

## Accountable for this profile

Unassigned. A profile that claims to encode a market's regulation needs someone accountable for the
claim, and nobody is named here yet. Until someone is, treat everything above as scope rather than
as specification.

## Status

The regulatory items are gated on Vietnamese counsel and none has a date. The currency and address
items are not gated on anything and are the first things that should be written.
