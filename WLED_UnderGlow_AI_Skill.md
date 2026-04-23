# WLED UnderGlow — AI Project Skill Reference

## 1. Objective

This document is the authoritative AI-maintained reference for the WLED UnderGlow project. It provides AI-specific guidelines, datasheet governance, and project references.

AI agents and maintainers must:
- Read this file first before making design or firmware recommendations
- Cross-reference datasheets for every suggested electrical/firmware behavior
- Avoid vague answers by grounding all recommendations in the component datasheets
- Ask for explicit datasheet URLs when unavailable or ambiguous (except for Texas Instruments websites at www.ti.com, where content can be accessed directly for datasheets, product pages, application notes, and other resources)
- Refer to [WLED UnderGLow.md](WLED UnderGLow.md) for detailed technical specifications, project status, design decisions, BOM, and other project details

## 2. Datasheet Governance (mandatory)

This project must always include component datasheet links in change records and feature decisions.

- For any suggested fix/change, include: component name, key datasheet section (e.g., "TPS2121 section 7.1") and exact constraint (voltage/current/timing)
- If a component is not in KiCad doc with datasheet link, stop and ask:
  - "Provide the exact URL for [part number] datasheet so I can validate before recommending behavior."

## 3. Known Datasheet Links from KiCad sources

| Component | Datasheet URL |
|---|---|
| ESP32-S3 | https://documentation.espressif.com/esp32-s3_datasheet_en.pdf |
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
| TPS274C65ASH | https://www.ti.com/lit/ds/symlink/tps274c65.pdf |
| ACM2012H-900-2P-T03 | https://product.tdk.com/system/files/dam/doc/product/emc/emc/cmf_cmc/catalog/cmf_automotive_signal_acm2012h-t03_en.pdf |
| CL10A226MO7JZN | https://weblib.samsungsem.com/mlcc/mlcc-ec-data-sheet.do?partNumber=CL10A226MO7JZN |
| PYU-AC_51 | https://yageogroup.com/content/datasheet/asset/file/PYU-AC_51_ROHS_L |

## 4. Missing/Absent critical datasheet links

- None for mandatory active components (ESP32, TCAN4550, TPS2121, TPSM82903, TPS2HC08, 74LVC1G14, 74LV1T126).
- If additional BOM parts are introduced, update this table and re-check.

## 5. From KiCad file references

### Primary schematic files containing datasheet data:
- `ESP32-S3-PICO-1.kicad_sch` (ESP32, caps, resistor families)
- `CANBus_Interface.kicad_sch` (TCAN4550 + I/O protection parts)
- `USB_Power_Interface.kicad_sch` (TPS2121, TPSM82903 + power path, vendor links)
- `WLED_Outputs.kicad_sch` (TPS2HC08, 74LV1T126, etc.)

### How to use these references
- When asked for a design decision update, inspect the relevant .kicad_sch section for `.Datasheet` property.
- Link to the same URL so stakeholders can directly validate.

## 6. Action points for AI and maintainers

1. Periodically re-run datasheet extraction from KiCad (search `property "Datasheet"`) and reconcile with BOM in [WLED UnderGLow.md](WLED UnderGLow.md).
2. On every firmware/or HW change, confirm whether the update touches:
   - voltage rating, current rating, timing requirement, thermal derating, or fault behavior of a referenced device.
3. If no datasheet in component symbol exists, add it together with any cross-check URL from vendor page.
4. Annotate all changes with revision history and date.
5. Refer to [WLED UnderGLow.md](WLED UnderGLow.md) for project status, design decisions, BOM, GPIO mapping, and other technical details.

## 7. Revision history

- v1.0 (2026-03-25): initial AI skill composition from original WLED UnderGlow master record.
- v1.1 (2026-03-25): added BOM table from JLCPCB CSV export.
- v1.2 (2026-03-25): updated BOM table to use original BOM from WLED UnderGlow.md with manufacturer and MPN details.
- v1.3 (2026-03-25): reverted BOM table to use JLCPCB CSV as the most up-to-date source.
- v1.4 (2026-03-25): updated BOM table to use comprehensive CSV from project root with full manufacturer/MPN/LCSC details.
- v1.5 (2026-03-25): synced project status with updates from WLED UnderGlow.md technical specification document.
- v2.0 (2026-03-30): Removed duplicated content (project overview, status, design decisions, BOM) to minimize overlap with [WLED UnderGLow.md](WLED UnderGLow.md). Updated to reference main document for technical details.
