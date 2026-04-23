# Opening Summary:
PCB available area: 19mm on top and bottom edges, 30mm from top to bottom, sides are populated with 2x10 female, 2.54mm pitch 8.5mm tall header pins for board stacking. therefore, the right and left sides of the board are not available for utilization of these connectors. there is a board stacked above this board, therefore vertical space is limited to approximately 7.5mm
total board working dimensions: 19.85x30x7.5mm

## needs:
### connector 1 (inputs):
* up to 6A +12V input power (preferably with a safety margin [^1]) [^2]
* CAN_H/CAN_L input signals
* 5 12-16V low speed vehicle input signals (dome, L/R turn, brakes, reverse)

### connector 2 (outputs):
* up to 6A of +12V output power (preferably with safety margin [^1]) [^2]
* 6 LED 5V data+GND signal pairs

[^1]: by "safety margin", I mean that given the choice between 2 connectors meeting all of the requirements, if 1 of them supports exactly 6A of current, and the other can support more than the required 6A (ex. 8A per pin or 2pins @ 3.5A per pin instead of just 2pins@3A) prefer the connector that can support the higher current (as a "safety margin")
[^2]: if a connector cannot be found that can handle 6 or more Amps per pin, then Pin-Paralleling is allowed

### connector 3:
* Relay +/-
* these signals may be incorporated into connector 1 or connector 2 if there are enough unused circuits to properly isolate it.  if done the positions of connector 1 and connector 2 will be swapped due to required location of "dirty/relay ground" point

## requirements:
* all connectors should be SMD, not THT
* connector 1 and 2 must be the same MPN, even if there are empty/unused pins
* connector 3 should be in the same (or similar) connector series to maintain a level of visual and professional consistency
* all connectors should be side-entry (horizontal)
* connector 1 will be placed along bottom edge (overhang is allowed)
* connector 2 will be placed along top edge (overhang allowed)
* connector 3 should also be on or near the top edge. It is not required to be ON the edge unless space allows. do not forget that there is another board stacked on top of this board, so Connector 3's wires will need an opening between Conn2 and the female headerpins, or enough clearance between the top of Conn2 and the 2nd board
* typical mfg's are JST, Molex, TE, Samtec, Hirose, etc. although your search should not be limited to these. they are simply the mfg's that came to mind
* DO NOT ASSUME OR GUESS AT PART NUMBERS OR DIMENSIONS. CONFIRM WITH DATASHEETS.
* When determining connector's height, do not forget to account for the height of the locking ramp and locking clip unless these are only on the header where it overhangs the board edge.

## other info:
* PCB is 6 layer
  - L1: as noted in [opening summary](#opening-summary) with +12V trace routing and minimal signal routing
  - L2: ground plane
  - L3: signal routing
  - L4: ground plane
  - L5: ground plane
  - L6: main layer for components
* project installation is inside GR86 cabin, behind glovebox
