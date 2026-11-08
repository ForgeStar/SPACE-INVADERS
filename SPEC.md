# SPACE INVADERS — Scope Lock

**One sentence:** Classic 1978 Taito Space Invaders — move, fire, defend the bunkers, watch the heartbeat speed up — single-file HTML/JS in the collection's cabinet style.

## Core loop
Move cannon → shoot invaders (30/20/10 pts by row) → formation steps side to side, drops a row at each edge, speeds up as it dies → enemy zigzag bullets + UFO flybys → hit a bunker or the cannon → 3 lives → formation reaches the ground or you run out of ships → game over with hi-score.

## MVP features
- Cannon: ◄/► or A/D move (clamped to play field), Space fire, **max 1 player bullet on screen** (original rule)
- Formation: 11×5 grid — squid row **30**, crab rows **20/20**, octopus rows **10/10**; 2-frame animation; steps 12 px/side, drops 14 px at each edge; **4-note heartbeat** (E3 D3 C#3 C3) that speeds up from 0.52 s/step (full) to 0.09 s/step (last invader); each wave starts one row lower
- Bunkers: 4 destructible shields (22×16 cell grids, diamond-radius craters), hit by BOTH player and enemy bullets
- Enemy fire: max 3 zigzag bullets on screen, fired from a random column's bottom-most survivor
- UFO: warbles across the top, 50/100/150/300 pts random, bonus flashes in the HUD like the original
- Wave clear → next wave (faster steps + faster enemy bullets); each new wave respawns fresh bunkers-free formation (bunkers persist from wave 1, like the original)
- Lives: 3 ships, extra ship every 10,000 pts, 2 s blinking invulnerability on respawn
- Invasion (formation bottom reaches the shield line) = instant game over, "THE INVADERS LANDED"
- Score popups at destruction point, hi-score in `localStorage` (`spaceinvaders.hiscore`)
- Attract screen: HIGH SCORE up top, title, classic score chart, drifting saucer, blinking "PRESS ANY KEY"
- Game over with 1.2 s input lockout (mashing fire doesn't restart), pause (P), mute (M), CRT cycle (C)
- Cabinet: Asteroids bezel placeholder (same 1024×1024 panel + screen hole — new Space Invaders art coming) + the collection's WebGL CRT glass shader
- Audio: Web Audio — 4-note invader heartbeat, fire blip, invader pop, bunker thud, ship explosion, warbling UFO, extra-ship jingle
- Touch: tap = start/fire, drag sideways = steer

## Controls
◄/► or A/D move · Space fire · P pause · M mute · C CRT · Any key/tap start

## Tech
Single `index.html`, vanilla JS + 2D canvas, no build step. Sprites are hand-authored 2-frame bitmaps pre-rendered to offscreen canvases at DPR. CRT shader + audio patterns copied from the Asteroids build (itself from LANDER/PONG).

## OUT OF SCOPE
- 2-player mode, attract-mode AI demo play
- Vertical screen (current bezel hole is landscape 1.064:1; re-layout is easy later when the new bezel art lands)
- Final Space Invaders bezel art (placeholder until Rex's new art is ready — glow mask must be re-baked with it)
- Mobile steering polish

## Tasks
- [x] 1. SPEC + asset copy (Asteroids bezel/plexi/reflect placeholders)
- [x] 2. Shell: splice game section into the Asteroids cabinet (title/meta/hint/header)
- [x] 3. Cannon + formation: movement, stepped march, animation, heartbeat, edge drop
- [x] 4. Bunkers: grids, craters, both bullet types
- [x] 5. Bullets: 1 player / 3 enemy zigzag, collisions, popups
- [x] 6. UFO + waves + invasion game over
- [x] 7. Lives, extra ship, attract screen, game over lockout
- [x] 8. README, NOTES, commit, preview, verify both render paths
