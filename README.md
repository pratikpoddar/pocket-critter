# Pocket Critter

A mobile-first virtual pet for grown-up 90s kids. Adopt an original critter, feed
it, play with it, send it to work, and watch its personality develop over weeks.

**The pet never dies.** Neglect makes it grumpy, droopy and guilt-inducingly
dramatic — never dead. This is a comfort product, not an anxiety machine.

Everything is original: art, names, copy, code. No borrowed species, sprites,
fonts or UI.

## Run it

It is one self-contained HTML file with no dependencies and no toolchain. The
only build step is a 40-line shell script that adds a `<head>` for self-hosting:

```bash
open pocket-critter.html      # the source, straight from disk
./build.sh && open index.html # the self-hosted build, with analytics
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

## Hosting: two outputs, one source

`pocket-critter.html` is the source of truth and is deliberately **headless** —
no `<!doctype>`, `<html>`, `<head>` or `<body>` — because a published Claude
artifact supplies that skeleton itself.

`./build.sh` wraps it into `index.html` for GitHub Pages, adding the one thing
an artifact cannot have: a real `<head>`. Everything host-specific lives in that
script — the `og:`/`twitter:` tags and the GoatCounter snippet — so the artifact
build stays clean and the hosted build gets what only a real domain can support.

```bash
./build.sh    # pocket-critter.html -> index.html
```

Edit `pocket-critter.html`, never `index.html`; the latter is generated and
carries a banner saying so.

## Analytics

GoatCounter, on the self-hosted page only: the published artifact's CSP admits
no analytics host, and `gcSend` no-ops there rather than pretending.

- **Pageviews** are counted per screen (`/home`, `/games`, `/shop`, `/room`,
  `/pet`, `/adopt`, `/visit`). `count.js` is loaded with `no_onload` because the
  `?pet=xxxxxxxx` deep link would otherwise register one distinct path per pet
  visited and bury the real routes.
- **Custom events** are sent as `ev-<name>`: `adopt`, `feed`, `play_game`,
  `job_start`, `job_collect`, `buy`, `share_click`, `postcard_make`,
  `visit_from_link`, `gift_sent`, `gift_seen`, `personality_set`, `tuck_in`.
- Every event is also buffered in `localStorage` (last 400), and
  `analyticsSummary()` rolls up the two numbers that matter — shares per user
  and visit-to-adopt — on the Critter card.
- `ANALYTICS_ENDPOINT` remains a second, independent sink for any collector that
  accepts a JSON POST body.

`count.js` refuses to count from `localhost`, so local development does not
pollute the data.

## Known gaps

1. **Per-pet Open Graph titles.** The site unfurls with static tags and
   `og-card.png` (rendered by the game's own postcard code). A per-pet unfurl —
   "Biscuit the Lampmoth needs soup" on a `?pet=` link — needs server-side
   rendering, which a static host cannot do.
2. **Web push.** No service worker, so no true push. `Notify` implements exactly
   two events — "misses you" after 36h and "shift complete" — with an `adapter`
   seam for a push backend, falling back to the local Notification API.

## Out of scope (deliberately)

Trading, PvP, multiple pets per user, email accounts, guilds, leaderboards,
real-money purchases, seasonal events.
