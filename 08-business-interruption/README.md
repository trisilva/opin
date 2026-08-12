# Module 08: Business Interruption

Business interruption coverage, which is usually written alongside a property policy rather than on its own.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- One field name contains spaces: `closure by public authority`. It needs a single-token name before it can travel. Inherited defect 9.
- The primary flow in `api.md` binds this coverage alongside property, which is how it is normally sold.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.

## Related pages

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The scope boundary and the decisions that apply everywhere are in
[`../SCOPE.md`](../SCOPE.md).
