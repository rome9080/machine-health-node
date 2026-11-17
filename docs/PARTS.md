# Parts Shortlist (Phase 1)

Status key: **Chosen** = picked after checking manufacturer or distributor pages on 2026-11-16. **Candidate** = proposed, details still to verify in the datasheet before the schematic uses it.

| Block | Part | What was checked | Status |
|-------|------|------------------|--------|
| FPGA | AMD Spartan-7 XC7S25-1CSGA225C | 225-ball CSGA, 13 x 13 mm, 150 I/O, 23,360 logic cells, Active. See D-004 in the design log | Chosen |
| Vibration sensor | ST LSM6DSO | LGA-14L 2.5 x 3 x 0.83 mm, accelerometer +/-2/4/8/16 g, accelerometer output rate up to 6.66 kHz, SPI and I2C, -40 to +85 C, Active | Chosen |
| Current ADC | TI ADS131M04 | 4 simultaneously sampling differential inputs, 24-bit, programmable gain up to 128, TSSOP-20 or WQFN-20, -40 to +125 C, up to 64 kSPS in the current datasheet (older revision said 32 kSPS, check the revision) | Chosen (3 CT channels plus 1 spare) |
| Input buck converter | TI LMR36015 | 4.2 to 60 V input with transient tolerance to 66 V, 1.5 A, 12-pin VQFN-HR 2 x 3 mm, power-good and enable pins, about 90 percent efficiency for 24 V to 5 V at 1 A, quiescent current about 26 uA | Chosen, exact variant to pick |
| FPGA rails (1.0 V, 1.8 V, 3.3 V) | Three step-down converters from the 5 V rail, sequenced | TI reference design TIDA-050000 uses TPS65023, TPS568215 and TPS62067. TPS62067 class (3 MHz, 2 A) is the candidate for each rail | Candidate, verify enable and power-good pins and input range |
| RS-485 | TI THVD1410 (or THVD1450) | THVD14xx family: 3 to 5.5 V supply, +/-18 kV IEC ESD, +/-15 V common-mode range, half duplex, -40 to +125 C, SOIC-8, VSSOP-8 or VSON-8. THVD1410 is the low-EMI 500 kbps part | Chosen family, verify which variant is 500 kbps |
| CAN FD | TI TCAN1042GV family | 4.5 to 5.5 V supply, CAN FD at 2 Mbps (G variants 5 Mbps), V variants have a VIO pin for 3.3 V logic, bus fault protection +/-58 V, -55 to +125 C, SOIC-8 or VSON-8, Active. CAN controller is written in RTL on the FPGA | Chosen family, pick exact orderable part |
| Wireless | Espressif ESP32-C3-MINI-1 | Not re-checked | Candidate |
| Temperature | TI TMP117 | Not re-checked. A service assembles the board, so the small leadless package is no longer a problem | Candidate |
| FPGA configuration flash | 64 Mbit QSPI flash (for example W25Q64) | Not checked. Confirm the bitstream size fits | Candidate |
| Clock | 100 MHz oscillator | Not checked | Candidate |
| USB and programming | USB-C with onboard USB-JTAG and UART bridge (FT2232H class) | Not checked. See open decision O-1 | Candidate |
| Current sensors | Split-core CT clamps with voltage output, burden sized for fan current | Not selected | To pick |

## Notes

- Power flow: 9 to 36 V input, then LMR36015 to a 5 V rail, then three converters for the FPGA's 1.0 V core, 1.8 V auxiliary and 3.3 V I/O rails, enabled in sequence. The 5 V rail also feeds the CAN transceiver.
- The earlier LiPo battery, charger and low-power regulator ideas from the first concept no longer apply. The board is powered from the 9 to 36 V input.
- Demo machine: an AC box or pedestal fan with a deliberately added imbalance. See D-008.
