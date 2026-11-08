# SPACE INVADERS — Classic Arcade Clone

A single-file clone of Taito's 1978 **SPACE INVADERS** in vanilla JS + canvas, rendered inside the collection's cabinet bezel with the CRT glass shader (barrel distortion, scanlines, reflections).

## Play

- Open [index.html](index.html) — that's the whole game (no build step, no dependencies).
- ◄/► or A/D: move · Space: fire · Any key: start
- Touch: tap to start · tap screen to fire · drag sideways: steer
- P: pause · M: mute · C: cycle CRT glass mode (Off / Arcade / Glass)

## Mechanics (per the 1978 original)

- **Invaders** in an 11×5 formation: squid row **30**, crab rows **20**, octopus rows **10** points. They march side to side with a 2-frame animation, drop a row at each edge, and the 4-note **heartbeat** speeds up as they die.
- **Bunkers**: four destructible shields with cratering — both your shots and their shots chew them down.
- **Fire**: max 1 of your bullets on screen (original rule); up to 3 enemy zigzag bullets at once, fired from the bottom-most invader in a random column.
- **UFO** warbles across the top for 50/100/150/300 points; the bonus flashes by the hi-score like the original.
- **Waves**: clear the formation for the next wave — it starts a row lower and fires faster. If the formation reaches the ground, **invasion**: instant game over.
- **Ships**: 3 to start, an extra ship every 10,000 points. A destroyed ship respawns at center with 2 s of blinking invulnerability. Remaining ships shown bottom-left.
- **Score popups** appear where things die, hi-score persists in `localStorage`.
- **Sound**: Web Audio — the 4-note invader heartbeat, square-wave fire blips, invader pops, filtered-noise explosions, warbling UFO, extra-ship jingle.

## Art

- Bezel (`bezel.webp`/`bezel.png`) is the **Asteroids placeholder** — same 1024×1024 panel and screen hole, so the glow/plexi/reflect/shader pipeline works unchanged. New Space Invaders bezel art is coming; when it lands, the glow fallback mask (`GLOW_FALLBACK_MASK`) must be re-baked (pipeline in `NOTES.md`).
- Sprites are hand-authored 2-frame bitmaps (classic shapes), pre-rendered to offscreen canvases at 2× for crisp `drawImage`.
- Plexiglass layer: worn clear-plastic texture — curved stress cracks, hairline scratches, dust — rendered in the CRT shader and lit from below, so only glowing screen pixels reveal the scratches.

## GitHub Pages

This repo is deployed as-is from the root, so the game lives at:

    https://forgestar.github.io/Space-Invaders/
