# Running the sweep

The sweep is one instruction naming one folder. That is deliberate. Whatever
fires it only needs to say this much:

```
Run the Best Bot sweep. Load the best-bot skill and follow it.
Watch folder: <ABSOLUTE PATH>

Notify only on a qualifying find, a window closing within three days, or a
failure that stopped the sweep. Stay silent otherwise.

Never state a price that was not loaded in this run. Never contact Best Buy.
Never buy, return or cancel anything. If you cannot load the best-bot skill,
check only Best Buy's own price for each active SKU in the ledger, skip
competitors, and say the full rule set was unavailable.
```

No purchase details. No SKUs. No dates. Adding a printer to the watchlist
means saving its confirmation to the folder, not editing a trigger.

## Mechanisms that exist today

Checked against Anthropic's published docs on 2026-10-02. All of these exist
and none carries a deprecation notice. Pick on durability, not novelty.

| Mechanism | Runs where | Survives the machine being off | Notes |
|---|---|---|---|
| Cowork scheduled task | Cloud | Yes | Hourly, daily or weekly. Needs `requires_local_device: true` when the check uses a browser on the user's own machine, and that flag cannot be added after creation |
| Cloud routine | Cloud | Yes | Cron, one hour minimum. Research preview, so treat it as subject to change |
| Desktop scheduled task | The user's machine | No, pauses when the app closes or the machine sleeps | One minute minimum. Fine for an always-on desktop |
| Managed agent scheduled deployment | Cloud | Yes | Cron to the minute via the Claude Platform API. Most durable, most setup |
| OS cron or Task Scheduler calling headless Claude | The user's machine | No, needs the machine awake | Ordinary system scheduling, fully under the user's control |
| GitHub Actions on a cron | GitHub | Yes | Works, but a price watch holding someone's purchase history in a repo is a poor fit |
| `/loop` | The session | No, expires after seven days | For watching something during one working session, not a 60-day window |

Claude Code hooks are event-driven only. `SessionStart`, `PreToolUse` and the
rest fire on session and tool lifecycle, never on a clock. Never suggest hooks
as a scheduling mechanism.

## The October 6, 2026 Cowork change, and why it matters here

Anthropic's support docs state that on October 6, 2026, new Cowork tasks on Pro
and Max plans run in the cloud, and the "Only on your computer" option in
Settings > General is removed. Existing scheduled tasks move to the cloud too,
including ones that use files on the user's computer, and the docs say those
"need the desktop app open".

Source: https://support.claude.com/en/articles/15520349-use-claude-cowork-on-web-desktop-and-mobile

This is not a deprecation. Scheduled tasks continue to exist. But it changes
where a sweep runs, and Best Bot reads a folder on the user's machine, so say
this plainly at setup rather than letting them find out through silent failures:

- If the watch folder is local and the sweep runs as a Cowork task, **the
  desktop app has to be open** when the task fires. A closed app means a sweep
  that cannot see the folder.
- A sweep that cannot reach its folder must say so. Treat it as a failure worth
  notifying about, never as "no new confirmations". Those look identical from
  the inside and only one of them is fine.
- If the user cannot keep the app open, the honest options are a mechanism that
  runs on their machine (a desktop scheduled task or OS cron), or turning on
  the Gmail top-up so confirmations reach the ledger without the folder being
  readable that minute. Say which tradeoff they are taking.

Re-check this before relying on it. It was accurate on 2026-10-03 and dated
features move.

## Choosing one

Ask, do not assume:

- **Unattended, machine often off:** a cloud mechanism. The Cowork scheduled
  task is the least setup.
- **Always-on desktop, wants it local and visible:** desktop scheduled task or
  OS cron.
- **Nothing on a timer:** a legitimate answer. The sweep works fine when the
  user types "check my watches". Say the tradeoff once, that a sweep nobody
  runs catches nothing, then respect the choice.

Whichever is chosen, say plainly what it is called, when it runs, and how to
turn it off. Say it at setup, not only at the end.

## Settings that decide whether it works at all

1. **Browser access.** Amazon and Walmart block plain fetches, so the check
   needs a real browser. On a Cowork scheduled task that means
   `requires_local_device: true` at creation time, and it cannot be added
   afterward.
2. **Notifications on.** The sweep is silent by design, so the one message that
   matters has to be able to reach them.
3. **Automatic approval.** A scheduled run driving a browser will stall on
   permission prompts with nobody there to answer, which is worse than no watch
   because the user believes it is running.
4. **UTC.** Cron is evaluated in UTC. Convert the user's local time, shift the
   day fields if the conversion crosses midnight, and confirm the local time
   back to them.

## Cadence

Daily suits almost everything. Twice daily only for a high-value item inside a
known sale period. Prices do not move hourly, and every run costs the user
something.

`check_after` on each watch stops a second sweep in the same day from
re-checking what was just checked.

## When a mechanism changes

Scheduling features move. Because the trigger carries only a folder path,
switching mechanisms means recreating one instruction, and the ledger, the
claim history and every active watch survive untouched. If the sweep stops
running, nothing is lost but the time it was not running, and `WATCHLIST.md`
still shows exactly where everything stood.
