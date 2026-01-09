# uC_Agri-PV

**Low-Power Sensor Board for Agri-Photovoltaic Research**

A custom PCB designed for environmental monitoring in agricultural photovoltaic installations. Built around the STM32U5 ultra-low-power microcontroller with LoRaWAN connectivity and multiple sensor interfaces.

![PCB Front View](uC_Agri-PV-front.png)

---

## Features

- **Ultra-Low Power Design** - Optimized for battery-powered field deployment with deep sleep support
- **LoRaWAN Connectivity** - Seeed LoRa-E5 module for long-range wireless data transmission
- **Multiple Sensor Interfaces** - ADC, I2C, OneWire, RS485 for various environmental sensors
- **Switchable Power Rails** - Individual MOSFET-controlled power for each sensor to minimize standby current
- **Local Data Storage** - MicroSD card slot for data logging
- **Flexible Power Input** - Battery, solar panel, or 5V DC input with integrated charging

---

## Hardware Specifications

| Component | Description |
|-----------|-------------|
| **MCU** | STM32U575ZIT6Q (ARM Cortex-M33, 160 MHz, ultra-low power) |
| **Wireless** | Seeed LoRa-E5 (STM32WLE5 + SX1262, LoRaWAN) |
| **WiFi/BLE** | ESP32-C6-WROOM-1-N8 (optional, MCU-controlled power) |
| **Storage** | MicroSD card slot (SDMMC 4-bit) |
| **Power** | Li-Ion battery (TP4056 charger), solar input, 5V DC |
| **RTC Backup** | SuperCap (FC0V104ZFTBR24) for timekeeping during power loss |

---

## Supported Sensors

| Sensor | Interface | Connector |
|--------|-----------|-----------|
| PAR (Photosynthetically Active Radiation) | ADC | X12 |
| UV Radiation | ADC | X17 |
| Soil Moisture | ADC | X16 |
| Soil Temperature (DS18B20) | OneWire | X15 |
| Wind Speed (Anemometer) | RS485 | X14 |
| BME680 (Temp, Humidity, Pressure, Gas) | I2C | X13 |
| Generic Analog (2 channels) | ADC | X18 |

All sensor power rails are individually switchable via GPIO-controlled MOSFETs for minimal deep sleep current consumption.

---

## Pin Definition

![Pin Definition](V2_0_Pin_definition.png)

For detailed GPIO mappings and communication protocols, see **[Hardware Pinout Documentation](HARDWARE_PINOUT.md)**.

---

## Power Management

### Power Input Selection (PWR Select)

| Position | Power Source |
|----------|--------------|
| Vdd_Bat | Lithium battery |
| Vdd_USB | USB 5V (from charger IC) |
| Vdd_Solar | Solar panel input |
| 5V_Input | External 5V DC jack |

### ESP32 Power Selection

| Jumper | Connection | Function |
|--------|------------|----------|
| 1-2 | MOSFET → Vdd | MCU-controlled (for deep sleep) |
| 2-3 | USB → Vdd | Powered only when USB connected (debug mode) |

**Note:** Do not bridge all three pins simultaneously - this can cause backfeed current through the MOSFET body diode.

---

## Connectors Overview

### Debug & Programming

| Connector | Function |
|-----------|----------|
| X20 (DEBUG_STM) | 10-pin ARM Cortex SWD |
| JTAG_ESP | ESP32-C6 programming header |
| LORA JTAG | LoRa-E5 programming header |
| UART_ESP | ESP32 UART (RXD0, TXD0) |

### GPIO Headers

| Connector | Signals |
|-----------|---------|
| X7 (GPIO STM) | General purpose STM32 GPIOs |
| X5 (GPIO STM) | Additional STM32 GPIOs |
| X4 (I2C STM) | I2C bus expansion |
| GPIO_ESP | ESP32-C6 GPIO breakout |

---

## Project Structure

```
uC_Agri-PV/
├── uC_Agri-PV.kicad_pro      # KiCad project file
├── uC_Agri-PV.kicad_sch      # Main schematic
├── uC_Agri-PV.kicad_pcb      # PCB layout
├── uC_Agri-PV.pdf            # Schematic PDF export
├── Bibliothek/               # Component libraries
├── bom/
│   └── ibom.html             # Interactive BOM
└── docs/
    ├── HARDWARE_PINOUT.md    # GPIO and communication details
    └── CHANGELOG.md          # Version history
```

---

## Documentation

- **[Hardware Pinout](HARDWARE_PINOUT.md)** - GPIO assignments, sensor data pins, communication protocols
- **[Changelog](CHANGELOG.md)** - Version history and changes between revisions
- **[Interactive BOM](bom/ibom.html)** - Component placement and bill of materials

---

## Version History

| Version | Date | Description |
|---------|------|-------------|
| v2.0 | 2026-01-08 | Complete redesign with optimized power management |
| v1.3 | 2025-12-18 | Intermediate version (never produced) |
| v1.2 | 2025 | Initial release |

See **[CHANGELOG.md](CHANGELOG.md)** for detailed changes.

---

## Getting Started

### Hardware Requirements

- ST-Link V2/V3 or J-Link debugger
- USB-C cable (for charging/ESP32 debug)
- Li-Ion battery (18650 or similar)
- Sensors as needed

### Firmware Development

The board is designed for use with:
- **STM32CubeIDE** / **STM32CubeMX** for STM32U5 firmware
- **ESP-IDF** or **Arduino** for ESP32-C6 (optional)
- **AT Commands** for LoRa-E5 module

---

## License

This hardware design is part of a master's thesis project for Agri-PV research.

---

