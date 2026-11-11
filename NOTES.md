# Space Invaders — project notes (for context recovery)

Single-file HTML5 game in `index.html` (~142KB). Same cabinet format as
`E:\Bionic Projects\Games\Asteroids` — that project's `NOTES.md` is the deep
reference for the glow/CRT/plexi pipeline. This game reuses its bezel,
shaders, and mask verbatim; only the game section differs.

## Status
- 2026-11-07: game complete, committed.
- 2026-11-08: **New SI bezel art is in** (`bezel.png`/`bezel.webp`, swapped by
  Rex; screen hole unchanged — same squircle rect). The bottom bezel art
  (mountains/volcano) overlaps the bottom ~10% of the screen, so the player
  row was raised 55 px (see Game layout). Glow mask re-baked from
  `ref/ScreenHole.png` (SI squircle) — see Glow section.
- 2026-11-08: tested an opaque-bezel / game-on-top variant (black `#panel`
  background + z-index re-stack, then `mix-blend-mode: screen` on `#screen`).
  Rejected by Rex — reverted, original hole stack restored.
- 2026-11-08: attract screen — score chart removed (the cabinet bezel art
  already shows the points chart); "PRESS ANY KEY" prompt moved to screen
  middle (H/2-8, hints line H/2+22).
- 2026-11-08: sprite phosphor bloom added as a separate `#bloom` canvas
  (CSS downsample-blur + screen blend over the CRT) — see CRT shader / plexi.

## Key files
- `index.html` — the whole game + shaders. One inline `<script>` at end of body.
  Structure: CSS shell → DOM → script (game section → CRT WebGL shader →
  plexi → glow) . The game section runs from `const W = 580` to just before
  `// ---------------- CRT glass`.
- `bezel.webp`/`bezel.png` — **Space Invaders cabinet art** (2026-11-08).
  The webp has the
  intentional veil alpha (screen-hole region ~80% opaque so baked reflections
  show over the game). **Never zero that alpha.** Same 1024×1024 panel, same
  squircle hole (screen rect left 186, top 201, 688×647) — that's why the
  screen CSS, glow mask, and shaders work unchanged.
- `Bezel_Raygun.webp` — red ray-gun cutout, 512×512 = bottom-left quadrant of
  the panel, alpha + **baked** drop shadow (real cabinet: separate cardboard
  under the glass; static art so no realtime shadow). DOM layer `#raygun`
  sits between `#panel` and `#reflect` (above bezel, below all glass layers),
  CSS `left:0; bottom:0; 45%×45%` (shrunk 10% from the native quadrant —
  size Rex approved; 35% was too small, don't go back).
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
- Bunkers: 4 shields at x-centers 125/235/345/455, top y=373, 46×34 screen px
  over a 22×16 cell grid. Invasion line INVADE_Y = 377 (formation bottom).
- Player: top y=439, x clamped 50–530, speed 280 px/s.
- 2026-11-08: player row + bunkers raised 55 px (was 494/428) because the new
  bezel's bottom art covers the screen bottom — matches the real cabinet.
- UFO: y=62, 95 px/s, scores 50/100/150/300.
- HUD: "1UP" + score top-left, HIGH SCORE top-center, UFO bonus flash left of
  the hi-score, ship icons bottom-left, "WAVE n" bottom-right.
- March (invader thumps): NOTE_FREQS = [62, 55, 49, 43] Hz (G1 F#1 D#1 G0,
  deep low thumps from Grok reference, 2026-11-08) on **square** (hard digital
  edge), vol 0.24, dur 0.12 s. `stepInterval()` auto-speeds the tempo as
  invaders die (0.52 s full → 0.09 s last, ÷(1+(wave-1)*0.05)), so the march
  quickens exactly like the original. If the low notes are too inaudible on
  small speakers, try [70, 62, 55, 49].
- UFO siren: **sustained** VCO+LFO (Grok-provided recipe, 2026-11-08): square
  600 Hz centre, sine LFO 5.3 Hz ±350 Hz, vol 0.06 (dropped 0.12 → 0.08 → 0.06;
  the sustained drone read louder than the one-shot sfx). `startUfoSnd()` on spawn,
  `stopUfoSnd()` on kill / exit / restart; mute follows the gain (M key).
- Hi-score key: `spaceinvaders.hiscore`.

## Glow — state
`GLOW_FALLBACK_MASK` is now baked from **`ref/ScreenHole.png`** (the SI
squircle), 2026-11-08. The old mask was traced from the *Asteroids* hole and
its left rim sat ~32 panel px right of the SI hole, which would misplace the
glow rim. Re-bake: `ScreenHole.png` (1024² white-opaque squircle on
transparent) → 512², `mask[i] = 0 if alpha>127 else 255` (0 = hole), two-pass
chamfer distance (8-neighbor, D8 = 1.41421356), pack `R = mask`,
`G = clamp(d, 0, maxDist/2) / (maxDist/2) * 255`, `B = 0`, `A = 255`, with
`GLOW.maxDist = 96` (so maxD = 48 texels). Base64 swapped into index.html in
run_python (never through edit_file_tool). A disk copy is `ref/glow_mask.png`.
Verified: decoded hole bbox (79,432,100,414) @512 == `ScreenHole.png`.
Re-bake recipe = Asteroids NOTES.md §"Re-baking the mask". If the hole ever
moves: also update the CSS `#screen` rect, the COMP_FS screen rect, and the
canvas W/H + layout constants. `vert_screen_mask.png` in ref/ suggests
vertical is possible.

## CRT shader / plexi
Identical to the Asteroids build (copied verbatim): Glass TV mode default,
COMB 0.5, procedural glass reflection OFF (baked `reflect.webp` instead),
plexi `PLEXI_AMT 2.2`. No `int` in fragment shaders (file:// WebGL taint →
all textures embedded base64 or disk-loaded for the TEMP line).
- **Sprite phosphor bloom** (2026-11-08): wide soft halo around bright
  sprites/text, matching Rex's real-cabinet photo. Implemented as a separate
  `#bloom` canvas (145×136 = ¼ res) inside `#screen`, placed **after** `#fx`.
  Each frame `bloomDraw()` (called right after `draw()` in the frame loop)
  downsamples `cv` into it, **then applies the same barrel warp as the CRT
  shader** (`BLOOM_BARREL = 0.18`, matching the glass pass's `1.0 + 0.18*r2`,
  per-pixel on the tiny canvas with bilinear sampling) so the halo tracks the
  barrel-distorted sprites and doesn't drift at the edges. The warp runs only
  when it's needed (`fxGL && crtMode === 2` = glass mode, the only mode with
  barrel); in arcade/off/no-WebGL it's a straight downsample. CSS on `#bloom`
  (rule next to `#fx`) then does `filter: blur(8px) brightness(1.5)` — a **true
  convolution**, so the halo is smooth, not the grainy "multiple copies of
  itself" the in-shader version had — plus `mix-blend-mode: screen` and
  `opacity: 0.8`. It sits above the CRT image but the opaque `#panel` bezel on
  top clips it to the glass hole. Tuning knobs: `blur()` = spread,
  `brightness()` = pop, `opacity` = overall strength.
  (An earlier in-shader `bloomPass()` — bright-pass + 24-tap 3-ring radial
  sample — read as grainy offset copies at any visible strength and was fully
  reverted; the fragment shader has **no** bloom now.)

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
