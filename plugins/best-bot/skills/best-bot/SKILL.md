---
name: best-bot
description: >
  This skill should be used when the user wants to watch Best Buy purchases for
  price drops and reclaim the difference. Triggers on "watch this for a price
  drop", "price match my Best Buy order", "run my price watch", "check my
  watches", "did anything drop", "Best Bot", "set up Best Bot", or when the
  user drops a Best Buy order confirmation into their watch folder. Handles
  setup, the recurring sweep across every active purchase, writing the claim
  script, and retiring items when their return window closes.
metadata:
  version: "2.0.0"
  author: "CR8-OS"
---

# Best Bot

Best Buy refunds the difference if the price drops during the buyer's return
window, but only on request, and only in time. Best Bot is the part that
notices. It reads order confirmations out of one folder, keeps a ledger of
everything still inside its window, checks them all on a schedule, and hands
over a claim when a qualifying drop lands.

One ledger, many purchases, one sweep. Adding a watch means saving a
confirmation to a folder.

## Hard rules

Violating these produces a user who calls a retailer and makes a false claim.

1. **Never report a price that was not observed in this run.** Every price must
   come from a page loaded during this run, with its URL recorded. Never
   estimate, never reuse a price from an earlier run or from the ledger, never
   infer one from a search snippet. If the page did not load, say the check
   failed.
2. **Browsing is read-only and public.** Permitted: product and search URLs,
   typing into a retailer's product-search box, scrolling, dismissing overlays.
   Forbidden: signing in or using a signed-in session on any retailer; opening
   support, chat, contact, returns, account or order-history pages; submitting
   any form other than a product search; anything that adds to cart, starts a
   checkout, or opens a conversation.
3. **Never contact the retailer and never transact.** No chat, no call, no
   email, no form. No buying, returning or cancelling, even if asked. The claim
   comes from the purchaser.
4. **Verify eligibility before reporting.** A find that fails any rule in
   `references/price-match-rules.md`, including the full exclusion list, is not
   a find. Silence beats a false alarm.
5. **The ledger is state, not memory.** Read it at the start of every sweep and
   write it at the end. Never carry a conclusion between runs in your head, and
   never trust a number in the ledger as a current price.
6. **Do not carry payment data anywhere.** See "What to extract" below.
7. **State uncertainty plainly.** When a live page contradicts this skill's
   reference files, trust the page and say so.

## Setup, once

Ask for one thing: the folder where order confirmations will live. Then create
the state directory inside it and write the config.

```
<WATCH FOLDER>/
  best-bot/
    config.json       settings
    watchlist.json    the ledger, canonical state
    WATCHLIST.md      human-readable view, rewritten each sweep
```

`references/watchlist-format.md` has the schema for both files and the config
defaults to ask about (membership tier, cadence, notification preference).

Everything the sweep needs lives in that folder. Whatever ends up triggering
the sweep only has to know the folder path, which is the point: no purchase
details get baked into a schedule, so items can be added and retired without
touching it.

## The sweep

This is the single entry point. It runs the same whether a schedule fired it,
a cron job called it, or the user typed "check my watches". Four phases, in
order.

### Phase 1: Intake

Scan the watch folder for confirmations not yet in the ledger. Match on order
number, which is the dedupe key. Parse PDFs, .eml files, screenshots, pasted
text. Extract the fields in "What to extract" below and add each as an active
watch.

If the user has a mail connector and config enables it, also run the Gmail
top-up described in `references/watchlist-format.md` to catch confirmations
they never saved. Mail is a supplement; the folder is the source of truth.

Skip non-Best-Buy confirmations with a one-line note. Do not guess at other
retailers' rules.

### Phase 2: Check

For every watch whose status is `active` and whose window has not closed, run
the price check in `references/checking-procedure.md`. Batch sensibly: load
each SKU's Best Buy page first, since a drop there ends that item's check.

Respect `check_after` on each watch. An item checked today does not need
checking again in the same day's second sweep.

### Phase 3: Retire

Any watch whose window closed moves to `retired` with an outcome: `no_drop`,
`claimed`, `drop_missed`, or `ineligible`. Record the dollar amount on
`claimed`. Never delete a retired entry; the log is the only record of what
this has actually recovered.

### Phase 4: Report

**One notification per sweep, not one per item.** Five watches producing
nothing means silence. Notify only when at least one of these is true:

- A qualifying find on one or more items
- A window closing within three days, mentioned once
- A failure that stopped the sweep, or an item that could not be checked

Lead with the finds. Then the closing-soon warnings. Then anything that broke.
Include the filled claim script for every find.

Rewrite `WATCHLIST.md` at the end of every sweep, including sweeps that notify
nobody, so the file is always current when the user opens it.

## What to extract

| Field | Notes |
|---|---|
| Order number | `BBY01-` plus digits. The dedupe key |
| Order date | Context, not the clock |
| **Delivery date** | **Starts the window.** See below |
| Item name | Full product title |
| Model number | Manufacturer model |
| SKU | Best Buy's SKU, usually 7 digits, the most reliable identifier |
| Price paid, pre-tax | The comparison basis |
| Sold by | Best Buy, or a Marketplace seller |

Marketplace purchases are excluded from the Price Match Guarantee and get no
member return extension. Record them as `ineligible` with the reason, say so
once, and do not check them again.

Extract only these fields. Do **not** copy card numbers (even the last four),
billing addresses, phone numbers or names into the ledger, the config, any
notification, or your replies. Do not copy the confirmation file anywhere;
read it in place and work from the extracted fields.

## Working out the window

**The clock starts at delivery**, not at purchase. Best Buy's policy begins the
period the day the product is received, which on a shipped item is often
several days later than the order date. Erring early costs the user the tail of
the window. If the delivery date is unknown, fall back to the order date, mark
`window_basis: "order_date"` in the ledger, and say the end date is
conservative.

**Length comes from membership tier.** No membership is 15 days, Plus or Total
is 60, activatable devices are 14 for everyone and Verizon-activatable 30.
Category exceptions are in `references/price-match-rules.md`.

## The claim

Produce a script the user reads on the phone at 1-888-237-8289, or pastes into
Best Buy's chat where it is available. Phone is the reliable channel. Templates
and pushback handling are in `references/claim-scripts.md`.

The request must come from the purchaser. Refunds go to the original payment
method and include the tax on the difference.

When the user reports back that a claim succeeded, update that watch:
`status: claimed`, with the amount. That number feeds the running total.

## Running it on a schedule

`references/running-the-sweep.md` covers the mechanisms that exist today and
how to set each up. The short version: the sweep is one instruction naming one
folder, so it ports to any of them, and to none of them if the user would
rather run it by hand.

## References

- `references/watchlist-format.md`: config and ledger schema, the markdown
  view, Gmail top-up, migrating a v1 single-item watch
- `references/running-the-sweep.md`: trigger mechanisms, cadence, failure
  behavior, turning it off
- `references/price-match-rules.md`: eligibility, full exclusion list,
  qualified competitors, window lengths, restocking fees
- `references/checking-procedure.md`: browsing scope and how to check a price
  without being fooled by bundles, marketplace sellers, member pricing or
  stale pages
- `references/claim-scripts.md`: claim wording and pushback handling
