# Sheet 01 — Low-voltage power and ESP32-S3

## 24 V input protection

| Ref | Device/value | Connection |
|---|---|---|
| J_PWR | 3-way locking connector | pins per Sheet 00 |
| F1 | 1 A preliminary fuse | series with +24V_IN |
| D1 | SMBJ33A unidirectional TVS | cathode to +24V_PROT, anode to 0V_IN |
| Q_RP1 | 60 V P-channel MOSFET | high-side reverse-polarity protection |
| R_RP1 | 100 kΩ | Q_RP1 gate to source |
| Z_RP1 | 12 V gate-source zener | Q_RP1 gate protection |
| C24_1 | 100 nF, 50 V X7R | +24V_PROT to GND_SELV |
| C24_2 | 47 µF, 50 V | +24V_PROT to GND_SELV |

Final fuse and MOSFET values depend on the future contactor coil and UI load.

## 5.5 V preregulator

Provisional U_BUCK is TPS54360B, a 60 V buck regulator qualified for the required temperature range.

- VIN: +24V_PROT with 2.2 µF and 100 nF local ceramics.
- EN: UVLO divider; preliminary rising threshold 17.5 V.
- BOOT: 100 nF to SW.
- SW: preliminary 22 µH inductor.
- OUT: +5V5_PRE.
- FB: divider calculated for 5.50 V.
- Output: 2 x 22 µF, 10 V X7R plus 100 nF.
- COMP values must be calculated from the final L/C set and verified for loop stability.

## Dedicated RDC rail

| Ref | Part/value | Connection |
|---|---|---|
| U_RDC_LDO | TLV76750 | IN=+5V5_PRE, OUT=+5V_RDC |
| C_RDC_IN | 1 µF X7R | at IN |
| C_RDC_OUT | 4.7 µF X7R | at OUT |
| FB_RDC | 600 Ω @ 100 MHz, rated at least 300 mA | before J_RDC |
| C_RDC_LOCAL1 | 100 nF X7R | sensor side |
| C_RDC_LOCAL2 | 10 µF X7R | sensor side |

The controlled BRCS datasheet remains authoritative for supply tolerance and capacitance.

## 3.3 V rail

Provisional U_3V3 is TPS62133 from +5V5_PRE, with at least 1 A pulse capability.

- 10 µF input capacitor.
- Inductor and output capacitors per final regulator calculation.
- 22 µF plus 100 nF at the ESP32 supply cluster.
- A supervisor prevents partial boot below the validated rail.

## ESP32-S3 module

U1 is ESP32-S3-WROOM-1-N8R8; industrial-temperature suffix must be confirmed.

- 100 nF at every 3.3 V supply pin.
- EN: 10 kΩ pull-up, 1 µF to ground, reset button to ground.
- GPIO0: 10 kΩ pull-up and boot button to ground.
- USB D+/D−: 22 Ω series resistors near U1 and low-capacitance ESD protection.
- RF keepout follows the exact Espressif module drawing.
- Prefer WROOM-1U and an external antenna for a metal distribution cabinet.

## Provisional GPIO map

| Function | GPIO | Reset state |
|---|---:|---|
| CONTACTOR_REQUEST | 4 | pulled low |
| WATCHDOG_WDI | 5 | inactive |
| RDC_TEST_CMD | 6 | inactive |
| RDC_DC_TRIP_SENSE | 7 | defined input |
| RDC_ACDC_TRIP_SENSE | 8 | defined input |
| FAULT_LATCH_SENSE | 9 | defined input |
| K1_AUX_L | 10 | input |
| K1_AUX_N | 11 | input |
| STATUS_LED_DATA | 12 | off |
| LOCAL_BUTTON | 13 | pulled |
| CP_PWM | 14 | off |
| CP_ADC | 15 | input |
| PP_ADC | 16 | input |
| I2C_SDA | 17 | pulled up |
| I2C_SCL | 18 | pulled up |
| USB D− / D+ | 19 / 20 | native USB |
| RFID SPI | 35–38 | inactive |

Allocation remains provisional until display and RFID modules are frozen.
