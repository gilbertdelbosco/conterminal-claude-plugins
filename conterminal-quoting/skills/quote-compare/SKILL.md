---
name: quote-compare
description: Compare a Conterminal quote with saved rates, tariffs or a known route, and price it on the customer's fuel-surcharge terms. Use when the user asks what they charged before on a lane, wants comparable or historical rates, wants to benchmark a price, or asks how a price splits into linehaul and fuel surcharge.
---

# Compare a quote

You need the quote's `opportunityId`: find it with `find_quote_requests` by its OP- or Q- reference or the customer, or take it from quote-request, quote-bid or a Conterminal quote link the user pastes.

1. **Find comparables** with `find_quote_comparables`: the `opportunityId`, or a `query` with the destination city and state and the `moveType`. Show each result's date, status and source, its match reasons and its material differences. A tariff is not proof that a customer accepted it or of what it included, and a draft is not an accepted quote. Establish equipment, waiting, chassis, fuel and toll terms before you treat one as a benchmark. Results are bounded; pass `cursor` for more.
2. **Compare with one** using `compare_quote_to_saved_rate`. Take `expectedRevisionId`, `expectedPricingFingerprint` and `expectedCustomerTermsMaterialId` from `get_quote_context`. Then either pass the comparable's `benchmarkSource` (`tariff` or `generated_quote`) and `benchmarkId`, or pass `operator_route` with a typed `benchmarkRoute` and the known per-move `customerRateCents`. Set `sameServiceTerms` to whether the service terms really match. The comparison is saved on the quote so it reopens in the app; the price doesn't change. Show miles, minutes, fuel, tolls and an equivalent rate only where the evidence supports them.
3. **Price on the customer's fuel terms.** Read `get_customer_quote_pricing` for the customer's mode and `version`: consolidated, `fsc` (linehaul plus the weekly regional EIA fuel surcharge) or `flat_fsc` (linehaul plus a flat amount per container). Then call `preview_customer_quote_pricing` with `expectedDirectoryCompanyId`, `expectedPolicyVersion` and `amountCents`. `amountCents` is the linehaul for `fsc` and `flat_fsc`, and the complete total for consolidated. It returns linehaul, fuel surcharge and total in cents with the exact EIA week and price. A missing, stale or wrong-region price needs review; never fall back to national data. Pass `flatFuelSurchargeCents` only when the user agreed a flat surcharge for this quote.
4. **Use the result** by saving it as in quote-request: `save_quote_decisions` with `customerRateCents` set to the returned `customerPricing.totalCents`, and `customerPricing` (and `fuelSurcharge` when present) in `customerTerms`.

Nothing here sends anything, and only step 4 changes a price.
