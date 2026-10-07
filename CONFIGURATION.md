# Configuration notes

## ACPI

| Table | Purpose in the original build |
|---|---|
| SSDT-PLUG-ALT | Processor objects and plugin-type for X86PlatformPlugin |
| SSDT-SBUS | BUS0 SMBus device paired with the MC__ → MCHC renames |
| SSDT-DMAC | Legacy DMA device compatibility |
| SSDT-EC | Darwin-only embedded-controller device |
| SSDT-HPET | HPET resource override, paired with the _CRS → XCRS patch |
| SSDT-USBX | USB power properties |
| SSDT-RTCAWAC | STAS-based legacy RTC selection |
| SSDT-6950XT-73A5-GPU-SPOOF | Original RX 6950 XT ACPI bridge/device-ID spoof |

The MC__ renames, HPET override and IRQ 2/8/0 patches are firmware-specific. They were carried over from the original working build, not newly proved necessary on every revision. Preserve related table/patch pairs. GPU cosmetic properties target `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)`; confirm your own path if the graphics layout differs.

## CPU profile provenance

`CPUFriendDataProvider.kext/Contents/Info.plist` is version 1.0.0, 21,110 bytes. Its SHA-256 is:

```
4b8788ecab7671170d9e33854dbfd4ea733b9c056711799fd11637703f2b7ac8
```

It matched the profile in the [developer-linked RKL provider archive](https://github.com/dortania/bugtracker/files/7691378/487281_CPUFriendDataProvider-RKL.zip) discussed in [issue 190](https://github.com/dortania/bugtracker/issues/190). It is codeless. This is provenance, not a claim of custom voltage/frequency calibration for this motherboard. CPUFriend is not a replacement for BIOS power limits, cooling or stability testing.

## Important retained settings

- `DummyPowerManagement = false`, `AppleXcpmForceBoost = false`, `AppleXcpmExtraMsrs = false`.
- `AppleXcpmCfgLock = true`, `AppleCpuPmCfgLock = false`.
- `ProvideCurrentCpuInfo = true`; CpuTopologyRebuild 2.0.2 enabled with no topology boot flag.
- `ForceAquantiaEthernet = true`, `DisableIoMapper = false`; original Aquantia setup used VT-d.
- `XhciPortLimit = false`; USBToolBox and UTBMap work together.
- `ResizeAppleGpuBars = 0`, `UEFI/Quirks/ResizeGpuBars = -1`.
- Memory-map quirks are retained from the working EFI. They are not a minimal set proved for all firmware versions.
- `SecureBootModel = Default`, `DmgLoading = Signed`, `Vault = Optional`; firmware Secure Boot is a separate mechanism.
- `csr-active-config` is zero; no AMFI-bypass boot argument is present. This does not certify the security state of any installed OS.
- `ScanPolicy = 2621699` is the original daily-use filter. Set to `0` on installer media as the README explains.
- Graphical picker, three-second timeout, auxiliary tools hidden until requested; audio assistance disabled.
- OpenShell, ControlMsrE2 and MmapDump remain as optional diagnostic tools. Do not use them to modify firmware variables without understanding the consequences.

## Publication changes

1. Replaced Generic serial/MLB with `REPLACE_ME`, UUID with an all-zero placeholder and ROM with six zero bytes.
2. Cleared corresponding identifiers and asset tags in the alternate PlatformInfo dictionaries.
3. Omitted every file except the active allowlisted EFI payload; no old configs, logs, machine dumps or unused CpuTscSync.
4. Removed MemTest86 and CFGLock tools and their configuration entries.
5. Omitted HfsPlus, graphical Resources and USBToolBox pending user download from upstream. Their entries/settings remain intact.
6. Expanded the three CPU-kext comments; no power, topology, GPU or Booter settings were retuned for publication.

No claim is made that the sanitized template alone is boot-ready: add prerequisites and personalize it first.
