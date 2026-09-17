# Pyre

A full-screen background sample: a photoreal wall of fire drawn by a single GLSL fragment shader,
looping seamlessly (every 14.4 seconds by default), with only a title and a copyright line on top.
A settings panel (lil-gui, top-left) shows and adjusts the shader parameters. Press `H` to hide it,
and use "Copy settings (JSON)" to take the current values away.

It has no module scripts (only Google Fonts and lil-gui from a CDN), so opening `index.html`
directly works.
A local server is still the more reliable way to load the fonts:

```bash
npx serve .          # from the repo root, then open /src/pyre/
```

Add `?phase=0.3` (any value from 0 to 1) to freeze the loop at that point, for screenshots.
