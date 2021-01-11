Tested version: 0.6.4

OpenCore 0.6.5 its on work...

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
- [x] **HDMI:** Works almost perfect. 
- [x] **Shutdown:** Yes
- [x] **Restart:** Yes
- [x] **Sleep/Wake:** Yes

### Not working
- dGPU (Any support in Mojave and up)



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

![Terminal](Images/Guide/BootableUSB.png)

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
**Enabled:**
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
1. [Lilu](https://github.com/acidanthera/lilu/releases) (First)
2. [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) (Second)
   - SMCProcessor
   - SMCSuperIO
   - SMCLightSensor (maybe not)
   - SMCBatteryManager
3. [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases) (Graphics)
4. [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2/releases) (PS/2 keyboard)
5. [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases) (LAN internet)
6. [AppleALC](https://github.com/acidanthera/AppleALC/releases) (Audio)
7. [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C/releases) (Trackpad Support)
   - VoodooI2CHID

### Block
Ignore

### Emulate
Ignore

### Force
We don't need to force any kext to load, so ignore

### Patch
Ignore

### Quirks
**Enabled:**
1. `AppleXcpmCfgLock` (We don't have options to unlock de CFG-Lock on the BIOS)
2. `DisableIoMapper` (If you have VT-d enabled on the BIOS (Prefered))
3. `DisableLinkeditJettison`
4. `PanicNoKextDump`
5. `PowerTimeoutKernelPanic`
6. `XhciPortLimit` (Needed for USBs type XHCI)

### Scheme
Ignore, leave that by default.

## Misc
### Boot
Leave Default

### Debug
Leave Default if you dont want debug information.

### Entries
Ignore

### Security
**Enabled:**
1. `AllowNvramReset` (For RESET the NVRAM on picker selector)
2. `AllowSetDefault` (Default disk for multiboot)
3. `BlacklistAppleUpdate` (Stop reciving some sensitive updates)
- `ScanPolicy` 0
- `SecureBootModel` Default
- `Vault` Optional

### Tools
Remove from `EFI/OC/Tools` everything


## NVRAM
### Add
| 7C436110-AB2A-4BBB-A880-FE41995C9F82 | Dictionary | Keys / Values |
|:--- |:---:|:--- |
| boot-args  | String | -v keepsyms=1 debug=0x100 alcid=3 -wegnoegpu -igfxnohdmi agdpmod=vit9696 |
| run-efi-updater | String | No |
| csr-active-config | DATA | 00000000 |
| prev-lang:kbd | String | en-US:0 |   

**What does each thing:**
- `boot-args` (Boot Arguments)
   - `-v keepsyms=1 debug=0x100` (Debuging)
   - `alcid=3` (Sets de audio to port 3)
   - `-wegnoegpu` (Disable dGPU GTX 1050 Ti)
   - `-igfxnohdmi` ()
   - `agdpmod=vit9696` (Disable board-id checker **ESSENTIAL FOR HDMI OUTPUT**)
- `run-efi-updater` (Disable macOS updates to EFI)
- `csr-active-config` (SIP configuration (Enabled), For more: [Disabling SIP](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html#disabling-sip))
- `prev-lang:kbd` (Sets custom language, For more: [AppleKeyboardLayouts.txt(opens new window)](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)

### Delete
Ignore

### LegacySchema
Ignore, we have native NVRAM

### WriteFlash `Enable`


## PlatformInfo
### Automatic `enabled`

### Generic
Download [GenSMBIOS (opens new window)](https://github.com/corpnewt/GenSMBIOS), and open the *GenSMBIOS.command* with *Right-Click > Open*, follow the intructions on the Terminal Window.

| Generic | Dictionary | Keys / Values |
|:--- |:---:|:--- |
| AdviseWindows  | Boolean | False |
| SystemMemoryStatus | String | Auto |
| MLB | String | *Your own with GenSMBIOS* |
| ProcessorType | Number | 0 |
| ROM | DATA | *Your own with GenSMBIOS* |
| SpoofVendor | Boolean | True |
| SystemProductName | String | MacBookPro15,3 |
| SystemSerialNumber | String | *Your own with GenSMBIOS* |
| SystemUUID | String | *Your own with GenSMBIOS* |

**These values are masked from the provided config file, make sure you enter your own before testing!**

### UpdateDataHub `Boolean` `Enable`
### UpdateNVRAM `Boolean` `Enable`
### UpdateSMBIOS `Boolean` `Enable`
### UpdateSMBIOSMode `String` `Create`


## UEFI
### ConnectDrivers `Boolean` `enabled`

### APFS
Leave everything default

### Audio
For now leave everything default

### Drivers (must-have)
1. `OpenRuntime.efi`
2. `HFsPlus.efi`

### Input
Ignore

### Output
Ignore

### ProtocolsOverride
Ignore

### Quirks
**Enabled:**
1. `DeduplicateBootOrder`
2. `ReleaseUsbOwnership` (Mainly for USB fixes)
3. `RequestBootVarRouting` (Redirects some Variables for macOS)


#BenchMarks:
![Cinebench R23]()

###GeekBench 5:
![GeekBench5_Score]()
https://browser.geekbench.com/v5/cpu/5707123
