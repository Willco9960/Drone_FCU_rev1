# Final manufacturing preparation — 2026-11-08

**Verdict: fabrication data prepared for quoting; production release remains HOLD.** No board or schematic edits were made in this run. There is no claimed working-hardware guarantee or verified cheapest delivered quote.

## Results and previous-review delta

The source PCB hash still equals the 2026-11-07 reviewed revision: `5d5d9df668ff5c6862805d0a10dd6b63afac1c4cca6bb96d63a87378d499f9c6`. Earlier review evidence is in `Drone_MCU_V1/reviews/2026-11-07-final/REVIEW.md`. Its manual IC pinout, USB return-path and power-network review remains applicable to the unchanged board. This run adds fresh CLI DRC/ERC, an all-categories DRC audit copy, complete silk-inclusive exports, independent Gerber parsing/render inspection, every-hole registration check, candidate BOM/CPL and cost research.

| Check | Result / basis |
|---|---|
| Saved-board DRC with refill, all track errors and schematic parity | 14 silk-size warnings, 0 parity findings, 2 excluded USB connection findings; no unexcluded copper/connectivity errors |
| All previously ignored DRC categories enabled on audit copy | Five off-center GND track/via endpoint warnings, no other new categories; source project settings unchanged |
| Off-center GND copper | Endpoint distances 0.1 or 0.2 mm; radial copper overlap 0.350 or 0.375 mm. Connected, not a missing trace; no reroute justified solely to center endpoints |
| Schematic ERC | 17 cached C_Small/library differences plus VCAP power-output and grounded IMU bidirectional-pin findings; 19 total, not zero |
| Gerber layers | 4 copper, 2 mask, 2 silk, 2 paste and outline, plus job file; B.Paste is correctly empty for top-only SMT |
| Drills | 49 plated objects (35 vias, 10 header holes, 4 USB slots); 6 NPTH (4 mounting holes, 2 USB locating holes) |
| Registration | 55/55 Excellon hole/slot positions AND dimensions match saved pcbnew objects within 0.002 mm; zero extras |
| Render inspection | Four copper layers reviewed; inner GND and IMU pour cutout visible. Top mask/paste silhouette reviewed. No new gross export defect observed |
| BOM/CPL parity | 31 populated components, 17 unique groups, all on top; 3 test pads deliberately omitted |
| J1 centroid | Corrected position file only, from pin-1 anchor to header center; no PCB move |

The two excluded USB findings are copper islands joined by U2's internal connections, pins 1–6 and 3–4. Exact ST USBLC6-2SC6 must be assembled. The grounded U5 pin 9 must remain configured as an input, never an output. VCAP pins must remain connected as required by the MCU design. The 17 library differences are not silently waived as identical symbols; saved capacitor connectivity was preserved and the differences remain a maintenance item.

## Power tree and release holds

USB VBUS → AP2112K-3.3 → H743, IMU and LED. VCAP is a separate MCU regulator output with two 2.2 µF capacitors, not a 3.3 V input. AP2112 loss at 5 V input is approximately 1.7 × load current: 0.255 W at 150 mA, 0.510 W at 300 mA. Its 184 °C/W reference thermal resistance corresponds to about 47 °C and 94 °C rise respectively. These are estimates, not measured board temperatures. USB input tolerance and ambient can worsen them.

| Hold | Why / close condition |
|---|---|
| Sustained-load power | A 480 MHz NN workload has not been established to fit the LDO thermal budget. Establish a conservative bench operating envelope and verify it on hardware, or redesign the supply before ordering for unrestricted load |
| Assembly sourcing and passive qualification | Candidate BOM has exact proposed parts, but all 17 JLC assembly availability entries and VCAP/LDO effective capacitance/ESR have not been qualified. Live quote matching and manufacturer curves required |
| Vendor engineering acceptance | 3313 stackup, carrier/fiducials, U5 stencil and component preview need actual JLC review. Neither pad-1 rotation corrections nor connector centroid alignment can be guaranteed from generic package conventions |

## USB, mechanical and assembly

Both USB data nets stay on F.Cu, 0.15 mm traces and no signal vias. Earlier dense geometric sampling found a continuous In1 ground reference beneath them. End-to-end copper skew depends on the Type-C contact branch; no full-speed meanders were added. Exact differential impedance and USB compliance are not established. The 3313 stackup is required to preserve the reviewed geometry. Do not silently accept 7628.

38 × 38 mm rounded outline, four 3.2 mm mounting holes at 30.5 mm pitch. This is a bench board; no actual final flight frame, screw-head/cable assembly or mechanical fit has been tested. The unshrouded J1 header is 2.54 mm, not the 1.27 mm Cortex connector. Never connect an unknown probe's 3.3 V power output as if it were a target-reference input.

U5 custom pad/paste geometry gives an aperture area ratio of about 0.764 at 0.10 mm stencil thickness and 0.637 at 0.12 mm. Request the thinner local stencil and vendor acceptance; this is an engineering recommendation, not assembler approval. J2 shell slots exported with correct saved dimensions. Its exact vendor drawing, centroid and placement orientation remain part of the assembly acceptance.

## Placement landmarks (top view, USB at top)

| Part | Physical orientation checkpoint |
|---|---|
| U1 H743 | Pin 1 on upper end of left pad row, board XY 110.825,113.000 mm |
| U2 USBLC6 | Pin 1 upper-left, 126.1125,108.0875 mm |
| U3 AP2112 | Pin 1 upper-right, 111.550,106.2625 mm |
| U5 ICM-42631 | Pin 1 upper end of left row, 117.3375,131.950 mm |
| D1 | Pad 1 cathode left at 125.6375,130.825; pad 2 anode right on +3V3 |
| J1 | Pin 1 top-left, 133.460,113.420; body center 134.730,118.500 |
| J2 | Opening faces upper board edge; check shell tabs and contact row, not just a rotation number |

Raw PCB coordinates above use KiCad screen Y; CPL Y is their negative. All final rotations must be checked in JLC preview. No blanket package-family offsets were applied.

## False positives and limits

The Gerber analyzer flags differing copper extents as two GR-002 registration warnings. Trace extents and plane extents legitimately differ. Independent drill registration and visual layer review found no corresponding offset. Gerbonara warns about KiCad's post-header G90 and a RoundRect macro parameter; files parsed and the hole geometry matched. Its filter-based pretty preview was unsuitable for Cairo, so the review used opaque-background single-copper-layer renders instead.

Earlier schematic, full PCB, cross-domain, thermal and EMC analyzer outputs were reviewed and retained for the unchanged source. No repeated generic analyzer reruns were needed. Previous EMC split-plane and capacitor courtyard heuristic warnings remain triaged in the prior report. Thermal analyzer's guessed load does not qualify this board. No SPICE simulator was found in the prior tool check, and no SPICE analysis is claimed. No full component lifecycle audit, qualified full-load simulation, exact MLCC manufacturer curve qualification, vendor placement review, measured USB/firmware/thermal validation or complete landed quote was completed.

The JLC MCP parts database was empty; the public community API returned HTTP 403. Official public pages established assembly eligibility for U1, U5 and J2, and the ICM-42688-P alternative; they did not expose a complete, current purchasable quote for the chosen BOM. Do not interpret missing search results as out of stock. Public listings can be cached and must be rechecked at ordering.

## Cost decision

Preserve the existing components pending a real quote. Recommend only two assembled prototypes, green mask, normal turnaround, no separately shipped stencil and no premium via processing. Current U5 requires Standard assembly. A TDK 42688-P alternative permits Economic but the observed $19.4364 sensor price can erase fee savings; a redesign is not justified without the full total. The 17-line BOM is not claimed to be globally cost-minimal. Shipping, carrier, wastage and optional storage can materially change the result.

See `ORDERING.md` for exact baseline choices, sources and export instructions. Files are marked QUOTE_ONLY because the release holds are real, not because the Gerber generation failed.
