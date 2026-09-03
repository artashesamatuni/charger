# OpenEVSE design-basis attribution

Parts of the EVSE controller architecture and subsequent hardware implementation are adapted from **OpenEVSE_PLUS v6.5.1**.

- Upstream project: https://github.com/OpenEVSE/OpenEVSE_PLUS
- Reference revision inspected: repository tree `b9f101e9b7bae0846ca72638b9d22bb8c49298d5`
- Reference schematic: `OpenEVSE_PLUS_v6.5.1/OpenEVSE_PLUS_v6.5.1.sch`
- Upstream hardware license: Creative Commons Attribution-ShareAlike 3.0 (CC BY-SA 3.0)
- License text: https://creativecommons.org/licenses/by-sa/3.0/

## Functions used as a design basis

- ±12 V control-pilot generation;
- pilot ADC/state measurement;
- proximity-pilot measurement;
- residual-current/GFCI self-test pattern;
- separate relay commands and feedback/weld detection;
- current-transformer measurement;
- power-on safe defaults.

## Intentional differences

- ESP32-S3 at 3.3 V replaces ATmega328P at 5 V.
- BRCS01C-05-H1 and an external Type A RCD replace the OpenEVSE GFCI implementation.
- EU single-phase Type 2, TN-S/TN-C-S and L+N disconnection replace the North-American baseline.
- 24 VDC controller input replaces the onboard mains supply.
- An independent window watchdog and hardwired set-dominant RDC latch are added.
- CP/PP input scaling and protection are recalculated for 3.3 V.
- Contactor selection and driver are deferred.

Derived hardware files must preserve attribution and be distributed under terms compatible with CC BY-SA 3.0. This notice is not a claim that OpenEVSE has reviewed or approved this design.
