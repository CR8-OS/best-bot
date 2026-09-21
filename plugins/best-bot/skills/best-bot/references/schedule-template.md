# Scheduled task template

Each run starts a fresh session with no memory of the conversation that set it
up. Everything the run needs goes in the prompt.

## Creating the task

Use the session's scheduled-task tooling (on Cowork, the Claude Code Remote
MCP `create_trigger`). Do not use an in-process scheduler — those die with the
session and the watch silently never runs.

Four settings decide whether the watch actually works. Get them right at
creation:

1. **`requires_local_device: true`** when the price check depends on a browser
   on the user's machine. This **cannot be added after the task is created**.
   A task created without it runs in the cloud with no access to that browser,
   and every run fails.
2. **Notifications on** (`notifications: {push: true, email: true}`). The
   watch is silent by design, so the one message that matters must be able to
   reach them. Left unset, delivery depends on a server default.
3. **Automatic approval.** A scheduled run that drives a browser will hit
   permission prompts with nobody there to answer. Tell the user at setup that
   the task needs automatic approval switched on in its settings, or runs will
   stall silently — which is worse than no watch, because they'll believe it's
   working.
4. **UTC cron.** The cron expression is evaluated in UTC. Convert the user's
   local time before writing it, shift the day fields if the conversion
   crosses midnight, and confirm the local time back to them.

Give the task a distinctive name that includes the item, and embed that exact
name in the prompt so a run can tell the user precisely which task to delete.

## Prompt template

Fill every bracket. Verify no placeholder survives before creating the task.

```
Daily Best Buy price-drop watch. Load the best-bot skill and follow it.

THIS TASK IS NAMED: [EXACT TASK NAME]

PURCHASE
- Retailer: Best Buy
- Order number: [ORDER NUMBER]
- Order date: [ORDER DATE]
- Delivery date: [DELIVERY DATE]   (the return window starts here)
- Item: [ITEM NAME]
- Model: [MODEL NUMBER]
- SKU: [SKU]
- Product URL: [BEST BUY PRODUCT URL]
- Price paid, pre-tax: [PRICE PAID]
- Membership: [none / My Best Buy Plus / My Best Buy Total]
- Return and exchange window ends: [WINDOW END DATE]

WHAT TO DO
1. If today is past [WINDOW END DATE], notify that the window has closed, tell
   the user to delete the scheduled task named above, and stop.
2. Check Best Buy's current price for SKU [SKU] using a browser.
3. If it is below [PRICE PAID], run the exclusion pass before treating it as a
   find — clearance, open-box, limited-quantity and special daily or hourly
   sale prices are excluded even when the price is Best Buy's own.
4. Otherwise check qualified competitors per the skill's checking procedure.
5. Apply every eligibility rule before treating anything as a find.
6. Notify ONLY on: a qualifying find; a window-closing warning if today is
   exactly three days before [WINDOW END DATE]; or a failure that prevented
   the check from running. Stay silent on an ordinary no-change run.
7. On a find, include retailer, exact URL, price, seller, stock status, the
   dollar difference versus [PRICE PAID], a rule-by-rule eligibility check,
   and the filled-in claim script.

RULES
- Never state a price that was not read from a page loaded in this run.
- Use a real browser. If no browser tool is available in this run, notify that
  the check could not run and stop. Do not substitute a plain HTTP fetch or a
  search-result snippet for a loaded page.
- Browsing is read-only and public: product and search URLs only. Do not sign
  in, do not open support, chat, contact, returns or order pages, do not add
  anything to a cart.
- Never contact Best Buy. The purchaser makes the claim.
- Never buy, return, or cancel anything.
- If you cannot load the best-bot skill in this run, check only Best Buy's own
  price for the SKU above, skip competitors entirely, and say in your
  notification that the full rule set was unavailable.
```

## Cadence

- Daily at a consistent local morning hour suits most purchases.
- Twice daily for items over $1,000 or during a known sale period.
- More often than that is noise — retail prices do not move hourly, and each
  run costs the user something.

## Turning it off

Tell the user at setup how to stop the watch: it appears in their scheduled
tasks list under the name you gave it, and can be deleted there, or they can
ask Claude to delete it by that name. Say this at setup, not only at the end —
a user who cannot find the off switch will not install the next thing you
build.
