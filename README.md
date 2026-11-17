# Industrial Machine Health Node

A 4-layer PCB that monitors the health of a machine (motor, pump, fan) by analysing vibration and motor current on-device. An FPGA running my own RISC-V CPU does the processing, and results go out over BLE, RS-485 (Modbus) or USB to a small software stack.

Full scope, parts, add-ons and phases are in [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md). Design decisions are logged in [DESIGN_LOG.md](DESIGN_LOG.md).

## Highlights

- 4-layer board: AMD Spartan-7 FPGA (BGA), multi-rail power from a 9 to 36 V input, USB, RS-485, CAN FD
- Runs my own 32-bit RISC-V core (see [riscv-cpu](https://github.com/rome9080/riscv-cpu))
- Vibration and motor-current analysis on the device
- Add-ons: signal integrity, design for manufacture, bare-metal C toolchain, edge ML, telemetry dashboard, CI pipeline, requirements traceability

## Status

| Phase | State |
|-------|-------|
| 0. Scope and plan | Done |
| 1. Architecture and part selection | In progress |
| 2. Schematic | Not started |
| 3. Layout | Not started |
| 4. Fabrication | Not started |
| 5. RTL, firmware, software | Not started |
| 6. Bring-up | Not started |
| 7. Testing and write-up | Not started |

## Repository layout

- `docs/PROJECT_PLAN.md`: scope, add-ons, phases, risks
- `DESIGN_LOG.md`: decision log
- `docs/archive/`: the earlier 2-layer sensor-node concept
- Hardware, RTL, firmware and software folders will be added as each phase starts
