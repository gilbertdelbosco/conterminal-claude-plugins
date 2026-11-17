---
name: quote-request
description: Price one drayage lane and prepare its rate card with the Conterminal connector. Use when the user pastes a customer's quote request or email, says "quote this", asks what a single move would cost, or wants a rate card or reply for one quote. For a sheet or list of many lanes, use quote-bid instead.
---

# Quote a request

Use the Conterminal connector's tools in this order. Each write waits for the user's agreement. Nothing in this workflow sends anything to the customer.

1. **Price it.** Call `quote_from_request` with the pasted text as `requestText`, plus `subject` and `from` when the user gave them. For a request already in Conterminal, find it with `find_quote_requests` and pass its `opportunityId` instead. It parses the lane, resolves terminals and the truck profile, and prices through the carrier's own engine. Nothing is saved. If it says the request has several lanes, switch to quote-bid.
2. **Show the result.** Lead with `calculation.comparisonTable`. Then show:
   - the status. Only `priced` is a finished total; `review_required`, `address_action_required` and `needs_input` are explained in `references/pricing-rules.md`;
   - the charges per move, and the total for the stated quantity;
   - the review reasons and the planning assumptions, such as a planning date or an empty return modeled back to the pickup terminal;
   - the economics in `calculation.operatingAnalysis.economics`: margin on the known cost lines and revenue per hour. When `complete` is false, say which costs are missing; it is not a full P&L.

   Ask for anything the result says is missing, then call `quote_from_request` again with the answer.
3. **Agree a price.** Show the engine subtotal before suggesting a different customer price. Record the price the user agrees as integer cents per move.
4. **Create the draft** when there is no `opportunityId` yet: `create_quote_draft_from_request` with the same `requestText` and a stable `sourceKey` (reuse it on a retry). Always set `sourceType`: `other` for text pasted into Claude, or `gmail` or `outlook` when the user copied it from that mailbox. The tool acknowledges the request to the user only, never to the requester.
5. **Save the decision.** Read `get_quote_context` for the quote to get its revision, effective total, pricing fingerprint and `customerTermsMaterialId`. Then call `save_quote_decisions` with the first three as guards, `customerRateCents`, a short `reason`, a new `sourceKey`, and the `pricingScenario` and `shipment` from step 1. Agreed services such as load-in, storage or overweight go in `customerTerms.customCharges`; whenever you change terms, pass the `customerTermsMaterialId` you read as `expectedCustomerTermsMaterialId` (null when the quote has no saved terms).
6. **Preview the rate card** with `preview_quote_rate_card`. Every catalog item in `card.accessorialReview` needs a treatment (included, conditional, not applicable or waived). Fill gaps with another `save_quote_decisions`, then preview again. Show the user the card.
7. **Approve** only after the user has reviewed that exact preview: `approve_quote_rate_card` with its `outputHash` as `expectedOutputHash` and `preliminary: true`. This is internal approval, not customer acceptance and not permission to send.
8. **Prepare the reply** with `prepare_quote_response`. Pass the approved preview's `outputHash` from step 7 as `expectedOutputHash`: it is required once the saved terms carry `customerPricing` (for example after `propose_rate_card`), and otherwise it guards against a card that changed since the user reviewed it. It saves an unsent reply and the PDF on the quote. Show the subject, recipients, body and attachment, and give the user the Conterminal link (the draft's `draftUrl` or the preview's `previewUrl`) with the reply's `reviewInstruction` so they can review and send it themselves.

**Never send anything, and never say that something was sent.** No tool here emails the customer.

## A request already in Conterminal

Find it with `find_quote_requests`: pass `reference` for an OP- or Q- reference, or the customer's name in `customer`, with `status` or a due date to narrow it. A pasted Conterminal link also carries the `opportunityId`. Each result's `readWith` names the next tool: `get_quote_context` for one lane, or `get_quote_bid_table` for a bid (switch to quote-bid). `get_quote_context` also returns the saved P&L in `economics`.

When the user opens an existing single-lane request that has no price yet, `propose_rate_card` does what the app's rate card builder does: it prices the request, applies a suggested price and service rows as the saved terms, and returns the guards for the next save. It writes a quote revision. Show its card, rationale, margin and flags, then continue at step 6.

## Rules

- Money is integer US cents per move. Never multiply a saved rate by the quantity.
- Every save uses the guards from the latest read or save result. If a save reports a stale revision or fingerprint, read `get_quote_context` again and show what changed before you retry.
- A tool error means the data is unavailable, never zero or empty. Retry a transient error once. A refusal whose message says what to change, such as a quote that changed since it was read or a missing recipient, is not transient: show the message and act on it. If the connector asks you to sign in, ask the user to reconnect Conterminal, then retry the same request.
- Quoting needs a Manager or Admin role in the connected carrier workspace.
