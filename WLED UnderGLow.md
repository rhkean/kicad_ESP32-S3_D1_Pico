# WLED UnderGlow: Technical Specification Document

## 1. Introduction

### Project Overview
The WLED UnderGlow is an automotive-grade lighting controller designed for extreme efficiency and reliability in vehicle underbody lighting applications. The system integrates LED strip control with CAN FD communication, advanced power management, and vehicle signal processing.

### Key Features
- Dual 30x30mm 6-layer PCBs (MCU Hat and I/O Hat) stacked on QuinLED DigUno base board
- ESP32-S3-Pico-1 microcontroller with integrated CAN FD via TCAN4550-Q1
- Sophisticated power muxing with reverse polarity protection
- High-side current sensing and multi-stage sleep cycles for battery protection
- 6-channel LED output with level shifting for WS2811/WS2814/SK6812 strips
- 5-channel vehicle signal inputs (turn signals, brake, reverse, dome light)
- Global health monitoring with wired-OR fault bus

### Target Application
Automotive underglow lighting system for Toyota GR86/Subaru BRZ, with CAN connectivity for vehicle integration and diagnostics.  Initial implementation will rely on signal inputs tapped from the various inputs to trigger various LED patterns, with intentions to rely on CAN packets in future interations.

## 2. System Architecture

### Overall Architecture
The system consists of three stacked boards:
- **QuinLED DigUno (Base):** Provides 5V regulated power from vehicle battery (via DCM connector), 5A fuse protection
- **I/O Hat:** (inserted into DigUno) includes CAN transceiver, vehicle inputs, LED data outputs, and load switches
- **MCU Hat:** (inserted into I/O Hat) ESP32-S3-Pico-1 with power management, USB-C interface, and antenna

### Grounding Strategy
- **Clean System GND (0V):** Referenced by ESP32-S3, level shifters, CAN transceiver, and TPS2HC08-Q1
- **Dirty GND (lifted +0.65V):** TPS2121 lifted via Schottky diode for reverse polarity protection
- **Header Isolation:** One GND pin dedicated to dirty GND (relay flyback), isolated until main board star-point
- **LED Data Returns:** Each LED data line paired with GND in twisted-pair, returned to clean GND

### Power Architecture
- Input: 12V vehicle battery via QuinLED DigUno
- Power Mux (TPS2121): Selects between USB-C (VBUS) and battery power with fast switchover (~5µs)
- Buck Converter (TPSM82903): 5V to 3.3V for MCU and digital logic
- Load Switches (TPS2HC08-Q1): Controls power to interior LED strip output and 1 exterior relay

## 3. Hardware Design

### MCU Hat Specifications
- **Dimensions:** ~30x~30mm, 6-layer PCB
- **Footprint:** QuinLED-ESP32 (D1 Mini32) compatible
- **Components:**
  - ESP32-S3-PICO-1 with 8MB Flash, 2MB PSRAM
  - 0402 chip antenna for WiFi/Bluetooth
  - TPS2121 power mux with reverse polarity protection
  - TPSM82903 3A buck converter
  - USB-C connector for power/programming
  - Status LEDs (power, activity, reverse polarity)
  - Reset and boot-mode buttons
- **Layer Zoning (L5):** Antenna ground (top 1/5), 3.3V power pour (middle 2/5), internal power bus (bottom 2/5)

### I/O Hat Specifications
- **Dimensions:** 30x30mm, 6-layer PCB
- **CAN Interface:**
  - TCAN4550-Q1 CAN FD transceiver/controller
  - SPI communication with MCU
  - Watchdog with suicide switch capability
- **Vehicle Inputs (5 Channels):**
  - Signals: L/R turn, brake, reverse (all active-high), & dome light (active-low)
  - Protection: 100k/26.1k voltage divider, 0.1µF filtering, 74LVC1G14 Schmitt inverters
  - Dome specific: Diode steering for current leakage prevention
- **LED Outputs (6 Channels):**
  - Level shifting: 74LV1T126 buffers (3.3V to 5V)
  - Termination: 33Ω series resistors for reflection damping
  - Data generation: ESP32 RMT peripheral with DMA
  - Grounding: Twisted-pair with clean GND return
- **Power Management:**
  - TPS2HC08-Q1 dual high-side switch (VSS to clean GND)
  - 3 Separate 2A@12V channels for interior LEDs
  - 1 exterior Bosch style relay connection with GND isolation to protect from relay flyback
  - Implemented reverse-polarity ground circuit protection to protect against incorrect battery terminal connections
- **Stackup:** 6-layer boards with optimized routing for high-density components
- **Component Placement:** Headers on L1 for vehicle interface, electronics clustered on L6
- **Connector Requirements:** JST NSHD series for wire-to-board connections
  - **SM24B-NSHDZS-TF 2x12 side-entry header (19 pins allocated):**
    - 1 CAN_H/CAN_L pair
    - 6 +12V/GND 1A Power input pairs (6A input)
    - 5 12V Vehicle input signals
  - **SM24B-NSHDZS-TF 2x12 side-entry header (24 pins allocated):**
    - 6 LED data/gnd pairs
    - 6 +12V/GND 1A Power output pairs (paralleled for 3 x 2A)
  - **??? 1x02 header:**
    - Relay +/-
- **BOM Consolidation:** 
  - Standardized resistor values
    - 5.1kΩ pull-ups/downs and line protection, 100k for weak pull-ups
    - Voltage dividers where needed 100k/26.1k dividers
    - termination resistors: 33Ω
    - 470Ω for some line filters

## 4. Firmware Design

### Microcontroller
- **Platform:** ESP32-S3-Pico-1
- **Base:** WLED
- **Peripherals:**
  - RMT: 6-channel LED data output (built into WLED)
  - SPI: CAN transceiver and load-switch communication; need UserMod for sleep and wake functionality
  - ADC: Current sensing from load switches; need UserMod for displaying current draw
  - GPIO: Vehicle inputs, load switch control, fault monitoring, need to create config template
- **Sleep Modes:** Light Sleep → Deep Sleep with 4-hour timer for battery protection

### CAN Communication
- **Protocol:** CAN FD with TCAN4550
- **Features:** Bus monitoring, wake-on-CAN, watchdog reset
- **Safety:** Suicide switch for system recovery

### LED Control
- **Supported Strips:** WS2812B, SK6812, WS2814
- **Timing:** Precise RMT DMA for protocol compliance; default functionality for WLED on S3 mcu
- **Power Control:** Independent enable for interior/exterior groups; controlled through WLED's relay setup

### Fault Monitoring
- **Global Health Bus:** Wired-OR interrupt on GPIO_03; Usermod?
- **Monitored Signals:** Load switch FLT, CAN INT

## 5. Design Decisions and Constraints

### Major Design Decisions
- **Power Muxing:** TPS2121 with 0.65V ground lift for reverse polarity protection
- **Input Protection:** 74LVC1G14 as buffers for vehicle signals
- **Dome Light Logic:** Diode-isolated pull-up to prevent current leakage
- **Global Health Bus:** Wired-OR fault monitoring system
- **BOM Consolidation:** Standardized resistor and capacitor values to minimize line items
- **Glitch Filter:** Implement the mcu's glitch filter to avoid extraneous capacitors on data lines.
  - NOTES:
    - will need to determine which pins to apply this to. (SPI, Fault Bus, etc.)
    - will need to disable feature before entering sleep mode and re-enable on wake-up

### Design Constraints
- **MCU Hat GPIO:** Fixed design, alterations must be handled on I/O hat.  Only changes that will improve future, unrelated uses of the MCU Hat will be considered.
- **BOM Costs:** Each additional line item adds $2 to assembly costs
- **PCB Density:** 30x30mm boards with 6 layers for high component density

## 6. Project Status and TODO List

### Completed
- [x] Schematic for both hats
- [x] PCB layout for MCU hat
- [x] Core BOM with manufacturer part numbers
- [x] Power management design with protection features
- [x] CAN watchdog and fault monitoring logic
- [x] Preliminary GPIO mapping and pin audit
- [x] Pin conflict resolution for DS18B20 vs SPI
- [x] Resolve potential voltage divider on TPS2HC08 EN pins
- [x] Select wire-to-board connectors

### In Progress
- [ ] reworking MCU Hat to incorporate series termination resistors on SPI lines (effectively dedicating GPIO11-13 to future SPI on MCU Hat)
- [ ] Rework schematic for TPS2HCS10-Q1 load switch
    - Does the WD_Trigger (IO02) need a '125 level shift buffer?
- [ ] PCB routing I/O hat (including dirty GND isolation)

### Pending
- [ ] Final GPIO audit
- [ ] Final BOM consolidation and JLCPCB quote
- [ ] Firmware development and testing
  - [ ] Sleep cycles
  - [ ] CAN controller initialization
    - [ ] configure GPIO1, GPO2, and $\overline{\text{INT}}$ pins
    - [ ] setup up WUP and wake request
  - [ ] SPI setup
    - [ ] enable internal pull-up on MISO (GPIO13)
    - [ ] define $\overline{\text{CS}}$ for CAN and Load-Switch
    - [ ] setup CAN SPI communications
    - [ ] setup load-switch SPI communications
  - [ ] FAULT handling
    - [ ] configure GPO2 for WatchDog Reset
    - [ ] configure GPIO1 as hardware watchdog heartbeat
  - [ ] implement ESP32-S3 glitch filter
    - [ ] MISO (GPIO13)
    - [ ] $\overline{\text{FAULT}}$ input pin - 1 cycle glitch filter
    - [ ] Vehicle inputs (Dome, Turn Signals, Reverse, and Brake) - 8 cycle glitch filter
- [ ] Vehicle installation

## Appendices

### A. Code Snippets


### B. GPIO Mapping
#### ESP32-S3-Pico-1 Proposed GPIO Mapping

| Pin | Signal Name | Function | Hardware Peripheral | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **IO_00** | **BOOT** | Strapping Pin | Input | Flash/Boot mode selector |
| **IO_01** | **(NC)** | — | — | Unassigned |
| **IO_02** | **WLED Activity LED & CAN WD pulse** | Digital Output | GPIO | Since WLED toggles this pin on each loop interation, we can use this as our watchdog pulse to the CAN controller |
| **IO_03** | **(NC)** | — | — | Unassigned |
| **IO_04** | **$\overline{\text{WKRQ}}$** | Digital Input | **RTC_GPIO4** | TCAN Wake Request |
| **IO_05** | **(NC)** | — | — | Unassigned |
| **IO_06** | **$\overline{\text{FAULT}}$** | Digital Input | **RTC_GPIO6** | Global $\overline{\text{FLT}}$ (TCAN + Load Switch) |
| **IO_07** | **(NC)** | - | - | Unassigned |
| **IO_08** | **(NC)** | — | — | Unassigned |
| **IO_09** | **TCAN4550 CS** | Digital Output | GPIO | TCAN4550 Chip Select |
| **IO_10** | **TPS2HC08-Q1 CS** | Digital Output | GPIO | TPS2HC08-Q1 Chip Select |
| **IO_11** | **SPI_MOSI** | Data Out | **FSPID** | SPI2 MOSI |
| **IO_12** | **SPI_CLK** | Clock | **FSPICLK** | SPI2 Clock |
| **IO_13** | **SPI_MISO** | Data In | **FSPIQ** | SPI2 MISO |
| **IO_14** | **LED1** | LED Data | **RMT / GPIO** | Interior LED Strip Channel 1 |
| **IO_15** | **LED2** | LED Data | **RMT / GPIO** | Interior LED Strip Channel 2 |
| **IO_16** | **LED3** | LED Data | **RMT / GPIO** | Exterior LED Strip Channel 3 (Relay) |
| **IO_17** | **LED4** | LED Data | **RMT / GPIO** | Exterior LED Strip Channel 4 (Relay) |
| **IO_18** | **LED5** | LED Data | **RMT / GPIO** | Exterior LED Strip Channel 5 (Relay) |
| **IO_19** | **USB_D-** | USB Data | **USB_OTG** | Native USB Peripheral |
| **IO_20** | **USB_D+** | USB Data | **USB_OTG** | Native USB Peripheral |
| **IO_21** | **LED6** | LED Data | **RMT / GPIO** | Exterior LED Strip Channel 6 (Relay) |
| **IO_38** | **(NC)** | — | — | Unassigned |
| **IO_39** | **(NC)** | — | — | Unassigned |
| **IO_40** | **$\overline{\text{Dome}}$** | Digital $V_{BAT}$ Input | GPIO | Active Low Dome Light Signal |
| **IO_41** | **Brake** | Digital $V_{BAT}$ Input | GPIO | Active High Brake Signal |
| **IO_42** | **Reverse** | Digital $V_{BAT}$ Input | GPIO | Active High Reverse Signal |
| **IO_43** | **RTurn** | Digital $V_{BAT}$ Input | GPIO | Active High Right Turn Signal |
| **IO_44** | **LTurn** | Digital $V_{BAT}$ Input | GPIO | Active High Left Turn Signal |
| **IO_45** | **Interior LED EN** | Digital Output | **Strapping Pin** | Interior LED Strip Enable. Pin is pulled low by default for boot strapping with $5.1\text{k}\Omega$ resistor |
| **IO_46** | **Relay EN** | Digital Output | **Strapping Pin** | High-Side Relay Trigger. Pin is pulled low for boot strapping with $5.1\text{k}\Omega$ resistor |
| **IO_47** | **(NC)** | — | — | Unassigned |
| **IO_48** | **(NC)** | — | — | Unassigned |

### C. Bill of Materials

| Qty | Value | Manufacturer | Manufacturer Part Number | Description |
|---|---|---|---|---|
| 17 | 22uF | Samsung Electro-Mechanics | CL10A226MO7JZNC | 22uF 16V X5R ±20% 0603 MLCC SMD |
| 29 | 0.1uF | Murata Electronics | GRT033R61E104KE01D | 100nF 25V X5R ±10% 0201 MLCC |
| 5 | 10pF | Murata Electronics | GRM0335C1H100FA01D | 10pF 50V C0G ±1% 0201 MLCC, SMD |
| 1 | 2450AT07A0100 | Johanson Technology | 2450AT07A0100 | 2.4GHz Chip Antenna 0402 |
| 1 | CMC 0805 | TDK | ACM2012H-900-2P-T03 | Common mode choke, 115mA, 80VDC, 51mH, 3.0 Ohm, ACT1210D-510-2P-TL00 |
| 1 | GT-USB-7014D | G-Switch | GT-USB-7014D | USB TYPE C, USB 2.0, SMD, USB-C, Mid-Mount, Sink 2.1mm, CH=-0.52mm |
| 1 | SM04B-SURS-TF(LF)(SN) | JST | SM04B-SURS-TF(LF)(SN) | Connectors 1x4, SMD,P=0.8mm,Surface Mount，Right Angle |
| 1 | SM05B-SURS-TF(LF)(SN) | JST | SM05B-SURS-TF(LF)(SN) | Connector 1x5, SMD,P=0.8mm,Surface Mount，Right Angle |
| 18 | D15V0H1U2LP-7B | Diodes Incorporated | D15V0H1U2LP-7B | unidirectional TVS diode |
| 4 | SZESD7241N2T5G | onsemi | SZESD7241N2T5G | ESD bidirectional diode, 24Vrwm, X2DFN-2, SOD882 |
| 11 | BAS16LP | Diodes Incorporated | BAS16LP-7B | Switching Diode, 300mA, 1.25Vf, 75V, DFN1006 |
| 4 | ESP_Header 2x10 | Samtec | TPD | see [.100" Pitch Socket Strips, 2.54 mm Square Post Socket Strips](https://www.samtec.com/flex-stacking/standard/100-pitch-sockets/) for options |
| 1 | Orange | XingLight | XL-0201UOC | CHIP LED 0201 ORANGE SMD |
| 1 | Red | XingLight | XL-0201SURC | CHIP LED 0201 RED SMD |
| 1 | Blue | XingLight | XL-0201UBC | CHIP LED 0201 BLUE SMD |
| 1 | Green | XingLight | XL-0201UGC | CHIP LED 0201 GREEN SMD |
| 1 | 0R | YAGEO | RC0201FR-070RL | Resistor |
| 28 | 5k1 | YAGEO | AC0201FR-075K1L | -55℃~+155℃ 25V 5.1kΩ 50mW Thick Film Resistor ±1% ±200ppm/℃ 0201 |
| 8 | 100k | YAGEO | AC0201FR-07100KL | 100kΩ, 25V, 50mW, Thick Film, ±1% 0201 SMD |
| 9 | 26k1 | YAGEO | AC0201FR-0726K1L | -55℃~+155℃ 25V 26.1kΩ 50mW Thick Film Resistor ±1% ±200ppm/℃ 0201 |
| 3 | 470R | YAGEO | AC0201FR-07470RL | SMD 470R OHM 1% 1/20W 0201 |
| 16 | 33R | YAGEO | AC0201FR-0733RL | RES SMD 33 OHM 1% 1/20W 0201 |
| 2 | SPST-NO | Omron Electronics Inc | B3U-3000P(M)-B | Side Button, 12V, 0,5A,  3.0x2.5mm, Ultra Small, SPST, SMD |
| 1 | ESP32-S3-PICO-1-N8R2 | Espressif | ESP32-S3-PICO-1-N8R2 | ESP32-S3 SOC, 8MB Flash, 2MB PSRAM |
| 1 | TPS2121 | Texas Instruments | TPS2121RUXR | 2.8-V to 22-V Priority Power MUX |
| 1 | TPSM82903 | TI | TPSM82903SISR | 3-17V, 3A input, stepdown, DC/DC converter, Texas uSIP-11 |
| 1 | TCAN4550-Q1 | Texas Instruments | TCAN4550RGYRQ1 | CAN-FD controller with integrated transceiver, 5Mbps, 3.3V to 5V supply, SPI interface, VQFN-20 |
| 1 | TPS2HCS10-Q1 | Texas Instruments | PTPS2HCS10AQPWPRQ1 | Automotive, dual-channel 11.3mΩ $R_{DS_{ON}}$ smart high-side switch with I2T wire protection, low-Iq mode and SPI, HTSSOP-16 SMD |
| 11 | 74LV1T125 | Nexperia | 74LV1T125GXH | Single, $\overline{\text{OE}}$ 3-State Buffer |
| 6 | 74LVC1G14 | Nexperia | 74LVC1G14GX,125 | Inverter, Schmitt Trigger |
| 1 | 40MHz | Murata Electronics | XRCGE40M000FZA2AR0 | Four pin crystal, GND on pins 2 and 4 |
