# JLCPCB order notes – HDMI-MIPI Bridge, micro-HDMI variant

Generated 2026-10-27 with KiCad 10.0.7 (docker `kicad/kicad:10.0.7`) from branch `micro-hdmi-jlc` (based on 6c68328).
Origin for gerbers, drill and CPL = board aux origin (bottom-left corner of the board).

## PCB options
| Option | Value |
|---|---|
| Layers | 4 |
| Dimensions | 68 × 45 mm |
| Thickness | **0.8 mm** (design says 0.908 mm, which JLC doesn't offer) |
| Impedance control | **Yes**, stackup **JLC04081H-3313**. TMDS and CSI-2 pairs are 0.1 / 0.1 mm, targeting 100 Ω diff; check with JLC's calculator. |
| Min via hole / diameter | **0.15 mm / 0.25 mm** option. Vias used: 0.15/0.30, 0.15/0.40, 0.20/0.60. This option costs extra. |
| Via covering | Tented |
| Solder mask / silk | Black / white |
| Surface finish | **ENIG** (two BGAs, 0.2 mm-pitch micro-HDMI pads) |
| Copper | 1 oz outer / 0.5 oz inner |

## Assembly
* Both sides: 93 parts on top, 18 on the bottom (passives + 4 SMT standoffs).
* 2 × BGA (TC358743XBG), so X-ray inspection is needed.
* Check rotations in JLC's placement preview, especially:
  * J2/J4 (micro-HDMI): mouth faces the top board edge.
  * U3/U6 (MSOP-8): pin 1 is bottom-left.
  * IC1–IC4 (SOT-23-5) and Y1/Y2 (pin 1 = OE).
  * U1/U4 (BGA A1) and J6.
* J2/J4/J6 CPL coordinates are pad-centroid; their footprint anchors aren't at the part centre.
* All 21 BOM lines have JLC stock for 10 boards (checked 2026-10-27). The tightest are J6 (Würth 68715014022): 17 in stock for 10 needed, and TC358743XBG(EL,NOK): 237.

## What changed vs. Antmicro rev 1.3.2
| Ref | Was | Now | LCSC |
|---|---|---|---|
| J2, J4 | Molex 471511001 HDMI-A (out of stock) | **Molex 467651001 micro-HDMI (Type D)**, KiCad footprint `HDMI_Micro-D_Molex_46765-1xxx` | C563878 |
| U3, U6 | NXP PCA9517ATP, HWSON8 (4 in stock) | **NXP PCA9517ADP,118**, same die, MSOP-8 | C80799 |
| Y1, Y2 | TXC 7C-27.000MBB-T (out of stock) | **HELE SSI027000E3CH**, 3.3 V CMOS, ±25 ppm, same pinout and land | C254224 |
| SP1–SP4 | Würth 9774060151R (out of stock) | **Sinhoo SMTSO2560CTJ**, M2.5 × 6 mm SMT standoff; PCB hole 3.7 → 4.25 mm | C2916361 |
| IC2, IC4 | TI TLV73312PDBVT (36 in stock) | **TI TLV74312PDBVR**, TLV743P with the same SOT-23-5 pinout, 300 mA | C485149 |
| IC1, IC3 | TI TLV73325PDBVT | TI TLV73325PDBVR (reel code, same part) | C882825 |
| C4–7, C47–50 | Walsin 0402B471K250CT | Yageo CC0402KRX7R9BB471, 470 pF 50 V X7R | C106207 |
| C8–11, C51–54 | TDK C1608X5R1V475M080AC | TDK C1608X5R1V475KT000E, 4.7 µF 35 V X5R | C329940 |
| R4, R17 | Bourns CR0402-FX-2001GLF | UNI-ROYAL 0402WGF2001TCE (JLC basic) | C4109 |
| R13, R26 | Bourns CR0402-FX-22R0GLF | UNI-ROYAL 0402WGF220JTCE (JLC basic) | C25092 |

Layout changes:
* **Micro-HDMI routing.** The connector sits at the same board edge, centred over the TPD8S009 ESD chip.
  * D2±/D0± come off the outer pad row on F.Cu.
  * D1±/CLK± (inner row) drop through 0.15/0.30 mm vias right in front of their pads and run on B.Cu. They come back up at the old pad row, keeping the original ESD-to-TC358743 routing.
  * Intra-pair skew is matched to 0.000 mm on all 8 pairs (`reports/tmds_lengths.txt`). Pair lengths grew by 3–6 mm, which is negligible for HDMI pair-to-pair skew.
* **DDC/HPD/CEC/5 V** re-routed to the new pinout.
* **U3/U6 (now MSOP-8)** moved 1.45 mm up into space freed by the smaller connector. R12/R25 nudged right by about 0.25 mm.
* **Y1/Y2 pads** moved to the datasheet land pattern (±1.27 mm, previously ±1.9 mm, which only half-covered the terminals).
* **Removed** 2 GND stitching vias per connector that fell inside the connector's shell keepout.
* **Project rules:** min via diameter 0.40 → 0.30 mm; min copper-to-edge clearance 0 → 0.25 mm.
* **Format:** files upgraded to KiCad 10.

## Validation (reports in `../reports/`)
* **DRC, project rules:** 0 unconnected, 0 schematic-parity issues.
  * Remaining items: library-not-found warnings (Antmicro's `lib/` isn't in the repo), the pre-existing overlapping In1 GND zones (same net, harmless) and silkscreen clipping.
  * Y1/Y2 silk outlines now touch the corrected pads, and connector silk runs off the board edge. The fab clips both.
* **DRC with JLC 4-layer rules (`jlc-4layer.kicad_dru`):** no track, clearance, annular, mask-bridge or edge violations.
  * Only finding: in the micro-HDMI via row (0.4 mm pitch), a via hole is 0.175 mm from the neighbouring via's copper, against a conservative 0.2 mm hole-to-copper rule.
  * That still meets JLC's via-to-via hole spacing (0.25 vs 0.2 mm) and copper clearance (0.10 vs 0.09 mm). If JLC's DFM flags it, the fallback is 0.15/0.25 mm vias.
* **ERC:** no new issues. Same library-not-found / off-grid warnings as the original design.
* **Gerbers** re-rendered and inspected (`reports/renders/`).
* **Silkscreen** still says "Rev. 1.3.2".
