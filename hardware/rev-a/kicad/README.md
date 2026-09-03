# KiCad schematic — Rev A

## Open the schematic

Open `evse-controller-rev-a.sch` in KiCad 8 or 9. It is stored in the KiCad legacy schematic format so it remains human-reviewable in GitHub. KiCad will offer to convert it; save the converted result as `.kicad_sch`.

## Current content

- protected 24 V input;
- provisional 5.5 V, 5 V RDC and 3.3 V power stages;
- ESP32-S3-WROOM-1;
- BRCS01C-05-H1 connector;
- fail-safe pull-up interpretation for both trip outputs;
- hardware safety-latch functional block;
- TPS3430-Q1 watchdog functional block;
- Type 2 CP/PP connector;
- future contactor interface, with contactor and coil driver DNP.

## Verification status

The text structure is balanced and contains 16 component blocks and 52 wire records. KiCad and ERC were not available on the authoring host, so graphical import, symbol resolution and ERC are **not yet verified**.

Do not assign PCB footprints or order a PCB from this draft. Several complex devices are represented as named functional connector blocks while their exact implementation is refined in the adjacent sheet-definition documents.

## Required KiCad review

1. Open and convert the legacy file.
2. Confirm that `RF_Module:ESP32-S3-WROOM-1` resolves with the installed symbol library.
3. Inspect power pin visibility and pin mapping.
4. Replace each named functional connector block with the final manufacturer symbol.
5. Add no-connect flags intentionally.
6. Run ERC with custom safety rules.
7. Export PDF and visually inspect every net and junction.

The contactor will be added only after its exact model and coil interface are frozen.
