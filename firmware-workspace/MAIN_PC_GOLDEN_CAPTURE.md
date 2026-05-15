# Main PC — golden capture notes (MSI Z390 + i5-9600K)

**Role:** CaptainDMA card is installed and enumerated on **this** PC after flash. **Golden capture** for onboard HD Audio should therefore be taken **on this same machine** (or you accept drift if capture host ≠ enumeration host).

## System (from user `msinfo32`)

| Field | Value |
|-------|--------|
| OS | Windows 10 Pro — Build **19045** |
| Board | **MSI MPG Z390 GAMING EDGE AC** (MS-7B17) |
| CPU | Intel **i5-9600K** (Coffee Lake — **Z390** PCH) |
| Secure Boot | On |
| Kernel DMA Protection | Off (user-reported) |

## Device Manager — “High Definition Audio Device” (Microsoft)

Screenshot shows:

- **Manufacturer:** Microsoft  
- **Location:** `Location 0 (Internal High Definition Audio Bus)`  

That usually means a **High Definition Audio bus / function driver node**, not necessarily the **PCI Express endpoint** whose **256-byte PCI config space** you program into the FPGA.

### What to capture for PCIe emulation

You need the **PCIe device function** that implements the **Intel HD Audio host controller** on Z390 (often appears under **System devices** as something like **Intel(R) High Definition Audio Controller** or similar — exact string varies by driver pack).

**Do this:**

1. In Device Manager, expand **System devices** and look for an **Intel**-branded HD Audio **controller** (PCI).  
2. Cross-check in **Arbor** / **PCI-Z** / **HWiNFO → Bus** for the **same VID:DID** and confirm it has **PCIe BARs** in config space.  
3. If only the Microsoft “Internal HDA Bus” child shows in Details, still use Arbor’s **PCI tree** to find the **parent PCIe function** above that audio stack.

Screenshots (in repo):

- [golden_capture/screenshots/main_pc_device_manager_hda.png](golden_capture/screenshots/main_pc_device_manager_hda.png)
- [golden_capture/screenshots/main_pc_msinfo32.png](golden_capture/screenshots/main_pc_msinfo32.png)

## Benchmark / hazard framing

You may use strict enumeration and logging as a **technical benchmark** of emulation quality. This repo still does **not** document steps aimed at **evading** third-party security or anti-cheat products.

**Hazards:** mis‑BAR, bad completions, or config-space mistakes can **hang, BSOD, or brick boot** until the card is removed or firmware is recovered. Mitigations: disk image, USB recovery, CMOS procedure, spare slot testing.
