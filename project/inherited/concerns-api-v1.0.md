# Concerns against the OPIN API Specification v1.0

Fourteen inconsistencies and omissions in the inherited API specification, catalogued while building
against it. They are the API-side companion to [`concerns-v1.2.1.md`](concerns-v1.2.1.md), which
catalogues twenty defects in the inherited data standard.

These affect every implementation of the inherited specification, not only this one.


The following are inconsistencies or omissions in OPIN v1.0 that an OPIN v1.1 publication should resolve upstream with the Open Insurance Initiative. They are listed here so this standard does not paper over them.

1. **Auth model is undeclared.** OPIN exposes two role tags (`admins`, `developers`) but no auth scheme. Every implementer will diverge until OPIN declares OAuth 2.0 (or equivalent) with normative scope names.
2. **Error model is undeclared.** OPIN's four endpoints declare bare `400`, `409` descriptions with no body schema. RFC 7807 should be normative.
3. **Pagination is inconsistent.** OPIN uses `skip`/`limit` query parameters on its four GET endpoints; this does not scale and is unstable under concurrent writes. Cursor-based pagination should be normative.
4. **Idempotency is undeclared.** No `Idempotency-Key` header convention exists. POST endpoints across the spec are therefore non-idempotent in the wire contract.
5. **Item-level retrieval is missing for the four motor resources.** OPIN declares only `POST /resource` and `GET /resource` (collection search). `GET /resource/{id}` and `PUT /resource/{id}` are absent across all four motor endpoints. The data standard implies they should exist.
6. **Endpoint coverage is non-uniform across the data standard.** Motor has CRUD, every other coverage type and every cross-cutting entity has none. The data standard documents 30+ entities; the API documents 4 endpoints on 4 resources. An OPIN v1.1 should publish endpoints for every entity in the data standard.
7. **Claim-to-coverage linkage is implicit.** The Claim schema lacks an explicit foreign-key field for the policy or coverage being claimed against. Implementations must associate via `policyNumber` carried in the payload.
8. **Receipt-to-policy linkage is missing.** The Receipt schema has no foreign-key fields for the policy or claim it settles. Reconciliation is impossible without one.
9. **Lifecycle gaps.** OPIN declares `policyStatus` (in force, cancelled, lapsed, extended) and `claimStatus` (open, closed, reopened), but does not specify which operations transition between them, nor lifecycles for Party, Product, or Receipt at all.
10. **`waitingPeriod` is a field with no operational signal.** It appears on petCoverage and tradeCreditCoverage but with no entry/exit semantics defined.
11. **Underwriting is absent.** Quoting, underwriting decision, and decline are not modelled. Term life and trade credit in particular need this.
12. **Endorsement is an enum without operations.** `endorsementType` enumerates seven values but OPIN declares no endpoint for applying an endorsement, and no rules for which fields each endorsement type may mutate.
13. **Schema naming inconsistency.** OPIN mixes camelCase (`motorCoverage`, `policyWording`) and PascalCase (`Vehicle`, `Driver`, `Claim`, `Personal`, `Commercial`, `Product`, `InsuranceEntity`, `Beneficiary`, `Receipt`, `PremiumBordereau`, `ClaimsBordereau`) for entity names. This standard uses camelCase paths uniformly to match REST convention.
14. **Field-name typos.** OPIN ships `tradeCreditTpe` (missing `y`), `GrosslLossReserve` (double `l`, mixed case), `IndemnityPeriod` (capital `I`). This standard preserves them verbatim for wire compatibility but flags them here.

These questions affect every OPIN implementation, not only this one. They are the work list for reaching v1.0.
