# Module 03: Motor

Motor coverage, the vehicle and the driver. This is the only module the inherited API specification covers with endpoints, so it is the pattern every other module's surface is built from.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- Read this module first even if you are not building motor, because every other module's endpoints mirror the shape declared here.
- The inherited specification declares eight endpoints, all collection-level. Item-level retrieval and update are absent and are added here.
- `Vehicle` bundles more than a hundred fields across registration, manufacturer specification, telematics, driver assistance, condition and consent. It is operationally large for one record. Inherited defect 10.

## Market profiles

None yet.

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
