# SPACE INVADERS — Classic Arcade Clone

A single-file clone of Taito's 1978 **SPACE INVADERS** in vanilla JS + canvas, rendered inside the collection's cabinet bezel with the CRT glass shader (barrel distortion, scanlines, reflections).

## Play

- Open [index.html](index.html) — that's the whole game (no build step, no dependencies).
- ◄/► or A/D: move · Space: fire · Any key: start
- Touch: tap to start · tap screen to fire · drag sideways: steer
- P: pause · M: mute · C: cycle CRT glass mode (Off / Clean)

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

- Bezel (`bezel.webp`/`bezel.png`) is the **Space Invaders cabinet art** — a 1024×1024 panel: the title, UFOs and invader creatures up top, a starfield, a retro TV frame around the screen, lunar terrain below, and the classic score table. It keeps the collection's squircle screen hole, so the glow/plexi/reflect/shader pipeline works unchanged. The webp carries an intentional veil alpha over the hole (~80% opaque) so the baked reflections show over the game — don't zero it.
- Ray gun (`Bezel_Raygun.webp`): a red ray-gun cutout (the panel's bottom-left quadrant, alpha + baked drop shadow). On the real cabinet it was a separate cardboard piece under the glass, so it sits above the bezel art and below the glass layers.
- Sprites are hand-authored 2-frame bitmaps (classic shapes), pre-rendered to offscreen canvases at 2× for crisp `drawImage`.
- Plexiglass layer: worn clear-plastic texture — curved stress cracks, hairline scratches, dust — rendered in the CRT shader and lit from below, so only glowing screen pixels reveal the scratches.
- Glow mask (`GLOW_FALLBACK_MASK`) is pre-baked from the SI squircle hole (`ref/ScreenHole.png`); re-bake it if the hole ever moves (pipeline in `NOTES.md`).

## GitHub Pages

This repo is deployed as-is from the root, so the game lives at:

    https://forgestar.github.io/SPACE-INVADERS/
