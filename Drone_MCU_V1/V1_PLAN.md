# Drone MCU V1: project plan and handoff

Last updated: 2026-10-31. The saved KiCad schematic is the source of truth for the current hardware. Plans and targets below are not verified hardware or performance claims.

## Objective

Build a USB-powered tabletop prototype to develop and evaluate a neural-network flight-stabilization approach. The board reads an IMU, streams timestamped sensor data and MCU model output to a computer, and supports real-time visualization and recording. It must be straightforward to program and debug. V1 has no motor actuation and does not fly.

The long-term goal is a 3-inch-frame autonomous quadcopter that can follow, track and orbit a subject. The flight controller must handle onboard stabilization and motor safety; heavy camera/vision computation is intended to run remotely on a desktop or laptop, which sends high-level movement commands to the aircraft. Video link, command link, range, latency, localization, obstacle handling, subject-loss handling, manual override and safety are future design work, not implemented capabilities.

## Collaboration and project layout

- William builds the schematic and PCB himself as a learning project. Guide him in meaningful groups with clear, readable interactive wiring diagrams. Avoid drawing out the work by asking him to perform one trivial action per round. After each saved group, inspect with KiCad MCP and give precise corrections.
- Do not edit the schematic, PCB or libraries unless he explicitly requests that particular edit. Preserve open-editor/unsaved state. If explicitly editing outside KiCad, start from a fresh saved copy, inspect the entire diff and compare authoritative KiCad-exported pin-level netlists before and after; coordinate editor reload.
- The V1 files are Drone_MCU_V1.kicad_pro, Drone_MCU_V1.kicad_sch (root), MCU.kicad_sch (hierarchical sheet), and Drone_MCU_V1.kicad_pcb. See AGENTS.md here for agent instructions and ../DECISIONS.md for chronological design history. References in the root AGENTS.md describe the earlier F722 project; do not apply them as V1 pin assignments.
- The earlier REV_1/Drone_FCU_rev1.kicad_sch schematic is retained as a historical/reference design. V1 is a fresh design; do not copy unresolved battery, ESC or MCU circuitry without an independent review.
- User prefers LCSC-available and reasonably priced parts and beginner-friendly footprints. Check current price/availability and manufacturer datasheets before selecting components. Use visual diagrams that are readable without overlapping text.

## Current saved state (2026-10-31)

- KiCad 10 hierarchy: root contains an MCU sheet; the MCU sheet is A3. U1 is STM32H743VITx with LQFP-100 footprint. V1 PCB is an initial blank board; no routed hardware has been verified.
- KiCad MCP root netlist reported U1 VDD 11/27/50/75/100, VBAT 6, VDDA 21 and VREF+ 20 on +3V3; VSS 10/26/49/74/99 and VSSA 19 on GND. C1-C5 and C7-C8 are 100 nF, C6 is 4.7 uF, C9 is 1 uF across +3V3/GND. Confirm actual placement and chosen component voltage ratings during layout/BOM review.
- VCAP pins 48 and 73 share a net, with C10 and C11 each 2.2 uF to GND. Do not connect VCAP to +3V3. Confirm capacitor ESR, DC-bias behavior, placement, regulator configuration and exact package requirements against ST documentation before fabrication.
- At last saved check, U1 NRST 14, PA13 72 and PA14 76 are unconnected; an SWD/reset circuit was proposed but has NOT been verified as placed. Check the saved project again before advising, because William may have unsaved/open changes.
- No verified V1 USB power circuit, IMU, SWD connector, oscillator, boot configuration, receiver, ESC, regulator, or firmware yet. Most capacitors do not have footprints. No complete ERC/DRC, datasheet cross-review, board layout, prototype test or flight validation has been performed.

## Planned hardware architecture

1. **MCU:** STM32H743VIT6 is the intended common candidate for V1 and future flight. Its 480 MHz / 2 MB flash / up to 1 MB total RAM gives headroom over the old F722, but a specific neural model plus flight firmware must be benchmarked. Verify all H743 supply pins, regulator mode, USB supply, VCAP capacitors, BOOT0, reset and clocks for the actual LQFP100 device against the datasheet and ST AN4938. Do not assume same-package F722 pin or power compatibility.
2. **IMU:** ICM-42688-P, intended SPI, is a candidate. Verify real symbol and footprint, supply and I/O voltage, pin mapping, decoupling, interrupt and reserved pins. Add it to a separate hierarchical Sensors sheet and ensure library portability. A 6-axis accelerometer/gyro can estimate roll/pitch under appropriate conditions but cannot provide absolute heading; yaw drifts.
3. **Programming and USB:** accessible SWD (PA13/PA14, target reference, ground and NRST) with a verified connector and external ST-LINK-compatible debugger is the primary programming/recovery path. USB-C for 5 V host power and telemetry; design CC resistors, ESD, data routing, current/power budget and a suitable 3.3 V supply. The 3.3 V supply symbols presently have no verified source. USB DFU/boot option is desirable only after verifying this exact part, pin/clock configuration and a live test; it is not the sole recovery method.
4. **Expansion:** reserve four motor-control-capable outputs, a receiver interface, and a separate companion/telemetry serial port for later revisions. Select pins by H743 alternate function, timer and DMA analysis. Do not imply DShot or a particular receiver protocol is already validated. Flight hardware and its pinouts are future decisions. User previously selected an AERO SELFIE 45 A 4IN1-ESC-LDO as a candidate; its connector family, mating pin numbering and cable orientation still need physical or reliable primary-source confirmation.
5. **Physical design:** aim for the future 30.5 x 30.5 mm flight-controller mounting pattern, subject to the actual frame and stack clearances. Decide V1 connector locations, board outline, layer count and assembly method before routing. Screen new parts for footprints and availability.

## Neural network, simulator and viewer

- Desktop: RTX 3080 (10 GB dedicated VRAM, 16 GB reported shared memory), i7-12700K, 32 GB RAM, Windows driver reported as 32.0.15.9174. Use as the candidate primary training/simulation machine; PyTorch CUDA is a proposed path, not installed or verified here. The M1 MacBook remains a candidate for programming, telemetry, visualization and lighter experiments.
- Proposed simulation starting point: gym-pybullet-drones + PyTorch, using a conventional PD/PID reference or reinforcement learning and a quadcopter dynamics model. Baseline sensor collection from hand motion tests sensing and noise, but does not establish motor-response dynamics or provide flight-control labels by itself. Vary model mass, inertia, motor dynamics, latency and sensor noise; reserve unseen trajectories/disturbances for evaluation.
- Train models offboard, convert/export using a toolchain confirmed compatible with the actual model and H743, then compare PC vs MCU inference numerically. Record model version, input/output units and coordinate frames, timestamp, CPU time, deadlines, RAM/flash, sample drops and telemetry throughput. User cares about training the stabilization NN, not just a visualization.
- In the viewer, one plane shows estimated measured board orientation. A second must be explicitly labeled a simulated response to the model recommendation, using a stated dynamics model and time horizon; proposed motor corrections are not themselves a physical attitude. Handle yaw drift, timestamps, coordinate frame calibration, disconnects and replay.
- Initial goals (unverified, not guaranteed): IMU sampling and inference 200 Hz, telemetry 100 Hz, viewer 30-60 Hz. Select and benchmark the actual model and adjust targets. Before any flight, validate closed-loop behavior against a conventional baseline and maintain independent arming, motor cutoff and command/link-loss behavior.
- NVIDIA Isaac Sim/Lab may be useful later for synthetic camera scenes. Isaac Sim 6.0.1 published 16 GB VRAM/RTX 4080 minimum; the user's 10 GB 3080 is below that published baseline, and shared memory is not dedicated VRAM. Do not assert that it works or cannot work without testing a specific supported configuration.

## Milestones and acceptance

1. **V1 schematic:** power, MCU, SWD/reset, USB, IMU and intentional expansion are specified and pin-checked. All selected parts and footprints are documented; hierarchy, ERC and pin-level exported netlist reviewed.
2. **V1 PCB:** frame fit, mounting, component clearances, power and return paths, IMU placement, USB routing, decoupling placement and DRC/fabrication outputs reviewed. No unsupported safety claim.
3. **Bench bring-up:** current-limited power procedure, inspect rails, confirm SWD and USB, read sensor identification and samples, check telemetry and behavior on disconnect. No ESC or propellers required.
4. **V1 training/demo:** collect/replay timestamped data, visualize measured orientation and simulated model response, evaluate a held-out simulation set, deploy and compare actual MCU inference, and report latency, sample loss and resource use.
5. **Later flight/autonomy:** separately complete and validate battery/ESC power and protection, receiver, four motors, conventional baseline stabilization, arming/failsafes, then independent simulation and controlled flight testing before adding remote vision and target-following.

## References

- H743 datasheet: https://www.st.com/resource/en/datasheet/stm32h743vi.pdf
- ST hardware design application note: https://www.st.com/resource/en/application_note/an4938-getting-started-with-stm32h74xig-and-stm32h75xig-mcu-hardware-development-stmicroelectronics.pdf
- ICM-42688-P: https://www.invensense.tdk.com/en-us/products/6-axis/icm-42688-p
- STM32Cube.AI: https://stm32ai.st.com/stm32-cube-ai/
- Simulator: https://github.com/learnsyslab/gym-pybullet-drones
- Burman, Quadcopter stabilization with Neural Network (UT Austin, 2016): user-provided PDF is not included in this repository. The report trained a small model to imitate a PD controller; its conclusion states it did not deploy the NN on the microcontroller for flight validation. Do not claim the paper proves NN flight performance.

## Git hygiene

Preserve the user's existing work. Do not stage or commit live *.lck files, .history folders, local *.kicad_prl preferences, or timestamped KiCad backup archives. Never copy credentials into the repo. Keep DECISIONS.md up to date when a meaningful decision changes, including rationale and remaining constraints.
