# Pocket Critter

A mobile-first virtual pet for grown-up 90s kids. Adopt an original critter, feed
it, play with it, send it to work, and watch its personality develop over weeks.

**The pet never dies.** Neglect makes it grumpy, droopy and guilt-inducingly
dramatic — never dead. This is a comfort product, not an anxiety machine.

Everything is original: art, names, copy, code. No borrowed species, sprites,
fonts or UI.

## Run it

It is one self-contained HTML file with no build step and no dependencies:

```bash
open pocket-critter.html
```

Fonts come from Google Fonts; everything else — art, sound, game loops — is
generated in the page.

## What's built

| System | Detail |
| --- | --- |
| Species | Blorbit, Snizzard, Gremble, Lampmoth — parametric SVG, 4 colorways each |
| Needs | Hunger 14h, Happiness 16h, Energy 13h full-to-empty, decayed lazily from timestamps |
| Moods | Joyful, Content, Hungry, Grumpy, Sleepy, Dramatic, Asleep, At work — each drives idle animation and dialogue |
| Growth | Baby → Kid → Adult, from days-since-adoption plus average care |
| Personality | Foodie, Playful, Sleepyhead, Grindset, Diva — earned from care patterns, hinted day 3, locked day 5 |
| Dialogue | `MOOD_LINES` ∪ `PERSONA_LINES`, 11–12 lines per mood × personality path |
| Economy | Buttons, from job shifts (4h/8h), capped mini-games, and a daily bonus |
| Shop | 4 foods, 3 toys, 6 hats, 6 decor slots |
| Mini-games | Soup Catch, Button Match, Moth Dash — one thumb, instant restart, keyboard fallbacks |
| Social | Visit-a-friend via `?pet=<id>`, read-only, gift a cookie once per day per visitor |
| Sharing | 1200×630 canvas postcard: room, critter, one true stat, visit link |

## Architecture

Single file, ordered top to bottom: tokens and CSS, then markup, then data
(species, colorways, dialogue, shop, jobs), then the engine, renderers, actions,
games, postcard and boot.

The pieces worth knowing about:

- **Meters are never ticked by a timer.** `simulate()` walks from `lastTick` to
  now in 5-minute steps, applying the sleeping/working/awake decay rate for each
  step. Weeks offline resolve correctly on the next open; there is no cron.
- **Dialogue is data.** Edit `MOOD_LINES`, `PERSONA_LINES` and `STAGE_LINES` to
  change what the critter says. No render code needs touching.
- **Sprites are parametric.** `SPRITES[species](colorway, stage)` returns SVG
  with classed parts (`.sp-all`, `.sp-lid`, `.sp-brow`, `.sp-mouth-g`, `.sp-hat`)
  that the mood CSS animates independently. The stage scale lives on an outer
  `.sp-scale` group because a CSS transform overrides an SVG transform attribute.
- **Shared state is optional.** Visit links and cookies use the artifact runtime's
  `db` capability; the page degrades to a clear message when it resolves `null`,
  and solo play never depends on it.

## Known gaps

Three things in the original spec are not fully deliverable in this hosting
model, and are stubbed at a clean seam rather than faked:

1. **Open Graph unfurling.** The page can't control the `<head>` of its own
   hosted URL, so links unfurl with the host's card, not
   "Biscuit the Moth needs soup". The postcard generator already produces exactly
   the image an `og:image` wants — this needs a real domain.
2. **Third-party analytics.** No script host for GoatCounter/Plausible is
   reachable under the page's CSP. The event layer is built with the intended
   event names and buffers locally; set `ANALYTICS_ENDPOINT` to a collector URL
   and events ship via `sendBeacon`.
3. **Web push.** No service worker, so no true push. `Notify` implements exactly
   two events — "misses you" after 36h and "shift complete" — with an `adapter`
   seam for a push backend, falling back to the local Notification API.

## Out of scope (deliberately)

Trading, PvP, multiple pets per user, email accounts, guilds, leaderboards,
real-money purchases, seasonal events.
