# Lawful donor scope (plan todo: donor-lawful-scope)

Firmware emulation work must stay within **authorized** hardware analysis and testing — consistent with [FPGA-DMA-EMULATION-STUDY-NOTES.md](../FPGA-DMA-EMULATION-STUDY-NOTES.md) **§SECURITY_NOTE** and **INV_LEGAL**.

## Confirm before golden capture

Check each box when true:

- [ ] Donor hardware is **owned by me** or I have **written authorization** from the owner to capture config space / traces / firmware emulation against it.
- [ ] The goal is **engineering validation** on designated lab systems (enumeration, BAR correctness, Drvscan-style checks, driver health), **not** evasion of security products or anti-cheat on third-party systems.
- [ ] I understand **FULL_EMU** requires behavioral parity (MMIO/interrupts), not only copying VID/DID.

## Donor strategy (pick one and note BDF)

- [ ] **Discrete PCIe card** — removable; capture while installed in lab PC  
  - Slot / BDF: `________________`  
- [ ] **Onboard PCIe function** — capture only on machines I control  
  - Machine ID / BDF: `________________`

## Sign-off

| Field | Value |
|-------|-------|
| Date | ________ |
| Initials | ________ |

*(Electronic checklist — fill when starting donor capture.)*
