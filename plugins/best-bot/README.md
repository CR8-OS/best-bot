# Best Bot

Watches a Best Buy purchase for price drops through your return and exchange
window, and writes the price-match claim when one lands.

Best Bot is an independent project. It is **not affiliated with, endorsed by,
or sponsored by Best Buy Co., Inc.** "Best Buy," "My Best Buy," and related
marks belong to their owner and are referenced here only to describe what this
tool works with.

See the [repository README](../../README.md) for install and usage.

## Contents

- `skills/best-bot/SKILL.md` — the skill
- `skills/best-bot/references/price-match-rules.md` — eligibility, the full
  exclusion list, qualified competitors, window lengths, restocking fees
- `skills/best-bot/references/checking-procedure.md` — browsing scope, and how
  to check a price without being fooled by bundles, marketplace sellers,
  member pricing, or stale pages
- `skills/best-bot/references/claim-scripts.md` — claim wording and pushback
  handling
- `skills/best-bot/references/schedule-template.md` — scheduled-task setup and
  the self-contained run prompt
- `assets/` — logo

## Asset notes

`best-bot-mark.svg` is the primary asset and the one to use as an icon: a
downward price arrow with the bot's eyes in the shaft. It is pure geometry and
renders identically everywhere. The `best-bot-logo*.svg` lockups set the
wordmark in a font stack (Inter, then system sans), so the wordmark's exact
shape depends on what the viewer has installed — use `best-bot-logo.png` where
the rendering must be fixed.

The logo was drawn for this project. Best Buy's marks, logo, and colors are not
used in it.

Palette: mint `#2FE3A6`, ink `#0B1B2E`.
