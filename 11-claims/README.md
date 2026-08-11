# Module 11: Claims

The claim, from first notification of loss through to settlement. One claim entity serves every coverage type.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- **`Claim` carries no foreign key to the coverage or the policy it belongs to.** The linkage runs through `policyNumber` by convention rather than by schema, which means two conformant implementations can fail to interoperate. Inherited defect 12, and the single most important thing on this page.
- `Receipt` has the same problem one module over. See [`../12-premium-receipts/`](../12-premium-receipts/).
- `Claim.lossCause` references a generic peril set without declaring which peril enum applies for a given coverage type. Inherited defect 14.
- Operational claim sub-states such as triage, awaiting documents and fraud review are out of scope by design. A claim's business status belongs to the standard; its position in one operator's workflow does not.

## Market profiles

Vietnam's statutory claim handling under Decree 67/2023/ND-CP lands on this module. It is not
written, and it is gated on Vietnamese counsel. See [`../../markets/vn/`](../project/markets/vn/).

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
