# PCB Rev A — placement and routing requirements

## Scope

This PCB contains only the low-voltage EVSE controller. Mains L/N and charging current never enter the PCB. The external assembly contains the breaker, Type A RCD, BRCS aperture conductors, contactor and high-current wiring.

Board target: 160 mm x 90 mm, 2 layers, 1.6 mm FR-4, 1 oz copper for the first controller prototype.

## Zones

| Zone | X range | Purpose |
|---|---:|---|
| A | 5–45 mm | CP/PP and PE-referenced analog interface |
| B | 50–85 mm | BRCS connector and hardwired safety logic |
| C | 90–120 mm | 24 V input and DC/DC power |
| D | 120–160 mm | ESP32-S3, UI and service interfaces |

Maintain an un-routed isolation corridor between the CP/PE-referenced domain and GND_SELV until the grounding architecture is frozen. No copper pour crosses this corridor.

## Placement rules

- ESP32-S3 antenna overhangs or faces the right board edge.
- No copper, vias, display, RFID antenna, cable or enclosure metal in the module antenna keepout.
- J_RDC is near the BRCS cable entry and away from contactor coils and high-current conductors.
- RDC pull-ups, buffers and test points are immediately behind J_RDC.
- Hardware trip path is short and does not pass through the ESP32 area.
- Watchdog and safety latch remain physically separate from UI/Wi-Fi circuits.
- CP TVS and source resistor are adjacent to J_CP_PP.
- Input fuse, reverse-polarity protection and 24 V TVS are adjacent to J_PWR.
- Test points remain accessible after enclosure assembly.
- Four M3 mounting holes have copper and component keepouts.

## Routing rules

- Default signal width: 0.25 mm.
- 24 V and 5 V distribution: at least 0.50 mm, increased after current calculation.
- RDC 5 V supply uses a quiet local route and dedicated return to the LDO.
- CP bipolar output uses a short route with no parallel run beside ESP32 clocks or antenna.
- Hardware trip and watchdog-inhibit nets do not share connectors or routing with LEDs, display or RFID.
- Use continuous GND_SELV plane only inside the SELV region.
- PE-referenced CP region uses its own reference copper until the isolation decision is validated.
- Do not add a GND_SELV-to-PE net tie by convenience.

## Release blockers

- all schematic functional blocks replaced with exact devices and pin numbers;
- controlled BRCS01C-05-H1 connector drawing;
- CP circuit calculation and IEC 61851 threshold review;
- insulation coordination for 4000 m;
- ERC clean with documented exceptions;
- PCB DRC clean;
- antenna and enclosure review;
- thermal review at +55 °C;
- schematic/PCB netlist comparison;
- rendered Gerber inspection.

The draft PCB is for mechanical planning only and must not be fabricated yet.
