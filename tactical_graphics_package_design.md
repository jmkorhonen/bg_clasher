# Open Tactical Graphics Package Design

## Purpose

Build an open source JavaScript/TypeScript package for drawing and editing
military tactical graphics for map-based tools.

The package should fill the gap between:

- `milsymbol`, which is excellent for APP-6 / MIL-STD-2525 unit and point
  symbols, with SVG and Canvas output.
- Mission Command `mil-sym-ts` / `mil-sym-java`, which can render multipoint
  MIL-STD-2525 tactical graphics, but is not primarily an ergonomic authoring,
  editing, and map-integration toolkit.

The intended first consumer is BG Clasher, but the package should be usable by
other browser map applications.

## Short Product Statement

This package should be "milsymbol for tactical graphics": a small, documented,
testable library that turns a symbol definition plus control points into
editable geometry and renderable vector output.

It should not try to become a full command-and-control system. It should solve
the practical map-tool problem:

1. Choose a tactical graphic.
2. Click map control points.
3. See the graphic while drawing.
4. Select it later.
5. Drag meaningful handles.
6. Serialize it.
7. Render it consistently across zoom levels and applications.

## Context And References

Useful existing projects:

- `milsymbol`: https://github.com/spatialillusions/milsymbol
  - Pure JavaScript.
  - Generates MIL-STD-2525 / STANAG APP-6 unit symbols.
  - Supports SVG and Canvas output.
  - MIT licensed.
  - Does not appear to be aimed at multipoint tactical graphics authoring.

- `missioncommand/mil-sym-ts`: https://github.com/missioncommand/mil-sym-ts
  - TypeScript port of US Army Mission Command rendering libraries.
  - Supports MIL-STD-2525D/E.
  - Exposes icon and web renderer APIs.
  - Apache-2.0 licensed.
  - Valuable reference and possible fallback renderer, but not enough by itself
    for a friendly map-editing package.

- `missioncommand/mil-sym-java`: https://github.com/missioncommand/mil-sym-java
  - Java renderer for icon symbols and geometric tactical graphics.
  - Apache-2.0 licensed.
  - Useful as a reference implementation and visual comparison source.

Avoid using proprietary implementations or copying proprietary examples.
Reference images and user-supplied sketches are acceptable as test fixtures if
their licensing is clear.

## Design Goals

- Pure geometry core with no dependency on Leaflet, OpenLayers, DOM, Canvas, or
  browser globals.
- Deterministic rendering from data.
- Map-scale tactical marks by default: repeated markings and symbol dimensions
  are measured in map units/meters, not screen pixels.
- Explicit scale modifier for intentionally enlarging or shrinking tactical
  marks.
- Good editing model: each control point has a semantic label and optional
  constraint behavior.
- Multiple output targets:
  - SVG primitives.
  - GeoJSON-like vector primitives.
  - Optional Canvas drawing instructions later.
- Leaflet adapter first.
- Easy to bundle into a single offline HTML application.
- Small starter symbol catalog, expanded incrementally.
- Strong visual regression tests using reference fixtures.

## Non-Goals For Version 1

- Complete MIL-STD-2525 / APP-6 coverage.
- Network collaboration.
- Backend services.
- Advanced military data model beyond what is required to render and edit
  graphics.
- Automatic order-of-battle reasoning.
- Proprietary symbology imports.
- Perfect conformance for every historical version of every standard.

## Package Shape

Recommended package name placeholder:

```text
@open-tactical-graphics/core
```

Possible later packages:

```text
@open-tactical-graphics/leaflet
@open-tactical-graphics/openlayers
@open-tactical-graphics/examples
```

For a first implementation, a single package with optional adapters is enough:

```text
src/
  core/
    types.ts
    geometry.ts
    render.ts
    symbols.ts
    edit.ts
    serialize.ts
  renderers/
    svg.ts
    geojson.ts
  adapters/
    leaflet.ts
  symbols/
    lines.ts
    boundaries.ts
    obstacles.ts
    tasks.ts
  tests/
  examples/
```

Build outputs:

```text
dist/index.esm.js
dist/index.cjs
dist/open-tactical-graphics.umd.js
dist/open-tactical-graphics.umd.min.js
dist/index.d.ts
```

The UMD build matters because BG Clasher is a single-file browser app. It can
load the package from a CDN during development and later inline or vendor the
UMD output if needed.

## Core Data Model

### TacticalGraphic

```ts
export interface TacticalGraphic {
  id?: string;
  symbolId: string;
  standard?: "APP6" | "2525C" | "2525D" | "2525E";
  affiliation?: "friendly" | "hostile" | "neutral" | "unknown";
  status?: "present" | "planned" | "suspected";
  points: GeoPoint[];
  modifiers?: Record<string, string | number | boolean | null>;
  style?: TacticalStyle;
  scale?: number;
}
```

### GeoPoint

```ts
export interface GeoPoint {
  lat: number;
  lon: number;
  alt?: number;
}
```

### TacticalStyle

```ts
export interface TacticalStyle {
  stroke?: string;
  fill?: string;
  strokeWidth?: number;
  opacity?: number;
  fillOpacity?: number;
  textColor?: string;
}
```

### SymbolDefinition

```ts
export interface SymbolDefinition {
  id: string;
  names: string[];
  sidc?: string;
  category: "line" | "area" | "point-task" | "obstacle" | "boundary";
  minPoints: number;
  maxPoints?: number;
  defaultStyle?: TacticalStyle;
  defaultScale?: number;
  drawMode: "point" | "line" | "area";
  controls: ControlPointDefinition[];
  render: RenderFunction;
}
```

### ControlPointDefinition

```ts
export interface ControlPointDefinition {
  id: string;
  label: string;
  role:
    | "endpoint"
    | "vertex"
    | "anchor"
    | "direction"
    | "width"
    | "radius"
    | "label-position";
  constraint?: ControlConstraint;
}
```

### Control Constraints

Some handles are not free vertices. This is essential for useful editing.

Examples:

- Bridge/gap width handle:
  - Label: `Adjust width`.
  - Always lies on the perpendicular normal through the midpoint of endpoint 1
    and endpoint 2.
  - Dragging endpoint 1 or endpoint 2 preserves current width.

- Point task symbol width handle:
  - Controls footprint width relative to direction axis.

- Sector boundary echelon marker:
  - Not a draggable geometry point in V1.
  - Always at the lengthwise midpoint unless later overridden.

## Coordinate Spaces

The core must distinguish three coordinate spaces:

1. Geographic coordinates:
   - Input and serialized data.
   - Latitude/longitude.

2. Local projected coordinates:
   - Internal geometry calculations.
   - Meters.
   - Use a local tangent/equirectangular projection centered on the graphic or
     on the current map viewport.

3. Display coordinates:
   - Final SVG or screen coordinates.
   - Pixels.

Do not define repeated tactical markings in screen pixels. Define them in meters
and convert for the target viewport.

Example:

```ts
renderGraphic(graphic, {
  projection,
  viewport,
  metersPerPixel,
});
```

`scale` multiplies symbol dimensions before projection:

```ts
effectiveBoxLengthMeters = baseBoxLengthMeters * (graphic.scale ?? 1);
```

## Geometry Engine

Core geometry utilities required:

- Distance along polyline.
- Point at distance along polyline.
- Tangent and normal at distance.
- Offset polyline by signed distance.
- Repeated marks along polyline.
- Split polyline around a centered label or echelon marker.
- Arc and semicircle generation.
- Polygon centroid and label placement.
- Hit geometry generation for easier selection.
- Local projection helpers.

Suggested primitives:

```ts
export interface RenderLine {
  type: "line";
  points: Vec2[];
  style: TacticalStyle;
  hitWidth?: number;
}

export interface RenderPath {
  type: "path";
  d: string;
  style: TacticalStyle;
  hitWidth?: number;
}

export interface RenderText {
  type: "text";
  position: Vec2;
  text: string;
  angle?: number;
  style: TacticalTextStyle;
}

export interface RenderPolygon {
  type: "polygon";
  points: Vec2[];
  style: TacticalStyle;
}
```

The core renderer returns primitives. SVG/GeoJSON/Leaflet adapters consume them.

## Rendering Rules For Initial Symbols

### 1. FLOT

Forward Line of Own Troops.

Input:

- Polyline with at least two points.
- Optional `frontSide`.

Rules:

- Render independent half-circles along the line.
- Do not draw a connecting line across the tops of the half-circles.
- Semicircles face the frontage side.
- Text `FLOT` appears near both ends.
- Text is aligned with the line and kept readable.
- Half-circles are true arcs or high-resolution approximations, not low-sided
  polygons.

### 2. FLET

Forward Line of Enemy Troops.

Rules:

- Similar to FLOT but with stacked opposing/paired marks according to the chosen
  reference.
- Ensure paired semicircles stack cleanly rather than facing away incorrectly.

### 3. FEBA

Forward Edge of Battle Area.

Rules:

- Repeated circle-with-X or agreed reference mark along a line.
- Map-scale spacing.

### 4. Fortified Line

Rules:

- Pattern resembles repeated open boxes connected along the baseline.
- Every module is equal length.
- Adjacent modules connect exactly.
- No random gaps from remainder rounding.
- Dynamic preview while drawing.

Implementation hint:

- Divide each segment into an integer number of equal modules.
- Each module owns a fixed fraction for the open box and a fixed fraction for
  the connector.
- The final module either ends at the segment endpoint cleanly or carries no
  extra connector past the endpoint.

### 5. Bridge / Gap

Rules:

- Shape resembles:

```text
\   /
 | |
/   \
```

- Control point 1: `Endpoint`.
- Control point 2: `Endpoint`.
- Control point 3: `Adjust width`.
- Width point is constrained to the midpoint normal.
- Moving either endpoint preserves width.
- Moving width handle changes only width, not endpoint positions or midpoint.

### 6. Sector Boundary

Rules:

- Line is split around the echelon marker.
- Echelon marker is centered at line midpoint.
- Marker is rotated in line with the local boundary segment.
- Company/battalion/regiment can use short colored bars instead of serif `I`.
- Brigade/division use centered `X` / `XX`.
- Gap should be only large enough for the marker, roughly one `X` of padding on
  either side.

### 7. Phase Line / LD / LOA

Rules:

- Straight or polyline.
- Labels at both ends.
- Text vertically centered on the line.
- Object name is taken from editable modifier/label, not hard-coded examples.

### 8. Attack Direction / Main Attack

Rules:

- Polyline with arrowhead.
- Direction and shape editable by control points.
- Supporting attack uses dashed line.
- Later: add doctrinal arrow variants.

### 9. Point Task Symbols

Rules:

- Use `milsymbol` where possible for point task symbols.
- Provide handles:
  - Anchor/move.
  - Direction/size.
  - Width/footprint.
- Keep generated SVG editable/replaceable by consumers.

## Symbol Catalog V1

Start with a deliberately small catalog:

```text
flot
flet
feba
phase-line
line-of-departure
limit-of-advance
main-attack
supporting-attack
ambush
attack-by-fire
support-by-fire
breach-point
fortified-line
bridge-gap
sector-boundary-company
sector-boundary-battalion
sector-boundary-regiment
sector-boundary-brigade
sector-boundary-division
assembly-area
mined-area
```

## Styling And Affiliation

Default colors:

```ts
friendly: "#2d74c4"
hostile: "#c43a3a"
neutral: "#f5a623"
unknown: "#666666"
```

Consumers can override all styles.

Line weight default for BG Clasher should be `1`.

The package should not hard-code BLUFOR/OPFOR names. Use standard affiliation
terms internally and allow application-specific labels externally.

## Editing API

The core should expose semantic handles separately from rendered graphics.

```ts
export function getEditHandles(
  graphic: TacticalGraphic,
  context: RenderContext
): EditHandle[];

export function moveEditHandle(
  graphic: TacticalGraphic,
  handleId: string,
  newPosition: GeoPoint,
  context: RenderContext
): TacticalGraphic;
```

Example handle:

```ts
export interface EditHandle {
  id: string;
  label: string;
  position: GeoPoint;
  role: ControlPointDefinition["role"];
  draggable: boolean;
}
```

The adapter decides how handles look. The core decides what moving them means.

## Selection / Hit Testing

Do not require users to click a one-pixel line.

Every rendered graphic should be able to return hit primitives:

```ts
export function renderHitGeometry(
  graphic: TacticalGraphic,
  context: RenderContext
): RenderPrimitive[];
```

Rules:

- Lines should have a default invisible hit width around 20 to 28 pixels.
- Area graphics should be selectable by border and interior if desired.
- Hit geometry should not affect exported visible SVG unless requested.

## Serialization

Package serialization should be stable and boring:

```json
{
  "version": 1,
  "symbolId": "bridge-gap",
  "standard": "APP6",
  "affiliation": "friendly",
  "points": [
    { "lat": 60.1, "lon": 26.1 },
    { "lat": 60.2, "lon": 26.2 },
    { "lat": 60.15, "lon": 26.18 }
  ],
  "scale": 1,
  "modifiers": {
    "uniqueDesignation": "GAP 1"
  },
  "style": {
    "strokeWidth": 1
  }
}
```

Avoid storing derived geometry. Store only user intent and control points.

## Leaflet Adapter

Leaflet adapter responsibilities:

- Convert render primitives into `L.LayerGroup`.
- Attach visible layers.
- Attach invisible hit layers.
- Emit selection events.
- Render draggable edit handles.
- Translate Leaflet drag events into `moveEditHandle` calls.
- Redraw on zoom/pan.

Example:

```ts
const layer = renderLeafletGraphic(graphic, {
  map,
  interactive: true,
  selected: selectedId === graphic.id,
  onSelect: id => selectGraphic(id),
  onChange: next => updateGraphic(next),
});
```

## BG Clasher Integration Strategy

BG Clasher must remain a single self-contained HTML file.

Integration path:

1. Build the package externally.
2. Produce UMD browser bundle.
3. During BG Clasher development, load from local or CDN script tag.
4. Once stable, inline or vendor the UMD bundle if offline behavior requires it.
5. Replace BG Clasher tactical prototype renderers one by one with package calls.
6. Keep BG Clasher's OOB/unit APP-6 icon generation through `milsymbol`.

BG Clasher should store tactical graphics using the package serialization shape
inside its existing scenario JSON export.

## Testing Strategy

### Unit Tests

Test pure geometry:

- Polyline length.
- Point at distance.
- Tangent/normal.
- Offset line.
- Repeated module placement.
- Bridge/gap width constraint.
- Sector-boundary midpoint split.

### Golden SVG Tests

For each symbol:

- Render fixed input.
- Compare SVG output to committed fixture.
- Use stable formatting to reduce noisy diffs.

### Visual Regression Tests

Use Playwright or similar:

- Render examples page.
- Capture screenshots at several zoom levels.
- Verify marks remain map-scale.
- Verify selection hit layers work.
- Verify handles appear when selected.

### Property Tests Later

Useful for geometry robustness:

- Random polylines.
- No NaN coordinates.
- No negative module lengths.
- No disconnected fortified-line modules.
- No sector-boundary gap larger than allowed marker padding.

## Example Fixtures

Create a simple examples page:

```text
examples/leaflet.html
examples/svg-gallery.html
examples/editing.html
```

The gallery should show:

- Friendly and hostile variants.
- Scale 0.5, 1, 2.
- Horizontal, vertical, diagonal, and bent polylines.
- Selected and unselected state.

## Documentation

Minimum docs:

- Installation.
- Browser CDN usage.
- Node/ESM usage.
- Leaflet example.
- Data model.
- Supported symbols table.
- Editing handles.
- Serialization.
- How to add a new symbol.
- Conformance notes and limitations.

## Milestone Plan

### Milestone 0: Repo Scaffold

- TypeScript package.
- ESM/CJS/UMD outputs.
- Basic test runner.
- SVG primitive renderer.
- Minimal docs.

Acceptance:

- `npm test` passes.
- `npm run build` produces browser bundle and type declarations.

### Milestone 1: Geometry Core

- Local projection.
- Polyline measurement.
- Tangent/normal helpers.
- Repeated marks.
- Hit primitives.

Acceptance:

- Unit tests for all geometry helpers.
- No DOM or map-library dependency in core.

### Milestone 2: First Custom Line Symbols

- FLOT.
- FLET.
- Fortified line.
- Phase line.
- Sector boundary.
- Bridge/gap.

Acceptance:

- Each symbol renders to SVG primitives and GeoJSON-like primitives.
- Golden SVG fixtures committed.
- Editing handle definitions present.

### Milestone 3: Leaflet Adapter

- Render graphics on Leaflet.
- Selection hit layers.
- Drag handles.
- Live redraw while dragging.

Acceptance:

- Leaflet example supports draw, select, drag, serialize.
- Bridge/gap width handle remains constrained.
- Fortified line has equal connected modules.

### Milestone 4: Point Task Symbols

- Integrate optional `milsymbol`.
- Ambush.
- Attack by fire.
- Support by fire.
- Breach point.

Acceptance:

- Works with and without `milsymbol`.
- Fallback symbols render if `milsymbol` is unavailable.

### Milestone 5: BG Clasher Trial Integration

- Replace BG Clasher prototype renderers with package bundle.
- Keep scenario JSON backward-compatible through migration.

Acceptance:

- Existing BG Clasher tactical graphics load.
- New package graphics draw/edit correctly.
- Single-file BG Clasher delivery remains possible.

## Claude Code Build Brief

Give Claude Code this task:

```text
Build the initial version of an open source TypeScript package for map-based
military tactical graphics.

Use the design in tactical_graphics_package_design.md as the authority.

Start with a pure geometry/rendering core, not a full application. Do not attempt
complete MIL-STD-2525 coverage. Implement the scaffold, core data types, SVG
primitive renderer, geometry helpers, and the first six custom graphics:

- FLOT
- FLET
- fortified line
- phase line
- sector boundary
- bridge/gap

The core must not depend on Leaflet or browser globals. Add a small Leaflet
example only after the core and SVG rendering are tested.

Important behavior:

- Repeated tactical marks are measured in meters/map units and multiplied by a
  scale property.
- Fortified-line modules must be equal length and connected.
- Bridge/gap width handle must stay on the midpoint normal and must be labeled
  "Adjust width"; endpoints must preserve width when moved.
- Sector-boundary echelon markers must be centered and the line gap should be
  only large enough for the marker plus small padding.
- Render invisible hit geometry for easier selection.

Deliver:

- TypeScript source.
- Tests for geometry and handle constraints.
- SVG gallery example.
- Leaflet editing example if time permits.
- README with API examples.
```

## Open Questions

- Package name and npm scope.
- MIT or Apache-2.0 license. MIT aligns with `milsymbol`; Apache-2.0 aligns with
  Mission Command.
- Whether to support APP-6 and MIL-STD-2525 variants as explicit visual profiles
  from the beginning, or start with one "BG Clasher reference" profile.
- Whether SIDC should be primary for all graphics, or whether package-specific
  stable IDs should be primary with SIDC as metadata.
- How much conformance language to claim before a larger reference suite exists.

## Recommendation

Start with stable package-specific symbol IDs and store SIDC as metadata where
known. Claim "APP-6 / MIL-STD-2525 inspired tactical graphics with incremental
standard conformance", not full conformance, until the fixture library is much
larger.

Use MIT unless there is a strong reason to align with Apache-2.0. MIT makes the
package easy to embed in small browser tools and is compatible with the likely
BG Clasher use case.
