# Drone MCU V1 — Economic assembly investigation

Date: 2026-11-08. **Research, not a production release or a modified design.** JLCPCB's catalog can change; recheck the actual quote. The already-uploaded design retains ICM-42631 and the existing BOM; no source KiCad files or order were changed in this investigation.

## Decision

The ICM-45686 is a plausible 6-axis SPI sensor for the bench prototype and is listed for both Economic and Standard PCBA. It is not a BOM-only drop-in: the present board connects its pin 9 to GND, whereas TDK's pin table says unused pin 9 should be left unconnected or tied to VDDIO. Grounded pins 2, 3 and 7 also use default weak pull-ups in the replacement and require correct firmware configuration; making them unconnected is an alternate hardware choice requiring further review. Pin 9 should be corrected in both schematic and PCB, not merely edited in the order. The changed board requires a full saved netlist/parity/DRC/placement review and a fresh manufacturing package.

This change may not reduce landed cost. Current JLC price and PCBA stock for the ICM-45686 were not exposed in the public JLC listing; recent LCSC retail pages displayed $11.8375 each, not an assembly quote. Current ICM-42631 quote is $3.9808 for two. Do not apply a generic retailer price as the final assembler price. Compare a temporary Economic quote first.

## ICM-45686 compatibility

| Pins | Present saved board | 45686 single-interface SPI | Assessment |
|---|---|---|---|
| 1 | MISO | AP_SDO | Functional match |
| 2,3 | GND | reserved in single-interface mode | Ground allowed in pin table; disable default weak pulls per its register instructions |
| 4 | IMU interrupt to MCU | INT1 | Functional match; firmware changes |
| 5 | 3.3 V | VDDIO | In allowed range |
| 6 | GND | GND | Match |
| 7 | GND | reserved | GND allowed; disable default weak pull-up per pin table |
| 8 | 3.3 V | VDD | In allowed range |
| 9 | GND | INT2/FSYNC/CLKIN | Conflict with TDK unused-pin instructions; leave open or tie to VDDIO after engineering review; do not enable driven output into GND |
| 10,11 | Unconnected | reserved in single-interface mode | Matches single-interface use; do not assume auxiliary interface available |
| 12 | CS, pull-up | AP_CS | Functional SPI match |
| 13 | SCK | AP_SCLK | Functional SPI match |
| 14 | MOSI | AP_SDI | Functional SPI match |

Pin table in [TDK DS-000577 pp. 27–29](https://invensense.tdk.com/wp-content/uploads/documentation/DS-000577_ICM-45686.pdf). The default weak pull-ups are not strong external pull-ups: this is an unnecessary steady bias current until firmware changes them, not an assertion of immediate damage. Datasheet calls them out individually; review startup/reset default and full register initialization. Pin 9 change still required for the specified unused-pin connection.

The 45686 supports SPI, I2C and I3C at the chip; the current V1 board is SPI wired, not a plug-and-play I2C/I3C board. I3C controller support and firmware must be separately established if wanted later. WHO_AM_I, initialization, output rates, ranges, register map, burst FIFO and interrupts must be ported and tested; do not assume 42631 firmware compatibility. The replacement is higher-performance than the PoC needs; suitability is about actual total cost and availability, not a requirement for high precision.

The TDK drawing describes a 2.5 × 3 mm 14-land package, 0.5 mm pitch and nominal 1.1625/0.9125 mm outer land center offsets. Saved U5 pcbnew pad centers are at those offsets. This is a **center-position screening result only**. Copper pad dimensions, package contact dimensions/tolerances, mask, paste and pin-1 orientation must be compared fully with the manufacturer/assembler and assembly preview before release. Existing U5 pads are 0.475 × 0.250 mm on the long rows, transposed on the other row. No footprint equivalence or vendor acceptance is claimed.

JLC listing: [ICM-45686 C22459454](https://jlcpcb.com/partdetail/TDKInvenSense-ICM45686/C22459454), Economic and Standard, Extended, X-ray required. [LCSC retail listing](https://www.lcsc.com/product-detail/Accelerometers_TDK-InvenSense-ICM-45686_C22459454.html) is not JLC PCBA stock or price.

## All 17 original BOM groups — Economic screening

Economic column means the official JLC part page explicitly reported Economic and Standard when retrievable. `Unknown` does not mean ineligible. User's saved live BOM screen showed all 17 matched under Standard, not that any are automatically Economic eligible. Basic/Extended column is from that screen.

| Refs | Exact JLC part | BOM class | Economic status | Action |
|---|---|---|---|---|
| U1 | C114409 STM32H743VIT6 | Extended | Yes | Keep MCU |
| U2 | C7519 USBLC6-2SC6 | Extended | Yes | Keep exact part; its internal connections complete USB data |
| U3 | C51118 AP2112K-3.3TRG1 | Extended | Yes | Keep for quote; full-load thermal limitation persists |
| U5 | C41432121 ICM-42631 | Extended | **No — Standard Only** | Evaluate 45686 with documented PCB/firmware changes |
| C1–C5,C7,C8,C12,C15 | C1711 100 nF 50 V X7R 0805 | Extended | Yes | Candidate Basic C49678; compare stock, footprint and quote |
| C6 | C2167346 4.7 µF 16 V X7R 1206 | Extended | Unknown | Candidate Basic C29823 4.7 µF 50 V X7R 1206; qualify effective capacitance, size and availability |
| C9,C13,C14 | C116352 1 µF 25 V X7R 0805 | Extended | Yes | Keep until qualified Basic equivalent is found; LDO capacitance/stability matter |
| C10,C11 | C126604 2.2 µF 50 V X7R 1206 | Extended | Unknown | MCU VCAP caps: do not replace based only on nominal capacitance; ESR and effective capacitance need review |
| C16 | C19110 2.2 µF 25 V X7R 0805 | Extended | Unknown | Check exact match in Economic selector |
| C17 | C1710 10 nF 50 V X7R 0805 | Basic | Yes | Keep |
| R1,R4 | C17414 10k 0805 | Basic | Unknown | Check Economic selector; nominal resistor spec alone doesn't prove class |
| R2,R3 | C27834 5.1k 0805 | Basic | Yes | Keep USB CC resistors |
| R5 | C17513 1k 0805 | Basic | Yes | Keep |
| D1 | C2295 KT-0805R red | Extended | Yes | Keep unless verified cheaper equivalent with cathode orientation |
| J2 | C165948 TYPE-C-31-M-12 | Extended | Yes | Keep; shell slots and placement review remain |
| SW1,SW2 | C231329 B3U-1000P | Extended | Yes | Keep |
| J1 | C225520 2×5 header | Extended | Unknown | Current listing lacked PCBA type; Economic supports through-hole in general, not proof this particular part is available |

Evidence: [U1](https://jlcpcb.com/partdetail/STMicroelectronics-STM32H743VIT6/C114409), [U2](https://jlcpcb.com/partdetail/C7519), [U3](https://jlcpcb.com/partdetail/DiodesIncorporated-AP2112K_33TRG1/C51118), [U5](https://jlcpcb.com/partdetail/TDKInvenSense-ICM42631/C41432121), [C1711](https://jlcpcb.com/partdetail/CL21B104KBCNNNC/C1711), [C116352](https://jlcpcb.com/partdetail/117606-CL21B105KAFNNNE/C116352), [C1710](https://jlcpcb.com/partdetail/CL21B103KBANNNC/C1710), [R2/R3](https://jlcpcb.com/partdetail/28584-0805W8F5101T5E/C27834), [R5](https://jlcpcb.com/partdetail/18201-0805W8F1001T5E/C17513), [D1](https://jlcpcb.com/partdetail/Hubei_KENTOElec-E6C0805UR/C2295), [J2](https://jlcpcb.com/partdetail/C165948), [switches](https://jlcpcb.com/partdetail/B3U-1000P/C231329).

### Potential Basic substitutions — candidates only

- C1–C5,C7,C8,C12,C15: [Yageo CC0805KRX7R9BB104 C49678](https://jlcpcb.com/partdetail/CC0805KRX7R9BB104/C49678), Basic, 100 nF, 50 V, X7R, 10%, 0805, economic eligible. Same nominal electrical and package family as C1711; verify stock, thickness and assembler footprint.
- C6: [Fenghua 1206B475K500NT C29823](https://jlcpcb.com/partdetail/30577-1206B475K500NT/C29823), Basic, 4.7 µF, 50 V, X7R, 10%, 1206, economic eligible. Candidate for the existing 4.7 µF 16 V cap, not accepted without physical and bias/capacitance review. Screen its placement model because the original C6 appeared as a checkerboard in JLC preview.
- C10/C11 VCAP: do not automatically select a cheaper cap by value and size alone. AN4938 calls for two 2.2 µF caps with ESR under 100 mΩ; actual capacitance under bias and frequency/temperature ESR still need qualification.

Even when a replacement is Basic today, catalog classification and feeder fees in an actual quote may differ. No parts were changed in the saved BOM or JLC quote.

## Cost model (not a quote)

The user's current Standard quote: PCB $8.00, assembly charges $68.18, components $32.57 = $108.75 before shipping/tax where applicable. Its setup + stencil = $34.02; feeder loading = $24.80. User's screen showed 13 Extended and four Basic groups.

JLC publishes Economic setup $8.18, stencil $1.53, and Extended feeder $3.07 per group; Standard feeder pricing applies to Basic as well. Economic could save approximately $24.31 in setup/stencil, but its per-Extended fee can be higher. As a transparent **undiscounted illustration only**, 13 × $3.07 = $39.91, which is $15.11 above the current $24.80 Standard feeder total. Net setup/stencil/feeder improvement would be around $9.20 before differences in sensor cost, promotions, other charges and shipping. Actual categories include discounted Extended tiers and current site prices change, so a real Economic quote is indispensable. Switching two truly full-fee Extended capacitor lines to Basic could save up to about $6.14 in that illustration, subject to actual discounted tiers and price differences. Never treat this as a promised checkout price.

Both the 45686 and the 42688-P are X-ray-required on the JLC listing, so do not remove X-ray from comparisons. 42688-P C1850418 previously displayed $19.4364 each and can be Economic, but may add about $35 for two vs the current IMUs and requires its own pin, land and firmware review. 42670-P offers SPI/I2C/I3C but JLC lists its C3288646 as Standard Only; it does not solve the blocker. 42607-P and 42605 listings examined also Standard Only. No lower-cost equivalent with confirmed footprint, actual availability and eligible Economic assembly has been established.

## Stackup and release

JLC says Economic supports standard stackup only; 4 layers and single-side assembly with through-hole components are supported, subject to material/finish/part limitations. User accepted standard stackup as a bench-prototype cost tradeoff. Original KiCad board uses 3313 outer dielectric 0.0994 mm; JLC's no-requirement standard example is 7628 0.2104 mm. USB impedance will change. Full-speed may work on a short board, but this is not proof or guarantee of USB function or 90 Ω. Retain GND plane continuity and test actual USB after fabrication. [Assembly capabilities](https://jlcpcb.com/capabilities/pcb-assembly-capabilities), [JLC stackups](https://jlcpcb.com/impedance).

Before any manufacturing: verify current JLC SKU eligibility/stock for the five unknown groups, actual Economic quote and total shipping, confirm U5 45686 datasheet/geometry and implement/check any accepted board changes, C6 model/assembly, polarity and placement, and the existing regulator power/thermal limitation. This V1 board is for bench IMU collection only, not drone flight.
