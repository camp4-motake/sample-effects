# CLAUDE.md — Hyalite LP

A one-page landing page: a three.js crystal hovering above rippling water.
Sections are Hero → About → Footer.

## Implementation notes

- three.js **r128**, loaded as ES modules from jsdelivr through an `<script type="importmap">`
  (`three` → `build/three.module.js`, `three/addons/` → `examples/jsm/`). r128 ships the same
  postprocessing chain under `examples/jsm/`, so the ESM move did not force a version bump; the pin
  stays because later versions reworked those addons. The addon files import the bare specifier
  `three`, which is why the import map is required rather than optional.
- The page is a single `<script type="module">`. Libraries are pulled in with **dynamic `import()`
  inside `try`/`catch`**, not static imports: a static import that fails (CDN down, or a browser
  without import-map support) would abort the whole module and take the fallbacks with it. Top-level
  `await` is fine here — one module, one entry point, so the WebKit multi-importer bug does not apply.
- Because it is a module, `file://` no longer works; the sample needs a local server.
- Post-processing: `RenderPass` → `UnrealBloomPass` → a custom `ShaderPass` that compresses tone
  while preserving hue, so highlights don't blow out to cyan. The `EffectComposer` renders into a
  `HalfFloatType` target so compositing stays in HDR.
- Scrolling goes through **Lenis 1.3.26**, imported from jsdelivr as `dist/lenis.mjs` (default
  export). Pinned so the class names the CSS relies on (`html.lenis`, `.lenis-stopped`) stay stable.
  It runs with `autoRaf: true`, so Lenis ticks in its own rAF, separate from the render loop. Only the
  rules of `lenis.css` this page needs are inlined, rather than pulling a second CDN file.
- Fallbacks: a rejected `import('three')` or a failed WebGL context sets `html.no-webgl`, and CSS
  draws a gradient background instead. If only the postprocessing addons fail to load, the scene
  still renders without bloom. Under `prefers-reduced-motion` the animation clock is scaled to
  0.35, the CSS animations stop, and Lenis is not created at all. Lenis is likewise skipped if its
  import fails — both cases fall back to native scrolling.

## Scene elements and where to tune them

| Element | Implementation | Main knobs |
|---|---|---|
| Crystal | `LatheGeometry` with 6 segments (hexagonal prism + pyramid caps), `toNonIndexed()` for flat normals | `R`, `HALF`, `CAP` (shape); `deep` / `mid` / `hi` in the crystal fragment shader (color) |
| Halo | Additive `Sprite` from a canvas radial gradient | `halo.scale`, `halo.material.opacity` |
| Water | Large plane + `ShaderMaterial`; noise perturbs the normal, and only where the reflection vector points at the crystal's vertical axis (a segment) does it light up | noise amplitudes in `height()`, normal strength `0.3`, sharpness exponents such as `pow(c, 140.0)` |
| Dust motes | `Points` + additive `ShaderMaterial` | `COUNT` |
| Bloom | `UnrealBloomPass` | its `(strength, radius, threshold)` args and `bloom.strength` in `frame()` |

## Scrolling

- `lerp: 0.085` is how softly Lenis follows the wheel (smaller = slower to catch up), and
  `wheelMultiplier: 0.9` slightly damps one wheel notch to keep the pace glacial. Touch is left on
  the browser's native scrolling (Lenis's `syncTouch` default).
- In-page links are intercepted and handed to `lenis.scrollTo(el, { duration: 1.6 })`; they no
  longer update `location.hash`.
- `scroll-behavior: smooth` is the fallback for when Lenis is absent. It cannot coexist with
  Lenis's programmatic scrolling, so `html.lenis { scroll-behavior: auto; }` turns it off once
  Lenis is live. Keep that rule keyed on the class — a `html:not(.lenis)` form would outrank the
  `prefers-reduced-motion` override on specificity and break it.
- `onScroll` is subscribed to both the native `scroll` event and Lenis's, so the camera tracks the
  interpolated position.

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
