# Preliminary hazard analysis and design FMEA — revision 0.1

## 1. Method

This is an early design-driving analysis, not a certification risk assessment. Scores use:

- Severity **S**: 1 negligible, 2 minor, 3 serious, 4 life-threatening/fire.
- Occurrence **O**: 1 remote, 2 occasional, 3 probable, 4 frequent before controls.
- Detection **D**: 1 almost certain detection, 2 likely, 3 difficult, 4 unlikely before harm.

Any severity-4 item requires explicit design treatment. Scores must be revisited after parts, schematics, construction, and test data exist.

## 2. Hazard register

| ID | Hazard / initiating fault | Potential effect | S | O | D | Required controls | Verification evidence | Residual status |
|---|---|---|---:|---:|---:|---|---|---|
| H-001 | Contactor welded closed | Type 2 plug remains energized when charging should stop | 4 | 2 | 2 | 2-pole rated contactor; auxiliary feedback; output-voltage sensing; latched fault; prominent indication; no retry | Welded-contact fault injection and output-sense tests | Open |
| H-002 | Contactor closes without valid vehicle request | Exposed energized contacts, arcing, vehicle damage | 4 | 2 | 2 | Normally-off coil; hardware safety gate; valid C state plus diode; state-machine ownership; feedback timing | Reset/glitch/state-transition fault injection | Open |
| H-003 | ESP32 freezes with coil commanded on | Charging continues without supervision | 4 | 2 | 2 | External/hardware watchdog or safety supervisor; retriggered enable; RCM direct trip; fail-open driver | Freeze each safety task and CPU; verify dropout | Open |
| H-004 | Residual AC/DC current is not interrupted | Electric shock; upstream Type A RCD may be blinded by DC | 4 | 2 | 3 | Conformity-appropriate RCD/RDC-DD; direct hardware trip; startup/periodic test; fault latch | Calibrated residual-current injection and self-test failure | Open |
| H-005 | PE conductor open/high resistance | Exposed conductive parts/vehicle may become hazardous during another fault | 4 | 2 | 3 | Robust PE path; bonding; strain relief; earth-continuity production test; installation requirements; monitoring strategy subject to earthing system | PE resistance, pull/strain, single-fault assessment | Open |
| H-006 | L/N reversal, PEN presented to EVSE, or unsupported earthing topology | Protection/sensing assumptions invalid; touch-voltage risk | 4 | 2 | 3 | Support TN-S/TN-C-S only with separate L/N/PE at input; prohibit internal PEN split; installation instructions; switch L and N | Wiring fault matrix and installation review | Open |
| H-007 | CP circuit falsely classifies State C | Contactor energizes at wrong time | 4 | 2 | 2 | Protected CP front end; independent comparator/ADC plausibility; stable thresholds/timing; diode check; measured PWM feedback | CP boundary/noise/open/short/diode tests | Open |
| H-008 | Advertised current exceeds cable/site/product limit | Overheating, breaker operation, fire | 4 | 2 | 2 | Minimum-of-all-limits algorithm; PP validation; hard 32 A maximum; bounded config; PWM readback | Limit combinations and corrupt-config tests | Open |
| H-009 | Loose terminal or degraded contact | Local overheating/fire below overcurrent trip | 4 | 3 | 3 | Rated terminals; torque/process controls; temperature sensor placement; derating/shutdown; thermal design margin | Temperature-rise and loose-connection abnormal test | Open |
| H-010 | Cable/plug overheats or is mechanically damaged | Burn/fire/shock | 4 | 2 | 3 | Certified 32 A cable/plug; strain relief; temperature monitoring if available; inspection guidance; current derating | Cable pull/flex and temperature-rise tests | Open |
| H-011 | Contactor fails to close or chatters | Contact damage, overheating, unstable charging | 3 | 2 | 2 | Coil supply supervision; auxiliary and output feedback; bounded transition; minimum retry policy; undervoltage lockout | Low-coil-voltage and feedback fault tests | Open |
| H-012 | Output voltage sensor reports safe falsely | Welded contact not recognized | 4 | 2 | 3 | Auxiliary feedback diversity; input/output correlation; startup test; sensor diagnostics | Stuck-at/open/short injection | Open |
| H-013 | Auxiliary supply brownout/reset during charging | Undefined GPIO or contactor chatter | 4 | 2 | 2 | Supervisors; GPIO pull-down; fail-open gate; coil hold-up/dropout design; reset-cause logging | Supply ramp/dip/interruption tests | Open |
| H-014 | Temperature sensor fails low | Thermal protection unavailable | 4 | 2 | 3 | Open/short diagnostics; dual/location diversity where justified; rate and cross-sensor plausibility | Sensor substitution/fault injection | Open |
| H-015 | Conducted/radiated disturbance corrupts control | Unexpected switching, reset, incorrect readings | 4 | 2 | 3 | Filtering, isolation, layout, shielding, watchdogs, debounce, safe reset | ESD/EFT/surge/RF immunity pre-compliance | Open |
| H-016 | Surge damages driver into energized failure | Contactor energized or control isolation compromised | 4 | 2 | 3 | Coordinated SPD; coil suppression; isolation; failure-mode review; fuse/protection | Surge and destructive component analysis | Open |
| H-017 | Water/condensation reaches hazardous circuits | Shock, tracking, corrosion, fire | 4 | 2 | 3 | Define IP/environment; drainage/vent strategy; spacing/coating as justified; environmental tests | IP, humidity, condensation assessment | Open |
| H-018 | Wi-Fi attacker issues charge/config/update commands | Unauthorized use or unsafe current configuration | 3 | 3 | 2 | Authenticated local API; TLS; unique credentials; bounded limits; signed OTA; no direct actuator commands | Security review and penetration tests | Open |
| H-019 | Malformed/corrupted firmware or OTA | Loss of function or unsafe behavior | 4 | 2 | 2 | Secure boot/signatures; rollback; update only de-energized; watchdog; safe boot state | Interrupted/tampered OTA and rollback tests | Open |
| H-020 | Flash/configuration corruption | Wrong current, thresholds, or safety policy | 4 | 2 | 2 | CRC/version; strict bounds; redundant critical values where justified; no-charge defaults | Bit-flip/corrupt-partition tests | Open |
| H-021 | Meter/current sensor under-reads | Overcurrent/overload not recognized by software | 4 | 2 | 3 | Upstream hardware overcurrent protection; sensor plausibility/calibration; product maximum independent of reading | Sensor gain/stuck-at tests | Open |
| H-022 | User unplugs under load | Arcing/contact wear | 3 | 2 | 2 | Correct CP response and contactor-opening sequence; connector conformity; timing validation | Disconnect tests at representative current | Open |
| H-023 | State D accepted without ventilation | Charging in unsafe ventilation condition | 4 | 1 | 1 | Explicit state-machine prohibition and test | State D simulation | Open |
| H-024 | Safety fault automatically clears repeatedly | Re-energization into persistent hazard | 4 | 2 | 2 | Fault-class policy; latched severe faults; limited retries; first-fault log; local/service reset | Repeated-fault scenario tests | Open |
| H-025 | Incorrect service wiring/test procedure | Shock or equipment damage during development/service | 4 | 2 | 3 | Touch-safe fixture; upstream isolation/protection; discharge verification; procedures; qualified operators | Fixture and procedure review | Open |
| H-026 | Reduced air dielectric strength at 4000 m | Insulation breakdown, tracking, shock, or fire | 4 | 2 | 3 | Apply altitude correction to clearances; review component insulation ratings; dielectric test with justified equivalent conditions | Insulation-coordination calculation and dielectric verification | Open |
| H-027 | Reduced cooling at altitude or +55 °C ambient | Contactor, terminals, cable, PSU, or enclosure overheats | 4 | 3 | 3 | Worst-case thermal model; high-temperature components; sensor placement; current derating; full-load thermal test | +55 °C chamber and 4000 m-equivalent thermal verification | Open |
| H-028 | IP65 display/button/RFID interfaces leak | Water ingress causes corrosion, shock, false inputs, or fire | 4 | 2 | 3 | Qualified sealed interfaces; gasket compression control; drainage/vent strategy; assembled-product IP test | IP65, humidity, thermal-cycle, and condensation tests | Open |
| H-029 | Pedestal movement, impact, or cable pull | Enclosure damage, conductor strain, loss of PE continuity | 4 | 2 | 3 | Pedestal structural design; anchoring specification; cable strain relief; PE routing margin; impact/pull testing | Mounting, impact, pull, and earth-continuity tests | Open |

## 3. Mandatory independent safety paths

The schematic shall demonstrate these paths without relying on Wi-Fi or a healthy application task:

1. Residual-current fault → hardware gate disabled → contactor coil de-energized.
2. Watchdog/supervisor timeout → hardware gate disabled → contactor coil de-energized.
3. Brownout/auxiliary-power loss → contactor driver defaults off.
4. Application reset/bootloader/OTA state → contactor request defaults off.
5. Local severe overtemperature trip should have a hardware shutdown path if analysis shows firmware-only response is insufficient.

## 4. Safety validation priorities

The first prototype test fixture shall prioritize H-001 through H-009 and H-013. Vehicle testing is prohibited until those fault responses have objective evidence and an independent schematic/test review is complete.

## 5. Decisions blocking risk reduction

- PE-monitoring approach within the confirmed TN-S/TN-C-S scope.
- Indoor/outdoor environmental target and IP rating.
- Exact upstream Type A RCD/MCB ratings and the integrated RDC-DD topology.
- Contactor feedback architecture and selected contactor.
- Cable plug temperature sensing is unavailable; thermal safety must not depend on it.
- Installation fault current, overvoltage category, pollution degree, and 4000 m altitude correction.
