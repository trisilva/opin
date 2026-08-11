# Contributing

Open an issue. No agreement to sign, no membership, no form.

## What is most useful

**A defect in the standard.** Something that is wrong, inconsistent, or missing a relationship the
model needs. These matter more than anything else here, because everyone building on the standard
carries them until they are fixed. Twenty are already catalogued in
[`standard/inherited/concerns-v1.2.1.md`](standard/inherited/concerns-v1.2.1.md) and more are
likely.

**A gap that stops two implementations interoperating.** The strongest version of this names the
two implementations and what they would disagree about. A gap nobody has hit is a question; a gap
that broke an integration is a defect.

**A correction against source.** If an entity, field or enum does not match the OPIN sheet it
claims to trace to, cite the sheet.

**A market with no profile.** Say which one and what makes it different, and cite the regulation. A
market difference with no statutory or operational driver behind it is usually base-standard work
that has not been recognised as such.

## What is out of scope

**Vendor product behaviour.** Distribution mechanics, commission ledgers, workflow sub-states,
operational service-level measurement. These sit above the standard and above every profile, in
whatever platform an implementer builds. This applies to proposals from maintainers exactly as it
applies to anyone else.

**Silent renames of misspelled fields.** They are misspelled on the wire in every existing
implementation. Correcting one breaks every caller, so it is held for a major version rather than
applied as a tidy-up. See [`VERSIONING.md`](VERSIONING.md).

**A profile that contradicts the standard.** If a market needs the standard to be different, that is
an issue against the standard. A local override is a fork with extra steps.

## Where a change goes

One question decides it. **Would another market need this too?** If yes, the standard. If no, a
profile. [`markets/README.md`](markets/README.md) carries the worked version of the test.

## What happens to your issue

Every decision records a reason, and a rejection without one is a defect in the process that you
should raise. [`GOVERNANCE.md`](GOVERNANCE.md) has the detail, including how editorship opens and
what happens when a maintainer disagrees with you.

Anything in the documents marked as a reading rather than a settled fact is explicitly open. Those
are the places where a well-argued objection should be expected to win.

## Proposals

Anything that changes the standard needs a proposal before it is accepted: what changes, what
breaks, who it affects, and what an existing implementation has to do about it. Corrections that do
not touch the wire skip this.

Say plainly if a change breaks compatibility. A proposal that hides a break is worse than one that
argues for it.

## Licence of contributions

Contributions are accepted under MPL 2.0, matching the repository. You keep your copyright.
