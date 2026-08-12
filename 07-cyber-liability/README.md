# Module 07: Cyber Liability

Cyber liability coverage and the insured business behind it.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- `business.cyberCoverageCategories` points at the `cyberLiabilityCoverage` sheet rather than the `cyberCoverageCategories` enum sheet it names. Inherited defect 8.
- Breach notification timelines are deliberately out of scope. They sit above the standard, in whatever platform an implementer builds.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.

## Related pages

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The scope boundary and the decisions that apply everywhere are in
[`../SCOPE.md`](../SCOPE.md).
