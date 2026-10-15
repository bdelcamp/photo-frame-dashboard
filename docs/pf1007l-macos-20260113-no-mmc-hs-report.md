# PF1007L January 2026 firmware on macOS: no-HS U-Boot exposed the eMMC

An independent Apple silicon test of a PF1007L running the January 2026 firmware, separate from
the [Intel Mac report](pf1007l-macos-20260113-report.md), found that the repository's base Route U build did not
expose the eMMC through USB mass storage. Restricting the eMMC to a 4-bit bus at 26 MHz while
leaving U-Boot's high-speed capabilities enabled did not help. A 4-bit / 26 MHz build with the
eMMC high-speed capabilities removed exposed the full eMMC as a read-only UMS disk. A complete,
segmented backup then passed the guarded verification.

This result is consistent with the no-HS behavior documented for a May 2026 unit, but no MMC trace
was captured here. It does not establish the exact failure mechanism or prove that the same change
works on every January unit.

## Unit and host

```text
Model: PF1007L
Firmware: YHK.F-null-A33-1001-D1024-PF1007L-321-OS-AIMOR.0.20260113
Android model property: AEEZO
AiMOR: 6.0.78
Host: macOS, Apple silicon
FEL: Allwinner A33, USB 1f3a:efe8
Stock USB: 1f3a:1000; no stock ADB device
eMMC: 30,777,344 sectors × 512 bytes = 15,758,000,128 bytes
```

No serial number, eMMC CID, backup hash, or Wi-Fi information is included.

## Observed results

| U-Boot configuration | Result |
| --- | --- |
| Repository base build, high-speed capabilities enabled | eMMC UMS did not appear |
| 4-bit bus / 26 MHz, high-speed capabilities still enabled | eMMC UMS did not appear |
| 4-bit bus / 26 MHz, eMMC high-speed capabilities removed | eMMC appeared as a read-only UMS disk; reported size matched 30,777,344 sectors |

The successful legacy-timing build also retained the START STOP correction and used
`CONFIG_SYS_DEVICE_NULLDEV` for the U-Boot console input. The full backup passed all 15 segments;
each segment was read twice from the disk and once from the image. An earlier attempt lost the UMS
device during segment 2, so the successful verification followed a fresh FEL/U-Boot session and a
resumed guarded backup. Disk identifiers can change between sessions; identify the eMMC afresh by
USB UMS identity, exact size, and read-only state.

## Interpretation and limits

Removing the eMMC high-speed capabilities was the configuration change associated with eMMC
enumeration on this unit. The 4-bit / 26 MHz setting alone did not help. This supports high-speed
timing as a useful diagnostic axis, but without an MMC trace it does not prove that the eMMC stopped
responding specifically after `HS_TIMING=1`, as captured on the May 2026 unit in the
[no-HS report](pf1007l-macos-20260529-no-mmc-hs-report.md).

This was a read-only U-Boot backup test, not evidence that every unit with the same firmware string
or eMMC capacity will behave the same way. Keep the no-HS change as a separately reviewed diagnostic
build; it is not a default change to Route U.
