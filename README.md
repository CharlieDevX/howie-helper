# Howie Helper

A phone-first PWA suite of tools for running a Hungry Howie's store. Installable,
runs offline, all data stays on the device (localStorage) — no accounts, no backend.

The home screen is a launcher; **Nightly Inventory** is the first live tool. Other
tools (Prep / Backup Tracker, Dough Build, Sheet-out Projector, Drawer Cash,
Sales / Labor Projections, Schedule Generator) are "Coming soon" placeholders.

## Run it

It's a single static file — no build step.

- **Locally:** `python3 -m http.server` in this folder, then open the printed URL
  on your phone (same Wi-Fi) and "Add to Home Screen."
- **Live:** GitHub Pages serves the `stable` branch.

## Branches (git workflow)

Three layers, two safety gaps — full playbook in [`docs/GIT_WORKFLOW.md`](docs/GIT_WORKFLOW.md).

- **`stable`** — what people actually run (GitHub Pages deploys this). Changes only
  on a deliberate release.
- **`main`** — working/integration version. Finished, tested features land here.
- **`feature/*` · `fix/*`** — one focused piece of work each, branched off `main`.

```
stable (LIVE) ──●───────────────●──▶  (changes only on release)
                 ▲ promote        ▲
main (dev)  ──●──●────────●───────●──▶
              \  /  merge when done
 feature/...  ●─●
```

## Docs

- [`docs/GIT_WORKFLOW.md`](docs/GIT_WORKFLOW.md) — git playbook
- [`docs/`](docs/) — feature specs (home screen, inventory counter)
