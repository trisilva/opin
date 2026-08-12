# Module 02: Products and Catalog

A product is what an insurer sells, as distinct from what a particular customer bought. "Comprehensive
motor, annual, in VND, under wording MT-2026" is a product. The policy one customer holds against it
is a coverage record, and it lives in one of the eight coverage modules.

That separation is why this module exists. Everything true of the product regardless of who buys it
sits here once, and every coverage record points at it rather than repeating it.

| Page | What it holds |
| :--- | :--- |
| [`data-model.md`](data-model.md) | The entities, fields, enumerated values and relationships |
| [`api.md`](api.md) | The endpoints, the flow that registers a product, the lifecycle and the error paths |

## How the model is shaped

**`Product`** is the sellable thing. It names its line of business, how it is priced and charged,
what currency it is in, and which policy wording governs it.

**`productCatalog`** is a fixed list of 65 lines of business, from motor and marine through medical,
engineering, life, cyber and pet. A product names exactly one. The list is the standard's answer to
"what kind of insurance is this", and it is broader than the eight coverage types the standard
models in detail, so a catalogue code exists for lines that have no coverage module.

**`policyWording`** is the legal document that says what is actually covered. Everything else in the
standard is structured data about the contract, and this is the contract.

Several small enumerations also live here because more than one module needs them:
`policyStatus` (in force, cancelled, lapsed, extended), `endorsementType`, `paymentMethod`,
`premiumPaymentFrequency` and `currency`. If a coverage module refers to one of these, this is where
it is defined.

## What to watch

**`premiumPaymentFrequency` is declared twice, incompatibly.** On `Product` it is typed as an
integer, and it also references an enumeration of the same name. Treat the enumeration as
authoritative and send a code from it. An implementation that reads the integer typing literally
will accept values the enumeration does not define.

**`policyWording` has no document URL.** It carries a name, a version and an effective date, which
is enough to identify which wording governed a policy on a given date. It does not carry the text.
Store the document yourself and reference it from your side.

**`productCatalog` has no parametric or index-linked codes.** Parametric weather cover and
index-linked microinsurance map only loosely onto existing entries such as 36 (purchase protection)
or 30 (personal accident). If you are building either, pick a code deliberately and record why.

**A product has no draft or deprecated state.** It exists or it does not. Whether a product is
open for new business is an operational question, so carry it as an extension field.

## Market profiles

This module is market-neutral. Nothing it defines varies by market, so no profile constrains it.
