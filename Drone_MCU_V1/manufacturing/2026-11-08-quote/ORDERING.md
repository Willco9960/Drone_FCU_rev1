# Drone MCU V1 — budget quote and manufacturing instructions

2026-11-08. **QUOTE ONLY — do not pay or release production yet.**

The routing review passes the active copper/connectivity rules with two intentional USBLC6 internal-connection exclusions. Final assembly stock, vendor placement approval, capacitor qualification and the sustained-load power budget are not closed. A clean DRC is not a substitute for those checks. No hardware or firmware has been tested. No order has been submitted.

## Recommended starting quote

Keep the present STM32H743VIT6 and ICM-42631. Start with **5 fabricated boards and 2 assembled boards**, not 5 assembled boards. This minimizes the number of expensive MCUs purchased before first bring-up. This is a budget recommendation, not a verified minimum checkout total. Compare the complete delivered total, including tax, handling, component wastage and shipping.

| JLCPCB field | Selection for this saved board |
|---|---|
| Base material | FR-4 |
| Different designs | 1 |
| Layers | 4 |
| Finished single-board dimensions | 38 × 38 mm |
| PCB quantity | 5 to start; keep assembly quantity separately at 2 |
| Delivery format | Single design; JLCPCB creates the required assembly carrier/panel |
| Thickness | 1.6 mm |
| PCB color | Green |
| Silkscreen | White |
| Surface finish | OSP (Organic Solderability Preservative) |
| Outer copper | 1 oz |
| Inner copper | 0.5 oz |
| Layer order | F.Cu / In1.Cu GND / In2.Cu GND / B.Cu |
| Required physical stackup | JLC04161H-3313 |
| Via covering | Tented |
| Via filling / capping | None; no resin fill or copper cap |
| Blind / buried vias | No |
| Castellated holes / edge plating / gold fingers | No / No / No |
| Electrical test | Standard included electrical test |
| Remove order number | No paid removal; place on back clear of holes/test pads |
| Build time | Normal, no expedited service |
| PCB Assembly | Yes |
| PCBA type with current U5 | Standard |
| Assembly side | Top only, including J1 through-hole installation |
| Assembled quantity | 2 |
| Fiducials / edge rails | Added by JLCPCB on carrier; approve panel drawing |
| X-ray | Required for U5; retain it |
| Confirm parts placement | Yes |
| Firmware programming / functional test | No; firmware/test program not supplied |
| Conformal coating / special packaging | None / standard protective packaging |
| Separate stencil shipped to you | No |
| Stencil/fixture storage | Decline optional storage if the quote allows it |
| Shipping | Lowest total delivered cost shown for your address, normal service |

**Stackup selection needs CAM confirmation.** Do not accept generic “No requirement” laminate in place of 3313. The saved outer dielectric is 0.0994 mm; the default 7628 option has a different spacing. If the quote exposes the stackup only after enabling “Impedance Control,” use that selector to request 3313 and have JLC clarify any charge and engineering changes before payment. This board has not been demonstrated to be a controlled 90-ohm differential pair; do not request blind trace-width edits or pay for an unsupported impedance certification. OSP is selected for low cost and flat fine-pitch lands; JLC must accept the U5 assembly process. Store any unassembled boards sealed and avoid prolonged air exposure.

Standard PCBA specifies a minimum processing size of 70 × 70 mm and requires rails/fiducials. A 38 × 38 mm finished PCB therefore needs their carrier/panel solution. **Do not change the actual board dimensions to 70 × 70 mm.** Obtain their carrier quote and confirm that the quantity represents two assembled finished boards, not two multi-board panels. USB overhang and depanelization stress near the MEMS sensor must be considered.

Sources: [JLCPCB assembly capabilities](https://jlcpcb.com/capabilities/pcb-assembly-capabilities), [physical stackups](https://jlcpcb.com/impedance), [ICM-42631 assembly listing](https://jlcpcb.com/partdetail/TDKInvenSense-ICM42631/C41432121).

## Cost comparison — what is known

| Item | Current ICM-42631 | Alternative ICM-42688-P |
|---|---|---|
| Assembly eligibility | Standard only | Economic and Standard |
| Exact supplier number | C41432121 | C1850418 |
| Current assembly price obtained? | No — dynamic page did not expose it | Public listing showed $19.4364 at quantity 1+ |
| X-ray | Required | Required |
| Design change approved/applied | No change | Not applied; pin modes, land pattern, BOM and firmware must be reviewed first |

The 42688-P is not automatically a cost saving. At two units its listed sensor cost is $38.87. The saved 42631 design may still be cheaper after Standard setup; a live quote is needed. LCSC retail inventory is not JLCPCB assembly inventory. Do not substitute a Tokmas-labelled or consignment placeholder merely because it has a similar name or a tiny displayed price.

Published JLCPCB fees retrieved 2026-11-08: Economic setup/stencil $8.18/$1.53; Standard single-side $25.56/$8.21. Standard charges $1.53 per Basic or Extended feeder; Economic $3.07 per Extended feeder. With 17 separate BOM lines, the Standard feeder illustration is $26.01. Setup + stencil + those feeders totals **$59.78 before parts, placement, X-ray, carrier, fixtures, shipping or tax**. This is a cost floor illustration, not an order quote. Required quantities/fees can differ when JLC processes the actual BOM. Avoid buying five assembled boards just to improve the displayed per-board average.

Sources: [official assembly fees](https://jlcpcb.com/help/article/pcb-assembly-price), [TDK ICM-42688-P listing](https://jlcpcb.com/partdetail/C1850418). The full landed total and the cheapest viable alternative remain unverified.

## Files prepared

| File | Use |
|---|---|
| Drone_MCU_V1_GERBERS_QUOTE_ONLY.zip | Fabrication upload: 11 layers + Gerber job + separate PTH/NPTH drills |
| BOM_QUOTE_ONLY.csv | 31 populated parts in 17 part groups, using the existing candidate MPNs |
| CPL_QUOTE_ONLY.csv | Exactly the same 31 designators; top side; mm; KiCad rotations normalized to 0–360 |
| placement_raw.csv | Unmodified KiCad position export for traceability |
| REVIEW.md | Results, release holds and limits |

BOM/CPL omit TP1, TP3 and TP4 (copper test pads). Mounting holes are mechanical, not purchased parts. J1's placement position was corrected from KiCad's pin-1 origin to the center of its 2 × 5 grid: X=134.730, Y=-118.500 mm. Other component origins remain KiCad values and require vendor preview alignment, especially J2. Negative Y coordinates are intentional: absolute KiCad export coordinates, not a mirrored bottom assembly.

Do not reuse an older fabrication ZIP, and do not upload the whole review folder as Gerbers. The quoted files are tied to PCB SHA256 `5d5d9df668ff5c6862805d0a10dd6b63afac1c4cca6bb96d63a87378d499f9c6`.

## Re-export in KiCad 10 (five actions)

1. **Save and check:** save schematic and PCB; refill zones with B; run DRC with schematic parity. Review exclusions explicitly. A changed board invalidates this package and its hash.
2. **Plot Gerbers:** PCB Editor → File → Fabrication Outputs → Gerbers (or File → Plot). Choose a new empty folder; select F.Cu, In1.Cu, In2.Cu, B.Cu, F.Mask, B.Mask, F.Silkscreen, B.Silkscreen, F.Paste, B.Paste and Edge.Cuts. Use Gerber, 4.6 precision, Protel extensions, X2 and net attributes, absolute origin, 1:1 filled output, no border/title and no drill marks. Subtract solder mask from silk. Plot.
3. **Generate drills:** Excellon, millimetres, decimal zeros, absolute origin, no Y mirror, separate PTH and NPTH; alternate oval-hole mode. Generate into the same folder. Open the resulting layers and drills in Gerber Viewer and check registration and outline, then ZIP only that folder's fabrication outputs.
4. **Export placements:** File → Fabrication Outputs → Component Placement. Choose CSV, mm, both sides, absolute origin; exclude DNP but do not exclude through-hole footprints because J1 is populated. Convert columns to `Designator,Mid X,Mid Y,Layer,Rotation`, omit test pads/holes, correct J1's pin-1 anchor to body center, and match references to the BOM. Do not apply a blanket 90°/180° package correction.
5. **Prepare assembly BOM:** use `Comment,Designator,Footprint,LCSC Part #,MPN`. The current schematic lacks finalized sourcing fields, so its default BOM is not a replacement for the supplied candidate BOM. Check each exact part in JLC's matcher, then approve the placement preview against pad 1/polarity. Upload Gerber ZIP, BOM and CPL as separate inputs. Stop before payment until the holds below are resolved.

Source: [JLCPCB KiCad Gerber/drill export guide](https://jlcpcb.com/help/article/how-to-generate-gerber-and-drill-files-in-kicad-9).

## Assembly review notes to supply with the quote

> Four-layer USB-powered bench prototype, finished size 38 × 38 mm. Require JLC04161H-3313 stackup (35 µm outer copper, 0.0994 mm outer prepreg, 15.2 µm inner copper, 1.265 mm core). Please quote the lowest-cost carrier/rails/fiducials for two assembled finished boards. Preserve the copper artwork pending approval of any engineering change. U5 is exact TDK ICM-42631 C41432121, custom LGA14 lands; retain X-ray. U5 lands are 0.475 × 0.250 mm, paste apertures 0.455 × 0.230 mm; please review a 0.10 mm local stencil thickness and compatibility with the remaining components. J1 is a populated 2.54 mm 2 × 5 through-hole header. U2 must be ST USBLC6-2SC6: its internal paired I/O pins complete the USB data paths. Please provide component placement preview and flag unavailable parts; no automatic substitutions.

These notes are prepared for the user; they have not been sent to JLCPCB.

## Before payment

1. Obtain actual assembly matching/stock, carrier charge and delivered quote for this BOM. Reject silent substitutions. Full lifecycle and capacitor bias/ESR qualification are not complete.
2. Resolve the AP2112 sustained-load requirement. It is not cleared for unrestricted 480 MHz NN operation. Either commit to a conservative, measured bench workload or redesign the regulator before fabrication; the 600 mA current rating alone does not solve heat.
3. Have JLC accept the exact stackup, IMU paste/mask plan and placement preview, including J2 body center, U1/U2/U3/U5 pin 1, D1 cathode and the J1 header.

Next action (under two minutes): open the Gerber ZIP on JLCPCB's quote page to obtain a quote, without paying.

