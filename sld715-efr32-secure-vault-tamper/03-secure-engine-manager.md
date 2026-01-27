# Secure Engine Manager

The Secure Engine Manager provides thread-safe APIs for the SE's mailbox interface. The SE Manager APIs related to tamper operations are listed in the following table.

For the SE's mailbox interface, see *Secure Engine Subsystem* in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

| **SE Manager API** | **Usage** |
|--------------------|-----------|
| sl_se_init_otp | Initialize SE OTP configuration (including tamper configuration on HSE-SVH and Series 3 Secure Vault devices). |
| sl_se_read_otp | Read SE OTP configuration (including tamper configuration on HSE-SVH and Series 3 Secure Vault devices). |
| sl_se_init_otp_key | Used during device initialization to upload the Public Command Key. |
| sl_se_read_pubkey | Read the stored Public Command Key. |
| sl_se_get_serialnumber | Read out the serial number (16 bytes) of the HSE device. |
| sl_se_get_challenge | Read out the current challenge value (16 bytes) for tamper disable. |
| sl_se_roll_challenge | Used to roll the current challenge value (16 bytes) to invalidate the Disable Tamper Token. |
| sl_se_disable_tamper | Temporarily disable tamper configuration using the Disable Tamper Token. |
| sl_se_get_status | Read the current HSE status (including recorded tamper status on HSE-SVH devices). |
| sl_se_get_reset_cause | Read the EMU-\>RSTCAUSE register from HSE devices after a tamper reset. |
| sl_se_get_tamper_reset_cause | Read the cached value of the EMU-\>TAMPERRSTCAUSE register after a tamper reset. |
| sl_se_enter_active_mode | Force the SE to remain active to enable the detection of glitch tamper events on the host Cortex-M33 core (see fourth note below) |
| sl_se_exit_active_mode | Exit active mode and allow the SE to sleep when not performing operations. This will prevent the detection of glitch tamper events when the SE is sleeping. This API should only be used if active mode was entered by calling `sl_se_enter_active_mode`. If active mode is set through a DCI command, it can only be disabled through a DCI command. (see fourth note below) |

**Notes**:

- The `sl_se_get_reset_cause` is only available on EFR32xG21B devices. The `EMU->RSTCAUSE` register can be directly read on other HSE-SVH and Series 3 Secure Vault devices.

- The `sl_se_get_tamper_reset_cause` is unavailable on EFR32xG21B devices, and SE firmware ≥ v2.2.1 is required.

- The SE Manager API document can be found at [https://docs.silabs.com/gecko-platform/latest/platform-security-api/sl-se-manager](https://docs.silabs.com/gecko-platform/latest/platform-security-api/sl-se-manager).

- Does not apply to EFR32MG21B parts.
