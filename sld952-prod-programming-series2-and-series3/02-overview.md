# Overview

More steps are involved in the production programming process of Series 2 and Series 3 devices compared to Series 1 devices. The steps vary if the device is to have Secure Boot enabled or disabled. For more information about Secure Boot, see [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/). Enabling Secure Debug is a recommended step in the process. For more information about Secure Debug, see [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

A general overview of the production programming steps is described in the following sections.

Silicon Labs provides [Custom Part Manufacturing Service (CPMS)](https://docs.silabs.com/iot-security/latest/iot-security-cpms/) to customize the users' security features and settings.

## Production Programming for Secure Boot-Disabled Device

The following figure illustrates the production programming flow for Secure Boot-disabled devices. It is possible to upgrade Series 2 and Series 3 devices deployed in the field without Secure Boot to Secure Boot with RTSL.

![Series 2 and Series 3 High-Level Production Programming Flowchart for Secure Boot-Disabled Devices](./resources/sld952-prod-program-flowchart.png)

Upgrading the SE Firmware and flashing the bootloader and application firmware are required in the production programming process. Provisioning the GBL Decryption Key for GBL payload decryption, Public Sign Key for Secure Boot, Public Command Key for Secure Debug Unlock, and enabling the Debug Lock are strongly recommended.

A more detailed version of the Series 2 and Series 3 production programming flowchart for a Secure Boot-disabled device is illustrated in the following figure.

![Series 2 and Series 3 Step-by-Step Production Programming Flowchart for a Secure Boot-Disabled Device](./resources/sld952-prod-program-stepbystep-flowchart.png)

**Notes**:

1. Refer to [Provisioning the GBL Decryption Key in Simplicity Commander](./07-key-provisioning.md#provisioning-the-gbl-decryption-key-in-simplicity-commander) on how to program the GBL Decryption Key to the Series 2 or Series 3 device.

2. The VSE devices store a Public Sign Key copy on the top page of the main flash for Secure Boot (see *Signing for ECDSA-P256-SHA256 Secure Boot* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/)).

3. The Public Command Key can also be used to temporarily disable anti-tamper protection on HSE-SVH devices (see [Anti-Tamper Protection Configuration and Use](https://docs.silabs.com/iot-security/latest/efr32-secure-vault-tamper/)).

4. Enabling the debug lock should be the final step in production, and the following debug lock options are available on Series 2 and Series 3 devices.

   - [Standard Debug Lock](./10-enabling-debug-lock.md#standard-debug-lock)

   - [Secure Debug Lock](./10-enabling-debug-lock.md#secure-debug-lock) (Public Command Key was provisioned)

   - [Permanent Debug Lock](./10-enabling-debug-lock.md#permanent-debug-lock)

   For more information about these debug lock options, see the section *Debug Lock State Transition* in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

5. For Series 3, the final  step is to close all code regions. See [_Bootloader Firmware Programming](./05-bootloader-firmware-programming.md) and [Application Firmware Programming](./06-application-firmware-programming.md) for instructions.

## Production Programming for Secure Boot-Enabled Device

The following figure illustrates the production programming flow for Secure Boot-enabled devices.

![Series 2 and Series 3 High-Level Production Programming Flowchart for Secure Boot-Enabled Devices](./resources/sld952-high-level-prod-program-flow-secureboot-enabled.png)

Upgrading the SE Firmware and flashing the **SIGNED** bootloader and application firmware are required in the production programming process. Provisioning the Public Sign Key and enabling Secure Boot are also needed in the production programming process to enable the Secure Boot option. Provisioning the GBL Decryption Key for GBL payload decryption, Public Command Key for Secure Debug Unlock, and enabling the Debug Lock are strongly recommended. Provisioning Tamper Configuration (HSE-SVH and Series 3 Secure Vault only) is also recommended.

A more detailed version of the Series 2 and Series 3 production programming flowchart for a Secure Boot-enabled device is illustrated in the following figure.

![Series 2 and Series 3 High-Level Production Programming Flowchart for Secure Boot-Enabled Devices](./resources/sld952-stepbystep-prod-program-flow-secureboot-enabled.png)

**Notes**:

1. The device will enter the Secure Boot failed state if the bootloader firmware is either unsigned or incorrectly signed (see [Bootloader Firmware Programming](./05-bootloader-firmware-programming.md)).

2. If the Secure Boot option is enabled in the bootloader, the application firmware must be signed (see [Application Firmware Programming](./06-application-firmware-programming.md)).

3. The VSE devices store a Public Sign Key copy on the top page of the main flash for Secure Boot (see section *Signing for ECDSA-P256-SHA256 Secure Boot* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/)).

4. On HSE-SVH and Series 3 Secure Vault devices, the anti-tamper protection configuration is provisioned with Secure Boot settings (see [Enabling Secure Boot and Tamper Configuration](./09-enabling-secure-boot-and-tamper-configuration.md)).

5. Refer to [Provisioning the GBL Decryption Key in Simplicity Commander](./07-key-provisioning.md#provisioning-the-gbl-decryption-key-in-simplicity-commander) on how to program the GBL Decryption Key to Series 2 or Series 3 devices.

6. The Public Command Key can also be used to temporarily disable anti-tamper protection on HSE-SVH devices (see [Anti-Tamper Protection Configuration and Use](https://docs.silabs.com/iot-security/latest/efr32-secure-vault-tamper/)).

7. Enabling the debug lock should be the final step in production, and the following debug lock options are available on the Series 2 device.

   - [Standard Debug Lock](./10-enabling-debug-lock.md#standard-debug-lock)

   - [Permanent Debug Lock](./10-enabling-debug-lock.md#permanent-debug-lock)

   - [Secure Debug Lock](./10-enabling-debug-lock.md#secure-debug-lock) (Public Command Key was provisioned)

   For more information about these debug lock options, see the section *Debug Lock State Transition* in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

8. For Series 3, the final step is to close all code regions. See [Bootloader Firmware Programming](./05-bootloader-firmware-programming.md) and [Application Firmware Programming](./06-application-firmware-programming.md) for instructions.
