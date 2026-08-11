# Versioning

## The standard

Semantic versioning, read against the wire rather than against the document.

| Change | Version | Example |
| :--- | :--- | :--- |
| A conformant implementation stops working | **Major** | Renaming a field, removing an enum value, tightening a type |
| Something is added and existing callers are unaffected | **Minor** | A new endpoint, a new optional field, a new enum value |
| The document is corrected and the wire is unchanged | **Patch** | A description fixed, a reference pointed at the right sheet |

The test is what an existing integration has to do, not how large the edit looks. Correcting a
misspelled field name is a one-character diff and a major change, because every caller sending the
old name breaks.

## Why the current version is 1.5.0

The inherited material sits at data standard v1.2.1 and API specification v1.0. This version is
1.5.0, and the gap between them is deliberate.

**The gap says this is not the initiative's next revision.** 1.3.x and 1.4.x are left unclaimed, so
if the Open Insurance Initiative publishes again, it has room on its own line and nothing here
collides with it. Numbering this 1.3.0 would have implied a continuity of authorship that does not
exist.

**It is also honest about the size of the change.** The inherited API specification publishes eight
endpoints, all of them motor. This version covers twelve modules and adds authentication, an error
model, pagination, idempotency, action endpoints and record lifecycle. That is not a point release.

**It stays on the 1.x line because the work is additive.** Nothing inherited was removed or
renamed, so an implementation built against v1.2.1 vocabulary still holds. Starting again at 1.0
would suggest a break that has not happened.

**One line, not two.** Two documents describing one standard at two version numbers is how the
inherited material came to disagree with itself about `termLifeType` and `termLifeRiders`, recorded
as concern 1 of the twenty. The data model and the API design now share a version.

## Draft versions

A version marked draft is being built against and is not ratified. It can change without a major
bump while it is a draft, which is exactly why the label is on it. Do not treat a draft as stable.

A version leaves draft when it declares its lifecycle transitions normatively and closes the
inherited concerns it claims to close. Nothing here carries a date it does not have.

## Market profiles

Profiles version separately from the standard and from each other, because Vietnam moving does not
mean Indonesia moved. A tag reads `vn-v0.2`. A standard tag reads `v1.5.0`.

Every profile declares the standard version it targets. A profile never targets a range.

## What is kept

Every published version stays readable at its tag. The working tree carries one version of the
standard, the current one, and each released version is reachable through its git tag and recorded
in [`CHANGELOG.md`](CHANGELOG.md). There is no directory per version, because a copy of the whole
standard per release accumulates trees nobody reads and that drift from the one that matters.

The inherited material is the one exception, and it is not an older version of this work. The
published v1.2.1 and v1.0 are kept unmodified in [`standard/inherited/`](standard/inherited/) so
that anyone can see exactly what was changed and by whom.

## Deprecation

Something being removed is marked deprecated in one minor version, carries a note saying what to
use instead, and is removed no earlier than the next major. Nothing is removed without appearing
deprecated first.

## Breaking changes that are already needed

Some inherited defects cannot be fixed without breaking the wire. Misspelled field names are the
clearest case: `creditLimitUtiilized` and `GrosslLossReserve` are wrong, and correcting them breaks
every implementation sending the current spelling.

These are held rather than applied. They accumulate against a future major version, each recorded
with the defect it closes, and they ship together so an implementer absorbs one break instead of
several. Until then the misspelling is what goes on the wire, and the correct name appears only as
a note.
