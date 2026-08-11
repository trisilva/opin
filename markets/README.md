# Market profiles

A market profile carries what one national market requires and other markets do not. That is all it
carries, and the constraint is what makes a profile worth reading.

## The test

Before anything enters a profile, one question decides where it belongs.

**Would another market need this too?**

If yes, it belongs in [`standard/`](../standard/), even when a single market surfaced it.
Authentication, error handling, pagination, idempotency and record lifecycle are the same problem
in every market. They arrived through work on Vietnam and they are not Vietnamese.

If no, it belongs in a profile. Statutory claim response times, national identity document types,
required regulatory reporting fields, local currency and address handling, data residency
obligations. These differ market by market because the law differs market by market.

This test has already been applied once and it moved most of the Vietnam profile into the standard.
See the v1.5.0 changelog.

## What a profile may not do

**A profile never contradicts the standard.** It constrains and it adds. If a profile needs the
standard to be different, that is an issue against the standard, not a local override.

**A profile never carries vendor product behaviour.** Distribution mechanics, commission ledgers,
workflow sub-states and operational service-level measurement sit above every layer here.

**A profile never redefines a field.** It may require a field the standard makes optional, restrict
an enum to the values the market permits, or add a field the market's regulator requires. It may
not change what an existing field means.

## What a profile contains

| File | What it holds |
| :--- | :--- |
| `README.md` | The market, the regulation that drives the profile, the standard version it targets |
| `profile.md` | The constraints and additions, one section each, with the legal or operational source for every one |
| `KNOWN-GAPS.md` | What this version does not settle |
| `CHANGELOG.md` | Versions of this profile |

[`_template/`](_template/) carries the scaffold.

## Profiles

| Market | Profile | Version | Targets | Status |
| :--- | :--- | :--- | :--- | :--- |
| Vietnam | `vn` | v0.2 | OPIN v1.5.0-draft | Working draft, and nearly empty. See [`vn/`](vn/) |

## Starting a profile for a new market

Open an issue naming the market and what makes it different. The useful version of that issue cites
the regulation, because a profile with no statutory driver usually turns out to be base-standard work
that has not been recognised yet.

Nobody has to be a maintainer to propose a profile. Someone does have to know the market's regulation
well enough to be accountable for what the profile claims, and that person is named in the profile's
README.
