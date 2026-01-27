# About the Gecko Bootloader

The Silicon Labs Gecko Bootloader is a configurable code library that can be used with all the newer Silicon Labs Gecko MCUs and wireless MCUs. It uses a specially-formatted update image file called a GBL file. The Gecko Bootloader has a two-stage design on Series 1 devices, where a minimal first stage bootloader is used to update the main bootloader. On Series 2 devices, the first stage bootloader is replaced by a Secure Engine and the Gecko Bootloader consists only of the main bootloader. The Secure Engine may be hardware-based, or virtual (software). If hardware-based, the implementation may be either with or without Secure Vault. Throughout this document, the following conventions will be used.

- **HSE**: Hardware Secure Engine, either with or without Secure Vault if not specified

- **VSE**: Virtual Secure Engine

- **SE**: Secure Engine (either HSE or VSE, in general)

Having a first stage bootloader or SE allows for field updates of the main bootloader, including adding new capabilities, changing communication protocols, adding new security features and fixes, and so on. The Gecko Bootloader consists of three component parts:

**Core:** The bootloader core contains the main function of both bootloader stages. It also contains functionality to write to the internal main flash, to perform a bootloader update, and to reset into the application flagging applicable reset reasons.

**Driver:** Different bootloading applications require different hardware drivers for use by the other components of the bootloader.

**Component/Plugin:** All parts of the main bootloader that are either optional or selectable for different configurations are implemented as components (in GSDK 4.0 and higher) or previously in plugins. Each component/plugin has a generic header file, and one or more implementations. The current release contains components for functionality like UART and SPI communication protocols, SPI flash storage, internal flash storage, and different cryptographic operations.

## Features

Gecko Bootloader features include:

- Field-updatable

- Secure boot

- Signed GBL firmware update image file

- Encrypted GBL firmware update image file

These features are summarized in the following sections and described in more detail in [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) or [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-series3-and-higher/). Protocol-specific information about using the Gecko Bootloader may be found in the following documents:

- [Using the Gecko Bootloader with EmberZNet](https://docs.silabs.com/zigbee/latest/using-gecko-bootloader-with-zigbee/)

- [Bootloading and OTA with Silicon Labs Connect SDK v3.x](https://docs.silabs.com/connect-stack/latest/bootloading-and-ota-with-connect-v3x)

- [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](https://docs.silabs.com/bluetooth/latest/using-gecko-bootloader-with-bluetooth-apps/)

### Field-Updatable

#### Series 1

On EFM32 and EFR32 Series 1 devices, field update capability for the Gecko bootloader is provided by a two-stage design where the bootloader has a first stage and a main stage. The minimal first stage of the bootloader is not field updatable, and can only update the main bootloader by reading from and writing to fixed addresses in internal flash memory. To perform a main bootloader update, the running main bootloader verifies the integrity and authenticity of the bootloader update image, writes it to internal flash, and issues a reboot into the first stage bootloader. The first stage bootloader verifies the integrity of the main bootloader update image before copying it to the main bootloader location, completing the update

#### Series 2

On Series 2 devices, field update capability for the Gecko bootloader is provided by the SE. To perform a main bootloader update, the running main bootloader verifies the integrity and authenticity of the bootloader update image, writes it to internal flash, and requests that the SE installs the update. The SE optionally verifies the authenticity of the main bootloader update image before copying it to the main bootloader location, completing the update. The same mechanism can be used to update the SE itself.

### Secure Boot

Secure boot is designed to prevent an untrusted application from running on the device. When Secure Boot is enabled, the bootloader enforces cryptographic signature verification of the application image on every boot using asymmetric cryptography. The signature algorithm used is ECDSA-P256-SHA256. The public key is written to the device during manufacturing, while the private key is kept secret. This ensures that the application was created and signed by a trusted party.

### Signed GBL Update Image File

The Gecko Bootloader supports enforcing cryptographic signature verification of the update image file in addition to Secure Boot. This allows the bootloader and application to verify that the application or bootloader update comes from a trusted source before starting the update process. The signature algorithm used is ECDSA-P256-SHA256. The public key is the same key as for secure boot, written to the device during manufacturing, while the private key is never distributed. This ensures that the GBL file was created and signed by a trusted party.

### Encrypted GBL Update File

The GBL update file can also be encrypted, to prevent eavesdroppers from getting hold of the plaintext firmware image. The encryption algorithm used is AES-CTR-128, and the encryption key is written to the device during manufacturing.
