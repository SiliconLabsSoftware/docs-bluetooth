# Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher

This guide describes the high-level implementation of the Silicon Labs Gecko Bootloader for Series 3 microcontrollers, SoCs (System on Chips), and NCPs (Network Co-Processors), and provides information on different aspects of configuring the Gecko Bootloader.

The Silicon Labs Gecko Bootloader is a common bootloader for all the newer MCUs and wireless MCUs from Silicon Labs. This guide is specific to Series 3 devices and above. The Gecko Bootloader can be configured to perform a variety of functions, from device initialization to firmware upgrades. Key features of the bootloader are:

- Usable across Silicon Labs Gecko microcontroller and wireless microcontroller families

- In-field upgradeable

- Configurable

- Enhanced security features, including:

- **Secure Boot**: When Secure Boot is enabled, the bootloader enforces cryptographic signature verification of the application image on every boot, using asymmetric cryptography. This ensures that the application was created and signed by a trusted party.

- **Signed upgrade image file**: The Gecko Bootloader supports enforcing cryptographic signature verification of the upgrade image file. This allows the bootloader and application to verify that the application or bootloader upgrade comes from a trusted source before starting the upgrade process, ensuring that the image file was created and signed by a trusted party.

- **Encrypted upgrade image**: The image file can also be encrypted to prevent eavesdroppers from acquiring the plaintext firmware image.

For Series 3 devices and above, Gecko Bootloader uses a proprietary format for its upgrade images, called GBL V4 (Gecko Bootloader file). These files have the file extension “.gbl”. See the *Gecko Bootloader File Format* page for more details. Series 3 and above devices do not support GBL versions lesser than v4.

On Series 3 devices, the Gecko bootloader consist of the main stage bootloader. The main bootloader is upgradable through the Secure Engine.

The Secure Engine provides functionality to install an image to code region 0 in the external flash.

To perform a bootloader upgrade, the running bootloader verifies the integrity and authenticity of the bootloader upgrade image file. The bootloader then writes the upgrade image to the upgrade location in external flash and requests that the Secure Engine installs it. On some devices, the Secure Engine is also capable of verifying the authenticity of the main bootloader update image against a root of trust. The Secure Engine itself is upgradable using the same mechanism. See the *Gecko Bootloader Operation - Secure Engine Upgrade* page for more details.

The bootloader consists of a common core, drivers, and a set of components that give the bootloader specific capabilities. The common bootloader core is provided as a full-source delivery. The common bootloader core contains functionality to parse GBL v4 files and flash their contents to the device.

The Gecko Bootloader can be configured to perform firmware upgrades in standalone mode (also called a standalone bootloader) or in application mode (also called an application bootloader), depending on the component configuration. Components can be installed in and configured through the Simplicity Studio IDE.

A standalone bootloader uses a communications channel to get a firmware upgrade image. NCP (network co-processor) devices always use standalone bootloaders. Standalone bootloaders perform firmware image upgrades in a single-stage process that allows the application image to be placed into flash memory, overwriting the existing application image, without the participation of the application itself. In general, the only time that the application interacts with a standalone bootloader is when it requests to reboot into the bootloader. Once the bootloader is running, it receives packets containing the firmware upgrade image by a physical connection such as UART or SPI. To function as a standalone bootloader with a physical connection, a component providing a communication interface such as UART or SPI must be configured.

An application bootloader relies on the application to acquire the firmware upgrade image. The application bootloader performs a firmware image upgrade by writing the firmware upgrade image to a region of flash memory referred to as the download space. The application transfers the firmware upgrade image to the download space in any way that is convenient (UART, over-the-air, Ethernet, USB, and so on). The download space is data flash or a section of the device’s flash. The Gecko Bootloader can partition the download space into multiple storage slots and store multiple firmware upgrade images simultaneously. To function as an application bootloader, a component providing a bootloader storage implementation must be configured.

Silicon Labs provides example bootloaders that come with a preconfigured set of installed components for configuration in either standalone or application mode. See the *Configuring the Gecko Bootloader* page. The following sections provide an overview of the Gecko Bootloader common core, drivers, and components. For details, including details on error codes and conditions, see the Gecko Bootloader API Reference, shipped with the SDK in the platform/bootloader/documentation folder.

## Core

The bootloader core contains the bootloader’s main functions. It also contains functionality to write to the external flash, an image parser to parse and act upon the contents of GBL v4 upgrade files, and functionality to boot the application in main flash.

A version of the GBL v4 image parser without support for encrypted upgrade images is also available. This version can be used in flash space constrained bootloader applications where encryption of the upgrade image is not required.

### Shared Memory

To exchange information between the bootloader and application, a section of SRAM is used. The contents of SRAM are preserved through a software reset, making SRAM suitable as a communication channel between bootloader and application.

The shared memory has a size of 4 bytes, and is located at the first address of SRAM, 0x20000000. It is used to store a single word containing the reason for a reset. The structure of the reset cause word is defined in the Reset Information part of the Application Interface, in the file **btl\_reset\_info.h**, as 16 bits containing the reason, and 16 bits of signature indicating if the word is valid or not. If the signature reads 0xF00F, the reset reason is valid.

All 16-bit reset reasons used by Silicon Labs have the most significant bit set to zero. If custom reset reasons are desired, it is recommended to set the most significant bit to avoid conflicting definitions.

## Drivers

Different applications for firmware upgrade require different hardware drivers for use by the other components of the bootloader.

Driver modules include:

- Delay: Simple delay routines for use with components that require small delays or timeouts.

- SPI Slave: Flexible SPI Slave driver implementation for use in communication components implementing SPI protocols. This driver supports both blocking and non-blocking operation, with DMA (Direct Memory Access) backing the background transfers to support non-blocking operation.

- UART: Flexible serial UART driver implementation for use in communication components implementing UART protocols. This driver supports both blocking and non-blocking operations, with DMA backing the background transfers to support non-blocking operation. Additionally, support for hardware flow control (RTS/CTS) is included.

## Components

All parts of the bootloader that are either optional or that may be exchanged for different configurations are implemented as components. Each component may have a configuration header file, and one or more implementations. Components include:

- Communication

- UART: XMODEM

- UART: BGAPI

- Compression

- Debug

- GPIO Activation

- Security

- Storage

### Communication

Communication components provide an interface for implementing communication with a host device, such as a computer or a micro-controller. Several components implement the communication interface, using different transports and protocols.

- BGAPI UART DFU: By enabling the BGAPI communication component, the bootloader communication interface implements the UART DFU protocol using BGAPI commands.

- UART XMODEM: By enabling the UART XMODEM communication component, the bootloader communication interface implements the XMODEM-CRC protocol over UART. This component makes the bootloader compatible with the legacy serial-uart-bootloader that was previously released with the EmberZNet wireless stack. See *AN760: Using the Ember Standalone Bootloader* for more information about legacy Ember standalone bootloaders.

### Compression

Compression components provide capability for the bootloader GBL file parser to handle compressed GBL v4 upgrade images. Each compression component provides support for one (de)compression algorithm. At the time of writing, decompression of data compressed with the LZ4 and LZMA algorithms is supported, through the *GBL Compression (LZ4)* and *GBL Compression (LZMA)* components.

### Debug

This component provides the bootloader with support for debugging output. If the component is configured to enable debug prints, short debug messages will be printed over Serial Wire Output (SWO), which can be accessed in multiple ways, including using Simplicity Commander, and by connecting to port 4900 of the Wireless Starter Kit TCP/IP interface.

To turn on debug prints, enable the Debug component and select **Debug prints**. Select **Debug asserts** to enable assertions in the source code.

### GPIO Activation

This component provides functionality to enter firmware upgrade mode automatically after reset if a GPIO pin is active during boot. The GPIO pin location and polarity are configurable.

- GPIO: By enabling the GPIO activation component, the firmware upgrade mode can be activated by push buttons.

### Security

Security components provide implementations of cryptographic operations as well as functionality to compute checksums and to read cryptographic keys from manufacturing tokens.

Modules include:

- AES: AES decryption functionality

- CRC16: CRC16 functionality

- CRC32: CRC32 functionality

- ECDSA: ECDSA signature verification functionality

- SHA-256: SHA-256 digest functionality

### Storage

These components provide the bootloader with multiple storage options for SoCs. All storage implementations must provide an API to access image files to be upgraded. This API is based on the concept of dividing the download space into storage slots, where each slot has a predefined size and location in memory and can be used to store a single upgrade image. Some storage implementations also support a raw storage API to access the underlying storage medium. This can be used by applications to store other data in parts of the storage medium that are not used for storing firmware upgrade images. Implementations include:

- **External Flash**: The external flash storage implementation uses the external flash of the device for upgrade image storage. Note that this storage area is only a download space and is separate from the portion of external flash used to hold the active application code.
