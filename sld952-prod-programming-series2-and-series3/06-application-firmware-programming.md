# Application Firmware Programming

If the Secure Boot option is enabled in the bootloader, a **SIGNED** version of the application firmware must be programmed to the flash.

Instructions on how to sign the application firmware can be found in sections *Signing for ECDSA-P256-SHA256 Secure Boot* and *Signing for Certificate-Based Secure Boot* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/).

The application firmware starting address is device dependent. For more information about the application starting address, see section *Memory Space For Bootloading* in [Bootloader Fundamentals](https://docs.silabs.com/mcu-bootloader/latest/bootloader-fundamentals/).

Flashing the application firmware using Simplicity Commander is similar to flashing the SE Firmware upgrade application.

```sh
commander flash <application file> --device <device name> --serialno <J-Link serial number>
```

where `<application file>` is the name of the application firmware file.

For TrustZone-aware applications, the `<application file>` is the combined image of Secure and Non-secure applications.

> **Note**: Do not use the `--masserase` option to flash the application firmware since it will erase the bootloader at the starting address.

For Series 3 devices, make sure that  all code regions are closed

```sh
commander security closeregion 1 -d simg301
```

If the region was not previously closed, the command completes successfully:

```sh
Successfully closed code region 1 (version 0x00000000)
DONE
```

If the region was already closed, the response is as follows:

```sh
ERROR: DCI command failed due to: Invalid command
DONE
```

**Notes**:

1. The default configuration is for the entire application to reside in region 1, applications that use additional regions must close those regions as well.

2. The default behavior of the commander flash command is to close the region once programming is complete.
