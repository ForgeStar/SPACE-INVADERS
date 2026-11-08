# Space Invaders — project notes (for context recovery)

Single-file HTML5 game in `index.html` (~142KB). Same cabinet format as
`E:\Bionic Projects\Games\Asteroids` — that project's `NOTES.md` is the deep
reference for the glow/CRT/plexi pipeline. This game reuses its bezel,
shaders, and mask verbatim; only the game section differs.

## Status
- 2026-11-07: game complete, committed. Bezel is the **Asteroids placeholder**
  on purpose — Rex is making new Space Invaders bezel art. When it lands:
  swap `bezel.webp`/`bezel.png` and **re-bake the glow mask** (see below).

## Key files
- `index.html` — the whole game + shaders. One inline `<script>` at end of body.
  Structure: CSS shell → DOM → script (game section → CRT WebGL shader →
  plexi → glow) . The game section runs from `const W = 580` to just before
  `// ---------------- CRT glass`.
- `bezel.webp`/`bezel.png` — **Asteroids art placeholder.** The webp has the
  intentional veil alpha (screen-hole region ~80% opaque so baked reflections
  show over the game). **Never zero that alpha.** Same 1024×1024 panel, same
  squircle hole (screen rect left 186, top 201, 688×647) — that's why the
  screen CSS, glow mask, and shaders work unchanged.
- `reflect.webp` — additive pulsing reflection overlay (copied from Asteroids).
- `plexi.webp` — worn-glass texture for the TEMP disk-load line:
  `document.getElementById("plexi").src = "plexi.webp";` (line ~1062). The
  embedded `PLEXI_TEX` data URL is still the Krea texture; the TEMP line means
  disk edits show on reload. Revert to `PLEXI_TEX` when the texture is final.
- `ref/` — MAME-style reference from Rex (`invaders_bezel.png` 1920×1080
  cabinet photo, `vert_screen_mask.png` 812×1080 vertical screen shape,
  `default.lay`, `invaders.cfg`, `si_characters.eps`). **Reference only** —
  the game format is the 1024×1024 bezel panel. `vert_screen_mask.png` hints
  the new bezel may have a VERTICAL screen; the current layout is landscape
  580×545 to match the current hole. Re-layout is a constants-only job
  (see Game layout below).
- `.gitignore` — same as Asteroids: `ref/`, `*.png` except `!preview.png`.

## Game layout (all in the game section, top of the script)
- Canvas 580×545 @ DPR 2 (same window as Asteroids' bezel hole).
- Play field x 30–550. Formation: 11 cols × 5 rows, COL_GAP 38, ROW_GAP 30,
  sprites 11×8 px @ PIX 2.5. Top y=84 (+16 per wave, cap +96). Steps 12 px,
  drops 14 px at the edges (~4 steps per direction — matches the original's
  proportions).
- Bunkers: 4 shields at x-centers 125/235/345/455, top y=428, 46×34 screen px
  over a 22×16 cell grid. Invasion line INVADE_Y = 432 (formation bottom).
- Player: top y=494, x clamped 50–530, speed 280 px/s.
- UFO: y=62, 95 px/s, scores 50/100/150/300.
- HUD: "1UP" + score top-left, HIGH SCORE top-center, UFO bonus flash left of
  the hi-score, ship icons bottom-left, "WAVE n" bottom-right.
- Heartbeat: NOTE_FREQS = E3/D3/C#3/C3 (164.81/146.83/138.59/130.81 Hz),
  step interval 0.52 s (full) → 0.09 s (last invader), ÷(1+(wave-1)*0.05).
- Hi-score key: `spaceinvaders.hiscore`.

## Glow — state
`GLOW_FALLBACK_MASK` is the **Asteroids** baked mask (same bezel, so correct
for now). When Rex's new bezel art lands:
1. He re-cuts the vis (R = plastic mask, 512×512) as before.
2. Re-bake pipeline = Asteroids NOTES.md §"Re-baking the mask" (chamfer
   distance, pack R/G, decode+compare in run_python, replace the
   `GLOW_FALLBACK_MASK` base64 in index.html — Python only, never paste it
   through edit_file_tool).
3. If the screen hole moves/changes shape: update the CSS `#screen` rect,
   the COMP_FS screen rect in the glow section, and the canvas W/H + layout
   constants. `vert_screen_mask.png` in ref/ suggests vertical is possible.

## CRT shader / plexi
Identical to the Asteroids build (copied verbatim): Glass TV mode default,
COMB 0.5, procedural glass reflection OFF (baked `reflect.webp` instead),
plexi `PLEXI_AMT 2.2`. No `int` in fragment shaders (file:// WebGL taint →
all textures embedded base64 or disk-loaded for the TEMP line).

## Verification (done 2026-11-07)
- `node --check` on the extracted inline script: OK.
- Headless Edge screenshot WITH GPU: attract screen correct (title, score
  chart, hi-score).
- Headless Edge `--disable-gpu`: identical result (dual-path rule from
  Asteroids NOTES.md; shaders copied verbatim so risk was low).
- Auto-start test copy (load → startGame after 150 ms, screenshot from
  scratchpad): full 11×5 formation, both anim frames, 4 bunkers, cannon,
  lives, WAVE 1, HUD, scanlines all rendering.

## Rex's preferences
- Casual, quick, direct tone. One source of truth.
- Identical behavior in Bionic in-app browser AND regular browser (file://).
- No debug UI / toggle buttons.
- Hi-score at top on start screen; 1.2 s game-over input lockout.
- Krea credits scarce — local tools first.
- He re-cuts bezel alpha/masks himself when he wants.

## Environment quirks
- run_python sandbox: no numpy; Pillow yes. Stage host files via stage_files;
  outputs land in scratchpad, move with the `copy` tool.
- Git Bash warns "could not find /tmp" — harmless.
- Headless Edge: `msedge --headless=new --user-data-dir=<scratch> 
  --window-size=1280,1024 --hide-scrollbars --virtual-time-budget=8000 
  --screenshot=... file:///...`. GPU path and `--disable-gpu` path.
- Don't paste the huge base64 strings (PLEXI_TEX, GLOW_FALLBACK_MASK) into
  edit_file_tool — Python only.
