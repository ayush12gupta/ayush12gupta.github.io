# CLAUDE.md

## What this repo is

`ayush12gupta.github.io` — personal site built on the [al-folio](https://github.com/alshedivat/al-folio) Jekyll theme (CV, publications, projects, blog, etc.).

## Branch structure — read this first

- **`master`** — the real Jekyll source (Gemfile, `_pages`, GitHub Actions build/deploy workflow, Docker configs). This is where the al-folio theme lives.
- **`gh-pages`** — plain compiled HTML/CSS/JS output (`.nojekyll` present, no Jekyll source, no `_config.yml`). **This is the branch GitHub Pages actually serves** (`https://ayush12gupta.github.io`, no custom domain/CNAME).

You are almost always working directly on `gh-pages` output, not the Jekyll source. There's no build step here — edits to files on this branch are live as soon as they're pushed.

**Caveat:** if `master` is ever rebuilt and redeployed to `gh-pages`, any hand-edits made directly on `gh-pages` (see below) get overwritten. Nothing currently keeps the two in sync automatically.

## Hand-maintained pages outside the Jekyll build

A few pages live only on `gh-pages` and are edited directly as raw HTML/JS (not part of the al-folio theme):

- `c4ts-n-plans/index.html` — "find Chloe" cat gif shuffle grid. Gif pool is the `GIFS` array (~line 195); `shuffleGifs()` picks 6 at random on click/load. Local gifs live in `c4ts-n-plans/gif/`; rest are hotlinked Giphy URLs.
- `assets/date/index.html` — standalone date-proposal page (`cat_dance.gif` etc. alongside it in `assets/date/`).
- `watch-party/index.html` — shared mp4 watch-party with side chat, peer-to-peer via PeerJS + a Metered.ca TURN relay (needed for restrictive/mobile NATs — see `watch-party/TURN-FIX.md` for why and how to rotate the TURN credential).

When adding to the gif pool, just append to the `GIFS` array — no build/registration step needed.
