## MIL-STD-2525 reference symbols

The following files in `symbol_refs\` contain SVG files used as references for MIL-STD-2525 tactical graphics.

The unedited originals are backed up to `symbol_refs\originals`. Do not touch them.

## Orders

The following graphics should be used for unit orders.

tasks-block.svg
- Order name: Block
- Expected behaviour: cp1 marks the endpoint of the order polyline and can be used to move the symbol. cp2 connects to order polyline or to the unit centerpoint if only one point is given. Dragging cp3 or cp4 widen/narrow and rotate the symbol so that cp2 stays connected to the order polyline and the line between cp1 and cp2 stays perpendicular to the line cp3-cp4. Line cp1-cp2 stays the same length unless only one order point has been given, in which case this line is drawn from unit centerpoint to cp1. Letter "B" remains at the midpoint of the line cp1-cp2.


tasks-breach.svg
tasks-bypass.svg
tasks-canalize.svg
tasks-clear.svg
tasks-contain.svg
tasks-counterattack.svg
tasks-counterattack_by_fire.svg
tasks-cover.svg
tasks-delay.svg
tasks-destroy.svg
tasks-disrupt.svg
tasks-fix.svg
tasks-follow_and_assume.svg
tasks-follow_and_support.svg
tasks-guard.svg
tasks-interdict.svg
tasks-isolate.svg
tasks-neutralize.svg
tasks-occupy.svg
tasks-penetrate.svg
tasks-relief_in_place.svg
tasks-retain.svg
tasks-retirement.svg
tasks-screen.svg
tasks-secure.svg
tasks-seize.svg
tasks-withdraw.svg
tasks-withdraw_under_pressure.svg
offensive_lines-attack_by_fire_position.svg
offensive_lines-axis_of_advance_aviation.svg
offensive_lines-axis_of_advance_feint.svg
offensive_lines-infiltration_lane_whiskey.svg
offensive_lines-main_attack.svg
offensive_lines-supporting_attack.svg


## Tactical graphics
### Positions and objectives

base_deployments-assault_position_victor.svg
- Expected behaviour: a polygon with text "ASLT PSN" centered. Optional name, in this case "Victor".

base_deployments-assembly_area_X-ray.svg
base_deployments-attack_position_x.svg
base_deployments-objective_1.svg
base_deployments-penetration_box.svg

### Command points
command_points-checkpoint.svg
command_points-contact_point.svg
command_points-coordination_point.svg
command_points-decision_point.svg
command_points-passage_point.svg
command_points-point_of_departure.svg
command_points-rally_point.svg
command_points-release_point.svg
command_points-start_point.svg
command_points-waypoint.svg

### Defensive lines
defensive_lines-antitank_ditch_complete.svg
defensive_lines-fortified_area.svg
defensive_lines-fortified_line.svg
- Expected behaviour: polyline with square "sawtooth" markings

defensive_lines-foxhole_or_emplacement.svg
defensive_lines-strong_point_Bravo.svg
defensive_lines-surface_shelter.svg
defensive_lines-wire_obstacle.svg

### Coordination and boundaries

lines_of_coordination-phase_line_axe.svg
- Expected behaviour: polyline; letters PL [name] (e.g. AXE in this example) equidistant at the both ends of the line

lines_of_coordination-line_of_departure_knife.svg
- Expected behaviour: polyline; letters LD (PL [name]) equidistant at the both ends of the line, (PL [name]) under letters LD

lines_of_coordination_limit_of_advance_hammer.svg
- Expected behaviour: polyline; letters LOA (PL [name]) equidistant at the both ends of the line,  (PL [name]) under letters LOA

lines_of_coordination-forward_line_of_own_troops_(FLOT).svg
- Expected behaviour: polyline composed of half circles or polygons with enough sides; letters "FLOT" at the both ends of the line, equidistant from line ends

lines_of_coordination-forward_line_of_enemy_troops_(FLET).svg
- Expected behaviour: polyline composed of two lines of half circles or polygons with enough sides; letters "FLET" at the both ends of the line, equidistant from line ends

lines_of_coordination-forward_edge_of_battle_area_(FEBA).svg
- Expected behaviour: polyline composed of symbols like in the file (a circle with x shape inside them)


sector_boundaries-army.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-army_group.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-battalion.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-brigade.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-company.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-corps.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-division.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-platoon.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-regiment.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-section.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-squad.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line

sector_boundaries-team.svg
- Expected behaviour: polyline; echelon symbol at the lengthwise center point of the line
### Passages
passages-bridge_or_gap.svg
- Expected behaviour: line drawn from cp1 to cp2
- Control points: dragging cp1 or cp2 moves line endpoints, cp3 controls width of the symbol

passages-ferry.svg
passages-ford_difficult.svg
passages-ford_easy.svg
passages-lane.svg


single_mine-unspecified_mine.svg
single_mine-wide_area_mine.svg
single_mine-antipersonnel_mine.svg
single_mine-antitank_mine.svg
single_mine-antitank_mine_directional.svg
single_mine-booby_trap.svg
single_mine-tripwire.svg
minefield-antipersonnel_mines.svg
minefield-antitank_mines.svg
minefield-antitank_mines_directional.svg
minefield-dummy_mined_area.svg
minefield-dummy_minefield.svg
minefield-mined_area.svg
minefield-unspecified_mines.svg
minefield-wide_area_mines.svg
obstacles-antitank_ditch_under_construction.svg
obstacles-concertina_single.svg
obstacles-roadblocks_craters_and_blown_bridges_complete.svg
obstacles-roadblocks_craters_and_blown_bridges_explosives_state_of_readiness_2_armed_but_passable.svg
obstacles-roadblocks_craters_and_blown_bridges_planned.svg
area_symbols-biologically_contaminated_area.svg
area_symbols-chemically_contaminated_area.svg
area_symbols-radioactive_area.svg
area_symbols-smoke.svg

