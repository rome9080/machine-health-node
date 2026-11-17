# Sensor Node PCB

A wireless vibration and temperature sensor node on a custom 2-layer PCB, designed in KiCad. The board is a battery-powered node that reads an IMU and a temperature sensor and reports over Bluetooth LE, intended for simple machine condition monitoring.

The goal of this project is to show the full design-to-board process: requirements, part selection, schematic, layout, fabrication, bring-up, and measurement. Every non-obvious choice is recorded in [DESIGN_LOG.md](DESIGN_LOG.md).

## Design summary

| Block | Part |
|-------|------|
| Microcontroller | ESP32-C3-MINI-1 module (BLE, native USB) |
| Motion sensor | LSM6DSO IMU |
| Temperature sensor | TMP102 |
| Battery charger | MCP73831, charges a single-cell LiPo over USB-C |
| 3.3 V regulator | TLV75733P, chosen for low quiescent current |
| Battery | 3.7 V LiPo, about 500 mAh |

Targets: board size 40 x 30 mm, whole-board sleep current under 50 uA, at least 7 days of battery life at one reading per minute. Hand-assembled.

## Status

| Phase | State |
|-------|-------|
| 1. Plan (requirements, parts, design log) | Done |
| 2. Schematic | Not started |
| 3. Layout | Not started |
| 4. Fabricate | Not started |
| 5. Bring-up and measurements | Not started |
| 6. Documentation | Not started |

## Repository layout

- `DESIGN_LOG.md`: requirements, parts shortlist, and the decision log
- KiCad project files will be added in Phase 2

## Tools

KiCad for schematic and layout. Firmware will be a short BLE sketch for the ESP32-C3.
