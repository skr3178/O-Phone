# Can You Actually Build a Phone From Scratch?

*One public FCC filing, two teardown videos, and our own attempt to spec a 10,000-unit
custom phone.*

## The spec this post orbits around

Current working selections (full decision log:
[custom_phone_design_decision_summary.md](custom_phone_design_decision_summary.md)):

| Subsystem | My selection | Working cost (@~10k units) |
|---|---|---|
| SoC | Qualcomm Snapdragon 7 Gen 4 (SM7750-AB, 4 nm)\* | $30–40 |
| RAM | 12 GB LPDDR5X | $10–15 |
| Storage | 256 GB UFS 4.0 | $15–22 |
| Display | 6.36" 1080×2400, 120 Hz AMOLED, matte glass option | $25–40 |
| Main camera | Sony LYTIA 808, 50 MP, OIS | $20–30 |
| Front camera | Samsung ISOCELL 3LU, 12 MP, autofocus | $3–7 |
| RF front-end | PA/PAMiD modules, tuners, switches — set by the band plan (§2) | $8–15 |
| Companion kit | RF transceiver + PMICs + Wi-Fi/BT + audio codec, platform-mandated (§2) | $12–20 |
| PMIC / power tree | matched to platform; not fully selected | $3–6 |
| Battery | 6000 mAh silicon-carbon, **removable** | $8–12 |
| Biometrics | Fingerprint in power button (Goodix), no Face ID | $1–3 |
| SIM | eSIM-only (no tray) | — |
| Charging | USB-C, 45 W PD | $2–6 (USB subsystem) |
| Audio | ≥3 mics, stereo speakers | $2–5 |
| Haptics | LRA actuator | $0.5–1.5 |
| Sensors | accel / gyro / mag / ambient light / proximity | $1–3 |
| Mechanical | chassis, frame, sealing, buttons, brackets | $15–30 |
| Thermal | graphite / heat spreader | $1–4 |
| OS | AOSP / custom Android, hardened (verified boot, TEE, SELinux enforcing) | — |
| **Bottom-up sum** | PCB/FPC/connectors still uncosted | **≈$157–260** |
| **Target** | complete hardware BOM @10k units (pre-2026-memory-surge; see §4) | **≈$170–220** |

\* Security caveat ([os.md](os.md) §2): ARM MTE unconfirmed on the 7 Gen 4 —
"disqualifying on its own" for our GrapheneOS-derived security spec — and no discrete
secure element in the BOM yet (StrongBox/Weaver). Verify both with the ODM before
locking the SoC. The gap between the sum row and the target row is the ODM negotiation (§5).

---

## 0. The tell in the FCC filing

- 2013 promise: "a fair, repairable phone." 2025 reality, hiding in the public record:
  the Fairphone 6's antenna report is authored by **T2 Mobile (Shanghai) — TCL's ODM
  arm** — measured at TCL's Ningbo lab. TCL-style part number: `BQA6CT0A15C0`.
- **Thesis: nobody builds a phone from scratch — not even Fairphone.** You enter at one
  of four altitudes (§6). This post walks the ladder with real photos, part numbers, and
  a real BOM.
- Why we can know: a US launch dumps its guts into the public record — 160 documents,
  459 MB, labeled antenna maps, naked-board photos. That filing + two teardowns + our
  own spec attempt = this post.

---

## 1. What's inside a modern phone

Specimen: Fairphone (Gen. 6) — €549, IP55, top iFixit repairability marks *[TODO:
verify the Gen-6 score; 10/10 is confirmed for Fairphone 5]*.

- **Module map:** mainboard (top 40%) · battery (middle) · daughterboard +
  speaker/vibe + USB-C module (bottom) · display · camera cover over two rear cameras,
  a ToF sensor, front camera, earpiece.
- **One screw type for the whole phone** (T5 Torx, ~26 screws). Battery swap ≈ 2 min,
  screen ≈ 5 min; no adhesive except the button-flex pads.
- **Battery trick:** bare pouch (4415 mAh, 3.91 V nom / 4.50 V limit, cell code
  1ICP6/61/67) glued to a stamped-steel carrier with screw ears — a floppy pouch becomes
  a screw-down part, no plastic shell.
- **Three interconnect classes, chosen by who breaks the joint:** B2B connectors →
  customer-swapped parts (battery, display, cameras, earpiece); pogo/spring contacts →
  technician-swapped parts (buttons, fingerprint, NFC, antennas, speaker, USB-C); coax →
  antennas (6+ MHF/IPEX receptacles on the mainboard; 3 colour-coded runs, silkscreened
  WHITE/RED/BLACK, to the daughterboard).
- **The mainboard has zero screws** — the camera cover's foam pads preload it onto pogo
  pins. And the third rear "camera" is a lie: it's the ToF sensor.
- Cost of repairability: IP55 not IP68, 9.6 mm thick, USB 2.0.

---

## 2. The silicon reality: you don't pick chips, you pick a platform

- Under the shields, a **Qualcomm solar system**: Snapdragon 7s Gen 3 (`SM7635`) with
  LPDDR5 stacked on top; around it the *mandatory companion kit* — `SDR735` RF
  transceiver · `PM7550` + `PM8550VS` PMICs · `WCN6755` Wi-Fi 6E/BT · `WCD937B` audio
  codec; Micron UFS 3.1 beside.
- **The modem is not a chip you choose** — it's in the SoC. The Quectel/SIMCom modules
  beginners (us included, §3) reach for pair only with non-cellular processors.
- What *is* chosen per design: the **RF front-end** — one Qualcomm QPM PA module, two
  PAMiD/receive modules (Skyworks-style `S55643-51`), tuners, switches — all dictated by
  the band plan.
- **11 antennas + NFC** in a €549 mid-ranger: 8 cellular (n41 and n77/78 each need four —
  4×4 MIMO/SRS), 3 Wi-Fi/BT IFAs, one 40.7 × 29.15 mm NFC loop; all laser-structured
  onto plastic around a metal core (hence the plastic-overmoulded frame).
- Even SAR is inherited: Qualcomm **Smart Transmit** power tables live in the modem's
  filesystem. You don't build this; you receive it.
- **Why you can buy a PC CPU but not a phone SoC:** PCs standardised OS↔hardware
  discovery (UEFI/ACPI), so any OS boots any board and the CPU is a retail commodity.
  Phones didn't — every SoC+board needs a bespoke BSP, so the chip is useless without
  it (gray-market Snapdragons on AliExpress prove it). Rockchip/Allwinner are the
  permissive end — which is exactly why §6's rung 3 exists. Full table:
  [SKR_learnings.md](SKR_learnings.md).

---

## 3. Our attempt: the spec sheet we wrote before we knew better

- Written before any teardown: 21 sections, SoC to sealing
  ([custom_phone_design_decision_summary.md](custom_phone_design_decision_summary.md)).
- Two reference phones inspired it: **Fairphone 6** (repairability template) and the
  **Jolla Phone** — Sailfish OS, privacy switch, replaceable 5450 mAh battery,
  assembled in Finland, €649–749, launching October 2026; proof a Dimensity 7100
  mid-ranger is enough and a tiny (ex-Nokia MeeGo) team can ship.
- Our picks vs. the phone we tore down (costs live in the top table):

| Subsystem | Our pick | Fairphone 6 |
|---|---|---|
| SoC | Snapdragon 7 Gen 4 | 7s Gen 3 — one gen older |
| RAM / storage | 12 GB LPDDR5X / 256 GB UFS 4.0 | 8 GB LPDDR5 / UFS 3.1 |
| Display | 6.36" 120 Hz AMOLED | 6.31" LTPO, 10–120 Hz |
| Main camera | Sony LYTIA 808, 50 MP, OIS | LYTIA 700C |
| Front camera | ISOCELL 3LU, 12 MP AF | 32 MP KD1 |
| Battery | **6000 mAh Si-C, removable** | 4415 mAh removable |
| Biometrics | Power-button fingerprint (Goodix) | same approach |
| SIM | **eSIM-only** | nano-SIM + eSIM + µSD |
| Charging | 45 W USB-C PD | 30 W, USB 2.0 |
| Haptics | LRA | coin ERM — a documented FP6 cost-cut |

- **Got right before the teardown:** removable battery, button fingerprint, modular
  ambitions, mid-range Snapdragon.
- **Got wrong:** a whole "pick a modem module" section (Quectel/SIMCom/Cat-1 bis) —
  meaningless on a Snapdragon phone; the modem is on-die and the real work is RFFE +
  an 8+3+NFC antenna system (§2). Spec §7 rewritten.
- **Voted against by both references:** our eSIM-only line (FP6: nano+eSIM+µSD; Jolla:
  dual nano, no eSIM). Flagged for revisit.
- **Still open, honestly:** RFFE, antennas, PMIC, Wi-Fi/BT, NFC, audio, haptics,
  thermal, sealing, BSP, certification — plus the two security-hardware gaps
  ([os.md](os.md) §2): MTE unconfirmed on 7 Gen 4, no secure element.

---

## 4. The money: BOM, royalties, and the certification wall

- **Bottom-up sum: ≈$157–260**, PCB/FPC uncosted (decision summary's own total:
  $159–267). The **$170–220 target is not the sum** — the gap is the ODM negotiation.
  Sanity check via Fairphone spares: battery €39.95 · display €89.95 · USB-C €19.95 ·
  camera €69.95.
- **The 2026 memory surge already broke it:** TrendForce (May 2026) — LPDDR5X contract
  prices +78–83% in 2Q26 (~$10/GB → ~$20/GB). Realistic now: 12 GB ≈ $40–60, 256 GB
  UFS ≈ $25–40 → component BOM **~$225–370**; landed @10k (assembly $8–15 + tooling
  amort $10–20 + cert amort $10–25) **~$255–430**. Memory, not the SoC, is the biggest
  line.
- **The royalty stack nobody puts in a launch deck:** buying the chip ≠ licensing the
  patents (Qualcomm's "no license, no chips" splits the two contracts). Cellular SEPs
  attach to the *device*: Qualcomm 3.25% of selling price (base capped at $400 →
  ≤~$13) · Nokia ≤€3 · Ericsson $2.50–5 · Huawei ≤$2.50 · InterDigital ~$1–2 →
  **~$20–25/device**, plus BT SIG (~$10k/product), Wi-Fi Alliance, HEVC (~$1–2; AV1
  free). Budget **~$25–35/device**. Contract nuance: white-label → ODM is licensee of
  record; semi-custom → *the brand* is liable. Put it in the contract.
- **Certification wall, with numbers:** enclosure tooling $50–150k; global cert stack
  (FCC + CE/RED + BIS + PTCRB/GCF, plus CTS/VTS if you ship GMS) ~**$100–250k per
  model** → $10–25/unit @10k. Scale: FP6's FCC campaign = 160 exhibits from one lab;
  SAR ~40 docs; n77/78 runs 8 parts. New since most write-ups: **EN 18031 (RED
  cybersecurity) mandatory in the EU since 1 Aug 2025** — FP6's DoC already cites it.
- **Software is a parallel BOM with payroll:** BSP + 8 years of updates = a staffed
  team. Google apps: small brands don't sign a MADA directly — path runs via a
  Google-authorized ODM/3PL + CTS/GTS/VTS + (since the 2018 EU ruling) a per-device
  EEA fee. De-Googled with commercial support = **Murena** (the €599 FP6 /e/OS SKU;
  also Teracube, Volla, SHIFT). Decide GMS vs. de-Googled **before** the BSP contract —
  it forks the plan.

---

## 5. The ecosystem: who actually builds phones (and how you'd buy in)

### 5.1 The ODM pyramid — who takes your call

- **ODMs** design and build most of the world's phones; brands do spec, story and
  distribution. The FP6's actual engineer is **T2 Mobile (TCL)** (§0). Qualcomm/MediaTek
  reference designs (QRD) are the starting point for everything.

| Tier | Who | Typical MOQ | Track record |
|---|---|---|---|
| 1 — giants | Huaqin, Wingtech, Longcheer (together ~70–76% of global ODM smartphone shipments, 2019–2023 counts), BYD Electronic, T2 Mobile, FIH (Foxconn) | 100k+ | Samsung A-series, Xiaomi, Honor, HMD; **BYD built the Nothing Phone (1)**; T2 built the FP6 |
| 2 — mid-size | Tinno (owns Wiko), Sprocomm (HMD/Nokia), Arima (built the Fairphone 3), Emdoor | ~10k–50k | the realistic band for a 10k-unit brand |
| 3 — Shenzhen "public model" (公模) houses | via Alibaba/Made-in-China; behind Unihertz, Teracube, Ulefone, Doogee | ~500–1k rebrand; ≥3k with hardware changes | Snapdragon 4 / Helio / Unisoc class only |

- The hype brands are this pyramid + marketing: OnePlus 1 = Oppo's factories with a
  community story; Nothing Phone (1) = a BYD design with a transparent back. Below the
  top-10 OEMs, nobody escapes the pyramid — you pick a tier and a story.
- **Component tier** (you specify, the ODM buys): BOE/TCL-CSOT/Tianma (displays) ·
  ATL/Sunwoda (batteries) · Sunny Optical/O-Film (cameras) · Skyworks/Qorvo/Murata
  (RFFE) · AAC/Goertek (acoustics/haptics). [TODO: verify]

### 5.2 How you engage — there is no shop

- **Tier 1–2 publish no catalogue** (checked: Tinno's and Sprocomm's "product" pages are
  category names + a phone number — reference designs embed NDA'd Qualcomm/MTK
  material). The menu is a sales process you qualify for: company brief + volume
  forecast → NDA → roadmap deck → samples → RFQ with your deltas → quote. Tier 1
  politely declines 10k-unit brands.
- **Public windows:** trade shows (MWC, Global Sources HK, Canton Fair) and Tier-3
  Alibaba listings — the only browsable specs/MOQ/prices (rebrand MOQ ~500–1k, ≥3k with
  hardware changes, per Alibaba's own buying guide).
- **Getting replies, in order:** named BD person on LinkedIn · trade-show booth (leave
  with WeChat contacts) · Alibaba "send inquiry" · Shenzhen sourcing agent · chip-vendor
  intro (needs credible volume) · Indian EMS (Dixon, Optiemus — built Nothing Phone (2a))
  if "Make in India" is the story.
- **Quote vocabulary:** MOQ (smallest run they'll accept) · NRE (one-time engineering/
  tooling fee for your changes) · FOB unit price (falls with volume) · **deltas** (your
  changes vs. their reference design — each one costs NRE and raises MOQ, so the art is a
  short list) · 30–50% deposit · tooling/ID/firmware ownership (make it yours).
- **The BOM's real job:** RFQ annex, cost yardstick and requirements record — not a
  shopping list. At 10k units you never hold a PO with Qualcomm; the ODM's platform
  catalogue *is* the SoC menu ([SKR_learnings.md](SKR_learnings.md)). BOM lines split
  into **spec** (you dictate: display, cameras, battery cell, enclosure, secure element)
  and **menu** (the platform bundle: SoC + PMIC + RFFE + companion kit + qualified
  memory combos).

### 5.3 The open-ish corner

- Pine64 (PinePhone), Purism (Librem 5), Fairphone, SHIFT, Teracube, Light Phone, Punkt,
  Jolla/Sailfish, /e/OS & Murena, LineageOS, postmarketOS.
- **Jolla, the counter-example to the Qualcomm default:** MediaTek SoC + Sailfish OS +
  Finnish assembly — an alternative stack (no GMS) alive 12+ years on niche volumes.
  fccid.io detail: Jolla's entire FCC history is one 2015 *tablet* filing — no Jolla
  phone was ever FCC-certified; the 2026 phone ships EU/UK/CH/NO only. [TODO: who does
  Jolla's hardware/ODM?]
- **The Sailfish nuance** ([os.md](os.md)): Jolla's business *is* OEM-licensing the OS —
  it runs on a normal Android BSP via libhybris; core open, but Silica UI and Android
  AppSupport **proprietary**. Our OS pick excluded it ("fails open source") for AOSP +
  optional /e/OS (Murena) SKU — irony: the values-inspiration company ships a
  part-closed OS. GrapheneOS's hardware requirements are our security spec instead
  (MTE + secure element = the two real gaps).
- **Sailfish pricing:** ~€50 one-off consumer licence (AppSupport, Exchange, VoLTE);
  ~€5/month subscription from Sailfish 5 (perpetual option retained); OEM royalty
  negotiated, unpublished. It buys a differentiated UI — not a way around the BSP.

---

## 6. The four entry paths

1. **White-label / rebrand** — ODM catalogue phone, your logo + firmware skin.
   ~$50k–200k, MOQ 0.5–10k (Alibaba public-model tier starts ~500–1k), ~6 months. You
   own brand + distribution. (Punkt, early Light-Phone-ish.)
2. **Semi-custom ODM** — your ID/mechanical on reference-design electronics; ODM does
   RF, antennas, certification. The **Fairphone route** — and ours. ~$1–5M, MOQ 10k+,
   12–24 months.
   - *Rung 2½ — Android smart-module route:* the one place "own board" is purchasable
     at 1–1,000 units: carrier board + QCM6490-class module (Quectel SG560D, Fibocom
     SC171, Thundercomm C6490; RB3 Gen 2 kit ~$399). Module vendor holds the Qualcomm
     license, ships the Android BSP, module is RF pre-certified (FCC/CE, often PTCRB) —
     collapses most of §4's cert wall for prototypes. Trade: last-gen silicon, ~$100+
     module, thick product.
3. **Linux-phone route** — non-Qualcomm AP (i.MX/Rockchip) + modem module
   (Quectel/Broadmobi) + community OS. No Qualcomm NDA, genuinely open — but 2016-era
   performance, no GMS. (PinePhone ~$150 BOM-class; Librem 5 — $2M+ and years late.)
   Where our original Quectel/Cat-1 bis idea actually lives.
4. **From scratch** — own board, RF, antennas, cert program. Nobody outside the top ~10
   OEMs; even they lean on Qualcomm reference designs.

**Our 10k-unit phone lands on path 2**, path 3 as the ideological fallback.

---

## 7. Feasibility verdict

- **Yes, you can "build a phone"** — for the rung-1–3 definitions of *build*. Capital:
  rung 1 low six figures · rung 2 low seven figures · rung 3 seven figures + a
  community · rung 4 don't.
- Survivors are values-based niches: Fairphone (repairability + EU regulation
  tailwind), Light Phone & Punkt (minimalism), Jolla (pivoted to B2B licensing;
  consumer phone returning October 2026). Graveyard: Essential — $330M raised, dead in
  2.5 years. *[TODO: 1–2 more graveyard cases with numbers]*
- **Rung-1 fingerprint, in our own folder** ([lightphone_specs.md](lightphone_specs.md)):
  Light Phone III = Qualcomm SM4450, Light Flip = MediaTek MT8873 — yet both share
  6 GB/128 GB, 50 MP sensor, 1800 mAh, IP54. Two SoC vendors, identical supporting
  spec: public-model sourcing with a custom OS on top.
- You cannot out-spec BBK/Samsung/Apple at any realistic volume; you can out-*value*
  them (repairability, privacy, minimalism).
- Closing: the FCC filing that started this post is the beginner's best textbook —
  every US phone's guts are public at fccid.io. Go read one. (Just not the Jolla's —
  the EU-only phone has never crossed the FCC's desk.)

---

## 8. Resources & further reading

- FP6 FCC filing: <https://fccid.io/2AUWUFP6> (grant 2025-08-24, photos public since 2026-02-21)
- iFixit FP6 teardown (video bXseyTdynCo) + repair manuals; JerryRigEverything durability (ov752bRItA0)
- Fairphone spec/impact pages; Fraunhofer lifetime-CO₂ study (5 yr use ≈ −⅓ footprint)
- Qualcomm Snapdragon 7-series product briefs (7s Gen 3 / 7 Gen 4 SM7750-AB)
- Our deep-dive docs: [teardown analysis](fairphone6_video_teardown_analysis.md) ·
  [FCC RF analysis](fairphone6_fcc_rf_analysis.md) ·
  [decision summary / BOM](custom_phone_design_decision_summary.md)
- ODM engagement: Google's certified-partner ODM list <https://www.android.com/certified/partners/> ·
  Counterpoint ODM/IDH landscape <https://www.counterpointresearch.com/insights/wingtech-huaqin-longcheer-continued-dominate-worlds-smartphone-odmidh-landscape-2019/> ·
  Tinno <https://en.tinno.com> · Sprocomm showroom <https://www.made-in-china.com/showroom/jayangun/> ·
  Alibaba OEM-vs-ODM buying guide <https://electronics.alibaba.com/buyingguides/custom-brand-smartphone-guide-oem-vs-odm-vs-diy>
- Sailfish OS licence terms/pricing: <https://docs.sailfishos.org/Support/Help_Articles/Sailfish_OS_Licence/>
- Memory price surge: TrendForce, 14 May 2026 <https://www.trendforce.com/presscenter/news/20260514-13044.html>
- SEP royalties: Qualcomm 5G handset licensing program (qualcomm.com PDF); published Nokia/Ericsson/Huawei/InterDigital rate caps
- Smart modules: Thundercomm C6490 dev kit <https://www.thundercomm.com/product/c6490-development-kit> ·
  Quectel SG560D <https://www.quectel.com/product/5g-sg560d-smart-module-series>
- *[TODO: Pine64/Purism postmortems, PTCRB/GCF fee schedules, EU repairability-index links]*

---

## Appendix candidates (may cut)

- Full antenna→band→power table (from SAR Part-0/Part-1)
- Full silicon ID table with confidence levels
- Disassembly order / fastener map table

---

## Draft status (internal — cut before publishing)

> **Status:** condensed draft — bullets + tables, peer-review fixes folded in
> (reconciled BOM table, 2026 memory-surge update, SEP royalty stack, cert-stack costs,
> GMS/MADA path, smart-module rung 2½, Jolla FCC detail, Light-Phone rung-1 case).
> **Remaining TODOs:** Jolla's ODM identity · Pine64/Purism postmortems · graveyard
> case numbers · component-tier verification · BYD/Nothing primary source · FP6 iFixit
> score check.
> **Working titles:** "Nobody Builds a Phone From Scratch (Not Even Fairphone)" ·
> "An Engineer's Field Guide to Building a Phone" · "The Four Ways Into the Phone Business"
> **Audience:** engineer-curious reader; payoff = honest feasibility verdict for a small team.
> **To make:** 2 original SVG diagrams — (1) architecture block diagram, (2) entry-path ladder.
> **Image licensing:** ✅ FCC exhibits (public record, carry the post) · ⚠️ iFixit stills
> (CC BY-NC-SA — attribute + same-license note) · ❌ JerryRig stills (internal only, link
> the video).
> **Evidence base:** FP6 teardowns (iFixit, JerryRigEverything) · FCC filing 2AUWUFP6
> (160 exhibits) · our 10k-unit BOM · reference phones: [fairphone.md](fairphone.md),
> [jollaPhone.md](jollaPhone.md).
> **Deep dives:** [fairphone6_video_teardown_analysis.md](fairphone6_video_teardown_analysis.md) ·
> [fairphone6_fcc_rf_analysis.md](fairphone6_fcc_rf_analysis.md) ·
> [custom_phone_design_decision_summary.md](custom_phone_design_decision_summary.md)
