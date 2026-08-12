# Module 10: Pet

Pet coverage and the insured animal.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- `petKind` covers five kinds of animal and `petBreed` enumerates dog breeds only. Cats, rabbits, birds and exotics have no breed values. Inherited defect 11.
- `waitingPeriod` appears on this coverage with no entry or exit semantics defined anywhere.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.

## Related pages

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The scope boundary and the decisions that apply everywhere are in
[`../SCOPE.md`](../SCOPE.md).
