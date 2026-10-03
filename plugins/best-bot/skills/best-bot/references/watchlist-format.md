# Config and ledger

Everything Best Bot knows lives in one folder the user names. Nothing is baked
into a schedule, which is what lets watches come and go without anyone editing
a trigger.

```
<WATCH FOLDER>/                 confirmations get saved here
  best-bot/
    config.json                 settings
    watchlist.json              canonical state, machine-written
    WATCHLIST.md                human view, rewritten every sweep
```

## config.json

```json
{
  "watch_folder": "<absolute path to the folder>",
  "default_membership": "plus",
  "cadence": "daily",
  "gmail_topup": false,
  "gmail_query": "from:bestbuy.com subject:(order OR confirmation) newer_than:90d",
  "notify": "finds_only",
  "currency": "USD"
}
```

- `default_membership`: `none`, `plus` or `total`. Used when a confirmation
  does not reveal the tier. Ask once at setup.
- `gmail_topup`: only true if a mail connector exists and the user opted in.
- `notify`: `finds_only` is the default and the right one. `verbose` reports
  every sweep and should be offered only for debugging.

## watchlist.json

```json
{
  "version": 2,
  "updated": "2026-10-02",
  "watches": [
    {
      "order_number": "BBY01-807254443337",
      "status": "active",
      "item": "ASUS ROG Zephyrus G14 14in 3K OLED",
      "model": "GA403WW-G14.R95080",
      "sku": "6613953",
      "product_url": "https://www.bestbuy.com/site/-/6613953.p",
      "price_paid_pretax": 3169.99,
      "sold_by": "Best Buy",
      "order_date": "2026-09-15",
      "delivery_date": "2026-09-17",
      "window_basis": "delivery_date",
      "membership": "plus",
      "window_days": 60,
      "window_ends": "2026-11-16",
      "source_file": "bestbuy-laptop-sep15.pdf",
      "added": "2026-09-18",
      "last_checked": "2026-10-02",
      "check_after": "2026-10-03",
      "outcome": null,
      "amount_recovered": null,
      "notes": []
    }
  ]
}
```

### Status values

| Status | Meaning |
|---|---|
| `active` | Inside its window, gets checked every sweep |
| `claimed` | A price match was granted. Record `amount_recovered` |
| `retired` | Window closed. `outcome` says what happened |
| `ineligible` | Marketplace seller or otherwise not matchable. Never checked |

### Outcome values, set when retiring

`no_drop`, `claimed`, `drop_missed` (a qualifying drop was reported but no
claim was confirmed), `ineligible`.

### Rules for writing it

- **Order number is the key.** Never create a second entry for one that exists.
  A re-saved confirmation updates the existing entry rather than duplicating it.
- **Never delete a retired entry.** The history is the only record of what this
  has recovered.
- **Never store a price seen on a product page.** The ledger holds the price
  paid and nothing else about pricing. Current prices live for the length of a
  run and die with it, which is what keeps Hard Rule 1 enforceable.
- **Write the whole file atomically** at the end of the sweep, not field by
  field mid-run. A sweep that dies partway should leave the previous ledger
  intact.
- If `watchlist.json` is missing or unparseable, do not silently start fresh.
  Say so, rebuild from the confirmations in the folder, and tell the user the
  claim history could not be recovered.

## WATCHLIST.md

Regenerate every sweep. This is what the user actually opens.

```markdown
# Best Bot watchlist
_Last swept 2026-10-02_

## Watching now (2)

| Item | Paid | Window ends | Days left | Last checked |
|---|---|---|---|---|
| ASUS ROG Zephyrus G14 | $3,169.99 | 2026-11-16 | 45 | 2026-10-02 |
| Brother HL-L2460DW printer | $179.99 | 2026-11-28 | 57 | 2026-10-02 |

## Closed (1)

| Item | Outcome | Recovered |
|---|---|---|
| ASUS ROG Zephyrus G14 (Sept) | Claimed | $620.00 |

**Recovered to date: $620.00 across 1 claim.**
```

## Gmail top-up

Only when `gmail_topup` is true and a mail connector is available. Search with
`gmail_query`, read matching messages, extract the same fields, and add any
order number not already in the ledger. Note `source: "gmail"` on those
entries so the user can tell which ones they never saved.

If the connector is missing or unauthorized, carry on with the folder and
mention it once in the next notification you were already sending. A broken
mail connector must never stop a sweep.

## Migrating a v1 watch

v1 kept one purchase per scheduled task, with the details written into the
trigger prompt. To migrate:

1. Create the watch folder and `best-bot/` state directory.
2. Read the purchase details out of the old trigger prompt and write them in as
   a watch, preserving the original window end date.
3. Create the single sweep trigger.
4. Tell the user which old triggers to delete, by name. Do not delete them
   yourself; a scheduled run cannot reliably identify its own trigger, and
   deleting the wrong one is worse than leaving a stale one.
