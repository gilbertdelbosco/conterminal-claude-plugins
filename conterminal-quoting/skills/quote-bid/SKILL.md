---
name: quote-bid
description: Price a customer's multi-lane drayage bid or rate sheet with the Conterminal connector and fill in their sheet. Use when the user shares a spreadsheet, CSV or list of lanes, mentions a bid, RFP or rate request, or asks for rates on many lanes at once.
---

# Price a bid

A bid is one quote request with many lanes. Use the Conterminal connector's tools in this order. Nothing here contacts the customer.

1. **Find the request.** You need its `opportunityId`. Find an existing bid with `find_quote_requests` by its OP- or Q- reference, or by `customer` with `lanes: "multi"`; a pasted Conterminal quote link also carries the ID. For a new request, get one from `create_quote_draft_from_request` with the customer's email text (`sourceType` set to `other` for text pasted into Claude).
2. **Read their sheet yourself.** Make one row per lane in the customer's own words: lane number, port or terminal, town with state and ZIP, equipment, volume and its period, and the move type only when the sheet states it. Never invent a port, town, equipment or volume that a row doesn't state. Note each lane's sheet row and which column letter holds each field, including the column the rate goes back in.
3. **Put the lanes on the request** with `import_quote_bid_lanes`: `rows` with each lane's `sheetRow`, and `source` with `fileName`, `sheetName` and `columns`. Use `mode: "replace"` on a draft nobody has priced. Use `mode: "append"` with `expectedVersion` to add up to 20 lanes to an existing bid. A request holds at most 100 lanes. Show the lanes in `needsAttention`.
4. **Review the table** with `get_quote_bid_table`. Show the lanes that are held or need attention, and ask which lanes the carrier won't bid. A priced lane carries its internal margin (`marginCents`, `marginBasisPoints`, `revenuePerHourCents`); `marginComplete` false means some costs are unknown. The margin never goes into the customer's sheet.
5. **Trim, then set shared defaults,** with one `save_quote_batch` call at the table's `version`. Mark each lane the carrier won't bid with `settings.noBid: true` (`false` restores it); a no-bid lane stays on the bid and is never priced or sent. In `defaults`, set what the lanes share: the planning terminal, the empty pickup or return location, the truck profile and the service date.
6. **Price five lanes at a time** with `calculate_quote_batch`: up to five `rowKeys`, the latest `expectedVersion` and a new `sourceKey` per call. A failed lane doesn't discard the others; retry only the failed lanes. Lanes placed by town or ZIP stay preliminary.
7. **Agree prices** lane by lane with the user, as integer cents per container. Save them with `save_quote_batch_rates`, using each lane's expected revision, total and pricing fingerprint from the latest result, and its `context.customerTermsMaterialId` as `expectedCustomerTermsMaterialId`. Report any lane that didn't save.
8. **Approve** the lanes the user reviewed. Preview the combined card first with `preview_quote_rate_card` (its `batch` with `expectedVersion` and the `rowKeys`). Then call `approve_quote_batch_rates` with each lane's `rowKey`, `revisionId`, `revisionToken` and `laneVersion`, and `preliminary: true`. This is internal approval, not customer acceptance and not permission to send.
9. **Answer the customer's sheet.** `export_quote_bid_rows` returns each lane's result by the customer's lane number and sheet row: a rate, `no_bid` or `pending`, with the column the rate goes in. A pending lane has no figure; never fill in an old one. An unapproved rate is a draft, so say so. Fill in their sheet for the user, or prepare the rate card and reply with `prepare_quote_batch_response` (approved lanes only) and give the user its `reviewUrl`.

**Never send anything, and never say that something was sent.** Filling in a sheet doesn't send it.

## Rules

- `get_quote_batch` has the terms, route evidence and the exact save and approval guards. Save and calculate results already carry fresh guards, so you don't need to read the batch again after them.
- Each lane has its own `opportunityId`, which works with the single-quote tools, for example to compare one lane with saved rates.
- Price lanes with `calculate_quote_batch`, not one `propose_rate_card` call on the whole bid: that prices every lane in a single request and can run out of time on a large bid.
- Money is integer cents per container, never an order total.
- A tool error means the data is unavailable. Retry a transient error once. If the connector asks you to sign in, ask the user to reconnect Conterminal, then retry.
- Quoting needs a Manager or Admin role in the connected carrier workspace.
