# Field Upgrade the SE Firmware

## Secure Boot-Disabled Device

Simplicity Commander or Gecko Bootloader can be used to upgrade the SE Firmware on a Secure Boot-disabled device. The following table lists the scenarios of SE Firmware upgrade on the Secure Boot-disabled device.

| **Secure Debug** | **Device Erase** | **Debug Lock** | **State** | **SE Firmware Upgrade** |
|------------------|------------------|----------------|-----------|-------------------------|
| Disabled | Enabled | Disabled | Unlock | Simplicity Commander or Gecko Bootloader |
| Disabled | Enabled | Enabled | Standard debug lock | Simplicity Commander or Gecko Bootloader |
| Enabled | Disabled | Enabled | Secure debug lock | Simplicity Commander or Gecko Bootloader |
| Disabled | Disabled | Enabled | Permanent debug lock | Gecko Bootloader |

### Simplicity Commander

To flash the SE Firmware upgrade application (e.g., `s2c1_se_fw_upgrade_app_1v2p9.hex`), run:

```sh
commander flash s2c1_se_fw_upgrade_app_1v2p9.hex --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Parsing file s2c1_se_fw_upgrade_app_1v2p9.hex...
Writing 49152 bytes starting at address 0x00000000
Comparing range 0x00000000 - 0x0000BFFF (48 KB)
Programming range 0x00000000 - 0x00001FFF (8 KB)
Programming range 0x00002000 - 0x00003FFF (8 KB)
Programming range 0x00004000 - 0x00005FFF (8 KB)
Programming range 0x00006000 - 0x00007FFF (8 KB)
Programming range 0x00008000 - 0x00009FFF (8 KB)
Programming range 0x0000A000 - 0x0000BFFF (8 KB)
DONE
```

The device should be unlocked before upgrading the SE Firmware if the standard or secure debug lock has been applied. The sections *Standard Debug Lock and Unlock* and *Secure Debug Unlock and Roll Challenge* in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/) describe how to unlock the device.

> **Note**: This method will **OVERWRITE** the bootloader and application firmware on the device. The user should then re-program the [bootloader](./05-bootloader-firmware-programming.md) and [application firmware](./06-application-firmware-programming.md) after the SE Firmware upgrade.

### Gecko Bootloader

Refer to section *Generate a GBL Upgrade Image File* (Secure Engine Upgrade) in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/) for details.

The Gecko Bootloader can still parse the SE GBL upgrade image file and flash its content to the device even if a debug lock is applied. The application firmware must be updated through the Gecko Bootloader after the SE Firmware upgrade if the SE GBL upgrade image file storage overwrites the existing application. Refer to section "Gecko Bootloader Operation - Secure Engine Upgrade" in [UG266](https://www.silabs.com/documents/public/user-guides/ug266-gecko-bootloader-user-guide.pdf)/[Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) for details.

## Secure Boot-Enabled Device

Simplicity Commander or Gecko Bootloader can be used to upgrade the SE Firmware on a Secure Boot-enabled device. The following table lists the scenarios of SE Firmware upgrade on the Secure Boot-enabled device.

| **Secure Debug** | **Device Erase** | **Debug Lock** | **State** | **SE Firmware Upgrade** |
|------------------|------------------|----------------|-----------|-------------------------|
| Disabled | Enabled | Disabled | Unlock | Simplicity Commander or Gecko Bootloader |
| Disabled | Enabled | Enabled | Standard debug lock | Simplicity Commander or Gecko Bootloader |
| Disabled | Disabled | Enabled | Permanent debug lock | Gecko Bootloader |
| Enabled | Disabled | Enabled | Secure debug lock | Gecko Bootloader |

> **Note**: Using Simplicity Commander to upgrade the SE Firmware on a Secure Debug Locked device is not recommended. It causes Secure Boot failure since the SE Firmware upgrade erases the signed host image for Secure Boot. The user may have issues when recovering a Secure Boot failure device with Device Erase disabled. See section *Recover Devices when Secure Boot Fails* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/) for details.

### Simplicity Commander

A signed SE Firmware upgrade application must be used for the upgrade. Instructions for signing the SE Firmware upgrade application are found in the sections "Signing for ECDSA-P256-SHA256 Secure Boot" and *Signing for Certificate-Based Secure Boot* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/).

If the `SECURE_BOOT_PAGE_LOCK_NARROW` or `SECURE_BOOT_PAGE_LOCK_FULL` in the [Secure Boot Items (mcu_flags) for Series 2](./09-enabling-secure-boot-and-tamper-configuration.md) table was enabled for Secure Boot or the standard debug lock applies, run:

```sh
commander security erasedevice --device EFR32MG21A010F1024 --serialno 440048205
```

to perform a device erase. Issue a power-on or pin reset to complete the device erase process.

```sh
Successfully erased device
DONE
```


To flash the signed SE Firmware upgrade application (`s2c1_se_fw_upgrade_app_1v2p9_signed.hex`), run:

```sh
commander flash s2c1_se_fw_upgrade_app_1v2p9_signed.hex --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Parsing file s2c1_se_fw_upgrade_app_1v2p9_signed.hex...
Writing 49152 bytes starting at address 0x00000000
Comparing range 0x00000000 - 0x0000BFFF (48 KB)
Erasing range 0x00000000 - 0x00007FFF (4 sectors, 32 KB)
Erasing range 0x00008000 - 0x0000BFFF (2 sectors, 16 KB)
Programming range 0x00000000 - 0x00001FFF (8 KB)
Programming range 0x00002000 - 0x00003FFF (8 KB)
Programming range 0x00004000 - 0x00005FFF (8 KB)
Programming range 0x00006000 - 0x00007FFF (8 KB)
Programming range 0x00008000 - 0x00009FFF (8 KB)
Programming range 0x0000A000 - 0x0000BFFF (8 KB)
DONE
```

**Notes**:

1. This method will **OVERWRITE** the signed bootloader and application firmware on the device. The user should then re-program the **SIGNED** [bootloader](./05-bootloader-firmware-programming.md) and [application firmware](./06-application-firmware-programming.md) after the SE Firmware upgrade.

2. If the `SECURE_BOOT_ANTI_ROLLBACK` in the tables in [Enabling Secure Boot and Tamper Configuration](./09-enabling-secure-boot-and-tamper-configuration.md) was enabled for Secure Boot, the device will prevent the signed SE Firmware upgrade when the host image version (e.g., Gecko Bootloader v1.12.0) is equal to or higher than the SE Firmware version (e.g., v1.2.9). Under this situation, the Gecko Bootloader should be used to upgrade the SE firmware. This method will **OVERWRITE** the bootloader version in SE flash with the SE Firmware version after the upgrade.

```sh
Parsing file s2c1_se_fw_upgrade_app_1v2p9_signed.hex...
Writing 49152 bytes starting at address 0x00000000
Comparing range 0x00000000 - 0x0000BFFF (48 KB)
Erasing range 0x00000000 - 0x00007FFF (4 sectors, 32 KB)
Programming range 0x00000000 - 0x00001FFF (8 KB)
Programming range 0x00002000 - 0x00003FFF (8 KB)
Programming range 0x00004000 - 0x00005FFF (8 KB)
Programming range 0x00006000 - 0x00007FFF (8 KB)
Programming range 0x00008000 - 0x00009FFF (8 KB)
Programming range 0x0000A000 - 0x0000BFFF (8 KB)
JLinkError: Failed to halt CPU.
DONE
```

### Gecko Bootloader

Refer to section *Generate a GBL Upgrade Image File* (Secure Engine Upgrade) in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/) for details.

The Gecko Bootloader can still parse the SE GBL upgrade image file and flash its content to the device even if a debug lock applies. The application firmware must be updated through the Gecko Bootloader after the SE Firmware upgrade if the SE GBL upgrade image file storage overwrites the existing application. Refer to section *Gecko Bootloader Operation - Secure Engine Upgrade* in [Developing and Debugging a Gecko Bootloader](https://docs.silabs.com/mcu-bootloader/latest/gecko-bootloader-developing-debugging/) for details.
