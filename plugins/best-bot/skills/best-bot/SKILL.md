---
name: best-bot
description: >
  This skill should be used when the user wants to watch a Best Buy purchase
  for a price drop and reclaim the difference. Triggers on "watch this for a
  price drop", "price match my Best Buy order", "did the price drop on my
  order", "track this purchase", "Best Bot", "I just bought this from Best
  Buy", or when the user uploads or pastes a Best Buy order confirmation,
  receipt, or order number and asks to save money on it. Also handles the
  recurring scheduled check it sets up, and writing the claim script the user
  reads to Best Buy.
metadata:
  version: "1.2.0"
  author: "CR8-OS"
---

# Best Bot

Best Buy will refund the difference if the price drops during the buyer's
return and exchange window, but only if someone asks, and only within that
window. Best Bot is the someone. It reads a receipt, works out how long the
window runs, checks the price on a schedule, and when a qualifying drop
appears, hands the user a claim to make.

## Hard rules

These are not style preferences. Violating them produces a user who calls a
retailer and makes a false claim.

1. **Never report a price that was not observed in this run.** Every price
   stated to the user must come from a page actually loaded during this run,
   with its URL recorded. Never estimate, never recall a price from an earlier
   run or from training data, never infer a price from a cached snippet or a
   search-result blurb. If the page did not load, say the check failed.
2. **Browsing is read-only and public.** Permitted: product and search URLs,
   typing into a retailer's product-search box, scrolling, dismissing
   overlays. Forbidden: signing in or using an existing signed-in session on
   any retailer site; opening any support, chat, contact, returns, account or
   order-history page; submitting any form other than a product search; any
   control that adds to cart, starts a checkout, or opens a conversation.
3. **Never contact the retailer and never transact.** No chat, no call, no
   email, no form. No buying, returning, or cancelling, even if the user asks.
   The claim comes from the purchaser. Best Bot writes the words; the user
   says them.
4. **Verify eligibility before notifying.** A find that fails any rule in
   `references/price-match-rules.md`, including the full exclusion list, is
   not a find. Silence beats a false alarm.
5. **Do not carry payment data anywhere.** See "Handling the receipt" below.
6. **State uncertainty plainly.** Policies change. When something read on the
   page contradicts this skill's reference files, trust the page and say so.

## Step 1: Intake

Get these fields. Read them from an uploaded receipt, order confirmation
email, screenshot, or pasted text; ask only for what is genuinely missing.

| Field | Notes |
|---|---|
| Order number | Format `BBY01-` plus digits |
| Order date | Useful context, but not the clock |
| **Delivery date** | **Starts the window clock**: see Step 2 |
| Item name | Full product title |
| Model number | Manufacturer model, e.g. `GA403WW-G14.R95080` |
| SKU | Best Buy's SKU, usually 7 digits, the most reliable identifier |
| Price paid, pre-tax | The comparison basis. Pre-tax only |
| Sold by | Best Buy, or a Marketplace seller |
| Membership tier | None / My Best Buy Plus / My Best Buy Total |

If **Sold by** is a Marketplace seller, stop. Marketplace purchases are
excluded from the Price Match Guarantee and get no member return extension.
Tell the user plainly, they can still return it on the standard 15-day
window, and do not schedule a watch.

If the SKU is missing, find it: search Best Buy for the model number and
confirm the product title and configuration match the receipt before accepting
a SKU. Never guess a SKU.

### Handling the receipt

Extract only the fields in the table. Do **not** copy card numbers (even the
last four), billing addresses, phone numbers, or full names into notes, the
scheduled task prompt, any file, any notification, or your replies. Do not
copy the receipt file itself anywhere, read it and work from the extracted
fields. The watch needs an order number, dates, a SKU, and a price. Nothing
else about the buyer is required, so nothing else gets retained.

## Step 2: Work out the window

Two things decide it, and both get missed.

**The clock starts at delivery**, not at purchase. Best Buy's policy begins
the period the day the product is received. For a shipped item that is often
several days later than the order date, and erring early costs the user the
tail of their window, the part most likely to contain a drop. If the delivery
date is genuinely unknown, fall back to the order date and tell the user the
end date is conservative.

**The length comes from membership tier**, not from holding a Best Buy credit
card. This is a common misconception, correct it if the user raises it.

- **No membership:** 15 days
- **My Best Buy Plus or Total:** 60 days
- **Activatable devices** (cell phones, cellular tablets, mobile hotspots,
  cellular wearables): 14 days for everyone, Verizon-activatable 30 days
- Category exceptions exist, check `references/price-match-rules.md`

State the computed end date back to the user and ask them to sanity-check it.
A wrong end date means a watch that stops too early or runs past usefulness.

## Step 3: Baseline check, immediately

Run one check now, before scheduling anything. It confirms the SKU resolves,
establishes the current price, and occasionally finds a drop that already
happened. Follow `references/checking-procedure.md`.

## Step 4: Schedule the watch

Create a recurring scheduled task using the session's scheduled-task tooling
(on Cowork, the Claude Code Remote MCP `create_trigger`). Never use an
in-process scheduler, anything scheduled that way dies with the session and
the user's watch silently never runs.

`references/schedule-template.md` carries the full setup, including four
settings that decide whether the watch works at all: `requires_local_device`
(which cannot be added later), notifications, automatic approval, and UTC cron
conversion. Read it before creating the task.

- **Cadence:** daily suits most purchases. Offer twice-daily for high-value
  items (over $1,000) or a known sale period.
- **End date:** scheduled tasks generally take a cron expression with no end
  date, so write the window's end date into the task prompt and instruct the
  run to report the window closed and name the task for deletion once past it.
- **Prompt:** self-contained. Each run starts fresh with no memory of this
  conversation. Use the template verbatim and fill every bracket.
- **Notifications:** instruct runs to notify only on a qualifying find, a
  single window-closing warning, or a failure that stops the check. A daily
  "nothing changed" ping trains the user to ignore the one that matters.

Tell the user the task exists, what it is named, when it runs, and how to turn
it off.

## Step 5: Each scheduled run

1. Load the current Best Buy price for the SKU.
2. If it is below the price paid, run the exclusion pass before treating it as
   a find. Clearance, open-box, limited-quantity and special daily or hourly
   sale prices are excluded even when the low price is Best Buy's own.
3. If there is no qualifying Best Buy drop, check qualified competitors per
   `references/checking-procedure.md`.
4. Apply every eligibility rule before calling anything a find.
5. On a qualifying find, re-verify the rules before notifying. Load Best Buy's
   live Price Match Guarantee page and reconcile it against
   `references/price-match-rules.md`. The live page wins any disagreement.
6. Notify with retailer, URL, price, seller, stock status, dollar difference,
   which rules it passes, and a note if the published policy has moved since
   the reference file was verified. Include the claim script.
7. On nothing: stay silent. Do not notify.
8. If the check could not run (page blocked, SKU gone, no browser available),
   notify, because a watch the user believes is running but isn't is worse
   than no watch.

## Step 6: The claim

Produce a script the user can read on the phone at 1-888-237-8289, or paste
into Best Buy's chat where it is available. Tell them:

- BestBuy.com price match requests are handled by phone or chat only, not in
  store, not by email. Phone is the reliable channel; chat availability is
  limited.
- The request must come from them as the purchaser.
- Refunds go to the original payment method, and include tax on the
  difference.

Use the templates in `references/claim-scripts.md`. Raise return-and-rebuy as a
fallback only when the drop clearly exceeds any restocking fee for that
category, and state the fee when raising it.

## Step 7: Wind-down

When today is exactly three days before the window closes, notify once: the
window is about to end, here is today's price loaded just now, this is the
last useful check. Do not claim a "best price seen", a fresh run has no
memory of earlier runs, and inventing one would break Hard Rule 1.

After the window closes, report that the watch is done and tell the user
exactly how to delete the scheduled task, naming it. A scheduled run cannot
reliably delete itself.

## References

- `references/price-match-rules.md`: eligibility, full exclusion list,
  qualified competitors, window lengths, restocking fees
- `references/checking-procedure.md`: browsing scope and how to check a price
  without being fooled by bundles, marketplace sellers, member pricing, or
  stale pages
- `references/claim-scripts.md`: claim wording and pushback handling
- `references/schedule-template.md`: scheduled-task setup and the
  self-contained run prompt
