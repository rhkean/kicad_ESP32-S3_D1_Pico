# BOM Sync Skill

## Description
This skill provides a standardized workflow for synchronizing the Bill of Materials (BOM) from KiCad schematic files to the project's documentation files. It ensures that both the AI skill reference (`WLED_UnderGlow_AI_Skill.md`) and the main technical specification (`WLED UnderGlow.md`) contain up-to-date, accurate BOM information extracted directly from the schematics.

## When to Use
- When schematic components are added, removed, or modified
- Before final BOM consolidation for manufacturing quotes
- To verify BOM consistency between design and documentation
- After resolving component substitutions or datasheet updates

## Prerequisites
- KiCad installed and accessible via command line (`kicad-cli`)
- Python 3 available for BOM processing scripts
- Project root CSV BOM file (`BOM-ESP32-S3 Pico D1.csv`) for LCSC part numbers and descriptions

## Process Steps

### 1. Export BOM from KiCad Schematics
Run the following command to export a detailed BOM from the root schematic:
```
kicad-cli sch export bom --fields Reference,Value,Footprint,Manufacturer,"Manufacturer Part Number",Qty --output bom_full.csv "ESP32-S3 Pico D1.kicad_sch"
```
This creates `bom_full.csv` with individual component entries including manufacturer and MPN data.

### 2. Group the BOM
Create and run a Python script to group identical components and generate a consolidated BOM:
```python
import csv
from collections import defaultdict

def main():
    with open('bom_full.csv', 'r') as f:
        reader = csv.DictReader(f)
        bom = defaultdict(lambda: {'Qty': 0, 'Refs': []})
        for row in reader:
            key = (row['Value'], row['Manufacturer'], row['Manufacturer Part Number'])
            bom[key]['Qty'] += 1
            bom[key]['Refs'].append(row['Reference'])
    
    with open('bom_grouped.csv', 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['Qty', 'Value', 'Manufacturer', 'Manufacturer Part Number', 'References'])
        for (value, manufacturer, mpn), data in bom.items():
            refs = ','.join(sorted(data['Refs']))
            writer.writerow([data['Qty'], value, manufacturer, mpn, refs])

if __name__ == '__main__':
    main()
```
Run: `python group_bom.py`

### 3. Update AI Skill File (`WLED_UnderGlow_AI_Skill.md`)
- Read `bom_grouped.csv` and the existing project root CSV (`BOM-ESP32-S3 Pico D1.csv`) to correlate LCSC part numbers
- Replace the BOM table in section 7 with the new grouped data, including LCSC column
- Update the revision history with a new entry noting the BOM sync date

### 4. Update Main Specification (`WLED UnderGlow.md`)
- Read `bom_grouped.csv` and the project root CSV for descriptions
- Replace the BOM table in Appendices C with the updated grouped BOM
- Ensure the table format matches: Qty | Value | Manufacturer | Manufacturer Part Number | Description

### 5. Cleanup Temporary Files
Remove all intermediate files created during the process:
```
rm bom_from_sch.csv bom_full.csv bom_grouped.csv group_bom.py bom_generator.py
```

## Validation Steps
- Verify that the total component count matches between schematic export and grouped BOM
- Cross-check quantities and part numbers against the original project CSV
- Ensure no components are missing manufacturer/MPN information
- Confirm that the AI skill file and main spec have identical BOM data

## Error Handling
- If KiCad export fails, check that the schematic file path is correct and KiCad is installed
- If Python script fails, verify CSV format and field names
- If BOM quantities don't match, review schematic for unplaced or DNP components
- If LCSC correlation fails, manually add missing part numbers to the project CSV

## Dependencies
- KiCad 8.0+ for BOM export functionality
- Python 3 with csv module
- Access to project root BOM CSV for procurement data

## Revision History
- v1.0 (2026-03-25): Initial BOM sync skill definition based on manual process execution.