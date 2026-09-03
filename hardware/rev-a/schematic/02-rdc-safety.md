# Sheet 02 — RDC interface and hardwired safety

> BRCS01C-05-H1 output polarity and timing are not yet controlled. This sheet defines functions and configurable polarity points; it is not production-ready.

## Trip input channels

RDC_DC_RAW and RDC_ACDC_RAW use separate identical channels:

1. series resistance sized from guaranteed sensor output current;
2. clamp/ESD network compatible with the output type;
3. Schmitt comparator or buffer;
4. selectable inversion footprint;
5. separate hardwired shutdown and read-only ESP32 paths.

Do not wire the raw outputs together.

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

## Blocking evidence

- controlled BRCS01C-05-H1 datasheet;
- output topology, polarity and absolute maximum ratings;
- TEST waveform and timing;
- CAL function;
- startup and sensor-power-loss behavior;
- certificate scope for the exact suffix and harness.
