# Controller schematic specification — Rev A / Draft 0.1

> Safety-critical draft. This document is a connectivity specification for schematic capture, review, and bench validation. It is not approved for connection to mains or a vehicle.

## 1. Scope of this revision

Detailed sheet definitions are now maintained under `hardware/rev-a/schematic/`. The contactor itself and its coil-specific driver are postponed; J_K1 defines their interface.

This first schematic increment covers:

- ESP32-S3 controller core;
- 24 V, 5 V, and 3.3 V low-voltage power tree;
- BITUO BRCS01C-05-H1 residual-current sensor interface;
- independent contactor-coil inhibit path;
- external window watchdog;
- contactor command and auxiliary feedback;
- local reset, service USB, status LEDs, and button.

CP/PP analog circuitry, isolated mains sensing, energy metering, display, and RFID are reserved for the following schematic increments.

## 2. Safety architecture

The power contactor is normally open. Its 24 VDC coil may be energized only when every independent permission is true:

```text
COIL_ENABLE =
    MCU_CONTACTOR_REQUEST
    AND WATCHDOG_OK
    AND RDC_POWER_OK
    AND NOT RDC_DC_TRIP
    AND NOT FAULT_LATCHED
```

The `RDC_DC_TRIP` signal has a direct hardware path that clamps the contactor MOSFET gate low. ESP32 firmware is not required to open the contactor after an RDC fault.

The ESP32 observes all safety signals for diagnostics, but software cannot override an asserted hardware inhibit.

## 3. Power domains

| Net | Nominal | Source | Loads | Notes |
|---|---:|---|---|---|
| `+24V_ISO` | 24 VDC | certified isolated AC/DC PSU | contactor coil, 5.5 V preregulator | PSU selection pending 4000 m review |
| `+5V_PRE` | 5.4–5.6 V | buck from 24 V | RDC low-noise LDO, 3.3 V regulator | allows LDO regulation margin |
| `+5V_RDC` | 5.00 V ±3% | dedicated low-noise LDO | BRCS01C only | ripple target below 150 mV |
| `+3V3` | 3.3 V | regulator from 5V_PRE | ESP32-S3 and logic | supervised |
| `GND_SELV` | 0 V | isolated PSU secondary | controller reference | PE connection strategy remains open |
| `PE` | protective earth | installation | Type 2 PE and enclosure bonding | never switched; not signal ground by default |

Do not connect `GND_SELV` to PE until the CP isolation/reference architecture is frozen and reviewed.

## 4. Main devices

| Ref | Selected device | Function | Status |
|---|---|---|---|
| U1 | ESP32-S3-WROOM-1-N8R8 | application controller and Wi-Fi | selected |
| U2 | TPS3430-Q1 | independent window watchdog | selected for draft |
| U3 | BRCS01C-05-H1 | 6 mA DC / 30 mA AC+DC residual-current sensor | preferred, certification evidence pending |
| U4 | TPS54360B, configured for 5.5 V | low-voltage preregulator | provisional; compensation calculation pending |
| U5 | TLV76750 or qualified equivalent | dedicated 5 V RDC LDO | provisional |
| U6 | TPS62133, configured for 3.3 V | ESP32 supply | provisional; power calculation pending |
| U7 | logic/supervisor block | power-good and safety gating | implementation pending review |
| Q1 | DNP in controller-only increment | future contactor driver | postponed until coil selection |
| Q2 | small-signal NPN/N-MOSFET | direct RDC gate clamp | provisional topology |
| D1 | DNP in controller-only increment | future coil suppression | postponed until coil selection |
| K1 | Baomain BMHC7-40, 2NO, DC/AC 24 V coil | L/N switching | preferred prototype candidate; conditional |

## 5. ESP32-S3 core connections

### 5.1 Mandatory module connections

- All U1 3.3 V pins connect to `+3V3` with local 100 nF capacitors and bulk capacitance per the current Espressif module guidance.
- All grounds connect to a continuous `GND_SELV` plane.
- `EN` has the Espressif-recommended RC reset network and a service reset pushbutton.
- Boot-strapping pins receive explicit safe pull resistors; no safety actuator is assigned to a strapping pin.
- Native USB D+/D− route as a 90 Ω differential pair to an ESD-protected IP20 service connector.
- The RF antenna is placed at the PCB edge with the manufacturer keepout. The IP65 variant may use WROOM-1U and an external antenna.
- Contactor and safety outputs have external pull-down/off states independent of ESP32 reset.

### 5.2 Provisional GPIO allocation

| Signal | Direction | Provisional GPIO | Power-up state | Notes |
|---|---:|---:|---|---|
| `CONTACTOR_REQUEST` | out | GPIO4 | pulled low | request only; not final coil drive |
| `WATCHDOG_WDI` | out | GPIO5 | low/static | serviced only by safety supervisor task |
| `RDC_TEST_CMD` | out | GPIO6 | inactive | transistor/buffered into U3 TEST |
| `RDC_DC_TRIP_SENSE` | in | GPIO7 | external defined state | readback only |
| `RDC_ACDC_TRIP_SENSE` | in | GPIO8 | external defined state | diagnostic/additional shutdown candidate |
| `FAULT_LATCH_SENSE` | in | GPIO9 | defined | read hardware latch state |
| `CONTACTOR_AUX` | in | GPIO10 | defined | isolated/protected input |
| `COIL_CURRENT_OK` | in | GPIO11 | defined | optional comparator |
| `STATUS_LED_DATA` | out | GPIO12 | off | never shares safety pins |
| `LOCAL_BUTTON` | in | GPIO13 | pulled | protected/debounced |
| `I2C_SDA` | I/O | GPIO17 | pulled up | sensors/UI expansion |
| `I2C_SCL` | out | GPIO18 | pulled up | sensors/UI expansion |
| `UART_TX_SERVICE` | out | GPIO43 | idle | IP20 service |
| `UART_RX_SERVICE` | in | GPIO44 | pulled | IP20 service |

GPIO allocation remains provisional until CP/PP, metering, display, RFID, PSRAM constraints, and module strapping are checked together.

## 6. BRCS01C-05-H1 interface

### 6.1 Connector J3

Use a keyed, locking six-position connector. Pin numbers shall match the exact ordered cable assembly and manufacturer drawing before release.

| J3 signal | BRCS01C-05 function | Controller connection |
|---|---|---|
| `RDC_VCC` | VCC | `+5V_RDC` through ferrite bead and local decoupling |
| `RDC_ACDC_TRIP` | Type A 30 mA AC or 6 mA DC trip | protected hardware input and ESP32 sense |
| `RDC_GND` | GND | `GND_SELV` |
| `RDC_CAL` | calibration/manufacturing function | test pad or controlled circuit; exact use pending datasheet |
| `RDC_TEST` | self-test command | buffered output from `RDC_TEST_CMD` |
| `RDC_DC_TRIP` | 6 mA DC trip | direct gate clamp, fault latch, and ESP32 sense |

Manufacturer documentation indicates that the 5 V version requires 4.85–5.15 V and recommends filtering close to the sensor. Exact output drive, logic thresholds, self-test timing, and CAL behavior must be verified against the supplied production datasheet.

### 6.2 Primary conductor routing

- Pass both L and N through the sensing aperture in the same physical direction.
- Do not pass PE through the aperture.
- Keep L and N adjacent and mechanically secured so conductor movement does not alter performance.
- Do not use the sensor body to support conductor weight.
- Respect bend radius and insulation; verify the conductor bundle fits the 17 mm aperture at the selected conductor cross-section.
- Locate the sensor before the 2-pole contactor unless the manufacturer's certified application explicitly requires another position.

## 7. Direct RDC shutdown path

### 7.1 Required behavior

1. A valid `RDC_DC_TRIP` shall turn Q1 off without firmware action.
2. The event shall set a hardware fault latch.
3. The latch shall remain asserted after the residual current disappears.
4. ESP32 reset, Wi-Fi commands, or a stuck-high `CONTACTOR_REQUEST` shall not clear it.
5. Clearing is allowed only with the contactor confirmed open, vehicle-side voltage absent, and an explicit local/service reset sequence.
6. Loss of `+5V_RDC` or invalid RDC self-test shall prevent Q1 from turning on.

### 7.2 Draft transistor path

- Q1 is the low-side contactor-coil MOSFET; its gate has a permanent gate-source pull-down.
- The normal gate drive is supplied only through the safety-enable chain.
- Q2 is connected as a dominant gate clamp: when `RDC_DC_TRIP` asserts, Q2 pulls Q1 gate to source.
- Size the Q2 input network from the measured guaranteed U3 output levels, not typical values.
- Add a hardware latch driven by the conditioned trip signal. Do not rely on an ESP32 GPIO to preserve the fault.
- Provide test points for raw trip, conditioned trip, latched fault, watchdog output, Q1 gate, and coil return.

This topology remains DRAFT until the BRCS01C output polarity and guaranteed voltage/current limits are confirmed. No PCB shall be ordered from an assumed polarity.

## 8. Watchdog and power supervision

U2 TPS3430-Q1 runs from `+3V3` and monitors `WATCHDOG_WDI`.

- Configure a window that cannot be correctly serviced by a permanently high, permanently low, too-fast, or stalled GPIO.
- `WDO_N` directly removes the safety-enable path and also resets or interrupts U1.
- The firmware safety task alone owns WDI.
- Do not service WDI from a timer ISR that can continue while the application state machine is dead.
- Startup state is contactor inhibited until the watchdog, power rails, RDC self-test, CP self-test, and contactor-open feedback are all valid.

Provide independent power-good signals for `+24V_ISO`, `+5V_RDC`, and `+3V3`. The coil driver remains off while any mandatory rail is outside its validated range.

## 9. Contactor driver

> K1 interface remains conditional. Use the BMHC7-40 variant with 2NO main contacts and a DC/AC 24 V electronic coil. The seller states 2 W maximum coil power. Q1 may switch its 24 VDC supply only after coil polarity, startup current, turn-off transient, minimum pickup voltage, and the manufacturer's permitted suppression method are verified on the purchased unit. Do not assume that an external flyback diode is suitable for the internal AC/DC coil electronics.

### 9.1 Interface J4

| Pin | Net | Description |
|---:|---|---|
| 1 | `+24V_ISO` | coil positive |
| 2 | `COIL_LOW` | switched return through Q1 |
| 3 | `AUX_COM` | auxiliary-contact common |
| 4 | `AUX_NO` | auxiliary closed indication |

### 9.2 Design requirements

- Q1 VDS rating: at least 60 V, subject to transient calculation.
- Q1 continuous/pulsed current: at least twice the worst hot coil current with SOA margin.
- Gate defaults off with U1 absent, unpowered, booting, or reset.
- Coil suppression uses a diode/TVS or zener clamp selected to meet both Q1 avalanche margin and required contactor drop-out time.
- A plain flyback diode shall not be accepted until welded-contact detection and required opening time are shown to tolerate its slower release.
- Sense coil current or at minimum detect driver/open-coil plausibility.
- Auxiliary feedback is protected and conditioned; it cannot energize Q1.

## 10. RDC self-test sequence

Before every contactor closure:

1. Confirm K1 auxiliary reports open and vehicle-side voltage sensing reports de-energized.
2. Confirm `+5V_RDC` is valid.
3. Confirm neither trip output is active.
4. Assert `RDC_TEST_CMD` using the polarity and duration specified by BITUO.
5. Require the expected trip transition inside the specified test window.
6. Require the hardware inhibit/latch path to block Q1.
7. Deassert TEST and require correct recovery of the raw output.
8. Clear the test latch only through the controlled safe reset path.
9. If any transition is absent, late, stuck, or contradictory, latch `RDC_SELF_TEST_FAILED` and prohibit charging.

Production test shall additionally inject calibrated residual-current waveforms through a separate test conductor and verify the complete sensor-to-contactor response time.

## 11. Sheet hierarchy for KiCad

| Sheet | File | Content |
|---:|---|---|
| 00 | `evse-controller.kicad_sch` | hierarchy and external connectors |
| 01 | `mcu.kicad_sch` | ESP32-S3, USB, boot/reset, service |
| 02 | `lv-power.kicad_sch` | 24 V input, buck, 5 V RDC, 3.3 V, supervisors |
| 03 | `rdc-safety.kicad_sch` | BRCS01C connector, trip conditioning, latch |
| 04 | `contactor.kicad_sch` | safety AND, Q1 driver, suppression, auxiliary input |
| 05 | `ui-basic.kicad_sch` | LEDs and button |
| 06 | `cp-pp.kicad_sch` | reserved next increment |
| 07 | `mains-metering.kicad_sch` | reserved next increment |
| 08 | `display-rfid.kicad_sch` | reserved next increment |

## 12. Blocking data before electrical release

- BITUO quotation, controlled datasheet, exact variant suffix, connector/cable drawing, and certificate/test evidence for BRCS01C-05-H1.
- Guaranteed polarity and electrical limits of DC TRIP, AC+DC TRIP, TEST, and CAL.
- Controlled BMHC7-40 datasheet and exact 2NO / DC/AC 24 V order code: pickup and dropout voltage, startup and steady-state current, permitted suppression, opening time, terminal conductor range/torque, electrical endurance, certifications, and altitude derating.
- Confirm a manufacturer-approved auxiliary contact compatible with this exact BMHC7 body. Output-voltage sensing remains mandatory and is not treated as equivalent to a mechanically linked mirror contact.
- Certified isolated AC/DC PSU rated for −25…+55 °C and 4000 m.
- Insulation coordination inputs: overvoltage category, pollution degree, material group, and required impulse voltage.
- Decision on whether AC+DC TRIP is part of the hardwired shutdown OR-chain or diagnostics only.

## 13. K1 candidate decision — Baomain BMHC7-40

- Preferred prototype configuration: `BMHC7-40`, `2NO`, `DC/AC 24 V`. Do not order the 2NC or 1NO+1NC versions for switching L and N.
- Published seller data: `Ue 250 VAC`, `Ui 500 V`, `AC-7a 40 A`, `AC-7b 18 A`, maximum coil power 2 W, operating temperature −5…+60 °C, pollution degree 2, IP20 body, and up to 100 switching operations per day.
- This is a better controller match than the XKJL1 candidate because the selected coil accepts 24 VDC and has low holding power. XKJL1-40/2 is retained only as an unqualified alternate.
- Rating margin is limited: the EVSE continuous current is 32 A versus a 40 A AC-7a rating. Validate both poles at 32 A and 230 V in the real IP65 enclosure at +55 °C; monitor contactor terminals and automatically derate charging current before unsafe temperature rise.
- The listed −5 °C minimum does not meet the EVSE requirement of −25 °C. The prototype must not be released for cold operation unless pickup, dropout, contact resistance, and mechanical operation pass testing at −25 °C or a qualified alternative is selected.
- Published impulse withstand information is variant-dependent, and no 4000 m rating is provided. Complete insulation-coordination review and high-altitude derating before PCB or enclosure release.
- No built-in auxiliary/mirror contact is shown for the selected 2NO product. Investigate a documented compatible side auxiliary; retain isolated load-side voltage sensing for welded-contact detection.
- Product page: https://baomain.com/products/baomain-dc-ac-modular-contactor-hum-free-bmhc7-40a-2pole ambiguous seller data must be replaced by a controlled manufacturer datasheet before release.
