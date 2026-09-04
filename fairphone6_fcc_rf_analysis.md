# Fairphone (Gen. 6) — FCC exhibit analysis: mainboard silicon, RF front-end, antennas

**Source:** [FCC_2AUWUFP6_Fairphone_Gen6/](FCC_2AUWUFP6_Fairphone_Gen6/) (FCC ID 2AUWUFP6, Sporton Kunshan, reports issued Jul 2025, HW **DVT2**, SW `FP6.DEV.15.66.0`).
Extracted working files: `FCC_2AUWUFP6_Fairphone_Gen6/_extracted/` (native-res board photos in `inpho/raw/`, antenna map `inpho/antmap_crop.png`, text dumps `*.txt`).
Companion doc: [fairphone6_video_teardown_analysis.md](fairphone6_video_teardown_analysis.md) (mechanical/module level).

---

## 0. Headline findings

1. **The FP6 is an ODM design.** The antenna report is authored by **T2 Mobile (Shanghai) Ltd** (TCL's ODM arm), measured at "TCL Ningbo RD team", device part number **BQA6CT0A15C0**. Fairphone specifies; TCL/T2 does the board, RF and antennas.
2. **Cellular baseband is inside the SoC** (Snapdragon 7s Gen 3, SM7635). What sits *outside* the SoC is: a discrete Qualcomm **RF transceiver (SDR735)**, Qualcomm PMICs (PM7550, PM8550VS + one more), a Qualcomm **Wi-Fi 6E/BT combo (WCN6755)**, a Qualcomm audio codec (WCD937x), and a **third-party + Qualcomm RF front-end** (PA modules, receive modules, tuners/switches).
3. **11 antennas + NFC.** 8 are WWAN (Ant 0–6, 8), 3 are Wi-Fi/BT IFAs (Ant 7, 9, 10). n41 and n77/78 each use **four** antennas (SRS antenna switching for 4×4 DL MIMO). All are LDS/FPC on the plastic-overmoulded mid-frame and camera cover; no metal-band antennas.
4. **Qualcomm Smart Transmit** (time-averaged SAR power control) with per-antenna, per-scenario power tables is used — that is a platform feature you inherit, not something you build.
5. Mainboard is **≈62 × 54 mm**, L-shaped, 6+ coax receptacles, dense on both faces. The daughterboard (~70 × 20 mm) is essentially passive (SIM/µSD reader, B2B, antenna feed pads).

---

## 1. Silicon identified on the mainboard (from internal photos, shields removed)

Confidence: **H** = legible full part number; **M** = partial marking + context; **L** = guess from marking style.

| Photo | Marking (as read) | Identification | Conf. |
|---|---|---|---|
| p2 p04/06/07/10 | `Qualcomm SM7635 000-AB` | **Snapdragon 7s Gen 3** SoC (TSMC 4 nm); baseband modem inside | H |
| p2 p04 | `FE449FPP` on top of SoC | PoP **LPDDR5 DRAM** stacked on the SoC (8 GB per spec; vendor not legible) | M |
| p2 p04/10 | Micron logo, `4EA92 JZL38` | **Micron UFS 3.1**, 256 GB | H (vendor) |
| p2 p03 | `Qualcomm SDR735 001` | **RF transceiver** (sub-6 GHz 5G/LTE/3G/2G) — discrete companion to SM7635 | H |
| p2 p08/09 | `Qualcomm WCN6755 003` | **Wi-Fi 6E + BT 5.4** combo (FastConnect 6700 class), 2 blue FEM/filter parts beside it | H |
| p1 p15/16 | `Qualcomm PM7550 001` | **Main PMIC** | H |
| p1 p15/18 | `8550VS 001` | **PM8550VS** companion PMIC / regulator | H |
| p2 p04/05 | `Qualcomm PMIV0108 001` (as read) | Additional Qualcomm PMIC (likely camera/display or charger-side sub-PMIC), surrounded by inductors | M |
| p2 p04/05 | `WCD937B 000` | Qualcomm **WCD937x Aqstic audio codec** | H |
| p1 p14 | `QPM 29xx…` (partly legible) next to 3 top-edge coax receptacles | Qualcomm **QPM-series PA / front-end module** | M |
| p1 p14/15/19 | 2× `SMM1 S55643-51 2502 H0C1` (large ~6×8 mm modules, one per shielded RF bay) | Third-party **PAMiD / receive front-end modules** — marking style matches Skyworks "Sxxxxx-yy" laser codes | M/L |
| p2 p03 | `5030 002 0E445`, `5040 003 0H450`, `5020 001 0H451` (small Qualcomm-style codes around SDR735) | Qualcomm **RFFE parts (QAT tuner / QET envelope-tracker / antenna-switch class)** | L |
| p1 p15/19 | `2625 004 F502 BSG` (BGA with dot-matrix code, near WHITE/RED coax) | Qualcomm RFFE (antenna switch/tuner class) | L |
| p1 p15/18, p2 p09 | `FE431 JJ6 000 B` ×2 | Small Qualcomm-marked companion (unknown) | L |
| p1 p16/17 | white LED + `I4230 T93AB1` driver, `X1G1 ADF9`, `RN4VXC S4511M` | **Rear flash LED is on the mainboard** (light-pipe/dome in camera cover) + LED driver + charger/PD-side small ICs; TX/RX test pads next to it | M |
| p2 p09 | metal-can MEMS with QR code | MEMS microphone (top mic) on mainboard | H |
| p2 p05 | `3020 2443` small QFN | unidentified (haptics driver / sensor?) | L |

Also on the mainboard: 6+ MHF/IPEX coax receptacles (3 along the top edge, 3 near the `RED / WHITE / BLACK` silkscreen), 4 camera/display B2Bs, battery B2B, interconnect B2B, ~12 gold pogo pads (button/mic/NFC/antenna spring contacts).

**Not on the mainboard:** NFC controller was not legibly identified (likely under the WCD/PMIC area or on the back face); the ToF sensor sits on the camera-cover flex; the microSD/SIM reader is on the daughterboard.

---

## 2. Antenna architecture (antenna map p1 p06, per-antenna photos p3 p01–p11, SAR Part-0 tables)

| Ant | Location on frame | Radios (from SAR/antenna report) | Type / notes |
|---|---|---|---|
| **0** | Top edge, right of camera bump | GSM850, WCDMA V, LTE B5/12/17/26/71, n5, n71, **n41 (4th ant)** | Main **low-band** WWAN TX/RX |
| **1** | Bottom-left corner + bottom edge left | *not in TX SAR tables* | Bottom **RX-only / diversity or GNSS** (untested → non-transmitting) |
| **2** | Right side, upper (below camera cover) | GSM1900, WCDMA II, LTE B2/4/25/66, n2, n66, B42, **n77/78** | Main **mid-band** WWAN TX + n77/78 |
| **3** | Left side, upper (near Moment switch) | B42, **n77/78** | n77/78 SRS/MIMO |
| **4** | Right side, top (beside camera cover) | LTE B7/38/41 (+HPUE), n7/n38/n41 (+HPUE), B42, n77/78 | Main **high-band TDD** TX |
| **5** | Bottom edge, right | n41 (bottom TX) | Bottom WWAN |
| **6** | Right side, lower | LTE B7/B41, n7/n41 | Bottom/side WWAN (SRS) |
| **7** | Top edge, left (above cameras) | Wi-Fi 2.4 GHz | IFA, peak −2.1 dBi |
| **8** | Left side, lower | B42, n77/78 | n77/78 SRS/MIMO |
| **9** | Left side, small (yellow box) | Wi-Fi 5 GHz / 6E | IFA, ~ +1 dBi |
| **10** | Top-right, inside camera-bump area | BT + Wi-Fi 2.4/5/6E | IFA, −3.8 dBi @2.4G, ~+1 dBi @5–7 GHz |
| **NFC** | Camera cover | 13.56 MHz | FPC loop **40.7 × 29.15 mm** + 0.23 mm ferrite, differential drive |

Grouping used for simultaneous-TX SAR: **top group = Ant 0/2/3/4/8, bottom group = Ant 5/6**.

Take-aways:
- **8 WWAN antennas is the floor** for a 5G phone with n41 HPUE and n77/78 4-layer MIMO — not a Fairphone luxury. n41 uses Ant 0/4/5/6, n77/78 uses Ant 2/3/4/8.
- WWAN antennas ring the **mid-frame periphery** (top edge, both long sides, bottom edge); Wi-Fi/BT are pushed into the **camera-cover / top corners**; NFC coil is under the camera cover. This works because the outer wall is **plastic over a metal die-cast core** — no antenna slots needed.
- Wi-Fi gains are modest (−4 to +1 dBi): plastic housing + small IFAs. Acceptable for a mid-ranger.

---

## 3. Radio capability & power (SAR Part-0/Part-1, test-report front matter)

| Item | Value |
|---|---|
| WWAN bands certified (US) | GSM 850/1900 · WCDMA II/V · LTE 2/4/5/7/12/17/25/26/38/41/42/66/71 · NR n2/n5/n7/n38/n41/n66/n71/n77/n78 (SA + NSA; n77/78 up to 100 MHz BW) |
| Power class | PC3 everywhere; **PC2 HPUE on LTE B41 / n41** (tune-up limit 26.5 dBm) |
| Typical Pmax (tune-up) | LB ≈ 24–24.5 dBm · MB ≈ 24 · B7/38/41 ≈ 24.5 · n77/78 21–24.5 dBm (Part 27/96), 13 dBm under Part 96 CBSD-adjacent conditions |
| Dual SIM | DSDS, single active TX (nano + eSIM) |
| SAR mgmt | **Qualcomm Smart Transmit** (time-averaged), receiver/earpiece detection lowers head power; per-antenna Plimit tables loaded via EFS |
| Wi-Fi | 802.11 a/b/g/n/ac/ax, 2.4/5/6 GHz, HE160 on 6 GHz, SISO/MIMO (2 antennas), 6 GHz "standard client + indoor client" (AFC-capable) |
| BT | BR/EDR/LE 5.4 |
| Reported SAR | Head 0.487 W/kg (10 g), body 1.388 W/kg — see [fairphone.md](fairphone.md) |

---

## 4. What this means for the custom-phone RF plan (feeds §7 of the decision summary)

1. **You do not pick a "modem".** On Snapdragon 7-series the baseband is in the SoC. Quectel/SIMCom modules only make sense with a non-cellular AP (i.MX/RK-class) — drop them.
2. **You inherit a Qualcomm companion-chip kit**, not decisions: RF transceiver (SDR7xx), PMICs (PM7550/PM8550-class), Wi-Fi/BT (WCN6xxx/7xxx), audio codec (WCD937x). These arrive with the reference design; they're BOM lines, not architecture work.
3. **The real engineering is (a) the RFFE and (b) the antennas**, and FP6 shows both are ODM work: T2 Mobile/TCL selected the PAMiD/DRx modules (Qualcomm QPM + third-party Skyworks-class parts), tuners/switches, and laid out 11 antennas + NFC, then ran the SAR/Smart-Transmit characterisation.
4. **Antenna count/placement is dictated by the band plan.** If the custom phone targets n41 HPUE + n77/78, budget 8 WWAN + 2–3 WLAN + NFC and give the mid-frame periphery to LDS antennas — which argues for the same plastic-overmoulded metal frame FP6 uses.
5. **Certification cost is a first-order item.** The FP6 FCC file is 160 documents / 459 MB from one lab; that plus CE/RED, carrier and PTCRB/GCF is the reason to buy the RF as a reference design through an ODM rather than design it.
