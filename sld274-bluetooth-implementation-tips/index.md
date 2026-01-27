# Implementation Tips

## Why Doesn't my Sample Application Run?

Starting in Bluetooth SDK version 2.7.x, the legacy bootloader was deprecated, which means that if you build and flash one of our sample applications, you must now also build and flash the Gecko bootloader. The most reliable method is to build a bootloader for your device depending on your requirements (e.g., OTA or UART DFU capability) and flash the bootloader image using Simplicity Commander, as described on this page: [Adding Gecko Bootloader to Bluetooth Projects](/bluetooth/{build-docspace-version}/bluetooth-bootloading-firmware-upgrade/adding-gecko-bootloader). This ensures that your device has both stages of the bootloader and will allow your application to run.

If your device does not have a bootloader, you might experience the application failing to start and the debugger showing the message "failed to read memory" in the disassembly window.

> **Notes**:
> - On series 1 devices (EFR32xG1x) both first and second stage bootloader must be flashed to the device. After building the bootloader, search for the image file that ends with *-combined.s37*.
> - On some devices the bootloader does not have a dedicated flash area (i.e., it is placed in the main flash to start address 0x00000000), and therefore flashing the application in .bin format may overwrite the flashed bootloader. Make sure to always use the .hex, .s37 or .gbl format when flashing the application to avoid erasing the bootloader.
> - If you flash one of the precompiled demos from Simplicity Studio, it will also automatically flash a bootloader. This is a fast way of flashing the bootloader if you are using the WSTK. If you are using custom hardware, build a custom bootloader.

For more information about the Gecko bootloader, see [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](/bluetooth/{build-docspace-version}/bootloader-user-guide-gsdk-4) or [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](/bluetooth/{build-docspace-version}/bootloader-user-guide-series3-and-higher).

## What is the Factory-Programmed Firmware in the BGMx Modules?

Bluetooth-capable SoCs (EFR32BG and EFR32MG devices) normally come without a preprogrammed firmware from the factory. However, some Bluetooth modules (such as BGM111, BGM121, and so on) are preprogrammed with a default firmware. The table below discusses the firmware that the different modules have when they are shipped from the factory. **Note that the default firmware is in most cases not suitable for production use because of the limitations mentioned later in this document.**

### Factory Firmware

| Modules | Factory Firmware |
|---------|-----------|
| :para[BGM111A256V2 BGM111A256V2R] :para[BGM111E256V2 BGM111E256V2R] :para[BGM113A256V2 BGM113A256V2R] | :custom-cell[These modules come with a default NCP firmware including Bluetooth stack v1.0.0 (build 615) + a 4k UART bootloader.]{style=valign-middle} |
| :para[BGM111A256V21 BGM111A256V21R BGM111E256V21 BGM111E256V21R] :para[BGM113A256V21 BGM113A256V21R ] :para[BGM121A256V2 BGM121A256V2R BGM121N256V2 BGM121N256V2R] :para[BGM123A256V2 BGM123A256V2R BGM123N256V2 BGM123N256V2R] :para[BGM11S12F256GA-V2 BGM11S12F256GA-V2R BGM11S22F256GA-V2 BGM11S22F256GA-V2R] | :custom-cell[These modules come with a default NCP firmware including Bluetooth stack v2.0.2 + a 16k UART bootloader. ]{style=valign-middle}|
| :custom-cell[Newer modules (such as BGM13P, BGM220P, etc.)]{style=valign-middle} | The default (factory programmed) firmware is listed in the data sheet of the module under Ordering Information. Pin configuration is also added to the data sheet. |

The pin configuration can be found in each device data sheets, in the typical connection diagrams chapter, Host UART figure.

### GATT Database of Factory Firmware

The factory firmware includes a default GATT database with default services, such as the Generic Access service. In most applications, users will need custom characteristics added to the GATT database. The GATT database of the device can be updated only by upgrading the device firmware. You can do this either by using the bootloader of the device or by flashing the new firmware to the device.

### Factory Firmware Bootloaders

The modules listed in the first two rows of the table are preprogrammed with a legacy UART bootloader (not Gecko UART Bootloader). These bootloaders have limited capabilities:

- With the 4k bootloader, you can't upgrade to the stack version v2.0 or newer

- With the 16k bootloader, you can't upgrade to the stack version v2.7 or newer, since the new versions need Gecko Bootloader

  > See [Bluetooth version 2.9.1 release notes](https://www.silabs.com/documents/login/release-notes/bt-software-release-notes-2.9.1.pdf)

- With the 16k bootloader, you can upgrade to stack versions v2.0, v2.1, v2.3, v2.4, v2.6. However, you may need to modify the linker script

- The legacy bootloader doesn't have upgrade capability

**Because of these limitations, expose the programming pins on your custom board and flash the devices with the latest Gecko Bootloader and the latest Bluetooth stack.** On new devices preprogrammed with the Gecko bootloader, use the Gecko Bootloader to upgrade both the firmware and the bootloader.

## Why is Pairing with iOS not Working?

Since iOS 9.1, **pairing without bonding** is no longer supported by iOS. Trying to pair when not in bondable mode will cause the connection to fail for devices running iOS 9.1 or newer.

## How to configure CTUNE

The examples have the crystal tune (CTUNE) settings for both HFXO and LFXO set by default to work with all of the Silicon Labs’
Bluetooth modules, reference designs, and radio boards. However, in some cases the end-product design requires specific crystal calibration, either per device or per design. The CTUNE value can be adjusted according to the design using the clock manager.
For more information on configuring the HFXO and LFXO, refer to the EFR32 Reference Manual.

### Default HFXO CTUNE Value

The system checks multiple sources for the default HFXO CTUNE value, using the following logical order:

1. CTUNE PSKEY is set. This key has ID 50 (32 in hex) and contains 2 bytes of data for the 16 bit CTUNE value. This can be programmed with the BGAPI command `sl_bt_nvm_save`. **This method is not recommended**.
2. If the `clock_manager_oscillator_calibration_override` component is included in the application, that will initialize oscillator calibration settings using override key values stored in NVM. The override is stored in NVM3 ID HFXO_CTUNE_NVM3_RESERVED_ID 0x89800. See [Clock Manager](https://docs.silabs.com/gecko-platform/latest/platform-service/clock-manager) for more details how to write and read the override value.
3. Calibration value exists in DEVINFO. Some modules contain a factory-programmed value in the DEVINFO-page.
4. Manufacturing token exists in the user data page. This is programmed by the developer, or it can be automatically set by Simplicity Studio if the board EEPROM contains the value. This token consists of 2 bytes, located at offset 0x0100 from the starting address of the User Data page. Refer to [Bringing Up Custom Devices for the EFR32MG and EFR32FG Families](https://docs.silabs.com/zigbee/latest/custom-nodes-efr32/) to how to use manufacturing tokens.
5. If nothing else is found, use the default value from the clock manager.

> Setting CTUNE via the BGAPI and NVM3 (step 1 above) is a Bluetooth-specific solution and is not recommended for new designs. New designs and applications should use the CTUNE setting mechanisms provided by the platform components, as these guarantee proper CTUNE settings regardless of which wireless stacks are running.
