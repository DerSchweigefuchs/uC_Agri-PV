## Änderungen in v1.3

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

---

### Noch offen (Layout)

- [ ] Schematic-Änderungen ins Layout übernehmen
- [ ] **Quarz X602** näher an STM32 platzieren (aktuell zu weit entfernt)

---

### Nicht umgesetzt (nicht nötig)

| Geplant | Begründung |
|---------|------------|
| ~~UART zwischen STM32 ↔ ESP32~~ | Bereits über SPI verbunden |
| ~~WakeUp-Pin zum LoRa-E5~~ | Modul wacht durch UART-Befehl auf |
| ~~Separater PMOS für PAR Receiver~~ | Hängt an schaltbarer 3V3-Schiene |
| ~~Separater PMOS für TLV9304~~ | Hängt an schaltbarer 3V3-Schiene |