# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⛔ Before touching code

**Never merge or push directly to `main` or `stable`.** All changes cross into them
via a Pull Request that Charlie reviews and merges on GitHub: feature/fix branch →
commit → push the branch → open a PR. Never run a local `git merge` into
`main`/`stable` and never `git push origin main`/`stable`. See
`docs/GIT_WORKFLOW.md` for the full playbook.

## What this is

Howie Helper is a phone-first, installable PWA suite of tools for running a Hungry
Howie's store. No build step, no backend, no accounts — it is a **single static
`index.html`** (~4200 lines: inline `<style>`, HTML skeleton, one big inline
`<script>`). All state lives in `localStorage`; everything runs offline. Vendored
libraries (`vendor/html2canvas.min.js`, `vendor/jspdf.umd.min.js`) are used for
image/PDF export from some tools.

## Run / develop

No build, no tests, no linter. Edit `index.html` and reload.

- **Serve locally:** `python3 -m http.server` in the repo root, then open the
  printed URL (use the LAN IP on your phone, same Wi-Fi) and "Add to Home Screen."
- **Live:** GitHub Pages serves `stable` at https://charliedevx.github.io/howie-helper/
- **Always-on dev preview:** `main` auto-publishes to `.../preview/` via
  `.github/workflows/deploy.yml` (isolated storage keys + a "DEV" badge).

## Architecture

Everything is in `index.html`'s single IIFE. The shape:

- **Shell / router.** `appView` (`"home"` | `"tool"`) + `activeTool` drive
  `renderApp()`, which writes into three fixed containers: `#bar`, `#view`,
  `#footer` (handles `elBar`/`elView`/`elFooter`). Rendering is plain
  `innerHTML = \`...\`` template strings; event handlers are wired up by
  re-querying the DOM after each render (`elView.querySelectorAll(...).onclick=...`).
  There is no framework and no virtual DOM — **a "render" fully rewrites the
  container's HTML, so re-attach listeners every render.**

- **Tool registry.** `const TOOLS = [...]` lists every tile with `id`, `name`,
  `icon`, `status` (`"active"` | `"soon"`), `descriptor`. `renderHome()` draws
  tiles from it; `"soon"` tiles are inert placeholders. `TOOL_VIEWS` maps a tool
  `id` → its render function (e.g. `nightlyInventory: ()=>Inv.render()`).

- **Tools are self-contained IIFE modules.** Each (`Inv`, `DrawerCash`, `Sched`,
  `Numbers`, `DoughBuild`) is `const X = (function(){ ... return { render, reset } })()`.
  A tool **draws its own bar, view, and footer** — `renderApp()` just calls
  `TOOL_VIEWS[id]()`. Most expose `reset()`, called from `openTool()` / `boot()`
  to re-enter the tool at its top view. Tools with multiple screens keep their own
  internal `route` state and re-render themselves.

- **Persistence.** Each module owns one `localStorage` key, all versioned:
  `howiehelper.shell.v1` (shell prefs: landing/last tool), `howiehelper.v1`
  (inventory), `howiehelper.drawerCash.v1`, `howiehelper.scheduleMaker.v1`,
  `howiehelper.numbers.v1`, `howiehelper.doughBuild.v1`. Loads/saves are wrapped in
  `try/catch` and degrade silently.

- **Shared helpers** (top of script): `uid()`, `fmt()` (number formatting),
  `esc()` (HTML-escape — use it for any user text put into a template string),
  `slug()`, `toast()`, `homeBtnHTML()`. The PWA manifest is generated at runtime as
  a blob (`injectManifest()`) — there is no separate manifest file.

### Adding a new tool

1. Add a row to `TOOLS` with a unique `id` and `status:"active"`.
2. Write a `const MyTool = (function(){ ... return { render, reset } })()` module
   with its own `localStorage` key `howiehelper.<id>.v1`.
3. Register it in `TOOL_VIEWS`: `myTool: ()=>MyTool.render()`.
4. If it needs reset-on-enter, add `if(id==="myTool") MyTool.reset();` in `openTool()`.
5. **Add the new storage key to the `sed` block in `.github/workflows/deploy.yml`**
   so the dev preview stays isolated from live data (see below).

## Deploy / data isolation (important)

`main` and `stable` are served from the same `github.io` origin, so they share
`localStorage`. The deploy workflow rewrites the `/preview/` (`main`) build with
`sed` to suffix every storage key with `.dev` and relabel the app "DEV". **Any new
or renamed storage key must be added to that `sed` block**, or the dev preview will
read/write live users' data.

## Git workflow (three layers, PR-only crossings)

Full playbook in `docs/GIT_WORKFLOW.md`. **The rule: never merge or push directly
to `main` or `stable` — they only change through a PR Charlie reviews and merges
on GitHub.**

- **`stable`** = what people run (GitHub Pages root). Changes **only** when a
  release PR is merged.
- **`main`** = working/integration branch. Features land here **via PR**;
  auto-published to `/preview/` but not to live users.
- **`feature/<thing>` / `fix/<thing>`** = one focused piece of work each, branched
  off `main`.

Everyday loop: branch off `main` → build & commit in small present-tense commits →
test locally → push the branch → **open a PR into `main`** (Charlie reviews &
merges) → delete the branch after merge. Releasing: **open a PR from `main` into
`stable`** for Charlie to merge — never a local merge or direct push. Pushing a
feature branch never changes what live users run.
