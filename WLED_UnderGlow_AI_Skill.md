# WLED UnderGlow — AI Project Skill Reference

## 1. Objective

This document is the authoritative AI-maintained status report for the WLED UnderGlow project. It is designed to be a single source of truth for design state, key decisions, BOM accountability, and datasheet dependencies.

AI agents and maintainers must:
- read this file first before making design or firmware recommendations
- cross-reference datasheets for every suggested electrical/firmware behavior
- avoid vague answers by grounding all recommendations in the component datasheets
- ask for explicit datasheet URLs when unavailable or ambiguous

## 2. Project Overview

- Project: WLED UnderGlow automotive LED + CAN lighting controller
- Architecture: two 30x30mm 6-layer boards (MCU hat and I/O hat) stacked onto QuinLED DigUno
- MCU: ESP32-S3-PICO-1
- CAN: TCAN4550-Q1 (CAN-FD transceiver/controller)
- Power: TPS2121-Q1 power multiplexer (USB-C + external 5V), TPSM82903 3.3V buck converter
- Load Control: TPS2HC08-Q1 high-side switch
- Signal conditioning: 5x 12V inputs through 100k/26.1k divider + 74LVC1G14 inverters
- Outputs: 6x LED strips, level shifted by 74LV1T126, driven by ESP32-S3 RMT

## 3. Current Status (Latest)

### Completed
- schematic and PCBs laid out (MCU hat + I/O hat)
- core BOM captured in `WLED UnderGLow.md`
- power mux design with reverse battery protection and 1.2A limit
- current sense design using 1.62kΩ sense resistor and ADC-based fault detection
- CAN watchdog/suicide logic + wired-OR fault bus
- pin mapping and preliminary MCU GPIO audit

### In-progress / pending
- final pin conflict resolution (SPI vs DS18B20 vs RMT)
- validate I/O hat "dirty ground" routing before final Gerber release
- firmware: 4-hour sleep state flow (Light Sleep → Deep Sleep)
- final BOM consolidation and quote for JLCPCB assembly options

## 4. Key Design Decisions

- L6 power mux uses TPS2121 with diode ground lift (0.65V) for reverse polarity. Verified only power stage has offset.
- Input gate protection uses schmitt-trigger 74LVC1G14; dome light uses diode-steered negated logic path.
- High-side switch uses TPS2HC08 with separate dirty-ground for relay/LED noise.
- LED outputs use 74LV1T126 level shifters and 33Ω series damping resistors.
- Global fault bus: OR-fault line to MCU GPIO for fault monitoring and system reset behavior.

## 5. Datasheet Governance (mandatory)

This project must always include component datasheet links in change records and feature decisions.

- For any suggested fix/change, include: component name, key datasheet section (e.g., "TPS2121 section 7.1") and exact constraint (voltage/current/timing)
- If a component is not in KiCad doc with datasheet link, stop and ask:
  - "Provide the exact URL for [part number] datasheet so I can validate before recommending behavior."

## 6. Known Datasheet Links from KiCad sources

| Component | Datasheet URL |
|---|---|
| ESP32-S3-PICO-1 | https://documentation.espressif.com/esp32-s3-pico-1_datasheet_en.pdf |
| TCAN4550RGY | https://www.ti.com/lit/ds/symlink/tcan4550.pdf |
| TPS2121RUXR | https://www.ti.com/lit/ds/symlink/tps2121.pdf |
| TPSM82903 | https://www.ti.com/lit/ds/symlink/tpsm82903.pdf |
| TPS2HC08-Q1 | https://www.ti.com/lit/gpn/tps2hc08-q1 |
| 74LVC1G14 | https://assets.nexperia.com/documents/data-sheet/74LVC1G14.pdf |
| 74LV1T126 | https://assets.nexperia.com/documents/data-sheet/74LV1T126.pdf |
| 2450AT07A0100 | https://www.johansontechnology.com/docs/1207/2450AT07A0100_rh8sEhS.pdf |
| D15V0H1U2LP | https://www.diodes.com/assets/Datasheets/D15V0H1U2LP.pdf |
| SDM1100LP | https://www.diodes.com/assets/Datasheets/SDM1100LP.pdf |
| ESD7241 | https://www.onsemi.com/pdf/datasheet/esd7241-d.pdf |
| ACM2012H-900-2P-T03 | https://product.tdk.com/system/files/dam/doc/product/emc/emc/cmf_cmc/catalog/cmf_automotive_signal_acm2012h-t03_en.pdf |
| CL10A226MO7JZN | https://weblib.samsungsem.com/mlcc/mlcc-ec-data-sheet.do?partNumber=CL10A226MO7JZN |
| PYU-AC_51 | https://yageogroup.com/content/datasheet/asset/file/PYU-AC_51_ROHS_L |

## 7. Current BOM from Project Root CSV (BOM-ESP32-S3 Pico D1.csv)

| Qty | Value | Manufacturer | Manufacturer Part Number | LCSC |
|---|---|---|---|---|
| 1 | 2450AT07A0100 | Johanson Technology | 2450AT07A0100 | C17531353 |
| 17 | 22uF | Samsung Electro-Mechanics | CL10A226MO7JZNC | C2762594 |
| 29 | 0.1uF | Murata Electronics | GRT033R61E104KE01D | C915760 |
| 5 | 10pF | Murata Electronics | GRM0335C1H100FA01D | C384956 |
| 1 | CMC 0805 | TDK | ACM2012H-900-2P-T03 | C448605 |
| 1 | GT-USB-7014D | G-Switch | GT-USB-7014D | C2843969 |
| 1 | SM04B-SURS-TF(LF)(SN) | JST | SM04B-SURS-TF(LF)(SN) | C545754 |
| 1 | SM05B-SURS-TF(LF)(SN) | JST | SM05B-SURS-TF(LF)(SN) | C5441000 |
| 18 | D15V0H1U2LP-7B | Diodes Incorporated | D15V0H1U2LP-7B | C1973850 |
| 4 | SZESD7241N2T5G | onsemi | SZESD7241N2T5G | C604770 |
| 11 | SDM1100LP-7 | Diodes Incorporated | SDM1100LP-7 | C6999905 |
| 4 | ESP_Header |  |  |  |
| 1 | Orange | XingLight | XL-0201UOC | C5440629 |
| 1 | Red | XingLight | XL-0201SURC | C3646923 |
| 1 | Blue | XingLight | XL-0201UBC | C3646921 |
| 1 | Green | XingLight | XL-0201UGC | C3646922 |
| 1 | 0R | YAGEO | RC0201FR-070RL | C106227 |
| 28 | 5k1 | YAGEO | AC0201FR-075K1L | C226588 |
| 8 | 100k | YAGEO | AC0201FR-07100KL | C226394 |
| 9 | 26k1 | YAGEO | AC0201FR-0726K1L | C910625 |
| 3 | 470R | YAGEO | AC0201FR-07470RL | C226558 |
| 16 | 33R | YAGEO | AC0201FR-0733RL | C226514 |
| 1 | 1k6 | YAGEO | AC0201FR-071K62L | C910613 |
| 2 | SPST-NO | Omron Electronics Inc | B3U-3000P(M)-B | C4364471 |
| 1 | ESP32-S3-PICO-1-N8R2 | Espressif | ESP32-S3-PICO-1-N8R2 | C7558093 |
| 1 | TPS2121 | Texas Instruments | TPS2121RUXR | C485916 |
| 1 | TPSM82903 | TI | TPSM82903SISR | C6290583 |
| 1 | TCAN4550RGYRQ1 | Texas Instruments | TCAN4550RGYRQ1 | C2876736 |
| 1 | TPS2HC08-Q1 | Texas Instruments | TPS2HC08PQVAHRQ1 | C52741465 |
| 6 | 74LV1T126 | Nexperia | 74LV1T126GXH | C547942 |
| 6 | 74LVC1G14 | Nexperia | 74LVC1G14GX,125 | C548273 |
| 1 | 40MHz | Murata Electronics | XRCGE40M000FZA2AR0 | C43786933 |

Note: This BOM is extracted from the comprehensive CSV in the project root, including manufacturer, MPN, and LCSC part numbers for procurement. Some items are excluded from BOM as noted in the CSV.

## 8. Missing/Absent critical datasheet links

- None for mandatory active components (ESP32, TCAN4550, TPS2121, TPSM82903, TPS2HC08, 74LVC1G14, 74LV1T126).
- If additional BOM parts are introduced, update this table and re-check.

## 9. From KiCad file references

### Primary schematic files containing datasheet data:
- `ESP32-S3-PICO-1.kicad_sch` (ESP32, caps, resistor families)
- `CANBus_Interface.kicad_sch` (TCAN4550 + I/O protection parts)
- `USB_Power_Interface.kicad_sch` (TPS2121, TPSM82903 + power path, vendor links)
- `WLED_Outputs.kicad_sch` (TPS2HC08, 74LV1T126, etc.)

### How to use these references
- When asked for a design decision update, inspect the relevant .kicad_sch section for `.Datasheet` property.
- Link to the same URL so stakeholders can directly validate.

## 10. Action points for AI and maintainers

1. Periodically re-run datasheet extraction from KiCad (search `property "Datasheet"`) and reconcile with BOM in `WLED UnderGlow.md`.
2. On every firmware/or HW change, confirm whether the update touches:
   - voltage rating, current rating, timing requirement, thermal derating, or fault behavior of a referenced device.
3. If no datasheet in component symbol exists, add it together with any cross-check URL from vendor page.
4. Annotate all future versions with a `## Revision` section and date.

## 11. Revision history

- v1.0 (2026-03-25): initial AI skill composition from original WLED UnderGlow master record.
- v1.1 (2026-03-25): added BOM table from JLCPCB CSV export.
- v1.2 (2026-03-25): updated BOM table to use original BOM from WLED UnderGlow.md with manufacturer and MPN details.
- v1.3 (2026-03-25): reverted BOM table to use JLCPCB CSV as the most up-to-date source.
- v1.4 (2026-03-25): updated BOM table to use comprehensive CSV from project root with full manufacturer/MPN/LCSC details.
