# Using Simplicity Connect Mobile App for OTA DFU

## Introduction

This tutorial explains how to perform a Device Firmware Upgrade (DFU) with Bluetooth Over-The-Air (OTA) update.

Any device that has OTA updates enabled in their GATT profile can have an OTA upgrade. Most of the example applications provided in the Bluetooth SDK already have OTA support built into the code. In these examples, the DFU mode is triggered through the Silicon Labs OTA service that is included as part of the application’s GATT database. OTA functionality can be added by installing either the `In-Place OTA DFU` or the `Application OTA DFU` software component in your project.

> **Note**: The `In-Place OTA DFU` or so called AppLoader method is an obsolete solution and not recommended for new developments.

For tutorial purposes, the `Bluetooth - SoC Empty` SDK example application will be upgraded to the `Bluetooth - SoC Thermometer` example application. Because the `Bluetooth - SoC Thermometer` has the Health Thermometer service for visual feedback, users can easily check that the functionality of the user application has changed.

### Requirements

- Wireless Starter Kit and Bluetooth-capable radio board
- Simplicity Studio 5
- Android or iOS mobile device

## First Steps

1. Download Simplicity Connect mobile app ([Android](https://play.google.com/store/apps/details?id=com.siliconlabs.bledemo) / [iOS](https://apps.apple.com/us/app/simplicity-connect/id1030932759)).

2. Connect the kit to your computer and select it in Simplicity Studio.

3. Create the **Bluetooth - SoC Empty** example project from Simplicity Studio Launcher.

4. Build the **Bluetooth - SoC Empty** project and flash the firmware image to the device.

5. Create the **Bluetooth - SoC Thermometer** example project.

6. Build the **Bluetooth - SoC Thermometer** project and double click the `create_bl_files.bat/sh/py` script in the project tree (BLE Post Build component might have to be installed) or use the [Post-Build Editor](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-building-and-flashing/post-build-editor) to generate the upgrade files. You may need to define two environment variables `PATH_SCMD` and `PATH_GCCARM` before running the script:

    <table>
        <thead>
            <tr>
                <th>Variable Name</th>
                <th>Example Variable Values</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td rowspan="3">PATH_SCMD</td>
                <td>C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\commander</td>
            </tr>
            <tr>
                <td>/Applications/Simplicity Studio 5.app/Contents/Eclipse/developer/adapter_packs/commander</td>
            </tr>
            <tr>
                <td>~/SimplicityStudio_v5/developer/adapter_packs/commander</td>
            </tr>
            <tr>
                <td rowspan="3">PATH_GCCARM</td>
                <td>C:\SiliconLabs\SimplicityStudio\v5\developer\toolchains\gnu_arm\12.2.rel1_2023.7</td>
            </tr>
            <tr>
                <td>/Applications/Simplicity Studio 5.app/Contents/Eclipse/developer/toolchains/gnu_arm/12.2.rel1_2023.7</td>
            </tr>
            <tr>
                <td>~/SimplicityStudio_v5/developer/toolchains/gnu_arm/12.2.rel1_2023.7</td>
            </tr>
        </tbody>
    </table>

    The script creates a folder named `output_gbl` under your project and multiple `.gbl` upgrade image files in this folder:

    - `application.gbl`: user application (including full Bluetooth stack)

    - `application-crc.gbl`: user application with a CRC32 checksum

    - `full.gbl`: user application and Bootloader/AppLoader (full update) for UART DFU, **not needed** in this example.

    - `full-crc.gbl`: user application and Bootloader/AppLoader (full update) with a CRC32 checksum for UART DFU, **not needed** in this example.

    A full update is needed only if the Bootloader/AppLoader needs to be updated. See [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps) for more details. Bootloader .s37 file has to be placed together with the create_bl_files scripts, named appropriately (i.e. bootloader-second-stage.s37) to be incorporated into the full.gbl file.

    ![output.gbl files](resources/outputgbl-batfile.png?darkModeUrl=resources/outputgbl-batfile-dark.png)

7. Transfer the `.gbl` files to your smartphone so the mobile app can find them. You can either transfer them via USB to any folder on your phone or place it to a cloud storage, which is available from your phone (e.g., Google Drive, Dropbox, iCloud, and so on).

8. Launch the Simplicity Connect mobile app.

## In the Simplicity Connect Mobile App

After you are in the app, do the following:

1. Go to the `Scan` menu and find and connect to your kit (default name "Empty Example").

2. Open the pop-up menu in the upper right corner and select `OTA Firmware`.

    ![OTA popup menu](resources/popup-menu.png?darkModeUrl=resources/popup-menu.png)

3. Select `Partial` and look for the gbl file in your smartphone.

4. Finally, tap `Upload` and your upgrade should start.

    ![OTA setup view](resources/blank-ota-setup-view.png?darkModeUrl=resources/blank-ota-setup-view.png)  ![OTA progress](resources/upload.png?darkModeUrl=resources/upload.png)

After the OTA process has finished, verify that the kit is now running the `Bluetooth - SoC Thermometer` example application. You can find the kit in the `Bluetooth Browser` with a new name "Thermometer Example".

## Note

To enable the Bluetooth OTA upgrade, the target device must be programmed with the Gecko Bootloader. This is an application bootloader, which requires that the new firmware image acquisition is managed by the application.

Running the "Demos" in Simplicity Studio will flash the bootloader and user application to the device. However, flashing an "Example Project" flashes the application only. If your OTA upload stops at 0% and you get a message on an Android phone saying "GATT CMD STARTED", that might indicate a missing or incorrect bootloader. In this case, do the following:

1. Click on Create New Project from the Launcher Perspective to create a Gecko Bootloader project for your kit. Select the **"Internal Storage Bootloader"** example project with a suitable configuration for your storage size.

2. In the `<projectname>.slcp` file of the bootloader project, you can configure some options.

3. Build the project and find the bootloader image in the build directory named `“GNU ARM <compiler version number> - Default”`. For Series 1 devices, you need the `<projectname>-`**combined**`.s37` file that is the combined image of the first stage bootloader and the main bootloader with a CRC32 checksum. For Series 2 devices, you need the `<projectname>.s37` file that is the main bootloader.

4. Flash the bootloader image to the device.

> **Note**: If you are using one of the **"Internal Storage Bootloader"** examples, you have to install the `Application OTA DFU` software component in your **"Empty Example"** project.

## Additional Resources

- [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps)

- [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](/bluetooth/{build-docspace-version}/bootloader-user-guide-gsdk-4)

- [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](/bluetooth/{build-docspace-version}/bootloader-user-guide-series3-and-higher)
