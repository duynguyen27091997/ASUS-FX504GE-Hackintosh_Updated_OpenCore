Tested version: 0.6.4

## Specs:
| Component | Name |
|:--- |:---:|
| Motherboard:  | FX504GE **HM370** |
| CPU: | Intel i7-8750H |
| RAM: | 16GB **SK Hyinix** HMA82GS6CJR8N-VK 2666Mhz |
| iGPU: | Intel UHD 630 (Mobile) |
| dGPU: | NVIDIA GeForce GTX 1050 Ti (DISABLED) |
| Wifi/BT: | Intel(R) Wireless-AC 9560 160MHz (Type CNVi) |
| Audio: | Realtek ALC255 |
| Ethernet: | Realtek RTL8111 |
| Trackpad: | ELAN1200 Precision TouchPad (HID Type) |
| Keyboard: | Standard PS/2 Keyboard |

[Amazon Page](https://www.amazon.es/dp/B07D4W2CY6/ref=cm_sw_em_r_mt_dp_gUF8FbYQW48NV) 1.199€ *purchased 18/09/2018*

### Working
- [x] **Tested with macOS Mojave 10.14.4, Catalina 10.15.6 and Big Sur 11.1**
- [x] **Wifi/Bluetooth:** (Thanks to itlwm.kext)
- [x] **Audio:** Realtek ALC255 (Thanks to AppleALC.kext with layout-id=3 setted in Device Properties)
- [x] **USB:** All internal and external ports (Thanks to SSDT-EC-USBX-LAPTOP.aml)
- [x] **Ethernet:** Realtek RTL8111 (Thanks to RealtekRTL8111.kext
- [x] **Trackpad:** (Working thanks to VoodooI2C.kext, VoodooI2CHID.kext and SSDT-XOSI.aml)
- [x] **Shutdown:**
- [x] **Restart:**
- [x] **Sleep/Wake:** (ON PROCESS)

### Not working
- dGPU (Any support in Mojave and up)
- HDMI, i can't find a way to make it work...



# INSTALLATION GUIDE

## Creating Booteable USB

### From macOS:
[*Installation Guide Apple Page*](https://support.apple.com/en-us/HT201372)

**Download installers:** [Big Sur](https://itunes.apple.com/us/app/macos-big-sur/id1526878132) - [Catalina](https://itunes.apple.com/us/app/macos-catalina/id1466841314) - [Mojave](https://itunes.apple.com/us/app/macos-mojave/id1398502828) - [High Sierra](https://itunes.apple.com/us/app/macos-high-sierra/id1246284741)

1. Connect a <=16 GB pendrive.
2. Open *Disk Utility* and Erase the USB with the name: *MyVolume*.
3. Open *Terminal* and use the proper commands for your macOS installer:
- Big Sur: `sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
- Catalina: `sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
- Mojave: `sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
- High Sierra: `sudo /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`


### From Windows:

[**Dortania's Guide**](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/winblows-install.html)




# OPENCORE Config.plist

## ACPI
### Add
1. `SSDT-EC-USBX.aml` (Fix the embedded controller and USB power)
2. `SSDT-PLUG.aml` (Allows for native CPU power management)
3. `SSDT-PNLF-CFL.aml` (Backlight support for Coffee Lake machines)
4. `SSDT-PMC` (Enable Native NVRAM for HM370 MotherBoard)
5. `SSDT-XOSI.aml` (This is for Trackpad ELAN HID)
5. `SSDT-dGPU-Off.aml` (PowerOff GTX 1050Ti)

### Delete
Not needed

### Patch
1. `Rename _OSI to XOSI`

### Quirks
Not needed in this build

## Booter
### Quirks
Enabled:
1. `AvoidRuntimeDefrag`
2. `EnableSafeModeSlide`
3. `ProvideCustomSlide`
4. `RebuildAppleMemoryMap`
5. `SetupVirtualMap`
6. `SyncRuntimePermissions`


## DeviceProperties
### Add
| PciRoot(0x0)/Pci(0x2,0x0) | Dictionary | Keys / Values |
|:--- |:---:|:--- |
| AAPL,ig-platform-id  | DATA | 00009B3E |
| device-id | DATA | 9B3E0000 |
| AAPL,slot-name | String | Internal@0,2,0 |
| enable-hdmi20 | DATA | 01000000 |
| framebuffer-unifiedmem | DATA | 00000080 |
| framebuffer-con1-busid | DATA | 01000000 |
| framebuffer-con1-enable | DATA | 01000000 |
| framebuffer-con1-flags | DATA | 87010000 |
| framebuffer-con1-has-lspcon | DATA | 01000000 |
| framebuffer-con1-index | DATA | 01000000 |
| framebuffer-con1-pipe | DATA | 12000000 |
| framebuffer-con1-preferred-lspcon-mode | DATA | 01000000 |
| framebuffer-unifiedmem | DATA | 00000080 |
| framebuffer-con1-type | DATA | 00080000 |
| model | String | Intel UHD 630 |

| PciRoot(0x0)/Pci(0x1f,0x3) | Dictionary | Keys / Values |
|:--- |:---:|:--- |
| layout-id  | DATA | 01000000 |

**What does each thing:**
- `AAPL,ig-platform-id` (iGPU Real id)
- `device-id` (Fake id)
- `AAPL,slot-name` (Internal iGPU Indentifier)
- `enable-hdmi20` (Enable 4K monitors and HDR content)
- `framebuffer-unifiedmem` (Increase VRAM from 1536 MB to 2048 MB)
- `framebuffer-patch-enable` (Enable framebuffer patches)
- `model` (Name Showed in *About This Mac*)
- `layout-id` (Sets the audio port to 3)

### Delete
Not needed

## Kernel
### Add
**ORDER MATTER!** Think about which kexts should load before which.
1. [Lilu](https://github.com/acidanthera/lilu/releases) (Essential)
2. [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) (Second always)
   - SMCProcessor
   - SMCSuperIO
   - SMCLightSensor (maybe not)
   - SMCBatteryManager
3. [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases) (Graphics)
4. [AppleALC](https://github.com/acidanthera/AppleALC/releases) (Audio)
5. [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C/releases) (Trackpad Support)
   - VoodooI2CHID
6. [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2/releases) (PS/2 keyboard)
7. [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases) (LAN internet)

### Quirks
1. `AppleCpuPmCfgLock` enabled (maybe not)
2. `AppleXcpmCfgLock` enabled
3. `DisableIoMapper` **enabled** (Disable VT-d)
4. `ThirdPartyDrives` **enabled** (TRIM for SSD)
5. `XhciPortLimit` **disabled** (no need)

## Misc
### Boot
1. `ConsoleBehaviourOs` as `ForceGraphics`
2. `ConsoleBehaviourUi` as `ForceText`
3. `HibernateMode` as `None`
4. `Resolution` as `1920x1080` (Or your actual screen size)
5. `ShowPicker` and `UsePicker` enabled

### Security
1. `AllowNvramReset` enabled (If you want to)
2. `ExposeSensitiveData` as `3` or `4`

## NVRAM
### Add
1. `7C436110-AB2A-4BBB-A880-FE41995C9F82`
   1. `boot-args` containing:
      1. `darkwake=0` (Sleep/wake)
      2. `-lilubetaall -vsmcbeta -alcbeta -wegbeta` (If you are on a newer beta)
      3. `-v debug=0x100 keepsyms=1` (Debugging, optional)
      4. `-no_compat_check` (In case OS doesn't support your machine)
      5. `-igfxmlr enable-dpcd-max-link-rate-fix` (Read WhateverGreen README)
   2. `LegacyEnable` enabled (Emulating NVRAM)

## PlatformInfo
1. `Automatic` **enabled**
2. `Update*` options **enabled**

### Generic
This is where you put SMBIOS data, make sure you provide the following:
1. `MLB`
2. `ROM` (MAC address)
3. `SpoofVendor` **enabled**
4. `SystemProductName` as `MacBookPro15,2` or similar
5. `SystemSerialNumber`
6. `SystemUUID`
**These values are masked from the provided config file, make sure you enter your own before testing!**

## UEFI
1. `ConnectDrivers` **enabled**

### Drivers (must-have)
1. `UsbKbDxe.efi`
2. `VboxHfs.efi`
3. `ApfsDriverLoader.efi`
4. `FwRuntimeServices.efi` (UEFI + AptioMemoryFix)
5. `VirtualSMC.efi`
6. `NvmExpressDxe.efi` (NVMe SSD)
7. `XhciDxe.efi` (Peripherals)

### Input
1. `KeySupport` disabled (already use `UsbKbDxe.efi`)

### Protocols
1. `ConsoleControl` enabled

### Quirks
1. `ClearScreenOnModeSwitch` enabled
2. `ProvideConsoleGop` enabled
3. `RequestBootVarRouting` enabled
4. `SanitiseClearScreen` enabled
