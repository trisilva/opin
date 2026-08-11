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
[`KNOWN-GAPS.md`](KNOWN-GAPS.md).

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
See [`KNOWN-GAPS.md`](KNOWN-GAPS.md). Nothing in this sweep changes what an implementation sends.

### Inherited

The data standard v1.2.1 and API specification v1.0 are kept unmodified in
[`project/inherited/`](project/inherited/), with the twenty catalogued defects at
[`project/inherited/concerns-v1.2.1.md`](project/inherited/concerns-v1.2.1.md) and fourteen
against the API specification at
[`project/inherited/concerns-api-v1.0.md`](project/inherited/concerns-api-v1.0.md). The API list
was previously an appendix inside the API document and had no home of its own.

### Not yet done

None of the twenty inherited defects is closed in this version. The version reorganises what
existed and establishes where a fix goes; it does not fix anything yet.

Two things carried forward unresolved from the previous publication. The data model and the API
design take opposite positions on whether misspelled field names are corrected or preserved, and
the API design governs the wire until that is settled. Lifecycle transitions are walked
conservatively in the API design and are not normative, so two conformant implementations can still
disagree about whether a transition is legal.

Both are in [`KNOWN-GAPS.md`](KNOWN-GAPS.md).

---

# Vietnam profile

## vn-v0.2 (2026-04-29, amended)

Published as OPIN-VN v0.2. Everything in it that was base-standard work has since moved to the
standard at v1.5.0. What remains is the scope of the profile, which is not yet written.

Original v0.2 scope, for the record: endpoint completion to close the gap between OPIN's full data
standard and its partial v1.0 API specification. No new entity schemas and no vendor product
behaviour. Twenty structural concerns in OPIN catalogued. Known gaps published.

Deliberately not carried forward from the earlier draft: coverage extensions that would have pulled
distribution, commission and product-specific behaviour into the profile.
