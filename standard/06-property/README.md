# Module 06: Property

Property coverage and the insured property, across buildings and contents.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- The roof construction enum is spelled `rootConstruction` in the inherited API and `roofConstruction` in the inherited data standard. Inherited defect 2.
- `numberOfBedrooms` appears twice on the property sheet. One of the two is meant to be `numberOfBathrooms`, per its own description. Inherited defect 4.
- `claimsOccurrence` is a Boolean on the coverage entity and a two-value enum on its own sheet. Pick one before you build. Inherited defect 3.

## Market profiles

None yet.

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
