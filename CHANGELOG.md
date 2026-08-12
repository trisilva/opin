# Changelog

The standard and each profile version separately. A standard tag reads `v1.5.0`, a profile tag reads
`vn-v0.2`. See [`VERSIONING.md`](project/VERSIONING.md).

---

# The standard

## v1.5.0-draft (unreleased)

Trisilva's work on top of the inherited OPIN v1.2.1 vocabulary, published openly. The first release
on this standard in four years.

The version stays on the 1.x line because nothing inherited was removed or renamed, and it skips
1.3 and 1.4 so the Open Insurance Initiative keeps room on its own line. See
[`VERSIONING.md`](project/VERSIONING.md).

### Two version lines became one

The Open Insurance Initiative published the data standard at v1.2.1 and the API specification at
v1.0, separately. That is how they came to disagree with each other about `termLifeType` and
`termLifeRiders`, recorded as concern 1 of the twenty inherited. They are now one standard on one
version line.

### Base-layer work moved out of the Vietnam profile

Authentication, the error model, cursor pagination, idempotency, item-level operations, lifecycle
endpoints and claim-to-coverage linkage were previously filed as the Vietnam market profile. None of
them are Vietnamese. Every market needs all of them, and they sat in a profile only because the
standard could not be changed at the time.

They are now in the standard. The Vietnam profile keeps only what is specific to Vietnam, which is
close to nothing until the statutory work is done. See [`project/markets/vn/`](project/markets/vn/).

This is the largest change in the version and it is a relocation, not a rewrite. No content was
altered in the move.

### One directory per module, at the root of the repository

The standard was two documents of about fourteen hundred lines each, one for the data model and one
for the API, each organised by the same twelve modules. A reader building one coverage line had to
work both documents in parallel.

It is now one directory per module, holding that module's data model, its API surface and a README
naming what to check before implementing it. Those twelve directories sit at the root of the
repository, because the standard is what the repository is for and it should be the first thing
visible in it. What applies to every module was lifted out beside them, to
[`conventions.md`](conventions.md), [`cross-module.md`](cross-module.md) and
[`SCOPE.md`](SCOPE.md).

Everything that is not the standard is in [`project/`](project/): governance, versioning,
conformance, the inherited material and the market profiles. Contributing, the code of conduct and
the security policy are in `.github/`, where GitHub reads them natively. `MAINTAINERS.md` was two
one-row tables and is now part of [`GOVERNANCE.md`](project/GOVERNANCE.md), which already carried
the reasoning behind both.

The split is a relocation. No normative content was rewritten in the move.

### No directory per version

`standard/v1.5.0-draft/` is gone and the working tree carries one version of the standard, the
current one. Released versions are reachable by git tag and recorded here. A copy of the whole
standard per release accumulates trees nobody reads.
[`VERSIONING.md`](project/VERSIONING.md) carries the reasoning.

### Country tracks became market profiles

`tracks/` is now [`project/markets/`](project/markets/) and the layer is called a market profile.
The old name had to be explained before the thing it named could be, and
[`CONFORMANCE.md`](project/CONFORMANCE.md) already called the third conformance level "market".
Profile version tags are unchanged, so `vn-v0.2` still reads `vn-v0.2`.

The profiles are a peer layer to the standard, and they are filed under `project/` only because
there is one of them and it is nearly empty. `project/markets/README.md` records what moves them
back to the root. The `_template/` scaffold is gone; what a profile contains is described in that
same README rather than mocked up as empty files.

### The annotation markers no longer name a retired market profile

Every line in the standard carries a marker saying where it came from. Two of them named the Vietnam
track, which stopped owning this material when the base-layer work moved into the standard. They are
swept, in prose only, across all twelve modules:

| Was | Is |
| :--- | :--- |
| `[OPIN-VN extension to API; OPIN schema reused]` | `[added]` |
| `[OPIN-VN normalisation]` | `[normalisation]` |
| `[OPIN-VN]` | `[added]` |

A normalisation is a spelling correction rather than an addition, so it did not collapse into
`[added]` the way the earlier note said it would. Two markers describing two different things is the
point of having markers.

Prose that named `OPIN-VN` as the actor was rewritten the same way, because a market profile was
being credited with decisions that belong to the standard. The annotation table in
[`conventions.md`](conventions.md) is rewritten to match.

**The three wire identifiers were deliberately not touched.** `api.opin-vn.{tld}`, `opin-vn.admin`
and `opin-vn.developer` still carry the retired name. They travel on the wire, so correcting them
breaks every caller, and they stay held for a major version alongside the misspelled field names.
See [`SCOPE.md`](SCOPE.md). Nothing in this sweep changes what an implementation sends.

### Defects are this standard's to fix, not a report to a dormant upstream

The documents still described themselves as a downstream track filing defects with a live Open
Insurance Initiative that would publish an OPIN v1.1 to fix them. That was the position before this
repository took the standard forward, and it survived the earlier moves because it is a change of
meaning rather than of naming. Thirty-five places, across all twelve modules,
[`conventions.md`](conventions.md), [`SCOPE.md`](SCOPE.md) and both concern catalogues.

`Upstream candidate` became `Change proposal` on twenty-eight concerns, because there is no upstream
to send them to any more: this is it. `A future v1.0 of this track must resolve this upstream with
the OPIN initiative` now names the inherited defect number and states the position this version
takes. `An OPIN v1.1 should add an explicit linkage field to Claim` is closed outright by the
`policyNumber` uniqueness rule.
[`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md) no longer says it is
published so the initiative can resolve the defects at source, and its duplicated intro paragraph is
gone.

None of this makes the initiative disappear from the record. It is still credited as the origin in
[`README.md`](README.md) and [`project/inherited/`](project/inherited/), and
[`project/VERSIONING.md`](project/VERSIONING.md) still leaves 1.3 and 1.4 unclaimed so it has room
if it publishes again. What changed is that nothing here waits on that happening.

Two things were corrected while in there. The cyber liability module said a future publication
should consider breach-notification timelines, which contradicted the scope boundary, where those
are listed as out of scope by design and never to be closed; the module now matches.

### `KNOWN-GAPS.md` became `SCOPE.md`

Same material, and a different job. The file held four things: how the entities link, what is
settled by convention, what is held for a major version, and what the standard will never cover.
Three of those four are decisions rather than gaps, and filing them under a name that says "gaps"
described the standard as a set of holes with some prose around them. `SCOPE.md` says what the file
does, which is draw the boundary. The page publishes at `/docs/standard/scope`, and the old path
redirects. And
`conventions.md` said an OPIN v1.1 should declare authentication, the error model, pagination and
idempotency, in the same file that declares all four of them.

### Inherited

The data standard v1.2.1 and API specification v1.0 are kept unmodified in
[`project/inherited/`](project/inherited/), with the twenty catalogued defects at
[`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md) and fourteen
against the API specification at
[`project/inherited/concerns-api-v1.0.md`](project/inherited/concerns-api-v1.0.md). The API list
was previously an appendix inside the API document and had no home of its own.

### Structural defects closed

**Claim-to-coverage and receipt-to-obligation linkage.** The inherited schema declared no foreign
key from `Claim` to a coverage and none from `Receipt` to the policy or claim that produced it, and
inferred both from correlation. This version declares the constraint the model was already
assuming: `policyNumber` is globally unique across the namespace, it travels in the payload where
the linkage matters, and `/claim` and `/receipt` expose `?policyNumber=` and `?claimNumber=` as
collection filters. Closes inherited defects 12 and 13.

**Trade credit lifecycle.** `tradeCreditCoverage` was the only coverage entity of the eight carrying
no `inceptionDate`, `expiryDate`, `status`, or premium, brokerage and endorsement fields, which left
it describing a credit limit with no policy around it. All thirteen are supplied, with the names,
types and value sets they carry everywhere else. Closes inherited defect 7.

**Lifecycle transitions are normative.** The state diagrams in each module's API page were
previously a conservative reading with no normative force, so two conformant implementations could
disagree about whether a transition was legal. They are now normative. A transition they do not draw
is not a transition an implementation may make.

**The naming rule is settled.** The data model and the API design took opposite positions on whether
misspelled inherited field names are corrected or preserved. The rule is now stated once: the API
governs anything that travels on the wire, and the data model governs what a field means. A
`[normalisation]` records what the field should have been called and never changes what an
implementation sends. See [`conventions.md`](conventions.md).

The remaining inherited defects are compatibility breaks held for the next major version, so that an
implementer absorbs them together rather than one at a time. They are in [`SCOPE.md`](SCOPE.md).

---

# Vietnam profile

## vn-v0.2 (2026-04-29, amended)

Published as OPIN-VN v0.2. Everything in it that was base-standard work has since moved to the
standard at v1.5.0. What remains is what is genuinely Vietnamese: money, addresses, identity
documents, statutory claim handling and personal data.

Original v0.2 scope, for the record: endpoint completion to close the gap between OPIN's full data
standard and its partial v1.0 API specification. No new entity schemas and no vendor product
behaviour. Twenty structural concerns in OPIN catalogued. Known gaps published.

Deliberately not carried forward from the earlier draft: coverage extensions that would have pulled
distribution, commission and product-specific behaviour into the profile.
