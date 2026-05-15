# Test ladder runbook (§TEST_LADDER)

Run after programming bitstream (see [VIVADO_RUNBOOK.md](VIVADO_RUNBOOK.md)).

| Step | ID | Gate | Notes |
|------|-----|------|-------|
| 1 | L1_ENUM | VID/DID/class/subsystem visible | Device Manager Hardware Ids / `lspci -nn` |
| 2 | L2_BARS | BAR sizes/types match donor | `lspci -vvv` or Arbor on emu device |
| 3 | L3_DRVSCAN | Matches golden capture | User tool vs `golden_capture/` artifacts |
| 4 | L4_DRIVER | No ⚠ / Code 10 on driver start | Install vendor/driver only if lawful test requires it |
| 5 | L5_FUNCTIONAL | Class smoke test | NIC/storage/etc. as applicable |
| 6 | L6_LOGS | Clean logs | Win: Event Viewer **System**; Linux: `dmesg` / `journalctl` |
| 7 | L7_STRESS | Sleep/resume or driver reload | PM + MSI/X paths |

## Record results

| Step | Pass/Fail | Evidence (paste path or note) |
|------|-----------|-------------------------------|
| L1 | | |
| L2 | | |
| L3 | | |
| L4 | | |
| L5 | | |
| L6 | | |
| L7 | | |

## Failure routing

See study notes **§SYMPTOM_ROUTE** and **§RM_S131–§RM_S133**.
