# Battlegroup Clasher

A browser-based map and tracking tool for tabletop-style wargaming, built around
real geography. Designed with [Sapper Studios' Battlegroup Clash: Baltics](https://www.sapperstudio.com/battlegr) in mind, but the data model is generic enough for most platoon- to battalion-level rules.

Current version: **v055** (beta).

## Quick start

1. Download `bg_clasher.html` for the latest version.
2. Open it in any modern browser (Chrome, Firefox, Edge, Safari — desktop). No
   install, no server, no accounts. The whole app is a single HTML file.
3. Use the role switcher (top-left: UMPIRE / BLUFOR / OPFOR) to choose your
   viewpoint. Start as **UMPIRE** to set up a scenario.
4. Click **Scenario** in the top bar to build an Order of Battle, define unit
   templates, and configure modifiers. When you're happy, **💾 Export Scenario**
   to save it to disk.
5. Place units, plot orders, measure ranges, manage modifiers, end turns.
   Hover the **?** button in the top bar to flash every tooltip in the UI at
   once, or click it to open the full in-app Manual.

Everything runs locally in the browser — nothing is sent anywhere.

## What is this for playtesters?

This is a **beta**. The core feature set is stable and I've tested it a bit, but I need outside eyes on:

- **Usability** — which workflows feel awkward, which buttons are hard to find,
  which concepts are unclear from the Manual alone.
- **Scenarios I haven't tried** — larger battles, nested formations, heavy
  modifier use, mounting chains, scenarios crossing the dateline or polar regions.
- **Cross-browser behaviour** — I've mostly tested in Firefox.
- **Data safety** — I want to know if any operation silently loses state. Always
  export before experimenting.
- **Rules-system fit** — Battlegroup Clasher ships with *Battlegroup Clash*
  modifier defaults, but the modifier library is editable per scenario. Tell me
  if the vocabulary (counter levels, permission model, visibility flags) works
  for other rules you care about.

## Feature tour

### Scenario building
- **Two-sided Order of Battle** (BLUFOR / OPFOR) organised as a tree of
  formations and units, with formation abbreviations.
- **Unit templates** with multiple mutually-exclusive states (e.g. Mounted /
  Dismounted), each carrying its own stats, movement rates, and range rings.
- **Modifier library**: overlays on units that are orthogonal to state. Can be
  booleans (REORG on/off) or counters with sequential levels (ATGM shots 2→1,
  Suppressed→Disrupted). Each modifier defines its colour, who can add/remove/step
  it, and whether the opponent can see it.
- **Map symbols** (APP-6 / custom SVG) and **game markers** (objectives, IEDs,
  named areas) with optional range rings.
- **Icon scale slider** with per-scenario min/max bounds so zoom levels can be
  tuned for different map scales.

### Map and tools
- **Leaflet-based** map (OpenStreetMap tiles by default). Zoom freely; icons
  and labels scale proportionally.
- **Unit placement** via Order of Battle (Deploy) or direct-from-template.
  Units get an APP-6 tactical icon (auto-generated or fetched from a
  GitHub/local icon repo), a designation label at bottom-left, and their
  parent formation's abbreviation at bottom-right.
- **Movement orders** — multi-segment paths with per-segment speed modes.
  Auto-computed ETAs, or lock the start/end times for convoy synchronisation.
- **Measure** and **Angle** tools with live snap; click on units or markers to
  use their exact position as a waypoint.
- **Drawing tools** — polyline, polygon, circle, arrow, text, symbol. Continuous
  drawing mode stays active after each shape; snap toolbar for angle/length
  constraints.
- **Layers** for drawings, with per-layer owner side, visibility toggle, and
  edit-rights restriction.

### Detection and roles
- **UMPIRE** sees everything. **BLUFOR** / **OPFOR** see only their own assets
  and whatever the umpire has marked as detected for their side.
- Three detection states per unit per side: **Unknown** (hidden),
  **Detected** (question-mark icon only), **Identified** (silhouette +
  numeric ID + fields flagged as visible to the opponent).
- Players can also place their own **Suspected enemy** markers — those are
  player-authored best guesses about where they think enemies are, independent
  of the umpire's detection system.
- **Game Mode** toggle: enforces a handover screen when switching roles and
  restricts the Scenario panel to umpire-only.

### Mount / dismount
- Any unit can carry any other unit as a passenger. Passengers are hidden from
  the map while mounted and move with their carrier. Dismount places them 250 m
  NW of the carrier, on top of the stack so they're immediately visible.

### Turns and synchronisation
- **End Turn** snapshots every unit's position, orders, notes, modifier state,
  and sync matrix; advances the turn counter; optionally auto-saves the scenario
  as `...T{N}_END_OF_TURN.json`.
- **Turn History** lets you jump back to any past snapshot (read-only until
  you revert).
- **Sync Matrix** per side: rows are formations and units, columns are turns,
  free-text cells for planned activities — useful for umpire pre-orders
  adjudication or player sync planning.

### Scenario portability
- **💾 Export Scenario** writes the full state (including all turn history) to
  a JSON file.
- **📂 Import Scenario** reads it back. Automatic migrations handle older file
  formats: modifier libraries are seeded if missing, detection states are
  renamed to match the current vocabulary, suspected-icon overrides are
  sanitised.

## Suggested playtest workflow

To shake out the most useful feedback in a 1–2 hour session:

1. **Set up a scenario from scratch** (5 min): name it, set a start date, pick
   a turn duration. Note whether the defaults feel reasonable.
2. **Build an OOB** (15 min): add 1 formation per side (give them abbreviations
   like "BG A" / "BG B"), add 2–4 units per formation with designations. Try
   mounting one unit inside another via the Unit Template's Available Modifiers
   whitelist.
3. **Deploy a few units** (5 min): from OOB to the map. Try the Deploy flow
   (recommended) and also the direct-from-template flow (▲ tool).
4. **Plot orders** (10 min): multi-segment moves, fixed start times, mixed
   speeds. Check that ETAs make sense and that unit icons show the right
   designation + formation labels.
5. **Try the modifier system** (15 min): toggle REORG on a unit, step the
   ATGM counter on another, mark a unit as Suppressed from the opposing role.
   Verify the opponent sees only the modifiers flagged visible-to-opponent.
6. **End a turn** (2 min): with "Save scenario file" checked. Make sure the
   saved file reloads cleanly.
7. **Reload in a fresh tab**: import the saved scenario. Everything should
   restore identically.
8. **Break things**: try unusual orderings (dismount a unit that just mounted,
   withdraw a carrier with passengers, delete an OOB entry that has a deployed
   unit, switch roles mid-modal). Report anything that crashes, loses data,
   or leaves the UI in a confused state.

## Reporting issues

When you hit a bug, please send me:

1. **What you did** (tool clicks, role, which panel was open).
2. **What you expected vs. what happened**.
3. **A scenario export** (💾 button) from right before the bug, if possible —
   that gives me exact state to reproduce from.
4. **Browser and OS** (e.g. "Firefox 125 on macOS 14").
5. **Console log** if the bug looks scripty (open Developer Tools → Console
   and copy any red lines).

Feature requests, wording suggestions, and rules-system compatibility notes
are all welcome too.

## Technical implementation

### Architecture

The whole app is a **single self-contained HTML file** (~380 KB for v055).
Opening the file:

- Loads Leaflet 1.9 and Leaflet.Draw from `unpkg.com` CDNs.
- Tile layer points at OpenStreetMap by default. No API keys required.
- Runs entirely in the browser. No backend, no persistent store beyond the
  files you export. This is intentional: it makes the tool easy to email,
  archive, and run offline (once cached).

### State model

All game state lives in a single top-level `state` object with roughly:

- `state.scenario` — metadata, icon scale, modifier library, suspected-icon
  overrides.
- `state.oob` — per-side trees of formations and units.
- `state.unitTemplates` / `markerTemplates` / `symbolTemplates` / `diceTemplates`
  — reusable definitions.
- `state.units` / `state.markers` — actually-placed entities on the map.
- `state.movementOrders` — pending per-unit movement orders.
- `state.layers` / `state.drawings` — arbitrary map drawings organised into
  layers with visibility and edit-rights flags.
- `state.turnHistory` — snapshots taken at each End Turn.
- `state.syncMatrix` — per-side planning matrix contents.
- `state.viewRole` — the current role (umpire / bluefor / opfor).
- Display-only flags (`showUnitAnchors`, `orderLabelMode`, `gameMode`, etc.)
  that aren't persisted into turn snapshots.

Everything in `state` round-trips through JSON cleanly, which is the basis for
both save/load and the Turn History snapshot/revert mechanism.

### Icons

Unit tactical icons come from three possible sources, in priority order:

1. **Custom SVG on the template or unit**, if provided.
2. **Local icon folder** configured in Scenario → General → "Local Icon Folder".
   Expected filenames are of the type `SIDE_Type_Name_Echelon.svg` (e.g. `BLUFOR_Mechanised_Infantry_Company.svg`).
3. **GitHub fallback**: fetched from the upstream repo of APP-6 icons. A tiny
   embedded set of "suspected enemy" fallbacks ship in the file so suspected
   markers render even offline.

Many thanks to [Spatial Illusions' Unit Generator](https://spatialillusions.com/unitgenerator/) for excellent tool for generating the SVG files!

Icon fetch results are cached in memory for the session. Failed fetches surface
in a dismissible top bar with a Retry button.

### Data migrations

`importScenario()` runs a forward-compatibility pipeline so older saves keep
working:

- Legacy suspected-icon filenames are stripped; scenarios fall back to the
  default.
- Detection-state values renamed (`'suspected'` → `'detected'`,
  `'detected'` → `'identified'`) so v047+ labels apply to old units.
- Missing modifier library is seeded with the Battlegroup Clash defaults.
- Units lacking a `modifiers` array get one so rendering code can iterate
  without null-guards.

### Dependencies

- [Leaflet](https://leafletjs.com/) 1.9 — map rendering and interaction.
- [Leaflet.Draw](https://github.com/Leaflet/Leaflet.draw) — polyline / polygon
  / circle / marker drawing tools.
- OpenStreetMap raster tiles (no key).
- Optional: user's own icon repo (GitHub or local folder).

No build step, no package.json, no framework. The rendering code is plain DOM
manipulation with template strings. This keeps the file small and editable in
any text editor.

## Browser compatibility

Tested and working:

- **Chrome / Edge** (Chromium 100+) — primary development target.
- **Firefox** 115+.
- **Safari** 16+ on macOS.

Known to **not** work:

- Internet Explorer. (ES2017+ syntax, modern CSS, arrow functions everywhere.)
- Mobile browsers are usable for viewing but the tool panels assume a
  landscape screen ≥ 1024px wide. Editing on a phone is painful.

Running from `file://` works but some browsers restrict cross-origin fetches
in that mode, which can prevent GitHub icon fetches. Two workarounds:

- Serve the file from a local web server (e.g.
  `python3 -m http.server` in the directory).
- Configure a local icon folder (Scenario → General) with the icons
  pre-downloaded.

## Known limitations in v055

These are deliberate simplifications for the beta, not bugs:

- **No rule automation**. Modifiers track state but don't apply any dice-roll
  modifiers or combat arithmetic for you — by design, to keep the tool
  rules-agnostic. You adjudicate off the chips.
- **Carrier movement undo**. Pressing undo after a carrier drag rolls back the
  carrier but not its passengers' stored lat/lng. Usually harmless (passengers
  are hidden while mounted) but visible if they're dismounted before the next
  move.
- **No multi-user sync**. Pass the scenario file between roles, or run multiple
  browsers on the same file.
- **No automatic line-of-sight calculation**. All visibility is umpire-set.
- **Mobile layout**. Works at read-only but not meaningfully editable.

## Keyboard shortcuts

| Key | Action |
|---|---|
| U | Place Unit |
| K | Place Game Marker |
| Y | Place Map Symbol |
| M | Movement order |
| L | Polyline |
| P | Polygon |
| C | Circle |
| A | Arrow |
| T | Text |
| D | Measure distance |
| V | Measure angle |
| E | Edit / delete |
| Esc | Cancel current action / close modal |
| Enter | Finish current drawing (where applicable) |

## Feedback

Send bug reports, feature requests, and general thoughts to Janne directly.
Including a scenario export makes issues 10× easier to reproduce.

## Licence

Proprietary during beta — please don't redistribute the file or derivative
scenarios without checking with Janne first. A public licence will follow
once the tool is stable.

Third-party components retain their own licences:

- Leaflet — BSD 2-Clause
- Leaflet.Draw — MIT
- OpenStreetMap tiles — ODbL (attribution required on shared screenshots:
  "© OpenStreetMap contributors")

---

*Battlegroup Clasher is built and maintained by Janne. Not affiliated with
the Battlegroup Clash rules authors.*
