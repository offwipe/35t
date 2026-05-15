# Golden capture on Windows 10 / 11 (step-by-step)

**Goal:** Freeze **facts** about the PCIe device you will emulate (usually **onboard HD Audio controller**) so RTL/IP changes match reality — Drvscan-style tools compare bytes later.

Prefer capture on the **same PC** where the FPGA card will enumerate after flash (user workflow: **main PC**). If you ever capture on another machine, document both and expect possible mismatch.

You do **not** need Linux if you use the tools below (Linux `lspci -xxxx` is optional).

---

## 0 — Main PC (MSI Z390 / i5-9600K)

See **[MAIN_PC_GOLDEN_CAPTURE.md](MAIN_PC_GOLDEN_CAPTURE.md)** — the “**High Definition Audio Device**” entry under *Sound* with **Microsoft** + **Internal High Definition Audio Bus** is often **not** the PCIe config-space donor; find the **Intel HD Audio PCIe controller** (commonly under **System devices**) for Arbor/PCI-Z.

---

## 1 — Identify the exact device

1. **Win + X** → **Device Manager**.
2. Expand **Sound, video and game controllers** *and* **System devices** — Intel HD Audio **PCIe controller** is often under **System devices** (wording varies).
3. Pick the device that matches **your onboard HD Audio PCIe function** (not USB headsets, not unrelated virtual audio).

**Rule:** One logical choice only — the **PCIe function** whose config space you will mirror.

---

## 2 — Minimum capture (always do this)

Right‑click device → **Properties**:

### Tab **Details**

Cycle **Property** dropdown and record:

| Property | Why |
|----------|-----|
| **Hardware Ids** | `PCI\VEN_xxxx&DEV_yyyy&SUBSYS_zzzzwwww&REV_nn` |
| **Compatible Ids** | Secondary hints |
| **Device instance path** | Stable instance string |
| **Location information** | Sometimes PCI bus related |

Screenshot or copy text into `firmware-workspace/golden_capture/hardware_ids.txt`.

---

## 3 — Stronger capture (recommended)

Standard Properties dialogs **do not** show full **256-byte config space** or capability chains.

Pick **one** path:

### Option A — Arbor (MindShare)

Matches JPShag README §5 workflow — decoded BARs, caps, extended caps.

1. Install Arbor (administrator).
2. **Local System** → **Scan**.
3. Find row matching **same VID/DID** as Hardware Ids.
4. Export / screenshot **PCI Config** view.

Save exports under `firmware-workspace/golden_capture/` (see [golden_capture/README.md](golden_capture/README.md)).

### Option B — PCI-Z (free, portable)

Often displays PCI configuration dump per device — export/screenshot for offline diff.

### Option C — HWiNFO64

**Bus** section → locate device → note BAR addresses / widths where shown (still supplement with raw dump if possible).

---

## 4 — Fill `donor_template.yaml`

Copy [golden_capture/donor_template.yaml](golden_capture/donor_template.yaml) → `donor_decoded.yaml`.

Translate:

- **Hardware Ids** → `vendor_id`, `device_id`, `subsystem_*`, `revision_id`.
- **Class code** — Arbor/HWiNFO usually shows `0403` style breakdown (base/sub/prog IF); encode as 24-bit hex per notes.
- **BARs** — only trust decoded tool output or raw dump — Device Manager alone may be insufficient.

---

## 5 — Raw hex dump (ideal regression artifact)

If you obtain **full config hex** (first 256 bytes minimum; extended 4K if tool exports it), save as:

`firmware-workspace/golden_capture/donor_config_space.hex`

Future compares use this as golden truth.

---

## 6 — Optional sanity check

After FPGA firmware boots on same machine class, **Hardware Ids** in Device Manager should match your template **if** emulation targets same identity — mismatches mean SV/IP drift (**INV_DUAL_SOURCE**).

---

## When stuck

Fetch JPShag authoritative README (`readme_authoritative_raw_url` in [FPGA-DMA-EMULATION-STUDY-NOTES.md](../FPGA-DMA-EMULATION-STUDY-NOTES.md) YAML) §5 — donor gathering narrative — then return here for Windows-specific mechanics.
