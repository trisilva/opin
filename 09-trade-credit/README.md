# Module 09: Trade Credit

Trade credit coverage, the debtor and the credit limit. Covers whole turnover, key accounts, single buyer and transactional structures.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- `tradeCreditCoverage` carries the same policy lifecycle as every other coverage type: `inceptionDate`, `expiryDate`, `status`, and the premium, brokerage and endorsement fields. The inherited standard omitted all thirteen from this module alone, and this version supplies them. Closes inherited defect 7.
- `tradeCreditTpe` (the sheet) and `creditLimitUtiilized` (the field) are both misspelled. Both are stable on the wire, so they are reported and left alone until a major version. Inherited defects 5 and 6.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.

## Related pages

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The scope boundary and the decisions that apply everywhere are in
[`../SCOPE.md`](../SCOPE.md).
