<p align="center">
  <img src="plugins/best-bot/assets/best-bot-logo.png" alt="Best Bot" width="400">
</p>

<p align="center">
  <strong>Retailers will refund the difference when a price drops. They just won't tell you.</strong>
</p>

---

Best Buy's Price Match Guarantee says that if they lower their own price during
your return and exchange period, they'll match it — upon request. That last
phrase is doing a lot of work. Nobody emails you. The window closes quietly.

Best Bot is the part that asks. Hand it a receipt; it works out how long your
window runs, watches the price on a schedule, and when a qualifying drop shows
up it tells you and writes the claim. You make the call — that part is yours,
and by design.

One anecdote, not a promise: the first watch we ran hit three days in, for
$620 back on a laptop bought the week before. Your mileage depends entirely on
whether the price moves.

## Install

```
/plugin marketplace add CR8-OS/best-bot
/plugin install best-bot@cr8-os
```

## Use

Give it your order details — upload the confirmation email, paste the receipt
text, or drop a screenshot:

> Watch this Best Buy order for a price drop.

It will confirm the details it extracted, work out your window end date, run a
first check immediately, and set up a recurring watch that stops when your
window does. After that it stays quiet unless there's something worth acting
on.

To stop it, delete the scheduled task it created, or just ask.

## What it needs

- **A browser in the session.** Amazon and Walmart block plain page fetches, so
  real price checks need a real browser. If the watch will use the browser on
  your own machine, the scheduled task has to be created with that requirement
  set — it can't be added afterward, and the skill handles this at setup.
- **Scheduled task support**, for the recurring watch. Without it, Best Bot
  will still do a one-off check and tell you what it found.
- **Automatic approval on the task**, or runs will stall waiting for a prompt
  nobody is there to answer.

## What it won't do

Deliberately:

- **It won't contact Best Buy.** No chat, no calls, no forms. It writes the
  script; you send it. A refund request should come from the person who made
  the purchase.
- **It won't buy, return, or cancel anything**, and it won't sign in to any
  retailer or add anything to a cart. Browsing is read-only and public.
- **It won't report a price it didn't load.** Every number it gives you comes
  from a page it opened during that run, with the URL attached. If a check
  fails, it says the check failed — it does not quietly fall back to a guess.
- **It won't keep your payment details.** It extracts an order number, dates,
  a SKU, and a price. Card numbers and addresses don't go into its notes, the
  scheduled task, its notifications, or its replies.
- **It won't ping you for nothing.** No news means no notification.

## Windows, accurately

Two things people get wrong, both of which cost money:

**The clock starts at delivery, not at purchase.** Best Buy's policy begins
the period the day you receive the product. On a shipped item that's often
several days of extra window.

**The length comes from membership, not from a Best Buy credit card.**

| | Window |
|---|---|
| No membership | 15 days |
| My Best Buy Plus / Total | 60 days |
| Activatable devices (phones, cellular tablets, hotspots, cellular wearables) | 14 days, everyone |
| Verizon activatable devices | 30 days, everyone |
| Marketplace purchases | Standard 15 days, no member extension, and not price-matchable |

Other category exceptions exist, along with the full price-match exclusion
list and the restocking fees that make "just return and rebuy" a bad idea for
cameras and drones. All of it is in the skill's reference files.

## Brand and legal

Best Bot is an independent project. It is **not affiliated with, endorsed by,
or sponsored by Best Buy Co., Inc.** "Best Buy," "My Best Buy," and related
marks belong to their owner and are referenced here only to describe what this
tool works with.

The Best Bot logo was drawn for this project. Best Buy's marks, logo, and
colors are not used in it.

Store policies change without notice. This plugin encodes Best Buy's published
policy as of September 2026 and instructs Claude to trust a live policy page
over its own reference files when the two disagree. Verify anything that
matters before you rely on it.

## Repo layout

```
.claude-plugin/marketplace.json     marketplace definition
plugins/best-bot/
  .claude-plugin/plugin.json        plugin manifest
  skills/best-bot/SKILL.md          the skill
  skills/best-bot/references/       policy rules, checking procedure,
                                    claim scripts, schedule template
  assets/                           logo
```

## License

MIT. See [LICENSE](LICENSE).
