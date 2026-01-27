# Gecko Bootloader and TrustZone

With GSDK v4.2.2, Gecko Bootloader comes with TrustZone support to build TrustZone aware solutions. This section outlines the major points that should be considered when using a TrustZone aware bootloader.

## Gecko Bootloader Operation

A TrustZone aware application can have the following configurations:

- Secure app only

- Non-secure app only

- Secure and Non-secure pair

With TrustZone enabled, the application bootloaders are entirely configured as Secure applications whereas the communication bootloaders are split into Secure and Non-secure application pairs. The communication interfaces are typically in the Non-secure part of the application and the GBL parser, and other functionalities are in the Secure part of the application. TrustZone aware applications are generated and built using pre-defined workspace applications. Refer to [Series 2 TrustZone](https://docs.silabs.com/mcu-bootloader/latest/series2-trustzone/) and [Software Project Generation and Configuration with SLC-CLI](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-tools-slc-cli/) for more details regarding TrustZone workspaces and the SLC CLI tool.

For the communication bootloaders to function properly, a combined bootloader binary image (containing both the Secure and Non-secure binary images) needs to be programmed onto the device. A combined bootloader binary can be obtained by either using the post-build steps defined for the workspace or using the Commander tool. Refer to [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/) for more details on how to use the Commander tool to combine multiple binaries into a single binary. Application bootloaders need to be built with the **Bootloader TrustZone Secure** component when building a TrustZone aware solution. The **Bootloader TrustZone Secure** component is required to set up the necessary configuration for the application to function correctly within a TrustZone aware solution.

To create a bootloader upgrade file, the combined bootloader binary (Secure and Non-secure) should be used. Refer to [Series 2 TrustZone](https://docs.silabs.com/mcu-bootloader/latest/series2-trustzone/) for more details on upgrading an existing application to TrustZone. Special care must be taken while building the TrustZone aware communication bootloaders for the xG21 family as the bootloader size increases from 16kB (non TrustZone aware solution) to 24kB (TrustZone aware solution). Similarly, the Bootloader – SoC Bluetooth Apploader OTA DFU application’s size increases from 72kB to 80kB with TrustZone enabled.

## Gecko Bootloader Configuration

The following table provides three of the configuration options available as part of the Bootloader Interface with TrustZone enabled.

<table class="classic"><colgroup><col width="50%"/><col width="50%"/></colgroup><thead><tr><th><p>Configuration Option</p></th><th><p>Description</p></th></tr></thead><tbody><tr><td><p>BOOTLOADER_DISABLE_OLD_BOOTLOADER_MITIGATION</p></td><td><p>Disables multi-tiered fallback logic. The fault handling logic as well as the USART auto-detection logic will be disabled. Default value is set to 0.</p></td></tr><tr><td><p>BOOTLOADER_DISABLE_NVM3_FAULT_HANDLING</p></td><td><p>Disables peripheral access fault handling. The fault handling triggered by an erroneous access of peripherals will be disabled. This should be disabled if all the peripherals that are in use by the bootloader have been properly configured by the Manually override security state of peripherals option. Default value is set to 0.</p></td></tr><tr><td><p>BOOTLOADER_MANUAL_OVERRIDE_SECURITY_STATE</p></td><td><p>Manually override the security state of peripherals in use by the bootloader. This option can be used to manually set a peripheral's access state before calling into the bootloader. Default value is set to 0.</p></td></tr></tbody></table>
