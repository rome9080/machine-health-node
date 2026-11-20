# Sheet 5: Communications (hierarchical sheet `Comms`, comms.kicad_sch)

Status: drawn 2026-11-20. Symbols from Ultra Librarian (THVD1410D, TCAN1042VDQ1) and Espressif's KiCad library (ESP32-C3-MINI-1). Page size A3.

## What it does
Connects the board to the outside world: RS-485 (Modbus RTU), CAN FD and BLE. All signals reach the FPGA by global labels; the sheet has two hierarchical pins, `3V3` and `5V`.

## Parts
| Ref | Part | Package | Supply |
|---|---|---|---|
| U10 | THVD1410 | SOIC-8 | 3V3 |
| U11 | TCAN1042V (VDQ1) | SOIC-8 | VCC 5V, VIO 3V3 |
| U12 | ESP32-C3-MINI-1 | module | 3V3 |

## RS-485
- RE and DE tied together to `RS485_DE`; `RS485_TX` to D; `RS485_RX` from R.
- A and B each through 10 ohm to `BUS_A` and `BUS_B`. 120 ohm termination in series with a 2-pin jumper (J7) between them. 3-pin terminal J8: A, B, GND.
- 100 nF on VCC. No bias resistors: the THVD1410 has internal fail-safe biasing.
- TVS (SM712 class) is not drawn yet.

## CAN FD
- VCC on 5V with 100 nF; VIO on 3V3 with 100 nF. `CAN_TX`, `CAN_RX`, `CAN_STB`. 10k from STB to GND for normal mode by default.
- 120 ohm termination with jumper J9 between `CANH` and `CANL`. 3-pin terminal J10: CANH, CANL, GND.

## BLE
- 3V3 with 10 uF and 100 nF. EN: 10k pull-up, 1 uF to GND, `BLE_EN` from FPGA. IO9 (`BLE_BOOT`), IO8 and IO2: 10k pull-ups to 3V3.
- UART: module TXD to `BLE_RX`, module RXD from `BLE_TX`. Other IOs unconnected (no-connect flags at the end of Phase 2).
- Antenna keep-out at layout: no copper or parts under the antenna end.

## Open items
RS-485 TVS; CAN ESD protection and choke; check ESP32-C3-MINI-1 strapping and pin details against the Espressif datasheet (the datasheet could not be opened during drawing); add the ESP32 footprint library at footprint assignment; JLCPCB stock for the -N4 variant.
