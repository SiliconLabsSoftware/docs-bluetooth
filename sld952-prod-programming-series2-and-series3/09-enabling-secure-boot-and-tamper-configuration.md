# Enabling Secure Boot and Tamper Configuration

The Secure Boot feature verifies the integrity and authenticity of the host application before allowing it to execute. Enabling this feature is **IRREVERSIBLE**, which means once enabled, Secure Boot can no longer be disabled throughout the life of the device. The Secure Boot settings are written to the one-time-programmable (OTP) memory. They cannot be changed once programmed.

On HSE-SVH and Series 3 Secure Vault devices, the anti-tamper configuration is provisioned with Secure Boot settings. The anti-tamper configuration determines the response from the HSE-SVH device if a tamper event occurs.

**Notes**:

- All tamper-related information in the following sections is only valid on HSE-SVH devices.

- For more information about anti-tamper configuration, see [Anti-Tamper Protection Configuration and Use](https://docs.silabs.com/iot-security/latest/efr32-secure-vault-tamper/).

- Except for the EFR32xG21B devices, other HSE-SVH devices require Simplicity Commander Version 1.12.2 or above for tamper configuration.

- Series 3 devices require Simplicity Commander Version 1.18.0, or higher

The `user_configuration.json` is a JSON file that contains the desired Secure Boot settings and anti-tamper configuration. Use the following command on the target device (e.g., EFR32MG21B010F1024) to generate a default configuration file.

```sh
commander security genconfig --nostore -o user_configuration.json --device EFR32MG21B010F1024
--serialno 440048205
```

```sh
DONE
```

> **Note**: The content of the JSON file is device dependent (`--device <device name>`).

The `security genconfig` command above generates a generic configuration file for **EFR32MG21B010F1024** consisting of the properties listed in the tables below. A text editor can be used to modify the default settings shown below to the desired configuration.

```sh
{
    "mcu_flags": { "SECURE_BOOT_ENABLE": true,
        "SECURE_BOOT_VERIFY_CERTIFICATE": false,
        "SECURE_BOOT_ANTI_ROLLBACK": true,
        "SECURE_BOOT_PAGE_LOCK_NARROW": false,
        "SECURE_BOOT_PAGE_LOCK_FULL": true
    },
    "tamper_levels": {
        "FILTER_COUNTER": 0,
        "WATCHDOG": 4,
        "SE_RAM_CRC": 4,
        "SE_HARDFAULT": 4,
        "SOFTWARE_ASSERTION": 4,
        "SE_CODE_AUTH": 4,
        "USER_CODE_AUTH": 0,
        "MAILBOX_AUTH": 0,
        "DCI_AUTH": 0,
        "OTP_READ": 4,
        "SELF_TEST": 4,
        "TRNG_MONITOR": 0,
        "PRS0": 0,
        "PRS1": 0,
        "PRS2": 0,
        "PRS3": 0,
        "PRS4": 0,
        "PRS5": 0,
        "PRS6": 0,
        "PRS7": 0,
        "DECOUPLE_BOD": 4,
        "TEMP_SENSOR": 0,
        "VGLITCH_FALLING": 0,
        "VGLITCH_RISING": 0,
        "SECURE_LOCK": 4,
        "SE_DEBUG": 0,
        "DGLITCH": 0,
        "SE_ICACHE": 4
    },
    "tamper_filter": {
        "FILTER_PERIOD": 0,
        "FILTER_THRESHOLD": 0,
        "RESET_THRESHOLD": 0
    },
    "tamper_flags": {
        "DGLITCH_ALWAYS_ON": false
    }
}

```

> **Note**: For `USER_CODE_AUTH` (user secure boot failed), recommended setting is 0 (Ignore) to avoid boot loops.

**Table: Secure Boot Items (mcu\_flags) for Series 2 Devices**

| **Name** | **Description** |
|----------|-----------------|
| SECURE_BOOT_ENABLE | If set, verifies the host image on the Cortex-M33 before releasing the Cortex-M33 from reset. |
| SECURE_BOOT_VERIFY_CERTIFICATE | If set, requires certificate-based signing of the host image. |
| SECURE_BOOT_ANTI_ROLLBACK | If set, prevents secure upgrading to a host image with a lower version than the image that is currently stored in flash. |
| SECURE_BOOT_PAGE_LOCK_NARROW | If set, locks flash pages that have been validated by the Secure Boot process to prevent re-flashing by other means than through the SE. Write/erase locks pages from 0 through the page where the Secure Boot host image signature is located, not including the last page if the signature is not on a page boundary. |
| SECURE_BOOT_PAGE_LOCK_FULL | If set, locks flash pages that have been validated by the Secure Boot process to prevent re-flashing by other means than through the SE. Write/erase locks pages from 0 through the page where the Secure Boot host image signature is located, including the last page if the signature is not on a page boundary. |

> **Note**: The host image is usually the Gecko Bootloader (GBL).

**Table: Secure Boot Items (mcu\_flags) for Series 3 Secure Vault Devices**

| **Name** | **Description** |
|----------|-----------------|
| SECURE_BOOT_ENABLE | If set, verifies the host image on the Cortex-M33 before releasing the Cortex-M33 from reset. |
| SECURE_BOOT_VERIFY_CERTIFICATE | If set, requires certificate-based signing of the host image. |
| SECURE_BOOT_ANTI_ROLLBACK | If set, prevents secure upgrading to a host image with a lower version than the image that is currently stored in flash. |

## Table: Tamper Items for HSE-SVH and Series 3 Secure Vault Devices

| **Name** | **Description** |
|----------|-----------------|
| tamper_levels | The tamper levels of different tamper sources. |
| tamper_filter | The settings for tamper filters. |
| tamper_flags | The settings for tamper flags. |

The following command writes the Secure Boot settings and anti-tamper configuration in `user_configuration.json` file to the device. This command can be executed only once per device.

```sh
commander security writeconfig --configfile user_configuration.json --device EFR32MG21B010F1024
--serialno 440048205
```

```sh
================================================================================

THIS IS A ONE-TIME configuration: Please inspect file before confirming: user_configuration.json
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
DONE
```

To check the device's Secure Boot settings and anti-tamper configuration, run the `security readconfig` command.

```sh
commander security readconfig --serialno 440048205
```

```sh
MCU Flags
Secure Boot                    : Enabled
Secure Boot Verify Certificate : Disabled
Secure Boot Anti Rollback	   : Enabled
Secure Boot Page Lock Narrow   : Disabled
Secure Boot Page Lock Full	   : Enabled

Tamper Levels
FILTER_COUNTER      : 1
WATCHDOG            : 4
SE_RAM_CRC          : 4
SE_HARDFAULT        : 4
SOFTWARE_ASSERTION  : 4
SE_CODE_AUTH	    : 4
USER_CODE_AUTH	    : 0
MAILBOX_AUTH	    : 1
DCI_AUTH	        : 0
OTP_READ	        : 4
SELF_TEST	        : 4
TRNG_MONITOR	    : 1
PRS0	            : 1
PRS1	            : 1
PRS2	            : 2
PRS3	            : 2
PRS4	            : 4
PRS5	            : 4
PRS6	            : 7
PRS7	            : 7
DECOUPLE_BOD	    : 4
TEMP_SENSOR	        : 2
VGLITCH_FALLING	    : 2
VGLITCH_RISING	    : 2
SECURE_LOCK	        : 4
SE_DEBUG	        : 0
DGLITCH	            : 2
SE_ICACHE	        : 4

Tamper Filter
Filter Period    : 10
Filter Threshold : 6
Reset Threshold  : 5

Tamper Flags
Digital Glitch Detector Always On: Disabled
DONE
```
