# Hardware Pin-Belegung & Kommunikation

**uC_Agri-PV v2.0** - STM32U575ZIT6Q (LQFP-144)

---

## 1. Sensoren - Datenempfang

### Übersicht: Welcher Sensor → Welcher Pin → Welches Protokoll

| Sensor | Daten-Pin STM32 | Protokoll | Enable-Pin | Beschreibung |
|--------|-----------------|-----------|------------|--------------|
| **PAR (Lichtsensor)** | PB1 (Pin 44) | ADC | PB10 (Pin 66) | Analogspannung 0-3.3V über TLV9304 OpAmp |
| **UV-Strahlung** | PB1 (Pin 44) | ADC | PB5 (Pin 134) | Analogspannung über OpAmp |
| **Bodenfeuchte** | PD12 (Pin 81) | ADC | PB11 (Pin 67) | Analogspannung 0-3.3V |
| **Bodentemperatur** | PA7 (Pin 42) | OneWire (USART2) | PB13 (Pin 74) | Dallas DS18B20, Half-Duplex |
| **Windgeschwindigkeit** | PA0/PA1 (Pin 33/34) | RS485 (UART4) | PB0 (Pin 43) | Digitale Daten über ISL83078 |
| **BME680 (Umwelt)** | PC1/PC0 (Pin 27/26) | I2C3 | PB14 (Pin 75) | Temp, Feuchte, Druck, Gas |
| **Batteriespannung** | PA6 (Pin 41) | ADC | - (immer an) | Spannungsteiler 100k/20k |

---

## 2. Analoge Sensoren (ADC)

Der STM32 empfängt Analogwerte über ADC1:

| Signal | GPIO | Pin | ADC-Kanal | Sensor | Wertebereich |
|--------|------|-----|-----------|--------|--------------|
| `Soil_Moisture` | **PD12** | 81 | ADC1_IN7 | Bodenfeuchte | 0-3.3V |
| `UV_Radiation` | **PB1** | 44 | ADC1_IN16 | UV/PAR über OpAmp | 0-3.3V |
| `Signal_wind_velocity` | **PB6** | 135 | ADC1 | Anemometer (analog) | 0-3.3V |
| `VBat_meas` | **PA6** | 41 | ADC1_IN11 | Batterie (÷6) | 0-3.3V → 0-19.8V |

**Datenfluss:** Sensor → (OpAmp) → ADC-Pin → ADC1 Peripheral → DMA/Register

---

## 3. I2C Sensoren (BME680)

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `I2C3_SDA` | **PC1** | 27 | Daten bidirektional | STM32 ↔ BME680 |
| `I2C3_SCL` | **PC0** | 26 | Takt | STM32 → BME680 |

- **Protokoll:** I2C, 100kHz oder 400kHz
- **Adresse:** 0x76 oder 0x77 (je nach SDO-Pin)
- **Enable:** PB14 (HIGH = Versorgung an)
- **Datenfluss:** STM32 sendet Adresse+Register → BME680 antwortet mit Messwerten auf SDA

---

## 4. OneWire Sensor (Dallas DS18B20 Bodentemperatur)

| Signal | GPIO | Pin | USART | Funktion |
|--------|------|-----|-------|----------|
| `USART2_ONEWIRE` | **PA7** | 42 | USART2 | Daten bidirektional (Half-Duplex) |

- **Protokoll:** OneWire über USART2 im Half-Duplex-Modus
- **Enable:** PB13 (HIGH = Versorgung an)
- **Datenfluss:** STM32 sendet Befehle → DS18B20 antwortet mit Temperatur (alles über PA7)

---

## 5. RS485 Sensor (Windmesser/Anemometer)

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `UART4_TX` | **PA0** | 33 | Daten senden | STM32 → ISL83078 → Sensor |
| `UART4_RX` | **PA1** | 34 | Daten empfangen | Sensor → ISL83078 → STM32 |
| `DE/RE` | **PB7** | 136 | Richtungssteuerung | HIGH=Senden, LOW=Empfangen |

- **Protokoll:** RS485 über UART4 + ISL83078 Transceiver
- **Enable:** PB0 (HIGH = Versorgung an)
- **Datenfluss:**
  - Senden: PB7=HIGH → PA0 (TX) → ISL83078 → RS485-Bus → Sensor
  - Empfangen: PB7=LOW → Sensor → RS485-Bus → ISL83078 → PA1 (RX)

---

## 6. SD-Karte (SDMMC1)

### Daten-Interface (4-Bit SDIO)

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `SD_DAT0` | **PC8** | 98 | Datenleitung 0 | STM32 ↔ SD |
| `SD_DAT1` | **PC9** | 99 | Datenleitung 1 | STM32 ↔ SD |
| `SD_DAT2` | **PC10** | 111 | Datenleitung 2 | STM32 ↔ SD |
| `SD_DAT3` | **PC11** | 112 | Datenleitung 3 | STM32 ↔ SD |
| `SD_CMD` | **PD2** | 116 | Kommando/Antwort | STM32 ↔ SD |
| `SD_CLK` | **PC12** | 113 | Takt | STM32 → SD |

### Steuerung

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `SD_EN` | **PA8** | 100 | Versorgung (HIGH = an) |
| `SD_Detect` | **PC6** | 96 | Kartenerkennung (LOW = eingesteckt) |

- **Protokoll:** SDMMC 4-Bit Modus
- **Datenfluss:** STM32 SDMMC1-Peripheral ↔ PC8-PC11 (Daten) + PD2 (Kommando)

---

## 7. LoRa-Modul (Seeed LoRa-E5)

### Primär: UART (AT-Befehle)

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `UART_TX` | **PA9** | 101 | Daten senden | STM32 → LoRa-E5 |
| `UART_RX` | **PA10** | 102 | Daten empfangen | LoRa-E5 → STM32 |

- **Protokoll:** USART1, 9600 Baud, 8N1
- **Datenfluss:** STM32 sendet AT-Befehle über PA9 → LoRa-E5 antwortet über PA10

### Alternativ: SPI

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `MOSI_LoRa` | **PE15** | 65 | Master Out | STM32 → LoRa-E5 |
| `MISO_LoRa` | **PE14** | 64 | Master In | LoRa-E5 → STM32 |
| `SCK_LoRa` | **PE13** | 63 | Takt | STM32 → LoRa-E5 |
| `NSS_LoRa` | **PE12** | 62 | Chip Select | LOW = aktiv |

### Alternativ: I2C

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `SCL_LoRa` | **PB8** | 138 | I2C1 Takt |
| `SDA_LoRa` | **PB9** | 139 | I2C1 Daten |

### Steuerung

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `LoRa_EN` | **PD13** | 82 | Versorgung (HIGH = an) |
| `NRST2` | **PD14** | 85 | Reset (LOW = Reset aktiv) |

---

## 8. ESP32-C6 (SPI Slave)

| Signal | GPIO | Pin | Funktion | Richtung |
|--------|------|-----|----------|----------|
| `ESP32_MOSI` | **PD4** | 118 | Master Out | STM32 → ESP32 |
| `ESP32_MISO` | **PD3** | 117 | Master In | ESP32 → STM32 |
| `ESP32_SCK` | **PD1** | 115 | Takt | STM32 → ESP32 |
| `ESP32_NSS` | **PD0** | 114 | Chip Select | LOW = aktiv |
| `ESP32_EN` | **PD5** | 119 | Enable | HIGH = an |

- **Protokoll:** SPI2, STM32 ist Master
- **Datenfluss:** STM32 sendet über PD4 (MOSI) → ESP32 antwortet über PD3 (MISO)

---

## 9. USB

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `USB_D+` | **PA12** | 104 | USB Daten+ |
| `USB_D-` | **PA11** | 103 | USB Daten- |
| `VBUS_SENSE` | **PC7** | 97 | 5V Erkennung (HIGH = USB angeschlossen) |

- **Protokoll:** USB 2.0 Full Speed (USB_OTG_FS)

---

## 10. Debug

### SWD Programmierung

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `SWDIO` | **PA13** | 105 | Daten bidirektional |
| `SWCLK` | **PA14** | 109 | Takt |
| `SWO` | **PB3** | 132 | Trace Output |
| `NRST` | - | 25 | Reset |

### Debug UART (Stiftleiste)

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| TX | PA2 | - | Debug Ausgabe |
| RX | PA3 | - | Debug Eingabe |

---

## Zusammenfassung: Alle Enable-Signale

| Signal | GPIO | Pin | Aktiv | Was wird versorgt |
|--------|------|-----|-------|-------------------|
| `PAR_EN` | **PB10** | 66 | HIGH | PAR Lichtsensor + OpAmp |
| `UV_EN` | **PB5** | 134 | HIGH | UV Sensor + OpAmp |
| `SoilMoist_EN` | **PB11** | 67 | HIGH | Bodenfeuchte-Sensor |
| `SoilTemp_EN` | **PB13** | 74 | HIGH | Dallas DS18B20 |
| `Wind_EN` | **PB0** | 43 | HIGH | Anemometer + RS485 |
| `BME_EN` | **PB14** | 75 | HIGH | BME680 |
| `SD_EN` | **PA8** | 100 | HIGH | SD-Karten-Slot |
| `LoRa_EN` | **PD13** | 82 | HIGH | LoRa-E5 Modul |
| `ESP32_EN` | **PD5** | 119 | HIGH | ESP32-C6 |

---

## Zusammenfassung: Alle Datenpins nach Protokoll

### ADC (Analogeingänge)
| Pin | Signal | Sensor |
|-----|--------|--------|
| PA6 | VBat_meas | Batteriespannung |
| PB1 | UV_Radiation | UV/PAR Sensor |
| PB6 | Signal_wind_velocity | Wind (analog) |
| PD12 | Soil_Moisture | Bodenfeuchte |

### I2C
| Bus | SCL | SDA | Gerät |
|-----|-----|-----|-------|
| I2C3 | PC0 | PC1 | BME680 |
| I2C1 | PB8 | PB9 | LoRa-E5 (optional) |

### SPI
| Bus | SCK | MOSI | MISO | NSS | Gerät |
|-----|-----|------|------|-----|-------|
| SPI1 | PE13 | PE15 | PE14 | PE12 | LoRa-E5 (optional) |
| SPI2 | PD1 | PD4 | PD3 | PD0 | ESP32-C6 |

### UART
| Bus | TX | RX | Gerät |
|-----|----|----|-------|
| USART1 | PA9 | PA10 | LoRa-E5 |
| USART2 | PA7 | PA7 | OneWire (DS18B20) |
| UART4 | PA0 | PA1 | RS485 (Anemometer) |

### SDMMC
| Bus | CLK | CMD | DAT0-3 | Gerät |
|-----|-----|-----|--------|-------|
| SDMMC1 | PC12 | PD2 | PC8-PC11 | SD-Karte |

---

## LEDs und Taster

| Signal | GPIO | Pin | Funktion |
|--------|------|-----|----------|
| `LED1` | **PE8** | 56 | Status-LED (HIGH = an) |
| `LED2` | **PE7** | 55 | Status-LED (HIGH = an) |
| `User_Button` | **PC13** | 7 | Taster (LOW = gedrückt) |
