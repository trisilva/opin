# OPIN-VN: the Vietnam country adaptation track

**Version 0.2. Working draft.**

OPIN-VN closes the operational gaps that OPIN leaves when it meets one national market. It adds no
entity schemas, encodes no vendor product behaviour, and redefines nothing OPIN already settles.

## Documents

| Document | What it covers |
| :--- | :--- |
| [`opin-vn-data-schema.md`](opin-vn-data-schema.md) | The OPIN v1.2.1 data standard rendered as one internally consistent entity-relationship reference across twelve modules. Every entity, field and enum traces to an OPIN sheet. |
| [`opin-vn-api-design.md`](opin-vn-api-design.md) | The API surface, module by module. Each entry is marked as inherited from OPIN or added by this track. |
| [`KNOWN-GAPS.md`](KNOWN-GAPS.md) | What v0.2 does not settle, and which of those are OPIN's to fix. Read before committing to an implementation. |

## What the track adds

OPIN's API specification is silent on the questions that decide whether an integration is
operable. This track answers them, once, for Vietnam.

- **Authentication.** OAuth 2.0 bearer tokens, with two scopes mirroring OPIN's two declared tags.
- **Error model.** RFC 7807 Problem Details, so a failure means the same thing to every caller.
- **Pagination.** Cursor-based, stable under concurrent writes, with OPIN's `skip` and `limit` form
  accepted as a fallback.
- **Idempotency.** So a retried request is safe rather than a second policy.
- **Item-level operations** on records OPIN describes only at collection level.
- **Lifecycle endpoints** for the full life of a record rather than its creation alone.
- **Claim-to-coverage linkage**, by convention at v0.2. See [`KNOWN-GAPS.md`](KNOWN-GAPS.md).

Only the motor module carries endpoints in OPIN itself. Everywhere else the endpoints here are
extensions that mirror OPIN's motor pattern, annotated inline.

## Modules

Twelve, matching the OPIN data standard: core parties and entities, products and catalog, motor,
travel, term life, property, cyber liability, business interruption, trade credit, pet, claims,
premium and receipts.

Trade credit is present for vocabulary completeness and is not usable as a wire contract at v0.2,
for reasons that belong to OPIN rather than to this track.

## Annotation conventions

Both documents mark every line with its provenance.

| Marker | Meaning |
| :--- | :--- |
| `[OPIN]` | Exists in OPIN verbatim. Kept as-is. |
| `[OPIN-VN extension to API; OPIN schema reused]` | The schema is OPIN's, the endpoint is not. This track adds it. |
| `[OPIN-VN normalisation]` | A non-controversial spelling or casing fix, with the original noted. |
| `[OPIN concern]` | A structural inconsistency or omission in OPIN itself, flagged for upstream. |

The markers exist so a reader can tell at a glance what they are adopting from the standard and
what they are adopting from Trisilva's reading of it. Anything marked `[OPIN-VN]` is a place where
disagreement is legitimate and an issue is welcome.

## Status and what comes next

v0.2 is the working baseline and is being built against today. The next patch tightens the
`policyNumber` and `claimNumber` request-body conventions into normative schema notes, which closes
the most fragile thing on this page without waiting on upstream.

A v1.0 requires normative lifecycle transitions and resolution of the OPIN-side concerns. Neither
has a date, and this track will not claim one it does not have.
