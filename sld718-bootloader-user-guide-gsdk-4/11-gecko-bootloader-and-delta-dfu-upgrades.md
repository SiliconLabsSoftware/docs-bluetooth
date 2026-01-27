# Gecko Bootloader and Delta DFU Upgrades

With GSDK v2024.6.1, Gecko Bootloader support comes with support for delta DFU upgrades. This section provides an introduction to Delta DFU upgrades and describes the major points that should be considered while using the Delta DFU component.

## Introduction to Delta DFU

The main function of the image delta DFU is to calculate the difference between the current firmware stored on the device and the new firmware to be updated. This difference is used to create a patch file that can be applied to the current firmware, resulting in the new firmware. The patch file is created using the Simplicity Commander tool, transferred to the device with any wireless protocol as a GBL, and applied on top of the current firmware by the Gecko Bootloader. The patch file creation and application are both implemented in a library that is shared between these tools.

The following diagram illustrates the complete process.

![Delta DFU process: patch creation, transfer, and firmware update](./resources/sld718-image23.png)

## Gecko Bootloader Operation

The Delta DFU upgrades can be enabled by including the Delta DFU component in the Gecko Bootloader. Simplicity Commander supports the creation of a Delta DFU GBL file.

With Delta DFU enabled, the application bootloaders are equipped to accept Delta DFU GBL files. These GBL files contain a patch file that is extracted into the slot. The Gecko Bootloader creates the new firmware using this patch file and the currently running firmware within the slot. The slot size should have enough number of pages to hold the GBL file, patch file, and the new firmware binary in the slot. Each file starts on a new page.

The Delta DFU upgrades are triggered in the same way as in an application bootloader. Once the bootloader finds a GBL that contains Delta Patch, it extracts the patch file and then re-constructs the new firmware file within the slot. If the new firmware is created correctly, then the bootloader copies this new firmware to the application area, completing the upgrade.

To create the GBL file, Commander needs two input files, one of the firmware that is current and one that is the upgrade. For Delta DFU upgrades to work, it is essential that the firmware on the device and the firmware used to create the GBL are the same. Also, the version dependency tag is included in Delta DFU and should be correctly set by the user.

Delta DFU operation is illustrated in the following figure:

![Delta DFU flow: patch extraction, firmware reconstruction, and upgrad](./resources/sld718-image24.png)
