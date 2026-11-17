# Wireless Vibration and Temperature Sensor Node: Design Log

Custom 2-layer PCB designed in KiCad. A battery-powered node that measures vibration (IMU) and temperature, then reports over Bluetooth LE. Intended for simple machine condition monitoring.

**Status:** Phase 1 (Plan) complete 2026-11-16. Next: Phase 2 (Schematic)
**Started:** 2026-11-16

> Figures marked "verify" are from memory or from a distributor listing. Check each against the current manufacturer datasheet before it goes into the schematic, and replace the "verify" tag with the datasheet page you used. Figures without that tag were checked against a datasheet or manufacturer product page on 2026-11-16.

---

## 1. Purpose

Show the full design-to-board process: requirements, part selection, schematic, layout, fabrication, bring-up, measurement. The board is a vehicle for demonstrating engineering decisions, so every non-obvious choice gets a log entry (section 5).

## 2. Requirements

| ID | Requirement | Target | Notes |
|----|-------------|--------|-------|
| R1 | Power source | Single-cell LiPo, 3.7 V nominal | Cell with built-in protection circuit, see D-007 |
| R2 | Charging | Over USB-C, no extra hardware | Charge current set for the cell chosen (<= 0.5C) |
| R3 | Programming | Over USB-C (native USB on MCU) | Plus UART pads as a fallback |
| R4 | Sensing: motion | 3-axis accelerometer, >= 1 kHz sample rate in bursts | Enough to see motor and fan vibration |
| R5 | Sensing: temperature | +/- 0.5 C or better | |
| R6 | Wireless | Bluetooth LE to phone or laptop | BLE only, see D-001 |
| R7 | Sleep current | < 50 uA for the whole board | Dominated by regulator quiescent current, see D-004. A 20 uA stretch target is not reachable with this class of LDO |
| R8 | Battery life | >= 7 days at one reading per minute | Verify by measurement in Phase 5 |
| R9 | Size | <= 40 x 30 mm | Two M2 mounting holes |
| R10 | Layers | 2 | Keeps fabrication cheap |
| R11 | Debug access | Test points: 3V3, GND, VBAT, VBUS, SDA, SCL, TX, RX | |
| R12 | Battery monitoring | Read battery voltage through an ADC pin | Divider must not drain the cell |
| R13 | Wake on motion | IMU interrupt wired to an MCU wake-capable pin | |

## 3. Block diagram

```
USB-C --> ESD protection --> Charger --> LiPo battery
 |                              |             |
 | (USB D+/D-)                  +-------------+--> 3.3 V regulator --> MCU (ESP32-C3 module)
 +--------------------------------------------------------------------> MCU            |
                                                                        |  I2C           |
                                                                IMU <---+--> Temp sensor
                                                                 |
                                                                 +--(INT)--> MCU wake pin
Battery --> voltage divider --> MCU ADC pin
```

## 4. Parts shortlist

| Block | Part | Alternative | Why |
|-------|------|-------------|-----|
| MCU | ESP32-C3-MINI-1 module | nRF52 module | Pre-certified antenna and native USB. No RF layout needed |
| IMU | LSM6DSO | BMI270 | Good vibration bandwidth, I2C/SPI, interrupt pin. verify |
| Temperature | TMP102 | TMP117 | TMP102 has leads, so it suits hand assembly. TMP117 is more accurate but leadless, see D-003 |
| Charger | MCP73831 (4.20 V variant), R_PROG = 5.1 kOhm | BQ24074 | Simple, one resistor sets current. No power-path, see D-002 |
| Regulator | TLV75733PDBVR (3.3 V, 1 A, SOT-23-5) | AP2112K-3.3 | Lower quiescent current than AP2112K, see D-004 |
| USB ESD | USBLC6-2SC6 | | Protects D+/D- |
| USB-C connector | 16-pin receptacle (power + USB 2.0) | | 5.1 kOhm on CC1 and CC2 |
| Battery | 3.7 V LiPo, about 500 mAh, with protection circuit | | See D-007 |
| Battery connector | JST-PH 2-pin | | Check polarity of the cell, vendors differ |

Fill in before ordering: manufacturer part number, supplier part number, package, stock, unit price. Buy from a supplier that ships to you (Digi-Key, Mouser, LCSC) and check stock the day you order.

## 5. Decision log

Format: Decision, Options, Choice, Reason, Status. Add a new entry whenever you choose between real alternatives.

### D-001: Wireless protocol
- **Options:** Bluetooth LE only, Wi-Fi, both
- **Choice:** BLE only. Wi-Fi is unused, though the ESP32-C3 hardware could do it.
- **Reason:** BLE keeps average current low. Wi-Fi association is power hungry for a battery node.
- **Status:** Decided 2026-11-16

### D-002: Charger without power-path
- **Options:** MCP73831 (simple linear charger) vs BQ24074 (power-path, runs system while charging)
- **Choice:** MCP73831
- **Reason:** Fewer parts and easier to explain. Known limitation: the load shares the battery node while charging, so the charger may not terminate if the board draws too much during charging. Acceptable for a sensor node that sleeps. Revisit in a v2.
- **Charge current:** I = 1000 V / R_PROG (verify in datasheet). 5.1 kOhm gives about 196 mA, which is about 0.4C for a 500 mAh cell. See D-007.
- **Status:** Decided 2026-11-16

### D-003: Assembly method and package constraints
- **Choice:** hand assembly (decided 2026-11-16). A soldering starter kit is already owned.
- **Constraint this creates:** use 0603 or larger passives, and keep every leaded part at 0.5 mm pitch or coarser.
- **Temperature sensor:** TMP117 (WSON-6, 2 x 2 mm, leadless) is hard by hand. Use TMP102 (SOT-563, has leads) unless hot air is available.
- **Unavoidable leadless parts:** the IMU (LGA), the ESP32-C3 module (ground pad underneath) and the USB-C receptacle (fine pitch). These need solder paste with a hot plate, hot air, or a reflow oven, not a plain iron. Confirm the kit includes paste, flux, and a hot air station or hot plate before ordering the board.
- **Status:** Decided 2026-11-16

### D-004: 3.3 V regulator and quiescent current
- **Concern:** the regulator's quiescent current (Iq) is drawn all the time, including during sleep, so it dominates the sleep budget in R7.
- **Options checked:**
  - AP2112K-3.3: 600 mA, Iq 55 uA typ per the Diodes datasheet (some distributors list 80 uA). Too high.
  - XC6220B331: 1 A. A figure under 50 uA for the B variant could not be confirmed, because the datasheet excerpt shows about 50 uA typ in high-speed mode. Excluded.
  - TLV75733P (TLV757P family, 3.3 V): 1 A, Iq 25 uA typ per the TI product page, dropout 425 mV max at 1 A, input 1.45 to 5.5 V, SOT-23-5. One retailer listing shows 40 uA typ, so confirm the table value in the datasheet.
- **Choice:** TLV75733PDBVR
- **Reason:** lowest Iq of the three options, enough current for the radio transmit peaks (the ESP32-C3 datasheet recommends >= 500 mA supply, verify), and a hand-solderable package.
- **Consequence for R7:** estimated sleep budget is about 25 uA (regulator) + about 5 uA (ESP32-C3 deep sleep, verify) + IMU and temperature sensor in low-power mode (verify). That should land near 35 to 45 uA, which meets the 50 uA target but not a 20 uA stretch. A v2 could use a low-Iq buck converter or a load switch. Confirm by measurement in Phase 5.
- **Status:** Decided 2026-11-16

### D-005: Battery polarity and protection
- **Choice:** LiPo with a built-in protection circuit and a JST-PH connector.
- **Risk:** JST-PH wiring is not standardised between sellers. Check the cell's red and black wire against the connector pinout before connecting, and mark polarity on the silkscreen.
- **Status:** Decided 2026-11-16

### D-006: Battery voltage measurement
- **Choice:** resistor divider (for example 2 x 1 MOhm) plus 100 nF to an ADC pin
- **Reason:** high resistance keeps divider leakage to about 2 uA at 4.2 V. Confirm the ADC input can tolerate this source impedance, or add a MOSFET to disconnect the divider when not sampling.
- **Status:** Proposed, finalise in Phase 2

### D-007: Battery capacity
- **Choice:** 3.7 V LiPo, about 500 mAh (400 to 600 mAh is fine), with protection circuit and a JST-PH 2-pin plug. No cell is owned yet, so buy one before Phase 5.
- **Reason:** R8 (7 days) needs an average draw below about 3 mA with a 500 mAh cell (500 mAh over 168 h), which the sleep budget in D-004 easily meets. A larger cell only makes the board bigger. 500 mAh also keeps charge current modest (D-002).
- **Status:** Decided 2026-11-16

## 6. Open questions

- ~~Q1: BLE or Wi-Fi~~ Answered: BLE only (D-001)
- ~~Q2: hand-solder or JLCPCB assembly~~ Answered: hand-solder (D-003)
- ~~Q3: budget~~ Estimate: bare boards about $10 to 15, parts about $25 to 40, battery about $10. Confirm before ordering.
- ~~Q4: LiPo capacity~~ Answered: no cell yet, buy about 500 mAh (D-007)
- ~~Q5: soldering tools~~ Answered: starter kit owned. Confirm it includes paste, flux, and hot air or a hot plate (D-003)

## 7. Phase checklist

- [x] Phase 1: Plan (requirements, parts, log started)
- [ ] Phase 2: Schematic (ERC clean, footprints verified)
- [ ] Phase 3: Layout (DRC clean, reviewed)
- [ ] Phase 4: Fabricate
- [ ] Phase 5: Bring-up and measurements
- [ ] Phase 6: Documentation and repo

## 8. Change history

| Date | Change |
|------|--------|
| 2026-11-16 | Initial requirements, block diagram, parts shortlist and decisions D-001 to D-006 |
| 2026-11-16 | D-001 decided (BLE only). D-003 decided (hand assembly), added Q5 on tools |
| 2026-11-16 | D-004 decided (TLV75733P after checking regulator Iq). Added D-007 (battery). All open questions closed. Phase 1 complete |
