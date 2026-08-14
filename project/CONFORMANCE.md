# Conformance

What it means to say an implementation conforms, and what it does not mean.

## Three levels

Implementations differ in what they can honestly claim, so conformance is graded rather than
binary.

**Level 1, vocabulary.** Entities, fields, enumerated values and types match the data model.
Anything sent under an OPIN name means what the standard says it means. An implementation
reaches this level with no HTTP surface at all, which matters for a carrier exposing OPIN
shapes through an existing gateway.

**Level 2, surface.** Level 1, plus the API design: resource paths, methods, status codes, the
error model, pagination and idempotency behave as specified. This is the level at which two
implementations can talk to each other.

**Level 3, market.** Level 2, plus a named market profile in full. Stated as the level plus the
profile, so "Level 3, vn-v0.2".

An implementation states its level and the standard version, so "Level 2, OPIN v1.5.0-draft". A
claim with no version attached is not a conformance claim, because the thing being claimed
against moves.

## What conformance never covers

Conformance is about the contract on the wire. It says nothing about whether an implementation
is correct, secure, performant, available or lawful in the market it runs in. It is not a
certification, there is no certifying body here, and nobody audits a claim made under it.

That boundary is deliberate. A standard that certified quality would need an authority to
enforce it, and an authority is what turns an open standard into a gate somebody owns.

## Extensions

An implementation may carry fields the standard does not define, and two rules make that safe.
They are wire behaviour rather than process, so they live with the rest of the wire behaviour in
[`conventions.md`](../conventions.md).

## The conformance artefacts

Four artefacts will make each level checkable, and each depends on the one before it.

| | Artefact | What it will make checkable |
| :--- | :--- | :--- |
| 1 | JSON Schema 2020-12 for every entity in the data model | Level 1, vocabulary |
| 2 | An OpenAPI 3.1 document for the surface | Level 2, surface |
| 3 | Worked examples per module, valid against the schemas | Both, and they are the first test fixtures |
| 4 | A validator running the examples against the schemas in CI | The standard against itself |

The fourth is the one that will matter most to a reader deciding whether to build on this. It
will mean the standard cannot contradict itself without the build saying so, which is precisely the
failure that produced the inherited divergence between the data standard and the API
specification.

**The prose is normative and the artefacts are derived from it.** Not the other way round, and
never both at once. Two normative descriptions of one standard is how the inherited documents
came to disagree about `termLifeType` and `termLifeRiders`, and repeating that mistake with a
schema alongside the prose would reproduce it exactly. Each artefact will be generated from the
normative text and labelled as derived, so a disagreement between them is a defect in the
generator rather than an open question for an implementer to adjudicate.
