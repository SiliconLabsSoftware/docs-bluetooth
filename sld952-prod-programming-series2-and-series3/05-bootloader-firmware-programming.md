# Bootloader Firmware Programming

If Secure Boot is enabled, a **SIGNED** version of the bootloader firmware must be programmed to the flash.

Instructions on how to sign the bootloader firmware can be found in sections *Signing for ECDSA-P256-SHA256 Secure Boot* and *Signing for Certificate-Based Secure Boot* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/).

The bootloader starting address is device dependent. For more information about the bootloader starting address, see [Memory Space for Bootloading](https://docs.silabs.com/mcu-bootloader/latest/bootloader-fundamentals/03-memory-space-for-bootloading).

Flashing the bootloader firmware using Simplicity Commander is similar to flashing the SE Firmware upgrade application.

```sh
commander flash --masserase <bootloader file> --device <device name> --serialno <J-Link serial number>
```

where `<bootloader file>` is the name of the bootloader firmware file.

For TrustZone-aware bootloaders, the `<bootloader file>` is the combined image of Secure and Non-secure bootloaders. To check the Boot status of the device, run the security status command.

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
Boot status	    : 0x20 - OK
DONE
```

Any Boot status other than `0x20 – OK` indicates that the secure boot process has failed. It means the bootloader firmware is either unsigned or incorrectly signed. The only way to recover is to flash a correctly-signed image (see section *Recover Devices when Secure Boot Fails* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/)).

For Series 3, make sure that code region 0 is closed.

```sh
commander security readregionconfig -d sixg301
```

```sh
Index        : 0
Size         : 32 kB
Protection   : Encrypted and authenticated
Closed       : False

Index        : 1
Size         : 1984 kB
Protection   : Encrypted and authenticated
Closed       : False

DONE
```

To close region 0, run the following command:

```sh
commander security closeregion 0 -d sixg301
```

> **Note**: the default behavior of the commander flash command is to close regions once programming is complete.

```sh
Successfully closed code region 0 (version 0x00000000)
DONE
```
