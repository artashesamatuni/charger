# Sheet 03 — Control Pilot and Proximity Pilot

> CP/PP values are preliminary and must be checked against the applicable IEC 61851-1 edition and the attached Type 2 cable before release.

## Isolation and grounding

CP is referenced to protective earth at the EVSE. The preferred design isolates the application electronics from the CP/PE domain:

- isolated PWM command into the CP domain;
- isolated DC/DC for CP-domain rails;
- isolated return of CP state;
- no direct GND_SELV-to-PE connection.

This keeps USB, display, RFID and service interfaces away from the PE-referenced CP domain.

## CP generator

Requirements:

- nominal 1 kHz PWM;
- approximately +12 V / −12 V;
- specified source resistance;
- hardware default State F/unavailable;
- short-circuit protection;
- independent monitoring of positive and negative levels;
- diode check.

```text
ESP32 CP_PWM
 -> digital isolation
 -> CP-domain push-pull driver
 -> source resistor network
 -> surge protection
 -> CP
```

Never drive CP directly from an ESP32 GPIO or a single 0–12 V transistor stage.

## CP measurement

Use two CP-domain comparator channels:

- positive voltage window for States A/B/C/D;
- negative half-cycle and vehicle-diode detector.

Return isolated digital state bits to the ESP32. Add a CP-domain ADC test point for calibration. Calculate thresholds from rail tolerance, source resistance, cable loading and normative state windows.

## PP measurement

For the attached cable, measure the PP coding resistor with:

- precision reference/pull-up in the PE-referenced domain;
- series protection;
- low-capacitance surge/ESD protection;
- filtered ADC or comparator measurement;
- isolation back to the controller if the isolated CP architecture is retained.

Advertised charging current is the minimum of the 32 A product limit, installation setting, cable PP limit, thermal limit and user configuration.

## Protection

CP and PP protection must survive connector ESD and foreseeable miswiring without clipping normal CP operation. PE is never fused or switched.
