# Power Control

This document describes how the power management on the uC_Agri-PV board works. The STM32 can control the power supply for most parts on the board. This allows to save energy by turning off unused components.

## Jumper Control

### Power Lanes

| Lanes   | Control STM32 PIN | Bridge Jumper | Comment                                       |
| ------- | ----------------- | ------------- | --------------------------------------------- |
| 3V3_VDD | -                 | -             | Always powered                                |
| +3.3V   | PB15 (3V3_EN)     | JP4           | Can be controlled by STM or bridged by jumper |

### Enable Jumper

The board has some Enable Jumper pins to measure the current for research purposes.

**How it works:**
- **Jumper closed:** Normal operation, current flows through the jumper
- **Jumper open:** You can connect a multimeter in series to measure the current consumption

For common use it is **recommended to close all Jumper pins**.

| Jumper | Name     | Necessary for using        |
| ------ | -------- | -------------------------- |
| JP1    | PWR_ESP  | ESP32                      |
| JP2    | PWR_MCU  | STM32 (Always recommended) |
| JP3    | PWR_SD   | uSD Card slot              |
| JP7    | LORA PWR | LoRa E5 module             |
| JP6    | LED PWR  | LEDs next to LoRa E5 module |


## STM32 Power Control Pins

The STM32 can control the power supply for nearly every part on the board. To activate a part it is important to check if there is a **necessary Enable Jumper**.

**Note:** All power control pins are **Active Low**. This means:
- Pin = LOW (0) → Power is ON
- Pin = HIGH (1) → Power is OFF

### On-board Power Control Pins

| Part      | STM32 PIN | Active | Comment                         |
| --------- | --------- | ------ | ------------------------------- |
| SD_EN     | PA8       | LOW    | Micro SD card slot              |
| ESP32     | PD5       | LOW    | ESP32 module                    |
| LoRa E5   | PD13      | LOW    | LoRa E5 communication module    |
| 3V3 OPV   | PG1       | LOW    | 3.3V for operational amplifiers |
| 3.3V Lane | PB15      | LOW    | General 3.3V power lane         |


### Sensors Power Control Pins

Each sensor has its own power control pin. This allows to power only the sensors you need.

| Sensor           | STM32 PIN | Active | Comment                        |
| ---------------- | --------- | ------ | ------------------------------ |
| Wind             | PB0       | LOW | Wind speed sensor              |
| UV               | PB5       | LOW | UV radiation sensor            |
| PAR              | PB10      | LOW | Photosynthetically Active Radiation |
| Soil Moisture    | PB11      | LOW | Soil moisture sensor           |
| Soil Temperature | PB13      | LOW | Soil temperature sensor        |
| BME680           | PB14      | LOW | BME680 air temperature/humidity/pressure |

## Power Select

### Board Power Supply (JP8)

The board can be powered from different sources. Use the **PWR Select** jumper (JP8) to choose your power source.

![Board Power Select](images/supply_power_select.png)

| Position | Pins  | Power Source         | Comment                              |
| -------- | ----- | -------------------- | ------------------------------------ |
| 1        | 1-2   | Vdd_Bat              | Lithium battery (X21)                |
| 2        | 3-4   | Vdd_USB              | USB 5V from charger IC               |
| 3        | 5-6   | Vdd_Solar            | Solar panel input (X22)              |
| 4        | 7-8   | 5V_Input             | External 5V DC jack (1A max)         |

**Note:** Only select ONE power source at a time.


### ESP32 Power (JP2)

The ESP32 module can be powered in two different ways. Use the **JP2** jumper to select the power mode.

![ESP32 Power Select](images/esp32_power_select.png)

#### Option A: MCU-controlled (recommended for normal operation)

![Option A](images/esp32_power_select_optA.png)

| Pins | Connection   | Function                                       |
| ---- | ------------ | ---------------------------------------------- |
| 1-2  | MOSFET → Vdd | ESP32 power is MCU-controlled (for deep sleep) |

The STM32 can turn off the ESP32 during deep sleep to save power.


#### Option B: USB powered (recommended for debugging)

![Option B](images/esp32_power_select_optB.png)

| Pins | Connection | Function                                           |
| ---- | ---------- | -------------------------------------------------- |
| 2-3  | USB → Vdd  | ESP32 powered when USB connected on J4 (debug mode) |

The ESP32 stays powered via USB independent of the STM32.

**WARNING:** Do not bridge all three pins simultaneously - this can cause backfeed current through the MOSFET body diode.
