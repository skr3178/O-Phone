# Fairphone (Gen. 6) — Teardown Video Analysis: Components & Design

**Sources (downloaded 2026-08-15 into `videos/`):**

| File | Video | Length | Useful range |
|---|---|---|---|
| `videos/ifixit_fp6_teardown.mp4` | iFixit — "The Most Ethical Smartphone Yet? Fairphone 6 Teardown & Review" (bXseyTdynCo) | 5:22 | 1:05–4:15 teardown |
| `videos/jerryrig_fp6_durability.mp4` | JerryRigEverything — "They said it was the best... (Fairphone 6 Durability Test)" (ov752bRItA0) | 11:27 | 0:35–5:40 teardown · 8:15–11:10 durability |

Transcripts: `videos/*_transcript.txt`. Frame contact sheets: `videos/sheets/`. Full-res key frames: `videos/key/`.
Cross-reference with [fairphone.md](fairphone.md) (spec sheet) and the FCC internal photos in [FCC_2AUWUFP6_Fairphone_Gen6/Internal-Photos/](FCC_2AUWUFP6_Fairphone_Gen6/Internal-Photos/).

---

## 1. Disassembly order & fastener map (what "modular" actually means here)

Every fastener in the phone is a **T5 Torx**. Total ≈ 26 screws + one clip-on cover. Nothing is glued except the pogo-pin contact pads on the frame (see §3).

| Step | What comes off | Retention | Time (JRE) | Frame |
|---|---|---|---|---|
| 1 | Lower back cover (bottom ~60 %) | 2× T5 (visible from outside) + slide | — | iFixit 1:06 |
| 2 | Battery | 4× T5 through steel bracket ears + 1× T5 on a small plastic cover; B2B connector on short flex | ~2 min | iFixit 1:16–1:30 |
| 3 | Upper back cover (camera bump section) | Clips only — pry off once battery is out | — | iFixit 1:42 |
| 4 | Display | 8× silver T5 around perimeter → lifts out; 1× T5 + bracket on display B2B under the panel | ~5 min | iFixit 1:52–2:16 |
| 5 | Bottom speaker/vibrator module | 5× black T5 | — | JRE 2:51 |
| 6 | USB-C port module | 2× T5 tabs; press-fit metal cage, rubber IP gasket | — | JRE 3:13 |
| 7 | Secondary PCB (daughterboard) with SIM/microSD reader | screws + B2B + 2 coax | — | iFixit 3:04 |
| 8 | Camera cover ("top plastics") — carries NFC coil, LDS antennas, ToF+flash flex | 6× T5 | — | JRE 3:17 |
| 9 | Front camera, earpiece speaker, ultrawide, main camera | B2B connectors only, "snap in like Lego" | — | JRE 4:05–4:35 |
| 10 | Mainboard | **No screws** — held by the camera cover; lifts straight out; 3 coax + pogo pins | — | JRE 5:11 |
| 11 | (Not user-serviceable) Side-button/fingerprint flex, volume flex, top-mic flex on pogo pads | Glued to frame | — | iFixit 4:00 |

Design observations:
- The **back cover is split in two**: a screwed lower panel and a clip-on upper panel. The two visible screws double as the structural tie for the whole rear — JRE's bend test showed "those two rear screws holding on for dear life" (10:56).
- **Battery-out-first is the gate** for everything else: the upper cover clips are only accessible once the battery leaves.
- Screen replacement does not require touching any adhesive → sacrifices ingress rating (IP55 rather than IP68).
- Fairphone 5 needed no tools for the battery; FP6 needs a T5 driver. Fairphone accepted the tool for a slimmer, stiffer, better-sealed housing.

---

## 2. Component inventory (what's actually inside)

### 2.1 Mainboard (silkscreen: `FP SPRING BAP6CT000FC1`)
Frame: `videos/key/jr_mainboard_front.jpg`, `ifx_mainboard_and_camcover.jpg`, `jr_mainboard_back_a/b.jpg`

- Small, **≈62 × 54 mm L-shaped** board (measured against the ruler in the FCC photos) occupying only the top ~40 % of the phone; cut-out at top centre for front camera + earpiece.
- **Two large soldered RF shield cans** on the top face (SoC/DRAM/PMIC and modem/RF areas — no lids removed in either video), plus a smaller can top-right. Fully shielded on the back face as well.
- **Connectors on the top face:** B2B for main camera (`PNNS4307` label), ultrawide, front camera (`DH2515 TSP A1`), earpiece speaker; two B2B at the bottom edge for the display flex and the mainboard→daughterboard interconnect flex; battery B2B on the right.
- **Three coax (MHF/IPEX) connectors** at bottom-right, **silkscreened WHITE / RED / BLACK** — cables are colour-coded jackets so a repairer cannot swap them (JRE 4:52: "never seen that before"). They run down the right side to the daughterboard antennas.
- **Pogo pins on the mainboard** (spring pins along the left edge and top-left, visible in `jr_mainboard_back_b.jpg`) mate to gold pads on flexes glued to the frame (§3).
- **DRC map "tattoo"** (Democratic Republic of Congo outline) printed on the shield can — Fairphone branding, not functional.
- Reported: recycled tin & copper in the PCB.

### 2.2 Cameras
- **Main 50 MP** (spec: Sony LYTIA 700C, 1/1.56") — large OIS module, ~11 × 11 mm footprint (JRE 4:09), B2B on the module itself.
- **Ultrawide 13 MP** — smaller fixed module, no OIS (JRE 4:19).
- **Front 32 MP** (Samsung KD1) with AF — sits in the frame cut-out above the mainboard, B2B, removable "like a little Lego" (JRE 4:23).
- **Third rear "camera ring" is a ToF depth sensor**, not a camera. It plus the **LED flash** live on a small flex (`ToF_Flash_FPC V0.0`) mounted *inside the camera cover*, not on the mainboard (`jr_camcover_nfc.jpg`). Removing the cover disconnects a single B2B for both.

### 2.3 Camera cover / "top plastics" (`0504-132U`, `V0.7 FP6`)
This one plastic part is doing a lot of work — it's the closest thing to an antenna carrier in the phone:
- **NFC coil** (black square with pickup contacts at the top-left edge; two spring pads).
- **LDS (laser-direct-structured) antenna traces** — the copper meander patterns visible on the *outside* face of the cover in `ifx_mainboard_and_camcover.jpg` (top-right and bottom-left corners). These couple to the mainboard via spring contacts.
- Glass lens windows for both cameras and the ToF; clear dome for the flash.
- The ToF/flash flex (§2.2).
- Foam/gasket pads that press the mainboard down (this is what holds the mainboard — no mainboard screws).

### 2.4 Battery (`videos/key/ifx_battery_label.jpg`)
Read directly off the label:

| Field | Value |
|---|---|
| Model | **F6FPHN** |
| Typical / rated | **4415 mAh / 17.26 Wh** — 4315 mAh / 16.87 Wh |
| Nominal voltage | **3.91 V** |
| Charge limit | **4.50 V** (high-voltage cathode chemistry) |
| Cell code | **1ICP6/61/67** → single pouch, ~6.0 mm × 61 mm × 67 mm |
| Mechanical | Pouch bonded to a thin **stamped-steel carrier plate** with 4 screw ears (sides) + 2 locating ears (top) |
| Connector | Small B2B on a short flex at the top edge (not spring contacts) |
| Other | Round white liquid-damage indicator sticker in the battery bay (JRE 1:45) |

iFixit: "battery is glued to a thin strip of metal designed to be light in weight and resource consumption, while sturdy enough to protect a lithium-ion pouch from unintended impact." The metal plate turns a floppy pouch into a screw-down FRU without a plastic shell.

### 2.5 Display
- **6.31" LTPO OLED, 10–120 Hz, BOE panel** (flex printed `BOE`, `jr_display_back.jpg`), 1400 nits peak, Gorilla Glass 7i.
- Display is bonded to its own **stamped metal backing plate** (steel/aluminium) with a **black graphite sheet** on the rear; the whole sub-assembly is what the 8 screws clamp.
- Flex exits at the bottom-centre, passes through a slot in the mid-frame, and lands on a B2B on the mainboard held by a **screwed bracket** — that bracket is the only "hidden" step in a screen swap.
- Durability: Mohs 6 scratches / 7 grooves (normal for GG7i); OLED survived ~35 s of open flame and mostly recovered (JRE 10:40).

### 2.6 Bottom speaker / vibrator module (`jr_speaker_module.jpg`)
- One black plastic carrier holds: **loudspeaker box** (mesh-covered), **coin-type vibrator motor** (`B54GX1`, ~10 mm ERM/coin — not an LRA), a **mic port with mesh gasket**, and **two pairs of spring contacts** (speaker + vibrator). Zero connectors on this module — it lands on pads on the daughterboard.
- Speaker is a "no balls inside" box (no acoustic mass beads) — cost-optimised.

### 2.7 USB-C port module (`jr_usbc_module.jpg`)
- **USB 2.0 only**, 30 W charging. Receptacle in a stamped-steel bracket, **knurled spring tab** for grounding, two screw tabs, **black rubber O-ring** on the barrel for IP55. Contacts to the daughterboard by press-fit/pads. Spare part €19.95.
- iFixit/JRE both flag USB 2.0 as the obvious BOM-cost cut.

### 2.8 Secondary PCB / daughterboard (`FP6 SecondaryPCB V04 BAH6CT000FC1`, `ifx_daughterboard.jpg`)
- Carries the **combined nano-SIM + microSD tray reader** (2 TB max), one B2B to the interconnect flex, gold pads for the USB-C module and speaker/vibe module, **large gold antenna spring-contact areas** on the board edges, and the landing for two of the coax cables (bottom antennas).
- Underneath it in the frame: a **conductive fabric mesh** for the bottom microphone port and grounding.

### 2.9 Earpiece / top
- **Modular earpiece speaker** module with waterproofing mesh, B2B (JRE 4:35). Stereo pair with the bottom speaker.
- Small noise-cancelling mic hole on the top edge; **no antenna lines** because the outer body is plastic (JRE 8:49).

### 2.10 Frame, thermal, and grounding (`jr_frame_thermal.jpg`, `ifx_pogo_pins.jpg`)
- **Metal mid-frame** (marked `FP6 V0 YQ M1-2`), die-cast, with a plastic overmoulded outer wall — the plastic is what you touch on the sides. JRE calls it aluminium; the spec's recycled-materials list mentions both aluminium and magnesium.
- **Thermal path is minimal:** a large **light-green soft thermal foam pad** on the mid-frame under the mainboard shield cans, plus **graphite on the display side** of the frame. No vapour chamber, no heat pipe. Fine for a Snapdragon 7s Gen 3.
- **Woven conductive-fabric gaskets** (grey rectangles) at several points on the frame = EMI grounding between shield cans and frame.
- Sides are ~9.6 mm thick — the frame is thick enough that JRE says it "does have some flex" but survives.

### 2.11 Buttons & side switch
- Right side: **power button with integrated capacitive fingerprint reader** and the **lime-green "Fairphone Moments" slider switch**; left side: two volume buttons (which can pop *out* of the frame and are hard to reseat — JRE 9:05).
- These are on **flexes glued to the mid-frame** with gold pads → mainboard pogo pins (§3).

### 2.12 Back panel accessories
- Lower back cover has a **recessed mounting area** with threaded inserts; Fairphone sells a card holder, finger loop, lanyard screws, and cases that bolt to it (JRE 1:09–1:23). Boxes carry braille.

---

## 3. Interconnect architecture — the key design idea

The FP6 uses **three interconnect classes**, chosen by how often the joint is expected to be broken:

| Class | Where used | Why |
|---|---|---|
| **B2B (board-to-board) connectors** | Battery, display, all 3 cameras, earpiece, ToF/flash flex, mainboard↔daughterboard interconnect | User-serviceable parts: reliable, low-profile, tolerant of many mating cycles |
| **Spring/pogo contacts** (pads on flex glued to frame, pins on the boards; or pads on modules) | Side-button/fingerprint flex, volume flex, top-mic flex, NFC coil, LDS antennas, speaker/vibe module, USB-C module | Removes ~6 connectors and their screws; module simply drops in. **Downside (iFixit 4:06):** the frame-side pads are glued and the flexes are delicate → button/fingerprint repair is fiddly |
| **Coax (3×, colour-coded)** | Mainboard → daughterboard antennas | Standard; colour coding + silkscreen makes them idiot-proof |

The mainboard therefore has **no screws of its own** — the camera cover's 6 screws and foam pads preload the pogo pins. That's a deliberate trade: fewer fasteners, but the mainboard's contact pressure depends on a plastic part.

---

## 4. Durability results (JRE 8:15–11:10)

| Test | Result |
|---|---|
| Screen scratch | Mohs 6 marks, 7 grooves — normal Gorilla Glass 7i |
| Sides / back | Plastic; scratches with a blade. Camera + ToF windows are glass |
| Buttons | Volume buttons can be pried out of the frame; don't re-seat well |
| Screen flame | ~35 s to white-out, OLED mostly recovered |
| Bend | Visible flex from back and more from front; **no crack, no separation** — the two rear cover screws + 8 display screws + metal mid-frame carry it |
| Ship colour | Green tint theme on by default (software) |

Net: IP55, IEC 60068-2-31 1.8 m drop, MIL-810H 1.5 m — screws instead of adhesive cost some stiffness/sealing but not enough to fail the bend test.

---

## 5. Design lessons mapped to the custom-phone BOM

Cross-referenced against [custom_phone_design_decision_summary.md](custom_phone_design_decision_summary.md).

### 5.1 Battery (your target: 6000 mAh Si-C, removable)
- **Copy the steel-carrier idea.** FP6 gets a screw-down FRU by laminating the pouch to a thin stamped plate with screw ears — no plastic shell, no wasted volume. A 6000 mAh Si-C pouch will be ~5.5–6 mm thick and larger in area; the plate also stiffens it against puncture.
- **FP6 uses a 4.50 V-limit cell (3.91 V nominal).** Your PMIC/charger and fuel gauge must be spec'd for the actual cell chemistry — Si-C cells commonly have a different voltage profile; don't reuse Li-ion default charge tables.
- Use a **B2B on a short flex** (like FP6), not spring contacts, for the battery — high current + user replacement favours a positive-latch connector.
- Put a **liquid-damage indicator** in the battery bay; it's a warranty tool.
- FP6 does not have wireless charging; if you want it, the coil has to go in the lower back cover with a pad interface, since that cover is removable.

### 5.2 Mechanical/enclosure
- **Two-piece back cover** (screwed lower + clip-on upper) is a neat trick: two visible screws only, and the upper section hides the camera/antenna carrier without more screws. If you're not chasing IP68, this is the lowest-cost route to a serviceable rear.
- **Single fastener type everywhere** (T5). Pick one driver and stick to it — repair-manual simplicity and one bit in the box.
- **Metal die-cast mid-frame with plastic overmould** — gives stiffness plus antenna-transparent sides with no antenna slot lines. Directly relevant if you're going eSIM-only with an unlocked cellular architecture (§7 of your BOM): plastic sides mean your RF FE can use LDS on inner plastic parts, no metal-band antennas.
- Screen held by 8 perimeter screws on a **metal backing plate** — separates "display module" from "frame". Cheaper to service, at the cost of IP rating and ~0.5 mm thickness. Your 6.36" AMOLED could use exactly this construction.

### 5.3 Electronics/architecture
- **Mainboard is small (~62×54 mm, top of phone) + a secondary PCB at the bottom + one interconnect flex.** Your Snapdragon 7 Gen 4 board can be laid out the same way; put SIM/eSIM, USB-C, speaker/vibe, mic and bottom antennas on the daughterboard so the expensive board never gets touched in a port repair.
- **Colour-code and silkscreen the coax** — trivial cost, big repair win.
- **Decide consciously between pogo pads and B2B per module.** FP6's rule of thumb: B2B for anything a customer will swap; pogo for things replaced only by a technician (buttons, fingerprint, antennas). Since your fingerprint sensor is in the power button, note that iFixit specifically calls that repair "fiddly" on FP6 because of the glued pad flex — consider a B2B for the side-key/fingerprint flex instead.
- **Thermal:** a soft foam pad + graphite is enough for a 7-series SoC. Don't budget for a vapour chamber unless sustained loads (your AI-chip use case?) demand it.
- **Haptics:** FP6 uses a cheap coin ERM on the speaker module. If you want better haptics, budget an LRA (~$1–2 more) — it's one of the few places FP6 obviously cut cost.
- **USB:** FP6 shipped USB 2.0 and both reviewers dinged it. Your 45 W PD target should pair with at least USB 3.x-capable wiring on the daughterboard/interconnect flex; make the port a discrete module with a gasket like FP6.

### 5.4 Cameras
- Third "camera" ring is a **ToF sensor sharing a flex with the flash inside the camera cover**. Nice packaging: keeps small optical parts off the mainboard. Your Sony LYTIA 808 + front 12 MP plan is simpler (no ToF), so you can drop this flex entirely.
- All three camera modules on the FP6 are individually B2B'd and individually sold as spares (€69.95 main, €34.95 selfie) — worth mirroring for your BOM/service pricing.

### 5.5 Sustainability/positioning notes surfaced in the videos
- Fraunhofer: keeping the phone 5 yr instead of 3 cuts footprint ~⅓ — the whole business case for modularity.
- FP6 spare-part pricing (battery €39.95, screen €89.95, USB-C €19.95, back cover €24.90) sets the market expectation if you sell parts.
- Loyalty program that rewards repairs/ownership years — a software/service idea, zero BOM cost.

---

## 6. Open items / things the videos do *not* show
- Neither video lifts the RF shield cans → SoC/DRAM/PMIC/modem part numbers not visible. **Resolved in [fairphone6_fcc_rf_analysis.md](fairphone6_fcc_rf_analysis.md)** from the FCC internal photos (SM7635 + SDR735 + WCN6755 + PM7550/PM8550VS + WCD937x, 11 antennas + NFC, ODM = T2 Mobile/TCL).
- Antenna count/placement — **resolved**: 8 WWAN + 3 WLAN + NFC, see FCC doc §2.
- Whether the mid-frame is Al or Mg — JRE says aluminium; the spec's recycled-material list mentions both.
- JRE's caption reads "screen is €9" — auto-caption error; the spec sheet says €89.95.

## 7. Key frames index (`videos/key/`)
| File | Shows |
|---|---|
| `ifx_mainboard_and_camcover.jpg` | Mainboard top face + camera cover with LDS antenna traces |
| `jr_mainboard_front.jpg` | Mainboard with all camera/earpiece modules populated |
| `jr_mainboard_back_a.jpg`, `_b.jpg` | Fully shielded back face, pogo pins on board edge |
| `jr_coax_colorcoded.jpg` | WHITE/RED/BLACK coax silkscreen, camera B2Bs |
| `ifx_pogo_pins.jpg` | Three glued pad-flex groups on frame (buttons/fingerprint, volume, top mic) |
| `jr_frame_thermal.jpg` | Mid-frame `FP6 V0`, thermal foam, EMI fabric gaskets |
| `jr_camcover_nfc.jpg` | Camera cover: NFC coil, ToF+flash flex, lens windows |
| `ifx_battery_label.jpg` | Battery label: F6FPHN, 4415 mAh, 3.91 V, 4.50 V, 1ICP6/61/67 |
| `jr_display_back.jpg` | BOE flex, display B2B + bracket, front cam/earpiece bay |
| `jr_speaker_module.jpg` | Speaker + coin vibrator + spring contacts + mic gasket |
| `jr_usbc_module.jpg` | USB-C module: steel cage, O-ring, ground tab |
| `ifx_daughterboard.jpg` | Secondary PCB with SIM/microSD reader, antenna pads |
| `jr_sim_tray.jpg` | Combined nano-SIM + microSD tray |
| `jr_earpiece.jpg`, `jr_maincam.jpg` | Earpiece module; 50 MP OIS module |
