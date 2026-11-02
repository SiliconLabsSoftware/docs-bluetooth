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
3. Create a new Gecko Bootloader project.
    - For NCP and RCP projects, use *NCP BGAPI UART DFU*
    - For project using In-Place OTA DFU:
        - On Series 1 devices or GSDK projects, use *SoC Bluetooth In-place OTA DFU*
        - On Series 2 devices, use *SoC Bluetooth AppLoader OTA DFU*
    - For project using Application OTA DFU:
        - On Series 1 devices or GSDK projects, use *Internal Storage*
        - On Series 2 devices, use *SoC Internal Storage*
        - On Series 3 devices, use *SoC Storage*
4. Generate and build the bootloader.
5. Flash the bootloader image to the device:
    - On Series 1 devices, flash the .s37 file ending with `-combined.s37`, which contains both first and second stage bootloaders.
    - On Series 2 and 3 devices, flash the .s37 files ending with `-crc.s37`.

### Second Method

1. Build your Bluetooth application.
2. Create a new Gecko Bootloader project.
    - For NCP and RCP projects, use *NCP BGAPI UART DFU*
    - For project using In-Place OTA DFU:
        - On Series 1 devices or GSDK projects, use *SoC Bluetooth In-place OTA DFU*
        - On Series 2 devices, use *SoC Bluetooth AppLoader OTA DFU*
    - For project using Application OTA DFU:
        - On Series 1 devices or GSDK projects, use *Internal Storage*
        - On Series 2 devices, use *SoC Internal Storage*
        - On Series 3 devices, use *SoC Storage*
3. Generate and build the bootloader.
4. Copy the bootloader image (the one that ends with `-combined.s37` or `-crc.s37`) and the application image into the same folder.
5. Merge the bootloader and the application image: 
    ```console
    commander convert bootloader-uart-bgapi-crc.s37 your_application.s37 -o app+bootloader.s37
    ```
6. Flash the merged image to the device.

### Third Method

1. Flash a demo to your device.
    - Flash the Bluetooth - SoC Thermometer demo to your device. This will flash the SoC Thermometer application with Bluetooth in-place OTA DFU type Gecko Bootloader.
    - **OR**: Flash the Bluetooth - NCP Empty demo to your device. This will flash the NCP Empty application with BGAPI UART DFU type Gecko Bootloader.
2. Build your Bluetooth application.
3. Flash the image to the device.

### Important notes

- When flashing the application image, use the `.hex` or `.s37` output file. Flashing `.bin` files may overwrite or erase the bootloader.
- commander.exe can be found here: 
    - Studio v5: C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\commander
    - Studio v6: C:\Users\\<USER_NAME>\\.silabs\slt\installs\archive\Simplicity Commander
- Before flashing, verify the start and end addresses of the bootloader and application images to ensure that their flash regions do not overlap. An overlap may overwrite part of the bootloader or application and prevent the device from booting correctly.
