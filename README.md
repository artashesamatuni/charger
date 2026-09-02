# Single-phase AC EVSE prototype

Engineering baseline for a European Mode 3 EV supply equipment prototype:

- 230 V AC, single phase
- 32 A maximum (7.4 kW nominal), configurable 6–32 A
- permanently attached cable with Type 2 vehicle connector (Case C)
- ESP32 controller with Wi-Fi
- IEC 61851 control pilot

Engineering documents:

- [System baseline](docs/system-baseline.md): external I/O, safety concept, self-tests, charging state machine, hardware architecture, and firmware partitioning.
- [System requirements](docs/system-requirements.md): traceable, verifiable product requirements.
- [Preliminary hazard analysis](docs/hazard-analysis.md): design FMEA and required independent safety paths.
- [Architecture and interface specification](docs/architecture-and-interfaces.md): frozen product options, subsystem partitioning, communications, UI, and schematic-sheet plan.
- [Thermal derating algorithm](docs/thermal-derating.md): sensor validation, current limiting, shutdown/recovery behavior, and environmental test matrix.

> This repository is an engineering prototype, not certified equipment. Mains testing requires appropriate competence, isolation, protection, enclosure, and test equipment. Do not connect a vehicle or public supply until the safety circuits and fault responses have been independently reviewed and tested.
