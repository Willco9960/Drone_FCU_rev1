# Design decisions and change log

Append a dated entry when a meaningful decision is made or changed. Record the choice, why, and any uncertainty or follow-up; update outdated entries by referring to the new decision rather than silently rewriting history. The schematic remains the source of truth for what is actually placed and connected.

## 2026-10-29 — Initial documented baseline

- **Learning workflow:** William builds the schematic and PCB with incremental guidance; the assistant checks each saved step with the KiCad MCP. Direct edits are reserved for changes he explicitly requests. Use readable visual wiring diagrams when useful.
- **Aircraft:** Basic 4-motor brushless quadcopter, 4S LiPo, separate 4-in-1 ESC, target 30.5 × 30.5 mm mounting. Avoid an all-in-one ESC for the first FC design to keep high-current motor circuitry out of the beginner PCB.
- **Flight controller:** STM32F722VETx MCU and ICM-42688-P SPI IMU. Motor outputs are labeled on PA0–PA3; receiver UART labels on PB10/PB11. DShot timer/DMA support and flight firmware remain unverified. A neural-network controller is a future experiment, not an implemented feature.
- **ESC target:** AERO SELFIE 45A 4IN1-ESC-LDO, selected as a lower-cost candidate for a separate ESC. Its manufacturer manual specifies 2S–6S, 30.5 mm mounting, an eight-wire 1.0 mm-pitch cable, and no BEC. Amazon price and availability can change. J3 is an unassigned 8-pin connector; the connector family, cable pin order and mating footprint are not confirmed. Source: https://cdn.shopify.com/s/files/1/0728/7321/4171/files/45A_4IN1-ESC_Manual_Book_EN-CN.pdf?v=1745301381
- **Regulated power:** AP2112K-3.3 LDO; LM2596S-5 buck under design because the chosen ESC has no 5 V BEC. Preliminary buck values: 100 µH inductor (assumes roughly 1 A load), 100 µF/50 V input capacitor, 220 µF/25 V output capacitor, 1N5825 catch diode. Size and validate against the actual load and transient conditions before fixing the BOM or powering hardware. Source: https://www.ti.com/lit/ds/symlink/lm2596.pdf
- **Power-source isolation:** D4/D5 B5819W diodes from `BUCK_5V` and `USB_VBUS` to `+5V`, with cathodes at `+5V`. This is provisional diode ORing: voltage drops, reverse leakage, USB source limits, simultaneous-power behavior, and thermal ratings require review. Verify that every future peripheral tolerates the reduced rail voltage.
- **USB:** USB-C receptacle uses independent CC pull-downs and USBLC6-2SC6 ESD protection; U3 VBUS and J2 VBUS share `USB_VBUS`. SWD test points and a reset circuit are present.
- **Battery monitoring:** 100 kΩ / 15 kΩ divider with 100 nF filter into PC0. The battery input is not yet connected; transient protection and MCU injection when unpowered must be assessed.
- **Portability:** A project-local IMU library and model are committed, but the placed symbol and footprint reference user-global library nicknames. A fresh clone must be checked before fabrication.
- **Tool caution:** During a trial layout edit on a temporary schematic, the KiCad MCP wire-deletion operation removed unrelated junctions. Discarded that result; no such edit was applied to the project. Compare whole-design netlists and diffs when using file-modifying tools.

## 2026-10-29 — Laptop vision and command link

- **Goal:** Start with a basic flight controller and later explore a camera and vision model running on the user's M1 MacBook. The laptop may send high-level commands to the flight controller; initial bench communication is planned over the existing USB-C connection.
- **Flight control:** Retain an onboard conventional stabilization baseline and independent motor safety and command-loss behavior while exploring neural-network control. The neural controller, laptop protocol, command-loss response, camera, and wireless link are future work and are not yet implemented or verified.
- **Next constraint:** Review USB data connectivity and power behavior, and plan a wireless link for untethered operation later. Do not assume USB communications or flight safety are already validated.

## 2026-10-29 — ESC battery connection documentation

- The selected 4IN1-ESC-LDO remains the candidate. Its manufacturer specification describes the 8-pin interface as having battery positive, battery negative, four motor signals, a current signal and an unused connection; it specifies no BEC. Source: https://atta.szlcsc.com/upload/public/pdf/source/20250514/5F318BBEFA286CFC68958F31AA3EDD0F.pdf
- Plan to consider the ESC battery-positive connection as the flight controller's battery input. Confirm that the purchased ESC matches the documented model, and verify connector series, contact numbering, cable orientation and actual electrical continuity before assigning J3 pins. Input protection and current capacity remain to be designed.

## 2026-10-29 — Provisional ESC connector assumption

- The user chose to proceed assuming an 8-pin JST-SH 1.0 mm connector for J3 before obtaining the ESC. This is a provisional design assumption, not a verified connector or cable pinout. Confirm the exact mating connector, footprint numbering and cable mapping against the physical hardware before fabrication or connection; revise the schematic if needed.

## 2026-10-29 — Provisional buck input and battery sensing

- The schematic was reannotated; references in the initial baseline above are historical. The saved schematic now has D1 B260A(LS) in series from BAT_4S to BUCK_IN, and D5 SMBJ20A from BUCK_IN to ground. These are provisional candidates; confirm actual parts, footprints, current, thermal performance, surge energy and layout before hardware use.
- The battery divider input moved from BAT_4S to BUCK_IN, downstream of the series diode and TVS, to reduce exposure to reversed input and input transients. The voltage measurement is lower than actual battery voltage by D1's load-dependent forward drop; firmware accuracy and calibration need review. Transients, ADC limits and injection with the MCU unpowered remain unresolved.

## 2026-10-29 — Battery divider headroom

- R7 in the reannotated schematic was changed from 15 kΩ to 10 kΩ while R5 remains 100 kΩ. The nominal divider ratio is 1:11, giving approximately 1.53 V at PC0 for a 16.8 V BUCK_IN and 2.95 V at a sustained 32.4 V BUCK_IN, ignoring tolerances and loading. C18 affects transient response.
- ADC behavior when unpowered, tolerances, TVS pulse and layout effects, battery voltage calibration after the diode, and ADC firmware configuration remain to be verified. These calculations do not establish safe power-up or protection.

## 2026-10-30 — Battery-sense isolation for initial testing

- Added JP1, an open solder jumper, between the BAT_SENSE divider/test point and ADC_BAT on STM32 PC0. The saved netlist confirms separate nets; choose an actually open footprint. Leave it open for initial power checks. Battery sensing is unavailable while open, and bridging it does not protect PC0.
- This is a test provision, not a final verified powered/unpowered input protection solution. Resolve PC0 startup, power-down and fault behavior and decide whether additional isolation is needed before bridging JP1 or relying on battery sensing in flight.

## 2026-10-30 — Provisional receiver connector

- Added J4 as a four-pin receiver connector: pin 1 +5V (after source-OR diodes), pin 2 GND, pin 3 RC_RX and pin 4 RC_TX. TX and RX are named from the flight controller perspective; receiver TX connects to RC_RX and receiver RX to RC_TX.
- Pinout, connector and receiver remain provisional. Verify receiver supply range with the diode-reduced rail, 3.3 V UART logic compatibility, protocol, and physical pin order before assigning a footprint or connecting hardware.

## 2026-10-30 — V1 bench prototype and long-term autonomy goal

- V1 is a USB-powered bench prototype: read the IMU, stream real-time data to the programming computer, display measured orientation, and run a small onboard neural-network inference whose proposed correction is visualized separately. V1 does not drive motors or fly; motor/ESC connections are reserved for later iterations.
- Long-term goal: a 3-inch FPV-frame drone with onboard stabilization and motor safety, plus camera-based subject tracking, following and orbiting. The user intends vision processing on a remote PC or laptop and high-level command transmission to the aircraft. Required video/command links, localization, obstacle handling, delay budgets, fail-safes and flight validation are open design work.
- STM32F722VETx and ICM-42688-P remain initial candidates, not a guarantee that a particular model or future flight firmware will fit. Benchmark inference, memory use and timing before committing to flight use. A 6-axis IMU provides no absolute heading reference; yaw drift and display conventions require consideration.
- Retain SWD for primary programming and debugging and USB data for bench telemetry; investigate the exact STM32 ROM bootloader/DFU path separately. Before fabrication, verify USB-only power behavior, programming access, IMU library portability, component footprints, ERC and board layout. The battery/ESC circuitry is optional for first USB bench bring-up but must be reviewed before connecting it.

## 2026-10-30 — V1 neural-network training emphasis

- V1 primarily supports development and training of a flight-stabilization neural network. Collect timestamped IMU data and model inputs/outputs, train models on the computer, deploy candidate models to the STM32, and visualize measured orientation and a separately labeled predicted correction in real time. No motor actuation or flight in V1.
- Handheld IMU recordings alone do not contain the motor-response dynamics or correct control labels needed to train a flying controller. Plan simulated dynamics, reference-control data or later instrumented flight data, and evaluate inference timing and behavior before any flight use. The precise model, training method, sensor-fusion method, telemetry protocol, estimator and evaluation criteria remain to be specified.

## 2026-10-30 — Plan one MCU family across V1 and flight

- User wants the V1 MCU to remain the intended MCU for later flight and remote-vision autonomy, avoiding repeated processor changes. The STM32H743VIT6 is the proposed planning candidate because of its additional compute and memory over the currently placed STM32F722VETx; feasibility remains contingent on an actual NN and flight firmware timing/memory budget, power and pin mapping, packaging, availability and cost.
- No MCU replacement has been made. A change from F722 requires a deliberate schematic redesign and complete datasheet-based review of power pins, regulator configuration, boot, SWD, clocks, USB, IMU SPI, motor timer/DMA, receiver UART and future telemetry. Same package does not establish electrical or pin compatibility. Heavy vision processing remains on the remote computer; the aircraft must retain onboard stabilization and failsafes.

## 2026-10-30 — V1 planning baseline

- Plan the STM32H743VIT6 as the common V1 and future-flight MCU, subject to full design review and measured model/firmware resource budgets. The schematic still contains the F722; MCU replacement and pin mapping remain to be completed deliberately.
- V1 acceptance: USB-powered bench-only IMU acquisition and timestamped telemetry, Mac orientation viewer and recording, small model trained on the computer and deployed on the MCU, a separately labeled model recommendation and simulation-based predicted response, with inference and telemetry performance measured. Motors and flight control are not enabled in V1.
- Planning targets, not verified performance: IMU and inference 200 Hz, USB telemetry 100 Hz, viewer 30-60 Hz. Choose and benchmark a baseline model and control semantics before fixing flash/RAM and timing budgets. Six-axis yaw has no absolute heading reference.
- Reserve accessible SWD, USB, receiver, four motor outputs, and a separate future companion/telemetry UART; verify H743 power configuration and package pins, SPI, boot, USB, timer/DMA and receiver voltages. Leave the existing ESC and battery sections provisional until their hardware review; no 4S is required for V1 bring-up.

## 2026-10-30 — Training computer

- The user also has a desktop with an RTX 3080, Intel i7-12700K and 32 GB DDR5 RAM. Plan to use it for heavier neural-network training and simulation experiments, while the M1 MacBook remains available for board programming, telemetry and visualization. Actual software compatibility and GPU memory capacity are to be checked on the target machine.
- Isaac Sim/Lab may be useful for later camera simulation, but the then-current Isaac Sim 6.0.1 documentation lists RTX 4080 and 16 GB VRAM as minimum specifications. An RTX 3080 is below that published baseline; do not assume it will run reliably. PyTorch plus a lighter drone simulator remains a V1 candidate.

## 2026-10-30 — Desktop GPU details

- User reports the RTX 3080 has 10 GB dedicated GPU memory, 16 GB shared memory, and Windows driver version 32.0.15.9174. Treat shared system memory separately from dedicated VRAM when assessing simulation workloads. The then-current Isaac Sim 6.0.1 published minimum is 16 GB VRAM; this desktop is below that baseline. Confirm actual driver compatibility and runtime behavior when installing any tools.

## 2026-10-31 — Organize legacy revision

- Moved the original F722 REV_1 KiCad project, project-relative symbol/footprint/3D libraries and library tables, legacy STM32Cube configuration, source note, and existing backups into `REV_1/`. The active H743 V1 project remains in `Drone_MCU_V1/`.
- The move does not change either schematic. KiCad-exported legacy netlists before and after differ only in source path and export timestamp; V1 and legacy MCP netlists also match before and after. The legacy IMU's pre-existing global-library portability issue remains unresolved.
