# The standard

Everything every market shares. What one market does not share lives in a
[market profile](../markets/README.md) instead.

**Current version: v1.5.0-draft.** A draft, being built against, not ratified.

## The twelve modules

One directory per module. Each holds the data model and the API surface for that module side by
side, plus a README naming what to check before implementing it.

| | Module | What it covers |
| :--- | :--- | :--- |
| 1 | [Core Parties and Entities](01-core-parties/) | The insurer, the broker, the personal and commercial policyholder, beneficiaries, addresses |
| 2 | [Products and Catalog](02-products-catalog/) | The product record and the catalogue it sits in |
| 3 | [Motor](03-motor/) | Motor coverage, vehicle, driver. The only module the inherited API covers |
| 4 | [Travel](04-travel/) | Travel coverage and the traveller |
| 5 | [Term Life](05-term-life/) | Term life coverage, the life insured, riders |
| 6 | [Property](06-property/) | Property coverage, buildings and contents |
| 7 | [Cyber Liability](07-cyber-liability/) | Cyber liability coverage and the insured business |
| 8 | [Business Interruption](08-business-interruption/) | Business interruption, usually written alongside property |
| 9 | [Trade Credit](09-trade-credit/) | Trade credit coverage and the debtor. Not usable as a wire contract |
| 10 | [Pet](10-pet/) | Pet coverage and the insured animal |
| 11 | [Claims](11-claims/) | The claim, from first notification through settlement |
| 12 | [Premium and Receipts](12-premium-receipts/) | Premium, receipts, and the bordereaux that report to reinsurers |

Modules 1, 2, 11 and 12 are cross-cutting and every coverage type uses them. Modules 3 to 10 are the
eight coverage lines.

## Above the modules

| Page | What it holds |
| :--- | :--- |
| [`conventions.md`](conventions.md) | Base URL, versioning, authentication, content type, error model, pagination, idempotency, action endpoints. Applies to every module. |
| [`cross-module.md`](cross-module.md) | How the entities relate across modules, and the universal claim submission flow. |
| [`KNOWN-GAPS.md`](KNOWN-GAPS.md) | What this version does not settle. Read it before committing to an implementation. |
| [`inherited/`](inherited/) | What was received from the Open Insurance Initiative, kept so it can be told apart from what was changed. |

## How to read this

**Building one coverage line.** Open that module's directory. Its README says what to check first,
then read `data-model.md` for what the fields mean and `api.md` for the surface. Read
[`conventions.md`](conventions.md) once, because it applies everywhere. If you are new to the
standard, read [Motor](03-motor/) first even if you are not building motor, because every other
module's endpoints mirror the shape declared there.

**Building in one market.** Read the module, then the [market profile](../markets/README.md) for
your market. A profile constrains and adds; it never changes what a field means.

**Fixing a defect.** [`inherited/concerns-v1.2.1.md`](inherited/concerns-v1.2.1.md) catalogues
twenty defects in the inherited data standard and
[`inherited/concerns-api-v1.0.md`](inherited/concerns-api-v1.0.md) catalogues fourteen in the
inherited API specification. They are the work list.

## Coverage is not uniform across the modules

The inherited API specification is three-tiered relative to the inherited data standard, and the
unevenness is inherited rather than designed.

- **Motor** is complete: entity schemas and eight collection-level endpoints. Item-level retrieval
  and update are absent and are added here.
- **Cross-cutting modules** (1, 2, 11, 12) have entity schemas and no endpoints. The endpoints here
  mirror the motor pattern over those schemas.
- **The other coverage modules** (4 to 10) have entity schemas and no endpoints, on the same
  footing.

No new entity schema is introduced anywhere in this version. The schemas are inherited and reused
unmodified.

## Annotation

Every line carries its provenance, so a reader can tell what came from the published standard and
what was added on top of it.

| Marker | Meaning |
| :--- | :--- |
| `[OPIN]` | In the published standard verbatim. Unchanged. |
| `[OPIN-VN extension to API; OPIN schema reused]` | The schema is inherited, the endpoint is not. Legacy marker from when this material sat in the Vietnam track; being replaced by `[added]`. |
| `[OPIN-VN normalisation]` | A spelling or casing correction, with the original noted. Legacy marker, same reason. |
| `[OPIN concern]` | A defect in the inherited standard. Catalogued in [`inherited/`](inherited/). |

The two legacy markers name a market profile that no longer owns this material. They are being
replaced and the inconsistency is visible rather than quietly swept, because a marker sweep across
2,700 lines is the kind of change that should be reviewable on its own.

## One caution about corrections

The data model and the API surface currently take opposite positions on misspelled field names. The
data model renders the corrected name and notes the original. The API preserves the original
verbatim, on the grounds that it is already on the wire in every existing implementation.

Until that is resolved, **the API governs anything that travels on the wire**, and a correction in
the data model reads as a note about what the field should have been called rather than as what
your implementation should send. This is a defect rather than a design, and it is in
[`KNOWN-GAPS.md`](KNOWN-GAPS.md).

## Sources

- OPIN Data Standard v1.2.1 (XLSX), published by the Open Insurance Initiative:
  https://docs.google.com/spreadsheets/d/1Y0Gk_LpTvTNEfoDMdIxeD7juv3E8FKcbE3mHUJNV5JY
- OPIN API Specification v1.0 (resolved JSON):
  https://github.com/The-Open-Insurance-Initiative/API-spec/blob/main/Open-Insurance-io-Open_Insurance_API-1.0-resolved.json
- Mermaid syntax reference, for the diagrams throughout: https://mermaid.js.org/intro/
