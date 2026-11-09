# Design decisions and change log

Append a dated entry when a meaningful decision is made or changed. Record the choice, why, and any uncertainty or follow-up; update outdated entries by referring to the new decision rather than silently rewriting history. The schematic remains the source of truth for what is actually placed and connected.

## 2026-11-03 — Onboard person tracking and orbit target

- William wants the future aircraft to run stabilization, person detection, tracking of a selected person and follow/orbit behavior onboard. The laptop is intended to display streamed camera video, let him select the subject, and request behavior or send supervisory commands. This supersedes earlier plans for laptop-based continuous vision/control. Training can still take place on the PC.
- Automatic obstacle avoidance is out of scope for now. Target identity through occlusions/crossings, person loss, camera-based distance/relative-position estimation, radio/video loss, manual override and safety remain open design and test requirements; detecting a person alone does not solve tracking or reliable following. The 100 m open-area direct laptop-link target remains provisional.
- A compact MCU with integrated wireless and AI support remains preferred, but no single-chip solution has been selected. Benchmark the actual person detector and tracker, camera acquisition, video compression/streaming, stabilization workload, motor output timing, memory and power together. V1 remains the existing H743 USB-powered bench prototype with no camera, motors or flight capability.

## 2026-11-03 — Future laptop link range target

- William selected a preliminary target of up to 100 m between the future drone and laptop in an open area, using the laptop's built-in Wi-Fi if feasible and no internet service. This is a design/test target, not an ESP32 range claim or guarantee.
- Verify telemetry, commands and intended camera video together with the selected laptop, antenna, airframe, orientations, software, radio settings and environmental conditions; record bidirectional packet loss, high-percentile/maximum latency, outages, signal, video bitrate/quality and command age. Define quantitative acceptance thresholds before selecting hardware, test first without propellers and maintain a validated operational margin inside the observed limit.
- Link-loss behavior, independent manual control and motor safety, video architecture and any host fallback remain future design work; V1 has no wireless or flight capability. Built-in laptop antenna, local interference, carbon structure and limited line of sight may constrain the target. The external-aircraft-antenna preference and Arduino tooling preference remain in force.

## 2026-11-03 — Antenna and firmware workflow preferences

- For the future flight board William prefers an external antenna over a PCB trace antenna and likes Arduino IDE and Arduino libraries for ESP development. These are preferences, not a selected MCU, antenna, radio architecture or design change to V1. ESP32-S3-MINI-1U/WROOM-1U modules offer external antenna connectors but need evaluation for usable GPIO, size, cost and assembly; a bare chip and connector require matching and RF design. Validate the antenna, coax and mounting around the frame and RF coexistence.
- Include a realistic provisional flight-board I/O budget (four ESC signals, IMU SPI and interrupt, optional RX UART, battery ADC, status LED and programming/debug access), allowing for memory, boot and debug pins; camera, ESC telemetry, GPS and other features could increase it. Verify actual pin allocation and motor-output timing concurrently with radio activity. Arduino libraries and examples are useful, but ESP-IDF integration and measured worst-case control timing may be needed.

## 2026-11-03 — Future compact integrated wireless MCU preference

- William wants a smaller, reasonably low-power and low-cost MCU in a later flight-oriented revision, preferably a non-BGA package with onboard wireless and some AI acceleration, to avoid a separate communications board where practical. This supersedes the 2026-10-30 preference to keep the same MCU in V1 and flight. Keep the existing H743 V1 unchanged; no new MCU has been selected or verified.
- ESP32-S3FN8 is a research candidate: QFN-56 exposed-pad 7 x 7 mm, dual-core up to 240 MHz, 2.4 GHz Wi-Fi and BLE, with vector instructions useful to some neural-network workloads. Espressif sources: https://www.espressif.com/products/socs/esp32-s3 and https://documentation.espressif.com/esp32-s3_datasheet_en.pdf. LCSC candidate C2913196: https://www.lcsc.com/product-detail/WiFi-Modules_Espressif-Systems-ESP32-S3FN8_C2913196.html. Recheck current stock, assembly class and total BOM cost.
- ESP32-C5 QFN48 6 x 6 mm is a wireless-focused comparison candidate with 2.4/5 GHz Wi-Fi, BLE and 802.15.4. It is single-core and no dedicated NN acceleration has been established; benchmark before judging suitability. Source: https://documentation.espressif.com/esp32-c5_datasheet_en.html
- A single wireless chip may handle PC telemetry and other compatible protocols, but ESP32 Wi-Fi/BLE is not automatically an ExpressLRS or camera video receiver; same-band systems can interfere and a shared RF module time-slices protocols. Confirm laptop/OS direct offline connection, camera link, control and recovery independence, telemetry range and worst-case latency, compute/RAM headroom, power under TX load, motor timer/DMA compatibility, failsafes and RF/antenna layout before selecting a flight architecture. Retain small recovery/programming test pads even if large headers/USB connectors are removed.

## 2026-11-02 — Provisional JLCPCB manufacturing rules

- Configured project rules for a proposed 2-layer FR-4 board with 1 oz finished copper, 1.6 mm thickness and green mask; actual manufacturing options must be confirmed at order time. JLCPCB capabilities: https://jlcpcb.com/capabilities/Capabilities?type=1
- Preserved 0.20 mm default track width and clearance, set 0.127 mm minimum track width and 0.10 mm absolute copper clearance, and added routing-width presets of 0.20, 0.25, 0.40 and 0.60 mm. The custom rule file allows 0.15 mm only between U5 pads. This does not relax U5 pad-to-track clearance, which remains governed by the default rules.
- Set standard vias to 0.70 mm diameter / 0.30 mm drill, minimum via annular ring 0.20 mm, minimum drill 0.30 mm, copper-to-hole 0.30 mm, hole-to-hole 0.45 mm and copper-to-edge 0.50 mm. Custom rules require NPTH drill at least 0.50 mm. These are conservative choices rather than claims of mandatory JLC minimums.
- Set silk clearance and line/text thickness to 0.15 mm, text height to 1.0 mm and solder-mask-opening to copper clearance to 0.09 mm. A targeted U5 F.Mask physical clearance rule checks 0.10 mm mask separation. Do not apply KiCad's global 0.10 mm mask-width merge as a substitute without reviewing the Gerbers: it changed the exported mask in testing. The prior and final F.Mask Gerbers otherwise matched excluding timestamps.
- KiCad CLI DRC reports 14 U5 mask graphic gaps of approximately 0.04–0.085 mm, 4 missing mounting-hole footprint library entries, and remaining silkscreen errors/warnings, plus 105 unrouted connections. These are unresolved, not approval for manufacturing. KiCad's scoped U5 physical mask rule is not a complete global solder-mask DFM check; paste, assembly, BOM, impedance/USB, actual stackup and frame still require review.

## 2026-11-02 — Provisional PCB component placement

- After U5 failed to import, added a global Drone_MCU_Sensors footprint-library entry pointing to the same project-local library. William subsequently confirmed U5 imported via KiCad. The project-local library remains the portable source; the global entry is specific to this machine and project location.
- William authorized preliminary placement, with routing left to him. Used the KiCad MCP to position 31 remaining components while preserving the existing MCU, USB connector, SWD header and four mounting-hole positions. U2 is near USB, C10/C11 near U1 VCAP pins, and U5 near the MCU SPI side. The USB connector body deliberately extends beyond the top edge. Placement remains provisional; check USB access, decoupler routing, sensor placement, chosen frame and actual hardware clearance.
- Compared all pad-to-net assignments before and after; KiCad-exported schematic has 19 connected groups, with none split or merged on the board. There are no tracks. The MCP courtyard check shows no electrical-component overlaps; the connector's edge overhang is deliberate. PCB DRC still reports 105 unconnected items, four missing mounting-hole library entries, 12 IMU pad clearance errors (0.15 mm between certain pads vs 0.20 mm current default), and silkscreen violations. Do not treat this as fabrication-ready. Confirm the target stackup and appropriate local rules; the sensor's 0.05 mm solder-mask web needs explicit manufacturer review.

## 2026-11-02 — Provisional V1 PCB outline and mounting holes

- At William's request, created a 38 x 38 mm PCB outline with 2 mm corner radii and four 3.2 mm non-plated mounting holes at 30.5 x 30.5 mm pitch. This follows the previous full-size stack and ESC target, but 3-inch frames vary; choose and verify an actual frame, board outline, screw or grommet size, and stack clearance before fabrication.
- The four board-only MCP-generated hole footprints are excluded from BOM and placement output. MCP generated their library ID as MountingHole:MountingHole_3.2mm, which is not present in the installed KiCad library. PCB DRC reports four library-lookup errors; replace them with verified available mounting-hole footprints before manufacturing. Do not suppress these warnings. The 4.2 mm visual pad diameter is not a screw-head, washer or grommet keepout; define actual hardware clearance during placement.
- No schematic components have been transferred or routed. Before schematic-to-board update, retain the four board-only mounting footprints; review any delete-unmatched-footprints option. Assembly BOM, package checks, power budget, IMU footprint manufacturing review and remaining schematic ERC items are still open.

## 2026-11-02 — V1 schematic cleanup and portable sensor library

- At William’s request, marked 72 unused STM32 GPIOs as no-connect for this bench revision. Future use requires removing the relevant marker and wiring the pin; this does not provide accessible expansion pads by itself.
- Refreshed cached Device:C_Small and power:GND symbol definitions from the installed KiCad libraries and removed redundant local ground and VTref labels where the power symbols already define those nets. Compared the complete connected-net membership against the pre-cleanup KiCad XML netlist; no connected pins changed.
- Copied the ICM-42631 symbol, footprint and existing ICM-42688-P package model to the project-local Drone_MCU_Sensors library and updated the placed symbol to use it. The existing globally available library remains unchanged. The model alignment, solder mask/stencil and manufacturer assembly acceptance still require verification.
- KiCad CLI ERC reports two remaining items: U1 VCAP pins 48 and 73 are intentionally tied per ST hardware guidance, and U5 pin 9 is grounded for the single-interface configuration and must not be configured as an output. Neither warning is globally suppressed. VCAP capacitor ESR/effective capacitance, actual BOM/assembly availability, USB power budget and physical bring-up remain open.

## 2026-11-02 — V1 sensor changed to ICM-42631

- William replaced the original ICM-42688-P with ICM-42631, now U5, as a lower-cost candidate. Saved KiCad netlist: 5/8 +3V3; 2/3/6/7/9 GND; 10/11 no-connect; 1 MISO, 12 CS, 13 SCK, 14 MOSI and 4 INT1 remain correctly connected. C15 100 nF, C16 2.2 uF and C17 10 nF remain across +3V3/GND; R4 10 kOhm pulls CS high.
- Both datasheets specify 14-lead LGA 2.5 x 3 x 0.91 mm, 0.5 mm pitch; the user verified the same package. The assigned User_Sensors:PQFN50P300X250X97-14N was checked pad by pad against the 42631 top-view package drawing: numbering, nominal centers, pitch and pin-1 marker agree. All 14 solder-mask openings are centered and extend 0.05 mm around pads; full-size paste openings are present. The nominal 0.05 mm mask web and stencil settings still require fabricator review. Its global 3D model is for the 42688-P; alignment and repository portability remain unverified. Firmware, exact JLCPCB assembly availability and the BOM cost remain unverified. Use only single-interface mode with grounded auxiliary pins and do not configure grounded pin 9 as an output.

## 2026-11-02 — Provisional V1 programming probe

- William chose the AITRIP ST-Link V2-style SWD probe (Amazon ASIN B0D22S8WVX) as the provisional V1 programmer. Keep USB-C DFU as an additional intended programming path and SWD for debugging and recovery.
- This third-party probe has not been tested with the STM32H743VIT6 or the user's Mac. Confirm its actual pinout, SWD logic voltage, reset support, software compatibility, and behavior before connecting it. Its listing describes 3.3 V and 5 V pins as power outputs: do not assume either is a target-voltage sense input or connect competing power sources.
- J1 uses Cortex 10-pin signal numbering, but its physical connector and the adapter to the probe's 2.54 mm jumper wires remain to be selected and verified. USB DFU boot configuration, clock and power requirements, and actual operation also remain to be verified.

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

## 2026-11-03 — Pre-routing cleanup and assembly review

- Replaced four missing board-only mounting-hole footprint links with the official M3 3.2 mm footprint. Assigned J2 a project-local variant with three silkscreen lines moved to F.Fab because its connector shell crosses the board edge. Moved overlapping reference designators to F.Fab for legibility. Pad-to-net mapping was checked unchanged; the board has no tracks.
- Preserved the IMU mask geometry: a shared opening was explored but generates 80 solder-mask bridge violations and is not approved by the assembler. Existing 14 U5 mask-clearance violations remain a fabrication gate, alongside stencil and assembly review. CLI DRC also reports 105 expected unrouted connections.
- Candidate LCSC components, VCAP uncertainties and AP2112 current/thermal estimates are documented in `Drone_MCU_V1/V1_BOM_REVIEW.md`. JLCPCB assembly availability, exact orientations and final power budget are unverified. V1 remains an unrouted bench prototype; do not order or claim flight readiness on this basis.

## 2026-11-03 — Begin power and ground routing

- Started, but did not finish, power/ground routing. Added a B.Cu GND zone with a notch under U5, two local ground returns and selected F.Cu USB_VBUS/+3V3 traces. Signal nets were not routed. The zone and isolated trace groups do not imply full power or ground connectivity.
- Refilled zone and ran KiCad DRC: 90 unconnected items and the 14 previously documented U5 mask-spacing findings; no new reported DRC violations. Further routing, placement/decoupling and power/thermal review, U5 mask/stencil approval and final manufacturing checks remain.

## 2026-11-04 — Full routing authorized; four layers approved

- User authorized full PCB routing and component relocation, and explicitly approved a four-layer JLCPCB stackup for USB impedance and ground reference. Actual stackup geometry and impedance still require validation.
- Trial routing on a separate copy failed completeness and DRC checks; no trial board was applied. The active board remains the partial two-layer layout. Review evidence and the outstanding work are in `Drone_MCU_V1/reviews/2026-11-04/REVIEW.md`. Full routing and order readiness remain unfinished.

## 2026-11-04 — Main board electrically routed; manufacturing hold

- At the user's explicit request, enabled four layers and routed the main board. Routing is on the outer layers; inner layers carry GND zones. Final KiCad refill/DRC/parity reports 0 unconnected items, 0 schematic parity issues and the existing 14 U5 mask-spacing violations.
- USB routing is NOT a verified differential pair or 90 ohm geometry. Power width/current suitability, precise physical stackup, decoupling placement, regulator budget and assembly release remain unfinished. Main board is not order-ready. Updated evidence is in `Drone_MCU_V1/reviews/2026-11-04/REVIEW.md`.

## 2026-11-04 — USB routing trials and outline repair

- The saved board was missing the lower-left Edge.Cuts corner arc; restored the exact arc from the prior board backup. KiCad DRC after refill reports a closed outline, zero unconnected items, and the existing 14 U5 mask-clearance findings.
- Tried a fixed parallel USB segment and rerouting the rest of the board on a copy. The best complete-connectivity candidate still has a VBUS/USB trace crossing, two copper-clearance violations and one starved ground thermal. It is not applied to the main board. Full USB differential routing and order readiness remain open.

## 2026-11-07 — Final routing review and local ground returns

- User authorized final routing/electrical review and repairs, excluding silkscreen legibility. Added five 0.70/0.30 mm GND vias at C13/C6/C9/C17 ground returns and near the MCU-side IMU clock transition; all original traces and all 214 pad geometries/net assignments were checked unchanged.
- Restored four PCB references to match the schematic (SW1/SW2/TP1/TP4), preserving descriptive labels as separate text. Explicit final CLI parity now has zero findings; DRC has only 14 silkscreen dimension warnings and the two justified U2 internal USB connection exclusions. No new DRC suppression was added.
- Review and evidence: Drone_MCU_V1/reviews/2026-11-07-final/REVIEW.md. Sustained-load AP2112 thermal budget, manufacturer-confirmed stackup/USB impedance, final BOM/capacitor qualification and assembly checks remain open. Older fabrication archives predate these changes and must be regenerated before ordering. This is a bench-prototype routing pass, not unrestricted full-load or assembly release.

## 2026-11-08 — Budget manufacturing quote package; production hold

- User requested minimum overall assembled cost. Recommend quoting five fabricated boards with only two assembled, using the current H743/ICM-42631 design; no part substitution or PCB edit applied. ICM-42631 requires Standard PCBA per the official listing. ICM-42688-P permits Economic but its observed unit cost may erase savings; cheapest delivered option is not established without a real quote.
- Complete current Gerbers, separate drills and candidate 31-component / 17-line BOM and CPL are in `Drone_MCU_V1/manufacturing/2026-11-08-quote/`. Read ORDERING.md and REVIEW.md before using them. Files marked QUOTE_ONLY are not production authorization. J1 centroid corrected in CPL only; rotations still require vendor preview.
- Fresh DRC has 14 silk warnings, two justified excluded U2 internal USB connections, zero parity findings and no unexcluded copper/connectivity errors. Audit copy enabled all ignored categories and found five connected, off-center GND track/via endpoints. Independent Gerber drill check matched 55/55 holes/slots within 0.002 mm. Source PCB hash remains 5d5d9df668ff5c6862805d0a10dd6b63afac1c4cca6bb96d63a87378d499f9c6.
- Production remains held for actual assembly matching/stock and capacitor qualification, stackup/carrier/stencil/placement approval, and sustained-load regulator requirements. AP2112 is not cleared for unrestricted full-speed NN operation. No order, payment, firmware test or hardware qualification performed.

## 2026-11-08 — OSP finish for budget quote

- User selected OSP instead of ENIG to save approximately $20. JLC describes OSP as flat and suitable for fine-pitch assembly; assembler acceptance for the custom U5 pads remains open. Unassembled boards need appropriate sealed storage and prompt soldering after opening. No board artwork changed.

