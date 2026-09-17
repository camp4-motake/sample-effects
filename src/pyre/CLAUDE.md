# CLAUDE.md — Pyre

A full-screen background of photoreal fire, modelled on a stock clip of a wall of flames on black
(tongues reaching about 60% of the height, bright yellow roots, orange bodies, black gaps).
Only two things sit on top: the title and the copyright. A lil-gui panel exposes the shader
parameters for checking and tuning.

## Implementation notes

- **Raw WebGL 1, no three.js.** It is one full-screen triangle and one fragment shader, so a
  library would add nothing. The shader is GLSL ES 1.00, so it also runs on WebGL2-only drivers
  through the WebGL1 context. With no module scripts, `file://` works.
- A failed `getContext('webgl')`, a shader compile or link error, or `webglcontextlost` sets
  `html.no-webgl`, and CSS draws a radial-gradient glow instead.
- `prefers-reduced-motion` slows the clock to 0.3× rather than stopping the fire, and turns off
  the CSS title animations. The media query is watched live.
- No explicit pause logic: the canvas is fixed full-screen, so it is always on screen, and rAF
  already stops in hidden tabs. The `contentvisibilityautostatechange` / IntersectionObserver
  approach from modern-web-guidance was considered and deliberately not used for that reason.
- `?phase=<0..1>` freezes the loop (it sets the GUI's Freeze and Phase). Use it for screenshots and
  for checking the seam.

## Settings GUI (lil-gui)

- **lil-gui 0.21.0**, UMD build (`dist/lil-gui.umd.min.js`, global `lil.GUI`) from jsdelivr. It is
  pinned because 0.21 renamed its CSS classes to a `lil-` prefix (`.lil-auto-place`,
  `.lil-controller`), and the placement override here relies on those names. The library injects its
  own CSS, so no stylesheet link is needed.
- The script is **injected dynamically after the render loop starts**. A slow or failed CDN load
  never delays or breaks the fire; `onerror` only logs a warning. When WebGL is unavailable the GUI
  is not built at all, because there is nothing to tune.
- Every tunable value lives in `DEFAULTS` in the JS. `params` is the live copy (with the `?phase`
  overrides applied), and `UNIFORM_KEYS` lists the keys sent to the shader; each key `foo` goes to
  `uniform float uFoo`, and all of them are uploaded each frame. To add a knob, add a key to
  `DEFAULTS`, a `uniform float` in the shader, the key in `UNIFORM_KEYS`, and a controller in
  `buildGui()`.
- Time state is `params.phase` alone (0–1); `frame()` advances it by `dt * speed / loop`. Do not
  reintroduce a separate seconds clock that has to be kept in sync with it.
- `Rise` and `Churn` sliders use step 1 on purpose: non-integer values break the seamless loop
  (see below). Keep that step.
- Placement: top-left on desktop, because the title owns the top-right. At `max-width: 600px` the
  panel spans the full width under the title, opens downward, and starts closed. It is not anchored
  to the bottom, where it would sit on the copyright.
- `Copy settings (JSON)` copies the current values (without phase/freeze) in a shape that can be
  pasted back into `DEFAULTS`, or logs them when the clipboard is unavailable. `Reset to defaults`
  calls `gui.reset()`. The `H` key toggles the panel.
- Headless Chromium (SwiftShader) screenshots show a transparent strip between the bottom of the
  open panel and its `max-height`, but only while the WebGL canvas is visible. This is a screenshot
  compositing artifact, not a page bug; don't "fix" it by changing the layout.

## How the seamless loop works (do not break this)

The JS passes `uPhase` (0–1) as the only time input. One cycle lasts `loop / speed` seconds
(18 / 1.25 = 14.4 s with the defaults). Every noise lookup uses `pnoise`, a gradient noise that repeats with period `rep`. The y lattice (rising) and the z lattice (shape
change) both repeat every `REP = 4`, and each fbm octave doubles both its frequency and `rep`.
Over one loop:

- the main flame moves up `REP * uRise` (4 periods by default) and forward in z by `REP * uEvol` (2 periods),
- the domain warp and the haze move up `REP` (1 period),
- the tongue-height map and the haze move forward in z by `evol * 0.5`, which is `2 * uEvol` — a
  whole number of periods only when `uEvol` is even (1 period at the default 2). Odd Churn values
  currently leave a seam in these two terms.

Each of these is a whole number of periods, so phase 1 renders like phase 0. A `readPixels`
comparison at `?phase=0` and `?phase=1` (Auto resolution off, fixed resolution) found only 19
channel values off by 1/255, which is float rounding. A frame 0.1 s apart differs by up to 252.
When adding a time-dependent term, make sure its y or z offset at phase 1 is a whole multiple of `REP`. Never scale y or z inside
`fbm` by anything except ×2 per octave. Constant offsets are fine, and x is not periodic, so it can
be scaled freely.

## Shader structure and tuning points

Values in the "GUI key" column are `DEFAULTS` keys (default in parentheses); the rest are
constants in the shader.

| Part | GUI key → uniform | What it does / fixed constants |
|---|---|---|
| Loop length / speed | `loop` (18), `speed` (1.25) | JS only. Screen rise speed ≈ `REP*rise*speed / (1.8 * loop)` heights per second |
| Rise / churn | `rise` (4) → `uRise`, `evol` (2) → `uEvol` | Periods per loop; **integers only** |
| Domain warp | `warp` (0.5) → `uWarp` | Scales `w.x * 1.9`, `w.y * 1.1`, `w.y * 0.3`; warp frequency `x * 1.3` is fixed |
| Tongue density | `stretch` (4.9) → `uStretch` | x frequency against the fixed `y * 1.8`; higher gives thinner, taller-looking tongues |
| Flame height | `height` (0.44) → `uHeight`, `tongue` (0.82) → `uTongue` | Used by both layers, together with `0.85 - y / h` |
| Gaps / raggedness | `gaps` (5) → `uGaps` | Higher means more black gaps and a more broken edge |
| Filaments | `strands` (0.7) → `uStrands` | Scales `strands`, which drives the `edge * strands * 0.6` whiskers and the `0.55 + 0.9 * strands` banding |
| Edge softness | `sharp` (0.3) → `uSharp` | Width of `body = smoothstep(0, uSharp, heat)`. The `+ 0.12` in `T` keeps edges orange |
| Intensity | `intensity` (7.1) → `uIntensity` | `T*T` gain |
| Colour | `temp` (0.69) → `uTemp` | `1 - exp(-T * vec3(3.0, 1.1*t, 0.30*t²))`, then `pow(..., vec3(1.1, 1.45, 2.0))`; higher pushes toward yellow/white |
| Depth | `back` (1) → `uBack` | Back layer × `vec3(0.32, 0.11, 0.03)`, hidden where the front layer is bright (`cover`); too strong fills the gaps with a red wash |
| Haze / vignette | `haze` (1.2), `vignette` (0.5) | Warm glow above the flames; edge darkening |
| Portrait | — | `k = pow(1/aspect, 0.6)` keeps tongues from stretching on tall screens; 1.0 would be fully width-based and too short |

## Performance

- The canvas renders at `params.quality × min(dpr, 2)` of CSS pixels (default `0.25`, GUI
  "Resolution") and CSS upscales it, since fire is soft. While "Auto resolution" is on, if more
  than about 20 frames drop below 40 fps, `quality` steps down by 0.1 to a floor of `0.4`. The GUI
  shows the FPS and the canvas size. The default `0.25` is below that floor on purpose: it was
  picked with the GUI. Auto resolution only ever lowers the value, so it stays at 0.25. The floor looks visibly chunky, but it only kicks in on weak GPUs or
  software GL.
- The cost is two `flame()` layers (6-octave fbm + 3-octave ridged each) plus a 3-octave warp ×2.
  Remove octaves before lowering `quality` any further.

## Design

- Background is pure black. Token: `--ash #f6ede4` (text).
- Type: **Questrial** for the title (uppercase, `letter-spacing: 0.32em`, with the trailing
  tracking cancelled by a negative `margin-right` so it lines up with the right padding).
  **Instrument Sans** for the copyright.
- The title is small (`clamp(18px, 1.9vw, 28px)`) and sits in the top-right, above where the
  flames reach. Its warm `text-shadow` pulses (`glow`) to suggest firelight.
- The copyright sits on the bright roots, so it gets a dark-brown double `text-shadow`. Do not drop
  that shadow.

## Decisions to keep

- Page copy is just the title and the copyright. Do not add nav, taglines or sections. (The GUI
  is a tool, not page copy.)
- The title was moved from a large centred Italiana heading to a small top-right Questrial one.
  Do not move it back.
- The back layer stays dim and red. A brighter back layer made the gaps between tongues disappear,
  which is the main thing that made it read as fake.
