# BG Clasher — Agent Guide

Context file for agentic coding tools (Claude Code reads `CLAUDE.md`, Codex reads
`AGENTS.md` — keep both identical, they are copies of this file).

## What this is

A personal, browser-based wargame map tool built around the *Battlegroup Clash*
ruleset, used live with playtesters. It is a **single self-contained HTML file**
with no backend or build step. Built on Leaflet.js + Leaflet.Draw, with milsymbol
for APP-6 icons and MapLibre/OpenFreeMap for optional vector reference layers.
Publicly hosted via GitHub Pages.

- Repo: `https://github.com/jmkorhonen/bg_clasher`
- Working file: `bg_clasher.html` (repo root)
- Published outputs: `bg_clasher_v{NNN}.html` (zero-padded, monotonically increasing)
- Icons: `icons/` → served at `https://raw.githubusercontent.com/jmkorhonen/bg_clasher/main/icons/`

## Hard constraints (do not violate without explicit instruction)

1. **One file.** All HTML, CSS, and JS live in the single `.html` file. No
   separate `.css`/`.js`, no modules, no build step, no bundler, no npm.
2. **No backend.** Everything runs client-side. Core scenario work remains usable
   without a server; raster/vector tiles and remote icons require network access.
3. **CDN dependencies only**, loaded via `<script>`/`<link>` from unpkg. Current
   dependencies are Leaflet 1.9.4, Leaflet.Draw 1.0.4, milsymbol 3.0.4,
   MapLibre GL 5.24.0, and maplibre-gl-leaflet 0.1.3. Do not add a package manager
   or npm dependencies.
4. **Vanilla JS only.** No frameworks (React/Vue/etc.), no TypeScript, no JSX.
5. **Inline event handlers** (`onclick="foo(...)"`) are used throughout, so any
   function referenced from markup **must be global** (declared as a top-level
   `function foo()` inside the single `<script>`). Don't wrap things in modules
   or closures that would break inline handlers.

## Running & testing

There is no build. To test, open the file in a browser. Because it fetches map
tiles and icon SVGs over the network, serve it locally rather than via `file://`:

```bash
python3 -m http.server 8000   # then open http://localhost:8000/bg_clasher.html
```

Tiles need internet; icon SVGs resolve from the GitHub raw URL above. Multi-display
sync uses `BroadcastChannel`, so test mirror windows as two tabs on the same origin.

## Architecture

Single global `state` object holds everything (scenario, viewRole, turnNumber,
templates, OOB trees per side, scenario drawing layers, map layers, units,
markers, drawings, movementOrders, logs, turnHistory). Mutating `state` then
calling the relevant render function is the core loop — there is no reactive
framework.

Navigate the code by the banner comments (`// ── SECTION ──`), not line numbers,
since the file is re-versioned often. Major sections:

- **State + default config/templates** — top of the `<script>`.
- **Map + map layers** — `L.map(...)`; one selected raster background plus
  independently ordered OpenFreeMap vector categories, imported GeoJSON, and
  geodesic hex grids live in `state.mapLayers`.
- **Unit / Order / Modifier systems** — core game objects and their APP-6-style rendering.
- **Scenario Builder** — general settings, modifiers, marker/symbol/dice templates.
- **Units panel** — umpire OOB builder for both sides and Unit Templates; player
  roles see only their own unit/OOB view.
- **Map Layers panel** — collapsible Scenario Layers and Map Layers groups.
- **Unit edit panel**, **Markers panel**, **Turns panel**, **Dice panel**, **Log panel**.
- **`init()`** — bottom; wires everything up.

## Persistence

No localStorage autosave. State is saved/loaded by **JSON file export/import**
(`Blob` download / `FileReader` upload). Scenario/OOB/templates are included in the
export. When changing the `state` shape, bump `state.version` and handle migration
on import.

Background-map scenario defaults are captured in memory when a scenario is
created, imported, or exported. "Restore scenario defaults" restores those
saved map-layer properties/order while retaining map layers added afterward and
never modifying `state.layers` (Scenario Layers).

## Icons (APP-6)

SVGs auto-resolve from the GitHub `icons/` folder by filename convention:
`SIDE_Type_Name_Echelon.svg`. Resolution is async with caching; icon scale is
configurable. Keep this naming scheme intact.

## Multi-display

"Pattern A": `BroadcastChannel` mirrors the main window to display windows, with
game-mode sync, undo sync, and optional per-role passwords. Mirrors have an
independent role and map view. (Networked play was considered and rejected in
favour of this simpler approach — don't re-introduce networking.)

## Roles

`bluefor` / `opfor` / `umpire`. Visibility is **role-based, not game-mode-dependent**:
opponent layers are fully hidden from players; editing restrictions apply in game mode.

## Workflow norms (how Janne works)

- **One concern per change.** Fix or build one thing; don't over-engineer or
  refactor adjacent code unasked.
- **Confirm before complex features.** Ask targeted clarifying questions on
  architecture/design decisions before building.
- **Brief rationale.** Keep explanations concise.
- **Versioned delivery.** Each accepted change produces the next
  `bg_clasher_v{NNN}.html`; keep the working `bg_clasher.html` current. Update the
  in-app manual and `README.md` when features change.
- Feedback is precise (exact angles, geometric descriptions) — implement to spec.

## Don'ts / reverted features

- **Seize order** was implemented and deliberately reverted — do not re-add unless
  explicitly requested.
- Don't surface the computed **sun altitude** number on the topbar (it was mistaken
  for temperature); keep it in the tooltip only.
- Don't split the file, add a build, or add a backend.
