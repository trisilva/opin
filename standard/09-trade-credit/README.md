# Module 09: Trade Credit

Trade credit coverage and the debtor. Present so the vocabulary is complete, and not usable as a wire contract at this version.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- **This module is structurally incomplete.** Unlike every other coverage entity, `tradeCreditCoverage` carries no `inceptionDate`, no `expiryDate`, no `status`, and no premium, brokerage or endorsement fields. Inherited defect 7. Do not build a policy lifecycle on it.
- `tradeCreditTpe` (the sheet) and `creditLimitUtiilized` (the field) are both misspelled. Both are stable on the wire, so they are reported and left alone until a major version. Inherited defects 5 and 6.

## Market profiles

None yet.

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
