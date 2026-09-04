# Custom Phone Design --- Decision Summary

**Status:** Working BOM / architecture summary\
**Date:** 2026-08-14

> This document consolidates the phone-design decisions and options
> discussed so far. "Selected" means the current working choice, not
> necessarily a production-qualified part. Costs are approximate working
> estimates at \~10k-unit scale where previously discussed.

## 1. Core compute

  -------------------------------------------------------------------------------
  Part              Options considered Current selection Why
  ----------------- ------------------ ----------------- ------------------------
  SoC               Snapdragon-class   **Qualcomm        Strong
                    mobile SoCs;       Snapdragon 7 Gen  performance/efficiency
                    Snapdragon 7 Gen 4 4, 4 nm**         balance for a \~10k-unit
                    was evaluated as                     custom phone; provides
                    the target                           an integrated smartphone
                                                         platform rather than
                                                         building around a
                                                         generic processor.
                                                         **Working cost:
                                                         \~\$30--40**

  RAM               Different LPDDR    **12 GB LPDDR5X** Good
                    configurations                       performance/longevity
                                                         without pushing BOM
                                                         excessively. Target
                                                         discussed was \~8533
                                                         MT/s, but the exact
                                                         supported speed needs
                                                         validation against the
                                                         selected platform.
                                                         **Working cost:
                                                         \~\$10--15**

  Storage           Different          **256 GB UFS      High-speed modern phone
                    capacities /       4.0**             storage while keeping
                    storage                              capacity and cost
                    technologies                         reasonable. **Working
                                                         cost: \~\$15--22**
  -------------------------------------------------------------------------------

## 2. Display

  ---------------------------------------------------------------------------------
  Part              Options considered          Current selection Why
  ----------------- --------------------------- ----------------- -----------------
  Display           Different phone             **6.36-inch,      Good balance of
                    sizes/resolutions/refresh   1080×2400, 120 Hz size, sharpness,
                    rates                       AMOLED**          refresh rate,
                                                                  power consumption
                                                                  and cost.
                                                                  **Working cost:
                                                                  \~\$25--40**

  ---------------------------------------------------------------------------------

## 3. Cameras

### Rear/main camera

  -----------------------------------------------------------------------
  Options considered      Current selection       Why
  ----------------------- ----------------------- -----------------------
  High-end Sony/Samsung   **Sony LYTIA 808, 50    Strong main-camera
  mobile image sensors;   MP, OIS**               capability without
  OIS-capable                                     moving to a much more
  configurations                                  expensive flagship
                                                  sensor stack. **Working
                                                  cost: \~\$20--30**

  -----------------------------------------------------------------------

### Front camera

  -----------------------------------------------------------------------
  Options considered      Current selection       Why
  ----------------------- ----------------------- -----------------------
  Various compact selfie  **Samsung ISOCELL 3LU,  Selected as the front
  sensors                 12 MP, autofocus**      camera sensor;
                                                  autofocus is useful for
                                                  selfie/video use.
                                                  **Working cost:
                                                  \~\$3--7**

  -----------------------------------------------------------------------

## 4. Battery

  -----------------------------------------------------------------------
  Options considered      Current selection       Why
  ----------------------- ----------------------- -----------------------
  Conventional            **6000 mAh              High energy density and
  Li-ion/Li-polymer       silicon-carbon          large capacity while
  pouch; different        removable battery**     retaining a
  capacities;                                     **removable/modular**
  modular/scalable                                battery concept.
  battery approaches;                             **Working cost:
  silicon-carbon                                  \~\$8--12**
  chemistry                                       

  -----------------------------------------------------------------------

For the battery architecture, the discussion emphasized selecting the
cell based on required capacity first, then optimizing dimensions,
chemistry, voltage/current capability, connector, protection, mechanical
retention and thermal constraints.

## 5. Biometrics

  -----------------------------------------------------------------------
  Options considered      Current selection       Why
  ----------------------- ----------------------- -----------------------
  In-display fingerprint; **Single side-mounted   Lower complexity/cost
  side capacitive         capacitive fingerprint  than an advanced
  fingerprint; facial     sensor integrated into  under-display or Face
  biometric hardware      the power button;       ID-style system, while
                          Goodix preferred**      retaining a
                                                  conventional fast
                                                  fingerprint unlock.
                                                  **Working cost:
                                                  \~\$1--3**

  Face ID / dedicated     **Not selected**        Avoids additional
  facial biometric                                sensors, processing,
  hardware                                        mechanical complexity
                                                  and BOM cost.
  -----------------------------------------------------------------------

## 6. SIM / cellular identity

  ------------------------------------------------------------------------
  Options considered      Current selection       Why
  ----------------------- ----------------------- ------------------------
  Physical SIM + eSIM;    **eSIM-only**           Eliminates physical SIM
  physical SIM only; eSIM                         tray/slot and associated
                                                  mechanical/RF/assembly
                                                  complexity.

  eSIM implementation     **Very small eSIM IC is Earlier work focused on
                          preferred**             minimizing the eSIM
                                                  footprint.
  ------------------------------------------------------------------------

## 7. Cellular modem / RF

**Status: modem is resolved (integrated in the SoC). The open items are the
RF front-end (RFFE) and the antenna architecture — both ODM/reference-design
work, not part selection from scratch.**

### 7.1 Modem — no longer a gap

-   The selected SoC (**Snapdragon 7 Gen 4, SM7750-AB**) carries the
    **Snapdragon 5G Modem-RF System on-die** (sub-6 GHz SA/NSA, 3GPP Rel-17,
    up to 4.2 Gbps DL, DSDA 5G+5G/4G) — per Qualcomm's product brief.
-   Reference point: the Fairphone 6 (Snapdragon 7s Gen 3, SM7635) FCC
    internal photos show exactly this split — baseband inside `SM7635`, with
    a discrete Qualcomm **RF transceiver `SDR735`**, PMICs **`PM7550` +
    `PM8550VS`**, Wi-Fi 6E/BT **`WCN6755`**, audio codec **`WCD937x`**
    around it. See [fairphone6_fcc_rf_analysis.md](fairphone6_fcc_rf_analysis.md).
-   Consequence: the external cellular modules previously listed (Quectel
    EG800Q/EG800AK/EG810M/EG25-G, SIMCom A7670, Cat-1 bis) are **dropped**.
    They only make sense with a non-cellular AP; a Snapdragon design does
    not use them and Android/BSP support assumes the integrated modem.

### 7.2 What comes with the platform (BOM lines, not decisions)

| Block | 7 Gen 4 companion (expected) | FP6 equivalent seen in FCC photos |
|---|---|---|
| RF transceiver | Qualcomm SDR7xx-class | `SDR735` |
| PMIC(s) | PM7xxx main + PM8550-class companion(s) | `PM7550`, `PM8550VS`, +1 |
| Wi-Fi/BT | Qualcomm FastConnect (Wi-Fi 7 / BT 6.0) WCN7xxx-class | `WCN6755` (Wi-Fi 6E) |
| Audio codec | WCD93xx | `WCD937B` |
| SAR control | Qualcomm Smart Transmit (time-averaged) | used; per-antenna Plimit tables in EFS |

**Working cost for the companion set (transceiver + PMICs + WCN + codec):
~\$12–20**, delivered as part of the Qualcomm reference design through the
ODM.

### 7.3 The actual gap #1 — RF front-end

Still to be locked, driven by the **band plan** (EU + US carriers assumed):

-   PA / PAMiD modules for LB / MB / HB / n77-78 (Qualcomm QPM/QTM or
    Skyworks/Qorvo/Murata equivalents). FP6 uses one Qualcomm QPM-series
    module + two identical third-party PAMiD/DRx modules (Skyworks-style
    marking `S55643-51`) inside two shielded RF bays.
-   Diversity-receive modules, antenna tuners (QAT-class), antenna switch
    modules, envelope tracker (QET-class) — FP6 shows ~6 small
    Qualcomm-marked RFFE parts around the transceiver.
-   **HPUE (PC2) on n41** if US carriers matter — FP6 certifies it (26.5 dBm
    tune-up); it sets the PA choice and thermal/SAR budget.
-   **Working cost: ~\$8–15** for the RFFE set at 10 k units, higher if
    the band list is long.

### 7.4 The actual gap #2 — antenna architecture

FP6's certified layout is the template (antenna map + SAR tables in the
FCC file):

-   **8 WWAN antennas + 3 Wi-Fi/BT IFAs + NFC loop (40.7 × 29.15 mm)** = 11
    + NFC. n41 uses 4 antennas (Ant 0/4/5/6) and n77/78 uses 4 (Ant
    2/3/4/8) for SRS/4×4 DL MIMO — **8 WWAN antennas is the floor** for a
    modern 5G phone, not optional.
-   All antennas are **LDS/FPC on the plastic-overmoulded metal mid-frame
    and on the camera-cover plastic**; WWAN rings the frame periphery,
    Wi-Fi/BT sits in the top corners/camera bump, NFC under the camera
    cover. This is the mechanical reason to keep the plastic-over-metal
    frame from §15 rather than a metal band.
-   Wi-Fi peak gains on FP6 are only −4…+1 dBi — acceptable target.
-   Deliverables to expect from the ODM: antenna gain report, SAR
    characterisation (Smart Transmit tables), simultaneous-TX grouping
    (FP6: top group Ant 0/2/3/4/8, bottom group Ant 5/6).

### 7.5 Decision

-   **Selected:** integrated Snapdragon 5G Modem-RF + Qualcomm companion
    kit; RFFE and antenna set to be executed by the ODM on the Qualcomm
    7-series reference design, using the FP6 (T2 Mobile / TCL-built)
    layout as the benchmark.
-   **Open:** final band list per launch region; PC2/HPUE yes/no; whether
    to buy the FP6-style 8+3+NFC antenna set or reduce (only possible if
    n41/n77-78 4-layer MIMO is dropped — not recommended).
-   **Certification (FCC/CE-RED/PTCRB-GCF/carrier)** is a first-order
    cost/time item and is the practical reason not to design the RFFE
    in-house at 10 k units. FP6's FCC filing alone is 160 exhibits.

## 8. Charging / USB

  ----------------------------------------------------------------------------------
  Part                    Current direction       Why
  ----------------------- ----------------------- ----------------------------------
  Connector               **USB-C**               Current standard for modern phones
                                                  and supports
                                                  charging/data/accessory functions.

  Charging                **45 W USB-C PD**       Reasonable fast-charging target
                                                  without pushing the thermal/power
                                                  system toward extreme charging
                                                  rates.

  USB subsystem           **Not fully specified** Controller/protection/ESD/mux/CC
                                                  implementation still needs to be
                                                  finalized. **Working subsystem
                                                  estimate: \~\$2--6+**
  ----------------------------------------------------------------------------------

## 9. Power-management electronics

  ------------------------------------------------------------------------
  Part                  Status                            Working estimate
  --------------------- --------------------- ----------------------------
  PMIC / power tree     **Required; not yet                       \~\$3--6
                        fully selected**      

  Battery protection /  **Required;                    Included in broader
  charging              architecture not                 power-system work
                        fully locked**        

  USB-C PD              **Direction selected;    Included in USB subsystem
  implementation        exact ICs not         
                        locked**              
  ------------------------------------------------------------------------

The PMIC and power tree need to be matched to the Snapdragon platform,
display, battery, charging current and peripheral rails.

## 10. Audio

Current requirement:

-   **At least 3 microphones** was identified as the working direction.
-   Speaker/audio amplifier/codec architecture remains to be finalized.
-   Approximate audio subsystem working estimate: **\~\$2--5**.

The three-microphone direction supports better voice capture/noise
handling and is more appropriate for a modern smartphone than a minimal
one-mic design.

## 11. Haptics

  -----------------------------------------------------------------------
  Options considered      Current direction       Why
  ----------------------- ----------------------- -----------------------
  Basic vibration motor / **LRA haptic actuator** Better modern
  LRA                                             smartphone haptic
                                                  response with
                                                  relatively low cost and
                                                  power.

  Cost estimate           **\~\$0.5--1.5**        Working estimate
  -----------------------------------------------------------------------

## 12. Sensors

A conventional smartphone sensor package is still required.

Expected categories include:

-   Accelerometer
-   Gyroscope
-   Magnetometer
-   Ambient light/proximity
-   Other required environmental/orientation sensing

**Status: not fully locked.** Working estimate discussed: **\~\$1--3**.

## 13. Wi-Fi / Bluetooth / GNSS / NFC

These were identified as required smartphone subsystems but **specific
ICs/modules were not yet locked**.

Remaining decisions include:

-   Wi-Fi generation and chipset
-   Bluetooth version/features
-   GNSS receiver/integration
-   NFC controller and antenna
-   Whether these functions are integrated into the chosen platform or
    require companion ICs

These choices need to be made together with the cellular/RF
architecture.

## 14. PCB and electronics

The BOM cannot stop at the named chips. The complete phone requires:

-   Main PCB
-   RF PCB/sections as applicable
-   FPCs
-   Board-to-board connectors
-   PMIC/power components
-   RF matching/passives
-   ESD protection
-   USB protection
-   Camera/display connectors
-   Antenna matching networks
-   Crystal/clock components
-   Miscellaneous passives and ICs

**Status: not fully costed/locked.**

## 15. Mechanical structure

Mechanical components identified as missing from the initial BOM:

-   Chassis/frame
-   Rear cover
-   Display/frame bonding
-   Battery retention
-   Camera brackets
-   Buttons
-   USB-C mechanical structure
-   Seals/gaskets
-   Internal brackets
-   Screws/adhesives
-   Antenna structures
-   SIM/eSIM-related mechanical parts
-   Waterproofing provisions

Working mechanical estimate previously discussed: **\~\$15--30+**,
depending heavily on material, tooling, tolerances and production
volume.

## 16. Thermal system

A Snapdragon-class SoC and 120 Hz AMOLED phone require a real thermal
solution.

Required elements may include:

-   Graphite/thermal films
-   Vapor chamber or heat spreader as appropriate
-   Thermal interface materials
-   Mechanical heat spreading into the frame

**Status: not fully selected.**

Working estimate: **\~\$1--4**.

## 17. Waterproofing / ingress protection

Water/dust resistance was identified as a missing BOM/system
requirement.

Potential requirements include:

-   Display/frame sealing
-   USB-C sealing
-   Speaker/microphone membranes
-   Button sealing
-   Camera-module sealing
-   Internal gasket/adhesive strategy

**Status: not yet locked.**

## 18. Operating system decision

### Options considered

  -----------------------------------------------------------------------
  OS/platform                         Assessment
  ----------------------------------- -----------------------------------
  Conventional Linux mobile OS        Maximum control, but substantial
                                      work is required for phone-specific
                                      hardware, drivers, modem
                                      integration, power management,
                                      camera stack and app ecosystem.

  **Android / AOSP**                  **Selected direction**

  iOS                                 Not realistically available for a
                                      third-party custom phone because
                                      iOS is tied to Apple's
                                      hardware/software ecosystem.
  -----------------------------------------------------------------------

### Current selection: AOSP / customized Android

The current direction is:

**Custom hardware → Linux kernel → Android HAL → AOSP → custom
system/UI**

Why:

-   Existing support for smartphone hardware
-   Mature modem/camera/GPU/display/power infrastructure
-   Large Android application ecosystem
-   Ability to customize the UI and system deeply
-   Much lower platform-development risk than creating a mobile OS from
    conventional Linux

### Security direction

The desired architecture is a hardened Android/Linux stack using:

-   Secure boot
-   Signed bootloader/kernel/system
-   Verified boot
-   SELinux in enforcing mode
-   Application sandboxing
-   Android permission model
-   Hardware-backed key storage / TEE
-   Encrypted user data

The key distinction discussed was:

**Android = Linux security primitives + a tightly integrated mobile
security architecture.**

A conventional Linux phone can also be highly secure, but more of the
security architecture has to be designed and maintained by the device/OS
builder.

## 19. Current locked BOM

  ------------------------------------------------------------------------
  Category              Current selection             Approx. working cost
  --------------------- --------------------- ----------------------------
  SoC                   Snapdragon 7 Gen 4, 4                     \$30--40
                        nm                    

  RAM                   12 GB LPDDR5X                             \$10--15

  Storage               256 GB UFS 4.0                            \$15--22

  Display               6.36", 1080×2400, 120                     \$25--40
                        Hz AMOLED             

  Main camera           Sony LYTIA 808, 50 MP                     \$20--30
                        OIS                   

  Front camera          Samsung ISOCELL 3LU,                        \$3--7
                        12 MP AF              

  Battery               6000 mAh                                   \$8--12
                        silicon-carbon,       
                        removable             

  Fingerprint           Side capacitive,                            \$1--3
                        power-button          
                        integrated; Goodix    
                        direction             

  Charging              USB-C PD, 45 W                                 ---

  SIM                   eSIM-only                                      ---

  OS                    AOSP/custom Android                            ---
  ------------------------------------------------------------------------

### Previously estimated hardware total

A preliminary bottom-up estimate was approximately **\$159--267 per
phone**, depending on the final RF, PCB, mechanical, display/camera
sourcing, thermal, audio and other subsystem choices.

A more focused target discussed was approximately **\$170--220** for the
complete hardware BOM, subject to final component selection and
manufacturing assumptions.

## 20. Major remaining BOM gaps

The main areas still requiring explicit part selection are:

1.  ~~5G/4G modem~~ — resolved: integrated in the Snapdragon SoC (§7.1)
2.  **RF front-end** (transceiver comes with the platform; PA/DRx/tuner/switch set still open — §7.3)
3.  **Supported cellular bands**
4.  **Antennas and RF architecture** (FP6 8 WWAN + 3 WLAN + NFC as template — §7.4)
5.  **Wi-Fi/Bluetooth**
6.  **GNSS**
7.  **NFC**
8.  **PMIC**
9.  **Battery-management/charging ICs**
10. **USB-C controller/protection**
11. **Audio codec/amplifier**
12. **Microphones**
13. **Speakers**
14. **Motion/environmental sensors**
15. **Haptic actuator**
16. **PCB/FPC/connectors**
17. **Thermal solution**
18. **Mechanical chassis/frame**
19. **Water/dust sealing**
20. **Final display/camera module integration**
21. **Android BSP/kernel/vendor support**
22. **Certification/testing**

## 21. Overall architecture

The current concept is therefore:

**Snapdragon 7 Gen 4** → **12 GB LPDDR5X** → **256 GB UFS 4.0** →
**6.36" 120 Hz AMOLED** → **Sony LYTIA 808 50 MP OIS main camera** →
**Samsung ISOCELL 3LU 12 MP AF front camera** → **6000 mAh removable
silicon-carbon battery** → **side capacitive Goodix fingerprint** →
**eSIM-only cellular identity** → **USB-C 45 W PD** → **custom
RF/cellular subsystem still to be finalized** → **AOSP/custom Android**
→ **hardened Android security architecture**

## Important status convention

**Selected** = current design decision.

**Direction** = architecture preference, but exact component still needs
selection.

**Not fully locked** = identified as necessary, but no final
component/implementation has been chosen yet.

**Cost** = working estimate only; it should not be treated as a supplier
quotation.
