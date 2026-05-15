# 35t — CaptainDMA / PCIe emulation workspace

Published Git repository: [github.com/offwipe/35t](https://github.com/offwipe/35t)

Lab notes and runbooks for **CaptainDMA 4.1th-35T** ([product page](https://captaindma.com/product/captain-dma-4-1th/)) custom emulation firmware work.

Upstream FPGA sources live in [`ufrisk/pcileech-fpga`](https://github.com/ufrisk/pcileech-fpga) (clone separately); this repo tracks **documentation, golden-capture templates, and session state**.

## Start here

| Doc | Purpose |
|-----|---------|
| [FPGA-DMA-EMULATION-STUDY-NOTES.md](FPGA-DMA-EMULATION-STUDY-NOTES.md) | Agent-oriented PCIe emulation reference + JPShag README mirror anchors |
| [firmware-workspace/README.md](firmware-workspace/README.md) | Index of runbooks and capture workflow |
| [firmware-workspace/WORKSPACE_RESOLUTION.md](firmware-workspace/WORKSPACE_RESOLUTION.md) | CaptainDMA **4.1th** → `CaptainDMA/35t484_x1` / Tcl script names |

## Hardware this project assumes

- **DMA board:** CaptainDMA **4.1th-35T** (Artix-7 35T), documented vendor install/help via vendor site / Discord linked from product page.
- **Lab PC (capture + flash/test):** documented in [firmware-workspace/SESSION_PROGRESS.md](firmware-workspace/SESSION_PROGRESS.md).

## Ethical / lawful use

Use only on systems and hardware you own or are explicitly authorized to test. This repo does **not** target anti-cheat evasion or stealth donor selection.
