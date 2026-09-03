# Sheet 00 — System interconnect (Rev A, Draft 0.2)

> Controller-only design. The power contactor is external and represented by connector J_K1. Do not connect this draft to mains or a vehicle.

## Functional partition

```text
24 V isolated input
  -> input protection
  -> 5.5 V buck -> 5 V RDC LDO -> BRCS01C
                 -> 3.3 V regulator -> ESP32-S3 and logic

Type 2 CP <-> protected CP interface <-> ESP32-S3
Type 2 PP  -> protected PP measurement -> ESP32-S3
BRCS01C    -> hardwired trip latch -> K1_HW_ENABLE
Watchdog   -> hardwired enable chain
```

## External connectors

### J_PWR — isolated controller supply

| Pin | Net | Function |
|---:|---|---|
| 1 | +24V_IN | 18–30 VDC from a certified isolated supply |
| 2 | 0V_IN | supply return |
| 3 | PE | protective earth; not joined to 0V on this sheet |

### J_RDC — BRCS01C-05-H1

| Pin | Net | Function |
|---:|---|---|
| 1 | +5V_RDC | filtered sensor supply |
| 2 | RDC_ACDC_RAW | combined AC/DC trip |
| 3 | GND_RDC | sensor return |
| 4 | RDC_CAL | calibration, reserved |
| 5 | RDC_TEST | self-test control |
| 6 | RDC_DC_RAW | 6 mA DC trip |

Pin order is provisional until checked against the supplied harness drawing.

### J_CP_PP — attached Type 2 cable

| Pin | Net | Function |
|---:|---|---|
| 1 | CP | control pilot |
| 2 | PP | proximity pilot |
| 3 | PE | protective-earth reference |

### J_K1 — future contactor assembly

| Pin | Net | Direction | Function |
|---:|---|---|---|
| 1 | +24V_K1 | out | protected coil feed |
| 2 | K1_CMD_N | out | controlled coil return |
| 3 | K1_AUX_L | in | future L-pole feedback |
| 4 | K1_AUX_N | in | future N-pole feedback |
| 5 | LOAD_V_PRESENT | in | isolated post-contactor voltage detector |
| 6 | GND_SELV | reference | low-voltage return |

The K1 drive stage is intentionally not populated. The safety chain exposes K1_HW_ENABLE; firmware cannot bypass it.

## Mandatory safe states

- ESP32 missing/reset, watchdog timeout, bad RDC supply, active RDC trip, or latched fault forces K1_HW_ENABLE low.
- CP defaults to unavailable until startup self-tests pass.
- Wi-Fi, MQTT, display, RFID, USB, and NTP never participate in the hardwired trip path.
