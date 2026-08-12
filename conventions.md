# Conventions

Read this once. Everything here applies to every module. Nothing that only one market needs belongs
here, because a market requirement goes in a [market profile](project/markets/README.md) instead.

## API conventions

Each line is annotated with whether OPIN declares it or whether it is added here.

- **Base URL**: `https://api.opin-vn.{tld}/v1` where `{tld}` matches the deploying tenant. `[added]`: OPIN does not specify a base URL convention beyond the SwaggerHub virtserver host.
- **Versioning**: URL-prefix versioning (`/v1`, `/v2`). `[OPIN]`: OPIN's `info.version` is `1.0`; URL prefix is implicit in the SwaggerHub server entry.
- **Authentication**: OAuth 2.0 Bearer tokens. `[added]`: OPIN's spec is silent on auth. OAuth 2.0 is adopted with two role scopes that mirror OPIN's two declared tags:
  - `opin-vn.admin` (full write, mirrors OPIN's `admins` tag).
  - `opin-vn.developer` (read-only, mirrors OPIN's `developers` tag).
  No third operator scope is introduced. State-changing operations live under `opin-vn.admin` only.
- **Content-Type**: `application/json` for all requests and responses. `[OPIN]`: OPIN declares `application/json` on every request and response body.
- **Error model**: RFC 7807 Problem Details. `[added]`: OPIN does not declare an error model beyond bare `400`, `409` descriptions on its four endpoints.
- **Pagination**: cursor-based via `?cursor=` and `Link` header on collection GETs. `[added]`: OPIN declares `skip` and `limit` query parameters on its four GET endpoints. This supersedes them with cursor-based pagination, which scales better and is stable under concurrent writes. The OPIN `skip`/`limit` form remains accepted as a fallback for backward compatibility.
- **Idempotency**: `Idempotency-Key` header on POST and PUT. `[added]`: OPIN does not declare idempotency.
- **Action endpoints**: state transitions are exposed as `POST /resource/{id}:action` (colon-action style). `[added]`: OPIN declares no action endpoints. The colon-action form is used for endorsements, cancellations, renewals, settlements, reopenings, and refunds because they map to OPIN data-standard concepts (`endorsementType` enum, `policyStatus` transitions, `claimStatus` transitions, `receiptType` reversals) but are not pure CRUD.

`[OPIN concern]`: four of the conventions above (auth, error model, pagination, idempotency) are
entirely absent from OPIN v1.0. They are declared above, because
without them every implementer diverges.

## Extensions

An implementation may carry fields the standard does not define. Two rules make that safe.

An extension field never reuses a name the standard defines, and never changes what a defined field
means. An extension is additive or it is a fork.

A caller that receives a field it does not recognise ignores it rather than failing. This is
required in both directions and it is what lets the standard add optional fields in a minor
version.

## Annotation

Every line in a module carries its provenance, so a reader can tell what came from the published
standard and what was added on top of it.

| Marker | Meaning |
| :--- | :--- |
| `[OPIN]` | In the published standard verbatim. Unchanged. |
| `[added]` | Not in the published standard. Added here. Where a schema is inherited and only the endpoint over it is new, the surrounding note says so. |
| `[normalisation]` | A spelling or casing correction to an inherited name, with the original noted beside it. |
| `[OPIN concern]` | A defect in the inherited standard. Catalogued in [`project/inherited/`](project/inherited/). |

A marker describes provenance and nothing else. It says where a thing came from, not how settled it
is. [`SCOPE.md`](SCOPE.md) holds the scope boundary, and
[`project/GOVERNANCE.md`](project/GOVERNANCE.md) is how to argue against any of it.

`[normalisation]` marks a correction in the data model only, and it never changes what your
implementation sends. The API preserves inherited spellings verbatim because they are already on the
wire. The naming rule below says which one governs where.

## Where the surface came from

The inherited API specification covered one module of the twelve. This version carries endpoints for
all of them, and the provenance markers let a reader see which is which.

- **Motor** is complete: entity schemas and eight collection-level endpoints. Item-level retrieval
  and update are absent and are added here.
- **Cross-cutting modules** (1, 2, 11, 12) have entity schemas and no endpoints. The endpoints here
  mirror the motor pattern over those schemas.
- **The other coverage modules** (4 to 10) have entity schemas and no endpoints, on the same
  footing.

No new entity schema is introduced anywhere in this version. The schemas are inherited and reused
unmodified.

## The naming rule

**The API governs anything that travels on the wire. The data model governs what a field means.**

Where the inherited standard misspelled a name, the API preserves the misspelling and the data model
records the corrected name beside it as a `[normalisation]`. So the normalisation tells you what the
field was meant to be called, and the API tells you what to send.

The rule protects working integrations. These names are already on the wire everywhere the inherited
standard was implemented, and correcting them would break those callers to make a document tidier.
The corrections ship together in one major version instead. The reasoning is in
[`SCOPE.md`](SCOPE.md).

## Sources

- OPIN Data Standard v1.2.1 (XLSX), published by the Open Insurance Initiative:
  https://docs.google.com/spreadsheets/d/1Y0Gk_LpTvTNEfoDMdIxeD7juv3E8FKcbE3mHUJNV5JY
- OPIN API Specification v1.0 (resolved JSON):
  https://github.com/The-Open-Insurance-Initiative/API-spec/blob/main/Open-Insurance-io-Open_Insurance_API-1.0-resolved.json
- Mermaid syntax reference, for the diagrams throughout: https://mermaid.js.org/intro/
