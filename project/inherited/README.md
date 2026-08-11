# Inherited material

What was received from the Open Insurance Initiative, kept so that it can be told apart from what
was changed.

## What is here

| Source | Version | Published | Licence |
| :--- | :--- | :--- | :--- |
| OPIN Data Standard | 1.2.1 | 8 December 2021 | MPL 2.0 |
| OPIN API Specification | 1.0 | 16 May 2022 | Apache 2.0 |

Both remain available from the Open Insurance Initiative at
[openinsurance.io/standards](https://openinsurance.io/standards/) and, for the API specification,
at [github.com/The-Open-Insurance-Initiative/API-spec](https://github.com/The-Open-Insurance-Initiative/API-spec).
Neither has been revised since publication.

Two catalogues of what is wrong with them, found while building against them.

| Catalogue | Against | Count |
| :--- | :--- | :--- |
| [`concerns-v1.2.1.md`](concerns-v1.2.1.md) | The data standard v1.2.1 | 20 |
| [`concerns-api-v1.0.md`](concerns-api-v1.0.md) | The API specification v1.0 | 14 |

## Why this folder exists

A continuation is only trustworthy if a reader can check it. Keeping the inherited material intact
means anyone can diff the current standard against what was published and see exactly what changed,
without taking the changelog's word for it.

The binary sources, the data standard workbook and the resolved API JSON, are not committed here.
They are the initiative's published artefacts and are linked above rather than copied, so a reader
gets them from the origin.

## The twenty

They are the work list. Each is a defect that every implementer currently carries a private patch
for, or lives with.

Four of them are structural and matter most: a claim carries no foreign key to the coverage or
policy it belongs to, a receipt carries no reference back to either, trade credit is missing the
lifecycle fields every other coverage type has, and the data standard and API specification
disagree with each other about two term life enums.

The rest are field name errors, type inconsistencies and enum problems. They are individually small
and collectively the reason this standard carries normalisation notes at all.

Fixing them is what this continuation is for. None is closed as of v1.5.0-draft.
