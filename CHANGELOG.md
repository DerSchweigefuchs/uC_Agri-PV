# Changelog

## v2.0 (2026-01-08) - Komplettes Neudesign

**Hinweis:** v2.0 ist ein komplettes Neudesign basierend auf den Erkenntnissen aus v1.2 und v1.3. Version 1.3 wurde nie produziert, sondern diente nur als Zwischenschritt.

### Layout (komplett neu)

- **Komplettes neues PCB-Layout** - sauberere Leiterbahnführung und optimierte Platzierung
- **Quarz X602 näher an STM32** - vorher zu weit entfernt (Issue aus v1.3 behoben)
- **Projekt umbenannt** von `µC_Agri-PV` zu `uC_Agri-PV` (ASCII-kompatibel)

### Power Management

- **DMP1045U-7 ersetzt IRLML6402** - neuer PMOS für 3V3-Schaltung (bessere Spezifikationen)
- **TPS61023DRLR Boost-Konverter** - neuer 3.3V Step-Up für Sensorversorgung
- **Separate analoge Masse (GNDA)** - getrennt von digitalem GND für bessere Signalqualität
- **Vollständige Abschaltung der Sensor-Peripherie:**
  - Pull-up Widerstände an schaltbare 3V3-Schiene gehängt (nicht mehr an permanente VDD)
  - Zugehörige ICs (z.B. OpAmps, Buffer) ebenfalls an schaltbare Versorgung
  - MOSFET trennt somit nicht nur Sensor, sondern alle zugehörigen Komponenten
  - → Eliminiert Leckströme über Pull-ups und ICs im Deep Sleep
- **Neue Power-Rails:**
  - `3V3_OPV_EN` - Enable für OPV-Versorgung
  - `3V3_EN` - Enable für schaltbare 3.3V-Schiene
  - `VDDA` - Separate analoge Versorgung
  - `VBat_meas` - Batteriespannungsmessung via ADC
  - `Vdd_11` - 1.1V Referenz/Core-Spannung
- **SuperCap für RTC** - FC0V104ZFTBR24 für Backup-Versorgung

### STM32 Peripherie

- **TLV9304IDR OpAmp** - für Signalaufbereitung (ADC-Eingang)
- **2.2µH Induktor (L201)** - für LC-Filter am ADC
- **DMG1012T-13 N-MOSFET (Q203)** - für Sensor-Schaltung
- **Neue Filterkondensatoren:**
  - 33nF (C201) - Tiefpassfilter
  - 15nF (C206) - Anti-Aliasing Filter
  - 4.7pF (C1) - Hochfrequenz-Filterung
- **ADC-Anbindung geändert** - optimierte Signalwege zum MCU

### Sensoren

- **TPS61023DRLR Boost** - 3.3V Versorgung für Sensoren (mit 1µH Induktor L801)
- **IRLML6402 PMOS (Q804/Q805)** - schaltbare Sensorversorgung
- **Spannungsteiler für ADC:**
  - 100k/20k Teiler für Batteriemessung (R803, R808)
  - 100k Pull-ups (R804, R805, R801)
- **22µF Pufferkondensatoren (C806, C807, C812)** - Entkopplung
- **Neue Sensor-Konnektoren:**
  - X12: Conn_01x04_Pin (Current Sink)
  - X13: Conn_01x07_Pin (Multi-Sensor)
  - X15, X16: Conn_01x03_Pin (Soil/Temp)

### LoRa

- **Status-LED hinzugefügt (LED_3)** - mit 1k Vorwiderstand (R19)
- **22 Ohm Serienwiderstand (R34)** - Impedanzanpassung
- **Separate Ground-Domain (GND1)** - für bessere HF-Isolation
- **4.7µF Entkopplungskondensator (C12)** - Modul-Versorgung
- **LoRa-E5 STEP-Modell** - 3D-Modell für Layout-Visualisierung (317990687)

### ESP32

- **Neuer Footprint** - `Bibliothek:ESP32C6WROOM1N8` statt externer Library
- **Referenz-Nummerierung** - U3 statt U8
- **0R Konfigurationswiderstände** - für flexible Pin-Belegung
- **GND-Netze reorganisiert** - sauberere Masseführung

### Debug & Programmierung

- **ARM-Cortex-Buchse korrigiert:** Pin 10 von JTRST → NRST (Pin 25 STM32U5)
  - Ermöglicht Flashvorgang auch im Deep Sleep
- **Debug-UART herausgeführt** (PA2/PA3 + GND + 3V3)
- **Debug-Konnektor (X20):** 3220-10-0100-00 (10-Pin ARM Cortex)
- **OneWire-Konnektor (X10):** 2x2 Pin für Dallas-Sensoren

### Komponenten-Bibliothek (neu)

| Komponente | Typ | Beschreibung |
|------------|-----|--------------|
| DMP1045U-7 | PMOS | Power-Switch (SOT-23) |
| NX1610SA 32.768kHz | Quarz | RTC-Quarz |
| FC0V104ZFTBR24 | SuperCap | RTC-Backup |
| TPS61023DRLR | Boost IC | 3.3V Step-Up |
| DMG1012T-13 | N-MOSFET | Low-Side Switch |
| 317990687 | LoRa-E5 | STEP-3D-Modell |
| TP4056-18650 | Footprint | Ladecontroller (korrigiert) |

### Sonstiges

- **BOM/iBOM aktualisiert** - interaktive Stückliste regeneriert
- **CSV-Export** - Stückliste als CSV
- **Bauteil-Referenzen** - komplett neu nummeriert

---

## v1.3 (2025-12-18) - Zwischenversion (nie produziert)

**Hinweis:** Diese Version diente als Planungs- und Entwicklungsschritt zwischen v1.2 und v2.0.

### Umgesetzt (Schematic)

#### Debug & Programmierung
- **ARM-Cortex-Buchse:** Pin 10 von JTRST → NRST (Pin 25 STM32U5)
  - Ermöglicht Flashvorgang auch im Deep Sleep

#### Power Management
- **Schaltbare 3V3-Peripherie-Schiene** für minimalen Deep Sleep Verbrauch
- **RTC Backup-Versorgung** via SuperCap für Uhrzeiterhalt bei kurzen Stromverlusten

#### Kommunikation
- **Debug-UART herausgeführt** (PA2/PA3 + GND + 3V3) für ST-Link oder externe Geräte

#### Layout
- **Ladecontroller Footprint/Symbol korrigiert**

### Offen geblieben (in v2.0 umgesetzt)

- Schematic-Änderungen ins Layout übernehmen
- Quarz X602 näher an STM32 platzieren

### Nicht umgesetzt (nicht nötig)

| Geplant                             | Begründung                        |
| ----------------------------------- | --------------------------------- |
| ~~UART zwischen STM32 ↔ ESP32~~     | Bereits über SPI verbunden        |
| ~~WakeUp-Pin zum LoRa-E5~~          | Modul wacht durch UART-Befehl auf |
| ~~Separater PMOS für PAR Receiver~~ | Hängt an schaltbarer 3V3-Schiene  |
| ~~Separater PMOS für TLV9304~~      | Hängt an schaltbarer 3V3-Schiene  |

---

## v1.2 (2025) - Ursprüngliche Version

- Erste vollständige Version des uC_Agri-PV Boards
- Basis-Layout mit STM32U5, ESP32-C6, LoRa-E5
- Sensoren: PAR, Bodenfeuchte, Temperatur
- SD-Karten-Slot für Datenlogging
- Lithium-Batterie Ladecontroller (TP4056)
