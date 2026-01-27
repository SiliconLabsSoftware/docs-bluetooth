# Debug Lock

## Overview

The debug access port connected to the Series 2 and Series 3 device's Cortex-M33 processor can be closed by issuing commands to the SE, either from a debugger over DCI or through the mailbox interface. Three properties govern the behavior of the debug lock.

| **Property** | **Description If Set** | **Default Value** |
|--------------|------------------------|-------------------|
| Debug Lock | The debug port is kept locked on boot. | Disabled |
| Device Erase | The Erase Device command is available. | Enabled |
| Secure Debug | Secure debug unlock is available. | Disabled |

The following sections describe how to interact with these properties and how to enable debug locks using the SE command interface over DCI. The status of the debug lock can be inspected using the [Read Lock Status](#debug-lock-command-reference) command.

## Standard Debug Unlock

The device is in standard debug unlock state if the debug lock properties are set to the default values.

| **Debug Lock** | **Device Erase** | **Secure Debug** | **Description** |
|----------------|------------------|------------------|-----------------|
| Disabled | Enabled | Disabled | All debug operations are allowed. |

## Standard Debug Lock

With the default properties in the table above, the device can be locked using the [Apply Lock](#debug-lock-command-reference) command. The typical flow for this configuration is simply to issue the Apply Lock command after the device has been programmed, either using a DCI command from the programming debugger or through the mailbox interface.

| **Debug Lock** | **Device Erase** | **Secure Debug** | **Description** |
|----------------|------------------|------------------|-----------------|
| Enabled | Enabled | Disabled | The Erase Device command will wipe the main flash and RAM, and then a reset will yield an unlocked device. |

The standard debug lock disables debug access port, but issuing a device erase wipes the device and enables the debug port again.

## Secure Debug Lock

For secure debug lock, the debug interface can temporarily be unlocked by answering a challenge, if the Secure debug property is enabled before locking.

| **Debug Lock** | **Device Erase** | **Secure Debug** | **Description** |
|----------------|------------------|------------------|-----------------|
| Enabled | Disabled (2) | Enabled (1) | Secure debug lock is enabled, which makes it possible to securely open the debug lock temporarily to reprogram or debug a locked device. |

**Notes**:

1. Secure debug lock is enabled in two steps before the debug lock is enabled:

    1. Install the Public Command Key using Simplicity Studio, Simplicity Commander, or directly through the SE Manager API.

    2. Enable secure debug using Simplicity Studio, Simplicity Commander, or directly through the SE Manager APIs.

2. Disable the device erase using Simplicity Studio or Simplicity Commander or directly through the SE Manager API. This is an **Irreversible** action and should be disabled **After** the secure debug is enabled.

3. The **Device Erase** option should **Disabled** only during production. If this setting is applied during development, it prevents the device from being re-flashed with new firmware using a debugger and significantly hinders debugging efforts.

## Secure Debug Unlock

To enable debugging or to reprogram the device, the debug port is temporarily opened. It automatically closes upon a power-on reset or device reset, and the device returns to the Secure Debug Lock state.

| **Debug Lock** | **Device Erase** | **Secure Debug** | **Description** |
|----------------|------------------|------------------|-----------------|
| Enabled | Disabled | Enabled | Device debug port remains unlocked, till power on rest or device rest is performed. |

## Permanent Debug Lock

The device can enter into **Permanent Debug Lock** state when both the **Device Erase** and **Secure Debug** properties are disabled.

| **Debug Lock** | **Device Erase** | **Secure Debug** | **Description** |
|----------------|------------------|------------------|-----------------|
| Enabled | Disabled | Disabled | The part cannot be unlocked. Devices with Permanent Debug Lock engaged cannot be returned for failure analysis. |

## Debug Lock State Transition

The following figure describes the transitions between different debug lock states.

![Debug Lock State Transition](./resources/sld714-debug-lock-state-transition.png)

1. Standard debug unlock can transit to **Standard Debug Lock** state, by enabling '**Debug Lock**' flag.

2. Standard debug lock can revert to **Standard debug unlock** via an erasedevice/device unlock command (erases the main flash and RAM). After the device is reset, the debug port remains unlocked until it is explicitly locked again.

3. Device can transit to **Secure Debug lock** state by enabling Secure Debug property.

4. Secure debug lock can use Debug Unlock Token to temporary transit to secure debug unlock, which does not erase the main flash and RAM but enables debug operations. The device reverts to the secure debug lock through a power on or pin reset.

5,6,7. From any state, device can transit to Permanent Debug lock by Disabling **Device erase** and **Secure debug**. This is terminal state and can not transit to any other state. This should be done only after development is completed.

> **Note**: Device can be brought back to "**Standard Debug unlock**" state from "**Secure Debug Lock**" state by erasedevice command, if **Device Erase** is Enabled. So it strongly recommended to customers to Disable "**Device erase**" during production.

## Debug Lock Command Reference

The commands for debug lock are described in the following table.

| **DCI Command (1)** | **Mailbox (SE Manager) API (2)** | **Description** | **Availability** |
|---------------------|----------------------------------|-----------------|------------------|
| Apply Lock | sl_se_apply_debug_lock | Enables the debug lock for the part. | While debug is unlocked. |
| Read Lock Status | sl_se_get_debug_lock_status | Returns the current debug lock status and con-figuration. | Always. |
| Disable Device Erase | sl_se_disable_device_erase | Disables the Erase Device command. This command does not lock the debug interface to the part, but it is an IRREVERSIBLE action for the part. | Always. |
| Disable Secure Debug | sl_se_disable_secure_debug | Disables the secure debug functionality that can be used to open a locked debug port. | While secure debug is enabled. |
| Enable Secure Debug | sl_se_enable_secure_debug | Enables the secure debug functionality that can be used to open a locked debug port. | While debug is unlocked and Public Command Key is provisioned. |
| Set debug options | sl_se_set_debug_options | Configures the TrustZone access permissions of the debug interface. (3) | While debug is unlocked. |
| Init Pub Key | sl_se_init_otp_key | Used to provision a single public key during device initialization. The public key cannot be changed once written, and the command will be unavailable for that key. | Available once for each key. |
| Read Pub Key | sl_se_read_pubkey | Reads the stored public key. | Always. |
| Get Challenge | sl_se_roll_challenge | Used to roll the current challenge value (16 bytes) to revoke secure debug access. (4) | While Public Command Key is uploaded. |

**Notes**:

1. Performing these commands over DCI is implemented in Simplicity Studio and Simplicity Commander.

2. The `sl_se_apply_debug_lock`, `sl_se_get_debug_lock_status`, `sl_se_init_otp_key`, and `sl_se_read_pubkey` are available on all Series 2 and Series 3 devices. Other APIs are only available on HSE and Series 3 devices. The SE Manager API document can be found at [https://docs.silabs.com/gecko-platform/latest/platform-security-api/sl-se-manager](https://docs.silabs.com/gecko-platform/latest/platform-security-api/sl-se-manager).

3. For more information about debug options, see [TrustZone Debug Authentication](./05-debug-unlock.md#trustzone-debug-authentication).

4. A new challenge will only be generated if the current one has been successfully used at least once. On Series 2 devices, there is no limitation on rolling a challenge, whereas on Series 3 devices, challenge can be rolled only 128 times.
