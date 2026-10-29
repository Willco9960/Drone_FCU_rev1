# Drone FCU project guidance

## Collaboration

This is William's learning project. Guide him through the schematic and PCB in small, meaningful steps; he normally places, wires, and routes the board himself. Use the KiCad MCP to review each saved step and give concrete corrections before moving on. Only edit the schematic, PCB, or libraries when he explicitly asks for that particular edit. For schematic changes, protect his unsaved work: KiCad's editor may overwrite external file edits. Work from a fresh saved copy, compare pin-level connectivity before and after, and coordinate reloading the editor.

Explain design choices in plain language, challenge assumptions, and be explicit about what is provisional versus verified. Prefer clear in-conversation visual diagrams over text-only wiring diagrams when useful. Use the ponytail:ponytail skill for relevant coding work. Keep `DECISIONS.md` updated when a meaningful design decision changes; include date, reason, and unresolved constraints.

## Project and goals

- KiCad 10 project: `Drone_FCU_rev1.kicad_pro`, schematic `Drone_FCU_rev1.kicad_sch`, PCB `Drone_FCU_rev1.kicad_pcb`. The design is unfinished; a schematic check is not a flight-readiness certification.
- Goal: a basic quadcopter flight controller for a 4S LiPo and a separate 4-in-1 brushless ESC, with a 30.5 mm mounting pattern. Keep the first PCB approachable for a KiCad beginner.
- User prefers parts available from LCSC and affordable options, but availability and pricing must be checked when sourcing. Radio receiver selection and motors/propellers remain open.
- A lightweight neural network for control is a future exploration, not a replacement for a verified baseline control and motor-safety design. Firmware and flight stability are not complete.

## Current design choices

- U1: STM32F722VETx. `MOTOR1`–`MOTOR4` are on PA0–PA3. UART labels `RC_TX`/`RC_RX` use PB10/PB11. Verify timer/DMA and firmware compatibility before promising DShot support.
- U4: ICM-42688-P IMU on SPI1 (PA5/PA6/PA7), PB0 chip select with 10 kΩ pull-up, PC4 interrupt. Check the datasheet and the actual schematic before modifying reserved pins or the unused INT2/FSYNC/CLKIN pin.
- U2: AP2112K-3.3 LDO for 3.3 V; J2 USB-C receptacle, separate 5.1 kΩ CC resistors, U3 USBLC6-2SC6 USB protection, and SWD test points. USB protection U3 VBUS is on `USB_VBUS`.
- ESC candidate: AERO SELFIE 45A 4-in-1, model 4IN1-ESC-LDO, 2S–6S, 30.5 × 30.5 mm mounting, no 5 V BEC. It is an external PCB; J3 is the flight-controller connector, not an ESC footprint. The manual specifies an 8-wire 1.0 mm-pitch cable but does not identify a connector series or numbered mating pinout. Do not assume JST-SH compatibility or cable orientation; verify before wiring J3 or assigning a footprint. Motors and peak current have not been selected.
- U5 LM2596S-5 buck circuit is in progress: D1 1N5825, L2 100 µH provisional for a roughly 1 A 5 V load, C8 100 µF 50 V, C9 220 µF 25 V. These values and component ratings are not finalized. U5 VIN is not yet connected to the battery. The inductor, diode and capacitors need actual parts, footprints, current, saturation, ESR, ripple and layout verification.
- D4/D5 B5819W Schottky diodes form the planned source OR: D4 anode on `BUCK_5V`, D5 anode on `USB_VBUS`, cathodes on `+5V`. Confirm voltage drop, current/thermal ratings and reverse leakage against the selected parts and connected loads. The rail after the diodes is below the source voltage and is not guaranteed to be exactly 5 V.
- Battery divider: `BAT_4S` → 100 kΩ R9 → `BAT_SENSE` to PC0, with 15 kΩ R8 and 100 nF C7 to ground. `BAT_4S` is not yet tied to an actual battery source. Assess input transients, ADC limits and unpowered-MCU injection before connecting a real battery.

## Before hardware

The ESC connector pinout, protection against battery transients/reverse connection, source isolation, load budget, USB behavior, component sourcing, thermal design, power-up sequencing, schematic ERC and PCB layout/DRC remain to be reviewed. Never connect a 4S battery directly to the AP2112 input. Do not treat the board as safe to power or fly until those reviews and appropriate bench tests are complete; remove propellers during motor testing.

## Libraries and tools

The repository includes `Drone_FCU.kicad_sym`, `Drone_FCU.pretty`, and `Drone_FCU.3dshapes` with project library tables. The placed IMU currently references the user's global `Sensor_Motion:ICM-42688-P` symbol and `User_Sensors` footprint, so a fresh clone may not resolve those paths even though project copies exist. Confirm and repair portability deliberately; don't silently edit the signed KiCad app's built-in libraries. Respect the user's preference for globally available symbols when importing new parts.

The KiCad MCP can inspect the schematic; use KiCad's exported netlist for an authoritative pin-level check, including power nets. A test of the MCP's schematic wire deletion on a temporary copy removed unrelated junctions; never assume a reported local edit has only local effects. Verify diffs and compare whole-design netlists before applying file edits. The MCP image exporter may fail to find `kicad-cli`; on the author's Mac it is at `/Applications/KiCad.app/Contents/MacOS/kicad-cli`.

Do not commit live `*.lck` files or transient `.history/` files. Preserve the user's project changes and do not overwrite open-editor state.
