# Upgrading Gecko Bootloader

## Introduction

On Series 1 devices, the Gecko Bootloader consists of two stages, the first stage and second stage (main bootloader).

- The first stage is a dummy one. It can only start the main stage and overwrite it with an already uploaded upgrade image. The first stage can only be flashed to the device and can't be upgraded.
- The main bootloader (or **`main`** for short) starts the application and it can upgrade the application in several ways, such as using UART, SPI, Bluetooth, and so on. It has other features as well, such as handling multiple storage slots, verifying images, and so on. The second stage can be upgraded, in case you want to add a new feature, apply a security patch or change configuration.

On Series 2 devices, the Gecko bootloader consists of only the main bootloader. The main bootloader is upgradable through the **Secure Element**. The secure element may be hardware-based (SE), or virtual (vSE). The Secure Element provides functionality to install an image to address 0x0 in internal flash, by copying from a configurable location in internal flash. This makes it possible to have a 2-stage design, where the main bootloader is not present. However, the presence of a main bootloader is assumed throughout this document. The Secure Element is also upgradable.

## Upgrading Using UART

This section describes upgrading the main bootloader by using UART and the **BGAPI UART DFU** type Gecko Bootloader.

First, flash both the first and the second stage of the bootloader to your device. To flash the first and second stage bootloader with a Bluetooth application, follow the steps described in [Adding Gecko Bootloader to Bluetooth Projects](./adding-gecko-bootloader.md).

If you have already flashed the bootloader, the main bootloader can be upgraded in three steps:

1. Upload new main bootloader image in the device (it will be put to a predefined address).

2. Reset the device -> the first stage/Secure Element will overwrite the main bootloader with the new image.

3. Upload the application again because some part of it may have been corrupted when the main  image was uploaded.

   ![UART upgrade](resources/bootloader-upgrade-uart.png?darkModeUrl=resources/bootloader-upgrade-uart.png)

Follow these steps to upgrade the main bootloader:

1. Create a new bootloader project to upgrade to.

2. The bootloader can only be upgraded and not downgraded. As a result, increment the version number up with every upgrade. If you want to upload the same bootloader version with another configuration, increase `BOOTLOADER_VERSION_MAIN_CUSTOMER`. Find the "Other" tab in the AppBuilder of the Bootloader project (opening .isc file), and add `BOOTLOADER_VERSION_MAIN_CUSTOMER` to the defines. Define a value greater than 0 and increment it with every upgrade.

3. Generate and build the bootloader project.

4. Create a bootloader upgrade file (.gbl) from the .s37 output file (1). For example,

   `commander gbl create bootloader.gbl --bootloader bootloader-uart-bgapi.s37`

   > Use the .s37 file which does not end with "–combined”. The combined image contains first and second stage, but only the second stage is needed at this point.

5. Upload the upgrade image. For example, if you have an BGAPI UART DFU type bootloader, use the uart-dfu application (2) to do this:

   `uart-dfu COMn 115200 bootloader.gbl`

6. The bootloader image temporarily overwrites the application image, hence you have to upload the application again.

7. Build your Bluetooth application.

8. Create an application upgrade file (.gbl) from the .s37 output file, (1). For example,

   `commander gbl create application.gbl --app my_bluetooth_project.s37`

9. Upload the upgrade image. For example, if the upgraded bootloader is an BGAPI UART DFU type bootloader, use the uart-dfu application (2) to do this:

   `uart-dfu COMn 115200 application.gbl`

Now your bootloader is upgraded and your application is restored.

Note:

**(1)** commander.exe can be found under *C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\commander*

**(2)** uart-dfu.exe can be found under
*C:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\v3.x\app\bluetooth\example_host\uart_dfu\exe* (after building the project with *make* in the *uart_dfu* folder)

## Upgrading Over-the-Air (OTA)

This section describes upgrading the second stage/main bootloader using the Bluetooth connection OTA and **Internal Storage Bootloader** type Gecko Bootloader.

First, flash both the first stage and the main bootloader to your device. In order to do that, follow the steps described in [Adding Gecko Bootloader to Bluetooth Projects](./adding-gecko-bootloader.md).

If you have already flashed the bootloader, the main bootloader can be upgraded in four steps:

1. Upload the new main bootloader image along with an apploader image. Note that this will overwrite the current application image.

2. Reset the device -> the first stage/Secure Element will upgrade the main bootloader with the new bootloader image. Note that this process will overwrite some part of the current apploader image.

3. Reset the device -> the main bootloader will upgrade the corrupted apploader image with the newly uploaded apploader image.

4. Upload the application image again. This will upgrade the corrupted application image.

   ![OTA upgrade](resources/bootloader-upgrade-ota.png?darkModeUrl=resources/bootloader-upgrade-ota.png)

Follow these steps to upgrade the second stage:

1. Create a new bootloader project to upgrade to. If you upgrade the bootloader OTA (not via UART), the address of Slot0 should match in the old and upgraded bootloader versions!! If you need to change your slot configuration, follow the instructions in section [Upgrading Bootloader with Slot Address Change](#upgrading-bootloader-with-slot-address-change).
2. The bootloader can only be upgraded and not downgraded. As a result, increment the version number up with every upgrade. To upload the same bootloader version with another configuration, increase `BOOTLOADER_VERSION_MAIN_CUSTOMER`. Find the "Other" tab in the AppBuilder of the Bootloader project (opening .isc file), and add `BOOTLOADER_VERSION_MAIN_CUSTOMER` to the definitions. Define a value greater than 0 and increment it with every upgrade.
3. Generate and build the bootloader project.
4. Open your Bluetooth application project or create a new one.
5. Build your project if you haven’t done so yet.
6. Copy the second stage (main) bootloader image **bootloader-storage-internal-ble.s37** (the .s37 file that does not end with –combined.s37) from your Bootloader project into your Bluetooth application project.
7. Rename the bootloader image to **bootloader-second-stage.s37**
8. Run the **create_bl_files.bat/sh/py** file (possibly from outside of Simplicity Studio), which creates the GBL files in the output_gbl folder. Note: you may need to set up some environmental variables as described in [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps/02-bgapi-uart-device-firmware-upgrade-dfu#creating-upgrade-images-for-the-bluetooth-ncp-application) first.
9. Do a full OTA update on your devices the same way as a full OTA for upgrading the stack/apploader and application, but use these two files:

    - apploader-bootloader.gbl
    - application.gbl

During the OTA process the device resets itself twice, so there is no need for user interaction. After the OTA update, your bootloader is updated. You can check its version in the *system_boot* event of the Bluetooth stack or in the Gecko Bootloader version characteristic of the Silicon Labs OTA service (see [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps)).

![Mobile Application OTA Setup](resources/bgapp-ota.png?darkModeUrl=resources/bgapp-ota.png)

## Upgrading Bootloader with Slot Address Change

The following section discusses the special case when you want to upgrade your Gecko Bootloader via Bluetooth with an image that has a different slot configuration than the previous Gecko Bootloader you used on your device.

To upgrade the bootloader with a new slot configuration is problematic because the upgrade image is uploaded by the old bootloader to the old slot address, but after the bootloader upgrade the new bootloader tries to load the apploader image from the new address – this will definitely fail.

The following workaround can be used to solve this issue:

To upgrade from bootloader A that has **Slot0 address A**, to bootloader B that has **Slot0 address B**, follow these steps:

1. You have a bootloader with **Slot0 address A** in your device. This will upload the upgrade image to **address A**.

2. First, upgrade to a temporary multi-slot bootloader with **Slot0 address B** and **Slot1 address A**. This can load images from **address A** but will upload the next upgrade image to **address B**.

3. Finally, upgrade to the single slot bootloader with **Slot0 address B**, as originally intended. This can load images from **address B**.

### Configuring the Temporary Multi-Slot Bootloader

1. Create a new bootloader project. Use Bluetooth Internal/External storage bootloader, as you normally do.

2. Define the customer version string to N+1, where N is the version number of the original bootloader. You can do this by defining `BOOTLOADER_VERSION_MAIN_CUSTOMER` on the Other tab in the AppBuilder.

3. On the Storage tab, define the slots. Define Slot0 to the new address (with the new slot size), and Slot1 to the old address (with the old slot size). The slots can overlap in this case.

4. The multi-slot configuration needs a bootload info table that takes 8 kB, which has to be defined somewhere outside of Slots and outside of the bootloader and apploader area. Go to Plugins page, select Common Storage, and define the Start address for bootload info to e.g., 8 kB before the Slot with lower address. Make sure that the start address is aligned with the flash page size.

5. Press Generate to generate your project.

6. Finally, the bootload info table should be normally initialized by the app. This table tells which slots should be bootloaded in which order. However, because there is no valid application image after the bootloader upgrade, modify the bootloader code, as follows:

    - Find storage-common/btl_storage.c in your project.
    - Right click on it and select 'Copy Linked File into Project'. This will copy btl_storage.c and btl_storage.h into the project from the SDK.
    - In storage-common_inc/btl_storage.h, change the following line

      `#include "bootloadinfo/btl_storage_bootloadinfo.h"`

      to

        `#include "plugin/storage/bootloadinfo/btl_storage_bootloadinfo.h"`

    - In storage-common/btl_storage.c, change the following lines

      ```text
      ret = storage_getBootloadList(slotIds, BTL_STORAGE_BOOTLOAD_LIST_LENGTH);
      if (ret != BOOTLOADER_OK) {
        BTL_DEBUG_PRINTLN("BI err");
        return ret;
      }
      ```

      to

      ```text
      ret = storage_getBootloadList(slotIds, BTL_STORAGE_BOOTLOAD_LIST_LENGTH);
      if (ret != BOOTLOADER_OK) {
        BTL_DEBUG_PRINTLN("BI err");
        slotIds[0] = 0;
        slotIds[1] = 1;
      }
      ```

   This will ensure, that the bootloader will try to bootload from Slot0 first and then from Slot1.

7. Build your project and upgrade to this bootloader normally, before upgrading to the final bootloader.

The final bootloader can be configured as expected, but make sure that its version number is N+2.

For more information about the Gecko Bootloader, see [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](/bluetooth/{build-docspace-version}/bootloader-user-guide-gsdk-4) or [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](/bluetooth/{build-docspace-version}/bootloader-user-guide-series3-and-higher).
