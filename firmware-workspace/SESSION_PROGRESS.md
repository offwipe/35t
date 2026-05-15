# Session progress log (§CODING_SESSION_PROTOCOL)

Copy this block into chat or append dated entry below when starting/finishing work.

## Template — BEFORE (B2/B3)

```
PHASE: Step __ / §RM___ / README §__
BOARD_TREE: CaptainDMA/35t484_x1 (or other: ________)
PCILEECH_ROOT: ___________________________________
GIT_COMMIT: ___________________________________
LAST_KNOWN_GOOD: ___________________________________
NEXT_GOAL: ___________________________________
```

## Template — AFTER (A1/A2/A3)

```
FILES_TOUCHED:
  -
REGENERATED_IP: yes/no
NEW_BITSTREAM: path ____________________
REBOOT_REQUIRED: yes/no
NEXT_STEP_FOR_NEXT_AGENT: ___________________________________
TEST_LADDER_GATE: L?_...
```

---

## Lab hardware (record for documentation only)

| Item | Value |
|------|--------|
| DMA board | CaptainDMA **4.1th-35T** — vendor [product page](https://captaindma.com/product/captain-dma-4-1th/) |
| Upstream FPGA dir | `CaptainDMA/35t484_x1` — see [WORKSPACE_RESOLUTION.md](WORKSPACE_RESOLUTION.md) |
| **Main PC** (enumeration + golden capture target) | **MSI MPG Z390 GAMING EDGE AC** (MS-7B17), **i5-9600K**, **Windows 10 Pro** build **19045**; Secure Boot **On**, Kernel DMA Protection **Off** (user-reported) |
| CaptainDMA card slot | **Main PC** PCIe slot after firmware ready (user correction — not GMKtec-only) |
| Other PCs | GMKtec N100 G3 (Win11) / HP EliteDesk + Proxmox — **optional** (e.g. flash convenience); **donor data** should follow **main PC** if card enumerates there |
| Flash workflow | Zadig / vendor tools — whichever machine user uses for USB-JTAG (document path when stable) |
| Public repo | [github.com/offwipe/35t](https://github.com/offwipe/35t) |

## Donor strategy (current)

| Choice | Status |
|--------|--------|
| Discrete PCIe donor card | **None** — user has no spare donor hardware |
| **Onboard HD Audio** | **Default donor** — golden capture from **main PC** (Z390) per [GOLDEN_CAPTURE_WINDOWS11.md](GOLDEN_CAPTURE_WINDOWS11.md) + [MAIN_PC_GOLDEN_CAPTURE.md](MAIN_PC_GOLDEN_CAPTURE.md) |

## Golden capture gate

- [ ] `golden_capture/hardware_ids.txt` (Device Manager Details)
- [ ] Decoded BAR/cap export (Arbor **or** PCI-Z **or** equivalent)
- [ ] `golden_capture/donor_decoded.yaml` filled from [donor_template.yaml](golden_capture/donor_template.yaml)

**RTL / PCIe IP edits blocked until** `donor_decoded.yaml` exists (and raw dump strongly preferred).

---

## Log entries

*(Append newest first)*

| ISO date | Phase summary | Operator |
|----------|---------------|----------|
| 2026-05-15 | **Correction:** CaptainDMA card enumerates on **main PC** (MSI Z390, Win10 19045), not lab-only; golden capture should follow **main PC** PCIe HD Audio controller — see [MAIN_PC_GOLDEN_CAPTURE.md](MAIN_PC_GOLDEN_CAPTURE.md). User accepts brick/BSOD risk for benchmark-style testing; still no evasion-oriented donor selection. **Blocked:** golden files. | Agent |
| 2026-05-15 | *(Superseded by row above — host mix-up)* Earlier note assumed GMKtec-only capture; user clarified **main PC** is enumeration + capture target. | Agent |
| 2026-05-15 | Added `firmware-workspace/` (resolution, lawful checklist, golden templates, Vivado + test runbooks). Resolved CaptainDMA **4.1th** → `CaptainDMA/35t484_x1` + `vivado_generate_project_captaindma_35t.tcl`. No local `pcileech-fpga` clone in `thirty-five` yet — user must clone and fill SESSION_PROGRESS. | Agent |
| 2026-05-15 | **Donor direction:** User asked HD Audio + obscure/non-DMA-common pick — **declined** (evasion/obscurity). **Lawful path:** emulate **only** the HD Audio–class PCIe function **they own**; docs = platform-matched Intel/vendor volumes + capture; see [DONOR_HD_AUDIO_LAWFUL_SCOPE.md](DONOR_HD_AUDIO_LAWFUL_SCOPE.md). **Next blocker:** user-supplied golden capture + checklist — **no RTL yet.** | Agent |
