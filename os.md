# OS Selection — Custom Phone

**Context:** this is a *new* board (Snapdragon 7-series, custom PCB — see
`custom_phone_design_decision_summary.md` §18–19). "Supported devices" lists on
community ROM sites don't apply: nobody has a device tree for hardware that
doesn't exist yet. Every OS below sits on top of the vendor BSP (bootloader,
kernel, HALs, radio/camera/display firmware), which Qualcomm ships under NDA and
which we obtain through an ODM/IDH — that is the gating item, not the OS.

**Recommendation:** AOSP as the base, optionally rebased onto LineageOS once the
port works, with /e/OS (Murena) as a route to a de-Googled consumer SKU.
GrapheneOS is not installable but its published hardware requirements are the
best available security spec — see §2.

## 1. OS comparison

| OS | Open source? | Maintainer | What it gives a *new* board | What we still need | Verdict |
|---|---|---|---|---|---|
| **AOSP** | Yes — Apache 2.0 (kernel GPLv2) | Google | The platform every Snapdragon BSP targets; full stack: kernel → HAL → framework | Vendor BSP (via ODM), our system/UI layer, GMS decision, CTS/VTS if we want Play | **Selected base** (matches §18) |
| **LineageOS** | Yes — Apache 2.0 | Community | A maintained upstream tree to rebase on, security backports, standard device-tree conventions | A working AOSP port *first*; then an active maintainer or the device gets dropped | **Optional later base** |
| **/e/OS (Murena)** | Yes — Lineage-derived | Murena SAS | De-Googled distro, App Lounge (Play proxy without Google login), OTA infra, they maintain it. Similar: CalyxOS, iodéOS | Commercial partnership (this is the €599 Fairphone /e/OS SKU); BSP first | **Optional, for a de-Googled SKU** |
| **GrapheneOS** | Yes — MIT | GrapheneOS Foundation | The reference for hardened Android; publishes an explicit hardware/firmware requirements list; states it plans OEM partnerships | Meet the full requirements list (§2); a partnership, not a self-serve install; not adoptable by third parties today | **Not installable — use as hardware-security spec / possible future partner** |
| **Sailfish OS** (Jolla Phone) | **Partly** — core open; Silica UI toolkit and Android AppSupport proprietary | Jolla | OEM licensing; runs on an Android BSP via libhybris | License fee; BSP still needed; small native app ecosystem | **Excluded — fails "open source"** |
| **postmarketOS / Ubuntu Touch / Mobian** | Yes | Communities | Mainline or Halium Linux | No shippable support for a current 7-series Snapdragon; we'd own modem/camera/power bring-up; tiny app ecosystem | **Excluded** (matches §18) |

Not applicable: iOS/HarmonyOS (closed, tied to vendor hardware).

## 2. GrapheneOS requirements vs. our current design

Source: <https://grapheneos.org/faq#supported-devices> (fetched 2026-08-15).
GrapheneOS supports Pixel 6 → 10 only, and says "broad device support is counter
to the aims of the project" — but it also says: *"we plan to partner with OEMs to
have devices produced meeting all our requirements … ideally shipping with
GrapheneOS."* So this list is both a design checklist for §18's "hardened Android
security architecture" and the entry ticket if we ever want that partnership.

| GrapheneOS requirement (verbatim) | Our design (`custom_phone_design_decision_summary.md`) | Status |
|---|---|---|
| Hardware memory tagging (ARM MTE or equivalent) | Snapdragon 7 Gen 4 selected. Qualcomm's full MTE implementation is reported for Snapdragon 8 Elite Gen 5; 7-series support unconfirmed | **Likely gap — disqualifying on its own. Verify with Qualcomm/ODM before locking SoC** |
| Hardware-based coarse-grained CFI (PAC/BTI) | Depends on core IP in the SoC | Verify |
| Isolated radios, GPU, SSD, media encode/decode | Platform IOMMU/SMMU; modem integrated in SoC | Verify in BSP |
| StrongBox keystore provided by secure element | **No secure element in BOM.** Check whether SoC has a Qualcomm SPU exposing StrongBox; else add discrete SE | **Gap** |
| Hardware key attestation for StrongBox | Follows from SE | Gap |
| Weaver disk-encryption key-derivation throttling via secure element | Follows from SE | Gap |
| Insider attack resistance for SE updates (owner auth required) | SE firmware policy | Gap |
| Inline disk-encryption acceleration with wrapped keys | UFS 4.0 selected; Snapdragon ICE supports this | Likely met via BSP |
| Verified boot + rollback protection, firmware and OS (AVB) | §18 lists secure/verified boot as direction | Direction matches; bootloader work in BSP contract |
| Verified boot key fingerprint shown for yellow state (full SHA-256) | Bootloader UI change | Needs BSP source access |
| Complete monthly ASB patches, no regular delay > 1 week | Gap #21 (BSP/vendor support) | **Contract item with ODM** |
| ≥ 5 years of device-support-code updates from launch | Qualcomm 7-series commitments typically shorter | **Contract item; risk** |
| Device support code updated to new AOSP releases within months | Gap #21 | Contract item |
| Linux 6.1 / 6.6 / 6.12 GKI | BSP kernel version | Verify with ODM |
| 64-bit-only device support code | Not specified | Specify to ODM |
| Wi-Fi MAC address randomization | §13 Wi-Fi not locked | Add as Wi-Fi chipset requirement |
| Hardware-level USB data disable / whole-USB disable | §8 USB subsystem not fully specified | Add as USB requirement (controllable data switch/mux) |
| JTAG/serial debug inaccessible while locked | Not specified | Add to §14/§15 board requirements |

**Net:** two hardware gaps (MTE, secure element) that must be resolved at SoC
selection, and roughly six items that are simply requirements to write into the
ODM/BSP contract. Everything else is bootloader/BSP work we need source access
for anyway.

## 3. What "free and open source" does not cover

| Item | Reality |
|---|---|
| Vendor BSP | Qualcomm → partners under NDA; we get it via ODM/IDH at ~10k units. Not downloadable. |
| Google Play / GMS | Requires a Google agreement and passing CTS. Without it: microG + banking-app friction, or partner with Murena. |
| Security patches | For a device no community owns, nobody patches it unless the ODM/BSP contract says so. |
| Certification | CTS/VTS, carrier certification, regulatory (FCC/CE/BIS…). Doc gap #22. |

## 4. Decision path

1. Choose the ODM/IDH on BSP availability and reference-design fit — *before* freezing the SoC.
2. Resolve MTE and secure-element questions as part of that choice (§2).
3. Contract BSP + monthly ASB merges + ≥ 5 years support explicitly.
4. Decide GMS vs. de-Googled — it forks the plan.
5. Build custom AOSP on the BSP.
6. Later, optionally: rebase to LineageOS or partner with Murena to offload maintenance.
