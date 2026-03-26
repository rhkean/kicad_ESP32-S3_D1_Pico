# WLED UnderGlow: Technical Specification Document

## 1. Introduction

### Project Overview
The WLED UnderGlow is an automotive-grade lighting controller designed for extreme efficiency and reliability in vehicle underbody lighting applications. The system integrates LED strip control with CAN FD communication, advanced power management, and vehicle signal processing.

### Key Features
- Dual 30x30mm 6-layer PCBs (MCU Hat and I/O Hat) stacked on QuinLED DigUno base board
- ESP32-S3-Pico-1 microcontroller with integrated CAN FD via TCAN4550-Q1
- Sophisticated power muxing with reverse polarity protection
- High-side current sensing and multi-stage sleep cycles for battery protection
- 6-channel LED output with level shifting for WS2812B/SK6812 strips
- 5-channel vehicle signal inputs (turn signals, brake, reverse, dome light)
- Global health monitoring with wired-OR fault bus

### Target Application
Automotive underglow lighting system for Toyota GR86/Subaru BRZ, with CAN connectivity for vehicle integration and diagnostics.

## 2. System Architecture

### Overall Architecture
The system consists of three stacked boards:
- **QuinLED DigUno (Base):** Provides 5V regulated power from vehicle battery (via DCM connector), 5A fuse protection
- **MCU Hat:** ESP32-S3-Pico-1 with power management, USB-C interface, and antenna
- **I/O Hat:** CAN transceiver, vehicle inputs, LED outputs, and load switches

### Grounding Strategy
- **Clean System GND (0V):** Referenced by ESP32-S3, level shifters, and CAN transceiver
- **Dirty GND:** TPS2HC08 and TPS2121 are lifted +0.65V via Schottky diode for reverse polarity protection
- **Header Isolation:** One GND pin dedicated to dirty GND (relay flyback), isolated until main board star-point
- **LED Data Returns:** Each LED data line paired with GND in twisted-pair, returned to clean GND

### Power Architecture
- Input: 12V vehicle battery via QuinLED DigUno
- Power Mux (TPS2121): Selects between USB-C (VBUS) and battery power with fast switchover (~5µs)
- Buck Converter (TPSM82903): 5V to 3.3V for MCU and digital logic
- Load Switches (TPS2HC08): Controls power to interior LEDs and exterior relay
- Current Sensing: 1.62kΩ sense resistor with ADC monitoring

## 3. Hardware Design

### MCU Hat Specifications
- **Dimensions:** 30x30mm, 6-layer PCB
- **Footprint:** QuinLED-ESP32 (D1 Mini32) compatible
- **Components:**
  - ESP32-S3-PICO-1 with 8MB Flash, 2MB PSRAM
  - 0402 chip antenna for WiFi/Bluetooth
  - TPS2121 power mux with reverse polarity protection
  - TPSM82903 3A buck converter
  - USB-C connector for power/programming
  - Status LEDs (power, activity, reverse polarity)
  - Reset and boot buttons
- **Layer Zoning (L5):** Antenna ground (top 1/5), 3.3V power pour (middle 2/5), internal power bus (bottom 2/5)

### I/O Hat Specifications
- **Dimensions:** 30x30mm, 6-layer PCB
- **CAN Interface:**
  - TCAN4550-Q1 CAN FD transceiver/controller
  - SPI communication with MCU
  - Watchdog with suicide switch capability
- **Vehicle Inputs (5 Channels):**
  - Signals: L/R turn, brake, reverse (active-high), dome (active-low)
  - Protection: 100k/26.1k voltage divider, 0.1µF filtering, 74LVC1G14 Schmitt inverters
  - Dome specific: Diode steering for current leakage prevention
- **LED Outputs (6 Channels):**
  - Level shifting: 74LV1T126 buffers (3.3V to 5V)
  - Termination: 33Ω series resistors for reflection damping
  - Data generation: ESP32 RMT peripheral with DMA
  - Grounding: Twisted-pair with clean GND return
- **Power Management:**
  - TPS2HC08 dual high-side switch
  - Separate channels for interior LEDs and exterior relay
  - Dirty GND isolation for relay flyback

### PCB Layout Guidelines
- **Stackup:** 6-layer boards with optimized routing for high-density components
- **Component Placement:** Headers on L1 for vehicle interface, electronics clustered on L6
- **Connector Requirements:** JST NSHD series for wire-to-board connections
  - 2x BM20B-NSHDZS-TFT (20-pin vertical)
  - 1x SM10B-NSHSS-TB (10-pin horizontal)
- **Header Pin Allocation:**
  - **BM20B-NSHDZS-TFT (17 pins allocated):**
    - 6 data/gnd pairs
    - 5 12V input signals
  - **BM20B-NSHDZS-TFT (14 pins allocated):**
    - relay+
    - relay- (dirty ground)
    - 2 sets 12V/12V/12V/GND/GND/GND sets for internal lighting for 3A each
  - **SM10B-NSHSS-TB:**
    - CAN_H/CAN_L
    - 4 12V
    - 4 GND
- **BOM Consolidation:** Standardized resistor values (5.1kΩ pull-ups/downs, 100k/26.1k dividers)

## 4. Firmware Design

### Microcontroller
- **Platform:** ESP32-S3-Pico-1
- **Peripherals:**
  - RMT: 6-channel LED data output
  - SPI: CAN transceiver communication
  - ADC: Current sensing
  - GPIO: Vehicle inputs, load switch control, fault monitoring
- **Sleep Modes:** Light Sleep → Deep Sleep with 4-hour timer for battery protection

### CAN Communication
- **Protocol:** CAN FD with TCAN4550
- **Features:** Bus monitoring, wake-on-CAN, watchdog reset
- **Safety:** Suicide switch for system recovery

### LED Control
- **Supported Strips:** WS2812B, SK6812, WS2814
- **Timing:** Precise RMT DMA for protocol compliance
- **Power Control:** Independent enable for interior/exterior groups

### Fault Monitoring
- **Global Health Bus:** Wired-OR interrupt on GPIO_03
- **Monitored Signals:** Load switch FLT, CAN INT
- **Current Sensing:** ADC-based load current monitoring with fault detection

## 5. Design Decisions and Constraints

### Major Design Decisions
- **Power Muxing:** TPS2121 with 0.65V ground lift for reverse polarity protection
- **Input Protection:** 74LVC1G14 as sacrificial barriers for vehicle signals
- **Dome Light Logic:** Diode-isolated pull-up to prevent current leakage
- **Global Health Bus:** Wired-OR fault monitoring system
- **BOM Consolidation:** Standardized resistor values to minimize line items

### Current Sense Specifications
- **Configuration:** 1.62kΩ sense resistor tied to system ground (0-3.3V range)
- **Range:** 5A load ≈ 2.68V typical
- **Fault Logic:** 3.3V reading indicates hardware fault (short/thermal)

### Design Constraints
- **MCU Hat GPIO:** Fixed design, alterations must be handled on I/O hat
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
- [ ] PCB routing I/O hat (including dirty GND isolation)

### Pending
- [ ] Final GPIO audit
- [ ] Final BOM consolidation and JLCPCB quote
- [ ] Firmware development and testing
  - [ ] Sleep cycles
  - [ ] CAN controller initialization
  - [ ] CAN SPI communications
  - [ ] FAULT handling
  - [ ] Current sensing of load switch
- [ ] Vehicle installation

## Appendices

### A. Code Snippets
#### Current Sensing from load switch
```c
float getInteriorLightAmps() {
  uint32_t raw = analogRead(SNS_PIN);
  
  // 1. Check for Hardware Fault Signal (Internal 4.5mA Source)
  if (raw > 3800) return -1.0; // Signal for "Chip is in Thermal/Short Protection"

  // 2. Convert ADC to Voltage (assuming 11dB attenuation / 3.1V FSR)
  float voltage = (raw / 4095.0) * 3.1;
  
  // 3. Convert Voltage to SNS Current (I = V/R)
  float i_sns = voltage / 1600.0;
  
  // 4. Convert SNS Current to Load Current (I_out = I_sns * K_sns)
  float i_load = i_sns * 3008.0;
  
  return i_load;
}
```

### B. GPIO Mapping
#### ESP32-S3-Pico-1 Proposed GPIO Mapping

| Pin | Signal Name | Function | Hardware Peripheral | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **IO_00** | **BOOT** | Strapping Pin | Input | Flash/Boot mode selector |
| **IO_01** | **$I_{SNS}$** | Analog Input | **ADC1_CH1** | 1.6kΩ current sense line |
| **IO_02** | **WLED Activity LED & CAN WD pulse** | Digital Output | GPIO | Since WLED toggles this pin on each loop interation, we can use this as our watchdog pulse to the CAN controller |
| **IO_03** | **$\overline{\text{FAULT}}$** | Digital Input | **RTC_GPIO3** | Global $\overline{\text{FLT}}$ (TCAN + Load Switch) |
| **IO_04** | **(NC)** | — | — | Unassigned |
| **IO_05** | **Sense Select** | Digital Output | GPIO | TPS2HC08 $I_{SNS}$ Channel Select |
| **IO_06** | **(NC)** | — | — | Unassigned |
| **IO_07** | **Diag Enable** | Digital Output | GPIO | TPS2HC08 Diagnostics Enable |
| **IO_08** | **(NC)** | — | — | Unassigned |
| **IO_09** | **$\overline{\text{WKRQ}}$** | Digital Input | **RTC_GPIO5** | TCAN Wake Request |
| **IO_10** | **SPI_CS** | Chip Select | **FSPICS0** | SPI2 ChipSelect |
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

The following Bill of Materials is sourced from the project root CSV file (`BOM-ESP32-S3 Pico D1.csv`).

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
| 11 | SDM1100LP-7 | Diodes Incorporated | SDM1100LP-7 | Schottky diode, filled shape |
| 4 | ESP_Header |  |  | Generic connectable mounting pin connector, double row, 02x10, top/bottom pin numbering scheme (row 1: 1...pins_per_row, row2: pins_per_row+1 ... num_pins), script generated (kicad-library-utils/schlib/autogen/connector/) |
| 1 | Orange | XingLight | XL-0201UOC | CHIP LED 0201 ORANGE SMD |
| 1 | Red | XingLight | XL-0201SURC | CHIP LED 0201 RED SMD |
| 1 | Blue | XingLight | XL-0201UBC | CHIP LED 0201 BLUE SMD |
| 1 | Green | XingLight | XL-0201UGC | Green LED |
| 1 | 0R | YAGEO | RC0201FR-070RL | Resistor |
| 28 | 5k1 | YAGEO | AC0201FR-075K1L | -55℃~+155℃ 25V 5.1kΩ 50mW Thick Film Resistor ±1% ±200ppm/℃ 0201 |
| 8 | 100k | YAGEO | AC0201FR-07100KL | 100kΩ, 25V, 50mW, Thick Film, ±1% 0201 SMD |
| 9 | 26k1 | YAGEO | AC0201FR-0726K1L | -55℃~+155℃ 25V 26.1kΩ 50mW Thick Film Resistor ±1% ±200ppm/℃ 0201 |
| 3 | 470R | YAGEO | AC0201FR-07470RL | SMD 470R OHM 1% 1/20W 0201 |
| 16 | 33R | YAGEO | AC0201FR-0733RL | RES SMD 33 OHM 1% 1/20W 0201 |
| 1 | 1k6 | YAGEO | AC0201FR-071K62L | -55℃~+155℃ 1kΩ 25V 50mW Thin Film ±0.1% ±25ppm/℃ 0201 AEC-Q200 |
| 2 | SPST-NO | Omron Electronics Inc | B3U-3000P(M)-B | Side Button, 12V, 0,5A,  3.0x2.5mm, Ultra Small, SPST, SMD |
| 1 | ESP32-S3-PICO-1-N8R2 | Espressif | ESP32-S3-PICO-1-N8R2 | ESP32-S3 SOC, 8MB Flash, 2MB PSRAM |
| 1 | TPS2121 | Texas Instruments | TPS2121RUXR | 2.8-V to 22-V Priority Power MUX |
| 1 | TPSM82903 | TI | TPSM82903SISR | 3-17V, 3A input, stepdown, DC/DC converter, Texas uSIP-11 |
| 1 | TCAN4550RGYRQ1 | Texas Instruments | TCAN4550RGYRQ1 | CAN-FD controller with integrated transceiver, 5Mbps, 3.3V to 5V supply, SPI interface, VQFN-20 |
| 1 | TPS2HC08-Q1 | Texas Instruments | TPS2HC08PQVAHRQ1 | Dual-Channel Automotive Smart High-Side Switch 9.7mΩ 7.5A, VQFN-HR-11 SMD |
| 6 | 74LV1T126 | Nexperia | 74LV1T126GXH | Buffer, 3-State |
| 6 | 74LVC1G14 | Nexperia | 74LVC1G14GX,125 | Inverter, Schmitt Trigger |
| 1 | 40MHz | Murata Electronics | XRCGE40M000FZA2AR0 | Four pin crystal, GND on pins 2 and 4 |
