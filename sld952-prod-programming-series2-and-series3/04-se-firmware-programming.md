# SE Firmware Programming

## Overview

Consistent with best practices for Internet of Things (IoT) security, the SE Firmware provided with Series 2 and Series 3 devices supports secure firmware updates. Silicon Labs will periodically release new versions of the SE Firmware to fix bugs and patch vulnerabilities, which may require updates to devices on the manufacturing line or to devices already in the field.

Silicon Labs operates under a **Security as a Shared Responsibility Model**. This model provides flexibility to system integrators to manage SE Firmware security updates on their own timetable based on their product's use case, risk assessment, agility of their manufacturing flow, and the agility of their field firmware deployment flow.

Series 2 and Series 3 devices are rarely shipped with the latest SE Firmware installed, meaning system integrators must add SE Firmware programming to their production programming flow.

In all cases, Silicon Labs recommends that system integrators:

Subscribe to security notifications by managing their notification settings in the Silicon Labs Support Portal. This is the easiest method to be notified of SE Firmware updates and discovered vulnerabilities.

Instructions for subscribing to notifications can be found here [https://community.silabs.com/s/article/get-notified-when-a-document-changes](https://community.silabs.com/s/article/get-notified-when-a-document-changes)

- Ensure they are installing the latest SE Firmware release in their manufacturing line.

- Be prepared to deploy [security-related field updates](./11-field-upgrade-the-se-firmware.md) to devices in the field.

## How to Check the SE Firmware Version on a Device

The SE Firmware version of the device can be found in two ways.

- Simplicity Studio

- Simplicity Commander

This application note mainly uses Simplicity Commander.

### Check the SE Firmware Version Using Simplicity Commander

To check the SE Firmware version on the device, issue the Simplicity Commander `security status` command.

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.14
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Disabled
Device erase	    : Enabled
Secure debug unlock : Disabled
Tamper status	    : OK
Secure boot	        : Disabled
Boot status	        : 0x20 - OK
DONE
```

In this example, the SE Firmware version on the EFR32MG21A is 1.2.14.

## How to Find the Latest SE Firmware

Silicon Labs strongly recommends installing the latest SE firmware on Series 2 and Series 3 devices to support the required security features. The latest SE firmware image (`.seu` or `.seuv2` and `.hex`) and release notes can be found in the Windows folder below.

For GSDK v3.2 and lower:

`C:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\<GSDK VERSION>\util\se_release\public`

For GSDK v4.0 and higher:

`C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\gecko_sdk\util\se_release\public`

For Simplicity SDK

`C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\simplicity_sdk\util\se_release\public`

## Serial Wire Debug (SWD)

The SE Firmware cannot be directly programmed to the SE using the SWD interface. Instead, an image containing the loader application and SE Firmware is flashed onto the host MCU. The SE Firmware is encrypted, versioned, and signed.

![SWD SE Firmware Upgrade Block Diagram](./resources/sld952-swd-se-firmware-upgrade-diagram.png)

Using the SWD interface, the user flashes the loader application onto the host. The host then runs the loader application, which checks the signature and version of the SE Firmware. If the signature check passes and the upgrade's version number is higher than the device's SE Firmware version, the firmware is applied to the SE.

The upgrade will not be applied if the signature check fails or if the upgrade's version number is less than or equal to the device's SE Firmware version. Trying to apply a lower SE Firmware version to the device does no harm, but the upgrade will be ignored. This also means the device's SE Firmware cannot be downgraded.

After the SE Firmware has been upgraded, the loader application can be overwritten with the application firmware via the SWD interface.

As detailed in diagrams in the [Overview](./02-overview.md), the steps to upgrade the SE Firmware are:

1. Connect Hardware: Connect the device's SWD interface with the WSTK and ensure proper connections.

2. Check Version: Check the SE Firmware version already on the device.

3. Flash SE Firmware: Flash the loader application onto the host processor.

4. Run: Allow the loader application to run and install the SE Firmware.

5. Re-Check Version: Ensure the update succeeded.

Each of these steps is described in more detail in the next sections.

### Connect Hardware

After connecting the device's SWD interface to the WSTK, try to read the device information using Simplicity Commander, to verify that proper connections were established to the device.

```sh
commander device info --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Part Number	   : EFR32MG21A010F1024IM32
Die Revision   : A1
Production Ver : 2
Flash Size	   : 1024 kB
SRAM Size	   : 96 kB
Unique ID      : 14b457fffe045a8e
DONE
```

### Check Version

To check the SE Firmware version on the device, issue the Simplicity Commander `security status` command.

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.13
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Disabled
Device erase	    : Enabled
Secure debug unlock : Disabled
Tamper status	    : OK
Secure boot	        : Disabled
Boot status	        : 0x20 - OK
DONE
```

**Notes**:

- The `Tamper status` item is device dependent.

- Updating SE firmware on Series 3 takes significantly longer than Series 2. If the SE firmware on your device is already up to date, skip the next step for efficiency.

### Flash SE Firmware

To flash the SE Firmware upgrade application, run:

```sh
commander flash --masserase s2c1_se_fw_upgrade_app_1v2p14.hex --device EFR32MG21A010F1024 --serialno 440048205
```

where `s2c1_se_fw_upgrade_app_1v2p14.hex` is replaced with the name of the SE Firmware upgrade application file.

```sh
Parsing file s2c1_se_fw_upgrade_app_1v2p14.hex...
Erasing chip...
Flash was erased successfully
Writing 57344 bytes starting at address 0x00000000
Comparing range 0x00000000 - 0x0000DFFF (56 KB)
Programming range 0x00000000 - 0x00001FFF (8 KB)
Programming range 0x00002000 - 0x00003FFF (8 KB)
Programming range 0x00004000 - 0x00005FFF (8 KB)
Programming range 0x00006000 - 0x00007FFF (8 KB)
Programming range 0x00008000 - 0x00009FFF (8 KB)
Programming range 0x0000A000 - 0x0000BFFF (8 KB)
Programming range 0x0000C000 - 0x0000DFFF (8 KB)
DONE
```

### Run

Allow the SE Firmware upgrade application to run for at least two seconds on Series 2 devices and for at least 10 seconds on Series 3 devices. After this time has elapsed, the SE Firmware should have been upgraded.

### Re-Check Version

Run the `security status` command again to check the upgraded SE Firmware version.

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.14
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Disabled
Device erase	    : Enabled
Secure debug unlock : Disabled
Tamper status	    : OK
Secure boot	        : Disabled
Boot status	        : 0x20 - OK
DONE
```

The version is now upgraded to 1.2.14.
