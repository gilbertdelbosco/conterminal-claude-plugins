---
name: rate-engine
description: Read the carrier's Conterminal quote settings, and change default accessorial charges or one customer's fuel-surcharge mode. Use when the user asks what their quote settings, minimums, per-mile rates, truck profiles or extra charges are, or wants to change a default charge or how a customer's quotes show fuel.
---

# Quote settings

## Read

- `get_quote_options` for a service date: truck profiles, the minimum charge, included miles, per-mile rates, planning MPG and the diesel benchmark, extra charges, yards and terminals. If there is no default yard, say that calculations leave out yard positioning.
- `get_carrier_rate_defaults`: the Billing default accessorial cards, their effective dates, typed rates and each card's `version`.

## Change default accessorial charges

Only an Administrator can, and only when the user explicitly asks.

1. Read `get_carrier_rate_defaults` and show each line that will change, with the old and new value.
2. After the user confirms, call `save_carrier_rate_defaults` with the `rateCardId`, that read's version as `expectedVersion`, a new `sourceKey`, and `changes` that patch only the intended fields. Edit a line by its existing `itemId`; a new line needs `chargeTypeCode`, `displayName`, `unit` and `pricingRule`.
3. Keep first-day flat charges separate from additional-day `per_unit_after_included` rates. Variable fuel stays `manual_quote`, with no invented amount.
4. Show the saved rates and the new version. These are the same records Billing uses. Saved quotes and customer-specific terms don't change.

## Change one customer's fuel-surcharge mode

This sets how that customer's future quotes show fuel.

1. Read `get_customer_quote_pricing` for one of the customer's quotes.
2. Confirm the new mode with the user: consolidated, `fsc` with its index and bands, or `flat_fsc` with a flat amount per container in cents.
3. Call `save_customer_quote_pricing` with `expectedDirectoryCompanyId`, `expectedVersion`, a new `sourceKey` and the `policy`. Existing quote prices, approvals and invoices don't change. To reprice a quote on the new mode, use quote-compare.

## Not available over the connector

The base rate (minimum, included miles, per-mile tiers), fuel and toll policy, MPG, truck profiles, costs and margin, geographic multipliers, fixed lane rates, the default yard and lane profiles can't be changed here. Tell the user to change them in Conterminal under Settings > Quotes, and never describe such a change as made.
