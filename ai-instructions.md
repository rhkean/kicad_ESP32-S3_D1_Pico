# AI Instructions for WLED UnderGlow Project

## Overview
This file contains mandatory instructions for all AI agents and human maintainers working on the WLED UnderGlow project. These instructions ensure consistency, accuracy, and compliance with design governance.

## Mandatory Requirements

### 1. Project Reference
- **Always read `WLED_UnderGlow_AI_Skill.md` first** before providing any design, firmware, or BOM recommendations.
- Use this file as the single source of truth for project status, key decisions, and component datasheets.
- If the AI skill file is outdated, update it before proceeding with recommendations.

### 2. Datasheet Governance
- **Cross-reference datasheets for every electrical/firmware behavior suggestion.**
- For any recommended fix, change, or feature:
  - Include component name, key datasheet section (e.g., "TPS2121 section 7.1"), and exact constraint (voltage/current/timing).
- If a component datasheet is not available in the AI skill file or KiCad sources:
  - Stop and request the exact URL from the user.
  - Do not proceed with vague or unvalidated recommendations.

### 3. BOM Accountability
- All BOM changes must be validated against the comprehensive CSV in the project root (`BOM-ESP32-S3 Pico D1.csv`).
- Include manufacturer part numbers (MPN) and LCSC codes in all procurement discussions.
- Avoid adding new line items without cost impact analysis (each additional BOM line adds ~$2 to assembly costs).

### 4. Design Decision Validation
- Ground all recommendations in verified component constraints from datasheets.
- For power, timing, or safety-critical features, cite specific datasheet sections.
- Maintain separation of "clean" and "dirty" grounds as per the grounding strategy.

### 5. Firmware and Hardware Changes
- Before suggesting code or schematic changes, confirm the update's impact on:
  - Voltage/current ratings
  - Timing requirements
  - Thermal derating
  - Fault behavior
- Use ESP32-S3 RMT for LED control, SPI for CAN, and ADC for current sensing as specified.

### 6. File Maintenance
- Update `WLED_UnderGlow_AI_Skill.md` with any new datasheets, BOM changes, or status updates.
- Sync changes between `WLED_UnderGlow_AI_Skill.md` and `WLED UnderGlow.md` to prevent discrepancies.
- Annotate revisions with dates and summaries.

## Key Components and Links
Refer to the AI skill file for the complete list of datasheets and BOM. Critical components include:
- ESP32-S3-PICO-1
- TCAN4550-Q1
- TPS2121-Q1
- TPSM82903
- TPS2HC08-Q1
- 74LVC1G14 / 74LV1T126

## Enforcement
AI agents must adhere to these instructions in all interactions within this project workspace. If in doubt, ask for clarification or additional datasheet links.