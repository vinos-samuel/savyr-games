# Game 01 — Marble Catapult (working name)

**Pitch:** Pull back a catapult to launch a marble at a target. Hit it and win
money that doubles every round. Spend money on more marbles with powers, and
merge same-color marbles into bigger, stronger ones.

**Main character:** The marbles — each color is a different power (speed,
shield so far).

**How you aim/shoot:** Catapult — pull back, release, marble flies in an arc.

**Round structure:**
- A level = 5 rounds.
- Round 5 of every level is a **boss round** — a boxing glove guards the hoop.
- Clear round 5 → move to the next level.

**The boss round glove — a punch, not a bob:** retracted (safe, tucked by
the backboard) → winding (a little shake, telegraphing what's coming) →
punching (extended over the rim, ~0.5s — this is the only phase that can
actually block a shot) → retracting → loops. Read the rhythm and time your
shot through the safe windows.

**Win a round:** Hit the target → earn money → clapping sound.
**Miss on a normal round (1-4):** No money, retry the *same* round → "dun dun
dun" fail sound.
**Blocked by the boss round's glove:** Just try the boss round again — the
glove is an obstacle to get past, not a fail condition on its own (unless
you have a Power Marble — see below).
**Actually missing the hoop on the boss round (5):** Lose the whole level,
back to round 1 → "dun dun dun" fail sound.

**The backboard is a real obstacle now, not just decoration.** Too much
power sails the marble past the rim and it can bank off the backboard's
face — reflects sideways like the ground bounce, and depending on the
angle it came in with, that bank can drop it straight into the hoop (a
trick shot) or knock it away into a miss. The red shooting-square on the
backboard is now a solid, opaque painted square (was a hollow outline).

**Economy:** Start with $0. First hit = $10. Every hit after that, the
payout doubles ($10 → $20 → $40 → $80...), resetting on a new level or a
level-losing miss.

**Marble Crates ($25):** Spend money to open a crate — reveals a random
cosmetic marble color and auto-equips it. Purely cosmetic, no color is
more powerful. Builds toward a "Colors: X/13" collection, and duplicates
are what you merge. A "Wear:" swatch row lets you switch back to any
previously-collected color at any time.

13 colors across 3 rarity tiers:
- Common (~10% each, ~70% combined): Green, Blue, Purple, Pink, Red, Yellow, Lime
- Rare (~6% each, ~24% combined): Cyan, Orange, Teal, Magenta
- Legendary (~2.7% each, ~5% combined): Gold, Silver

**Merge → Power Marbles:** 2 duplicates of any color (not necessarily
matching each other) merge into 1 Power Marble. The one power so far: on
the boss round, a Power Marble automatically shoves the glove aside if it
would have blocked you, and your shot keeps flying toward the hoop.

**Steady Aim ($40):** Pay for a shot that cannot miss the hoop. Overrides
however you actually pulled with a real computed arc from wherever you
released straight to the rim — it still flies through the air like a
normal shot (and can still get blocked by the boss glove, that's a
separate problem Power Marbles solve), it just always lands. Consumes
itself after one shot regardless of outcome.

**Champion's Arena (the big saved-up goal):** Lifetime earnings (separate
from spendable money — spending on crates/Steady Aim never sets this back)
builds toward $500. At $500, permanently unlocks a gold court + a 🏆 badge
next to the level label. Purely cosmetic, never resets.

**Saving:** Progress (level, round, money, lifetime earnings, arena unlock,
marble color, colors collected, Power Marbles) is saved to the browser's
local storage after every shot/crate/merge/purchase, and restored on load
— a refresh no longer wipes everything out.

**Ending:** Not decided yet — game may just keep getting harder, or have a
final boss. Revisit later.

---
### Build log
- Slice 1: static scene — slingshot, one marble, basketball hoop on screen.
- Slice 2: drag-to-aim, trajectory preview, launch physics, hit/miss, ghost
  trail on miss, clap/dun-dun-dun sounds, ball falls through and bounces on
  a hit, city skyline + basketball court aesthetics.
- Slice 3: rounds (5 per level, visible HUD, miss resets to round 1), money
  ($10 first hit doubling each hit, resets to $10 on new level/miss, never
  goes down), boss round (round 5) — a boxing glove bobs in front of the
  rim and blocks the shot if it's in the way, difficulty scaling (hoop
  moves further/higher and hit tolerance tightens each level, capped so it
  stays possible).

### Open assumptions to confirm with Savyr
- Money resets to $10 payout at the start of every level (not one giant
  doubling streak across the whole game) — flag if he pictured it differently.
- Boss round is currently the *only* difficulty knob at round level; between
  levels, the hoop also moves and gets less forgiving.

### Bug fixed: only the boss round should cost you the level
Originally coded so ANY miss reset you to round 1 — corrected so a miss on
rounds 1-4 just retries that same round; only failing the boss round (5)
sends you back to round 1 and resets the payout.

### Bug fixed: getting blocked by the glove isn't a "miss"
First pass made a glove block behave exactly like a boss-round miss (reset
to round 1). Corrected — being blocked just means try the boss round again;
only actually missing the hoop on the boss round resets you to round 1.

### Bug fixed: red backboard square was floating in the wrong spot
It's decorative (real backboards have one) but was positioned centered on
the rim instead of on the backboard, so it looked like a second target —
aiming at it always missed the actual hoop. Moved it onto the backboard.

### Bug fixed: level 2 was mathematically impossible
The difficulty scaling moved the hoop farther/higher each level, but the
catapult's power/pull range were never re-checked against that — level 2
onward had no pull that could reach the hoop at all, no matter how well
Savyr aimed. Fixed by simulating every pull strength/angle combo against
each level's hoop position (see the sim scripts we ran) and retuning:
`POWER` 10.5→12.5, `MAX_PULL` 95→100, and gentler per-level increments
(hoop distance/height/tolerance) that plateau after level 7 instead of
keeping shrinking forever. Verified levels 1, 2, and 7 all have a real,
non-hair-trigger reachable pull.

### Bug fixed: the boss glove was unbeatable by design
It bobbed ±30px but its own hit radius was ~31.6px — bigger than the bob
range — so it always reached back to cover its resting spot no matter
where in the cycle it was. There was never a real safe window. Reworked
into a punch state machine instead of a bob (see the Round structure
section above) — most of the cycle is now genuinely safe, with a short,
telegraphed danger window instead of near-constant overlap.

### Design update: money isn't purely cosmetic anymore
Earlier slices deliberately kept crates/shop cosmetic-only. Vinos and
Savyr asked for a paid gameplay edge (Steady Aim) and a gameplay-affecting
merge power (Power Marbles vs. the glove) — a deliberate change from that
earlier assumption, not an oversight. Cosmetics (crates, Champion's Arena)
and gameplay-affecting purchases (Steady Aim, Power Marbles) now coexist
on purpose.

### Feature: local storage persistence
Round, level, money, lifetime earnings, arena unlock, marble color, colors
collected, and Power Marbles now survive a page refresh. Saved after every
shot, crate, merge, and purchase under the key `marbleCatapultSave`.

### Bugs fixed: Power Marble, Steady Aim, backboard, and color-picker
Four things Savyr reported as "doesn't do anything":
- **Power Marble / Steady Aim** were actually wired correctly, but had zero
  feedback — the push-back message got overwritten by SWISH/MISS almost
  instantly, and Steady Aim's boosted tolerance was invisible. Added a
  distinct yellow particle burst + a message that survives to the final
  outcome ("⚡ POWER SAVE — SWISH!") for the glove push, and a pulsing
  dashed ring around the rim showing the actual boosted hit zone while
  Steady Aim is armed.
- **Choosing marble color** was never built — crates auto-equip whatever
  they reveal with no way back. Added a "Wear:" swatch row in the shop
  showing every collected color; click one to re-equip it.
- **Backboard bank shots were a real bug**, not just hard to hit: the
  collision only checked the ball's position at the end of each frame
  (not the path it traveled during the frame, unlike the rim/glove which
  already used a proper segment test) — a fast ball could tunnel straight
  past it. Worse, simulating the shot geometry showed the catch zone was
  positioned too high for how a shot naturally overshoots at scoring
  angles, so it was rarely reachable even when it didn't tunnel. Fixed
  both: switched to a segment-crossing test, and extended the catch zone
  down to ground level so a wider range of overpowered shots can bank.
  Verified with a simulation: reachable combos in the natural 30-60°
  scoring-angle range went from ~15/300 to 245/496.

### Open question for Savyr: should Champion's Arena grant a power?
Currently purely cosmetic (gold court + trophy badge), per the original
"cosmetics as the spend" request. Flagged since expectations may have
shifted now that Steady Aim and Power Marbles introduced paid gameplay
edges — worth deciding explicitly rather than assuming.

### Bug fixed: marble shop created a soft-lock
First pass made every shot consume a marble. Miss with $0 and 0 marbles left
= permanently stuck (can't shoot to earn money, can't afford a marble).
Fixed — shooting is free and unlimited again; buying marbles builds a
collection for merging/powers later, it doesn't gate play.

### Bug fixed: only the boss round losing the hoop resets the level
(Same class of bug as above, caught in the same review pass — see the two
"only the boss round" entries above for the two separate corrections:
normal-round misses don't reset, and glove blocks don't reset.)

### Polish pass: "modern game" pass (Tier 1 + Tier 2 + 3D buildings)
Was reading as flat/early-2000s. Added, all in plain Canvas 2D (no build
tools, no external assets):
- Embedded the Baloo 2 font (same one from the orientation page) for all
  in-game HUD/message text, instead of the system font
- Radial-gradient shading on the marble, rim, and glove so they read as
  round objects, not flat stickers
- A soft shadow under the marble that shrinks/fades with height — also a
  free depth cue for how high the shot is
- A confetti burst at the rim on every SWISH
- Squash-and-stretch on the marble (stretches in flight, squashes on
  bounce) plus a subtle spin so rolling/flying actually reads as motion
- Result messages (SWISH/MISS/BLOCKED) pop in with an ease-out-back bounce
  instead of snapping into place
- The money HUD ticks up to the new total instead of jumping instantly
- Restyled the shop bar to match the game (gradient marble icon, gradient
  button with a pressed state, drop shadow) instead of looking like a
  plain web form
- City skyline buildings are now pseudo-3D — each has a shaded roof and
  side face (like a cheap isometric illustration trick), with small gaps
  between buildings so the extrusion is actually visible instead of
  hidden behind the next building's front face

### Economics redesign: crates + lifetime savings goal
Replaced the flat "$10 buy a marble" placeholder with three ideas Savyr
picked (collect-and-reveal, cosmetics-only spend, save toward one big
thing), combined into one system — see the Economy section above for the
mechanics. Key design choice: spendable money and lifetime earnings are
tracked separately, so opening crates never sets back progress toward
Champion's Arena — spending and saving both feel good at the same time,
nothing to feel bad about.

### Bug fixed: dragging didn't work on a phone/tablet at all
Reported as "I tapped it and nothing happened." The touch handlers never
called `preventDefault()`, so the browser's default touch behavior (scroll
the page) hijacked the drag gesture before the game ever saw it move —
this only shows up on an actual touchscreen, not a mouse, which is why it
tested fine on a laptop. Fixed three ways together, since any one alone
isn't fully reliable across browsers: `touch-action: none` on the canvas,
`preventDefault()` in the touch handlers, and registering the touch
listeners as `{ passive: false }` (Chrome defaults touch listeners to
passive for scroll performance, which silently ignores `preventDefault`
unless you opt out explicitly). Verified by dispatching a real touch
sequence — drag now tracks correctly and launches on release.

### Design change: Steady Aim redone as "can't miss," not "more forgiving"
The tolerance-boost version was reported as "nothing happens — it just
gives a glowing zing." Not a bug, a design miss: a 1.5× tolerance widening
is nearly invisible in practice, and the only feedback was a decorative
ring. Replaced with a real mechanic — on release, if Steady Aim is armed,
the shot's velocity is recomputed from scratch (ignoring whatever was
actually pulled) to land a real, physically-simulated arc straight in the
rim, launched from wherever the marble was released. Verified with a
deliberately terrible pull (tiny distance, aimed away from the hoop
entirely) — still a guaranteed SWISH.

### Feature: 6 more marble colors (7 → 13)
Added Red, Yellow, Lime (common), Teal, Magenta (rare), and Silver
(legendary) — see the Marble Crates section above for the full list and
odds. Kept the same three-tier rarity shape, just wider. Along the way,
fixed a real bug found while testing: the marble's shading gradient had a
hardcoded rose highlight and dark-red shadow left over from when the
marble was always red — every other color was rendering muddy underneath
its shading. Now the highlight/shadow are computed from the marble's
actual color (`shadeColor()`, the same helper the buildings use), so all
13 colors shade correctly. Verified visually across a dozen colors in the
"Wear:" row.
