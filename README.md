# O-Phone

Working notes for a custom ("open-ish") phone design: BOM and architecture
decisions, OS selection, and reference analysis of comparable devices
(Fairphone Gen 6, Jolla Phone 2026, Light Phone).

## Contents

- `custom_phone_design_decision_summary.md` — working BOM / architecture decision summary
- `os.md` — OS selection (AOSP base), GrapheneOS hardware-requirements checklist vs. this design
- `SKR_learnings.md` — notes (why phone SoCs aren't retail parts, ODM procurement)
- `lightphone_specs.md`, `fairphone.md`, `jollaPhone.md` — reference device specs
- `fairphone6_fcc_rf_analysis.md` — RF/antenna analysis from the Fairphone Gen 6 FCC filing
- `fairphone6_video_teardown_analysis.md` — teardown analysis from iFixit / JerryRigEverything videos
- `FCC_2AUWUFP6_Fairphone_Gen6/` — README + index for the FCC ID 2AUWUFP6 filing
  (the 160 PDFs themselves are not committed; the index links each to its source)
- `videos/` — key frames, contact sheets and transcripts used by the teardown analysis
  (full videos not committed; source links in `videos/README.md`)

## Not committed — external source links

Files used locally but excluded from the repo (size and/or third-party copyright);
get them from the source:

- **Apple iPhone SE2 (A2296) teardown sample report** — Fomalhaut Techno Solutions /
  Yole SystemPlus. Product page: <https://fomalhaut.co.jp/ja/apple-iphone-se2-teardown-report/>
  (original file: `https://www.reverse-costing.com/media/Apple_iPhoneSE2_A2296_Rev2.pdf`)
- **Fairphone Gen 6 FCC exhibit PDFs** (160 files, ~560 MB) — see
  [`FCC_2AUWUFP6_Fairphone_Gen6/README.md`](FCC_2AUWUFP6_Fairphone_Gen6/README.md)
  and [`INDEX.md`](FCC_2AUWUFP6_Fairphone_Gen6/INDEX.md) for per-file links
- **Teardown/durability videos** (iFixit, JerryRigEverything) — see
  [`videos/README.md`](videos/README.md)
