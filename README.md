# Kryptonite
![Script Version](https://img.shields.io/github/release/mayankk2308/kryptonite.svg?style=for-the-badge)
![macOS Support](https://img.shields.io/badge/macOS-10.13.4+-orange.svg?style=for-the-badge) ![Github All Releases](https://img.shields.io/github/downloads/mayankk2308/kryptonite/total.svg?style=for-the-badge) [![paypal](https://www.paypalobjects.com/digitalassets/c/website/marketing/apac/C2/logos-buttons/optimize/34_Yellow_PayPal_Pill_Button.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=mayankk2308@icloud.com&lc=US&item_name=Development%20of%20Kryptonite&no_note=0&currency_code=USD&bn=PP-DonationsBF:btn_donate_SM.gif:NonHostedGuest)

Kryptonite enables external GPUs on Macs using Thunderbolt 1 and 2 without compromising on Mac security features such as **System Integrity Protection**, **FileVault**, and **Authenticated-Root**.

Unlike [PurgeWrangler](https://github.com/mayankk2308/purge-wrangler), which requires these security features disabled and modifies kernel extensions on the root volume, **Kryptonite** injects patches via EFI and performs them in memory, when the offending kernel extensions load. This project supercedes the PurgeWrangler system.

## System
**Kryptonite** leverages [OpenCore](https://github.com/acidanthera/OpenCorePkg) with a heavily simplified configuration for native Macs to inject kernel/kext patches into macOS during boots. The patches themselves are implemented in a **kernel extension** named **Kryptonite** that leverages [Lilu](https://github.com/acidanthera/Lilu) which can patch kexts and processes in memory.

You can control **Kryptonite**'s behavior using boot-args specified in the OpenCore **config.plist**. The kernel extension supports the following boot arguments:

| Boot Arg | Description |
| :----------------: | :------------ |
| `-krydisable` | Disables **Kryptonite** on boot. |
| `-krydbg` | Enables debugging for **Kryptonite**. Must be used alongside `-liludbg`. |
| `-krybeta` | Enables **Kryptonite** on beta/untested versions of macOS. Must be used with `-lilubeta`. |
| `krygpu=` | Provide GPU vendor to patch for. Must be `AMD` or `NVDA`. |
| `krytbtv=` | Provide Thunderbolt NHI version. Required for **macOS ≤ 10.15**. Must be `1` or `2`. |

## Features
With **Kryptonite**, you get the following benefits over **PurgeWrangler**:
1. You only have to set up **Kryptonite** once, and it will continue to work through Apple software updates.
1. You can use your mac with all security features enabled - excluding **T2 chip** if used on those Macs.
1. Because all patches are performed in memory, your system is untouched when booted without the **Kryptonite/OpenCore** disk.
1. Automatic patching for all installations of macOS booted via the **Kryptonite/OpenCore** disk.
1. Free benefits from **OpenCore** such as the ability to enable iGPUs and inject DSDT overrides to address **error 12** in Bootcamp.

Additional benefits on **macOS Big Sur** and later:
1. Boot volume seal is not tampered with - meaning a truly native experience without compromises.
1. **FileVault** can now be used without compromise on old macs along with eGFX support.
1. Smaller **delta software updates** are supported as system is clean and security features are enabled.

## Installation
The steps are as follows:
1. If you are using this on a **T2 mac**, please disable **T2 security**.
1. If you have used **PurgeWrangler** before, it must be uninstalled:

   ```shell
   purge-wrangler -u
   ```
   You should also enable SIP and make sure your system can successfully boot. On macOS Big Sur or later, I recommend reinstalling macOS to re-seal your boot volume.
1. Go to **Disk Utility** and [create](https://osxdaily.com/2020/06/29/how-create-new-partition-mac/) a new **MS-DOS (FAT32)** partition (internal or external) **if you do not already have a bootloader disk**.
1. Download **Kryptonite-RELEASE** from the [Releases](https://github.com/mayankk2308/kryptonite/releases). If you want to emit logs for testing, download the **DEBUG** version.
1. Unzip and copy the `EFI` folder to your created disk. Then edit the **config.plist** file and add the required **boot-args** you need. Check the [System](https://github.com/mayankk2308/kryptonite#system) section for more information.

### Key Notes
1. Installer is disabled as there are some issues and work is being done on it.
1. The installer can configure existing bootloaders as well. Just let the installer know when asked and follow the prompts.
1. APFS volumes are currently not shown in the installer for formatting or resizing. This feature may be added later.
1. The kernel extensions are automatically disabled on untested/beta versions of macOS. To enable them, follow [these instructions](https://github.com/mayankk2308/kryptonite#beta-versions-of-macos).

## Uninstallation
Uninstalling **Kryptonite** is very straightforward:
1. On boot, press and hold **OPTION** key.
1. Select your macOS boot volume instead of **Kryptonite**.
1. Press **CTRL + ENTER** to set it as default boot volume and boot normally.
1. Delete the **Kryptonite** partition/disk via **Disk Utility**.
1. [Reset NVRAM](https://support.apple.com/en-us/HT204063) only if SIP is currently enabled for your system. Otherwise, delete `boot-args` as follows:
   ```shell
   sudo nvram -d boot-args
   ```

At **step 4**, you can alternatively keep the disk and use it on-demand by selecting it manually during boot. If you want to use **OpenCore** but remove **Kryptonite**, you can simply disable the kernel extension in your **config.plist**.

### Debugging
If you have issues, please share your logs. To do this, first ensure you create the bootloader again and use **DEBUG** resources using the installer. If you have a pre-configured OpenCore setup (such as with OpenCore Legacy Patcher), then enable debug mode as follows: https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/debug.html

Additionally, make sure to add the following boot-args for kext debugging:
```shell
-liludbgall -krydbg liludump=60
```

You can add the boot-args to the OpenCore **config.plist** boot-args section alongside your other arguments. When you boot the debug configuration for OpenCore, you will find the logs generated next to the `EFI` folder on your bootloader disk. For the kext logs from **Lilu**, check `/var/log/` folder for logs. For debugging, we would need both these files.

### Things Missing in the Installer
- Downloading NVIDIA Web Drivers for using a Maxwell or Pascal NVIDIA GPU on **macOS High Sierra**.
- Detecting and resizing APFS containers and create usable disks for **Kryptonite** during installation.
- Disabling discrete GPUs on Macs that need it to allow for displays connected to external GPUs to function.

### Configuration
To manually edit configurations, use [ProperTree](https://github.com/corpnewt/ProperTree#on-nix-systems) to open the **config.plist** file on your bootloader. This file is located on your bootloader disk in the `EFI/OC/` directory. If you are comfortable doing so, you can edit the file in **TextEdit** - just be careful with the format and XML tags. This section describes some common configuration changes you may want to make:

#### Beta Versions of macOS
By default, **Kryptonite** will be disabled on **beta** or **untested** versions of macOS. To enable this, you need to update the **boot-args** in your **config.plist**. Specifically, you need to **add** the following arguments:
```shell
-lilubeta -krybeta
```
Add these after the already-present **boot-args**.

#### Disabling NVIDIA Discrete GPU
If you are using an AMD eGPU with a Mac that has a discrete NVIDIA GPU, display outputs may not work on the eGPU. To fix this, you can disable the discrete GPU as follows:
1. Configure the bootloader to power off the NVIDIA GPU. Follow instructions [here](https://dortania.github.io/OpenCore-Install-Guide/extras/spoof.html). Use the **DeviceProperties** approach on that page.
2. Switch mux to iGPU:
  ```shell
  sudo nvram FA4CE28D-B62F-4C99-9CC3-6815686E30F9:gpu-power-prefs=%01%00%00%00
  ```
  Sometimes this may not work. A good indicator that it worked is that when you boot, the boot chime is heard but there is a small delay before the display backlight comes on. If it does not work, there is no other option but to retry.
  
Once configured, you will most likely not require any changes with respect to eGPU support. If there is a newer release of the **Kryptonite** packages and you want to get them, simply start the installation process (refer to section above) and when asked if you are already using OpenCore, answer no. Select your existing **Kryptonite** disk and format it. After that just follow the instructions in the script and you will have the latest packages.

## License
This project is licensed under [GPL-3.0](./LICENSE.md), while its underlying dependencies such as [OpenCore](https://github.com/acidanthera/OpenCorePkg) and [Lilu](https://github.com/acidanthera/Lilu) are licensed under BSD-3-Clause license.

## Credits

### Software and Frameworks
- [Apple](https://www.apple.com) for macOS.
- [acidanthera](https://github.com/acidanthera/) and it's contributors for [OpenCore](https://github.com/acidanthera/OpenCorePkg).
- [acidanthera](https://github.com/acidanthera/) and it's contributors for [Lilu](https://github.com/acidanthera/Lilu).

### Patches
- [@mayankk2308/@mac_editor](https://egpu.io/forums/profile/mac_editor/) for:
  - Thunderbolt patches for native eGFX support on **macOS 10.13.4-10.15.1**.
  - Updated Thunderbolt patches for native eGFX support on **macOS 10.15.1+**.
  - Bypass for Thunderbolt driver compatibility (**IOPCITunnelCompatible**) checks on **macOS 10.13.4+**.
- [@goalque](https://egpu.io/forums/profile/goalque/) for support for NVIDIA eGFX on **macOS 10.13.4+**.
- [@rgov](https://github.com/rgov) for Ti82 Thunderbolt patches - adapted for [Lilu](https://github.com/acidanthera/Lilu) by [@mac_editor](https://egpu.io/forums/profile/mac_editor/).
