# System requirements — revision 0.1

## 1. Conventions

- **Shall** indicates a mandatory, verifiable requirement.
- Verification methods: **I** inspection, **A** analysis, **T** test, **D** demonstration.
- Status **Allocated** means the requirement has an identified implementation owner but is not yet verified.
- Normative values and timings marked **TBD-N** must be taken from licensed standards before design release.

## 2. Product and rating requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| PR-001 | The EVSE shall provide IEC 61851 Mode 3 AC charging through a permanently attached cable with a Type 2 vehicle connector. | System | I/T | Allocated |
| PR-002 | The EVSE shall operate from 230 V AC nominal, 50 Hz, single-phase L/N/PE supply. | Power | T | Allocated |
| PR-003 | The EVSE shall have a rated continuous output current of 32 A and nominal power of 7.4 kW. | Power/thermal | A/T | Allocated |
| PR-004 | The configured charging-current range shall be 6–32 A. | Firmware/CP | T | Allocated |
| PR-005 | The product shall default to output de-energized on startup, reset, brownout, firmware failure, or loss of auxiliary power. | Safety hardware | A/T | Allocated |
| PR-006 | Wi-Fi availability shall not be required to execute a safety shutdown. | Architecture | I/T | Allocated |
| PR-007 | The common electronics shall support an IP20 development configuration and an IP65 outdoor-target configuration. | Mechanical/system | I/T | Allocated |
| PR-008 | The EVSE shall support TN-S and TN-C-S installations in which separate L, N, and PE conductors are presented at the EVSE input. | Installation/electrical | I/T | Allocated |
| PR-009 | The EVSE shall not accept or perform a PEN split; TN-C-S PEN separation shall occur upstream in accordance with applicable installation rules. | Installation/electrical | I | Allocated |
| PR-010 | The EVSE shall operate over an ambient-temperature range of −25 °C to +55 °C. | System/thermal | T | Allocated |
| PR-011 | The EVSE shall be designed for installation altitude up to 4000 m. | Electrical/thermal | A/T | Allocated |
| PR-012 | The IP65 configuration shall support wall and pedestal mounting without compromising ingress protection, cooling, cable strain relief, or user access. | Mechanical | I/T | Allocated |

## 3. Power-path requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| PW-001 | Both L and N supplied to the vehicle shall be opened by the power contactor. | Contactor | I/T | Allocated |
| PW-002 | PE shall not be switched and shall remain bonded from input to cable PE. | Wiring/enclosure | I/T | Allocated |
| PW-003 | All conductors, terminals, cable assemblies, contacts, and protection devices in the charging path shall be rated for at least 32 A continuous duty under worst-case enclosure temperature and installation conditions. | Electrical/thermal | I/A/T | Allocated |
| PW-004 | The contactor shall be rated for the applicable utilization category, prospective fault current, switching endurance, coil voltage, and operating temperature. | Electrical | I/A | Open |
| PW-005 | The design shall independently sense input supply presence and vehicle-side output voltage with suitable isolation. | Sensing | I/T | Allocated |
| PW-006 | The design shall obtain contactor mechanical state through a positively linked/appropriate auxiliary contact or an equivalently justified diagnostic arrangement. | Contactor | I/A/T | Open |
| PW-007 | Unexpected voltage on the vehicle output while the contactor is commanded open shall inhibit charging and latch a welded/stuck-contactor fault. | Safety/firmware | T | Allocated |
| PW-008 | Failure to observe contactor closure within the specified transition time shall cause the coil command to be removed and a fault recorded. | Safety/firmware | T | Allocated |
| PW-009 | Surge, overcurrent, auxiliary-circuit fusing, and EMC protection shall be coordinated with the installation and component ratings. | Power entry | I/A/T | Open |
| PW-010 | The installation shall provide the coordinated branch MCB and Type A RCD in an external distribution board. | Installation | I | Allocated |
| PW-011 | Product installation instructions shall state the required upstream MCB, Type A RCD, SPD/overvoltage assumptions, conductor sizes, and prospective fault-current limits. | Installation/electrical | I/A | Open |
| PW-012 | The external C32 circuit breaker and Type A 40 A / 30 mA RCD shall be excluded from the EVSE schematic and BOM; only the L/N/PE boundary and installation requirement shall be documented. | System/electrical | I | Allocated |

## 4. Control-pilot and proximity requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| CP-001 | The EVSE shall generate the IEC 61851 control pilot with nominal ±12 V levels and 1 kHz PWM where applicable. | CP hardware | T | Allocated |
| CP-002 | CP voltage, frequency, duty cycle, source impedance, rise/fall characteristics, tolerances, and transition timing shall meet the controlled edition of IEC 61851-1. | CP hardware/firmware | T | TBD-N |
| CP-003 | The EVSE shall distinguish stable CP states A, B, C, D, E, and F using thresholds, debounce, and timing compliant with IEC 61851-1. | CP firmware | T | TBD-N |
| CP-004 | The EVSE shall verify the vehicle diode signature before energizing the power contactor. | CP hardware/firmware | T | Allocated |
| CP-005 | State D shall not energize the contactor because forced ventilation is not supported. | State machine | T | Allocated |
| CP-006 | The EVSE shall measure PP and shall never advertise more current than the cable assembly permits. | PP/limit manager | T | Allocated |
| CP-007 | An open, shorted, implausible, or uncalibrated PP measurement shall inhibit charging. | PP/limit manager | T | Allocated |
| CP-008 | For 6–51 A advertisement, the firmware shall encode current using the applicable IEC 61851 PWM relationship and shall verify the generated duty cycle by measurement. | CP firmware/hardware | T | Allocated |
| CP-009 | When the allowable current is below 6 A, the EVSE shall suspend charging using standards-compliant pilot behavior and shall not advertise a sub-6 A current. | State machine | T | Allocated |
| CP-010 | CP loss, short, invalid plateau, invalid diode result, or commanded/measured PWM disagreement shall prevent or terminate power delivery. | Safety/CP | T | Allocated |

## 5. Electrical-protection requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| SF-001 | Residual-current protection shall be implemented using a conformity-appropriate arrangement for EU Mode 3 EVSE. | Protection architecture | I/A/T | Open |
| SF-002 | If Type A upstream RCD coordination is selected, the EVSE shall use an IEC 62955-compliant RDC-DD providing the required 6 mA DC detection function. | Certified module | I/T | Open |
| SF-003 | A residual-current trip shall remove contactor coil energy through a safety path that does not depend solely on the ESP32 application. | Safety hardware | I/T | Allocated |
| SF-004 | The RDC-DD/RCM test function shall be exercised at startup and at the interval required by the chosen device and applicable standard. | Safety hardware/firmware | I/T | Open |
| SF-005 | A failed residual-current self-test shall inhibit charging and latch a fault. | Safety supervisor | T | Allocated |
| SF-006 | Isolation, creepage, clearance, insulation, and protective bonding shall be calculated for the final overvoltage category, pollution degree, material group, altitude, and enclosure. | PCB/mechanical | I/A/T | Open |
| SF-007 | Accessible SELV and network circuitry shall have the required isolation from mains, contactor-coil transients, and CP fault conditions. | Electrical/PCB | I/T | Allocated |
| SF-008 | The enclosure shall prevent access to hazardous live parts in normal use and foreseeable single-fault conditions. | Mechanical | I/T | Open |
| SF-009 | Safety-critical protective functions shall be evaluated for single faults and common-cause failures before schematic release. | Safety analysis | A | Allocated |
| SF-010 | The EVSE shall incorporate an IEC 62955-conformity-appropriate RDC-DD function and expose its fault and test results to the safety supervisor. | Certified RDC-DD/safety | I/T | Allocated |
| SF-011 | The RDC-DD fault output shall participate in the independent hardware contactor-disable chain. | Safety hardware | I/T | Allocated |
| SF-012 | The IP65 configuration shall maintain its ingress-protection target with the cable, glands, vents, display/RFID windows, buttons, and service closures installed. | Mechanical | I/T | Open |
| SF-013 | Creepage, clearance, insulation coordination, component voltage ratings, and dielectric verification shall account for operation at 4000 m, including the applicable altitude correction above 2000 m. | Electrical/PCB | A/I/T | Open |

## 6. Thermal and measurement requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| TM-001 | Temperature shall be measured at the highest-risk power termination/contactor region and on the control PCB. | Thermal/electrical | I/T | Allocated |
| TM-002 | Temperature sensors shall be diagnosed for open circuit, short circuit, stuck reading, and implausible rate of change where technically feasible. | Firmware | T | Allocated |
| TM-003 | The EVSE shall progressively derate advertised current before reaching the shutdown threshold. | Limit manager | T | Allocated |
| TM-004 | Exceeding the shutdown threshold or losing a required temperature sensor shall stop charging. | Safety supervisor | T | Allocated |
| TM-005 | Derating, shutdown, and restart thresholds shall be derived from measured component and enclosure temperature rise with margin. | Thermal | A/T | Open |
| TM-006 | The product shall apply a validated automatic current-derating curve whenever thermal conditions do not permit 32 A continuous operation. | Thermal/system | A/T | Allocated |
| TM-007 | Thermal derating shall only maintain or reduce the currently permissible advertised current; it shall never override a lower product, cable, site, user, or backend limit. | Limit manager | T | Allocated |
| TM-008 | If the thermally permissible current falls below 6 A, the EVSE shall suspend charging using standards-compliant pilot behavior and open the contactor. | State machine | T | Allocated |
| TM-009 | Thermal current increases shall be rate limited and require a stable cooldown interval; safety-related reductions and shutdowns shall not be delayed by that rate limit. | Thermal/limit manager | T | Allocated |
| TM-010 | A missing, stale, open, shorted, or implausible mandatory thermal sensor shall prevent charging or apply a separately justified fail-safe limit. | Safety supervisor | T | Allocated |
| TM-011 | Thermal shutdown shall latch for the charging session and shall not automatically resume until all relevant sensors remain below validated restart thresholds for a defined cooldown time. | State machine | T | Allocated |
| ME-001 | Voltage, current, power, and cumulative energy shall be measured for diagnostics and user display. | Metering | T | Allocated |
| ME-002 | Metering accuracy shall not be represented as billing-grade unless the complete measurement and conformity pathway supports that claim. | Product | I | Allocated |
| ME-003 | Energy metering in revision 0.1 shall be intended for indication, diagnostics, and statistics only; MID/billing conformity is out of scope. | Product/metering | I/T | Allocated |

## 7. Control and software requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| SW-001 | Only the charging state machine may request contactor closure; only the safety supervisor may provide the final hardware enable. | Architecture | I/T | Allocated |
| SW-002 | The advertised current shall be the minimum of product, PP cable, site, thermal, load-management, and user/backend limits. | Limit manager | T | Allocated |
| SW-003 | Every actuator command shall be range-checked and validated against current state and safety conditions. | Firmware | I/T | Allocated |
| SW-004 | Task and hardware watchdogs shall force the output to a de-energized condition when safety-related execution deadlines are missed. | Firmware/hardware | T | Allocated |
| SW-005 | Safety inputs shall include freshness, range, and plausibility status; stale or invalid mandatory inputs shall inhibit charging. | Firmware | T | Allocated |
| SW-006 | Fault processing shall preserve the first fault cause and a snapshot of safety-relevant measurements and state. | Diagnostics | T | Allocated |
| SW-007 | Safety-related configuration shall be versioned, bounded, integrity checked, and replaced with a no-charge or conservative state when invalid. | Configuration | T | Allocated |
| SW-008 | No network command shall directly control the contactor GPIO or bypass the state machine. | Architecture | I/T | Allocated |
| SW-009 | The device shall continue safe local operation during Wi-Fi loss according to the configured authorization policy. | Firmware | T | Allocated |
| SW-010 | Firmware updates shall only begin with the contactor open and charging unavailable. | Update manager | T | Allocated |
| SW-011 | OTA shall use authenticated images, an A/B or equivalent recovery design, and automatic rollback after failed boot validation. | Bootloader/update | I/T | Allocated |
| SW-012 | Production configuration shall consider secure boot, flash encryption, credential protection, disabled debug paths, and TLS identity provisioning. | Security | I/A/T | Open |
| SW-013 | The controller shall use an ESP32-S3 module and ESP-IDF. | Controller | I | Allocated |
| SW-014 | The EVSE shall provide a protected Wi-Fi SoftAP commissioning mode with a local web configuration interface. | Network/UI | D/T | Allocated |
| SW-015 | The EVSE shall operate as a Wi-Fi station after provisioning and shall provide the local web interface on the configured LAN. | Network/UI | D/T | Allocated |
| SW-016 | Loss of station connectivity shall not automatically erase credentials or continuously expose an unsecured commissioning AP. | Network/security | T | Allocated |
| SW-017 | The device shall synchronize civil time through configurable NTP servers, while all safety deadlines and sequencing use monotonic timers independent of NTP. | Time service | T | Allocated |
| SW-018 | MQTT integration shall support telemetry, availability, commands, and configuration through a versioned topic and payload schema. | MQTT | D/T | Allocated |
| SW-019 | MQTT commands shall be authenticated by the transport/configuration policy, range checked, acknowledged, and passed through the normal state machine. | MQTT/security | T | Allocated |
| SW-020 | MQTT over TLS shall be the default for non-local or untrusted networks; plaintext MQTT shall require an explicit local-development setting. | MQTT/security | I/T | Allocated |

## 8. User and service requirements

| ID | Requirement | Allocation | Verify | Status |
|---|---|---|---|---|
| UI-001 | The EVSE shall visibly distinguish unavailable, available, vehicle connected, charging, suspended, and faulted states. | UI | D/T | Allocated |
| UI-002 | A latched safety fault shall not be remotely cleared unless the specific hazard analysis explicitly permits it. | UI/network | T | Allocated |
| UI-003 | A welded-contactor or residual-current fault shall require the defined local/service recovery procedure. | Service | D/T | Allocated |
| UI-004 | Service diagnostics shall expose state, first-fault code, measurements, firmware/hardware versions, reset cause, and self-test results without exposing credentials. | Diagnostics | D/T | Allocated |
| UI-005 | The base user interface shall provide status LEDs and one local button. | UI hardware | D/T | Allocated |
| UI-006 | Button actions shall be time-qualified and state dependent; no button action may directly energize the contactor. | UI/firmware | T | Allocated |
| UI-007 | The architecture shall provide optional isolated/SELV interfaces for a touch display and RFID reader without changing safety authority. | UI hardware | I/T | Allocated |
| UI-008 | Phone configuration and control shall initially be available through a responsive local web interface; packaging it as a PWA or native app is a later product decision. | Web UI | D | Allocated |
| UI-009 | The IP65 configuration shall include status LEDs, a local button, RFID, and a touch display. | UI/mechanical | I/D/T | Allocated |
| UI-010 | The display, RFID window/antenna, and button integration shall preserve the assembled IP65 rating and required impact/environmental performance. | UI/mechanical | I/T | Open |
| UI-011 | The design shall not depend on a temperature sensor embedded in the attached Type 2 plug. | System | I | Allocated |

## 9. Verification gates

| Gate | Exit criteria |
|---|---|
| G0 Architecture | Product options frozen; requirements and hazard analysis reviewed |
| G1 Low-voltage prototype | CP/PP simulator passes all states/faults; no mains connected |
| G2 Power prototype | protected fixture verifies switching, sensing, RCM, watchdog, and thermal faults |
| G3 Vehicle test | independent safety review complete; controlled vehicle charging at limited then full current |
| G4 Pre-compliance | electrical safety, EMC, thermal, environmental, and abnormal-operation tests completed |
