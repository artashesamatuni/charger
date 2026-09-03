# Sheet 02 — RDC interface and hardwired safety

> This revision uses the public BRCS01C datasheet V1.0.1 behavior for the 5 V version. Production release still requires the controlled datasheet supplied with the purchased BRCS01C-05-H1 lot.

## Confirmed 5 V interface behavior

- VCC is 5 VDC ±3%.
- Pin 2 is AC&DC Trip.
- Pin 4 is CAL.
- Pin 5 is TEST.
- Pin 6 is DC Trip.
- In the normal state the trip outputs pull to GND.
- On the corresponding trip the output changes to high impedance.
- A change from GND to high impedance is also permitted by the product behavior; the controller must treat it as a fault.
- Direct connection of pins 2 or 6 to a relay/contactor is prohibited by the application note.

## Fail-safe trip input channels

RDC_DC_RAW and RDC_ACDC_RAW use separate identical channels:

1. 10 kΩ preliminary pull-up to supervised +3V3, so trip, missing sensor power, or an open signal wire becomes logic high;
2. 1 kΩ preliminary series resistor into a 5 V-tolerant Schmitt buffer/comparator;
3. optional small filter capacitor footprint, DNP until response-time testing;
4. split conditioned output into the hardwired shutdown path and an ESP32 read-only input.

Select the final buffer only after its thresholds are checked against the guaranteed BRCS low-output voltage. Do not wire the two raw outputs together. The hardwired path must not depend on ESP32 execution.

## Set-dominant fault latch

```text
SET = RDC_DC_TRIP
   OR RDC_ACDC_TRIP
   OR WATCHDOG_FAULT
   OR RDC_POWER_BAD
   OR CP_HARD_FAULT

RESET = LOCAL_SAFE_RESET
    AND K1_OPEN_CONFIRMED
    AND NOT LOAD_V_PRESENT
```

Requirements:

- SET dominates RESET.
- Power-up defaults to faulted/inhibited.
- ESP32 cannot directly force RESET.
- Loss of logic power removes contactor permission.
- Provide test points on raw trips, conditioned trips, SET, RESET, Q and /Q.

## Hardware contactor permission

```text
K1_HW_ENABLE =
    CONTACTOR_REQUEST
    AND WATCHDOG_OK
    AND RDC_5V_OK
    AND NOT RDC_DC_TRIP
    AND NOT RDC_ACDC_TRIP
    AND NOT FAULT_LATCHED
```

The final stage must default OFF while unpowered. K1_HW_ENABLE terminates at J_K1; the future coil driver cannot bypass it.

## Watchdog

U_WD is provisionally TPS3430-Q1.

- Use window mode.
- WDO directly inhibits K1 and sets the latch.
- Only the firmware safety task owns WDI.
- A free-running interrupt must not service WDI.
- Timing components remain DNP until firmware scheduling is frozen.

## TEST and CAL drivers-TEST:** drive pin 5 high for 40–100 ms in the first prototype; the public datasheet specifies activation above 40 ms. Use an open-drain or series-buffered driver whose reset state is low.
- **CAL:** pull pin 4 to GND for 50–100 ms using an open-drain transistor. Leave it high impedance otherwise.
- Calibration is allowed only with the contactor open and no current through the sensing aperture.

## RDC self-test

Before every closure request:

1. verify open contactor feedback and no load-side voltage;
2. verify +5V_RDC and inactive raw trips;
3. stimulate TEST for the specified duration;
4. require raw and conditioned transitions;
5. verify K1 permission disappears and the latch sets;
6. release TEST and verify recovery;
7. perform the controlled safe reset;
8. prohibit charging on any missing, late, stuck, or contradictory response.

Production test must inject calibrated AC and smooth-DC residual currents through the aperture and measure total response time.

## Source and remaining blocking evidence

Public sources:

- https://www.bituo-technik.com/news/brcs01c-05-h1-facilitate-designing-a-reliable-built-in-dc-6ma-rcd-at-ac-ev-chargers/
- https://bituo-technik.com/wp-content/uploads/2024/01/TS_BRCS01C_EN_V1.0.1_202312.pdf

Still required before release:

- controlled datasheet and connector drawing supplied for the purchased BRCS01C-05-H1 lot;
- guaranteed output low voltage and sink current;
- exact self-test response-time limits and startup settling time;
- certificate and test-report scope for the exact suffix and harness;
- validation that signal-wire disconnection, loss of 5 V, stuck-low, and stuck-high faults all prohibit charging.
