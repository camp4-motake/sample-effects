# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A growing collection of web expression samples — three.js / WebGL / CSS demos, one per folder
under `src/`. `src/hyalite` is the first one.

There is **no build step, package manager, or test runner**: no `package.json`, no lockfile,
so there is nothing to `npm install` and no `npm test`. Dependencies are loaded from a CDN with
plain `<script>` / `<link>` tags.

```
src/<sample>/
  index.html   # the whole sample: HTML + CSS + JS in one file
  README.md    # what it is, how to open it
  CLAUDE.md    # design intent, tuning points, decisions that must not be reverted
```

## Running a sample

```bash
npx serve .                 # from the repo root, then open /src/<sample>/
python3 -m http.server 8000 # same thing without npx
```

Opening `index.html` directly works, but a local server makes Google Fonts and jsdelivr loads
reliable.

## Verifying changes

There is no lint or automated test — the only check is looking at the page in a browser.
After touching a shader, a CSS value, or animation timing, view the result and, for samples with
scroll choreography, check both the top of the page and the bottom.

## Conventions for samples

- **One sample = one folder = one self-contained `index.html`.** Keep HTML, CSS and JS in that
  single file; pull external libraries from a CDN at a pinned version (three.js is pinned to r128
  in hyalite — see its CLAUDE.md for why).
- Put sample-specific knowledge in that sample's `CLAUDE.md`: what each tunable number does, and
  which changes were deliberately made and must not be undone. Keep this root file for things that
  apply to every sample.
- Provide graceful degradation: a CSS-only fallback when WebGL is unavailable (hyalite sets
  `html.no-webgl`), and slower or disabled motion under `prefers-reduced-motion`.
- Code comments are written in Japanese; user-facing copy on the pages is English.

## Commits

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):
`<type>(<scope>): <subject>`, where the scope is the sample folder name (`hyalite`) or `repo` for
root-level files. Keep one logical change per commit. Run `/commit-changes` to do this — the skill
in `.claude/skills/commit-changes/` has the full format, the allowed types, and examples.

Attribution and co-author trailers are switched off for this repository in
`.claude/settings.json`; do not add them by hand.
