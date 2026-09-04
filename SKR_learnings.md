# SKR Learnings — Custom Phone

## Why you can buy a PC CPU but not a phone SoC

| | PC CPU (Intel/AMD) | Phone SoC (Snapdragon 7 Gen 4, Dimensity) |
|---|---|---|
| **How it's sold** | Retail, one unit, any store | Contract only, to qualified OEM/ODMs, MOQ in the tens of thousands |
| **Package** | Socketed (LGA1851 / AM5) — drop it in | Bare BGA, ~0.35 mm pitch, DRAM stacked on top — needs a board designed around it |
| **Board** | Standardized motherboards from dozens of vendors | Bespoke, per device, designed by the ODM |
| **Firmware / boot** | UEFI + ACPI — a *standard* the OS relies on | Vendor bootloader + device tree + vendor drivers, per board — no standard |
| **Installing an OS** | Windows/Linux ISO boots on any board | No generic image can exist; the OS build *is* the board port |

**Why:** the PC industry standardized how the OS discovers hardware
(UEFI/ACPI/PCIe enumeration), so any OS boots on any board and the CPU can be a
retail commodity. Phones never did — each SoC+board needs bespoke firmware and
drivers, so the chip is only useful to whoever also has the BSP, and Qualcomm
sells both together to the same closed circle. (Arm SystemReady/EBBR is an
effort to standardize this; phone SoCs don't comply.)

**Nuances**

- Gray-market Snapdragons exist on AliExpress/Taobao — salvaged BGA parts for
  phone-repair reballing. Buyable, useless without a board and BSP.
- It's a gradient: Rockchip/Allwinner sell via distributors in small quantities
  with freely-leaking SDKs (hence hobbyist boards). Broadcom (Raspberry Pi) —
  you buy the board, not the chip. Qualcomm/MediaTek phone parts sit at the
  closed end.

**Consequence for the design doc:** the "Snapdragon 7 Gen 4, ~$30–40" line in
`custom_phone_design_decision_summary.md` §1 is a valid BOM figure, but the
procurement path is through the ODM, who is Qualcomm's customer. At 10k units
you never hold a PO with Qualcomm — the ODM's reference-design catalogue is
effectively your SoC menu, so pick the ODM before freezing the SoC.
