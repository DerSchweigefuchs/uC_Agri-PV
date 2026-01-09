# Hardware Pinout & Communication

**uC_Agri-PV v2.0** - STM32U575ZIT6Q (LQFP-144)

---

## 1. Sensors - Data Reception

### Overview: Which Sensor → Which Pin → Which Protocol

| Sensor | Data Pin STM32 | Protocol | Enable Pin | Description |
|--------|----------------|----------|------------|-------------|
| **PAR (Light Sensor)** | PB1 (Pin 44) | ADC | PB10 (Pin 66) | Analog voltage 0-3.3V via TLV9304 OpAmp |
| **UV Radiation** | PB1 (Pin 44) | ADC | PB5 (Pin 134) | Analog voltage via OpAmp |
| **Soil Moisture** | PD12 (Pin 81) | ADC | PB11 (Pin 67) | Analog voltage 0-3.3V |
| **Soil Temperature** | PA7 (Pin 42) | OneWire (USART2) | PB13 (Pin 74) | Dallas DS18B20, half-duplex |
| **Wind Speed** | PA0/PA1 (Pin 33/34) | RS485 (UART4) | PB0 (Pin 43) | Digital data via ISL83078 |
| **BME680 (Environmental)** | PC1/PC0 (Pin 27/26) | I2C3 | PB14 (Pin 75) | Temp, humidity, pressure, gas |
| **Battery Voltage** | PA6 (Pin 41) | ADC | - (always on) | Voltage divider 100k/20k |

---

## 2. Analog Sensors (ADC)

The STM32 receives analog values via ADC1:

| Signal | GPIO | Pin | ADC Channel | Sensor | Value Range |
|--------|------|-----|-------------|--------|-------------|
| `Soil_Moisture` | **PD12** | 81 | ADC1_IN7 | Soil moisture | 0-3.3V |
| `UV_Radiation` | **PB1** | 44 | ADC1_IN16 | UV/PAR via OpAmp | 0-3.3V |
| `Signal_wind_velocity` | **PB6** | 135 | ADC1 | Anemometer (analog) | 0-3.3V |
| `VBat_meas` | **PA6** | 41 | ADC1_IN11 | Battery (÷6) | 0-3.3V → 0-19.8V |

**Data Flow:** Sensor → (OpAmp) → ADC Pin → ADC1 Peripheral → DMA/Register

---

## 3. I2C Sensors (BME680)

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `I2C3_SDA` | **PC1** | 27 | Data bidirectional | STM32 ↔ BME680 |
| `I2C3_SCL` | **PC0** | 26 | Clock | STM32 → BME680 |

- **Protocol:** I2C, 100kHz or 400kHz
- **Address:** 0x76 or 0x77 (depending on SDO pin)
- **Enable:** PB14 (HIGH = power on)
- **Data Flow:** STM32 sends address+register → BME680 responds with measurement data on SDA

---

## 4. OneWire Sensor (Dallas DS18B20 Soil Temperature)

| Signal | GPIO | Pin | USART | Function |
|--------|------|-----|-------|----------|
| `USART2_ONEWIRE` | **PA7** | 42 | USART2 | Data bidirectional (half-duplex) |

- **Protocol:** OneWire via USART2 in half-duplex mode
- **Enable:** PB13 (HIGH = power on)
- **Data Flow:** STM32 sends commands → DS18B20 responds with temperature (all via PA7)

---

## 5. RS485 Sensor (Anemometer)

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `UART4_TX` | **PA0** | 33 | Transmit data | STM32 → ISL83078 → Sensor |
| `UART4_RX` | **PA1** | 34 | Receive data | Sensor → ISL83078 → STM32 |
| `DE/RE` | **PB7** | 136 | Direction control | HIGH=Transmit, LOW=Receive |

- **Protocol:** RS485 via UART4 + ISL83078 transceiver
- **Enable:** PB0 (HIGH = power on)
- **Data Flow:**
  - Transmit: PB7=HIGH → PA0 (TX) → ISL83078 → RS485 bus → Sensor
  - Receive: PB7=LOW → Sensor → RS485 bus → ISL83078 → PA1 (RX)

---

## 6. SD Card (SDMMC1)

### Data Interface (4-Bit SDIO)

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `SD_DAT0` | **PC8** | 98 | Data line 0 | STM32 ↔ SD |
| `SD_DAT1` | **PC9** | 99 | Data line 1 | STM32 ↔ SD |
| `SD_DAT2` | **PC10** | 111 | Data line 2 | STM32 ↔ SD |
| `SD_DAT3` | **PC11** | 112 | Data line 3 | STM32 ↔ SD |
| `SD_CMD` | **PD2** | 116 | Command/Response | STM32 ↔ SD |
| `SD_CLK` | **PC12** | 113 | Clock | STM32 → SD |

### Control

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `SD_EN` | **PA8** | 100 | Power supply (HIGH = on) |
| `SD_Detect` | **PC6** | 96 | Card detection (LOW = inserted) |

- **Protocol:** SDMMC 4-bit mode
- **Data Flow:** STM32 SDMMC1 peripheral ↔ PC8-PC11 (data) + PD2 (command)

---

## 7. LoRa Module (Seeed LoRa-E5)

### Primary: UART (AT Commands)

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `UART_TX` | **PA9** | 101 | Transmit data | STM32 → LoRa-E5 |
| `UART_RX` | **PA10** | 102 | Receive data | LoRa-E5 → STM32 |

- **Protocol:** USART1, 9600 baud, 8N1
- **Data Flow:** STM32 sends AT commands via PA9 → LoRa-E5 responds via PA10

### Alternative: SPI

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `MOSI_LoRa` | **PE15** | 65 | Master Out | STM32 → LoRa-E5 |
| `MISO_LoRa` | **PE14** | 64 | Master In | LoRa-E5 → STM32 |
| `SCK_LoRa` | **PE13** | 63 | Clock | STM32 → LoRa-E5 |
| `NSS_LoRa` | **PE12** | 62 | Chip Select | LOW = active |

### Alternative: I2C

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `SCL_LoRa` | **PB8** | 138 | I2C1 Clock |
| `SDA_LoRa` | **PB9** | 139 | I2C1 Data |

### Control

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `LoRa_EN` | **PD13** | 82 | Power supply (HIGH = on) |
| `NRST2` | **PD14** | 85 | Reset (LOW = reset active) |

---

## 8. ESP32-C6 (SPI Slave)

| Signal | GPIO | Pin | Function | Direction |
|--------|------|-----|----------|-----------|
| `ESP32_MOSI` | **PD4** | 118 | Master Out | STM32 → ESP32 |
| `ESP32_MISO` | **PD3** | 117 | Master In | ESP32 → STM32 |
| `ESP32_SCK` | **PD1** | 115 | Clock | STM32 → ESP32 |
| `ESP32_NSS` | **PD0** | 114 | Chip Select | LOW = active |
| `ESP32_EN` | **PD5** | 119 | Enable | HIGH = on |

- **Protocol:** SPI2, STM32 is master
- **Data Flow:** STM32 transmits via PD4 (MOSI) → ESP32 responds via PD3 (MISO)

---

## 9. USB

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `USB_D+` | **PA12** | 104 | USB Data+ |
| `USB_D-` | **PA11** | 103 | USB Data- |
| `VBUS_SENSE` | **PC7** | 97 | 5V detection (HIGH = USB connected) |

- **Protocol:** USB 2.0 Full Speed (USB_OTG_FS)

---

## 10. Debug

### SWD Programming

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `SWDIO` | **PA13** | 105 | Data bidirectional |
| `SWCLK` | **PA14** | 109 | Clock |
| `SWO` | **PB3** | 132 | Trace Output |
| `NRST` | - | 25 | Reset |

### Debug UART (Header)

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| TX | PA2 | - | Debug output |
| RX | PA3 | - | Debug input |

---

## Summary: All Enable Signals

| Signal | GPIO | Pin | Active | What is Powered |
|--------|------|-----|--------|-----------------|
| `PAR_EN` | **PB10** | 66 | HIGH | PAR light sensor + OpAmp |
| `UV_EN` | **PB5** | 134 | HIGH | UV sensor + OpAmp |
| `SoilMoist_EN` | **PB11** | 67 | HIGH | Soil moisture sensor |
| `SoilTemp_EN` | **PB13** | 74 | HIGH | Dallas DS18B20 |
| `Wind_EN` | **PB0** | 43 | HIGH | Anemometer + RS485 |
| `BME_EN` | **PB14** | 75 | HIGH | BME680 |
| `SD_EN` | **PA8** | 100 | HIGH | SD card slot |
| `LoRa_EN` | **PD13** | 82 | HIGH | LoRa-E5 module |
| `ESP32_EN` | **PD5** | 119 | HIGH | ESP32-C6 |

---

## Summary: All Data Pins by Protocol

### ADC (Analog Inputs)
| Pin | Signal | Sensor |
|-----|--------|--------|
| PA6 | VBat_meas | Battery voltage |
| PB1 | UV_Radiation | UV/PAR sensor |
| PB6 | Signal_wind_velocity | Wind (analog) |
| PD12 | Soil_Moisture | Soil moisture |

### I2C
| Bus | SCL | SDA | Device |
|-----|-----|-----|--------|
| I2C3 | PC0 | PC1 | BME680 |
| I2C1 | PB8 | PB9 | LoRa-E5 (optional) |

### SPI
| Bus | SCK | MOSI | MISO | NSS | Device |
|-----|-----|------|------|-----|--------|
| SPI1 | PE13 | PE15 | PE14 | PE12 | LoRa-E5 (optional) |
| SPI2 | PD1 | PD4 | PD3 | PD0 | ESP32-C6 |

### UART
| Bus | TX | RX | Device |
|-----|----|----|--------|
| USART1 | PA9 | PA10 | LoRa-E5 |
| USART2 | PA7 | PA7 | OneWire (DS18B20) |
| UART4 | PA0 | PA1 | RS485 (Anemometer) |

### SDMMC
| Bus | CLK | CMD | DAT0-3 | Device |
|-----|-----|-----|--------|--------|
| SDMMC1 | PC12 | PD2 | PC8-PC11 | SD Card |

---

## LEDs and Buttons

| Signal | GPIO | Pin | Function |
|--------|------|-----|----------|
| `LED1` | **PE8** | 56 | Status LED (HIGH = on) |
| `LED2` | **PE7** | 55 | Status LED (HIGH = on) |
| `User_Button` | **PC13** | 7 | Button (LOW = pressed) |
