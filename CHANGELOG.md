# Changelog

## v2.0 (2026-01-08) - Complete Redesign

**Note:** v2.0 is a complete redesign based on lessons learned from v1.2 and v1.3. Version 1.3 was never produced and served only as an intermediate step.

### Layout (completely new)

- **Complete new PCB layout** - cleaner trace routing and optimized placement
- **Crystal X602 closer to STM32** - previously too far away (issue from v1.3 resolved)
- **Project renamed** from `µC_Agri-PV` to `uC_Agri-PV` (ASCII-compatible)

### Power Management

- **DMP1045U-7 replaces IRLML6402** - new PMOS for 3V3 switching (better specifications)
- **TPS61023DRLR boost converter** - new 3.3V step-up for sensor supply
- **Separate analog ground (GNDA)** - separated from digital GND for better signal quality
- **Complete shutdown of sensor peripherals:**
  - Pull-up resistors connected to switchable 3V3 rail (no longer on permanent VDD)
  - Associated ICs (e.g., OpAmps, buffers) also on switchable supply
  - MOSFET now disconnects not only the sensor but all associated components
  - Eliminates leakage currents through pull-ups and ICs during deep sleep
- **New power rails:**
  - `3V3_OPV_EN` - Enable for OpAmp supply
  - `3V3_EN` - Enable for switchable 3.3V rail
  - `VDDA` - Separate analog supply
  - `VBat_meas` - Battery voltage measurement via ADC
  - `Vdd_11` - 1.1V reference/core voltage
- **SuperCap for RTC** - FC0V104ZFTBR24 for backup power

### STM32 Peripherals

- **TLV9304IDR OpAmp** - for signal conditioning (ADC input)
- **2.2µH inductor (L201)** - for LC filter at ADC
- **DMG1012T-13 N-MOSFET (Q203)** - for sensor switching
- **New filter capacitors:**
  - 33nF (C201) - Low-pass filter
  - 15nF (C206) - Anti-aliasing filter
  - 4.7pF (C1) - High-frequency filtering
- **ADC connection changed** - optimized signal paths to MCU

### Sensors

- **TPS61023DRLR boost** - 3.3V supply for sensors (with 1µH inductor L801)
- **IRLML6402 PMOS (Q804/Q805)** - switchable sensor supply
- **Voltage dividers for ADC:**
  - 100k/20k divider for battery measurement (R803, R808)
  - 100k pull-ups (R804, R805, R801)
- **22µF buffer capacitors (C806, C807, C812)** - decoupling
- **New sensor connectors:**
  - X12: Conn_01x04_Pin (Current Sink)
  - X13: Conn_01x07_Pin (Multi-Sensor)
  - X15, X16: Conn_01x03_Pin (Soil/Temp)

### LoRa

- **Status LED added (LED_3)** - with 1k series resistor (R19)
- **22 Ohm series resistor (R34)** - impedance matching
- **Separate ground domain (GND1)** - for better RF isolation
- **4.7µF decoupling capacitor (C12)** - module supply
- **LoRa-E5 STEP model** - 3D model for layout visualization (317990687)

### ESP32

- **New footprint** - `Library:ESP32C6WROOM1N8` instead of external library
- **Reference numbering** - U3 instead of U8
- **0R configuration resistors** - for flexible pin assignment
- **GND nets reorganized** - cleaner ground routing

### Debug & Programming

- **ARM Cortex connector corrected:** Pin 10 from JTRST → NRST (Pin 25 STM32U5)
  - Enables flashing even during deep sleep
- **Debug UART exposed** (PA2/PA3 + GND + 3V3)
- **Debug connector (X20):** 3220-10-0100-00 (10-pin ARM Cortex)
- **OneWire connector (X10):** 2x2 pin for Dallas sensors

### Component Library (new)

| Component | Type | Description |
|-----------|------|-------------|
| DMP1045U-7 | PMOS | Power switch (SOT-23) |
| NX1610SA 32.768kHz | Crystal | RTC crystal |
| FC0V104ZFTBR24 | SuperCap | RTC backup |
| TPS61023DRLR | Boost IC | 3.3V step-up |
| DMG1012T-13 | N-MOSFET | Low-side switch |
| 317990687 | LoRa-E5 | STEP 3D model |
| TP4056-18650 | Footprint | Charge controller (corrected) |

### Miscellaneous

- **BOM/iBOM updated** - interactive BOM regenerated
- **CSV export** - BOM as CSV
- **Component references** - completely renumbered

---

## v1.3 (2025-12-18) - Intermediate Version (never produced)

**Note:** This version served as a planning and development step between v1.2 and v2.0.

### Implemented (Schematic)

#### Debug & Programming
- **ARM Cortex connector:** Pin 10 from JTRST → NRST (Pin 25 STM32U5)
  - Enables flashing even during deep sleep

#### Power Management
- **Switchable 3V3 peripheral rail** for minimal deep sleep consumption
- **RTC backup supply** via SuperCap for timekeeping during brief power losses

#### Communication
- **Debug UART exposed** (PA2/PA3 + GND + 3V3) for ST-Link or external devices

#### Layout
- **Charge controller footprint/symbol corrected**

### Remaining Open (implemented in v2.0)

- Transfer schematic changes to layout
- Place crystal X602 closer to STM32

### Not Implemented (not necessary)

| Planned                             | Reason                              |
| ----------------------------------- | ----------------------------------- |
| ~~UART between STM32 ↔ ESP32~~      | Already connected via SPI           |
| ~~WakeUp pin to LoRa-E5~~           | Module wakes up via UART command    |
| ~~Separate PMOS for PAR receiver~~  | Connected to switchable 3V3 rail    |
| ~~Separate PMOS for TLV9304~~       | Connected to switchable 3V3 rail    |

---

## v1.2 (2025) - Original Version

- First complete version of the uC_Agri-PV board
- Base layout with STM32U5, ESP32-C6, LoRa-E5
- Sensors: PAR, soil moisture, temperature
- SD card slot for data logging
- Lithium battery charge controller (TP4056)
