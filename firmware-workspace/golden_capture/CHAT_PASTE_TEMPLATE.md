# Golden capture — copy into chat when filled

**Instructions:** Fill every `FILL_*` block. Paste the whole file (or this section) into Cursor chat when done.

---

## A — Which device (exactly one)

**Target:** The **PCI Express** function that is the **Intel High Definition Audio host controller** on your Z390 (the chip that owns PCI config space + BARs for onboard HD Audio).

**Not the target (usually):** *Sound, video and game controllers* → **High Definition Audio Device** with **Manufacturer: Microsoft** and **Location: Internal High Definition Audio Bus** — that is typically a **child** on the HDAudio bus, **not** the PCIe config-space donor.

### How to find it in Device Manager

1. Open **Device Manager**.
2. Open **View** → **Devices by connection** (helps see tree).
3. Expand **System devices** (keep **Sound, video…** closed at first).
4. Look for a name like:
   - **Intel(R) High Definition Audio Controller**, or  
   - **Intel(R) Smart Sound Technology** / similar **Intel** + **Controller** wording.
5. **Right-click** that entry → **Properties** → confirm **Details** tab shows **Hardware Ids** starting with `PCI\VEN_8086` (Intel) or another vendor if your board uses a different PCH audio block — **still must be the PCIe controller**, not the Microsoft HDA bus device.

**If you cannot find it:** In **View → Resources by type** or search Device Manager for **8086** in Details (slow). Easiest path is **Arbor** PCI tree (next section).

**Selected device display name (exact string from Device Manager):**  
`FILL_DEVICE_MANAGER_NAME`

---

## B — Device Manager → Details (minimum)

On the **PCIe controller** from section A:

**Properties → Details tab.** For each property below, copy value into the table.

| Property (dropdown) | Value |
|----------------------|-------|
| Hardware Ids | `FILL` |
| Compatible Ids | `FILL` |
| Device instance path | `FILL` |
| Location information | `FILL` |
| Bus number / PCI bus (if shown) | `FILL` |
| Class GUID | `FILL` |
| Driver INF | `FILL` |

---

## C — PowerShell (optional quick copy)

Run **Windows PowerShell** as normal user (admin not always required):

```powershell
Get-PnpDevice -Class MEDIA,SYSTEM | Where-Object { $_.FriendlyName -match 'Audio|HDA|Smart Sound' } |
  Format-Table Status, Class, FriendlyName, InstanceId -AutoSize
```

Copy the **full table output** into chat (redact nothing except serials you consider sensitive).

---

## D — Strong capture (pick one; Arbor preferred per JPShag README §5)

### Option 1 — Arbor

1. Install **Arbor** (MindShare), run as admin if prompted.
2. **Local System** → **Scan** / **Rescan**.
3. In the device list, find the **same** device as section A (match **VID:DID** from Hardware Ids, e.g. `VEN_8086&DEV_xxxx`).
4. Open **PCI Config** / configuration view.
5. Export or screenshot:
   - Full **decoded** capability list + BARs.
   - Raw **hex** dump if Arbor can export it.

**Paste:** summary text + attach export file in chat if small, OR paste hex as code block.

### Option 2 — PCI-Z

1. Download **PCI-Z** (portable), run.
2. Find the PCI device row matching **VID/DID** from section B.
3. Open **Configuration space** / dump view.
4. Copy **256 bytes** (minimum) as hex or export.

**Paste:** hex block or describe file attached.

---

## E — Parsed fields (for `donor_decoded.yaml` later)

From Hardware Ids `PCI\VEN_vvvv&DEV_dddd&SUBSYS_ssssbbbb&REV_rr` (example shape):

| Field | Hex value |
|-------|-----------|
| Vendor ID | `0xFILL` |
| Device ID | `0xFILL` |
| Subsystem Vendor ID | `0xFILL` |
| Subsystem ID | `0xFILL` |
| Revision ID | `0xFILL` |
| Class Code (24-bit) | `0xFILL` (from Arbor if not obvious from INF) |

**BAR summary** (from Arbor / PCI-Z only — do not guess):

| BAR | Enabled | Size | Type (32/64 MEM, IO) | Prefetch |
|-----|---------|------|----------------------|----------|
| 0 | | | | |
| 1 | | | | |
| … | | | | |

**First capability pointer (hex offset):** `0xFILL`  
**MSI / MSI-X / PM notes:** `FILL`  
**DSN present?** yes/no — if yes: `0x................`  

---

## F — Screenshots (optional but helpful)

Attach if easy:

1. Device Manager **System devices** showing the **PCIe controller** entry selected.  
2. **Details → Hardware Ids** for that same device.  
3. Arbor or PCI-Z main window for that BDF.

(Repo already has earlier screenshots; new ones after you identify the **controller** are even better.)

---

## G — Host context (one line each)

| Item | Value |
|------|-------|
| OS | e.g. Windows 10 Pro 19045 |
| Board | MSI MPG Z390 GAMING EDGE AC |
| Secure Boot | On/Off |
| Kernel DMA Protection | On/Off |

When this template is filled, paste into chat → next step is converting to `golden_capture/donor_decoded.yaml` in the repo.
