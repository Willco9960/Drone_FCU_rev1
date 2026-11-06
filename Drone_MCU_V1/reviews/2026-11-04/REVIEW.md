# Routing review — manufacturing hold

2026-11-04. The user authorized full routing, component relocation and a four-layer JLCPCB board. Full routing was NOT completed. No experimental PCB was substituted for the active board.

## Verified baseline

KiCad 10.0.7 CLI checked a fresh saved copy with schematic parity and zone refill: 90 unconnected items and 14 existing U5 solder-mask separation findings. No additional parity violations were reported. ERC reports the VCAP power-output pair and grounded bidirectional U5 pin 9. Those require explicit manufacturer/configuration rationale; they have not been blanket excluded. Baseline netlist and reports accompany this review.

## Trials and tooling limitations

A separate copy enabled In1.Cu and In2.Cu with draft GND and +3V3 pours. This was only layer-count/zone preparation, NOT a validated physical dielectric stackup. No USB impedance calculation or compliant paired route was completed.

MCP create_netclass reported persisted classes but failed its in-memory SetClass call. The exported DSN reported zero assignments and kept the power nets in its default class. Explicit DSN trial rules were used to investigate this; do not assume MCP success establishes applied routing widths.

Freerouting 2.5.0 native macOS arm64 CLI failed with a GraalVM serialization error. The Java release ran using an isolated Temurin 25 JRE. Trial one imported 332 track segments and 17 vias; KiCad after refill reported 24 unconnected items and 23 violations: 14 existing mask findings, five undersized 0.12 mm traces, three edge-clearance violations and one starved thermal. Its USB paths were not verified as differential routing. Trial two removed the back-layer plane obstacle from the router input and reduced trial power width from 0.4 to 0.25 mm; the router still reported 22 unrouted items. This second result was not accepted or applied.

Temporary experiments are in /private/tmp/drone-full-review and may disappear. The active project remains the original partial two-layer layout. The full-route request remains unfinished.

## Required before order release

- Implement the approved four-layer plan with a specific JLCPCB dielectric/copper stackup. Calculate and verify 90 ohm USB geometry and route the complete connector–ESD–MCU pair over continuous reference copper, including fanouts, stubs, length mismatch and transitions.
- Improve decoupling placement; complete every supply, VCAP, ground and signal connection. Verify actual widths and current/temperature assumptions, not just netclass settings. Review return continuity after zone fill.
- Resolve the known IMU mask/stencil design and assembly acceptance without silently ignoring errors.
- Finish the BOM and actual assembly-part eligibility, polarity and orientation review; resolve AP2112 load/thermal limitations and USB startup/current constraints. See V1_BOM_REVIEW.md.
- Repeat full schematic/footprint/netlist review, ERC, parity, DRC and visual Gerber/drill/assembly-output inspection on the final exact files. Zero airwires alone is not approval.

Sources: https://jlcpcb.com/impedance and https://jlcpcb.com/pcb-impedance-calculator . JLC lists stackup-dependent dielectric/copper inputs; no calculator result was obtained in this review.

## Subsequent work on the main board — still on manufacturing hold

The user explicitly requested main-board routing. The main PCB now has four enabled copper layers, routing confined to F.Cu/B.Cu, and GND pours on In1.Cu/In2.Cu plus the prior B.Cu pour. The original board is backed up at /private/tmp/drone-main-before/Drone_MCU_V1.kicad_pcb. Component placement and schematic connectivity were not redesigned.

A corrected router input discarded conflicting draft wires/plane obstacles and included the small SMD pad-clearance allowance; an inset routing boundary retained actual 0.5 mm board-edge clearance. KiCad's final main-board check after refill reports **0 unconnected items, 0 schematic-parity issues and 14 U5 mask-spacing violations**. No other reported DRC violation remains. ERC still reports the two previously documented items. Reports accompany this note.

This is electrical routing completion only, NOT order readiness or the completed requested differential routing. USB DP and DM take different routes/layer transitions; total segment sums (including connector branches) are 31.017 mm and 28.135 mm, respectively, and are not measurements of differential skew. USB geometry uses 0.15/0.20 mm widths with no verified 90 ohm physical stackup. The power nets also have 0.15/0.20 mm tracks; their suitability has not been established from a complete current/thermal budget. A specific dielectric stackup is not yet configured. Decoupling loops/placement and ground return geometry remain to be reviewed physically.

The requested USB coupled routing, width verification, power/thermal and full assembly release checks remain unfinished. Do not generate an order from this PCB merely because airwires are zero.
