---
name: commit-changes
description: Commit the current changes in this repository as Conventional Commits. Use when the user asks to commit, stage and commit, or split work into commits (e.g. "コミットして", "/commit-changes", "commit this"). Splits unrelated changes into separate commits and writes each message in Conventional Commits format.
---

# commit-changes

Commit what is currently in the working tree, as one or more Conventional Commits.

## Steps

1. Look at the state before deciding anything:

   ```bash
   git status --short
   git diff            # unstaged
   git diff --cached   # already staged
   git log --oneline -10   # match the style already in this repo
   ```

2. Confirm a committer identity exists. If both are empty, stop and ask the user which
   name and email to use instead of committing with a guessed identity:

   ```bash
   git config user.name; git config user.email
   ```

3. Group the changes. One commit = one logical change. A sample's `index.html` and its
   `README.md`/`CLAUDE.md` belong together when they describe the same change; an unrelated
   root-level edit does not. Stage each group explicitly (`git add <paths>`), never `git add -A`
   when more than one group exists.

4. Write the message (see the format below) and commit. Use a heredoc so the body keeps
   its line breaks:

   ```bash
   git commit -F - <<'MSG'
   feat(hyalite): add caustics to the water surface
   MSG
   ```

5. Report what was committed with `git log --oneline -<n>`. Do not push unless asked.

## Message format

```
<type>(<scope>): <subject>

<body — optional, wrapped at 72 chars, explains why>
```

- **type** — `feat` (new sample or new visual element), `fix`, `perf`, `refactor`, `style`
  (formatting only), `docs`, `chore` (gitignore, settings, tooling), `revert`.
- **scope** — the sample folder name: `hyalite`. Use `repo` for root-level files (README,
  CLAUDE.md, .gitignore, .claude/). Omit the scope only when a change genuinely spans everything.
- **subject** — imperative mood, lowercase, no trailing period, 72 characters or less.
  "add", not "added" or "adds".
- **body** — include it when the *why* is not obvious from the diff: a shader constant that was
  tuned by eye, a decision that must not be reverted, a CDN version that is pinned for a reason.
  Skip it for one-line obvious changes.
- **breaking change** — append `!` after the scope (`feat(hyalite)!: …`) and start a body
  paragraph with `BREAKING CHANGE: `.

Do not add attribution or co-author trailers.

## Examples

```
feat(hyalite): add caustics to the water surface
fix(hyalite): keep the crystal from clipping the camera at scroll end
perf(hyalite): halve the dust particle count on narrow screens
docs(repo): document the one-sample-per-folder layout
chore(repo): move .gitignore to the repository root
```
