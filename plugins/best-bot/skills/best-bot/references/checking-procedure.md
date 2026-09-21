# Checking procedure

## Browsing scope — read this before opening anything

Browsing here is read-only and public.

**Permitted:** navigating to product and search URLs; typing into a retailer's
product-search box; scrolling; dismissing cookie or modal overlays.

**Forbidden:** authenticating, or using an existing signed-in session, on any
retailer site; navigating to any support, chat, contact, returns, account, or
order-history URL; submitting any form other than a product search; any control
that adds to cart, starts a checkout, or opens a conversation.

Prefer a logged-out or private context. Costco, Sam's Club, BJ's and Amazon all
show member or Prime-gated prices to signed-in visitors, and a member price is
not matchable — from the page alone you often cannot tell which price you are
looking at.

## Tools

Use a real browser. Amazon and Walmart block plain HTTP fetches and return
bot-check pages or empty shells; a scripted fetch that appears to "succeed"
against them usually returned nothing usable.

If no browser is available in the session, say the check could not run and
stop. Do not substitute a plain HTTP fetch or a search-engine snippet for a
loaded page — snippets go stale, strip seller and stock context, and are the
single most likely source of a wrong claim.

## Order of checks

1. **Best Buy's own SKU page.** `https://www.bestbuy.com/site/-/<SKU>.p`
   resolves; prefer the full product URL when known. Read: current price,
   "Sold by", stock and fulfillment, and any deal countdown or badge.
2. **If the Best Buy price is below the price paid, run the exclusion pass
   before celebrating.** Is it badged clearance, open-box, or limited
   quantity? Is it a daily or hourly deal? Those are excluded even though the
   price is Best Buy's own. If it survives the exclusion pass, stop here — no
   competitor check can improve on the retailer matching itself.
3. **Amazon and Walmart**, searched by model number first, then by a
   distinctive product-title phrase if the model number returns nothing.
4. **Spot-check remaining qualified retailers** when steps 1-3 found nothing
   and the item is one a specialist might carry (B&H and Micro Center for
   computing, Abt and P.C. Richard for appliances, Crutchfield for car and
   home audio).

## Reading a listing correctly

For every candidate price, record and check:

- **URL** — the exact page. Needed for the claim.
- **Seller** — "Sold by" or "Ships from and sold by". A third-party name
  disqualifies it. On Amazon, "Ships from Amazon, Sold by <someone else>" is
  still a third-party sale and is excluded.
- **Condition** — new only.
- **Stock** — in stock and available now. "Only 1 left" and "limited
  quantity" language disqualifies.
- **Bundle** — read the full title. Vendors hide bundles in the title tail
  ("... + Traix MousePad"). A bundled title is not a match.
- **Configuration** — confirm CPU, GPU, RAM, storage, size and color against
  the receipt. Retailers list near-identical variants side by side.
- **Conditions** — coupon checkboxes, card-specific pricing, membership
  pricing, subscription pricing, deal countdowns.

**If a price is only revealed by adding the item to a cart, do not add it.**
Record "price not publicly displayed" and move on. Cart-gated MAP pricing is
not worth breaking the no-cart rule for.

## Anti-hallucination discipline

- Quote the price as it appeared on the loaded page.
- If a page fails to load or a bot check appears, record it as "could not
  check", not as "no lower price found". These are different, and the user
  deserves to know which happened.
- Never fill a gap from memory. A price from a previous run is history, not a
  current price.
- Before notifying, restate the find against each rule in
  `price-match-rules.md` — including the full exclusion list — and confirm it
  passes. Write the check out; do not assert it.

## What a good find looks like

Retailer, exact URL, price, seller, stock status, condition, dollar difference
versus price paid, and a rule-by-rule pass. If any one of those is unknown, the
find is not ready to report.
