# HowieHelper — Home Screen / App Shell (Feature Spec)

A feature spec for the **home screen** of HowieHelper — the launcher the user lands on to pick which store tool to use. Written to hand to Claude Code for implementation. Pairs with `HowieHelper_Inventory_Counter_Spec.md` (the first tool, now labeled **Nightly Inventory**).

## Purpose

HowieHelper is growing from one tool into a small suite of store tools. The home screen is the shell that holds them: a simple launcher that lists the available tools, lets the user open one, and gets out of the way. New tools get added to the list over time, so the shell must make adding/removing tools easy.

## Platform

Same as the rest of the app: installable web app / PWA, phone-first, runs offline, all data on-device (localStorage). No accounts, no backend.

## Navigation model

- **Open behavior: jump to last-used tool.** On launch, the app reopens whatever tool the user was last in. If there is no last-used tool (first run, or last tool no longer exists), it opens the home screen.
- **Home button.** Every tool shows a Home control (e.g. a house icon in the top bar) that returns to the home screen / tool picker.
- **Back within a tool** keeps working as it does today (e.g. Nightly Inventory's Count → Overview → back).
- The last-used tool is persisted to localStorage and updated whenever the user opens a tool.

```
last-used tool persisted  ──▶  launch reopens it
        ▲                              │
        │  (open a tool)               │ (Home button)
        └──────────  Home screen  ◀────┘
```

## Home screen layout

**Vertical list** of tool rows (full-width, one per tool — chosen for clarity and longer names).

Top bar:
- App title **Howie Helper**.

A **Settings** entry sits at the **bottom of the home screen** (below the tool list), not in the top bar — see the Settings section.

Each tool row shows:
- An **icon** (left).
- The **tool name** (bold).
- A **one-line descriptor** (muted).
- A **status indicator** on the right:
  - **Active** tools are fully tappable and open the tool.
  - **Coming soon** tools show a muted "Coming soon" badge, are visually greyed, and are **not** tappable (tap does nothing, or shows a small "not ready yet" toast).

Rows are grouped only if useful; a single flat list is fine for v1. Order is configurable in code (see registry) and should be easy to change.

## Tool registry (the list)

Drive the home screen from a single registry array so tools are easy to add/remove/reorder. Suggested shape:

```
Tool {
  id            // stable key, e.g. "nightlyInventory"
  name          // "Nightly Inventory"
  descriptor    // short line under the name
  icon          // emoji or icon name
  status        // "active" | "soon"
  route         // which view/tool to open when tapped (active only)
}
```

### v1 registry

| Order | Tool | Status | Descriptor (draft) |
|---|---|---|---|
| 1 | **Nightly Inventory** | active | Fast nightly count → on-hand totals to key in |
| 2 | **Prep / Backup Tracker** | soon | Track prep & backups through the night |
| 3 | **Dough Build** | soon | How much dough to build to |
| 4 | **Sheet-out Projector** | soon | Project skins/dough to sheet out |
| 5 | **Drawer Cash** | soon | Count the cash drawer |
| 6 | **Sales / Labor Projections** | soon | Data-driven sales & labor forecasts |
| 7 | **Schedule Generator** | soon | Build the staff schedule |

Notes:
- **Nightly Inventory** is the existing inventory counter tool, just relabeled. Everything in `HowieHelper_Inventory_Counter_Spec.md` is that tool's internal behavior.
- Tools 2–7 only need placeholder ("Coming soon") tiles for now; their functionality is out of scope.
- The data-driven tools (#6) still depend on getting manager/Jason permission to pull store POS data — placeholder only for now.

## Coming-soon behavior

- "Coming soon" tiles exist so the home screen previews the roadmap, but they do nothing yet.
- Adding a real tool later = flip its `status` to `active`, give it a `route`, and build the tool behind it. No home-screen redesign required.

## Settings (two levels)

Settings are split into **global** and **per-tool**:

- **Global settings** — a **Settings** row pinned to the **bottom of the home screen**. Holds app-wide configuration that isn't specific to any one tool (e.g. store name/branding, default landing behavior, app reset). Opened from home, with a back/Home control to return.
- **Per-tool settings** — **each tool/page has its own settings connection** for that page's configuration, reached from inside the tool (e.g. a gear in the tool's top bar). Nightly Inventory's existing settings (item list, sections, per-item constants, reset counts) become this tool's per-tool settings — unchanged in behavior, just reframed as belonging to that page. Every future tool ships with its own settings entry the same way.

This keeps each tool's configuration with the tool, while global app settings live in one place off the home screen.

## Persistence

- `lastTool`: id of the last-opened active tool (for jump-to-last-used).
- Each tool keeps its own storage keys (e.g. Nightly Inventory's existing `howiehelper.v1` night data) — the shell doesn't touch tool data.

## Out of scope (v1)

- The actual functionality of any "Coming soon" tool.
- Multi-user sync, logins, cloud backup.
- Reordering tools from the UI (order is set in code for v1).

## Decided

- App name shown in the UI: **Howie Helper**.
- Settings: **global** Settings row at the bottom of the home screen, plus **per-tool** settings inside each tool (Nightly Inventory's current settings become its per-tool settings).
- Tool names confirmed: Nightly Inventory, Prep / Backup Tracker, Dough Build, Sheet-out Projector, Drawer Cash, Sales / Labor Projections, Schedule Generator.
