# Module 01: Core Parties and Entities

The insurer, the broker, the personal and commercial policyholder, beneficiaries and addresses. Every other module refers back to these.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | Entities, fields, enumerated values and relationships. What a thing is. |
| [`api.md`](api.md) | Resources, endpoints, primary flow, lifecycle and routing. How a caller asks for it. |

## Before you implement

- `Personal.gender` carries three values (m, f, o). Some jurisdictions require more or a different representation. Inherited defect 17.
- `Commercial` has no `legalForm` reference, although trade credit's `Debtor` carries one. Inherited defect 18.
- `policyWording` carries a name and nothing else, with no version and no effective date, so a wording document cannot be traced for compliance. Inherited defect 16.
- The inherited API specification publishes schemas here but no endpoints. The endpoints in `api.md` are added by mirroring the motor pattern.

## Market profiles

Vietnam's identity document types and its personal data obligations both land on this module.
Neither is written yet. See [`../../markets/vn/`](../../markets/vn/).

## Where this sits

Part of the standard at v1.5.0-draft. Conventions that apply to every module are in
[`../conventions.md`](../conventions.md). Relationships that span modules are in
[`../cross-module.md`](../cross-module.md). The full gap list is
[`../KNOWN-GAPS.md`](../KNOWN-GAPS.md).
