# sample-effects

A collection of self-contained web expression samples (three.js, WebGL, CSS).
Each sample lives in its own folder under `src/` and runs in a browser with no build step.

| Sample | What it is |
|---|---|
| [`src/hyalite`](src/hyalite) | One-page landing page with a three.js crystal hovering over rippling water |

## Running

```bash
npx serve .                 # then open /src/hyalite/
```

Opening a sample's `index.html` directly works too, but a local server makes the
font and CDN loads more reliable.
