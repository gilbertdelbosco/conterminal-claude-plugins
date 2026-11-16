# Pricing rules for Conterminal quotes

## Money and quantity

- Amounts are integer US cents. `customerRateCents` and every charge are per move, that is per container.
- Quantity applies only to the selling total. Never multiply a per-move rate by it when you save.
- Show the per-move charges and the total for identical moves separately. Different lanes are separate quotes.

## Statuses

- `priced`: a finished total.
- `review_required`: a partial subtotal (`knownPerMoveSubtotalCents`). `reviewReasons` says what is missing, such as a requested service with no configured charge.
- `address_action_required`: the engine corrected an address or could not place it. The user must accept the correction before it is used.
- `needs_input`: a stop, the customer or another fact is missing. The result lists which.
- A failed or missing value is unavailable, never zero.

## Places and routes

- Terminals can be given by name or code, such as Maher, APM Elizabeth or PNCT. The server resolves them.
- A town or ZIP without a street is a planning area. The price stays preliminary until the address is known.
- An import with no stated empty return is modeled back to the pickup terminal and labeled a planning assumption. Pass `emptyReturn: "none"` only when the user says the stops are complete as given.
- Omitted yards use the carrier's default yard. When no default yard is configured, yard positioning is left out; say so.
- Fuel and tolls are estimates, not paid expenses. Route choices are measured options, not guaranteed worst cases.

## Dates

- An unknown service date uses a planning date, today in New York, labeled provisional.
- Use `serviceDateBasis: "confirmed"` only for a date the customer confirmed.

## Guards

- `save_quote_decisions` needs `expectedRevisionId`, `expectedEffectiveTotalCents` and `expectedPricingFingerprint` from the latest `get_quote_context` or save result. Whenever it changes terms, it also needs that read's `customerTermsMaterialId` as `expectedCustomerTermsMaterialId`, or null when the quote has no saved terms.
- `sourceKey` makes a retry safe. Reuse it for an identical retry; use a new key for changed content.
- A changed price or terms makes an earlier rate-card approval stale. Preview and approve again.

## Economics

- `calculation.operatingAnalysis.economics` gives the margin on known cost lines (`marginCents`, `marginBasisPoints`) and `revenuePerHourCents` for a fresh calculation. `belowFloor` is true when a complete margin is under the carrier's margin floor.
- `complete: false` means some cost lines, such as truck or fixed cost, are missing, so the margin is not a full P&L.
- A saved quote read with `get_quote_context` returns `economics`: the P&L at the saved price on the saved route, as the app's P&L shows it, with cost `lines`, `marginCents`, `marginBasisPoints`, `revenuePerHourCents`, `complete` and `driverType` (`byDriver` holds each driver's margin when the lane keeps both). Each `get_quote_bid_table` row carries its lane's margin the same way.
- Unknown actual expenses stay unknown: a missing cost line is not zero.
