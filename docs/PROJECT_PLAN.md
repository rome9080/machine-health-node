# Project Plan: Industrial Machine Health Node

A 4-layer board that watches a machine (motor, pump, fan, compressor) and flags trouble before it fails. An FPGA running my own RISC-V CPU analyses vibration and motor current in real time. Results go out over BLE, RS-485 (Modbus) or USB to a small software stack.

This replaces the earlier 2-layer sensor-node idea (archived in `docs/archive/`). That board was a good first exercise but too small to stand out.

## The problem

Unplanned machine downtime is expensive, and most failures (worn bearings, imbalance, misalignment, electrical faults) show up in vibration and motor current well before the machine stops. Condition monitoring catches this early. The aim is a compact, industrial-style node that does the analysis on the device instead of streaming raw data.

## Core hardware (candidates, verify every part and package against its datasheet)

| Block | Candidate | Notes |
|-------|-----------|-------|
| FPGA | AMD Spartan-7 XC7S25, CSGA225 (0.8 mm BGA) | Fallback: Lattice iCE40UP5K in QFN-48 if BGA assembly is not realistic |
| Configuration | QSPI flash plus JTAG header | |
| FPGA rails | 1.0 V core, 1.8 V aux, 3.3 V I/O | Needs sequencing, check the datasheet |
| Input power | 9 to 36 V DC | Reverse-polarity protection, TVS, buck converter |
| Vibration | Digital 3-axis accelerometer over SPI, at least 4 kHz output rate | Candidates LSM6DSO, ADXL355 |
| Motor current | 3 clamp-on CT inputs into a 16-bit SPI ADC | Board never touches mains, only the isolated CT output |
| Temperature | Digital temperature sensor | |
| Wireless | ESP32-C3-MINI-1 as a BLE co-processor | |
| Buses | RS-485 (Modbus RTU), CAN FD transceiver, USB-C | |
| Debug | JTAG, UART, labelled test points | |

**Why 4 layers:** the BGA needs inner layers to escape, three supply rails want solid planes, and USB and the sensor buses need clean return paths. Stackup: signal / ground / power / signal.

**Safety rule:** the board is never connected to mains voltage. Current is measured only through split-core CT clamps.

## Add-ons

Each add-on is a self-contained deliverable that also speaks to a group of employers. Hardware add-ons must be in the first board revision, so they are decided now. Software add-ons can grow after.

| ID | Add-on | Deliverable | Who it speaks to |
|----|--------|-------------|------------------|
| H1 | My RISC-V core on the FPGA | Core running on Spartan-7, verified with Verilator and cocotb against Spike, timing report | AMD, Intel, Qualcomm, NVIDIA, Tenstorrent |
| H2 | Industrial power input and rail sequencing | 9-36 V input stage, multi-rail supply, scope captures of start-up, ripple and efficiency | AMD (power design), Siemens, Schneider, Eaton, Honeywell |
| H3 | Motor-current and power-quality channel | CT inputs, ADC, RMS and harmonic analysis in RTL | Hydro One, OPG, Bruce Power, GE Vernova, Westinghouse |
| H4 | Industrial and vehicle buses | RS-485 Modbus RTU and CAN FD transceiver, tested against a master | Siemens, Schneider, Magna, GM, Tesla |
| H5 | Signal integrity and layout rigor | Stackup, impedance calculations, return-path notes for USB and high-speed lines | AMD, NVIDIA, Qualcomm, Ciena |
| H6 | Design for manufacture and test | Test points, JTAG access, BOM costing, fabrication and assembly outputs, bring-up checklist | Tenstorrent, Celestica, L3Harris, Lockheed Martin |
| S1 | Bare-metal C toolchain for my core | Startup code, linker script, C firmware built with the RISC-V GCC toolchain | Intel compiler teams, Google, systems roles |
| S2 | FFT and feature extraction | RTL FFT block plus a Python golden model that the RTL is checked against | All hardware and software roles |
| S3 | Edge ML anomaly detection | Train a small model on collected data, quantise to int8, run inference on the core or ESP32-C3 | Cohere, NVIDIA, AMD, Google |
| S4 | Telemetry backend and dashboard | Gateway, REST or MQTT API, time-series storage, web dashboard, Docker, tests | Shopify, RBC, TD, Amazon, DoorDash, GM |
| S5 | CI/CD pipeline | GitHub Actions running KiCad ERC and DRC, Verilator regression, and the firmware build on every push | All software roles |
| P1 | Requirements traceability and test report | Matrix linking each requirement to its design item and test, with measured results | Hitachi Rail, Lockheed Martin, L3Harris, nuclear and utilities |

**Tier 1 (do these):** H1, H2, H5, H6, S1, S5, P1. **Tier 2 (as time allows):** H3, H4, S2, S3, S4. The hardware in H3 and H4 still goes on the first board so it is available later.

## Phases (rough estimate, about 150 to 170 hours, 8 to 12 weeks part time)

1. Architecture and part selection, about 10 h
2. Schematic, about 25 h
3. Layout, about 35 h
4. Review and fabrication, about 10 h plus waiting
5. RTL, firmware and software developed in simulation while the board is being made, about 40 h
6. Bring-up, about 20 h
7. Testing, measurements and write-up, about 20 h

## Risks

- **Assembly:** BGA and QFN parts are not realistic with an iron. Options are hot-air or hot-plate reflow with a stencil, paying a service to assemble the hard parts, or the iCE40 fallback.
- **Cost:** roughly $150 to $300 all-in with assembly (estimate, to be verified before buying).
- **Scope:** too many add-ons can sink the project. Tier 1 comes first, and every add-on has to produce something measurable.

## Open questions

- Which exact companies is Roman applying to? The add-on list above targets common employers found in job postings and should be retargeted to the real list.
- Assembly route and budget.

## Sources

Employer examples come from job postings seen in November 2026:
- AMD Markham hardware design co-op: https://simplify.jobs/p/a130716c-fb2f-4134-9ac5-6e93002f40e0/Hardware-Design-Engineer-InternCo-op
- AMD hardware engineering posting (board design, signal integrity, power design): https://careerbeacon.com/en/job-15/1516284/amd/hardware-engineering-intern-co-op-undergrad-summer-2025-onsite-hybrid/markham-on
- Hydro One co-ops: https://app.energyhire.com/job-board/university-co-op-student-stations-engineering-electrical-12-months-fall-2025-toronto-on-bda23136-a458-478c-bd2a-5e6e83349039
- Hitachi hardware test engineering intern: https://www.iagora.com/work/en/offer/internship-canada-it/6462084
- Westinghouse engineering intern: https://careers.westinghousenuclear.com/job/Kitchener-Intern%2C-Engineering-ON-N2P-0A4/1421192300/
- Tenstorrent manufacturing co-op: https://job-boards.greenhouse.io/tenstorrentuniversity/jobs/5162150007
