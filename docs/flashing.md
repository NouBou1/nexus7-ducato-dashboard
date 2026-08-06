# Flashing walkthrough — Nexus 7 (2012, `grouper`)

The full procedure used to get AOSP 7.1.2 onto the tablet. Written for the 2012 Wi-Fi model (`grouper`); the 2013 model is `flo` and needs different images.

> ⚠️ **Unlocking the bootloader wipes the device.** Back up anything you care about first.

## Contents

1. [Prerequisites](#1-prerequisites)
2. [Developer options & USB debugging](#2-developer-options--usb-debugging)
3. [Unlocking the bootloader](#3-unlocking-the-bootloader)
4. [Installing TWRP recovery](#4-installing-twrp-recovery)
5. [Wiping](#5-wiping)
6. [Flashing the ROM and GApps](#6-flashing-the-rom-and-gapps)
7. [Dashboard configuration](#7-dashboard-configuration)
8. [Command reference & troubleshooting](#8-command-reference--troubleshooting)

---

## 1. Prerequisites

**On the computer**

- Android SDK Platform Tools (`adb` and `fastboot`)
- Google USB Driver, on Windows
- A Micro-USB **data** cable — a charge-only cable will not enumerate the device

**Files to download**

- TWRP recovery `.img` for `grouper`
- The ROM `.zip` — here, AndDiSa's AOSP 7.1.2 build
- OpenGApps or MindTheGapps `.zip`, matching the Android version and architecture (`arm`, 7.1). The `pico` variant is the right choice on this hardware.
- Magisk `.zip`, if you want root

## 2. Developer options & USB debugging

1. **Settings → About tablet**
2. Tap **Build number** seven times until developer mode is enabled
3. Back in Settings → **Developer options**, enable **USB debugging** (and *OEM unlocking* if the option exists)

## 3. Unlocking the bootloader

Connect the tablet and confirm the RSA prompt on its screen, then:

```bash
adb devices              # tablet should be listed
adb reboot bootloader    # reboot into fastboot mode
fastboot oem unlock      # newer builds: fastboot flashing unlock
```

Confirm **YES** on the tablet with the volume keys and the power button. The device factory-resets itself.

*Alternative way into fastboot, from powered off: hold **volume down** + **power**.*

## 4. Installing TWRP recovery

```bash
fastboot devices                              # verify the connection
fastboot flash recovery twrp-3.x.x-grouper.img
```

Then use the volume keys to select **Recovery Mode** and confirm with power. Do not let the device boot into the system first — some stock builds restore the factory recovery on boot.

## 5. Wiping

In TWRP: **Wipe → Advanced Wipe**, select

- Dalvik / ART Cache
- System
- Cache
- Data

and swipe to confirm.

> Do **not** wipe *Internal Storage* if you have already copied the ROM zip onto the device.

## 6. Flashing the ROM and GApps

### Via ADB sideload (what I used)

In TWRP: **Advanced → ADB Sideload**, swipe to start, then from the computer:

```bash
adb sideload aosp-7.1.2-grouper-AndDiSa.zip
```

Restart sideload in TWRP for each additional package:

```bash
adb sideload open_gapps-arm-7.1-pico.zip
adb sideload Magisk.zip          # optional
```

### Via internal storage or USB-OTG

1. Copy the zips across with `adb push` or an OTG stick
2. **Install** → select the ROM zip
3. **Add more Zips** to queue GApps and Magisk
4. Swipe to confirm, then **Reboot System**

The first boot takes several minutes.

## 7. Dashboard configuration

### Boot when power is applied

Run from the bootloader:

```bash
fastboot oem off-mode-charge 0
```

Powered-off devices now boot when power appears instead of showing a charging animation. This is what makes the tablet start with the ignition.

*This flag is device- and bootloader-specific — it works on `grouper`, but do not assume it carries over to other hardware.*

### Screen timeout

**Developer options → Stay awake** keeps the display on while charging.

### Automation (Tasker / MacroDroid)

- Power connected → screen on, Bluetooth on, launch the car launcher
- Power disconnected → screen off, deep sleep / airplane mode

### Launcher

A car launcher gives you large touch targets and a layout usable while driving. Agama Car Launcher is what is running here; Car Web Guru is a comparable alternative.

## 8. Command reference & troubleshooting

| Command | Description |
| :--- | :--- |
| `adb devices` | List connected devices |
| `adb reboot bootloader` | Reboot into fastboot |
| `adb reboot recovery` | Reboot into TWRP |
| `adb push file.zip /sdcard/` | Copy a file to internal storage |
| `fastboot devices` | Verify the fastboot connection |
| `fastboot reboot` | Reboot out of fastboot |

**Device not detected in fastboot (Windows).** Open Device Manager and assign the *Android Bootloader Interface* driver manually. This is the single most common stumbling block.

**Bootloop after flashing.** Back into TWRP, wipe Cache and Dalvik/ART Cache, reboot. If it persists, the ROM and GApps versions probably do not match.

**Very slow charging.** These tablets charge slowly by design, and a thin cable makes it worse. Check the cable's conductor cross-section before blaming the tablet.
