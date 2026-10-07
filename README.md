# ASUS PRIME Z690M-PLUS D4 · i9-12900K · RX 6950 XT

An OpenCore build guide and sanitized EFI template from a real, daily-used Hackintosh.

**Read before copying:** this is a hardware-specific historical configuration, not a universal EFI. You must **add the three upstream components below and generate your own machine identity before booting**. The original computer has been sold; this publication has been statically checked, not boot-tested again after sanitization.

## Hardware and tested baseline

| Component | Original build |
|---|---|
| Motherboard | **ASUS PRIME Z690M-PLUS D4** — model confirmed by the owner |
| CPU | Intel Core i9-12900K, 8 P-cores + 8 E-cores, 24 threads |
| GPU | AMD Radeon RX 6950 XT, 16 GB; device-ID spoof required |
| Memory | 32 GB: 4 × 8 GB Crucial Ballistix DDR4-2400 |
| Storage | WD Black SN850X and ADATA SX8200NP NVMe drives |
| Working network adapter | Aquantia AQC107-B0, observed at 10 GbE |
| Audio used | Focusrite Scarlett Solo USB interface |
| OpenCore | 1.0.7 RELEASE |
| macOS recorded in the original audit | Tahoe 26.6.2, build 25G83 |
| SMBIOS | MacPro7,1 |

The motherboard's built-in Ethernet is **Intel 1 GbE**, not Aquantia. This EFI does not contain IntelMausi, and onboard Ethernet was not established as working. If you do not have the original Aquantia adapter, arrange a supported network connection before installation and identify/configure your own Ethernet controller. See the [ASUS specifications](https://www.asus.com/motherboards-components/motherboards/prime/prime-z690m-plus-d4/techspec/).

The original BIOS version, GPU board-partner model and cooling configuration were not recorded. Do not assume that every BIOS revision or PCIe-slot arrangement behaves identically.

## What worked

| Feature | Status and scope |
|---|---|
| macOS boot and normal desktop use | Worked on the original computer |
| RX 6950 XT acceleration | Metal acceleration and 16 GB VRAM observed with the included spoof |
| CPU power management | X86PlatformPlugin/XCPM observed; CPUFriend profile retained |
| Hybrid CPU topology | CpuTopologyRebuild 2.0.2 enabled in the final supplied EFI |
| Sleep/wake | **Worked according to the owner**; no claim of a new post-publication test |
| USB | Original 15-port map worked for the owner's setup; validate your own sockets and case headers |
| Aquantia 10 GbE | Worked with the original adapter and VT-d arrangement |
| NVMe / TRIM | Both original drives had TRIM enabled |
| USB audio | Focusrite Scarlett Solo was used |
| Onboard audio | **Untested — never used by the owner**; AppleALC is not included |
| Onboard Intel Ethernet, Wi-Fi, Bluetooth, Thunderbolt | Not validated by this publication |
| Intel UHD 770 graphics | Not supported for macOS acceleration; connect displays to the Radeon |
| Hibernation / FileVault / dual boot | Not certified by this guide; sleep is not the same as hibernation |

## 1. Download and prepare the EFI

Download this repository with **Code → Download ZIP**, extract it, and work on a copy. The `EFI` directory contains the sanitized configuration, ACPI tables and redistributable runtime components. [Component versions](components.json) and [credits/licensing](THIRD_PARTY.md) are included.

Add these upstream components yourself; their redistribution terms were not established here. **They remain enabled in config.plist and must not be skipped.**

| Obtain from upstream | Copy into this repository's EFI |
|---|---|
| [OcBinaryData: HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) — use **Download raw file**, not Save Page | `EFI/OC/Drivers/HfsPlus.efi` |
| [OcBinaryData](https://github.com/acidanthera/OcBinaryData) — download repository ZIP, take its entire `Resources` folder | `EFI/OC/Resources/` |
| [USBToolBox kext 1.2.0 release](https://github.com/USBToolBox/kext/releases/tag/1.2.0) — extract `USBToolBox.kext` | `EFI/OC/Kexts/USBToolBox.kext` |

Keep the included `UTBMap.kext`. Do **not** add `UTBDefault.kext` alongside that map. OcBinaryData assets are downloaded separately and are not version-pinned by this repository; test the resulting picker on your USB first.

Use [ProperTree](https://github.com/corpnewt/ProperTree) to edit the plist, and [OpenCore 1.0.7 RELEASE](https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.7) for its matching `Utilities/ocvalidate` tool. Do not mix a newer sample plist, drivers and an older OpenCore binary.

## 2. Generate a unique Mac identity — mandatory

The published values are placeholders, not a usable identity. Do not use someone else's serial number, board serial or UUID.

1. Run [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and generate a **MacPro7,1** identity.
2. In `EFI/OC/config.plist → PlatformInfo → Generic`, replace:

   | Field | Set it to |
   |---|---|
   | `SystemSerialNumber` | Your generated serial, replacing `REPLACE_ME` |
   | `MLB` | Your generated board serial, replacing `REPLACE_ME` |
   | `SystemUUID` | Your generated UUID, replacing the all-zero UUID |
   | `ROM` | A stable, unique six-byte value; commonly your Ethernet MAC without separators, entered as **Data**, not a string |

3. Keep `SystemProductName = MacPro7,1` and `Automatic = true`. The alternate identity fields in DataHub/SMBIOS/PlatformNVRAM are deliberately blank; OpenCore derives them from Generic in this configuration.
4. Keep your personalized copy private, including any backup plists. Do not commit these values to a public fork or attach an unredacted config to an issue.

Follow the [Dortania PlatformInfo and identity guidance](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#platforminfo) for account-service preparation. A generated identity does not guarantee Apple services will work.

## 3. BIOS preparation

These are installation starting points, **not a recovered export of the owner's BIOS settings**. Menu names vary by BIOS revision. Photograph your current settings and keep a recovery path before changing them.

| Setting | Starting point / reason |
|---|---|
| Boot mode / CSM | UEFI boot; disable CSM |
| Firmware Secure Boot | Disable for the unsigned OpenCore boot chain; on ASUS, this may be labelled OS Type → Other OS. This is separate from OpenCore's `SecureBootModel` |
| Fast Boot | Disable while installing/troubleshooting |
| SATA mode / Intel VMD / RAID | AHCI for SATA; avoid VMD/RAID for the target macOS disk. **Changing this can prevent an existing Windows installation from booting** |
| Primary display | PCIe/discrete GPU; use the RX 6950 XT outputs, not motherboard HDMI/DP |
| Above 4G Decoding | Enable |
| Resizable BAR | Keep the EFI's existing `ResizeAppleGpuBars = 0` and `UEFI → Quirks → ResizeGpuBars = -1`; use ReBAR disabled as an initial troubleshooting baseline |
| XHCI Hand-off | Enable if exposed |
| VT-d | Enable for the tested Aquantia path. This EFI keeps `DisableIoMapper = false` |
| CPU cores / Hyper-Threading | Keep all P-cores, E-cores and Hyper-Threading enabled |
| CFG Lock | Prefer disabled if a normal BIOS option exists. The supplied EFI retains `AppleXcpmCfgLock = true` as a workaround; do not modify hidden firmware variables blindly |
| CPU power / voltages | Start at stock, with suitable cooling; do not start by copying an overclock |
| Memory | Start with known-stable settings; enable XMP only after establishing stability |

The owner later reported XMP II plus a setting described as “ASUS AI Overclock 2”. That wording was not independently verified against this board's BIOS. It is a benchmark condition, **not an installation requirement or recommended voltage/power recipe**.

General BIOS background: [Dortania Intel desktop BIOS guidance](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#intel-bios-settings). Alder Lake needs the additional CPUID/topology handling already present here; do not replace this entire configuration with a Comet Lake sample.

## 4. Make the installer and first boot

1. Back up important data. Use a **spare USB drive** for the installer and keep another known-good boot/recovery option. Formatting erases the selected drive; verify its identity before proceeding.
2. Obtain the macOS installer from Apple using the [Dortania installer instructions](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/). On a Mac, follow the full-installer/createinstallmedia route. A recovery installer on Windows/Linux also needs a working network connection.
3. Use a GUID-partitioned USB with a FAT32 EFI system partition. Mount that USB's EFI partition and place the completed `EFI` folder at its root, so the bootloader is `EFI/BOOT/BOOTx64.efi`. Avoid an accidental `EFI/EFI` nesting.
4. **Installer visibility:** the original daily-use EFI has `Misc → Security → ScanPolicy = 2621699`, restricting the picker to certain APFS/NVMe/USB entries. On the **installer USB copy**, set `ScanPolicy = 0` so installer and recovery filesystems are not filtered out. This broadens scanning; after setup, choose your desired policy deliberately.
5. Run OpenCore **1.0.7** `ocvalidate` against your edited config. On macOS, `plutil -lint EFI/OC/config.plist` checks plist syntax but is not a substitute for ocvalidate. Check every enabled ACPI/kext/driver/tool path exists, particularly the three downloaded prerequisites.
6. Select the USB's **UEFI** boot entry in ASUS firmware. In the OpenCore picker, press **Space** to reveal auxiliary/recovery entries if needed. Select the macOS installer.
7. In Disk Utility, choose only your intended destination disk. For a clean installation, use GUID/APFS. This erases that disk; disconnect unrelated drives if you are unsure.
8. Complete installation. Keep booting via the USB across reboots, selecting the macOS Installer entry while installation is in progress, then the installed system.
9. Do not copy the EFI to an internal disk until cold boot, restart, display acceleration, USB, networking and sleep/wake work reliably from USB.

The recorded macOS version is a historical tested baseline, not a promise that every newer installer/update will work. This project does not distribute macOS.

## 5. Validate your hardware

- **GPU:** verify Metal support and correct VRAM. The included SSDT targets the original PCIe/ACPI path. A different slot, GPU, bridge layout or firmware can require a different path.
- **CPU:** check load behaviour, temperatures and sustained performance—not just the cosmetic CPU name. Confirm CPUFriend and its provider are both enabled, with the provider after CPUFriend.
- **USB:** test every intended socket with both USB 2 and USB 3 devices. Test USB-C in both orientations where relevant. Case/front-panel wiring can differ even with the same motherboard.
- **Networking:** verify the adapter you actually use. Do not expect the Aquantia quirk to drive Intel Ethernet.
- **Sleep:** the owner reports it worked; repeat sleep/wake tests with your own USB devices and network adapter.
- **Storage:** check the correct disks are visible and TRIM is enabled for supported SSDs.

### USB mapping matters

The included UTBMap has 15 logical ports: HS01–HS06 as USB 3 Type-A companions, HS07–HS08 as USB 2 Type-A, HS09 internal, and SS01–SS06 as USB 3 Type-A. There is no verified physical socket diagram. It is **not a guarantee that every port on this board is enabled**, particularly Type-C and different front-panel headers.

Use the [USBToolBox mapping tool](https://github.com/USBToolBox/tool) to create your own map when port tests differ. Keep `XhciPortLimit = false`. Do not blindly add more ports or enable an old port-limit workaround.

## Why these CPU and GPU settings are here

- **CPUID emulation:** `55060A00…` / `FFFFFFFF…`, with `ProvideCurrentCpuInfo = true`, supplies the compatibility path used by this Alder Lake build.
- **SSDT-PLUG-ALT:** exposes processors and sets `plugin-type = 1` so X86PlatformPlugin can attach.
- **CpuTopologyRebuild 2.0.2:** enabled in the final EFI. Boot arguments contain only `agdpmod=pikera`; no `ctrsmt=full` flag is supplied. See [upstream topology documentation](https://github.com/b00t0x/CpuTopologyRebuild) before changing its mode. Reported logical/core labels can be misleading; they are not proof that physical cores are disabled.
- **CPUFriend + CPUFriendDataProvider:** the provider is the original developer-reference RKL profile linked from [Dortania issue 190](https://github.com/dortania/bugtracker/issues/190), not a profile newly calibrated for every 12900K. Replacing it previously reduced this machine's measured performance; the restored file is retained byte-for-byte. See [configuration notes](CONFIGURATION.md) for its fingerprint.
- **RX 6950 XT spoof:** the SSDT presents device ID `73A5` as supported `73BF`. `agdpmod=pikera` is retained. Cosmetic DeviceProperties do not replace the SSDT's device-ID spoof.
- **RestrictEvents:** supplies the truthful CPU label and MacPro7,1 PCI-warning handling. Changing the label is not a performance optimization.

### Owner-reported Geekbench 6 results

| Configuration reported at the time | Single-core | Multi-core |
|---|---:|---:|
| [Earlier baseline](https://browser.geekbench.com/v6/cpu/19228296) | 2192 | 11058 |
| XMP II + reported ASUS overclock setting | 2489 | 11836 |
| [Then enabled CpuTopologyRebuild 2.0.2](https://browser.geekbench.com/v6/cpu/19228835) | **2690** | **12484** |

These are individual runs, not a controlled multi-run study or a guarantee for another build. BIOS settings, thermals, background activity and memory configuration all affect results. The middle result was supplied by the owner without a public result link. Do not compare these scores directly with Geekbench 7.

## Install internally, update and recover

Once USB testing succeeds, back up the target disk's existing EFI **outside that EFI partition**, then copy the tested `EFI` folder to the correct internal EFI partition. Keep the working USB intact. Do not overwrite another operating system's boot files without a backup and a deliberate dual-boot plan.

Before updating OpenCore, kexts, BIOS or macOS:

1. Save a private copy of your working, personalized EFI and record versions/settings.
2. Make one category of changes at a time on a USB copy.
3. Update the config schema and OpenCore/driver versions together, then run the matching ocvalidate.
4. Boot-test USB first. A syntactically valid config does not prove a successful boot.

If boot fails, use the known-good USB from the firmware boot menu and restore the previous EFI. For diagnosis, temporarily add `-v` to `boot-args` on the test copy and photograph the error. Do not disable security protections or toggle multiple ACPI/memory quirks at random. If NVRAM reset is genuinely needed, use OpenCore's documented reset procedure; this package does not add a ResetNvramEntry driver, and reset can clear boot selections and other firmware variables.

## Publication notes

Original boot settings are retained except identity redaction, removal of optional CFGLock/MemTest tools and descriptive CPU comments. The inactive CpuTscSync file, old config and private metadata are excluded. HfsPlus, USBToolBox and graphical resources must be fetched upstream as described above.

Checks performed: plist parse, OpenCore 1.0.7 schema validation, component-path inspection with explicit upstream-only prerequisites, identity-value scan and CPUFriend profile hash check. **No post-sale boot test is claimed.**

This is an educational, unsupported community configuration, supplied without warranty. Review Apple's applicable macOS licence terms before use. Do not treat this as Apple or ASUS support, or as an assurance of compatibility with every update.

See [configuration details](CONFIGURATION.md), [third-party notices](THIRD_PARTY.md) and [file checksums](SHA256SUMS.txt).
