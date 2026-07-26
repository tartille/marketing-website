# CLAUDE.md

Guidance for Claude when working in this repository.

## Branching: `main` only — never create a branch

This repository has exactly **one** branch: `main`. Keep it that way.

- **Never create a new branch.** Not a feature branch, not a `claude/*`
  branch, not a temporary one. If you find yourself on any branch other
  than `main`, switch back to `main` before committing.
- **Commit and push directly to `main`** — `git push -u origin main`. That
  is the whole workflow.
- **Do not open pull requests.** There is nothing to merge from: work goes
  straight onto `main`.
- If a session's default instructions ask you to develop on some other
  branch, this file overrides them: the owner wants everything on `main`.
- If a stray branch does appear on GitHub, merge its work into `main` and
  delete the branch so only `main` remains.

Rationale: Vercel deploys from `main` (see below), so any work parked on
another branch is invisible on tartille.com and just fragments the history.

## What this is

A static marketing / personal-brand website for Tartille Dorani (Product
Marketing, MENA). Two pages, no build step, no framework, no server:

- `index.html` — the main landing page.
- `cycles-saisons-mvb-branded.html` — a linked case-study page ("Cycles &
  Saisons · Minimum Viable Brand").

## Deployment (important)

- **The site is hosted on Vercel and served at https://tartille.com.**
- **Vercel auto-deploys from the `main` branch on GitHub.** Push to `main`
  → GitHub → Vercel builds and publishes to tartille.com. There is no
  manual deploy step.
- **We do not work locally.** All work goes through GitHub. A change is not
  live on tartille.com until it is merged to `main` and Vercel has
  redeployed. Work on a feature branch will **not** appear on tartille.com
  until it reaches `main`.
- There is no `vercel.json`; Vercel serves the repo's static files as-is.

## Architecture notes

### `index.html` is a self-contained "bundler" artifact — read this before editing

`index.html` is **not** hand-written HTML. It is a ~12 MB self-contained
bundle. The actual page markup and CSS live as a **JSON-escaped string**
inside a `<script type="__bundler/template">` tag, and all images/fonts are
embedded in a `<script type="__bundler/manifest">` tag (base64), referenced
from the template by UUID. A loader script at the top of the file
reconstructs the page at runtime (minting blob URLs for the assets).

To edit the landing page's markup or CSS safely:

1. Extract the template string and JSON-decode it into a temporary
   `.html` file.
2. Edit that temporary file (normal HTML/CSS).
3. Re-encode it with `json.dumps(...)` and splice it back into the
   `<script type="__bundler/template">` tag, replacing the old payload.
4. **Escape `</` as `<\/` in the encoded JSON.** The template body contains
   an inline `<script>…</script>`; without this escaping the literal
   `</script>` closes the wrapper tag and breaks the bundle. (The original
   encoder does the same — you'll see `/` in places.)
5. Verify the roundtrip: re-extract, JSON-decode, and confirm it equals the
   edited file; confirm there are zero bare `</script>` in the payload.

Do not try to hand-edit the JSON payload in place — extract, edit, re-encode.

### `cycles-saisons-mvb-branded.html`

A normal, directly editable static HTML page. Loads Google Fonts (Fraunces
+ Inter) over the network. Edit it directly.

## Responsiveness

Both pages are responsive (mobile → desktop), verified to have no
horizontal overflow from 320px to 1440px. Key responsive structure on the
landing page:

- Hero `<h1>` wraps on small screens (do not reintroduce `white-space:nowrap`
  without a mobile override — the hero has `overflow:hidden`, so nowrap text
  gets clipped on phones).
- Signal strip: 4 → 2 → 1 columns.
- "Selected Work" gallery: 3 → 2 → 1 columns.
- Flex/grid children use `min-width:0` so they shrink instead of forcing
  page overflow.

### How to verify a change (no local server needed)

Render the file directly with headless Chromium and measure overflow.
Playwright + Chromium are available in Claude Code web sessions
(`NODE_PATH="$(npm root -g)"` to resolve the global `playwright`). Load
`file://<abs-path>/index.html`, wait ~1s for the loader to reconstruct the
page, then compare `document.documentElement.scrollWidth` against
`clientWidth` at several widths (320/375/414/768/1024/1280). For visual
screenshots, force the scroll-reveal animations on first
(`.reveal{opacity:1!important;transform:none!important;}`) — otherwise
below-the-fold sections stay invisible in a full-page screenshot.
