# Vietnam profile

**Profile version 0.2. Targets OPIN v1.5.0-draft.**

## What a profile is for

A profile carries what one market requires that a global standard cannot decide for it, and nothing
else. It constrains and it adds. It never changes what a field means, because a field that means one
thing in Hanoi and another in Manila is not part of a standard.

This profile is deliberately thin, and that is the design working rather than a gap in it.
Authentication, error handling, pagination, idempotency and record lifecycle are the same problem in
Hanoi as in Manila, so they belong to [the standard](../../../) where every market gets them.

The more a profile carries, the less the standard settled. A thick profile is a symptom.

## Money

Vietnamese dong is unit-denominated. It carries no minor units, so the two-decimal assumption that
most money handling makes does not hold here.

Monetary amounts in this market are integers in dong, with currency `VND` under ISO 4217 and an
exponent of zero. An implementation that stores minor units and divides by one hundred on the way
out will be wrong by two orders of magnitude on every amount it writes.

This has a wire consequence, which is why it is stated in the profile rather than left to the reader.
Premium, sum insured, deductible, reserve and receipt amounts are all affected.

## Addresses

Vietnam's administrative structure does not map onto a generic address shape without a decision, so
the profile makes one. Administrative units are carried as named components in the standard's address
entity rather than flattened into free-text lines, because an address that has been flattened cannot
be matched, aggregated or filed with a regulator afterwards.

The ordering is largest to smallest, which is the order a Vietnamese address is written and spoken.
An implementation that renders an address for display reverses nothing.

## Identity documents

The party model's identity document types are extended with the documents Vietnam actually issues.
The base standard's enum was not built around them, and a party identified by a document type the
enum cannot name is a party an implementation has to describe in free text.

The extension follows the extension rules in [`conventions.md`](../../../conventions.md), so a caller
that does not recognise a Vietnamese document type still parses the record.

## Statutory claim handling

Decree 67/2023/ND-CP governs claims settlement in Vietnam. It works through contract-stated
timeframes rather than through one fixed national response window, so the profile models the
timeframe as a value carried on the coverage record and not as a constant in the standard.

That distinction is the whole modelling decision. A national constant would be simpler and it would
be wrong, and it would also be the kind of thing that has to be unpicked in every other market whose
regulator works the same way.

**This section is a modelling decision, not a legal opinion.** An implementation operating in Vietnam
takes the timeframe from its own policy wording and its own counsel. The profile says where the value
lives, not what the value is.

## Personal data

Law 91/2025 on personal data protection carries residency and handling obligations that reach what an
implementation may transmit and where it may hold it.

The profile's position is that residency is a deployment property rather than a field. A record does
not carry a flag saying where it may live; the deployment it was created in is what decides that, and
the standard's job is to make sure nothing in the wire contract forces a personal record across a
border. Nothing in this profile requires a Vietnamese policyholder record to be readable outside
Vietnam.

## What does not belong here

Anything another market would also need. That goes to the standard, and the test is in
[`project/markets/README.md`](../README.md).

Anything that only makes sense because of one vendor's software. Distribution mechanics, commission
ledgers, workflow sub-states and operational service-level measurement all sit above this profile, in
whatever platform an implementer builds.

## Editorial accountability

Profile changes are proposed and accepted the same way standard changes are, through
[`project/GOVERNANCE.md`](../../GOVERNANCE.md). A profile that encodes a market's regulation carries a
named editor before it is cited as authority for a compliance position, and the governance page is
where that name is recorded.
