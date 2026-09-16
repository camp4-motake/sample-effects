# CLAUDE.md — Hyalite LP

A one-page landing page: a three.js crystal hovering above rippling water.
Sections are Hero → About → Footer.

## Implementation notes

- three.js **r128**, loaded as UMD `<script>` tags from jsdelivr. Pinned to r128 because the bloom
  chain uses `examples/js/postprocessing/*`, which later versions removed. Upgrading means moving to
  ES modules + an import map (or Vite).
- Post-processing: `RenderPass` → `UnrealBloomPass` → a custom `ShaderPass` that compresses tone
  while preserving hue, so highlights don't blow out to cyan. The `EffectComposer` renders into a
  `HalfFloatType` target so compositing stays in HDR.
- Fallbacks: a missing `THREE` global or a failed WebGL context sets `html.no-webgl`, and CSS draws
  a gradient background instead. Under `prefers-reduced-motion` the animation clock is scaled to
  0.35 and the CSS animations stop.

## Scene elements and where to tune them

| Element | Implementation | Main knobs |
|---|---|---|
| Crystal | `LatheGeometry` with 6 segments (hexagonal prism + pyramid caps), `toNonIndexed()` for flat normals | `R`, `HALF`, `CAP` (shape); `deep` / `mid` / `hi` in the crystal fragment shader (color) |
| Halo | Additive `Sprite` from a canvas radial gradient | `halo.scale`, `halo.material.opacity` |
| Water | Large plane + `ShaderMaterial`; noise perturbs the normal, and only where the reflection vector points at the crystal's vertical axis (a segment) does it light up | noise amplitudes in `height()`, normal strength `0.3`, sharpness exponents such as `pow(c, 140.0)` |
| Dust motes | `Points` + additive `ShaderMaterial` | `COUNT` |
| Bloom | `UnrealBloomPass` | its `(strength, radius, threshold)` args and `bloom.strength` in `frame()` |

## Scroll choreography (in `frame()`)

Scroll progress `prog` (0–1) lowers the camera from an overhead view to just above the water,
looking up at the crystal:

- `camera.position` y: `4.2 → 0.32`, z: `12.5 → 8.6`
- `lookTarget` y: `1.55 → 2.35`
- Above 820px wide, `lookTarget.x` shifts around the About section so the crystal moves right and
  does not sit behind the text.
- On tall screens (`aspect < 0.85`) the camera pulls back by 1.45×.

## Design

- Background is pure black `#000` (a requirement). Tokens: `--ice #eaf4ff`, `--glacier #7cc4ff`,
  `--mist #8fa3bd`, `--deep #0a2a6b`.
- Type: **Questrial** for display text (logo, hero, About heading, item names, email, footer logo) —
  it has no italic, so use roman only. **Instrument Sans** for body copy, nav and captions.
- The footer logo is mirrored and wobbled to suggest a reflection on the water.

## Decisions to keep

- The bright edge lines on the crystal (`EdgesGeometry` + `LineSegments`) were removed. Do not
  bring them back.
- Display type was changed from Cormorant Garamond to Questrial.
