# OPIN country adaptation tracks

**OPIN is not ours.** The [Open Insurance Initiative](https://openinsurance.io) publishes the OPIN
Data Standard and the OPIN API Specification. Trisilva adopts both as they are.

This repository holds the **country adaptation tracks** that Trisilva authors on top of OPIN, one
per market, and publishes openly. It is not a fork of OPIN. It redefines nothing OPIN already
settles. It contains no Trisilva product specification.

## Why a country track exists

OPIN gives the industry a shared vocabulary. It does not, on its own, give a single national market
an operable contract. Between the two sits a set of questions every integrator in that market
answers privately, slightly differently, and then maintains forever: how a caller authenticates,
what an error means, how a long list is paged, whether a repeated request is safe to retry, how a
claim references the coverage it belongs to.

A country adaptation track settles those questions once, in the open, for one market. That is all
it does. Where OPIN is silent on an operational matter, the track closes the gap. Where OPIN has
already decided, the track defers.

## The three tiers

| Tier | Layer | Owner | In this repository |
| :--- | :--- | :--- | :--- |
| 1 | OPIN Data Standard v1.2.1 and OPIN API Design v1.0 | Open Insurance Initiative | No. Adopted upstream, unmodified. |
| 2 | Country adaptation track, one per market | Trisilva, published open | Yes. This is the repository. |
| 3 | Vendor platform extensions | The vendor building on tiers 1 and 2 | No, by design. |

Tier 3 is where a vendor's own extensions live, and it is deliberately outside this repository.
Trisilva's Tier 3 platform is InsureFlow, and none of it is specified here. If something in a track
below only makes sense because of a Trisilva product, that is a defect in the track. Please report
it.

## Tracks

| Market | Track | Version | Status | Source |
| :--- | :--- | :--- | :--- | :--- |
| Vietnam | OPIN-VN | v0.2 | Working draft | [`vn/`](vn/) |

Vietnam is the first track. Others follow the markets Trisilva enters, on the same terms.

## What "working draft" means here

v0.2 is complete enough to build against and is being built against today. It is not a ratified
standard, and this repository does not pretend otherwise.

Two things are true of it at once. Every entity, field and enum traces back to a sheet in the OPIN
v1.2.1 data standard, so the vocabulary is stable. And its lifecycle semantics are implied rather
than declared normatively, one module carries structural holes inherited from OPIN, two foreign-key
conventions are carried in request bodies rather than in the schema, and the two documents disagree
with each other about whether OPIN's misspelled field names are corrected or preserved.

All of that is written down in [`vn/KNOWN-GAPS.md`](vn/KNOWN-GAPS.md) rather than left for a reader
to discover. A track that hid its gaps would be worth less than one that names them, because the
gaps are exactly what an implementer needs before committing.

## Upstream concerns

Twenty structural issues in OPIN itself surfaced while authoring the Vietnam track: missing foreign
keys, a coverage entity without lifecycle fields, inconsistencies between the data standard and the
API specification, and a run of field-name typos that are stable on the wire and should not be
silently corrected.

They are listed in [`upstream/opin-concerns.md`](upstream/opin-concerns.md) so the Open Insurance
Initiative can resolve them at source. Carrying local patches for them indefinitely would fork OPIN
by accretion, which is the outcome this repository exists to avoid.

## Using a track

The tracks are documents, not a service. They describe a wire contract that any party can
implement. Trisilva runs its own implementation and so can anyone else, and an implementation built
to a track belongs to whoever built it.

## Comment and correction

Open an issue. Corrections against OPIN source, gaps in a track, and questions about a market that
has no track yet are all in scope. Proposals that would pull vendor product behaviour into a track
are not, and will be closed with a pointer to this README.

## Licence

The OPIN Data Standard is published under MPL 2.0 and the OPIN API Specification under Apache 2.0.
The tracks here are derived from both, so this repository is licensed under **MPL 2.0** to match
the more restrictive upstream terms. See [`LICENSE`](LICENSE).

Trademarks and product names appearing in any track are used for identification only and belong to
their respective owners.

---

Maintained by [Trisilva](https://trisilva.ai). Trisilva builds insurance operations software in
Southeast Asia and authors these tracks for the markets it works in. The tracks are open because a
shared model is worth what it is shared by, and a carrier that builds to one should still own what
it built afterwards.
