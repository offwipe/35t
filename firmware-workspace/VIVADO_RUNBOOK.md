# Vivado runbook — JPShag README alignment + CaptainDMA 4.1th

When uncertain on ordering, fetch **`readme_authoritative_raw_url`** from [FPGA-DMA-EMULATION-STUDY-NOTES.md](../FPGA-DMA-EMULATION-STUDY-NOTES.md) YAML frontmatter.

## Preconditions

- [ ] Lawful donor checklist completed: [LAWFUL_DONOR_CHECKLIST.md](LAWFUL_DONOR_CHECKLIST.md)
- [ ] Golden capture started or complete: [golden_capture/README.md](golden_capture/README.md)
- [ ] `pcileech-fpga` cloned — paths: [WORKSPACE_RESOLUTION.md](WORKSPACE_RESOLUTION.md)

## Step 1 — Generate project (README §7.1)

1. Launch Vivado.
2. **Tcl Console:** `Window → Tcl Console`
3. `cd` to **CaptainDMA/35t484_x1** (use forward slashes).
4. Run:

```tcl
source vivado_generate_project_captaindma_35t.tcl -notrace
```

5. **Open generated `.xpr`** (`File → Open Project`).

## Step 2 — Edit SV configuration (README §6) — before or after IP

Typical file under `CaptainDMA/35t484_x1/src/`:

- `pcileech_pcie_cfg_a7.sv` (name confirmed after clone)

Update: `cfg_vendorid`, `cfg_deviceid`, subsystem IDs, revision, class code, **`cfg_cap_pointer`**, **`cfg_dsn`**, payload size encodings per **§ENC_TBL_PAYLOAD** in study notes.

## Step 3 — Customize PCIe IP (README §7.2)

1. Sources → IP → `pcie_7x_0.xci` → **Customize IP**
2. Match **all** identification fields with SV (**INV_DUAL_SOURCE**).
3. **BARs:** sizes/types/prefetch/disable unused — match golden donor.
4. **Link:** speed/width within FPGA + slot limits.
5. Capabilities: enable only what donor has **and** RTL honors.
6. Apply → regenerate IP output products → clear Critical Warnings.

## Step 4 — Lock IP

```tcl
set_property -name {IP_LOCKED} -value true -objects [get_ips pcie_7x_0]
```

## Step 5 — BAR behavior RTL (README §8.2)

Edit `pcileech_tlps128_bar_controller.sv` (path per clone) for decode + completions + byte enables.

## Step 6 — TLP / MSI path (README §8.3–10)

Inspect `pcileech_pcie_tlp_a7.sv` + core wrapper — **§INV_MSIX_VERIFY**.

## Step 7 — Synth / impl / timing (README §11.1)

1. **Run Synthesis** → review **[Synth 8-327]** / **[Synth 8-256]** if present.
2. **Run Implementation** → **Report Timing Summary** → **WNS ≥ 0** required.
3. **Generate Bitstream**.

Optional non-GUI batch: inspect `vivado_build.tcl` in board directory for scripted flow.

## Step 8 — Program FPGA (README §11.2)

Hardware Manager → Open Target → Program `.bit` from `*.runs/impl_1/`.

Cold reboot host if enumeration stale.

## Step 9 — Validation

Continue to [TEST_LADDER_RUNBOOK.md](TEST_LADDER_RUNBOOK.md).
