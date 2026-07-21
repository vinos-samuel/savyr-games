# Savyr's Game Lab

Vinos and his son Savyr (7.5) build mini games together here, using Claude Code.
Vinos drives the keyboard; Savyr is the designer. Goal: teach Savyr how a game
gets built by actually building his own, one small step at a time.

## How a session works
Say **"let's build a game"** (or `/build-a-game`) to start the interview-and-build
workflow. Full process lives in `.claude/skills/build-a-game/SKILL.md` — read it
before running a session.

## Stack — kept deliberately boring
- **Single HTML file per game.** HTML + CSS + JS, no build step, no npm, no
  frameworks. Open it and it runs — that's the whole point for a 7-year-old.
- `<canvas>` for anything with movement/action (like Plants vs Zombies style).
- Plain DOM (buttons, divs) for card/board games (like Uno, Monopoly, chess).
- Only reach for a second file (or a library) if a game genuinely needs it —
  don't add tooling Savyr can't see the point of yet.

## Folder structure
```
Games/
  GAMES_LOG.md          <- one entry per game: name, idea, date, what he learned
  01-<game-slug>/
    index.html           <- the whole game
    GAME_CARD.md          <- the one-page design doc from the interview
  02-<game-slug>/
    ...
```
Number folders in build order so it reads like a portfolio over time.

## Ground rules for Claude
- Explain every coding concept in a kid analogy tied to something he already
  loves (chess, Lego, Uno, PvZ) — never jargon first.
- Build in the smallest possible playable increments. Get something on screen
  in the first few minutes, every session — momentum matters more than scope.
- Let Savyr drive decisions (colors, names, rules, what's "too easy/too hard").
  Vinos and Claude execute; Savyr designs.
- After each game, append an entry to `GAMES_LOG.md`.

## Current Sprint — Game 01: Marble Catapult (`01-marble-catapult/`)

**Completed this session:**
- Slingshot-aim + basketball-hoop core loop: drag-to-pull, trajectory preview, physics flight, hit/miss detection, ghost trail on miss.
- Rounds/levels: 5 rounds per level, round 5 is a boss round, miss on a normal round retries same round, miss on boss round resets to round 1.
- Difficulty scaling per level (hoop distance/height/tolerance), tuned via simulation after level 2 was found mathematically unreachable.
- Money economy: $10 first hit doubling per hit, resets on new level/boss-round loss.
- Marble Crates ($25): random cosmetic color reveal, weighted rarity, builds a 7-color collection.
- Merge: 2 duplicate colors → 1 Power Marble; Power Marble auto-shoves the boss glove aside if it would've blocked a shot.
- Steady Aim ($40): boosts hoop hit-tolerance 1.5x for one shot, then consumes itself.
- Champion's Arena: lifetime-earnings meter (separate from spendable money) unlocks a permanent gold court + trophy badge at $500 — deliberately cosmetic only, per Vinos/Savyr's explicit "cosmetics as the spend" call.
- Backboard: real bank-shot collision (ball can reflect off it into the hoop or away), opaque shooting-square fill.
- Boss glove reworked from a "bob" (mathematically unbeatable — its hit radius exceeded its bob range) into a punch state machine: retracted → winding → punching (only phase that blocks) → retracting.
- localStorage persistence (`marbleCatapultSave` key) for round/level/money/arena/colors/power marbles.
- "Modern game" polish pass: gradient shading, drop shadows, confetti particles, squash/stretch + spin on the marble, ease-in message pop, animated money counter, restyled shop UI, pseudo-3D extruded city-skyline buildings.
- Color-picker swatch row ("Wear:") so a previously-collected color can be re-equipped, not just whatever the last crate revealed.

**Playtest-round bug fixes, all individually verified** (merge/crate/steady-aim via direct function calls, shot outcomes and glove timing via manually-driven physics frames since automated browser tabs throttle `requestAnimationFrame` when not OS-focused):
1. Merge no longer deletes a color from the collection — `discoveredColors` (only grows) is now separate from `colorCounts` (the mergeable duplicate counter).
2. Steady Aim originally just widened hit tolerance 1.5x with a decorative ring — reported as "nothing happens." Redesigned per explicit request into a real "can't miss" mechanic: on release, if armed, velocity is recomputed from scratch into a physically-simulated arc from wherever the marble was released straight into the rim, ignoring the actual pull. Verified with a deliberately terrible pull (wrong direction, tiny distance) — still guaranteed SWISH.
3. Idle marble jitter fixed — `finishShot`'s reset now zeroes `flightVx`/`flightVy`/`marbleRotation` when a shot resolves.
4. Boss glove now escalates by level — `gloveCycle()` shrinks the safe "retracted" window and lengthens the "punching" window as `levelNum` increases, capped at level 7 (same plateau as hoop difficulty).
5. Persistence re-verified working end-to-end (save → reload → all fields restored). Not a code bug — if it still resets, check `file://` vs. the local server, or a private/incognito window.
6. Dragging didn't work on a phone/tablet at all ("I tapped it, nothing happened") — touch handlers never called `preventDefault()`, so the browser's native scroll gesture hijacked the drag before the game saw it move. Fixed with all three parts needed together: `touch-action: none` on the canvas, `preventDefault()` in the handlers, and `{ passive: false }` on the touch listener registration (Chrome silently ignores `preventDefault` on passive listeners). Verified by dispatching a real touch sequence.

**Marble colors expanded 7 → 13**: added Red, Yellow, Lime (common), Teal, Magenta (rare), Silver (legendary). Found and fixed a real bug while doing this — the marble's shine/shadow gradient had a hardcoded rose-highlight/dark-red-shadow left over from when the marble was always red, so every other color was rendering muddy underneath its own color. Now highlight/shadow are computed from whichever color is equipped (`shadeColor()`, same helper the buildings use).

**Where we stopped:** Vinos reported the shop still showing "7/7" after the color expansion. Checked the file on disk — `MARBLE_COLORS` correctly has all 13 entries and the display line reads `MARBLE_COLORS.length` dynamically, not a hardcoded 7. Almost certainly a stale browser tab holding the old JS in memory from before the edit — told him to hard-refresh (Cmd+Shift+R) or reopen the tab.

**Follow-up session — design change + 3 new features:**
1. **Boss-round miss no longer costs the level.** Vinos/Savyr called this out directly: missing on the boss round used to reset `roundNum` to 1 (lose all progress in the level). Now a boss miss just retries the boss round, same as any other miss — removed that branch from `finishShot()`. The only remaining `roundNum = 1` is the legitimate "beat the boss, advance a level" reset.
2. **Crates no longer auto-equip.** `openCrate()` used to call `equipColor()` on every pull, silently switching whatever marble you were shooting with. Now a crate only adds the color to your collection — you equip a color deliberately via the swatch row.
3. **"New color" message bug fixed.** `isNew` was checking `colorCounts[name]` (which merging zeroes out), so a color you'd already found could show "New color!" again after you merged its duplicates away and pulled it a second time. Now checks `discoveredColors` (the permanent record) instead.
4. **Crate lock once the collection is truly finished.** New `collectionComplete()` = own every color AND no mergeable duplicates left (`discoveredColors.size === MARBLE_COLORS.length && !hasAnyMergeablePair()`). Once true, `Open Marble Crate` disables permanently (and `openCrate()` itself no-ops as a second guard) — confirmed by Savyr as the intended end state, not a bug.
5. **Boss variety by level** — `bossTier()`: levels 1-3 unchanged (single glove, single punch), levels 4-6 the glove punches twice per cycle (added a `rewind` state between two `punching` phases), levels 7+ two gloves at once (second glove offset in both position — guards a bit below the rim — and clock phase, so they don't punch in lockstep). Refactored the old singular `gloveState`/`gloveTimer` into a `gloves` array (`ensureGloves()` rebuilds it when the tier's glove count changes) with `gloveCenterFor()`/`dangerousGloveCenters()` replacing the old single-glove functions. A Power Marble shove now clears every glove for that shot, not just one.
6. **Fire Trail cosmetic** — second lifetime-earnings milestone (`TRAIL_GOAL = 1000`, above the $500 arena goal), unlocks a continuous ember-spark particle trail behind the marble during flight (`spawnTrailSpark()`, called each `stepFlight()` step). Same "cosmetic only, tied to lifetime earnings" pattern as the arena. New progress bar in the shop UI under the arena bar.
7. **Shiny marble variants** — 1-in-100 chance (`SHINY_CHANCE`) per crate pull, independent of color rarity. Tracked as `discoveredShinyColors` (separate from `discoveredColors`) and `marbleIsShiny` (separate from `marbleColor`) so a shiny is a distinct equippable variant of a color, not a replacement for the normal one. Gets its own swatch (gold ring, CSS `.swatch.shiny`) next to the normal swatch for that color, and a rotating foil-shimmer streak drawn across the marble/shop icon when equipped. All new state persisted in `saveGame()`/`loadGame()` with safe fallbacks for older saves.

All 7 changes verified directly (function calls + forced RNG/state, not visual playtesting) since automated tabs throttle `requestAnimationFrame` — see testing note below.

**Next:** possibly moving GitHub/Vercel deploy so Savyr can share a link with friends (in progress); after that, marble powers beyond the single push-back and an ending/win-state decision (still undecided per the Game Card).

**Architecture decisions worth remembering:**
- Collision checks must use segment-crossing tests (prev position → current position), not point-in-region tests — the backboard shipped with the latter first and a fast ball tunneled straight through it before the fix.
- Money is intentionally split into two pools: spendable (crates, Steady Aim) vs. lifetime earnings (Champion's Arena) — spending never sets back the arena goal, by design.
- Cosmetic vs. gameplay-affecting purchases now deliberately coexist (Steady Aim and Power Marbles are real gameplay edges; crates/arena are cosmetic-only) — a conscious change from the original "everything's cosmetic" rule.
- Collected colors vs. mergeable duplicates are two separate pieces of state on purpose (`discoveredColors` set vs. `colorCounts` map) — anything that's "have you ever gotten this" should never shrink, only counters of "how many spare ones to spend" should.
- Any per-color or per-object visual effect (shading, highlights) must derive from that object's actual color/state at render time, not a hardcoded value from whatever the first version happened to be — the marble gradient bug is exactly this class of mistake.
- Mobile touch support needs all three of `touch-action: none`, `preventDefault()`, and `{ passive: false }` together — missing any one silently breaks drag-to-aim on a real touchscreen even though mouse input on a laptop works perfectly.

**Testing note for future sessions:** automated browser tabs (both the old preview tool and Claude-in-Chrome) report `document.hidden = true` since they're not the OS-focused window, and Chrome throttles `requestAnimationFrame` to near-zero for hidden tabs — this stalls the game's physics/animation loop during automated testing even though it runs fine at 60fps for an actual focused player tab. Work around it by manually driving frames: call `stepFlight(t)` or `draw(t)` directly in a loop with hand-incremented timestamps instead of relying on the real rAF loop.
