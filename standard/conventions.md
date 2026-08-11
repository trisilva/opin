# API conventions

These apply to every module. Nothing that only one market needs belongs here, because a market
requirement goes in a [market profile](../markets/README.md) instead.


These apply across every module. Each line is annotated with whether OPIN declares it or whether OPIN-VN adds it for operational viability.

- **Base URL**: `https://api.opin-vn.{tld}/v1` where `{tld}` matches the deploying tenant. `[OPIN-VN]`: OPIN does not specify a base URL convention beyond the SwaggerHub virtserver host.
- **Versioning**: URL-prefix versioning (`/v1`, `/v2`). `[OPIN]`: OPIN's `info.version` is `1.0`; URL prefix is implicit in the SwaggerHub server entry.
- **Authentication**: OAuth 2.0 Bearer tokens. `[OPIN-VN]`: OPIN's spec is silent on auth. OPIN-VN adopts OAuth 2.0 with two role scopes that mirror OPIN's two declared tags:
  - `opin-vn.admin` (full write, mirrors OPIN's `admins` tag).
  - `opin-vn.developer` (read-only, mirrors OPIN's `developers` tag).
  No third operator scope is introduced. State-changing operations live under `opin-vn.admin` only.
- **Content-Type**: `application/json` for all requests and responses. `[OPIN]`: OPIN declares `application/json` on every request and response body.
- **Error model**: RFC 7807 Problem Details. `[OPIN-VN]`: OPIN does not declare an error model beyond bare `400`, `409` descriptions on its four endpoints.
- **Pagination**: cursor-based via `?cursor=` and `Link` header on collection GETs. `[OPIN-VN]`: OPIN declares `skip` and `limit` query parameters on its four GET endpoints. OPIN-VN supersedes this with cursor-based pagination, which scales better and is stable under concurrent writes. The OPIN `skip`/`limit` form remains accepted as a fallback for backward compatibility.
- **Idempotency**: `Idempotency-Key` header on POST and PUT. `[OPIN-VN]`: OPIN does not declare idempotency.
- **Action endpoints**: state transitions are exposed as `POST /resource/{id}:action` (colon-action style). `[OPIN-VN]`: OPIN declares no action endpoints. OPIN-VN uses the colon-action form for endorsements, cancellations, renewals, settlements, reopenings, and refunds because they map to OPIN data-standard concepts (`endorsementType` enum, `policyStatus` transitions, `claimStatus` transitions, `receiptType` reversals) but are not pure CRUD.

`[OPIN concern]`: four of the seven conventions above (auth, error model, pagination, idempotency) are entirely absent from OPIN v1.0. An OPIN v1.1 publication should declare them, otherwise every implementer will diverge.
