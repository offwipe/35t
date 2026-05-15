# HD Audio donor — lawful engineering scope

This note replaces **not** a secret device pick list. Per the attached **post-notes lawful firmware plan** and project policy: no donor selection optimized for **evasion**, **obscurity**, or **non-use for DMA**.

## What “good documentation” usually means for HD Audio

HD Audio on PCs is often **not** one tidy generic PCIe endpoint datasheet:

- **Intel-style onboard controllers** appear as a PCIe function (historically common **bus:dev.fn** pattern **00:1f.3** on many desktops — **not universal**). Programming detail is spread across **Intel EDC / PCH or SoC register volumes** for **your exact chipset generation**, plus the **Intel HD Audio architecture/spec** for the audio engine/link layers.
- **Discrete PCIe sound cards** use **vendor-specific** PCI IDs, BARs, and firmware; “full datasheet” availability varies; **Linux driver source** (`sound/pci/hda/…`) often substitutes.

Authoritative public entry points (you choose the volume that matches **your** CPU/PCH):

- Intel HD Audio — PCI configuration / PCR-style documentation appears under chipset-specific books (examples from Intel’s site: **800 Series PCH Vol 2**, **400 Series / Comet Lake U PCH**, **Core Ultra SOC I/O registers**) — search Intel EDC for **“High Definition Audio PCI Configuration”** for your platform.
- Broader audio programming: **Intel High Definition Audio Specification** (rev updates published by Intel; large PDF ecosystem).

## Practical emulation consequence (CaptainDMA 35T)

**FULL_EMU** of HD Audio is **heavy**: CORB/RIRB, codecs, DMA buffers, interrupts, power management — easy to fail Drvscan/driver probes if anything is “almost right.”

Before committing RTL:

1. Confirm **BAR sizes** and **MSI/MSI-X** layout from **your golden capture** fit **Artix-7 35T BRAM budget** ([FPGA-DMA-EMULATION-STUDY-NOTES.md](../FPGA-DMA-EMULATION-STUDY-NOTES.md) **INV_BAR_REALITY**).
2. Prefer starting from **golden capture + Linux `snd-hda-intel` trace** (authorized lab machine) over guessing from a mismatched chipset PDF.

## Decision you must make (not the assistant)

**Donor A — Onboard Intel (AMD/other) HD Audio controller**  
You emulate the **exact function** on **your** lab PC’s motherboard — documentation path = **that platform’s** vendor register docs + capture. **No separate PCIe card required.**

**Donor B — Discrete PCIe HD Audio / sound card you physically own**  
Documentation path = vendor + driver + optional analyzer traces.

We do **not** select among strangers’ silicon for “hidden DMA.” We bind emulation to **your captured device**.

### Onboard vs discrete — engineering only (not stealth)

Some users ask whether onboard HD Audio is “less common / harder to detect” than other donors. **Do not use detection probability as a criterion.** Choose onboard **because it is the PCIe function your lab PC actually exposes** and you can capture authoritatively — not because of anti-analysis folklore.
