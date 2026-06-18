
- 


Good! 

- reduce the size of the larger of unit scale icons a bit more, maybe to 60% of status bar height
- move the map zoom buttons to below toolbar, with as many pixels separating the zoom buttons from the toolbar as are now separating toolbar from the layer name display. Also, scale up the map zoom buttons so that their width is the same as the toolbar width.
- change the name of Layers button to Map Layers
- reorder the map layers pane. The scenario layers should be grouped on top, the map layers grouped below them. Both should be collapsible. 
- Hex layers should belong to map layers.
- Raster background should be a layer named "Raster map" that can be rearranged and has both visibility checkbox and a opacity control, but remain selector-based as it is now (only one raster layer should be needed at a time).
- Instead of having one vector overlay layer, each of the individual sub-layers should be a layer that can be reordered and whose visibility can be controlled - this permits, for example, making road layers visible above hex layers so that it is easier to see where they are. If possible, the colour of each vector layer should be adjustable.
- Also add to the very bottom of the background map group a background color chooser.
- Wire "save scenario" so that layer ordering and details are saved, and add to background map group button "Restore scenario defaults".
- in scenario layer edit dialog boxes, add option to export the layer as GeoJSON; default file name suggested should be the [scenario_name]_turn_[number]_[layer_name].geojson (automatically safed by removing any problematic characters and replacing spaces with underscores, etc.). Add similar save capability to hex layers; the default name should be [scenario_name]_hex_grid_[layer_name].geojson.
- in both scenario layers and background map groups, add option to import GeoJSONs as new layers.
- change the hex layer default "measured as" choice to point-to-point.
- Investigate and fix the issue with hex layers that when adding a new hex layer while the map zoom is too large for the maximum number of hexes, the layer is added but hexes appear somewhere else. Maybe the best fix: if the map is zoomed too far out to fill the whole map with hexes, instead of warning that hex layer is truncated to 4000 hexes, just draw the 4000 hexes centered at the current map centerpoint.
- the hex layer details should show the shape and size of the layer (rows x columns, n hexes).
- add separate colour picker for hex fill colour.
- Change line and fill opacity controls from 0-1 range to 0%-100% range.
- The manual anchor latitude and longitude controls don't seem to work properly; hex grid doesn't move when value is changed. Also 4 decimal point accuracy should be enough.
- Likewise, pressing "Anchor = map center" button doesn't seem to work.
- change the name of "Anchor = map center" button to "Center hex grid to map center"
- the "coverage = current view" button produces odd results when the map zoom isn't just right; better remove it, as center hex grid to map center + manual adjustment should be enough.
- remove Log button from the top bar as there's one already at the status bar.
- in Plans pane for players (blufor and opfor), change the titles "Blufor plans" and "Opfor plans" to simply "Orders", likewise for Sync matrix
- in Umpire scenario builder General subpane, allow Markdown formatting in the scenario Description.
- move the OOB builder and Unit template editor functionalities in Umpire Scenario pane to Units pane. In the Umpire view, it should show sub-panes "Order of Battle" (under which are both blufor and opfor OOBs) and "Unit Templates."
- to save a bit of screen space, in player Units views, change the pane title from Units to Blufor units and Opfor units, and remove the Blufor/opfor subtitles 


Good job and good catches. Implement changes so that restoring scenario defaults is not destructive - if layers have been added they should remain. Note also that restore scenario defaults should only restore the background map defaults, including any hex grids created - not modify the scenario layers. 

Make the OOB import refresh the Units display, and add warnings if user attempts to import unsupported geometries (or other imports). The warnings should preferably detail what is wrong.

Remove Add unit button from the toolbar and assign its hotkey (U) to open Units pane.

Change the Draw polygon symbol in the toolbar from a hexagon to something more intuitively polygon-like, like irregular polygon.

Update readme, manuals, and AGENTS.md.