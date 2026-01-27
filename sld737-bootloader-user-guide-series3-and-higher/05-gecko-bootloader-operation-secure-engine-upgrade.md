# Gecko Bootloader Operation - Secure Engine Upgrade

The Secure Engine is upgradable and for both application and standalone type of bootloader a GBL file containing the Secure Engine Upgrade image has to be flashed or sent to the bootloader.

A bootloader upgrade can also be included in the same GBL file in application mode, or as a second GBL file in standalone mode. The figures that illustrate Gecko Bootloader operation in this section do not provide information about the bootloader memory layouts for different devices. For more details refer to the *Memory Space for Bootloading* section in [Bootloader Fundamentals](https://docs.silabs.com/bootloader/latest/bootloader-fundamentals/).

Signed and encrypted Secure Engine upgrade images are provided by Silicon Labs through Simplicity Studio. Upgrade images with the same or lower version number than the running Secure Engine will be ignored.

To download Secure Engine firmware images, connect a Series 3 device and select a preferred SDK. The Secure Firmware **Update to x.x.x** link appears in the Launcher Perspective, as shown in the following figure.

![screenshot](resources/sld737-image7.jpeg)

Click **Update to x.x.x** next to Secure FW: x.x.x. A warning dialog box appears. Click **Yes** to continue.

![screenshot](resources/sld737-image8.jpeg)

The Launcher Perspective is then updated so that the current Secure Firmware version and link version are the same.

![screenshot](resources/sld737-image9.jpeg)

The Secure Engine firmware images can be found in the *util/se\_release/public* directory under the Gecko SDK. Simplicity Studio displays the SE firmware version available in the Gecko SDK selected.

## Secure Engine Upgrade on Bootloaders with Communication Interface (Standalone Bootloaders)

The process is illustrated in the following figure.

![Standalone Bootloader: Secure Engine Bootloader Upgrade](resources/sld737-image10.png)

1. The device reboots into the bootloader.

2. A GBL file containing only a Secure Engine upgrade image is transmitted from the host to the device.

3. The device reboots into the Secure Engine.

4. The Secure Engine is replaced by the new version found in the pre-configured upgrade location.

5. The device boots into the bootloader.

### Downloading and Applying a Secure Engine GBL Upgrade File

When the bootloader has entered the receive loop, a GBL upgrade file containing a Secure Engine upgrade is transmitted to the bootloader. When a packet is received, it is passed to the image parser. The image parser parses the data and returns Secure Engine upgrade data in a callback. The bootloader core implements this callback and flashes the data to flash at the pre-configured storage data region.

When a complete Secure Engine upgrade image is received, the bootloader signals the Secure Engine that it should enter firmware upgrade mode. This is done by the Secure Engine communication interface that is used to signal that bootloader upgrade is ready to be performed.

## Secure Engine Upgrade on Application Bootloaders with Storage

The process is illustrated in the following figure.

![Application Bootloader: Secure Engine Upgrade](resources/sld737-image11.png)

1. A single GBL file containing a Secure Engine upgrade image is downloaded onto the storage medium of the device.

2. The device reboots into the bootloader.

3. The upgrade image will be fetched directly from the GBL file in storage instead of first copying the image to the pre-configured upgrade location.

4. The device reboots into the Secure Engine.

5. The Secure Engine is replaced by the new version found in the pre-configured upgrade location (or directly from storage, ref. 3b).

6. The device boots into the main bootloader.

7. The bootloader applies the application image from the GBL upgrade file.

8. The device boots into the application. Secure Engine upgrade is complete.
