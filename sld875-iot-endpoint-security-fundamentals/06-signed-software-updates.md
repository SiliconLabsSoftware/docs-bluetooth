# Signed Software Updates

*The product shall only support signed software updates. While it is critical that all products be updatable, it is just as critical that these update images be secured. A manufacturer must cryptographically sign update images to prevent tampering during deployment. The product must not use unsigned updates, as they could be fraudulent.*

Silicon Labs development tools support building signed upgrade images and securely updating devices in the field, through the Silicon Labs Gecko Bootloader. The Gecko Bootloader can be configured to perform a variety of functions, from device initialization to firmware upgrades. Key features of the bootloader are:

- Usable across Silicon Labs Gecko microcontroller and wireless microcontroller families

- In-field upgradeable

- Configurable

- Enhanced security features, including:

- Secure Boot: When Secure Boot is enabled, the bootloader enforces cryptographic signature verification of the application image on every boot, using asymmetric cryptography. This ensures that the application was created and signed by a trusted party.

- Signed upgrade image file: The Gecko Bootloader supports enforcing cryptographic signature verification of the upgrade image file. This allows the bootloader and application to verify that the application or bootloader upgrade comes from a trusted source before starting the upgrade process, ensuring that the image file was created and signed by a trusted party.

- Encrypted upgrade image file: The image file can also be encrypted to prevent eavesdroppers from acquiring the plaintext firmware image.

On Series 1 devices, the Gecko Bootloader has a two-stage design, first stage and main stage, where a minimal first stage bootloader is used to upgrade the main bootloader. The first stage bootloader only contains functionality to read from and write to fixed addresses in internal flash. To perform a main bootloader upgrade, the running main bootloader verifies the integrity and authenticity of the bootloader upgrade image file. The running main bootloader then writes the upgrade image to a fixed location in internal flash and issues a reboot into the first stage bootloader. The first stage bootloader verifies the integrity of the main bootloader firmware upgrade image, by computing a CRC32 checksum before copying the upgrade image to the main bootloader location.

On Series 2 and Series 3 devices, the Gecko Bootloader consists only of the first stage bootloader. The main bootloader is upgradable through the hardware peripheral Secure Engine. The Secure Engine provides functionality to install an image to the base address of flash, by copying from a configurable location in flash. To perform a main bootloader upgrade, the running main bootloader verifies the integrity and authenticity of the bootloader upgrade image file. The running main bootloader then writes the upgrade image to the upgrade location in flash and requests that the Secure Engine install it. The Secure Engine is also capable of verifying the authenticity of the main bootloader update image against a root of trust. The Secure Engine itself is upgradable using the same mechanism.

In summary, Series 2 and Series 3 devices support a hardware root of trust and a Secure Boot process that verifies the authenticity and integrity of Gecko Bootloader, whereas in Series 1 devices, the authenticity and integrity of Gecko Bootloader are assumed trusted and are not explicitly checked.

The Gecko Bootloader can enforce application image security on two levels:

- Secure Boot refers to the verification of the authenticity of the application image in main flash on every boot of the device. When Secure Boot is enabled, the cryptographic signature of the application image in flash is verified on every boot, before the application is allowed to run. Secure Boot is not enabled by default in the example configurations provided by Silicon Labs, but enabling it is highly recommended to ensure the validity and integrity of firmware images.

- Secure Firmware Upgrade refers to the verification of the authenticity of an upgrade image before applying the upgrade and optionally enforcing that upgrade images are encrypted. The Secure Firmware Upgrade process uses symmetric encryption to encrypt the upgrade image, and asymmetric cryptography to sign the upgrade image in order to ensure its integrity and authenticity.

For more information on Silicon Labs’ support for software update security, refer to the following:

- Bootloaders in general: [Bootloader Fundamentals](https://docs.silabs.com/mcu-bootloader/latest/bootloader-fundamentals/).

- The Gecko Bootloader in general: [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) or *UG266: Silicon Labs Gecko Bootloader User’s Guide for GSDK 3.2 and Lower*

Using the Gecko Bootloader with specific protocols:

[Using the Gecko Bootloader with EmberZNet](https://docs.silabs.com/zigbee/latest/using-gecko-bootloader-with-zigbee/)

[Using the Gecko Bootloader with Silicon Labs Connect](https://docs.silabs.com/connect-stack/latest/using-gecko-bootloader-with-connect/)

[Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](https://docs.silabs.com/bluetooth/latest/using-gecko-bootloader-with-bluetooth-apps/)

[Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/)
