# Sheet 03 — Control Pilot and Proximity Pilot

> Architecture based on OpenEVSE_PLUS v6.5.1 and adapted for ESP32-S3, 3.3 V ADC limits and EU Type 2. See ../../OPENEVSE-ATTRIBUTION.md.

## OpenEVSE reference circuit

The upstream v6.5.1 schematic uses:

- LMV358 on +12 V and a generated negative rail;
- TC1044S charge pump for the negative rail;
- PWM-driven pilot buffer;
- 820 Ω pilot series resistor;
- P6SMB16CA pilot TVS;
- pilot measurement network using 56 kΩ, 100 kΩ and 200 kΩ resistors into a 5 V MCU ADC;
- separate PP_READ input;
- independent relay control, GFCI test and weld-detect signals.

We retain the topology but do not reuse the 5 V ADC divider unchanged.

## CP power rails

- +12V_CP from the protected 24 V input using a regulated converter.
- −12V_CP generated with TC1044S-compatible charge-pump topology only if output-current and temperature calculations pass.
- Local 10 µF and 100 nF capacitors at both rails.
- CP-domain ground/reference strategy must be frozen with the PE/SELV insulation review.

## CP generator

```text
ESP32 CP_PWM (1 kHz)
 -> default-off gate/buffer
 -> LMV358-class amplifier powered from +12V_CP / -12V_CP
 -> 820 Ω OpenEVSE baseline source resistor
 -> P6SMB16CA-class bidirectional protection
 -> Type 2 CP
```

Design requirements:

- approximately +12 V / −12 V waveform;
- State F/unavailable during reset, boot, watchdog fault or safety inhibit;
- duty cycle limited to the configured safe current;
- output short-circuit survival;
- CP output voltage read back independently;
- negative half-cycle/vehicle-diode validation.

The final source resistance must satisfy IEC 61851-1 across amplifier output impedance, resistor tolerance and temperature. The 820 Ω upstream value is a starting point, not an automatic release value.

## 3.3 V CP measurement

The OpenEVSE 5 V sensing network cannot connect directly to ESP32-S3. Preliminary protected divider:

- R_CP_IN = 390 kΩ from CP to CP_SENSE;
- R_CP_BIAS = 82 kΩ from +3V3_A to CP_SENSE;
- R_CP_GND = 120 kΩ from CP_SENSE to GND_A;
- C_CP = DNP initially; select after response-time simulation;
- rail clamps must limit injection current without clipping valid states;
- buffer with a rail-to-rail 3.3 V op-amp or comparator before the ESP32 ADC.

Ideal calculated CP_SENSE is approximately 0.41 V at CP=−12 V and 3.08 V at CP=+12 V. Recalculate with tolerances, clamp leakage, ADC error and CP rail limits before release.

Use comparator windows for safety/state detection; treat the ESP32 ADC as measurement/diagnostics rather than the only state discriminator.

## PP measurement

Adopt OpenEVSE's separate PP_READ concept, adapted for the attached Type 2 cable:

- precision pull-up/reference;
- protected and filtered 3.3 V ADC input;
- thresholds for the actual Type 2 PP resistor values;
- open/short plausibility checks.

Advertised current is the minimum of:

```text
32 A product limit
installation configuration
PP cable limit
thermal derating
local/user limit
remote load-management limit
```

## Protection and validation

- CP and PP protection must survive connector ESD and foreseeable miswiring.
- PE is never fused or switched.
- Validate States A/B/C/D/E/F, diode check, CP short, PP open/short and reset behavior with an EV simulator.
- Validate 1 kHz frequency, duty-cycle accuracy, rise/fall time and voltage levels over −25…+55 °C.
- Do not claim OpenEVSE compatibility or approval; this is an attributed adaptation.
