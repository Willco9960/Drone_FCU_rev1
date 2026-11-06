# V1 preliminary parts and assembly review

Updated 2026-11-03. This is a candidate BOM for the existing, unrouted USB-powered bench prototype. It is **not an assembly order or a JLCPCB stock reservation**. Retail LCSC availability and JLCPCB assembly availability are separate, change with time, and must be confirmed in the actual JLCPCB order/quote. The MCP JLCPCB database contained no catalog data for these lookups. The schematic does not yet embed verified LCSC fields; do not export a final assembly BOM from it without completing those fields and checking orientations and footprints.

| References | Qty | Candidate part / LCSC | Review |
|---|---:|---|---|
| U1 | 1 | ST STM32H743VIT6 / [C114409](https://www.lcsc.com/product-detail/C114409.html) | LQFP-100, 14 x 14 mm. Confirm silicon revision and startup clock/USB firmware. |
| U2 | 1 | ST USBLC6-2SC6 / [C7519](https://www.lcsc.com/product-detail/C7519.html) | SOT-23-6. Verify exact ST part/assembly placement and USB connector-to-protection-to-MCU routing. |
| U3 | 1 | Diodes Inc. AP2112K-3.3TRG1 / [C51118](https://www.lcsc.com/product-detail/C51118.html) | SOT-23-5. Thermal and USB-current limits remain a design gate; see below. |
| U5 | 1 | TDK ICM-42631 / [C41432121](https://www.lcsc.com/product-detail/C41432121.html) | LGA-14. Mask/stencil/assembly acceptance open; do not approve assembly yet. |
| C1–C5, C7, C8, C12, C15 | 9 | Samsung CL21B104KBCNNNC / [C1711](https://www.lcsc.com/product-detail/C1711.html) | 100 nF, 50 V, X7R, 0805, ±10%. |
| C6 | 1 | KEMET C1206C475K4RACTU / [C2167346](https://www.lcsc.com/product-detail/C2167346.html) | 4.7 µF, 16 V, X7R, 1206, ±10%. Effective value at 3.3 V and operating temperature not yet verified. |
| C9, C13, C14 | 3 | Samsung CL21B105KAFNNNE / [C116352](https://www.lcsc.com/product-detail/C116352.html) | 1 µF, 25 V, X7R, 0805. Effective capacitance at 3.3/5 V and LDO stability not yet verified. |
| C10, C11 | 2 | Murata GCM31CR71H225KA55L / [C126604](https://www.lcsc.com/product-detail/C126604.html) | 2.2 µF, 50 V, X7R, 1206. Strong VCAP candidate; manufacturer-specific guaranteed ESR versus frequency, DC bias and temperature still require confirmation. |
| C16 | 1 | Samsung CL21B225KAFNNNE / [C19110](https://www.lcsc.com/product-detail/C19110.html) | 2.2 µF, 25 V, X7R, 0805. Check effective value and TDK requirements. |
| C17 | 1 | Samsung CL21B103KBANNNC / [C1710](https://www.lcsc.com/product-detail/C1710.html) | 10 nF, 50 V, X7R, 0805. Check against the device configuration and TDK guidance. |
| R1, R4 | 2 | UNI-ROYAL 0805W8F1002T5E / [C17414](https://www.lcsc.com/product-detail/C17414.html) | 10 kΩ 1%, 0805. |
| R2, R3 | 2 | UNI-ROYAL 0805W8F5101T5E / [C27834](https://www.lcsc.com/product-detail/C27834.html) | 5.1 kΩ 1%, 0805, independent USB-C CC pull-downs. |
| R5 | 1 | UNI-ROYAL 0805W8F1001T5E / [C17513](https://www.lcsc.com/product-detail/C17513.html) | 1 kΩ 1%, 0805. |
| D1 | 1 | KENTO KT-0805R / [C2295](https://www.lcsc.com/product-detail/C2295.html) | Red 0805 LED; verify component polarity against footprint pin 1 and assembler orientation before release. |
| J2 | 1 | HRO TYPE-C-31-M-12 / [C165948](https://www.lcsc.com/product-detail/C165948.html) | Confirm exact footprint, shell-tab drills and JLCPCB treatment of through-hole shell tabs. |
| SW1, SW2 | 2 | Omron B3U-1000P / [C231329](https://www.lcsc.com/product-detail/C231329.html) | SMD switches; verify footprint and assembly availability. |
| J1 | 1 | CJT A2541WV-2x5P / [C225520](https://www.lcsc.com/product-detail/C225520.html) | 2x5 2.54 mm through-hole header. Verify mating cable/pin numbering; through-hole assembly may require separate service or hand soldering. |
| TP1, TP3, TP4 | 3 | PCB test pads | No component to purchase or assemble. |

## VCAP

ST [AN4938](https://www.st.com/resource/en/application_note/an4938-getting-started-with-stm32h74xig-and-stm32h75xig-hardware-development-stmicroelectronics.pdf) specifies two 2.2 µF ceramic capacitors, each below 100 mΩ ESR, with the VCAP pins tied. A published [Infineon reference](https://www.infineon.com/export/sites/default/cn/partners/partner-ecosystems/greater-china/autoeco/led/TLD5095_TLD5098_DC-DC_Multi-topology_Controller_IC.pdf) identifies the Murata GCM31CR71H225KA55 family as an automotive counterpart of the GRM31 2.2 µF / 50 V part and illustrates approximately 5 mΩ ESR at 400 kHz and about 1.7 µF at 18 V bias. These are example/typical curves, not a guaranteed upper ESR bound across frequency, temperature and lot. The MCU nominal core voltage is much lower, but specific curves, applicable frequency and ST minimum effective capacitance must be confirmed before marking this production-approved. Place the capacitors close to their VCAP connections and provide a short return.

## Power and thermal budget

For the current 5 V USB-to-3.3 V LDO, approximate dissipation is **P = (5 − 3.3) × I**, ignoring small quiescent current. At 150 mA that is 0.255 W; at 226 mA it is 0.384 W; at 348 mA it is 0.592 W. The AP2112 SOT25 data sheet gives a reference junction-to-ambient thermal resistance of 184 °C/W; those points imply rises of about 47 °C, 71 °C and 109 °C under that test condition, not a measured temperature on this PCB. Actual PCB copper, ambient, airflow, USB input tolerance and load determine the result. The 600 mA current rating does not prove that the package can dissipate the corresponding heat.

ST's STM32H743 datasheet shows 480 MHz operation with all peripherals enabled around 226 mA typical and 348 mA maximum at 25 °C junction for the specified conditions (rev V table). These are examples, not this firmware's measured draw; additional IMU, LEDs, I/O and leakage add load. At the 348 mA example the LDO thermal margin can be poor, and MCU maxima vary strongly with junction temperature and workload. Firmware must start conservatively, respect source-advertised/USB configured current, and remain within a measured thermal/current budget. Do not claim USB or 480 MHz operation is validated without testing. If full workload exceeds the budget, redesign the regulator/power path before ordering rather than assuming the 600 mA label is sufficient.

Sources: [ST STM32H743 datasheet](https://www.st.com/resource/en/datasheet/stm32h743vi.pdf), [Diodes AP2112 datasheet](https://www.diodes.com/datasheet/download/AP2112.pdf), [Microchip USB power overview](https://developerhelp.microchip.com/xwiki/bin/view/applications/usb/power-delivery/).

## IMU mask and stencil

[TDK AN-000393](https://invensense.tdk.com/wp-content/uploads/2023/05/an-000393_imu-pcb-design-and-mems-assembly-guidelines.pdf) recommends non-solder-mask-defined lands and mask openings 0.10 mm larger in each dimension, and suggests a 90% stencil-opening-to-land ratio under standard conditions. Current PCB has full-size paste and +0.05 mm mask expansion per side; some mask webs measure approximately 0.04–0.085 mm. JLCPCB's [capabilities](https://jlcpcb.com/capabilities/pcb-capabilities) cite 0.10 mm mask bridges for 1 oz standard colors and a 1:1 opening capability, while its [mask guide](https://jlcpcb.com/blog/basic-design-of-solder-mask) notes standard processing can modify narrow webs. Shrinking to 1:1 without review would conflict with TDK's nominal NSMD recommendation; ganging openings changes bridging and exposed-copper behavior. Need a specific JLCPCB/TDK-approved mask, stencil and assembly plan or a verified alternative before fabrication. No manufacturer confirmation has been obtained.
