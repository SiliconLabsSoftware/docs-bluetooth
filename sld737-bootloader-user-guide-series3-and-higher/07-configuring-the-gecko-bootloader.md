# Configuring the Gecko Bootloader

## Configuring Storage

Gecko Bootloaders configured as application bootloaders must include an API to store and access image files. This API is based on the concept of storage slots, where each slot has a predefined size and location in memory and can be used to store a single upgrade image. Slots are configured in the **Bootloader Storage Slot Setup** component.

When multiple storage slots are configured, a bootload list is used to indicate the order in which the bootloader should access slots to find upgrade images. If multiple storage slots are supported, the application should write the bootload list by calling bootloader_setImageToBootload before rebooting into the bootloader to initiate a firmware upgrade process. The bootloader attempts to verify the images in these storage slots in sequence and applies the first image to pass verification. If only a single storage slot is supported, the bootloader uses this slot implicitly. A maximum of three slots may be configured in the **Bootloader Storage Slot Setup** component.

### Storage Configuration

When configuring a Gecko Bootloader to obtain images fromflash, modify the following.

The **base address of the storage area** should be configured in the **Common Storage** component. This is the address at which the bootloader will place the prioritized list of storage slots to attempt to bootload from, if more than one storage slot is configured. In the default configuration, only a single storage slot is configured, so this value is set to 0, and isn’t used. If more than one storage slot is configured, this value needs to be configured too.

The **location and size of the storage slots** can be configured using the **Bootloader Storage Slot Setup** Component (supports a maximum of three configurable storage slots). The addresses input here are absolute addresses (they are not offsets from the base address). If more than a single slot is configured, enough space must be reserved between the base address as configured in the **Common Storage** component and the first storage slot configured in the **Bootloader Storage Slot Setup** component. Enough space to fit two copies of the bootload list must be reserved. These two copies need to reside on different flash pages, to provide redundancy in case of power loss during writing. Two full flash pages therefore need to be reserved. The following figure illustrates how the storage area can be partitioned.

![Internal Storage Area Configurations](resources/sld737-image17.png)

>**Note**: The storage area partitioning in the example for two storage slots above does not take any NVM system into account. If using an NVM3 , take care to place and size the storage area in such a way that bootloader storage does not overlap with NVM3.

## Compressed Upgrade Images

The Gecko Bootloader optionally supports compressed GBL files. In a compressed GBL file, only the application upgrade data and bootloader upgrade data is compressed, any metadata (if present) stays uncompressed.  Signature and encryption operations on a compressed GBL work identically to on an uncompressed GBL.

To be able to use compressed upgrade images, a decompressor for the relevant compression algorithm must be added to the Gecko bootloader. The following table shows which compression algorithms are supported by the Gecko Bootloader, and which Bootloader component should be added to enable the feature. The table also shows how much space the decompressor takes up in the bootloader, and how big of a size reduction to expect for the compressed application upgrade image. Be aware of the bootloader size requirement. The bootloader space might be too small to fit the decompressors, depending on the device and enabled components.

| Compression Algorithm | Component | Bootloader Size Requirement | Application Upgrade Size Reduction (typical) |
|-----------------------|-----------|-----------------------------|-------------------------------------------------|
| LZ4 | GBL Compression (LZ4) | \< 1 kB | ~ 10% |
| LZMA | Bootloader Compression (LZMA) | ~5 kB flash, 18 kB RAM | ~ 30% |

It is important to note that the compressed GBL file stays compressed while being transferred to the device, and while it is stored in the upgrade area. It is decompressed by the bootloader when the upgrade is applied. This means that the running application in main flash will be identical to one that was installed using an uncompressed (normal) GBL file.

Compressed GBL files can only be decompressed by the bootloader when running standalone, not through the Application Interface. This means that upgrade image verification performed by the application prior to reboot will not attempt to decompress the application upgrade, it will only verify the signature of the compressed payload. After rebooting into the bootloader, it will decompress the image as part of the upgrade process.

### LZMA Compression Settings

LZMA decompression is only supported for images compressed with certain compression settings. Simplicity Commander automatically uses these settings during gbl4 creation.

- Probability model counters: lp + lc \<= 2. Simplicity Commander uses lp=1, lc=1.

- Dictionary size no greater than 8 kB. Simplicity Commander uses 8 kB.

Together, these settings cause the decompressor to require 18 kB of RAM for decompression – 10 kB for the counters and 8 kB for the dictionary.

The Gecko bootloader only supports decompressing payloads that contain the end mark as the last 8 bytes of the compressed stream.

## Bootloader Example Configurations

The following sections describe the key configuration options for the example bootloader applications.

>**Note**: Security features are disabled for all example configurations. In development, Silicon Labs strongly recommends enabling security features to prevent unauthorized parties from uploading untrusted program code. See the *Using Application Image Security Features* section on the *Gecko Bootloader Security Features* page to learn how to configure the security features of the Gecko Bootloader.

### UART XMODEM Bootloader

Standalone bootloader for Series 3 devices running the EmberZNet PRO and Silicon Labs Connect protocol stacks, using XMODEM-CRC over UART.

In this configuration, the **UART XMODEM** communication component, **XMODEM Parser** component, and **Bootloader UART Driver** component are installed. For the example application to run on a custom board, the GPIO ports and pins used for UART need to be configured in the **Bootloader UART Driver** component. Here, Hardware Flow Control can be enabled or disabled, and the baud rate and pinout can be configured.

The **GPIO activation** component is also installed by default, allowing bootloader entry into firmware upgrade mode by activating a GPIO through reset. The GPIO pin used can be configured here. This component can be uninstalled if this functionality is not desired.

### BGAPI UART DFU Bootloader

Standalone bootloader for the Bluetooth protocol stack, using the BGAPI protocol for UART DFU. This bootloader should be used for all NCP-mode Bluetooth applications.

In this configuration, the **BGAPI UART DFU** communication component and **Bootloader UART Driver** component are installed. For the example application to run on a custom board, the GPIO ports and pins used for UART need to be configured in the **Bootloader UART Driver** component. Here, Hardware Flow Control can be enabled or disabled, and the baud rate and pinout can be configured.

The GPIO activation component is also installed by default, allowing bootloader entry by activating a GPIO through reset. The GPIO pin used can be configured here. This component can be uninstalled if this functionality is not desired.

### Storage Bootloader

Application bootloader for all wireless protocol stacks, using flash to store upgrade images received over the air by the application.

Multiple examples are provided, including configurations for 2 MB, 3 MB, 4 MB & 8 MB flash memory devices . **The storage layout should be modified before running the bootloader on any other devices**. In this configuration, the flash and common storage components are installed. The base address of the storage area is configured in the **Common Storage** component. The location and size of the storage slots can be configured using the **Bootloader Storage Slot Setup** component (provides up to three configurable storage slots). Default example applications are provided with configurations for both single storage slot and multiple storage slots.

The default storage slot configurations provided by the Gecko Bootloader **must** be configured to match the use-case-specific application configurations, as shown below.

| Sample Applications | Storage Offset | Storage Size |
|---------------------|----------------|--------------|
| Storage Bootloader (Single OTA Image of size 792kB) | 0x010E0000 | 0x000C6000 (811,008 bytes) |
| Storage Bootloader (Single OTA Image of size 1180kB) | 0x0128F000 | 0x00127000 (1,210,368 bytes) |
| Storage Bootloader (Single OTA Image of size 1540kB) | 0x01379000 | 0x00181000 (1,577,984 bytes) |

## Image Acquisition Application Example Configuration

These examples illustrate applications that acquire and store a GBL upload image for an application bootloader. For the running bootloader to accept an application upgrade, the new application version must be higher than the existing version.

## Setting a Version Number

To distinguish between different builds of the Gecko Bootloader, it is useful to set a version number. To perform a bootloader upgrade, not only must the running bootloader pass its integrity checks (see the *Downloading and Applying a Bootloader GBL Upgrade File* section on the *Gecko Bootloader Operation Bootloader Upgrade* page), but the bootloader upgrade image must also have a higher version number than the running bootloader image. A version number can be set using Simplicity Studio by configuring the **Bootloader Version Main Customer** option of the **Bootloader Core** component. This macro will be picked up by the config file **btl\_config.h**, where it is combined with the version number of the Gecko Bootloader files provided by Silicon Labs.

## Hardware Configuration

The Gecko Bootloader uses the Pin Tool for configuration of pinout and other hardware-related settings. When Pin Tool configuration is available for a bootloader component, the relevant settings are shown in the Component Editor for that component.

![Example of USART Configuration for UART Driver](resources/sld737-image18.png)

The standalone Pin Tool User Interface can also be used to configure settings for Gecko Bootloader if desired.

While the Pin Tool provides configuration for many different peripherals, the Gecko Bootloader uses only the following Pin Tool modules:

- SERIAL is used by the UART Driver component to configure baud rate, flow control mode and pinout.

- VCOM is used by the UART Driver component to enable the serial interface if necessary (only on Silicon Labs Wireless Starter Kits).

- BTL_BUTTON is used by the GPIO Activation component.

- CMU HFXO frequency setting is used by the delay driver to calibrate timing if the core is running from the HFXO.

Other settings, like CMU oscillator configuration or DCDC configuration, are not taken into consideration by the default bootloader code. If using these configuration settings is desired, the required code must be added in btl_main.c.

>**Note**: While the delay driver uses the HFXO frequency setting from Pin Tool, the HFXO enable setting is not used to initialize the HFXO on startup. This setting is only used when calling the bootloader through the Application Interface, and the application has switched to the HFXO prior to calling the Bootloader Application Interface API.
