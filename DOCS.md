# UMBRA — Technical Documentation

> Read this file to understand the whole game **without reading the code**.
> Everything lives in a single `index.html` (zero-build, Three.js via importmap CDN).
> All balance numbers are centralised in **§14 `CONFIG`** and the **`LEVELS`** array.

---

## 0. FEATURE STATUS (read first)

| Feature | Status | Where (function / marker) |
|---|---|---|
| Core loop: rotate construct → match cast shadow to target | ✅ | `evaluate()`, `loop()` |
| Offscreen silhouette render + **IoU** match detection (throttled) | ✅ | `renderSil()`, `computeIoU()`, `CONFIG.IOU_EVERY` |
| Solvable-by-construction targets (rendered from solution pose) | ✅ | `loadLevel()` (renders `rtTarget` at identity) |
| Single-light levels (1 shadow) | ✅ | `layoutPanels()` (`level.lights===1`) |
| **Dual-light** levels (2 shadows at once) | ✅ | `layoutPanels()` (`level.lights===2`) |
| Construct builder from primitive specs | ✅ | `buildConstruct()`, `GEO`, `LEVELS[].prims` |
| Drag-to-rotate (arcball yaw/pitch) | ✅ | `applyYawPitch()` |
| Roll (two-finger twist / ⟲⟳ buttons / Q–E keys) | ✅ | `applyRoll()`, `applyRollAbsolute()`, `#rollRow` |
| Per-level allowed axes (difficulty lock) | ✅ | `axisAllowed()`, `LEVELS[].axes` |
| Closeness meter + rising-glow + proximity tone | ✅ | `setMeter()`, panel `uGlow`, `Audio.prox()` |
| Solve: bloom flash + shadow fills + chime + haptic | ✅ | `doSolve()`, `animateSolve()`, `flashConstruct()` |
| Star rating (closeness + no-hint), saved per level | ✅ | `doSolve()`, `CONFIG.STAR_IOU2/3` |
| Level select grid (lock / stars / dual badge) | ✅ | `buildSelectGrid()`, `isUnlocked()` |
| Progress unlock (finish N → unlock N+1) | ✅ | `isUnlocked()` |
| Hint (ghost the target brighter; caps stars at 2) | ✅ | `#hintBtn`, `hintT`, `S.hintUsed` |
| Onboarding "How to play" (first run) | ✅ | `#how`, `Save.get('onboarded')` |
| Audio: ambient pad + SFX (WebAudio synth) + mute | ✅ | `Audio` object |
| Settings: music / sfx / haptics | ✅ | `Settings`, `bindSwitch()` |
| localStorage persistence (`umbra.*`) | ✅ | `Save`, §13 |
| CDN-fail guard (Reload message, no black screen) | ✅ | top-level `try/catch`, `#reload` |
| Graceful perf (DPR cap, bloom only on bright px) | ✅ | `renderer.setPixelRatio`, bloom threshold 0.82 |

**Backlog / not built:** daily reward, leaderboard, PWA/offline (intentionally omitted — calm single-player puzzle).

---

## 1. Overview & Concept

### Market context (research 2026-06-05)
- **Genre hot:** calm / cozy 3D *spatial* puzzles (rotate-to-reveal) remain evergreen — **Shadowmatic** (Apple Design Award 2015), Monument Valley, Mekorama, Evo Explores, Prune.
- **Theme/style:** high-fidelity light & shadow, minimal and meditative → our take = **neon shadow-theater** on the arcade palette (periwinkle `#9b8cff`).
- **Story/vibe:** *umbra* = the dark core of a shadow; quiet, museum-calm shadow-play.
- **Competitor gap:** Shadowmatic is paid, mobile-only, no web build; H5/web has almost no true rotate-the-shadow puzzle. Gap = a **free, instant, mobile-first** browser version.
- **Hook:** "Shadowmatic's *aha* moment, free in your browser — then doubled: late levels cast **two** shadows you must solve at once."

### Concept
- **Genre:** 3D shadow-silhouette puzzle (rotate-to-match).
- **Core loop (1 line):** *"Rotate a glowing construct until the shadow it casts lines up with the target silhouette; the closer it gets the brighter it glows, snap it in to solve — then later, solve two shadows at once."*
- **Win / lose:** Win = every light's shadow matches its target (IoU ≥ `SOLVE_IOU`). There is **no lose state** (calm puzzle); you simply keep rotating.
- **Fantasy:** calm, clever, satisfying "aha" of a jumble resolving into a clear shape.
- **2D or 3D:** **3D (Three.js)** — depth, soft shadow and the dual-projection trick *are* the game.
- **Layout:** **Strategy A — Mobile-frame** (`#app` ≤ 480px portrait, centred + letterboxed on desktop, no responsive breakpoints). Portrait suits the stacked shadow-screens.

## 2. Tech stack
- **Render:** Three.js **r0.169** via importmap CDN (`three`, `three/addons/`). `EffectComposer` + `UnrealBloomPass` (strength 0.5, threshold 0.82) + `OutputPass`.
- **Build:** zero-build, single `index.html`, ES modules, no bundler.
- **Storage:** `localStorage` (`umbra.*`). **Audio:** WebAudio synth (no files).
- **CDN-fail guard:** Three.js is `await import()`-ed inside `try/catch`; failure shows the `#reload` panel.

## 3. State machine
`mode ∈ menu → playing → (paused) → solved → (next/replay/select)`
- **menu:** title + Play / Level select / How to play (overlay `#menu`).
- **how / select:** onboarding & level grid overlays.
- **playing:** construct is interactive; `evaluate()` runs each frame.
- **paused:** `#pause` overlay (settings); pad muted.
- **solved:** `doSolve()` locks, awards stars, then shows `#solved`.
Scene overlays are toggled by `show()` / `hideAll()`; `#app.overlay-open` hides the HUD.

## 4. Gameplay & rules
- A **construct** (composition of neon primitives) floats in a beam of light.
- Each **light** has an orthographic "shadow camera" aimed at the construct; the construct's **silhouette** from that camera **is** the shadow it casts (a directional/parallel-light projection).
- That silhouette is rendered to an offscreen target and **painted on a back screen** as a soft dark shadow, together with the faint **target silhouette**.
- The player **rotates** the construct (drag = yaw/pitch, twist/buttons/keys = roll). Disallowed axes per level are ignored (`axisAllowed`).
- **Match detection:** a few times per second (`CONFIG.IOU_EVERY`) the silhouette is read back and compared to the target mask via **IoU** (intersection ÷ union). When **every** light's IoU ≥ `CONFIG.SOLVE_IOU` for `CONFIG.SOLVE_HOLD` seconds → **solved**.
- Because each target is rendered from the construct's **actual solution pose**, every level is solvable by construction, and symmetric constructs naturally allow multiple correct angles.

---

## 5. LEVEL STRUCTURE ⭐

### 5.1 Level model
- **Discrete levels** (15), rising difficulty. Each = one construct + 1 or 2 lights.
- The displayed target is **not hand-drawn**: at load the construct is set to its **solution pose** (identity quaternion) and its silhouette is rendered into `rtTarget` (per light). The level then starts from a **scrambled** orientation; the player un-scrambles it.

### 5.2 Where levels are defined
- Array **`LEVELS[]`** near the top of the module. One entry:
```javascript
{ name:'Key',                 // shown in HUD / select / solved
  glyph:'a key',              // human label of the silhouette (docs/aria)
  lights:1,                   // 1 = one shadow, 2 = dual-light
  axes:['pitch'],             // which rotation axes input controls (yaw|pitch|roll)
  scramble:{pitch:64},        // starting offset from solution, per axis (degrees)
  prims:[                     // the construct: a list of primitives
    {t:'torus', s:[0.86,0.86,0.5], p:[-1.0,0,0]},      // t=type s=scale[x,y,z] p=pos r=rot°
    {t:'cyl',   s:[0.17,1.7,0.17], p:[0.15,0,0], r:[0,0,90]},
    ... ] }
```
- Primitive types (`GEO`): `box`, `sphere`, `cyl`, `cone`, `torus`. `s` scales the unit geometry (sphere/cyl/cone build ellipsoids/elliptic shapes); `p` position; `r` Euler degrees.
- **Light A** looks along −Z → silhouette = projection onto the **XY** plane (the "front" you design).
- **Light B** (dual only) looks along −X → silhouette = projection onto the **ZY** plane (the "side").
- Solution quaternion is always **identity**; design each construct so its solution-pose silhouette reads as the intended shape.

### 5.3 The 15 levels
| # | Name | Lights | Axes (input) | Scramble (°) | Silhouette reads as |
|---|------|:--:|---|---|---|
| 1 | Halo | 1 | yaw | yaw 74 | a ring |
| 2 | Arrow | 1 | yaw | yaw 66 | an arrow |
| 3 | Cat | 1 | yaw | yaw 70 | a cat (head + ears + body) |
| 4 | Key | 1 | pitch | pitch 64 | a key (bow + shaft + teeth) |
| 5 | Bird | 1 | yaw, pitch | 58 / 42 | a bird (body + beak + tail) |
| 6 | Bottle | 1 | yaw, pitch | 54 / 46 | a bottle |
| 7 | Fish | 1 | yaw, pitch | 62 / 40 | a fish |
| 8 | Sailboat | 1 | yaw, pitch | 56 / 44 | a sailboat |
| 9 | Umbrella | 1 | yaw, pitch, roll | 40 / 50 / 64 | an umbrella |
| 10 | Rabbit | 1 | yaw, pitch, roll | 44 / 46 / 58 | a rabbit |
| 11 | Anchor | 1 | yaw, pitch, roll | 52 / 48 / 70 | an anchor |
| 12 | Disc & Bar | **2** | yaw, pitch | 62 / 38 | disc (A) + bar (B) — one cylinder |
| 13 | Plus & Bar | **2** | yaw, pitch | 58 / 44 | plus (A) + bar (B) — crossed boxes |
| 14 | Arrow & Disc | **2** | yaw, pitch, roll | 50 / 44 / 54 | arrow (A) + disc (B) — one arrow |
| 15 | Cat & Key | **2** | yaw, pitch, roll | 54 / 52 / 66 | cat (A) + key (B) — one tangle |

### 5.4 Progression
- Levels unlock in order: level `i` is playable if `i===0` or `stars[i-1] > 0` (`isUnlocked()`).
- Tiers: **A** L1–4 single-axis (input locked to one axis) · **B** L5–8 two-axis · **C** L9–11 three-axis incl. roll · **D** L12–15 dual-light.

---

## 6. DIFFICULTY STRUCTURE ⭐

There are **no Easy/Normal/Hard modes** — difficulty is the **level curve** itself, driven by three independent knobs per level:

| Knob | Field | Effect |
|---|---|---|
| Number of axes to correct | `axes[]` | 1 axis (trivial) → 3 axes incl. roll (hard). Input is **locked** to these axes, so early levels can't be over-rotated. |
| Scramble magnitude | `scramble{}` | Larger offsets ⇒ lower start IoU ⇒ more rotation to solve. Measured start IoU ranges ~0.07 (Anchor) to ~0.61 (Disc&Bar). |
| Light count + construct ambiguity | `lights`, `prims` | Dual-light requires one pose to satisfy **two** targets at once; more/ambiguous primitives read as a jumble from wrong angles. |

**Curve:** gentle on-ramp (chunky symmetric shapes, one axis) → multi-axis recognizable objects → roll → dual-light finale. Solve tolerance is constant (`SOLVE_IOU`), so difficulty comes purely from how hard the matching pose is to find.

To tune difficulty → see **§15.2**.

---

## 7. SCORING (stars) ⭐

UMBRA scores **closeness**, not points. Each level awards **1–3 stars**, best kept per level.

### 7.1 Star formula (`doSolve()`)
Let `peak` = the highest closeness (min IoU across all lights) reached during the attempt.
```
stars = 1                                   // solved at all (IoU ≥ SOLVE_IOU)
if peak ≥ CONFIG.STAR_IOU2 (0.94)  → 2
if peak ≥ CONFIG.STAR_IOU3 (0.965) AND hint not used → 3
```
- "Match %" shown on the solved screen = `round(peak·100)`.
- Using the **Hint** sets `S.hintUsed = true` and caps the run at 2 stars.
- Only an **improvement** is saved: `if (stars > previous) stars[level] = stars`.

### 7.2 Closeness
- `closeness` = smoothed `min(IoU over lights)`; drives the meter, the target glow uniform `uGlow`, the proximity tone, and the ambient-pad volume.

### 7.3 Record
- Per-level stars in `localStorage` key `umbra.stars` (object `{levelIndex: stars}`).
- Menu / select show **total / 45** (`STAR_MAX = 15 × 3`).

To change scoring → see **§15.3**.

## 8. Economy
None (no currency).

## 9. Items / power-ups
None. The only assist is the **Hint** (`#hintBtn`): briefly brightens the target silhouette (`hintT`), marks `hintUsed` (caps 2★). There are no consumables.

## 10. Audio (`Audio` object, WebAudio synth)
| Event | Sound |
|---|---|
| Ambient bed | slow detuned sine **pad**, volume tracks closeness (`startPad`/`setPad`) |
| Approaching a match | rising **proximity tone** as closeness crosses steps (`prox`) |
| Solve | 5-note **chime** arpeggio + per-star notes (`chime`) |
| UI tap / roll | soft `click` |
- Init lazily on first gesture (`Audio.ensure`, `gestured` flag); `navigator.vibrate` is gated behind `gestured` so headless captures stay error-free. Mute button + Settings (music/sfx/haptic).

## 11. Controls
- **Drag (1 finger / mouse):** yaw (horizontal) + pitch (vertical) — only for axes in `level.axes` (`applyYawPitch`).
- **Roll:** two-finger twist (`pointermove` with 2 pointers → `applyRollAbsolute`), on-screen **⟲ ⟳** buttons (`#rollRow`, shown only when `roll` is allowed), or **Q / E** keys.
- **Keyboard:** arrows = yaw/pitch, Q/E = roll, Esc = pause/resume.
- **Hint:** 💡 brightens the target (caps stars). **Mute:** 🔊. **Menu/Pause:** ☰.

## 12. State object `S`
```
mode      'menu'|'playing'|'paused'|'solved'
lvl       current level index (0-based)        level  → LEVELS[lvl]
startQuat scrambled starting orientation (THREE.Quaternion)
closeness smoothed min-IoU across lights (0..1)  peak  → best closeness this attempt
solved    bool      holdT  seconds the match has been held
hintUsed  bool (caps stars)                       iouTimer/idle  throttle + settle timers
```
Construct orientation is `construct.quaternion` (rotated about the construct's own centre).

## 13. localStorage keys (`umbra.*`)
| key | meaning |
|---|---|
| `umbra.stars` | `{levelIndex: stars}` best stars per level |
| `umbra.lastLevel` | last level played |
| `umbra.onboarded` | first-run How-to flag |
| `umbra.muted` | master mute |
| `umbra.music` / `umbra.sfx` / `umbra.haptic` | settings toggles |

---

## 14. BALANCE NUMBERS (single source of truth) ⭐
All in the **`CONFIG`** object at the top of the module:
```javascript
const CONFIG = {
  SIL_RES: 256,        // offscreen silhouette / IoU resolution (px square; higher = sharper edges)
  SOLVE_IOU: 0.90,     // per-light IoU required to solve (all lights must pass)
  SOLVE_HOLD: 0.26,    // s the match must be held before it locks
  IOU_EVERY: 0.11,     // s between throttled IoU read-backs (NOT every frame)
  FIT: 1.95,           // fallback ortho half-extent (auto-fit per level overrides this)
  STAR_IOU2: 0.94,     // peak closeness for 2nd star
  STAR_IOU3: 0.965,    // peak closeness for 3rd star (also requires no hint)
  ROT_SPEED: 0.0095,   // radians of rotation per pixel dragged
  ROLL_BTN: 0.16,      // radians per ⟲/⟳ tap or Q/E press
  LIGHT_A: [0,0,-1],   // front light  → XY-plane silhouette
  LIGHT_B: [-1,0,0],   // side light   → ZY-plane silhouette (dual levels)
  ACCENT: 0x9b8cff,    // periwinkle
};
```
- Light cameras are **auto-fit** to each construct's bounding radius at load (`constructRadius()` → `panel.setFit(r·1.1)`), so nothing clips at any rotation regardless of `FIT`.
- Level data: the **`LEVELS[]`** array (§5.2).

---

## 15. HOW-TO recipes ⭐

### 15.1 Add a level
1. Append an entry to **`LEVELS[]`** (§5.2): set `name`, `glyph`, `lights`, `axes`, `scramble`, and `prims`.
2. Design `prims` so the **solution-pose** silhouette reads as your shape: light A sees the **XY** projection (front), light B (if `lights:2`) sees the **ZY** projection (side). Keep single-light shapes fairly planar (small Z) so wrong angles collapse to a clear "jumble".
3. Verify solvability: open `?shot=play:N`; the QA `window._U.probe(N-1)` returns `solved` IoU (must be ≈1.0) and `start` IoU (should be < ~0.88 so it isn't pre-solved).
4. Update **§5.3** table here (same commit).

### 15.2 Tune difficulty
1. Edit that level's `axes` (fewer = easier) and `scramble` angles (smaller = easier) — **not** `CONFIG.SOLVE_IOU`.
2. Re-run the verify sweep (`node .qa-harness/umbra.mjs verify`) — confirm `solved≈1.0`, `start<0.88`.
3. Update the **§5.3 / §6** tables here.

### 15.3 Change scoring
1. Edit `CONFIG.STAR_IOU2` / `STAR_IOU3` (and `SOLVE_IOU` for the solve gate).
2. Mirror the new thresholds in **§7.1** here.

### 15.4 Change the match feel / lights
- Solve tolerance: `CONFIG.SOLVE_IOU` (lower = more forgiving) and `SOLVE_HOLD` (longer = must settle).
- Light directions: `CONFIG.LIGHT_A` / `LIGHT_B`. Match resolution / IoU accuracy: `CONFIG.SIL_RES`.

---

## 16. Update history
- **2026-06-05** (initial): 15 levels (11 single-light + 4 dual-light), IoU shadow-match engine, stars, level select, hint, audio. Single `index.html`, Three.js r0.169.
- **2026-06-05** (visual pass): brighter "lightbox gallery" look — luminous twilight backdrop (gradient `scene.background`), bright frosted shadow-screens with high-contrast dark shadows + crisp white-hot target outlines, sharper silhouettes (`SIL_RES` 192→256), luminous glass construct, lighter menu scrim.

> **Last updated:** 2026-06-05 · branch `main` (visual pass)
