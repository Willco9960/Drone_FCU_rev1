# V1 routing and electrical review — 2026-11-07

Verdict: saved routing passes KiCad's active electrical/geometry rules and schematic parity after the repairs below. No new hard wiring fault was found. This is a bench-prototype routing review, not unrestricted 480 MHz power approval or assembly release. Silkscreen dimensions were intentionally excluded from remediation.

## Changes applied

- Added five GND through vias, all 0.70 mm diameter / 0.30 mm drill: (116.000,108.925) at C13's existing ground trace; (107.050,124.475) at C6; (127.625,128.575) at C9; (120.900,135.700) at C17; (113.100,129.400) near the MCU-side IMU clock transition. Four shorten existing capacitor ground paths; the fifth connects the two reference planes near the signal transition. No signal tracks were rerouted.
- Restored schematic references USB_VBUS→TP4, 3V3→TP1, BOOT→SW2, NRST→SW1. Preserved the descriptive labels as separate silk text at the previous locations, sizes, rotations and stroke widths. Actual references moved to F.Fab. These were parity repairs, not a silkscreen-legibility redesign.
- Refilled and saved. Compared all existing track UUIDs, nets, endpoints, widths and layers: unchanged. All 214 pad UUIDs, positions, sizes, numbers and net assignments are unchanged. Component positions, rotations, values and footprint identities are unchanged.
- Final PCB SHA256: `5d5d9df668ff5c6862805d0a10dd6b63afac1c4cca6bb96d63a87378d499f9c6`. Evidence: `change_verification.json`.

## Final checks

KiCad CLI with `--schematic-parity --all-track-errors --severity-all --refill-zones` reports 14 silkscreen dimension warnings, zero schematic parity findings, and two excluded unconnected items. There are no non-excluded connectivity or copper errors. See `final_drc.json`. Earlier default MCP DRC did not expose the eight parity findings; the explicit parity run caught and resolved them.

The two exclusions are the internal U2 USBLC6-2SC6 connections: pins 1–6 for D− and 3–4 for D+. Both sides have copper terminating at the corresponding pads. ST's functional diagram shows these paired I/O connections. They are not permission to omit U2: a bare PCB has two disconnected copper islands per USB data net, joined by the assembled device. Keep the exact intended ESD part.

### USB

- Both data nets are entirely on F.Cu at 0.15 mm width, with no data vias. Raw filled-polygon sampling every 0.05 mm found In1.Cu ground underneath every sampled data-trace centerline. This is a geometry check, not field-solver or USB compliance validation.
- Total copper length, including connector branches: D+ 22.429 mm, D− 24.320 mm. Those totals are NOT an end-to-end skew measurement because Type-C has duplicated contacts.
- Pad-to-pad paths: J2.A6→U2.3 9.846 mm; J2.B6→U2.3 12.676 mm; U2.4→U1.71 9.753 mm. J2.A7→U2.1 11.953 mm; J2.B7→U2.1 9.193 mm; U2.6→U1.70 12.367 mm. Package-internal lengths are not included. Depending on mating orientation, corresponding PCB path differences are approximately 4.721 mm or 0.869 mm. At full-speed USB these millimetre-scale differences alone do not justify adding meanders; an estimated 5 mm FR4 delay is only tens of picoseconds. This is engineering judgment, not a measured eye diagram.
- The saved stackup has 0.0994 mm outer prepreg and 35 µm outer copper. A manufacturer-confirmed 90-ohm differential geometry remains open; the analyzer's single-ended estimates do not prove differential impedance. Do not assume a generic four-layer order reproduces the saved stackup.
- PA9/VBUS sense is unused and there is no external crystal. Application firmware must configure USB appropriately for this bus-powered design and provide a valid 48 MHz USB clock/HSI48 synchronization. SWD remains primary recovery; no ROM-DFU success claim is made.

### Power, MCU, reset and debug

- KiCad netlist agrees with H743 LQFP100 power, VCAP, USB, SWD, reset and boot pin numbering in DS12110 figure 5. VDD 11/27/50/75/100, VBAT 6, VREF+ 20 and VDDA 21 connect to +3V3; VSS 10/26/49/74/99 and VSSA 19 connect to GND.
- VCAP pins 48/73 share C10/C11's separate rail, not +3V3. Each has a nearby 2.2 µF capacitor: routed pin-to-cap lengths 2.250 mm and 2.517 mm. Actual BOM capacitance/ESR over bias and temperature still needs qualification.
- Main USB_VBUS copper is 0.4 mm; +3V3 uses 0.25/0.4 mm. No obvious thin high-current bottleneck was found for the intended bench load. This is not an IPC thermal qualification of every branch.
- AP2112 pins match its SOT25 table: 1 VIN, 2 GND, 3 EN tied to VIN, 4 NC, 5 VOUT. C14/C13 provide nominal 1 µF input/output capacitance. The added C13 via shortens the return path.
- The major remaining electrical limitation is AP2112 heat. At 5 V input, 3.3 V output and 300 mA, estimated dissipation is 0.51 W; the datasheet's 184 °C/W reference gives about 94 °C rise. At 150 mA the estimate is 0.255 W / 47 °C rise. Actual board thermal resistance and workload are unmeasured. The thermal analyzer assumed only 0.165 W and cannot clear full-load operation. Begin at conservative clocks/load and measure; redesign the power supply if sustained workload exceeds the measured budget. A 600 mA rating does not establish usable dissipation on this PCB.
- NRST has 100 nF and a reset button; H743 has an internal permanent pull-up, so the analyzer's missing-external-pull-up warning is not a demonstrated defect. BOOT0 has 10 kΩ pull-down and a button to 3V3. SWDIO/CLK go to PA13/PA14, with NRST and GND at J1. The 2.54 mm header and probe power behavior still need cable verification; do not connect a probe's power output as if it were VTref.

### IMU and ground

- U5 pin assignment agrees with DS-000350 single-interface SPI mode: 1 MISO, 4 INT1, 5 VDDIO, 6 GND, 8 VDD, 12 CS, 13 SCK, 14 MOSI; 2/3/7 grounded; 10/11 open. Pin 9 is grounded for unused FSYNC: firmware must not enable it as a driven interrupt output.
- Both inner copper layers contain GND fills, with one filled region each. The explicit 3.5 × 3.0 mm zone keepout spans B.Cu/In1.Cu/In2.Cu underneath U5. Top-layer routing has outward pad escapes and no cross-body signal route visible in the copper plot. The keepout excludes pours, not future tracks/vias; preserve this constraint during later editing.
- Short reference gaps occur at signal antipads and the intentional IMU cutout. These should not be called globally split ground planes. Keep SPI slew rate and clock conservative for bring-up; the second clock transition remains farther from a ground stitching via because of local clearances. No congested reroute was introduced solely to meet an arbitrary 1 mm heuristic.
- C13/C6/C9/C17 proximity-to-ground-via warnings disappeared in the rerun after the four local return fixes.

## Analyzer false positives and limitations

Ran schematic, full PCB, cross-domain, thermal, EMC and Gerber analyzers, plus authoritative KiCad XML netlist/DRC and raw pcbnew geometry checks. Compared current findings against project notes and the pre-fix snapshot.

- The cross-domain tool incorrectly treats routed +3V3 islands as a split reference plane; this board's reference planes are GND.
- EMC treated copper-layer type `signal` as evidence that the inner layers carry signal traces; raw board/plots show ground planes there. Its opposite-outer-layer return heuristic also overstates coverage gaps. Neither its score nor thermal score is a release criterion.
- Coarse courtyard bounding boxes flag capacitor/MCU overlaps that KiCad's actual courtyard check does not report. Signals at 0.15 mm are not inherently undersized merely because a generic heuristic prefers 0.20 mm.
- Gerber analyzer flags different copper extents as registration errors; differing extents are expected for traces versus pours. It found no missing required copper layers or drill problems. This run did not independently complete a rendered Gerber-viewer registration review.
- Schematic ERC reports 17 embedded capacitor-symbol/library differences, a VCAP power-output conflict and the grounded IMU bidirectional-pin warning. These are not fixed by blindly replacing libraries or disconnecting pins. The capacitor library differences remain a documentation/library-maintenance item; pin-level mappings are preserved.
- Five project DRC categories remain ignored: missing courtyard, off-center via endpoint, tuning geometry, footprint filters and footprint type. No new suppression was added. This run did not re-enable those categories; the main active geometric checks and explicit parity were run.
- No SPICE simulator was found on PATH or in the checked common vendor directories; no SPICE simulation was run. No extracted datasheet library/MPN-enriched BOM is present; automated electrical findings are consistency checks. Manual manufacturer-document checks above provide targeted evidence only. Lifecycle/stock audit, exact passive qualification, connector mating drawing, screw-head clearance, final BOM/CPL orientations and assembler approval remain outside this routing completion.
- No measured hardware, firmware timing, EMI, eye diagram or load-transient test has been performed. Full-load thermal and final manufacturing stackup remain open.

## Files and tool recovery

Before mutation, `before_fixes.kicad_pcb` was saved. Initial MCP via insertion failed and unexpectedly switched backend. Four incomplete trace additions were removed by UUID; the complete PCB hash then matched the pre-fix backup exactly. After the user saved and closed the editor, explicit-unit SWIG via insertion succeeded. All final edits were made through MCP, with the final comparison above.

`gerbers_after/` contains fresh copper, masks, paste, edge and drill review outputs reflecting the final board. It intentionally omits silk because this review excludes it; it is NOT a complete order ZIP. Older fabrication ZIPs predate these fixes and must not be ordered as the final revision.

## Manufacturer evidence

- [ST H743 DS12110 Rev 11](https://www.st.com/resource/en/datasheet/stm32h743zi.pdf): same family datasheet, LQFP100 figure 5; NRST section; supply and VCAP requirements. The VI URL timed out; the ZI URL serves the family document covering V packages.
- [Diodes AP2112](https://www.diodes.com/assets/Datasheets/AP2112.pdf): SOT25 pin table, nominal capacitors, thermal resistance.
- [ST USBLC6-2](https://www.st.com/resource/en/datasheet/usblc6-2.pdf): functional diagram, paired I/O terminals.
- [TDK DS-000350 Rev 1.1, distributor-hosted manufacturer PDF](https://datasheet.lcsc.com/datasheet/pdf/01f1a12efe333b71350ad9221ae6b099.pdf?productCode=C41432121): pp. 20–23 pin modes and application circuit.

Next action: reopen the saved PCB. Before an order, close the power-budget and manufacturer stackup/BOM/assembly checks; regenerate a complete fabrication package from this revision.
