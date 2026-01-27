# Overview

The Silicon Labs Gecko Bootloader is a common bootloader for all the newer MCUs and wireless MCUs from Silicon Labs. The Gecko Bootloader can be configured to perform a variety of functions, from device initialization to firmware upgrades. Key features of the bootloader are:

- Useable across Silicon Labs Gecko microcontroller and wireless microcontroller families

- In-field upgradeable

- Configurable

- Enhanced security features, including:

  - **Secure Boot**: When Secure Boot is enabled, the bootloader enforces cryptographic signature verification of the application image on every boot, using asymmetric cryptography. This ensures that the application was created and signed by a trusted party.

  - **Signed upgrade image file**: The Gecko Bootloader supports enforcing cryptographic signature verification of the upgrade image file. This allows the bootloader and application to verify that the application or bootloader upgrade comes from a trusted source before starting the upgrade process, ensuring that the image file was created and signed by a trusted party.

  - **Encrypted upgrade image file**: The image file can also be encrypted to prevent eavesdroppers from acquiring the plaintext firmware image.

The Gecko Bootloader uses a proprietary format for its upgrade images, called GBL (Gecko Bootloader file). These files have the file extension “.gbl”. See section [Gecko Bootloader File Format](./02-gecko-bootloader-file-format.md#gecko-bootloader-file-format) for more details.

On Series 1 devices, the Gecko Bootloader has a two-stage design, first stage and main stage, where a minimal first stage bootloader is used to upgrade the main bootloader. The first stage bootloader only contains functionality to read from and write to fixed addresses in internal flash. To perform a main bootloader upgrade, the running main bootloader verifies the integrity and authenticity of the bootloader upgrade image file. The running main bootloader then writes the upgrade image to a fixed location in internal flash and issues a reboot into the first stage bootloader. The first stage bootloader verifies the integrity of the main bootloader firmware upgrade image, by computing a CRC32 checksum before copying the upgrade image to the main bootloader location.

On Series 2 devices, the Gecko bootloader consists only of the main stage bootloader. The main bootloader is upgradable through the Secure Engine. The Secure Engine may be hardware-based, or virtual (software). If hardware-based, the implementation may be either with or without Secure Vault. Throughout this document, the following conventions will be used.

- HSE - Hardware Secure Engine, either with or without Secure Vault if not specified

- VSE - Virtual Secure Engine

- SE - Secure Engine (either HSE or VSE, in general)

The Secure Engine provides functionality to install an image to address 0x0 in internal flash, by copying from a configurable location in internal flash. This makes it possible to have a 2-stage design, where the main bootloader is not present. However, the presence of a main bootloader is assumed throughout this document.

To perform a main bootloader upgrade, the running main bootloader verifies the integrity and authenticity of the bootloader upgrade image file. The running main bootloader then writes the upgrade image to the upgrade location in flash and requests that the Secure Engine installs it. On some devices, the Secure Engine is also capable of verifying the authenticity of the main bootloader update image against a root of trust. The Secure Engine itself is upgradable using the same mechanism. See [Gecko Bootloader Operation - Secure Engine Upgrade](./05-gecko-bootloader-operation-secure-engine-upgrade.md#gecko-bootloader-operation-secure-engine-upgrade) for more details.

The main bootloader consists of a common core, drivers, and a set of components that give the bootloader specific capabilities. The common bootloader core is now provided as a full-source delivery, as opposed to previously being a combination of compiled libraries and source code while the components continue to be delivered as source code. The common bootloader core contains functionality to parse GBL files and flash their contents to the device.

The Gecko Bootloader can be configured to perform firmware upgrades in standalone mode (also called a standalone bootloader) or in application mode (also called an application bootloader), depending on the component configuration. Components can be installed in and configured through the Simplicity Studio IDE.

A standalone bootloader uses a communications channel to get a firmware upgrade image. NCP (network co-processor) devices always use standalone bootloaders. Standalone bootloaders perform firmware image upgrades in a single-stage process that allows the application image to be placed into flash memory, overwriting the existing application image, without the participation of the application itself. In general, the only time that the application interacts with a standalone bootloader is when it requests to reboot into the bootloader. Once the bootloader is running, it receives packets containing the firmware upgrade image over-the-air using Bluetooth or by a physical connection such as UART or SPI. To function as a standalone bootloader with a physical connection, a component providing a communication interface such as UART or SPI must be configured.

An application bootloader relies on the application to acquire the firmware upgrade image. The application bootloader performs a firmware image upgrade by writing the firmware upgrade image to a region of flash memory referred to as the download space. The application transfers the firmware upgrade image to the download space in any way that is convenient (UART, over-the-air, Ethernet, USB, and so on). The download space is either an external memory device such as an EEPROM or dataflash or a section of the device’s internal flash. The Gecko Bootloader can partition the download space into multiple storage slots and store multiple firmware upgrade images simultaneously. To function as an application bootloader, a component providing a bootloader storage implementation has to be configured.

Silicon Labs provides example bootloaders that come with a preconfigured set of installed components for configuration in either standalone or application mode. See section [Configuring the Gecko Bootloader](./07-configuring-the-gecko-bootloader.md#configuring-the-gecko-bootloader). The Silicon Labs Gecko SDK Suite also includes precompiled bootloader images for several different EFR32 devices. As of this writing, the images shown in the following table are provided.

>**Note**: The bootloader security features are not enabled in these precompiled images.

Table Prebuilt Bootloader Images

<table class="classic"><colgroup><col width="15%"><col width="20%"><col width="30%"><col width="15%"><col width="20%"></colgroup><thead><tr><th>Use</th><th>Wireless Stack</th><th>Image Name</th><th>Mode</th><th>Interface</th></tr></thead><tbody><tr><td><p>SoC</p></td><td><p>EmberZNet PRO</p></td><td><p>SPI Flash Storage Bootloader</p></td><td><p>Application</p></td><td><p>SPI Serial Flash</p></td></tr><tr><td><p>SoC</p></td><td><p>Bluetooth</p></td><td><p>Bluetooth In-Place OTA DFU Bootloader</p></td><td><p>Application</p></td><td><p>OTA/internal flash</p></td></tr><tr><td><p>NCP</p></td><td><p>EmberZNet PRO</p></td><td><p>UART XMODEM Bootloader</p></td><td><p>Standalone</p></td><td><p>UART (EZSP)</p></td></tr><tr><td><p>NCP</p></td><td><p>Bluetooth</p></td><td><p>BGAPI UART DFU Bootloader</p></td><td><p>Standalone</p></td><td><p>UART (BGAPI)</p></td></tr></tbody></table>

Note that on devices with a dedicated bootloader area (EFR32xG12 and later Series 1 devices), if the device is configured to boot to the bootloader area (that is, if bit 1 of the Config Lock Word 0 CLW0[1] is set), an image always must be present in the bootloader area. The device is factory-programmed with a dummy bootloader that simply jumps directly to the application in main flash. This means that when flashing a bootloader to a device with a dedicated bootloader area, this dummy bootloader is replaced. If later during development using the bootloader is no longer desired, CLW0[1] must be cleared or the dummy bootloader needs to be re-flashed. Platform-specific prebuilt dummy bootloader images are located in _./platform/bootloader/util/bin/_. Note that since the dummy bootloader only consists of a few instructions and doesn't pad out the remainder of the bootloader area, only the first flash page (where the first-stage bootloader resides) is overwritten, so the main stage bootloader would likely remain intact after programming the dummy bootloader. If desired, the rest of the flash pages in the bootloader area can then be erased separately.

On devices that do not have a dedicated bootloader area (EFR32xG1 and EFR32 Series 2), a dummy bootloader is not needed.

The following sections provide an overview of the Gecko Bootloader common core, drivers, and components. For details, including details on error codes and conditions, see the Gecko Bootloader API Reference, shipped with the SDK in the platform/bootloader/documentation folder.

The bootloader area can be fully erased using the `commander device pageerase --region @bootloader` command with Simplicity Commander. In this state, the device will not boot until CLW0[1] is cleared or the dummy bootloader is flashed. For more information on how to use Simplicity Commander with Gecko bootloader, see section [Simplicity Commander and the Gecko Bootloader](./08-simplicity-commander-and-the-gecko-bootloader.md#simplicity-commander-and-the-gecko-bootloader).

## Core

The bootloader core contains the bootloader’s main functions. It also contains functionality to write to the internal flash, an image parser to parse and act upon the contents of GBL upgrade files, and functionality to boot the application in main flash.

The image parser can also optionally support the legacy Ember Bootloader (EBL) file format, but none of the security features offered by the Gecko Bootloader are supported if support for legacy EBL files is enabled.

A version of the GBL image parser without support for encrypted upgrade images is also available. This version can be used in flash space constrained bootloader applications where encryption of the upgrade image is not required.

### Shared Memory

To exchange information between the bootloader and application, a section of SRAM is used. The contents of SRAM are preserved through a software reset, making the SRAM suitable as a communication channel between bootloader and application.

The shared memory has a size of 4 bytes, and is located at the first address of SRAM, 0x20000000. It is used to store a single word containing the reason for a reset. The structure of the reset cause word is defined in the Reset Information part of the Application Interface, in the file **btl_reset_info.h**, as 16 bits containing the reason, and 16 bits of signature indicating if the word is valid or not. If the signature reads 0xF00F, the reset reason is valid.

All 16-bit reset reasons used by Silicon Labs have the most significant bit set to zero. If custom reset reasons are desired, it is recommended to set the most significant bit to avoid conflicting definitions.

In addition to the reset causes defined in the Reset Information documentation, the bootloader will enter firmware upgrade mode if the shared memory contains the value 0x00000001. This value is supported for compatibility with certain legacy Bluetooth applications.

## Drivers

Different applications of firmware upgrade require different hardware drivers for use by the other components of the bootloader.

Driver modules include:

- Delay: Simple delay routines for use with components that require small delays or timeouts.

- SPI: Simple, blocking SPI master implementation for communication with external devices such as SPI flashes.

- SPI Slave: Flexible SPI Slave driver implementation for use in communication components implementing SPI protocols. This driver supports both blocking and non-blocking operation, with DMA (Direct Memory Access) backing the background transfers to support non-blocking operation.

- UART: Flexible serial UART driver implementation for use in communication components implementing UART protocols. This driver supports both blocking and non-blocking operation, with DMA backing the background transfers to support non-blocking operation. Additionally, support for hardware flow control (RTS/CTS) is included.

## Components

All parts of the bootloader that are either optional or that may be exchanged for different configurations are implemented as components. Each component may have a configuration header file, and one or more implementations. Components include:

- Communication

  - UART: XMODEM

  - UART: BGAPI

  - SPI: EZSP

  - Bluetooth: AppLoader

- Compression

- Debug

- GPIO Activation

- Security

- Storage

  - Internal flash

  - External SPI flash

### Communication

The Communication components provide an interface for implementing communication with a host device, such as a computer or a micro-controller. Several components implement the communication interface, using different transports and protocols.

- BGAPI UART DFU: By enabling the BGAPI communication component, the bootloader communication interface implements the UART DFU protocol using BGAPI commands. See _AN1053: Bluetooth® Device Firmware Update over UART for EFR32xG1 and BGM11x Series Products_ for more information about this legacy bootloader.

- Bluetooth:AppLoader: By enabling the Bluetooth AppLoader communication component, the bootloader communication interface implements over-the-air device firmware upgrade functionality using Bluetooth. See [Using the Gecko Bootloader with the Silicon Labs Bluetooth Applications](https://docs.silabs.com/bluetooth/latest/using-gecko-bootloader-with-bluetooth-apps/) for more information.

- EZSP-SPI: By enabling the EZSP-SPI communication component, the bootloader communication interface implements the EZSP protocol over SPI. This component makes the bootloader compatible with the legacy ezsp-spi-bootloader that was previously released with the EmberZNet wireless stack. See _AN760: Using the Ember Standalone Bootloader_ for more information about legacy Ember standalone bootloaders.

- UART XMODEM: By enabling the UART XMODEM communication component, the bootloader communication interface implements the XMODEM-CRC protocol over UART. This component makes the bootloader compatible with the legacy serial-uart-bootloader that was previously released with the EmberZNet wireless stack. See _AN760: Using the Ember Standalone Bootloader_ for more information about legacy Ember standalone bootloaders.

### Compression

The Compression components provide capability for the bootloader GBL file parser to handle compressed GBL upgrade images. Each compression component provides support for one (de)compression algorithm. At the time of writing, decompression of data compressed with the LZ4 and LZMA algorithms is supported, through the _GBL Compression (LZ4)_ and _GBL Compression (LZMA)_ components.

### Debug

This component provides the bootloader with support for debugging output. If the component is configured to enable debug prints, short debug messages will be printed over Serial Wire Output (SWO), which can be accessed in multiple ways, including using Simplicity Commander, and by connecting to port 4900 of the Wireless Starter Kit TCP/IP interface.

To turn on debug prints, enable the Debug component and select **Debug prints**. Select **Debug asserts** to enable assertions in the source code.

On Series 1 devices, also select the GPIO peripheral in the Pin Tool.

### GPIO Activation

This component provides functionality to enter firmware upgrade mode automatically after reset if a GPIO pin is active during boot. The GPIO pin location and polarity are configurable.

- GPIO: By enabling the GPIO activation component, the firmware upgrade mode can be activated by the push buttons.

- EZSP GPIO: The EZSP communication protocol over SPI can be used together with this component. By enabling the EZSP GPIO component, the firmware upgrade mode can be entered by activating the _nWake_ pin.

### Security

Security components provide implementations of cryptographic operations as well as functionality to compute checksums and to read cryptographic keys from manufacturing tokens.

Modules include:

- AES: AES decryption functionality

- CRC16: CRC16 functionality

- CRC32: CRC32 functionality

- ECDSA: ECDSA signature verification functionality

- SHA-256: SHA-256 digest functionality

### Storage

These components provide the bootloader with multiple storage options for SoCs. All storage implementations have to provide an API to access image files to be upgraded. This API is based on the concept of dividing the download space into storage slots, where each slot has a predefined size and location in memory and can be used to store a single upgrade image. Some storage implementations also support a raw storage API to access the underlying storage medium. This can be used by applications to store other data in parts of the storage medium that are not used for storing firmware upgrade images. Implementations include:

- **Internal Flash**: The internal flash storage implementation uses the internal flash of the device for upgrade image storage. Note that this storage area is only a download space and is separate from the portion of internal flash used to hold the active application code.

- **SPI Flash**: Two components are available for SPI Flash storage implementation.

  1. **SPI Flash Storage**: The SPI flash storage implementation supports a variety of SPI flash parts. The subset of devices supported can be configured in this component in Gecko Bootloader. (The default configuration if no devices are selected is to include drivers for all supported parts.) Including support for multiple devices requires more flash space in the bootloader. The SPI flash storage implementation does not support any write protection functionality. Supported SPI flash parts are shown in Table 1.2.

  2. **SPI Flash Storage SFDP**: The SPI Flash storage SFDP implementation supports all the SPI flash parts that support SFDP (Serial Flash Description Parameter) Standard from JEDEC. The SPI Flash type is detected automatically by querying the SFDP Parameter table present within the flash memory. All the properties of the SPI Flash are accessed from this parameter table. This component is not configurable since the flash is automatically detected. Any flash that supports the SFDP standard can be used with the Gecko Bootloader. However, performance delays might occur when compared to using the **SPI Flash Storage** component.

>**Note**: Low power devices are recommended for battery-operated applications. Use of the other listed devices will decrease battery life due to higher quiescent current, but this can be mitigated with external shutdown FET circuitry, if desired.

Table Supported Serial Dataflash/EEPROM External Memory Parts

<table class="classic"><colgroup><col width="50%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>Manufacturer Part Number</th><th>Size (kB)</th><th>Quiescent Current (µA Typical)*</th></tr></thead><tbody><tr><td><p>Macronix MX25R8035F (low power)</p></td><td><p>1024</p></td><td><p>0.007</p></td></tr><tr><td><p>Macronix MX25R6435SF (low power)</p></td><td><p>8192</p></td><td><p>0.007</p></td></tr><tr><td><p>Macronix MX25R3235F (low power)</p></td><td><p>4096</p></td><td><p>0.007</p></td></tr><tr><td><p>Spansion S25FL208K</p></td><td><p>1024</p></td><td><p>15</p></td></tr><tr><td><p>Winbond W25X20BVSNIG (W25X20CVSNJG for high-temperature support)</p></td><td><p>256</p></td><td><p>1</p></td></tr><tr><td><p>Winbond W25Q80BVSNIG (W25Q80BVSNJG for high-temperature support)</p></td><td><p>1024</p></td><td><p>1</p></td></tr><tr><td><p>Macronix MX25L2006EM1I-12G (MX25L2006EM1R-12G for high-temperature support)</p></td><td><p>256</p></td><td><p>2</p></td></tr><tr><td><p>Macronix MX25L4006E</p></td><td><p>512</p></td><td><p>2</p></td></tr><tr><td><p>Macronix MX25L8006EM1I-12G (MX25L8006EM1R-12G for high-temperature support)</p></td><td><p>1024</p></td><td><p>2</p></td></tr><tr><td><p>Macronix MX25L1606E</p></td><td><p>2048</p></td><td><p>2</p></td></tr><tr><td><p>Macronix MX25U1635E (2V)</p></td><td><p>2048</p></td><td><p>2</p></td></tr><tr><td><p>Atmel/Adesto AT25DF041A</p></td><td><p>512</p></td><td><p>15</p></td></tr><tr><td><p>Atmel/Adesto AT25DF081A</p></td><td><p>1024</p></td><td><p>5</p></td></tr><tr><td><p>Atmel/Adesto AT25SF041</p></td><td><p>512</p></td><td><p>2</p></td></tr><tr><td><p>Micron (Numonyx) M25P20</p></td><td><p>256</p></td><td><p>1</p></td></tr><tr><td><p>Micron (Numonyx) M25P40</p></td><td><p>512</p></td><td><p>1</p></td></tr><tr><td><p>Micron (Numonyx) M25P80</p></td><td><p>1024</p></td><td><p>1</p></td></tr><tr><td><p>Micron (Numonyx) M25P16</p></td><td><p>2048</p></td><td><p>1</p></td></tr><tr><td><p>ISSI IS25LQ025B</p></td><td><p>32</p></td><td><p>8</p></td></tr><tr><td><p>ISSI IS25LQ512B</p></td><td><p>64</p></td><td><p>8</p></td></tr><tr><td><p>ISSI IS25LQ010B</p></td><td><p>126</p></td><td><p>8</p></td></tr><tr><td><p>ISSI IS25LQ020B</p></td><td><p>256</p></td><td><p>8</p></td></tr><tr><td><p>ISSI IS25LQ040B</p></td><td><p>512</p></td><td><p>8</p></td></tr></tbody></table>

\* Quiescent current values are as of December 2017; check the latest part specifications for any changes.

## Compatibility

1. Silicon Labs recommends building the Gecko Bootloader and the application using the same GSDK. This implies that, if the Gecko Bootloader has been built using GSDK v4.0, it is recommended that the application also be built using GSDK v4.0.

2. Backward compatibility is supported wherein the Gecko Bootloader is built using a previous SDK version and the application is built using a newer SDK version.

3. In general, Silicon Labs does not recommend building the Gecko Bootloader using a newer SDK version with the application built using an older SDK version.

The below table attempts to explain the above-mentioned guidelines:

<table class="classic"><colgroup><col width="36%"/><col width="32%"/><col width="32%"/></colgroup><thead><tr><th><p>SDK version X &lt; SDK version Y</p></th><th><p>Application built using SDK version X</p></th><th><p>Application built using SDK version Y</p></th></tr></thead><tbody><tr><td><p>Gecko Bootloader built using SDK version X</p></td><td><p>Recommended</p></td><td><p>Compatible</p></td></tr><tr><td><p>Gecko Bootloader built using SDK version Y</p></td><td><p>Not recommended</p></td><td><p>Recommended</p></td></tr></tbody></table>
