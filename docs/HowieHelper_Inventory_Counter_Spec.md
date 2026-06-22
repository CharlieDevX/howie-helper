# HowieHelper — Inventory Counter (Feature Spec)

A feature spec for the first tool in **HowieHelper**, a phone app for running a Hungry Howies store. Written to be handed to Claude Code for implementation.

## Purpose

A fast counting aid for the nightly inventory. The user walks the store, counts each item using whichever input is easiest, and the app shows the **on-hand total per item** to key into the store computer.

The store computer already produces targets and order quantities once counts are entered, so this app does **not** handle par levels, targets, ordering, or projections. It only makes *collecting the count* faster.

## Platform (recommended, his call)

Installable web app / PWA for v1 — works on the phone via "Add to Home Screen," runs offline, stores data on the device. No accounts, no backend. Can move to native later if it gets adopted. The spec below is behavior-focused and works for either.

## Core model

Items belong to **sections**: Pop, Cheese, Meats, Boxes (list is editable/reorderable). Pop is first; within Pop the order is Water, Gatorade, 2-liter, 20oz. Each item has a **count method** that decides its input fields and total math. Three methods:

1. **Weight** — cheese and meats (pounds)
2. **Box stacks** — pizza/bread/deep-dish boxes
3. **Pop** — crates + cooler rows

Every item produces one number: its **On Hand total**, derived live from the inputs.

## Screens

### 1. Overview (landing for the feature)

- Header: date (auto-filled, editable) and "Counted by."
- Progress indicator: "X of Y items counted."
- Item list grouped by section. Each row shows: item name, its current On Hand total (or "—" if not yet counted), and a checkmark once counted.
- Tap a row to open its Count screen.
- Footer button: **Summary / Export**.

### 2. Count screen (one item at a time)

- Shows the item name, its method, and its preset constants (weights, stack sizes, crate/row units) as read-only reference (editable in Settings).
- Shows the method's input fields (see below).
- Shows the live **On Hand total** prominently — updates instantly on every input change.
- Helper text where useful (e.g., pan conversions for meats).
- Buttons: **Save & Next** (advance to next item) and **Save & Back** (return to Overview).

### 3. Summary / Export

- Clean list of every item and its on-hand total, ordered the way the store computer expects entry.
- Copy-to-clipboard and/or share.
- "Reset all counts" to start fresh for the next night.

## Combined input control (num pad + clicker)

This is the key interaction. **Every numeric field is one control that supports both:**

- A large value display in the middle.
- **− and +** clicker buttons on each side for precise, one-at-a-time tally counting (good for bags, stacks, crates while walking the store).
- Tapping the value opens an on-screen **num pad** overlay (0–9, clear, done) for fast entry of bigger numbers (good for eye-weighed line pounds, loose box counts).

So the same field handles "tap +1 for each bag" and "type 24" without switching modes.

## Count methods & math

### Weight items (Cheese, Meats) — unit: pounds

Fields: **Boxes**, **Bags**, **On-line lb** (eye-weighed amount on the make line).
Per-item constants: **Box lb**, **Bag lb**.

`Total lb = Boxes × BoxLb + Bags × BagLb + OnLineLb`

Presets:

| Item | Box lb | Bag lb |
|---|---|---|
| Mozzarella | 30 | 5 |
| Pepperoni | — | 12.5 |
| Sausage | — | 5 |
| Beef | — | 5 |
| Bacon | — | 5 |
| Grilled chicken | — | 5 |
| Ham | 18 | 6 |

Pan helper notes (show on the count screen): 5 lb bag = 3 × 1/6 pans; 12.5 lb pepperoni = 3 × 1/3 pans; ham 18 lb box = 3 × 6 lb bags, 1 bag = one 1/3 pan.
Suggested input feel: Boxes/Bags = clicker; On-line lb = num pad.

### Box items — unit: boxes

Fields: **Unfold stacks**, **Fold stacks**, **Loose**.
Per-item constants: **Per-unfold-stack**, **Per-fold-stack** ("na" where that size isn't folded into stacks).

`On Hand = UnfoldStacks × PerUnfold + (FoldStacks × PerFold, if applicable) + Loose`

Presets:

| Item | Per unfold stack | Per fold stack |
|---|---|---|
| Pizza boxes (junior) | 100 | na |
| Pizza boxes (small) | 100 | na |
| Pizza boxes (medium) | 100 | 15 |
| Pizza boxes (large) | 50 | 15 |
| Pizza boxes (X-large) | 50 | na |
| Bread boxes | 100 | 15 |
| Deep Dish boxes | 100 | na |

Suggested input feel: stacks = clicker; Loose = num pad. When per-fold-stack is "na," hide/disable the Fold stacks field.

### Pop items — unit: individual units

Fields: **Crates**, **Loose (crate)**, **Front (bottles)**, **Back (bottles)**.
Per-item constants: **Units/crate**, **Front max**, **Back max**. Front/Back max are reference caps (the full-shelf count). **Back max blank/"na" means the item has no back-cooler row, so the Back field is hidden.**

Counting rules (revised after real use):

- **Cooler rows are counted as actual bottles, not full rows.** Front and Back are +1/−1 tallies; the Front/Back max values are just the full-shelf reference so you know the cap. Rows are usually partial after a night of sales.
- **Each row is one soda type.** The Front/Back fields show a hint like "up to 7 / type" (the per-row, per-flavor capacity). For v1 this is just a reference label — an in-depth per-flavor type counter is a separate future tool, not part of this one.
- **Crates are counted as full crates × Units/crate, plus a Loose counter.** A full crate is the Units/crate value, but some crates are partial/opened — the **Loose** field tallies the individual bottles in those.

`Total = Crates × UnitsPerCrate + LooseCrate + FrontBottles + BackBottles`

Cooler layout / presets:

| Item | Units/crate | Front cooler | Back cooler |
|---|---|---|---|
| 2-liter bottles | 8 | 2 rows × 5 = **10 max** | 1 row × 5 = **5 max** |
| 20oz bottles | 24 | 1 row × 7 = **7 max** | 1 row × 7 = **7 max** |
| Gatorade | 24 | 2 rows × 7 = **14 max** | 1 row × 7 = **7 max** |
| Water | 24 | 1 row × 7 = **7 max** | 1 row × 7 = **7 max** |

Suggested input feel: Crates and Front/Back bottles = clicker (+1 each); Loose and large counts = num pad available.

## Settings (light, v1)

- Edit the item list: add/remove items, reorder, reorder sections.
- Edit per-item constants (weights, stack sizes, crate/row units).
- Reset counts.

## Data model (suggestion)

```
Item {
  id
  name
  section        // "Cheese" | "Meats" | "Boxes" | "Pop"
  method         // "weight" | "box" | "pop"
  constants      // method-specific, e.g. { boxLb, bagLb } / { perUnfold, perFold } / { perCrate, perRow }
  counts         // method-specific entered values
  total          // derived
  counted         // bool
}
```

Persist to device storage, keyed by count date. (localStorage for the PWA version.)

## Out of scope (v1)

- Par levels, targets, order quantities — the store computer handles these.
- Sales/labor projections, scheduling — separate future features.
- Multi-user sync, logins. Data stays local to the device for now.

## Open questions for later

- Should counts auto-save per item or only on "Save"?
- Do you want a per-section subtotal on the Overview?
- Any items missing from the four sections above?
