# Module 02: Products and Catalog

The product record and the catalogue it sits in. A product instantiates exactly one of the eight coverage types.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- `Product.premiumPaymentFrequency` is typed as an integer and also references an enum of the same name. One of the two has to go. Inherited defect 20.
- The inherited API specification publishes schemas here but no endpoints. The endpoints in `api.md` are added by mirroring the motor pattern.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.

## Related pages

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The scope boundary and the decisions that apply everywhere are in
[`../SCOPE.md`](../SCOPE.md).
