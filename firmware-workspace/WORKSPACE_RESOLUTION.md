# Workspace resolution (§CODING_SESSION_PROTOCOL B1)

## Status: `thirty-five` repo

| Check | Result |
|-------|--------|
| `pcileech-fpga` present under this workspace | **No** — only study notes live here today |
| Next action | Clone upstream (or fork) into a path **you** choose; record it in [SESSION_PROGRESS.md](SESSION_PROGRESS.md) |

## Upstream repository

- **Primary:** https://github.com/ufrisk/pcileech-fpga  
- **CaptainDMA vendor readme:** https://github.com/ufrisk/pcileech-fpga/blob/master/CaptainDMA/readme.md  

### Clone (example)

```bash
cd C:\Users\dh-m\Documents\fpga
git clone https://github.com/ufrisk/pcileech-fpga.git
```

Adjust drive/path to match your machine.

---

## CaptainDMA 35T **4.1th** (desktop PCIe) — resolved paths

Per **CaptainDMA/readme.md** firmware table, **CaptainDMA 4.1th** maps to FPGA project directory **`35t484_x1`**.

| Artifact | Path relative to repo root |
|----------|----------------------------|
| Board / RTL / IP | `CaptainDMA/35t484_x1/` |
| **Generate Vivado project** | `CaptainDMA/35t484_x1/vivado_generate_project_captaindma_35t.tcl` |
| **Batch build helper** | `CaptainDMA/35t484_x1/vivado_build.tcl` |
| Sources | `CaptainDMA/35t484_x1/src/` |
| IP | `CaptainDMA/35t484_x1/ip/` |

### Vivado Tcl console (after clone)

```tcl
cd C:/path/to/pcileech-fpga/CaptainDMA/35t484_x1
pwd
source vivado_generate_project_captaindma_35t.tcl -notrace
```

Use **forward slashes** in Tcl on Windows.

Then **File → Open Project** and select the generated `.xpr` in the same directory (exact basename is defined inside the Tcl script — inspect if unsure).

### Flashing note (hardware)

CaptainDMA readme: **4.1th** flashing follows **PCIeSquirrel** instructions but uses CaptainDMA-specific firmware image:

- https://github.com/ufrisk/pcileech-fpga/blob/master/PCIeSquirrel/readme.md  

---

## Other CaptainDMA variants (reference only)

| Hardware | Directory (upstream) | Generate script (typical) |
|----------|----------------------|---------------------------|
| CaptainDMA M2 x1 | `CaptainDMA/35t325_x1/` | `vivado_generate_project_captaindma_m2x1.tcl` |
| CaptainDMA M2 x4 | `CaptainDMA/35t325_x4/` | `vivado_generate_project_captaindma_m2x4.tcl` |
| CaptainDMA 75T | `CaptainDMA/75t484_x1/` | (verify in dir listing) |
| CaptainDMA 100T | `CaptainDMA/100t484-1/` or `100t484_x1` | (verify in GitHub tree) |

Always **`Glob`** / list the directory after clone — script names are authoritative locally.

---

## Verification checklist

- [ ] `pcileech-fpga` cloned at: `___________________________`
- [ ] Opened `CaptainDMA/35t484_x1/` and confirmed **`vivado_generate_project_captaindma_35t.tcl`** exists
- [ ] Recorded absolute paths in **SESSION_PROGRESS.md**
