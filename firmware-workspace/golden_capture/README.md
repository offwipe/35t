# Golden donor capture (§RM_S05, §TEST_LADDER L2/L3)

See **[GOLDEN_CAPTURE_WINDOWS11.md](GOLDEN_CAPTURE_WINDOWS11.md)** for Device Manager / Arbor / PCI-Z steps on **Windows 11**.

## Outputs (store in this folder)

| File | Description |
|------|-------------|
| `donor_decoded.yaml` | Copy from `donor_template.yaml` — filled IDs/BARs/caps |
| `donor_config_space.hex` or `.txt` | Raw dump (256 B minimum; extended if tools export 4K) |
| `donor_lspci_vvv.txt` | Optional: `lspci -vvv -s <BDF>` (Linux) |
| `screenshots/` | Arbor exports / Device Manager Details if Windows |

## Windows — quick identifiers

1. Device Manager → device → **Details** → Property **Hardware Ids** / **Device instance path** (note BDF-style info where visible).
2. Prefer **Arbor** or equivalent for full config decode per JPShag README §5.

## Linux — capture example

```bash
lspci -nn
lspci -vvv -s 03:00.0
sudo lspci -vvv -xxxx -s 03:00.0   # extended hex; requires privileges on many distros
```

Replace `03:00.0` with donor BDF.

## Drvscan / diff

After FPGA emulation builds, compare tool output against **`donor_config_space`** golden — goal: zero unexpected mismatches on fields you intentionally cloned.
