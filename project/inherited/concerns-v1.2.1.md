# Concerns against the inherited data standard

These are inconsistencies and gaps in the OPIN standard itself, surfaced while building against it.
They are this standard's to resolve, each closed in a version of this standard rather than reported
onward, and this list is the work list for doing that.

Scope note: this list excludes choices an implementer makes (whether to introduce a Policy aggregate
root, whether to give Coverage an abstract base) and excludes domain areas outside insurance proper
(event streams, telematics ingestion, distribution channels, parametric oracles, commission
ledgers). Those are downstream concerns, not OPIN defects.

Sources: OPIN Data Standard v1.2.1 (the XLSX, treated as authoritative) and OPIN API Specification
v1.0 (the resolved JSON). Where the two disagree, the disagreement is itself listed below.

A note on the typos. Field names that are wrong but stable on the wire are recorded, not corrected.
Silently fixing a name would break compatibility with every implementation already built to OPIN, so
those corrections wait for a major version and ship together. See
[`VERSIONING.md`](../VERSIONING.md).

1. **`termLifeType` and `termLifeRiders` XLSX vs API divergence.** XLSX `termLifeType` has 4 values; API has 3. XLSX `termLifeRiders` has 4 values; API has 5 (the API adds `Convertible term`, which conceptually belongs in `termLifeType`). Resolve which side is authoritative and align both.

2. **`rootConstruction` vs `roofConstruction` API typo.** API JSON spells the roof construction enum as `rootConstruction`; XLSX correctly uses `roofConstruction`. Fix in API.

3. **`propertyCoverage.claimsOccurrence` Boolean vs enum.** Field is typed Boolean on `propertyCoverage`, but a standalone two-value `claimsOccurrence` enum sheet exists. Choose one form across the standard.

4. **`property` sheet duplicate field name.** `numberOfBedrooms` listed twice; one occurrence is intended to be `numberOfBathrooms` per its description.

5. **`tradeCreditTpe` sheet name typo.** Should be `tradeCreditType`.

6. **`tradeCreditCoverage.creditLimitUtiilized` field name typo.** Should be `creditLimitUtilized`.

7. **`tradeCreditCoverage` missing standard policy lifecycle fields.** Unlike every other coverage entity, `tradeCreditCoverage` does not carry `inceptionDate`, `expiryDate`, `status`, premium fields, brokerage fields, or endorsement fields. Add them.

8. **`business.cyberCoverageCategories` reference points to wrong sheet.** Field references the `cyberLiabilityCoverage` sheet rather than the `cyberCoverageCategories` enum sheet.

9. **`businessInterruptionCoverage.closure by public authority` field name contains spaces.** Normalise to a single-token field name.

10. **`Vehicle` entity bundles 100+ fields across registration, OEM specs, telematics, ADAS, condition, and consent.** Operationally large for a single record. Consider whether OPIN should split Vehicle into static and dynamic sub-entities.

11. **`petKind` covers 5 kinds but `petBreed` enumerates dogs only.** Cat, rabbit, bird, and exotic-pet breeds unmodelled. Either restrict breed to dogs explicitly, add per-kind breed enumerations, or specify a free-text fallback.

12. **`Claim` lacks an explicit foreign key to Coverage or Policy.** Inferred from `claimNumber`/`policyNumber` correlation. Add explicit reference.

13. **`Receipt` lacks `policyRef` and `claimRef` linkage fields.** Reconciliation requires these foreign keys. Add explicit references.

14. **`Claim.lossCause` references generic `perils` without specifying which peril enum.** Polymorphic resolution by coverage type is implied but not declared. Make explicit.

15. **`ClaimsBordereau.GrosslLossReserve` field name typo.** Should be `GrossLossReserve`.

16. **`policyWording` is a single-property entity (just `name`) without version control.** Compliance traceability for wording documents is not supported. Add `version`, `effectiveDate`, and document URL fields.

17. **`Personal.gender` enum is binary plus `other` (m / f / o).** Some jurisdictions require additional values or a different non-binary representation. Review for jurisdictional fit, including Vietnam.

18. **`Commercial` lacks a `legalForm` reference to the `legalEntity` enum.** Trade Credit's `Debtor` carries `legalForm`; the universal `Commercial` party does not. Add for consistency.

19. **Pervasive low-impact spelling errors across sheets.** `Number/fFloat` (lower-f) appears across multiple coverage sheets; `engnitionOn`/`engnitionOff`, `logitude`, `laneDepartureWarnning`, `decelrationRate`, `countryOfRegisteration`, `volcanic erruption`, `unkown or hit and run`, `Multi-media laibilities`, `Theft of intectual property`, `claimsOcuurence`, `medicalConditon`, `occcupation`, `redundnt`, `tracktor`, `sprinler leakage`, `Strom` (storm). Codes are stable; descriptions and field names should be cleaned up in a v1.2.2 patch.

20. **`Product.premiumPaymentFrequency` typed `Number/integer` but references the `premiumPaymentFrequency` enum.** Type-vs-reference inconsistency; choose one.

These are all genuine OPIN issues. Closing them in the standard is preferable to carrying `[normalisation]` patches indefinitely, which is what every implementer had to do while there was nowhere to send them. They are filed here as the work list; please raise or correct any of them as an issue.
