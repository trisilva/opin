# Conformance

What it means to say an implementation conforms, and what it does not mean.

## The honest position first

**Conformance is not yet testable.** There is no validator, no test suite and no machine-readable
schema in this repository. An implementer can read the standard and follow it, and nobody can check
the result mechanically.

That is a gap and it is named here rather than left for a reader to discover. Until the artefacts
below exist, any claim that something "conforms to OPIN" is a statement about intent, not a result.
Treat it accordingly, including when it comes from us.

## What conformance will mean

Three levels, because implementations differ in what they can honestly claim.

**Level 1, vocabulary.** Entities, fields, enumerated values and types match the data model.
Anything sent under an OPIN name means what the standard says it means. An implementation can reach
this level with no HTTP surface at all, which matters for a carrier exposing OPIN shapes through an
existing gateway.

**Level 2, surface.** Level 1, plus the API design: resource paths, methods, status codes, the
error model, pagination and idempotency behave as specified. This is the level at which two
implementations can actually talk to each other.

**Level 3, market.** Level 2, plus a named market profile in full. Stated as the level plus the
profile: "Level 3, vn-v0.2".

An implementation states its level and the standard version: "Level 2, OPIN v1.5.0-draft". A claim
with no version attached is not a conformance claim.

## What conformance never covers

Conformance is about the contract on the wire. It says nothing about whether an implementation is
correct, secure, performant, available or lawful in the market it runs in. It is not a
certification, there is no certifying body here, and nobody audits a claim made under it.

## Extensions

An implementation may carry fields the standard does not define. Two rules make that safe.

An extension field never reuses a name the standard defines, and never changes what a defined field
means. An extension is additive or it is a fork.

A caller that receives a field it does not recognise ignores it rather than failing. This is
required in both directions and it is what lets the standard add optional fields in a minor
version.

## Getting to testable

In order, because each depends on the one before it.

1. **JSON Schema 2020-12** for every entity in the data model. This makes Level 1 checkable.
2. **An OpenAPI 3.1 document** for the surface. This makes Level 2 checkable.
3. **Worked examples** for each module, valid against the schemas, which are also the first test
   fixtures.
4. **A validator** that runs the examples against the schemas in CI, so the standard cannot
   contradict itself without the build saying so.

Until step 1 lands, the prose is normative and there is nothing else. When the artefacts arrive
they are derived from the prose and clearly labelled as derived, and the prose stays normative
until a version says otherwise. Two normative descriptions of one standard is the mistake that
produced the inherited divergence between the data standard and the API specification, and it is
not repeated here.
