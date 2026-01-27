
# Uploading Firmware Images Using OTA DFU

## Introduction

Gecko Bootloader can load application images into the application area from the following sources:

- Host controller via **UART/SPI**
- **Internal flash** (if the internal flash is big enough to accommodate both the application code and an upgrade image)
- **External flash**

OTA DFU (Over-The-Air Device Firmware Upgrade) is **not** implemented in the Gecko Bootloader. To upload images OTA, you will need a dedicated Bluetooth application. You can either use Apploader or user application, as follows:

- **Apploader**. The Apploader is a simple application, which is separated from the main application and has a minimal Bluetooth stack that handles the upload process.
- **User application** extended with the ability to upload firmware images to the flash.

The Apploader overwrites the application area directly (with help of the Bootloader). It can do so because it runs independently of the user application.

The user application cannot overwrite itself, hence it has to place the upgrade image into a dedicated flash area (Bootloader Slot) first, and then ask the bootloader to load the image into the application area.

![Sources for uploading images](resources/sources-for-uploading-images.png?darkModeUrl=resources/sources-for-uploading-images.png)

## OTA DFU Sequence Implemented in Apploader

Apploader uses the the following procedure to upload images:

1. App is running.

   ![Image storage when the app is running](resources/ota-1.png?darkModeUrl=resources/ota-1.png)

2. Device is reset into DFU mode.

3. In this mode the bootloader starts the Apploader instead of the Application.

4. The Apploader advertises and waits for a Bluetooth connection.

5. The Apploader waits for the OTA start command (using OTA control characteristic).

6. The Apploader starts receiving the GBL file (using OTA data characteristic).

7. The headers of the GBL file are stored in RAM. Note: if you have an Apploader and Bootloader from before Bluetooth SDK v3.0, the headers are stored in the flash, in Storage Slot 0.

   ![GBL Header in storage](resources/ota-2.png?darkModeUrl=resources/ota-2.png)

8. The Apploader uses the API of the Gecko Bootloader to parse the GBL file.

9. After parsing the headers, the rest of the GBL file is decoded on-the-fly and the application is copied to the application area, overwriting the old application. Again, the Apploader uses the API of the bootloader for decoding and copying.

   ![New application in storage](resources/ota-3.png?darkModeUrl=resources/ota-3.png)

10. The Apploader receives the OTA end command and restarts the device in normal mode.

## OTA DFU Sequence Implemented in User Application

Developers should decide how to upload a firmware image into the Bootloader Slot. It is however strongly recommended to use the same OTA service and procedure that is used by the Apploader, making the two processes compatible. Therefore, when implementing OTA DFU in the User Application, it is recommended to follow this procedure:

1. App is running (device is not reset because the uploader is implemented in the user application).

   ![Application in storage](resources/ota-slot-1.png?darkModeUrl=resources/ota-slot-1.png)

2. If there are multiple slots, the remote device chooses which slot to upload to, using a custom characteristic (e.g., slot0. Slot0 can be either in the internal or in the external flash. The application does not have to know this because flash is handled by the bootloader).

3. The application waits for the OTA start command (using OTA control characteristic).

4. The application starts receiving the GBL file (using OTA data characteristic).

5. The GBL file is saved into slot0/slot1 *as it is*, using the bootloader API (the application does not have to know where Slot0/Slot1 is located because the bootloader API handles the writing).

   ![New application in storage](resources/ota-slot-2.png?darkModeUrl=resources/ota-slot-2.png)

6. The uploader receives the OTA end command.

7. *If there are multiple slots, the remote device may upload a new image to the other slot(s)*.

8. The remote device chooses which slot to bootload from using a custom characteristic.

9. The device is reset in bootloader mode, using the bootloader API.

10. The bootloader parses the given slot, retrieves the Application code from the GBL file, and copies the code to the application area from the internal/external flash.

    ![New application copied in storage](resources/ota-slot-3.png?darkModeUrl=resources/ota-slot-3.png)

11. The bootloader starts the new Application.

## Differences

From external viewpoint, the OTA process is almost the same whether the target application uses Apploader or handles the update in the user application code. The main steps (simplified) are:

1. Connect to the target device.

2. Reboot the target device into DFU mode if needed.

3. Upload the new firmware, packaged into a GBL file.

4. When the upload is finished, Gecko bootloader installs new firmware on top of the old version.

The main difference when compared to Apploader-based OTA is that there is no separate DFU mode and step 2 in the above list is not needed at all. The GBL file is uploaded to the target device under the control of the user application.

Some of the benefits of this approach are:

- The user application remains fully functional while the OTA image is being uploaded.

- It is possible to implement customer specific OTA service/protocol (with application level security if needed).

- EM2 deep sleep is supported during OTA upload.

- BLE security features (link encryption, bonding, and so on) can be used.

> Sleep and BLE encryption/bonding are not supported by Apploader to reduce the flash footprint.

An obvious disadvantage of this solution is the requirement to have a dedicated flash space available as a download area.

## Interaction between Gecko Bootloader and User Application

Gecko Bootloader has a key part in the OTA update. After GBL file is uploaded, Gecko Bootloader takes care of parsing the GBL file and installing the new firmware. The bootloader is also needed during the OTA file upload to write the image to the proper flash area.

Gecko Bootloader has an application interface exposed through a function table in the bootloader. This means that the user application can call functions implemented in the bootloader, even though the application is running in normal mode. In other words, the bootloader API functions are called from the user application context, but the implementation is in the Gecko bootloader project. For example, the user application can call the function **bootloader_writeStorage** to write data into the download area without knowing where the download area is actually located.

A list of the key bootloader API calls needed to implement OTA update is in [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps/04-working-with-apploader-and-secure-boot).

To access the Bootloader API, install the Bootloader Application Interface software component. This component is by default installed in the Bluetooth sample apps.

## Code Examples

You can find multiple code examples related to application level OTA DFU under [https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/firmware_upgrade](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/firmware_upgrade).
