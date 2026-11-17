---
name: rate-engine
description: Read and change the carrier's Conterminal quote settings, including the base rate, fuel and toll treatment, truck, costs, commission and default accessorial charges, and set one customer's fuel-surcharge mode. Use when the user asks what their quote settings, minimums, per-mile rates, truck profiles or extra charges are, or wants to change one of them or how a customer's quotes show fuel.
---

# Quote settings

## Read

- `get_rate_engine_settings`: the carrier's saved settings, with one version per section in `sectionVersions`, and the sections the connector can and can't change. Costs are marked saved or defaults. `canApply` is false unless the user can change settings, which takes an Administrator.
- `get_quote_options` for a service date: what the quote calculator uses, including truck profiles, the minimum charge, included miles, per-mile rates, planning MPG and the diesel benchmark, extra charges, yards and terminals. If there is no default yard, say that calculations leave out yard positioning.
- `get_carrier_rate_defaults`: the Billing default accessorial cards, their effective dates, typed rates and each card's `version`.

## Change quote settings

Only an Administrator can, and only when the user explicitly asks. This covers:

- company and website quote settings, quote expiry and service states;
- fuel-surcharge and toll treatment;
- the routing truck;
- the base rate (minimum charge, included miles, per-mile rate and tiers), with start and end dates;
- internal costs, commission, and commission on single extra charges;
- default accessorial cards, now or from a future date, and which extra charges on today's card are offered.

1. Read `get_rate_engine_settings`. If `capabilities.setupRequired` is true, the first setup happens in Conterminal under Settings > Quotes. If `canApply` is false, say that an Administrator has to make the change.
2. Preview with `preview_rate_engine_changes`: the read's `sectionVersions` as `expectedVersions`, a short `reason`, and a `patch` with only the fields the user asked to change. Fields left out keep their saved values. Add `sampleScenarios` only when the user wants routes priced before and after; each route spends routing and fuel lookups.
3. Show every changed field with its before and after value, how dated rules change (a base rate edited in place, started on a date, or ended), and the linehaul examples. Nothing is saved yet.
4. After the user confirms, call `apply_rate_engine_changes` with the preview's `reviewToken` and a new `sourceKey`; reuse that `sourceKey` only to retry the same change. A `reviewToken` lasts 30 minutes. If it expired, or a section changed after the preview, read and preview again.
5. Show what was saved. Saved quote prices, customer terms, issued files, website publication and the partner API don't change, and nothing is sent.

## Change default accessorial charges

Only an Administrator can, and only when the user explicitly asks.

1. Read `get_carrier_rate_defaults` and show each line that will change, with the old and new value.
2. After the user confirms, call `save_carrier_rate_defaults` with the `rateCardId`, that read's version as `expectedVersion`, a new `sourceKey`, and `changes` that patch only the intended fields. Edit a line by its existing `itemId`; a new line needs `chargeTypeCode`, `displayName`, `unit` and `pricingRule`.
3. Keep first-day flat charges separate from additional-day `per_unit_after_included` rates. Variable fuel stays `manual_quote`, with no invented amount.
4. Show the saved rates and the new version. These are the same records Billing uses. Saved quotes and customer-specific terms don't change.

A card that starts on a future date, or a card changed together with other settings, goes through the preview and apply steps above, with `scheduledAccessorials` or `accessorials` in the patch.

## Change one customer's fuel-surcharge mode

This sets how that customer's future quotes show fuel.

1. Read `get_customer_quote_pricing` for one of the customer's quotes.
2. Confirm the new mode with the user: consolidated, `fsc` with its index and bands, or `flat_fsc` with a flat amount per container in cents.
3. Call `save_customer_quote_pricing` with `expectedDirectoryCompanyId`, `expectedVersion`, a new `sourceKey` and the `policy`. Existing quote prices, approvals and invoices don't change. To reprice a quote on the new mode, use quote-compare.

## Not available over the connector

Geographic rules, website publication and the partner API, fixed lane rates, the default yard, lane profiles and the first setup can't be changed here. Tell the user to change them in Conterminal under Settings > Quotes, and never describe such a change as made.
