# Canon — Web `/m/<id>` message-receipt page & the compass arrival

_Last updated: 2026-06-26. Repo: pointward-website (serves pointward.app via GitHub Pages)._

## The page
- **`404.html` is the only file that renders `pointward.app/m/<id>`.** GitHub Pages serves
  it for unmatched paths; the inline script self-parses `/m/<id>` (`getMessageId()`).
- All CSS/JS is **inline in `404.html`**. There is **no shared/global stylesheet** in the
  repo (`index.html`, `privacy.html` have their own inline styles). A change inside
  `404.html`'s `<style>` **cannot** affect any other page. Scope new styles to a
  page-local class; never edit `body`/`:root`/the global `prefers-reduced-motion` rule.

## How a thought arrives (data flow)
1. `load()` → `fetch` anon RPC `get_message(p_id)` on Supabase ref `jlbgdlgwtrkmqcfnomlr`.
2. `render(msg)` sets the sender name and calls `mountTeaser(msg.instrument, …)`.
3. **`msg.instrument` is a `SenderStyle.rawValue`** (the wire style), NOT the instrument
   name. The HomeLink app's `createAndShareLink(instrument: style.rawValue)` writes it.

## ⭐ The instrument → teaser mapping (the thing that's easy to get wrong)
`mountTeaser` looks up `msg.instrument` in the `TEASERS` registry. Keys MUST equal
`SenderStyle.rawValue`. Each instrument in the app maps to a wire style via
`Instrument.senderStyle` (HomeLink `Instrument.swift:71`):

| App instrument | wire `instrument` value | web core |
|---|---|---|
| **compass (free, vintage brass)** | **`glow`** | **`coreGlow`** ← the COMPASS arrival |
| bow | `bowArrow` | `coreBow` |
| flick | `fingerFlick` | `coreFlick` |
| rocket | `rocket` | `coreRocket` |
| wind | `firefly` | `coreFirefly` |
| wand | `wand` | `coreWand` |
| plane | `plane` | `corePlane` |
| birthday | `birthday` | `coreBirthday` |
| firework | `firework` | `coreFirework` |
| (unknown / legacy) | — | `coreGeneric` |

**Key fact:** the compass instrument travels as **`glow`** (`Instrument.compass.senderStyle
== .glow`). So **the compass web-arrival animation lives in `coreGlow`** — there is no
`compass` key. (There is no `shootingStar` core either → falls to `coreGeneric`.)

To preview a specific core with no fetch: `?demo=<wireKey>` (e.g. `?demo=glow` = compass,
`?demo=rocket` = rocket, `?demo=generic` = fallback). `?teasertest=1` runs the self-test.

## What `coreGlow` renders now (as of 2026-06-26, commit 2e2b033)
A **vintage brass compass** matching the app's `VintageFaceView` (HomeLink
`SkinFaceView.swift`): burnished face `#221a0e→#120d06`, triple brass ring, 15° ticks +
30° degree numbers + N/E/S/W + intercardinals at the app's radii (92/73/54), surveyor
crosshair, dashed inner ring. Needle = gold `#FFD700` north / aged-brass `#8b6914` south;
it seeks, overshoots, snap-settles on north, then a **lock "wow"**: gold bloom flash +
double expanding lock-ring + emoji pop. Pure CSS animation (`.mc-*` classes + `mcFaceMarks()`).
Brass palette is **scoped to `.mc-*`** — page background stays `#0d0d14`. Old glow-orb
core preserved in git history (commit `e7ca95a`).

## Invariants — do not break
- Preserve the anon `get_message` fetch/display.
- The page must continue to **NOT mark the message opened** (a web glance ≠ an in-app read;
  see the comment in `render()`).
- Install CTA: keep the live TestFlight link `…/join/rjAS4cnk` and its wording
  ("Open Pointward" / "get Pointward" / "free · no ads · for real") unchanged.
- Per-instrument cores (rocket etc.) are independent — change one without touching others.
- Clipboard bridge: install taps copy `https://pointward.app/m/<id>` (cold-receive path).

## History (so we don't repeat the wrong runs)
- A first attempt added a full-page compass **background** + edited the kicker — reverted
  (commit `e7ca95a`) as "too weak / wrong shape."
- A second attempt replaced the **whole** teaser stage with the compass, so EVERY
  instrument (incl. rocket) wrongly showed it — fixed in `2e2b033` by reverting all
  instruments to their original cores and putting the compass only in `coreGlow`.
