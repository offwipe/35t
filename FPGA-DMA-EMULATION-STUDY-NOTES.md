---
doc_type: agent_reference
audience: coding_agents
project: captaindma_artix7_35t_pcie_emulation
primary_upstream_repo: https://github.com/ufrisk/pcileech-fpga
study_repo: https://github.com/JPShag/PCILeech-DMA-Firmware
readme_authoritative_raw_url: https://raw.githubusercontent.com/JPShag/PCILeech-DMA-Firmware/main/README.md
readme_fetch_policy: Option_A_do_not_embed_full_readme_use_raw_url_for_verbatim_sections
ignore_sources_in_study_repo:
  - README-old.md
  - README-v2-old.md
wiki: https://github.com/JPShag/PCILeech-DMA-Firmware/wiki/Introduction
user_success_targets:
  - full_bar_support
  - drvscan_clean
  - windows_device_manager_no_warning_triangle
  - donor_behavior_parity
on_uncertainty_or_blocker: FETCH_official_JPShag_README_via_readme_authoritative_raw_url_then_notes_mirror
coding_phase_protocol: DOCUMENT_where_you_are_BEFORE_act_then_DOCUMENT_changes_AFTER_see_SECTION_CODING_SESSION_PROTOCOL
read_order_for_agents:
  - AGENT_DIRECTIVE
  - CODING_SESSION_PROTOCOL
  - INV_CONSTRAINTS
  - SOURCE_MAP
  - README_MIRROR_INDEX
  - FILE_CONTRACT
  - WORKFLOW_ORDERED_STEPS
  - README_MIRROR_AGENT_NOTES
  - SYMPTOM_ROUTE
  - ENC_TBL_PAYLOAD
---

# FPGA PCIe emulation — agent reference (CaptainDMA / Artix-7 35T)

## §AGENT_DIRECTIVE

1. **Before edits:** Load **§INV_CONSTRAINTS** and **§FILE_CONTRACT**. Do not contradict an invariant unless the user explicitly overrides it.
2. **When debugging:** Use **§SYMPTOM_ROUTE** first; cross-check **§FAILMODE**.
3. **When configuring PCIe:** Keep **`pcileech_pcie_cfg_a7.sv`** (or board equivalent) and **`pcie_7x_0` IP GUI** in lockstep — **§INV_DUAL_SOURCE**.
4. **Naming:** JPShag docs use placeholder paths (e.g. `pcileech-wifi-main`). Resolve **actual** paths inside the user’s `pcileech-fpga` fork / CaptainDMA board tree via repo search — never invent paths.
5. **Citation:** Facts tagged `[SRC:<id>]` map to **§SOURCE_MAP**. For JPShag **§2–§14 operational mirrors**, jump to **§README_MIRROR_INDEX** / **§README_MIRROR_AGENT_NOTES**.
6. **When stuck, unsure, or “how do I…?”:** **Stop guessing.** Fetch and consult the **official JPShag `README.md`** at **`readme_authoritative_raw_url`** (YAML frontmatter) — it is the **authoritative step-by-step** source. Use **§README_MIRROR_AGENT_NOTES** only as a fast index; fall through to the README for procedural nuance, screenshots/refs, and ordering details not fully spelled out in this file.
7. **Coding / RTL / TCL / Vivado actions:** Follow **§CODING_SESSION_PROTOCOL** every session: record **where you are**, consult README + this doc **before** changes, then **document what changed after**.
8. **Never substitute wiki/third-party summaries for JPShag README** when resolving ambiguity — README first for JPShag-authored procedure (wiki optional Supplement).

---

## §CODING_SESSION_PROTOCOL

Agents **must** run this loop whenever touching firmware, constraints, Tcl, or Vivado IP — including small edits.

### B — BEFORE any moves

| Step | Action |
|------|--------|
| B1 | **Locate workspace:** Confirm repo root, **`pcileech-fpga` (or fork) path**, and **exact board variant directory** (e.g. Tcl script name) via listing/`Glob` — do not assume placeholders from tutorials. |
| B2 | **State phase:** Map current goal to **§WORKFLOW_ORDERED_STEPS** step index **and/or** **§README_MIRROR_INDEX** anchor (e.g. “Step 4 / §RM_S072”). Write this explicitly to the user (or append to session summary). |
| B3 | **Record baseline:** Note branch/commit if Git used; note last known-good behavior (e.g. “enumerates VID:DID” / “Drvscan fails on cap offset”). |
| B4 | **If procedure unclear:** Fetch **`readme_authoritative_raw_url`** and jump to the matching README section (same numbering as **§README_MIRROR_***). Only then refine with **§README_MIRROR_AGENT_NOTES** + **§INV_CONSTRAINTS**. |

### D — DURING work

| Step | Action |
|------|--------|
| D1 | Prefer smallest change that satisfies the phase; keep SV ↔ PCIe IP (**INV_DUAL_SOURCE**) consistent in **one** edit series when IDs/BARs/caps change. |
| D2 | If behavior contradicts expectation → **§SYMPTOM_ROUTE**; if root cause still unclear → **README troubleshooting §13** + hardware README §11–12 — fetch README rather than improvising. |

### A — AFTER changes complete

| Step | Action |
|------|--------|
| A1 | **Document deltas:** List modified paths (`*.sv`, `*.xdc`, `*.tcl`, regenerated `.xci`); note whether **bitstream rebuild** / **IP regenerate** / **cold reboot** required. |
| A2 | **Update phase:** State next recommended step from **§WORKFLOW_ORDERED_STEPS** (or README) so the next agent picks up without re-discovery. |
| A3 | **Verification hint:** Point to **§TEST_LADDER** minimum gate appropriate (e.g. post-BAR-edit → **L2_BARS** + host poke plan). |

**Invariant:** If an agent would answer “I think…” without having opened the README section for that topic → **fetch README first**, then answer.

---

## §INV_CONSTRAINTS

Hard rules for agents (violations cause enumeration failures, driver Code 10, or silent MMIO bugs).

| ID | MUST / MUST NOT |
|----|-------------------|
| INV_DUAL_SOURCE | **MUST** keep PCIe identification fields consistent between **SystemVerilog cfg module** and **Vivado PCIe IP customization** (`pcie_7x_0`) where both define the same logical fields. [SRC:JPShag README §7.2.2, Checklist §4] |
| INV_BAR_REALITY | **MUST NOT** advertise BAR sizes / memories that exceed **implementable** FPGA backing (BRAM budget on **35T**) without a deliberate sparse/register backing strategy. Oversized dropdown-only BARs → probe failures. [SRC:JPShag README §8.2.1, notes] |
| INV_CAP_CHAIN | **MUST** reproduce donor **capability linked list** layout faithfully enough for OS/driver parsers (`cfg_cap_pointer` → subsequent next-pointers). Misaligned or fake caps → Drvscan failures or undefined driver behavior. [SRC:JPShag README §8.1.2] |
| INV_CAP_ENABLE | **MUST NOT** enable extended capabilities in IP (AER, MSI-X tables, FLR, VC…) without **matching runtime behavior** — advertised-but-unhandled caps are a primary driver-warnings source. [SRC:JPShag README §9.1] |
| INV_TREADY | **MUST** honor PCIe core **AXI-Stream `tready`** on transmit paths; bursting when `tready=0` violates FC assumptions → hangs / malformed TLP. [SRC:JPShag README §13.3] |
| INV_TIMING | **MUST** close timing (**WNS ≥ 0** on required domains) before treating a build as valid; negative slack is not an acceptable “maybe”. [SRC:JPShag README §11.1.2] |
| INV_MSIX_VERIFY | **MUST NOT** assume MSI-X works because the IP GUI checkbox is set — **verify** against actual wrapper/`pcileech_pcie_tlp_a7.sv` integration (may need explicit doorbell TLP path). [SRC:Lesson3, FW-Guide-v4 §8.3.2] |
| INV_LEGAL | **MUST** scope work to authorized hardware/systems only; document assumes legitimate owner-authorized firmware customization. [SRC:JPShag README §15.3] |

---

## §SOURCE_MAP

Use this table to fetch authoritative prose; this file is the distilled operational summary.

| SRC_ID | Location | Role |
|--------|----------|------|
| JPShag README | `README.md` in study repo; **raw:** `https://raw.githubusercontent.com/JPShag/PCILeech-DMA-Firmware/main/README.md` | Primary narrative guide (§§1–18). Fetch when verifying long procedural detail not duplicated below. |
| JPShag Checklist | `…/Checklist.md` | Linear checkbox workflow; step parity with README. |
| JPShag FW-G4 | `…/FW-Guide-v4.md` | Condensed guide; **DUMP vs FULL EMU** terminology; MSI-X MEMWR notes. |
| JPShag L3 | `…/Lesson 3_ Advanced PCIe Configuration and Interrupt Handling.md` | MPS DevCap/DevCtrl math; DLLP/FC overview; MSI vs MSI-X implementation commentary. |
| PG054 | AMD/Xilinx doc — 7 Series Integrated Block for PCIe | Core pins, MSI/shaping, constraints — **hardware truth** for IP. |
| UG939 | Vivado ILA | Embedded debug procedures. |

---

## §PROJECT_CONTEXT

Fixed facts about the user’s stated hardware (confirm revision with user docs if ambiguous).

```yaml
board_vendor_hint: CaptainDMA
fpga_part_family: xilinx_7series
fpga_density_class: artix7_35t
pcie_hard_block: integrated_block_pcie_7x
guide_equivalent_board_class: squirrel_35t_artix7  # same density class in JPShag prose
open_core_repo: ufrisk/pcileech-fpga
# Resolved upstream (CaptainDMA 35T desktop "4.1th") — see firmware-workspace/WORKSPACE_RESOLUTION.md
captaindma_41_desktop_upstream:
  repo_subdir: CaptainDMA/35t484_x1
  vivado_generate_project_tcl: vivado_generate_project_captaindma_35t.tcl
  vivado_build_tcl: vivado_build.tcl
  vendor_readme: https://github.com/ufrisk/pcileech-fpga/blob/master/CaptainDMA/readme.md
doc_placeholder_paths_ignore:
  - pcileech-wifi-main  # example only; resolve real board_variant directory per clone
implementation_workspace_index: firmware-workspace/README.md
```

---

## §TERMINOLOGY

| Term | Definition |
|------|------------|
| DUMP_EMU | Static config-space/BAR header parity only; may pass shallow scans; driver bring-up often fails. [SRC:FW-G4 §8.4] |
| FULL_EMU | Dynamic parity: correct completions, PM, MSI/MSI-X behavior, optional VDMs, donor MMIO semantics. User targets imply FULL_EMU. [SRC:FW-G4 §8.4] |
| Drvscan | External/static validator (user tool). Treat as **byte-level config-space diff** vs golden donor capture — not defined in JPShag README text. |
| Donor | Physical PCIe device whose capture defines golden IDs/BARs/caps/behavior. |

---

## §FILE_CONTRACT

Expected responsibilities — grep repo for exact paths (`board_variant` differs per fork).

| Relative pattern | Responsibility |
|--------------------|------------------|
| `**/src/pcileech_pcie_cfg_a7.sv` | VID/DID/subsystem/rev/class, **`cfg_cap_pointer`**, **`cfg_dsn`**, encoded **max payload / max read request**, other cfg-space-facing regs exposed by design. |
| `**/ip/**/pcie_7x_0.xci` | PCIe IP — **same IDs**, **BAR enables/sizes/types**, link speed/width, capability enables. |
| `**/src/pcileech_tlps128_bar_controller.sv` | BAR decode; MRd→**CplD**; MWr posted handling; **byte enables**; per-BAR isolation. |
| `**/src/pcileech_pcie_tlp_a7.sv` | TLP mux/demux; DMA-side traffic; possible **MSI-X doorbell** injection depending on fork. |
| `**/src/pcileech_*_top.sv` | Clock/reset/integration — training failures often trace here + constraints. |
| `**/*.xdc` | Timing/pin constraints — required for timing closure. |
| `vivado_generate_project_<board>.tcl` | Canonical project regeneration script name pattern. |

**IP lock commands (verbatim):**

```tcl
set_property -name {IP_LOCKED} -value true -objects [get_ips pcie_7x_0]
```

```tcl
set_property -name {IP_LOCKED} -value false -objects [get_ips pcie_7x_0]
```

---

## §WORKFLOW_ORDERED_STEPS

Execute in order when building or substantially modifying emulation (mirror **Checklist.md**).

1. **Capture donor** — full config space dump (Arbor export or `lspci -xxxx`) + decoded interpretation; store golden hex. [SRC:JPShag §5, Checklist §1]
2. **Edit cfg SV** — IDs/class/revision/subsystem/**`cfg_cap_pointer`**/**`cfg_dsn`** + payload encoding regs. [SRC:JPShag §6, Checklist §2]
3. **Regenerate Vivado project** — `cd` board dir; `source vivado_generate_project_<board>.tcl -notrace`. [SRC:JPShag §7.1]
4. **Customize `pcie_7x_0`** — match **every** SV field that duplicates IP; configure **BARs**, link, caps. [SRC:JPShag §7.2, Checklist §4]
5. **Align BRAM/IP backing** for each enabled BAR to implemented decode depth. [SRC:JPShag §8.2.1]
6. **Implement BAR semantic behavior** — not just sizing (register map vs sparse). [SRC:JPShag §8.2.2–8.2.3]
7. **Verify interrupt path** — MSI vs MSI-X per donor; ILA proof after driver enables interrupts. [SRC:Lesson3, JPShag §8.3]
8. **Synthesize → implement → report_timing_summary** — fix **WNS < 0**. [SRC:JPShag §11.1]
9. **Generate bitstream → program → cold reboot host** (Windows especially). [SRC:JPShag §11.2–11.3]
10. **Validate ladder** — enumeration → Drvscan diff → driver bind → functional smoke → Event Viewer/dmesg clean. See **§TEST_LADDER**.

---

## §ENC_TBL_PAYLOAD

`max_payload_size_supported` / `max_read_request_size_supported` style fields use **PCIe 3-bit encoding** (repeat here so agents never re-derive from memory):

| Bytes | `3'b___` |
|-------|----------|
| 128 | `000` |
| 256 | `001` |
| 512 | `010` |
| 1024 | `011` |
| 2048 | `100` |
| 4096 | `101` |

[SRC:JPShag README §8.1.3, Checklist §5.2]

---

## §MPS_DECODE

For comparing donor vs emu **negotiated** MPS (Lesson 3 intuition):

```text
capable_bytes  = 1 << ((DevCap_dw & 7) + 7)
in_effect_bytes = 1 << (((DevCtrl_dw >> 5) & 7) + 7)
```

DevCap/DevCtrl reside in **PCIe capability structure** (Cap ID `0x10`), offsets relative to that structure’s base — locate linked list from standard config header.

[SRC:Lesson3 §2.2]

---

## §TEST_LADDER

| Step | Pass criterion |
|------|----------------|
| L1_ENUM | Correct VID/DID/class/subsystem visible (`lspci`, Device Manager Hardware Ids). |
| L2_BARS | `lspci -vvv` BAR sizes/types match donor advertisement. |
| L3_DRVSCAN | Zero mismatches vs golden donor config hex (user tool). |
| L4_DRIVER | No ⚠ in Device Manager; no immediate Code 10 after driver install/start. |
| L5_FUNCTIONAL | Class-appropriate smoke test (NIC/storage/etc.). |
| L6_LOGS | Windows Event Viewer System PCIe/driver errors absent for device; Linux `dmesg` clean post-bind. |
| L7_STRESS | Sleep/resume, reload driver — exercises PM + MSI/X paths. |

Drvscan **⊄** driver correctness → always run **L4+** after L3 passes.

---

## §SYMPTOM_ROUTE

| SYMPTOM_ID | Observation | First lever |
|------------|-------------|-------------|
| S_NONE | No device / root port fault LED | Physical seat; aux power; try **Gen1 x1** max in IP; ILA `link_up`/reset/pcie_clk if exposed. [SRC:JPShag §13.1] |
| S_BAD_ID | Unknown device / wrong HW ID | Diff SV vs IP vs golden dump; rebuild bitstream; cold reboot. |
| S_WARN_DM | ⚠ / Code 10 after bind | Event Viewer; ILA early MRd/CfgRd paths; MSI-X table programming vs emulate path **§INV_MSIX_VERIFY**. |
| S_HANG | MMIO hang / freeze | Completion timeout — BAR decode dead-end; `tready` stall; UR storm. |
| S_SLOW | Low throughput / DMA stalls | Payload sizing vs negotiated MPS; FC/`tready`; split completions. |

---

## §FAILMODE

Atomic failure modes (expand diagnostics from JPShag §10 / §13.3):

- **FM_CPL_MALFORMED** — CplD wrong Tag/RID/Completion ID/byte-count/status/Lower Address discipline.
- **FM_CPL_TIMEOUT** — Non-posted request never completed.
- **FM_BE_IGNORE** — MWr byte enables mishandled → sticky status bits / corrupted sequences.
- **FM_ORDER** — Posted vs non-posted ordering expectation violated (intermittent).
- **FM_FAKE_CAP** — Capability advertised without handler logic.

---

## §TOOL_INDEX

| Tool class | Examples |
|------------|----------|
| donor_capture | Arbor (MindShare); `lspci -nn/-vvv/-xxxx`; Windows Device Manager Details |
| static_diff | Drvscan (user); hex diff golden capture |
| host_poke | Linux `devmem2`; Windows RW-Everything [SRC:JPShag §8.2.3]; `devcon` enumeration checks [SRC:FW-G4] |
| rtl_build | Vivado (2023.x+ mentioned in JPShag README); 7-Series device support installed |
| probe | Vivado ILA (**UG939**); `(* mark_debug = "true" *)` nets [SRC:JPShag §12.1] |
| bus_trace | Teledyne/Keysight-class PCIe analyzer (gold standard for TLP parity) |

Lab BIOS note: IOMMU/VT-d and Windows Kernel DMA Protection/HVCI often **disabled for lab** — behavior differs from hardened production hosts. [SRC:JPShag §3.2]

---

## §VENDOR_MMIO_PROTOCOL

**Canonical:** **§RM_S092_EMULATING_VENDOR_SPECIFIC_FEATURES** — full numbered workflow lives there (avoid drifting duplicates).

---

## §EXT_CAPS_EXAMPLES

**Canonical:** **§RM_S091_IMPLEMENTING_ADVANCED_PCIE_CAPABILITIES** — extended cap list + agent stub/read-write rule.

---


## §SYNTH_WARNINGS

Investigate (do not waive blindly):

- `[Synth 8-327]` — unconnected ports / floating inputs
- `[Synth 8-256]` — inference / optimization anomalies

[SRC:JPShag §11.1.1]

---

## §CDC_NOTE

Multi-clock designs: synchronizers, async FIFOs, gray code for counters; run Vivado CDC reporting. [SRC:JPShag §14.1]

---

## §SECURITY_NOTE

DMA emulation is dual-use; authorized testing only; restore platform protections on non-lab systems. [SRC:JPShag §15.3]

---

## §DOC_PAGEREF_AMD

| Doc ID | Use |
|--------|-----|
| PG054 | 7-series PCIe hard block — authoritative interface semantics |
| UG904 / UG901 | Implementation / synthesis context |
| UG939 | ILA |

---

## §README_MIRROR_INDEX

Dense mirror of JPShag **README.md** (§§2–14) for agents. **Authoritative prose** remains at `readme_authoritative_raw_url` (frontmatter); this section is **operational compression** + stable anchors.

| Anchor | JPShag README section |
|--------|------------------------|
| §RM_S02 | Key Definitions |
| §RM_S03 | Device Compatibility (+ §3.1–3.3) |
| §RM_S04 | Requirements (+ §4.1–4.3) |
| §RM_S05 | Gathering Donor Device Information (+ §5.1–5.2) |
| §RM_S06 | Initial Firmware Customization (+ §6.1–6.2) |
| §RM_S07 | Vivado Project Setup (+ §7.1–7.2) |
| §RM_S08 | Advanced Firmware Customization (+ §8.1–8.3) Part 2 |
| §RM_S09 | Emulating Device-Specific Capabilities (+ §9.1–9.2) |
| §RM_S10 | TLP Emulation (+ §10.1–10.2) |
| §RM_S11 | Building, Flashing, Testing (+ §11.1–11.3) Part 3 |
| §RM_S12 | Advanced Debugging (+ §12.1–12.2) |
| §RM_S13 | Troubleshooting (+ §13.1–13.3) |
| §RM_S14 | Emulation Accuracy (+ §14.1–14.2) |

---

## §README_MIRROR_AGENT_NOTES

### §RM_S02_KEY_DEFINITIONS

| Term | Agent-use summary |
|------|-------------------|
| **DMA** | Peripheral ↔ host RAM without per-byte CPU involvement; FPGA DMA path must stay coherent with probe/driver expectations. |
| **PCIe** | Layered serial fabric; enumeration + runtime traffic are **TLPs** on trained link. |
| **TLP** | MRd/MWr/Cpl/CplD/CfgRd/CfgWr/VDM; completions must mirror **Tag**, **Requester ID**, **Completer ID**, **byte count**, **status**, byte enables as applicable. |
| **BAR** | Declares MMIO/IO **window size/type/prefetch**; host assigns base; FPGA **decodes offsets** and responds with legal data/completions. |
| **FPGA / Bitstream** | Synthesized/implemented config → `.bit` programmed via JTAG. |
| **MSI / MSI‑X** | Message interrupts via memory writes; cfg-space caps + **runtime doorbell behavior** must match donor — **§INV_MSIX_VERIFY**. |
| **DSN** | 64-bit extended capability; drivers may key/license on it — mirror donor if present. |
| **Configuration space** | Type-0 EP: standard header + caps (+ extended config); **pointer chain** must be self-consistent. |
| **Donor** | Golden reference device for IDs/BARs/caps/**behavior**. |
| **Root Complex / Endpoint** | Host bridge vs leaf device — emu target is **EP**. |
| **HDL** | PCILeech path is **SystemVerilog**-heavy (`*.sv`). |

**Agent rule:** When a term affects **both** IP GUI and SV (`cfg_*`), diff **both** after every change. [SRC:JPShag README §2]

---

### §RM_S03_DEVICE_COMPATIBILITY

#### §RM_S031_SUPPORTED_FPGA_BASED_HARDWARE

| Platform class (guide) | Agent notes |
|------------------------|-------------|
| **Artix-7 35T** (“Squirrel” class) | User **CaptainDMA 35T** — treat as same **resource class**; confirm board variant Tcl name in clone. |
| **Artix-7 75T** (Enigma-X1) | More BRAM/LUT headroom for wider BAR backing / logic. |
| **Artix-7 100T** (ZDMA) | Higher throughput headroom. |
| **Kintex-7** | Higher lane/GT capability — not user board but principles transfer; IP family differs — always select matching **device family** in Vivado. |

**VERIFY:** `vivado_generate_project_<board>.tcl` exists for **this** board folder before editing sources. [SRC:JPShag README §3.1]

#### §RM_S032_PCIE_HARDWARE_CONSIDERATIONS

| Topic | Lab recommendation (guide) | Agent implication |
|-------|---------------------------|-------------------|
| **IOMMU / VT-d / AMD-Vi** | Often **disabled** for unrestricted DMA lab work | Enumerated addresses vs physical RAM behavior changes when re-enabled — document test matrix. |
| **Kernel DMA Protection / HVCI / Memory integrity (Win)** | Often **disabled** on lab images | Same as above; production hosts diverge. |
| **Secure Boot** | May need off when HVCI coupled | Note dependency chain in user runbook. |
| **Thunderbolt security** | Lower levels may be needed for TB-attached experiments | Usually N/A for desktop PCIe slot FPGA cards. |
| **Physical slot** | Match **lane width** connector (x1 card in x16 socket OK electrically often x1 trained); donor **link width** should not exceed **motherboard + FPGA** wiring | IP **max link width** capped by smallest of {donor cap, FPGA hardware, slot wiring}. |

**VERIFY:** BIOS PCIe Gen forced downgrade if training unstable (`S_NONE`). [SRC:JPShag README §3.2]

#### §RM_S033_SYSTEM_REQUIREMENTS

| Resource | Guide guidance | Agent checkpoint |
|----------|----------------|------------------|
| **CPU** | Modern multi-core | Vivado parallel jobs benefit from cores. |
| **RAM** | ≥16 GB, **32 GB+** ideal | OOM during route → reduce jobs / close apps. |
| **Disk** | **SSD**, guide cites large free space (order **100–200 GB** class for toolchain + runs) | Monitor `impl_1` run folders growth. |
| **OS** | Win10/11 x64 Pro/Ent or Linux x64 (Ubuntu LTS common) | Paths: Tcl `cd` uses **`/`** even on Windows. |
| **Peripherals** | JTAG programmer (Platform Cable USB II, Digilent HS2/HS3, or onboard FTDI-JTAG), spare USB, correct slot | Programming fails → driver/JTAG chain before blaming RTL. |

[SRC:JPShag README §3.3]

---

### §RM_S04_REQUIREMENTS

#### §RM_S041_HARDWARE

| Item | Role |
|------|------|
| **Donor PCIe device** | Source of golden config + behavioral traces. |
| **DMA FPGA card** | Target of bitstream — user: CaptainDMA 35T. |
| **JTAG programmer** | Loads `.bit`; enables ILA debug over same path. |

#### §RM_S042_SOFTWARE

| Component | Agent action |
|-----------|--------------|
| **Xilinx Vivado** | Install **7 Series** device support; prefer recent stable (guide mentions **2023.x+**). |
| **VS Code** (+ SV extension, e.g. mshr-h Verilog-HDL/SystemVerilog) | Edit `*.sv`; keep tab policy consistent with repo. |
| **`pcileech-fpga` clone** | `git clone https://github.com/ufrisk/pcileech-fpga.git` — user may use fork with CaptainDMA tree. |
| **Arbor** | Donor cfg decode GUI — capture exports. |
| **Alternatives** | Telescan PE (registration); `lspci -nn/-vvv/-xxxx`; Device Manager **Hardware Ids**. |

#### §RM_S043_ENVIRONMENT_SETUP

1. Install Vivado → include **Synthesis, Implementation, Programming & Debugging**, correct **family**.  
2. Install VS Code + SV highlighting/lint optional.  
3. Clone `pcileech-fpga`; `cd` into **board-specific** subdirectory (never assume `pcileech-wifi-main`).  
4. Prefer **dedicated lab machine/VM** for BIOS/security-relaxed testing; disable AV interference cautiously during JTAG sessions only if policy allows.

[SRC:JPShag README §4]

---

### §RM_S05_GATHERING_DONOR_DEVICE_INFORMATION

#### §RM_S051_USING_ARBOR_FOR_PCIE_DEVICE_SCANNING

**Procedure (agent-automatable checklist):**

1. Install Arbor (administrator install if Windows UAC prompts).  
2. Open **Local System** → **Scan / Rescan**.  
3. Identify donor by **VID:DID**, slot/BDF, OEM string.  
4. Open device → **PCI Config** / decoded cfg view.  
5. Export / screenshot **raw + decoded** — store as golden artifact tied to **donor firmware revision**.

**FAIL:** Multiple identical IDs — disambiguate by **physical slot / BDF**. [SRC:JPShag README §5.1]

#### §RM_S052_EXTRACTING_AND_RECORDING_DEVICE_ATTRIBUTES

**Must-record fields (enumeration & Drvscan):**

| Field | Width / notes |
|-------|----------------|
| Vendor ID, Device ID | 16-bit each |
| Subsystem Vendor ID, Subsystem ID | 16-bit each |
| Revision ID | 8-bit |
| Class Code | 24-bit (base/sub/prog IF) |
| **BAR0–BAR5** | Each: enabled?, **size**, MEM vs IO, **32 vs 64-bit**, **prefetch** |
| **Capabilities** | PM caps; PCIe cap link fields; **MSI vs MSI‑X** vector counts |
| **Extended:** **DSN**, **AER**, others | Record **offsets** + payload |
| **PCIe link** | Advertised max speed/width (from PCIe capability) |

**Storage format for agents:** structured doc (YAML/CSV/Markdown table) **plus** raw hex dump (`lspci -xxxx` or Arbor export) for bitwise tools.

[SRC:JPShag README §5.2]

---

### §RM_S06_INITIAL_FIRMWARE_CUSTOMIZATION

#### §RM_S061_MODIFYING_CONFIGURATION_SPACE

| Symbol (typical) | Action |
|------------------|--------|
| `cfg_vendorid`, `cfg_deviceid` | Hex `16'hXXXX` from donor |
| `cfg_subsysvendorid`, `cfg_subsysid` | From donor |
| `cfg_revisionid` | `8'hRR` |
| `cfg_classcode` | `24'h______` |
| `cfg_cap_pointer` | **First** capability offset — **must** match donor chain entry |
| Payload encoding regs | Use **§ENC_TBL_PAYLOAD** consistent with IP GUI |

**Primary file pattern:** `**/src/pcileech_pcie_cfg_a7.sv` (board-dependent naming possible).

**CHECK:** After edits, grep duplicate identifiers also inside **`pcie_7x_0`** customization — **INV_DUAL_SOURCE**. [SRC:JPShag README §6.1]

#### §RM_S062_INSERTING_THE_DEVICE_SERIAL_NUMBER_DSN

| Condition | `cfg_dsn` action |
|-----------|------------------|
| Donor exposes DSN | Set **64-bit** donor value |
| No DSN / confirmed irrelevant | `64'h0` (document rationale) |
| Drvscan/driver expects non-zero | Mirror donor — do not invent branded serials arbitrarily |

[SRC:JPShag README §6.2]

---

### §RM_S07_VIVADO_PROJECT_SETUP_AND_CUSTOMIZATION

#### §RM_S071_GENERATING_VIVADO_PROJECT_FILES

```tcl
cd <ABS_PATH_TO_BOARD_DIR_USING_FORWARD_SLASHES>
pwd
source vivado_generate_project_<board>.tcl -notrace
```

**THEN:** `File → Open Project → *.xpr` (e.g. `pcileech_squirrel_top.xpr` pattern — actual name depends on board).

**VERIFY:** Sources pane lists `*.sv`, `*.xdc`, IP cores; Messages clean on open when possible. [SRC:JPShag README §7.1]

#### §RM_S072_MODIFYING_IP_BLOCKS

| IP artifact | Actions |
|-------------|---------|
| `pcie_7x_0.xci` | Right-click → **Customize IP** |
| **Identification** tab | Mirror **all** IDs/class/revision/subsystem consistent with SV |
| **BARs** tab | Enable/disable per donor; size/type/prefetch/**64-bit** match |
| **Link** | Max speed/width ≤ feasible hardware |
| **Capabilities** | PM, MSI/MSI‑X, extended features — **only if handled** |
| Post-edit | Regenerate output products; fix Critical Warnings |
| Lock | `set_property IP_LOCKED true` on `pcie_7x_0` — see **§FILE_CONTRACT** |

[SRC:JPShag README §7.2]

---

### §RM_S08_ADVANCED_FIRMWARE_CUSTOMIZATION

#### §RM_S081_CONFIGURING_PCIE_PARAMETERS_FOR_EMULATION

| Parameter set | Where configured | Agent verification |
|---------------|------------------|-------------------|
| **Max link speed / link width** | PCIe IP **Link / PCIe Capabilities** | Matches donor **advertised** caps, capped by FPGA/socket |
| **`cfg_cap_pointer`** | `pcileech_pcie_cfg_a7.sv` | Points to **first** cap structure; 4-byte aligned |
| **Max payload / max read request supported** | IP **Device Capabilities** area + SV regs | Encodings **§ENC_TBL_PAYLOAD** |
| **Negotiated MPS** runtime | OS + bridge programming | Compare donor vs emu with `lspci -vv` after boot — **§MPS_DECODE** |

[SRC:JPShag README §8.1]

#### §RM_S082_ADJUSTING_BARS_AND_MEMORY_MAPPING

| Layer | Task |
|-------|------|
| **IP BAR tab** | Legal PCIe advertisement |
| **BRAM IP / external memory** | Depth covering implemented offsets (resource limit on 35T) |
| **`pcileech_tlps128_bar_controller.sv`** | Decode `bar_hit[*]` / addresses; drive **CplD** for MRd; apply **byte enables** on MWr |
| **Multiple BARs** | Isolated `if/else if bar_hit[i]` branches; **no internal overlap** |

**TEST:** Scripted MRd/MWr across BAR boundaries + default/unmapped offsets (return safe patterns consistent with donor if known). [SRC:JPShag README §8.2]

#### §RM_S083_EMULATING_PM_AND_INTERRUPTS

| Concern | Implementation hints |
|---------|----------------------|
| **PM capability** | Enable in IP if donor has PM; reflect supported **D-states** — avoid advertising unsupported deep sleeps |
| **PMCSR behavior** | Core handles much protocol — user logic may need **clock gating**/DMA disable on low power transitions |
| **MSI** | Often via PCIe core **cfg_interrupt** family signals — confirm signal names in **PG054** wrapper |
| **MSI‑X** | May require **explicit MWr/MWr64 TLP** “doorbell” via TX stream — **§INV_MSIX_VERIFY** |
| **Interrupt timing** | Pulse vs level expectations — ILA `msi_req`/`cfg_interrupt` vs driver ISR enrollment |

[SRC:JPShag README §8.3]

---

### §RM_S09_EMULATING_DEVICE_SPECIFIC_CAPABILITIES

#### §RM_S091_IMPLEMENTING_ADVANCED_PCIE_CAPABILITIES

**Examples called out in guide:** `AER`, `DSN`, `VC/MFVC`, `PTM`, `LTR`, `FLR`.

**Agent rule:** For each enabled extended cap, enumerate **which registers** the donor driver reads/writes; stub reads must return **legal reset defaults**, writes must **update shadow state** or silently discard per donor behavior — never leave analyzer-visible **non-spec** responses if avoidable.

[SRC:JPShag README §9.1]

#### §RM_S092_EMULATING_VENDOR_SPECIFIC_FEATURES

**Workflow:**

1. **Trace MMIO** (analyzer) during driver init + workload — record address, length, BE, data patterns.  
2. **RE driver** static (`Ghidra`/`IDA`) for magic offsets and command SM states.  
3. Implement **`case`/FSM** in BAR logic; optional **VDM** RX/TX if donor uses vendor-defined messages.  
4. **Regression:** side-by-side trace donor vs FPGA for critical sequences.

[SRC:JPShag README §9.2]

---

### §RM_S10_TRANSACTION_LAYER_PACKET_TLP_EMULATION

#### §RM_S101_UNDERSTANDING_AND_CAPTURING_TLPS

| Concept | Agent notes |
|---------|-------------|
| **Header fields** | Fmt/Type, Length, RID, Tag, Addr, First/Last DW byte enables, TC/Attr — vary by TLP type |
| **Common types** | MRd → CplD; MWr posted (no completion unless error path); CfgRd/CfgWr completions |
| **Capture method** | Inline **protocol analyzer** gold standard; document **enumeration burst** separately from **steady-state** |
| **Documentation** | For each critical sequence store: **order**, **timing gaps**, **payload hex**, expected responses |

[SRC:JPShag README §10.1]

#### §RM_S102_CRAFTING_CUSTOM_TLPS_FOR_SPECIFIC_OPERATIONS

| Activity | File / interface |
|----------|------------------|
| Parse RX from core | User RX AXI-Stream (`s_axis_rx_*` naming per PG054) |
| Emit completions | BAR controller + TX mux coordination |
| Emit MSI-X doorbell | Often **`pcileech_pcie_tlp_a7.sv`** TX path |
| Compliance | Honor negotiated **MPS**, proper **Cpl status**, observe **`tready`** |

**FAILMODES:** **§FAILMODE**, **§RM_S133**. [SRC:JPShag README §10.2]

---

### §RM_S11_BUILDING_FLASHING_AND_TESTING

#### §RM_S111_SYNTHESIS_AND_IMPLEMENTATION

| Stage | Pass gate |
|-------|-----------|
| **Synth** | 0 errors; investigate **[Synth 8-327]**, **[Synth 8-256]** — **§SYNTH_WARNINGS** |
| **Synth util** | BRAM/LUT within device |
| **Impl timing** | **WNS ≥ 0** required domains |
| **Timing failure fix** | Pipeline / refactor / constraint fixes — not “hope” |

Bitstream path pattern: `*.runs/impl_1/*.bit` (exact project basename varies). [SRC:JPShag README §11.1]

#### §RM_S112_FLASHING_THE_BITSTREAM

1. Board seated; JTAG connected; host powered.  
2. Vivado **Hardware Manager → Open Target → Auto Connect**.  
3. Right-click FPGA → **Program Device** → select `.bit`.  
4. **Cold reboot** host OS if enumeration stale (especially Windows).

[SRC:JPShag README §11.2]

#### §RM_S113_TESTING_AND_VALIDATION

Use **§TEST_LADDER** verbatim gates.

Additional guide bullets:

| Host | Checks |
|------|--------|
| **Windows** | Device Manager tree placement + **Hardware Ids** property strings |
| **Linux** | `lspci -nn`, `lspci -vvv` BAR decode |
| **Logs** | Event Viewer **System** (PCIe/driver); `dmesg`/`journalctl` PCIe filtered |
| **Functional** | Class-specific exercises (NIC/storage/USB hub emulation patterns listed narratively in README §11.3.2) |

[SRC:JPShag README §11.3]

---

### §RM_S12_ADVANCED_DEBUGGING_TECHNIQUES

#### §RM_S121_USING_VIVADO_ILA

| Step | Detail |
|------|--------|
| Plan probes | RX/TX stream `tdata/tvalid/tready/tlast/tkeep`, BAR state, interrupt strobes, `bar_hit` |
| Insert ILA IP | Match clock domain to sampled signals |
| `mark_debug` | Attribute nets in HDL for quick probe promotion |
| Trigger | Example: specific **Fmt/Type** nibble on RX `tdata`, or BAR offset compare |
| Arm | Hardware Manager → capture → waveform correlation with host driver timestamps |

[SRC:JPShag README §12.1]

#### §RM_S122_PCIE_TRAFFIC_ANALYSIS_TOOLS

| Tier | Tools |
|------|-------|
| **Hardware analyzer** | Teledyne LeCroy / Keysight-class — full TLP decode & triggering |
| **Software adjunct** | `lspci` variants; PCILeech client tests DMA plumbing — **not** full-bus passive observe |
| **Comparison discipline** | Export donor trace → export emu trace → diff transaction list |

[SRC:JPShag README §12.2]

---

### §RM_S13_TROUBLESHOOTING

#### §RM_S131_DEVICE_DETECTION_ISSUES

| Cause cluster | Indicators | Fix vectors |
|-----------------|------------|-------------|
| **ID mismatch** | Unknown device, wrong driver bind | SV/IP/golden diff; rebuild; cold boot |
| **Link training fail** | Nothing at BDF / root error | Seat/power/aux PCIe; lower Gen/width; verify resets/clocks constraints |
| **Power** | Random disappearance | PSU/aux cables |
| **RTL wrapper errors** | Synth critical warnings on PCIe instance | Fix hierarchy before silicon trial |
| **Early ILA** | Link never `link_up` | Probe PHY/status outputs permitted by core |

[SRC:JPShag README §13.1]

#### §RM_S132_MEMORY_MAPPING_AND_BAR_CONFIGURATION_ERRORS

| Cause cluster | Indicators | Fix vectors |
|-----------------|------------|-------------|
| **BAR size/type mismatch** | Driver BAR remap quirks / fault on first MMIO | IP BAR tab vs Arbor |
| **BRAM too shallow** | Wrong data beyond implemented depth | Resize BRAM or shrink advertised BAR (policy decision — user approval) |
| **Decode bug** | MRd timeout / BSOD on specific offset | ILA address path; fix `case`/ranges |
| **Overlap** | Heisenbugs on concurrent BAR traffic | Red-team internal memory map |

[SRC:JPShag README §13.2]

#### §RM_S133_DMA_PERFORMANCE_AND_TLP_ERRORS

| Cause cluster | Indicators | Fix vectors |
|-----------------|------------|-------------|
| **Malformed TLP** | Analyzer flags | Header builder audit |
| **FC / backpressure** | Intermittent stall | Respect **`tready`**; FIFO depths |
| **DMA engine inefficiency** | Low throughput | Burst sizing, pipelining |
| **Cpl timeout / UR / CA** | Hang / driver abort | Completion FSM audit; BAR decode completeness |

[SRC:JPShag README §13.3]

---

### §RM_S14_EMULATION_ACCURACY_AND_OPTIMIZATIONS

#### §RM_S141_TECHNIQUES_FOR_ACCURATE_TIMING_EMULATION

| Technique | Application |
|-----------|-------------|
| **XDC discipline** | All clocks defined; false paths reviewed intentionally |
| **Timing closure** | Positive **WNS** — prerequisite for fair behavioral comparison |
| **CDC** | **§CDC_NOTE** — synchronizers / async FIFO / gray |
| **Simulation** | SV testbench BFMs for BAR MRd/MWr smoke before silicon loops |
| **Accelerated models** | Optional delays if donor timing-sensitive (advanced) |

[SRC:JPShag README §14.1]

#### §RM_S142_DYNAMIC_RESPONSE_TO_SYSTEM_CALLS

| Mechanism | Agent implementation |
|-----------|------------------------|
| **Cfg wr mid-runtime** | Driver may flip command registers via cfg space — shadow/update SV-visible registers |
| **BAR command regs** | FSM per donor protocol — idle/init/active/error |
| **PM transitions** | Gate clocks/DMA per **D-state** |
| **Interrupt acknowledge paths** | Some drivers clear IRQ via MMIO — mirror clears |
| **Vendor commands / VDM** | Extend decode tables when traces prove necessity |

[SRC:JPShag README §14.2]

---

## §CHANGELOG

| ISO Date | Change |
|----------|--------|
| 2026-05-15 | Human-expanded notes from JPShag materials. |
| 2026-05-15 | Restructured for **agent consumption**: YAML frontmatter, invariants, file contracts, routable IDs, removed conversational redundancy. |
| 2026-05-15 | **Option A:** added `readme_authoritative_raw_url` + expanded **`§README_MIRROR_*`** covering JPShag README §§2–14 per user outline (dense tables/procedures). |
| 2026-05-15 | **§CODING_SESSION_PROTOCOL** + **§AGENT_DIRECTIVE** updates: mandatory official README consult when stuck/uncertain; Before/During/After phase documentation for coding; YAML `on_uncertainty_or_blocker` / `coding_phase_protocol`. |
| 2026-05-15 | **`firmware-workspace/`** implementation package + **PROJECT_CONTEXT** CaptainDMA 4.1th path hints (`35t484_x1`, `vivado_generate_project_captaindma_35t.tcl`). |
