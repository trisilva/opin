# Module 05: Term Life

Term life coverage, the life insured, and the riders attached to the policy.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- The inherited data standard and the inherited API specification disagree here. `termLifeType` carries four values in one and three in the other, and `termLifeRiders` carries four in one and five in the other. Inherited defect 1, and the clearest case of the two documents diverging.
- Until that is closed, the data standard governs what a field means and the API design governs what travels on the wire. See [`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).

## Market profiles

None yet.

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
