# Module 12: Premium and Receipts

Premium, receipts, and the premium and claims bordereaux that report to reinsurers.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- **`Receipt` carries no `policyRef` and no `claimRef`.** A payment cannot be reconciled back to the policy or claim that produced it from the schema alone. Inherited defect 13.
- `ClaimsBordereau.GrosslLossReserve` is misspelled and stable on the wire, so it is reported and left alone until a major version. Inherited defect 15.
- The inherited API specification publishes schemas here but no endpoints. The endpoints in `api.md` are added by mirroring the motor pattern.

## Market profiles

Vietnamese dong is unit-denominated and does not carry the minor units the standard's money handling
assumes, which has a wire consequence. It is not written yet. See
[`../../markets/vn/`](../project/markets/vn/).

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
