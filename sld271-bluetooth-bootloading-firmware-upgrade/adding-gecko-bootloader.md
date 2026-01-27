# Adding Gecko Bootloader to Bluetooth Projects

**Bluetooth projects are configured so that, by default, they need a bootloader. However, the example projects do not include Gecko Bootloader by default, so you have to add it separately.**

> If you are sure that you won't need a bootloader, you may uninstall the "OTA DFU" and "Bootloader Application Interface" software components from your project. This makes it possible to start the application without a bootloader. However, it is strongly recommended to keep these components and add a bootloader to your project to make firmware upgrades possible.

Although some devices are shipped with preprogrammed bootloaders, it is always recommended to flash the latest Gecko Bootloader to your device.

- EFR32BG1 devices are preprogrammed with the legacy bootloader. The legacy bootloader is a one-stage simple bootloader with limited capabilities compared to the Gecko Bootloader. UART DFU and OTA DFU are the two types of the legacy bootloader. UART DFU upgrades the firmware using UART while OTA DFU upgrades the firmware using the Bluetooth connection. Devices are preprogrammed with UART DFU bootloader, but the support for the legacy bootloader was discontinued in SDK v3.0. As a result, you have to add Gecko Bootloader to your project to overwrite the legacy bootloader. Gecko Bootloader also has UART and OTA configuration, which is compatible with the legacy bootloaders.
- EFR32BG12 and EFR32BG13 devices are preprogrammed with a dummy bootloader, which can start the application but can't upgrade it. As a result, it is essential to overwrite it with the Gecko Bootloader.
- EFR32BG21 and EFR32BG22 devices are not preprogrammed with any bootloader.
- BGM modules are usually preprogrammed with a UART bootloader, see 'What is the Factory-Programmed Firmware in the BGMx Modules?' in [Implementation Tips](/bluetooth/{build-docspace-version}/bluetooth-implementation-tips/index#what-is-the-factory-programmed-firmware-in-the-bgmx-modules) or the data sheet of your module.

## Instructions for Adding a Gecko Bootloader to a Bluetooth Project

### First Method

1. Build your Bluetooth application.

2. Flash your Bluetooth application (.s37 or .hex or .bin) to the device.

3. Create a new Gecko Bootloader project, e.g., Bluetooth in-place OTA DFU Bootloader or BGAPI UART DFU Bootloader. You can find these example projects after selecting your device under the Example Projects & Demos tab of the Launcher view of Simplicity Studio 5.

4. Generate and build it.

5. For series 1 devices flash the .s37 file that ends with `–combined.s37` (e.g., `bootloader-uart-bgapi-combined.s37`). For series 2 devices flash the .s37 files that ends with `-crc.s37`.

6. To flash a new version of the application, ensure that you use .hex or .s37 (or .gbl) format because the .bin format will overwrite the bootloader on some devices.

### Second Method

1. Build your Bluetooth application.
2. Create a new Gecko Bootloader project, e.g., Bluetooth in-place OTA DFU Bootloader or BGAPI UART DFU Bootloader. You can find these example projects after selecting your device under the Example Projects & Demos tab of the Launcher view of Simplicity Studio 5.
3. Generate and build it.
4. Copy the bootloader image (the one that ends with `-combined.s37` or `-crc.s37`) and the application image into the same folder.
5. Merge the bootloader and the application image:

    `commander convert bootloader-uart-bgapi-crc.s37 your_application.s37 -o app+bootloader.s37`

6. Flash the merged image to the device.

### Third Method

1. Flash a demo to your device.

    - Flash the Bluetooth - SoC Thermometer demo to your device. This will flash the SoC Thermometer application with Bluetooth in-place OTA DFU type Gecko Bootloader.
    - **OR**: Flash the Bluetooth - NCP Empty demo to your device. This will flash the NCP Empty application with BGAPI UART DFU type Gecko Bootloader.

2. Build your Bluetooth application.

3. Flash the .hex or the .s37 file to your device.

**Note**: commander.exe can be found here: C:\SiliconLabs\SimplicityStudio\vX\developer\adapter_packs\commander.
