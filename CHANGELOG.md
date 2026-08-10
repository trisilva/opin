# Changelog

Versions are per track, not per repository. A tag reads `vn-v0.2`.

## OPIN-VN

### v0.2 (2026-04-29)

The working baseline, and the first version published openly.

- Scope set to endpoint completion only: close the gap between OPIN's full data standard and its
  partial v1.0 API specification. No new entity schemas, no vendor product behaviour.
- Data schema renders OPIN v1.2.1 as one internally consistent entity-relationship reference across
  twelve modules, with every entity, field and enum traced to an OPIN sheet.
- API design adds authentication, an error model, cursor pagination, idempotency, item-level
  operations and lifecycle endpoints, each annotated as an addition rather than as OPIN.
- Twenty structural concerns in OPIN itself catalogued and published at
  `upstream/opin-concerns.md`.
- Known gaps published. See `vn/KNOWN-GAPS.md`.

Known at publication and unresolved: the data schema and the API design take opposite positions on
whether OPIN's misspelled field names are corrected or preserved. The API design governs the wire
until this is settled.

Deliberately not carried forward from the earlier draft: coverage extensions that would have pulled
distribution, commission and product-specific behaviour into the track. Those belong above it.
