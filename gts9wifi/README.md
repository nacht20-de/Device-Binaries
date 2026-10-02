## Firmware Infos

- **Device:** Samsung Galaxy Tab S9 Wi-Fi (SM-X710)
- **Region:** EUX
- **Version:** `X710XXS5BYA5` / `BOOT.MXF.2.1.1-00218-KAILUA-1`

## Notes

Same Qualcomm build as `dm2q`. Except for the files listed below, every binary
and RawFile is byte-identical to `dm2q`, so the dm2q patches apply unchanged.

Device-specific (differ from dm2q, taken from this firmware, unpatched):
Ebl, DisplayDxe, VariableDxe, XBLCore, and SamsungDxe CcicDxe, ChgDxe, MuicDxe,
RedriverDxe, SecEnvDxe, SubPmicDxe, VibDxe.

## Patches / Fixes

### ClockDxe:

- **Reason:** To keep Display turned on while UEFI Boot.
- **Patch:** The DCD Disable Dependencies Function Call has been Removed.
- **Patch Creator:** [Gustave Monce](https://github.com/gus33000)
- **Note:** Identical to dm2q's patched file.

### PmicDxe:

> [!NOTE]
> Must be paired with the SPMIDxe Patch.

- **Reason:** To make UEFI not Crash during UEFI Boot.
- **Patch:** Minimal PMIC Init has been Removed to avoid a Crash.
- **Patch Creator:** [Kancy Joe](https://github.com/sunflower2333)
- **Note:** Identical to dm2q's patched file.

### SPMIDxe:

> [!NOTE]
> Must be paired with the PmicDxe Patch.

- **Reason:** To make UEFI not Crash during UEFI Boot.
- **Patch:** Removed the SPMI PIC Init Function.
- **Patch Creator:** [Kancy Joe](https://github.com/sunflower2333)
- **Note:** Identical to dm2q's patched file.

### TzDxeLA:

- **Reason:** To make UEFI not Crash during UEFI Boot.
- **Patch:** The Global TZ Applet Variable has been Changed to `TRUE` from `FALSE`.
- **Patch Creator:** [N1kroks](https://github.com/N1kroks/)
- **Note:** Identical to dm2q's patched file.

### UFSDxe:

> [!TIP]
> UFS will still enter Sleep State after Exit Boot Services. <br>
> To prevent this, Set `UEFIExitUfsSSURequired` to `0` in the Configuration Map.

- **Reason:** To allow the usage of UFS.
- **Patch Nr. 1:** The UFS Sleep call has been Replaced with the UFS Wakeup Call.
- **Patch Nr. 2:** Added UFS Link Wake Up.
- **Patch Creators:** [Kancy Joe](https://github.com/sunflower2333) & [N1kroks](https://github.com/N1kroks)
- **Note:** Identical to dm2q's patched file.

### UsbConfigDxe:

- **Reason:** To allow the usage of the USB Port.
- **Patch:** Removed IOMMU Detach from Exit Boot Services.
- **Patch Creator:** [Gustave Monce](https://github.com/gus33000)
- **Note:** This firmware's UsbConfigDxe differs from dm2q's in 4 debug-metadata bytes only.
  The same 3-byte patch (`0x55DC`: `A8 01 00 34` -> `0D 00 00 14`) was re-applied to this build.

### UsbMsdDxe:

- **Reason:** For better Mass Storage usage.
- **Patch:** Changed Removable State to Non-Removable.
- **Patch Creator:** [N1kroks](https://github.com/N1kroks)
- **Note:** Identical to dm2q's patched file.

### ButtonsDxe:

- **Reason:** To make the Power Button usable as Enter (Key Confirm) in UEFI.
- **Patch:** The Power Key (id 1) emitted `ScanCode 0x80` only, which no UEFI
  menu consumes; it now emits `UnicodeChar 0x0D` (`CHAR_CARRIAGE_RETURN`)
  with `ScanCode 0` instead. Volume Keys are unchanged (their values keep the
  high half zero; `UnicodeChar` is pre-zeroed in the handler).
- **Patch Details:** Two instructions in the Key Handler
  (file offsets `0x3ACC` and `0x3AE0`, identical to RVAs in this PE):
  `movz w8, #0x80` -> `movz w8, #0xD, lsl #16` and
  `strh w8, [x29, #0x18]` -> `str w8, [x29, #0x18]`, turning the 16-bit
  ScanCode-only store into a 32-bit store of `{ScanCode 0, UnicodeChar CR}`
  into the queued `EFI_INPUT_KEY`.
- **Patch Creator:** [nacht20-de](https://github.com/nacht20-de), adapted from
  the gts8p ButtonsDxe patch by [Robotix22](https://github.com/Robotix22)
  (which achieves the same Power-as-Enter behavior with a branch rewrite).

