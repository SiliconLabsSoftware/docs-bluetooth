# Tamper Responses

A [tamper source](./05-tamper-sources.md) can lead to a series of different autonomous responses from the HSE. These responses are listed in the following table.

| **Level (1)** | **Response (2)** | **Description** |
|---------------|------------------|-----------------|
| 0 | Ignore | No action is taken |
| 1 | Interrupt | Triggers the SETAMPERHOST interrupt on the host |
| 2 | Filter | Increases a counter in the tamper filter |
| 4 | Reset | Resets the device |
| 7 | Erase OTP | Erases the device's OTP configuration |

**Notes**:

1. Levels 3, 5, and 6 are reserved.

2. These responses are cumulative:

   - If a filter response is triggered, it will also trigger an interrupt.

   - If a reset response is triggered, it will supersede the interrupt. The filter counter and interrupt flag are cleared at reset.

   - If an erase OTP response is triggered, it will erase the OTP and reset the device. The device will fail to boot and become unusable.

## Interrupt

If a tamper source is configured to respond with the interrupt response or higher (≥ level 1), the `SETAMPERHOST` interrupt line to the host Cortex-M33 will be pulsed and make the NVIC trigger the corresponding interrupt handler (`SETAMPERHOST_IRQHandler`).

After the interrupt has been handled, the tamper status can be found by reading the HSE status (using `sl_se_get_status` in the [SE Manager](./03-secure-engine-manager.md)), which contains a list of all the tamper sources that have been triggered since the last time the status was read. Reading HSE status clears the registered tamper sources.

> **Note**: Enabling the `SEMAILBOXHOST` clock for the tamper source is required to trigger the `SETAMPERHOST` interrupt in most HSE-SVH and Series 3 Secure Vault devices. EFR32xG21B does not require this.

## Filter

The HSE has a filter to debounce spurious tamper events. The filter has a counter that is periodically reset. If a tamper source is configured to the filter response (level 2), when it is triggered, the counter is increased. If the counter value reaches a configurable threshold, the `Filter counter` tamper source is triggered, which can configure to lead to any other responses (1, 4, or even 7).

Only a single shared filter counter is available, so the cumulative triggering of all tamper sources configured to the filter level will increase the same counter. The filter can be programmed to use one of the trigger thresholds and reset periods provided below. The filter counter is zero upon a tamper or normal reset.

## Filter Trigger Threshold

- Value (n): 0 to 7

- Filter Trigger Threshold: 256/$2^n$ (256 to 2)

## Filter Reset Period

- Value (n): 0 to 31

- Filter Reset Period: 32 ms \* $2^n$ (32 ms to ~795.4 days)

## Example Filter Configuration

For example, consider a device with a Filter Trigger Threshold of 3 and Filter Reset Period of 5. If that device detects 32 (256/$2^3$) Filter response events in 1.024 seconds (32 ms \* $2^5$), the `Filter counter` tamper source will trigger.

## Reset

The reset response resets the HSE and Cortex-M33. After a tamper reset, the last reset cause can be directly read from `EMU-\>RSTCAUSE` register or using `sl_se_get_reset_cause` in the [SE Manager](./03-secure-engine-manager.md). In cases where the reset was caused by a tamper response, the source of the tamper can be determined by calling `sl_se_get_tamper_reset_cause` in the SE Manager. (Note that this API is not available for EFR32xG21B-based parts). See [Tamper Sources](./05-tamper-sources.md) for more information. Tamper reset occurs when the HSE sends a request to the Cortex-M33’s EMU, which issues a hard reset.

If a tamper reset is triggered during boot, this can lead to a boot loop. To debug such a scenario, the HSE has a tamper reset counter and enters diagnostic mode if the counter reaches a programmable threshold. Users can issue a non-tamper reset to clear the tamper reset counter before the programmable threshold is reached.

In diagnostic mode, the Cortex-M33 is held in reset and only DCI commands are available. The device will remain in diagnostic mode until a power-on or pin reset occurs.

For more information on the SE's DCI, see [Secure Engine Subsystem](https://docs.silabs.com/iot-security/latest/series2-secure-debug/03-secure-engine-subsystem/) in *Series 2 and Series 3 Secure Debug*.

## Erase OTP

The Erase OTP response is the strongest reaction the HSE can take, and it will make the device and all wrapped secrets unrecoverable. After this response, the device will no longer be able to boot or connect to a debugger.

This response should typically only be used in situations where the device believes that it is under an actual attack, for instance through the detection of several voltage or digital glitches in a short time window.
