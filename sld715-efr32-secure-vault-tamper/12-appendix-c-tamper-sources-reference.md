# Appendix C: Tamper Sources Reference

This section provides the tamper sources for devices other than EFR32xG21B.

## Tamper Sources on Other Series 2 HSE-SVH Devices

| **Type** | **Number** | **Name** | **Description** | **Default Level** |
|----------|------------|----------|-----------------|-------------------|
| SE Hardware | 0 | Reserved | — | — |
| " | 1 | Filter counter | Filter counter reached the configured threshold value | 0 (Ignore) |
| " | 2 | SE watchdog | Internal SE watchdog expired | 4 (Reset) |
| " | 3 | Reserved | — | — |
| " | 4 | SE RAM ECC2 | A 2-bit, non-correctable error in the SE RAM has occurred. | 4 (Reset) |
| " | 5 | SE hard fault | The SE core has encountered a hard fault exception indicating that an invalid memory access was attempted. | 4 (Reset) |
| " | 6 | Reserved | — | — |
| SE Software | 7 | SE software assertion | SE firmware has triggered an assertion, indicating that one of several sanity checks has failed and that normal operation cannot continue without a reset. | 4 (Reset) |
| " | 8 | SE secure boot | Secure boot of SE firmware failed | 4 (Reset) |
| " | 9 | User secure boot | Secure boot of host firmware failed | 0 (Ignore) |
| " | 10 | Mailbox authorization | Unauthorized command received over the Mailbox interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 11 | DCI authorization | Unauthorized command received over the DCI interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 12 | OTP read | OTP or flash content could not be properly authenticated | 4 (Reset) |
| " | 13 | Reserved | — | — |
| " | 14 | Self test | A check of the integrity of the SE's internal storage failed during boot up. | 4 (Reset) |
| " | 15 | TRNG monitor | The TRNG monitor performs a number of tests on the collected entropy data. If any of these tests fail, this tamper source is triggered. | 0 (Ignore) |
| Hardware | 16 | Secure lock | This tamper source indicates that the guarding mechanism (comparing the locks with their logical complement) of the debug port locks has failed. | 4 (Reset) |
| " | 17 | Digital glitch | Digital Glitch detector detected an event | 0 (Ignore) |
| " | 18 | Voltage glitch | Voltage Glitch Detector detected an event | 0 (Ignore) |
| " | 19 | SE ICACHE | The SE's instruction cache uses a checksum to verify the integrity of the data. This tamper source is triggered if the checksum is invalid. | 4 (Reset) |

| **Type** | **Number** | **Name** | **Description** | **Default Level** |
|----------|------------|----------|-----------------|-------------------|
| " | 20 | SE RAM ECC1 | SE RAM 1-bit ECC error occurred | 0 (Ignore) |
| " | 21 | BOD [1] | Brown-Out-Detector threshold alert | 4 (Reset) |
| " | 22 | Temperature sensor [1] | SE temperature is monitored to be within 5 degrees C of the limits for the device. If the limit is exceeded, this tamper source will be triggered. | 0 (Ignore) |
| " | 23 | DPLL fall | DPLL lock failed low | 0 (Ignore) |
| " | 24 | DPLL rise | DPLL lock failed high | 0 (Ignore) |
| " | 25 | PRS0 or ETAMPDET | PRS consumer for SE Tamper 25 or ETAMPDET asserted | 0 (Ignore) |
| " | 26 - 31 | PRS1 - 6 or PRS0 - 5 [1] | PRS consumer for SE Tamper 26 - 31 asserted | 0 (Ignore) |

**Notes**:

- [1] These tamper sources are available down to EM2. Other sources are available in EM1 and above.

- On EFR32xG23B and later devices, the default behavior is to detect tamper events only when the SE core is active. To detect tamper events when the SE is not performing operations, call `sl_se_enter_active_mode()`. This prevents the SE from sleeping and will result in higher current draw.

- In other HSE-SVH devices, tamper sources 25 to 31 are used for External Tamper Detect (ETAMPDET) if present and PRS consumers. Devices with ETAMPDET (e.g. EFR32xG25B) will have 6 PRS consumers (26 to 31) and devices without ETAMPDET will have 7 PRS consumers (25 to 31).

- The **ETAMPDET** source gets triggered when any of the ETAMPDET channels are asserted.

## Tamper Sources on SixG301 Devices

| **Type** | **Number** | **Name** | **Description** | **Default Level** |
|----------|------------|----------|-----------------|-------------------|
| SE Hardware | 0 | Reserved | — | — |
| " | 1 | Filter counter | Filter counter reached the configured threshold value | 0 (Ignore) |
| " | 2 | SE watchdog | Internal SE watchdog expired | 4 (Reset) |
| " | 3 | Crypto error | HostSymCrypto (1) | 4 (Reset) |
| " | 4 | SE RAM ECC2 | A 2-bit, non-correctable error in the SE RAM has occurred. | 4 (Reset) |
| " | 5 | Reserved | -- | - |
| " | 6 | SE major fault | SE major fault detected(2) | 4 (Reset) |
| SE Software | 7 | L2 ICACHE | L2 instruction cache error | 4 (Reset) |
| " | 8 | Reserved | -- | - |
| " | 9 | User secure boot | Secure boot of host firmware failed | 0 (Ignore) |
| " | 10 | Mailbox authorization | Unauthorized command received over the Mailbox interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 11 | DCI authorization | Unauthorized command received over the DCI interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 12 | SE Software Assert | SE software triggers an assert | 4 (Reset) |
| " | 13 | Reserved | — | — |
| " | 14 | Self test | A check of the integrity of the SE's internal storage failed during boot up. | 4 (Reset) |
| " | 15 | TRNG monitor | The TRNG monitor performs a number of tests on the collected entropy data. If any of these tests fail, this tamper source is triggered. | 0 (Ignore) |
| Hardware | 16 | Secure lock | This tamper source indicates that the guarding mechanism (comparing the locks with their logical complement) of the debug port locks has failed. | 4 (Reset) |
| " | 17 | Glitch Detector | Any tamper detection | 0 (Ignore) |
| " | 18 | OTP Alarm | OTP alarm triggered(3) | 4 (Reset) |
| " | 19 | SE ICACHE | The SE's instruction cache uses a checksum to verify the integrity of the data. This tamper source is triggered if the checksum is invalid. | 4 (Reset) |
| “ | 20 | SE RAM ECC 1 | SE RAM 1-bit ECC error | 0 (Ignore) |
|  | 21 | Brownout detect | Brown-out-detector threshold alert | 4 (Reset) |
|  | 22 | Temperature sensor | On-device temperature sensor | 0 (Ignore) |
|  | 23 | DPLL Lock | DPLL lock failure | 0 (Ignore) |
|  | 24 | SoC PLL | SoC PLL failure | 0 (Ignore) |
|  | 25 | eTamper detect | External tamper detect | 0 (Ignore) |
|  | 26 | KSU ECC 1 | KSU ECC 1-bit error | 0 (Ignore) |
|  | 27 | KSU ECC 2 | KSU ECC 2-bit error | 4 (Reset) |
|  | 28 | QSPI Reseed | QSPI reseed error | 4 (Reset) |
|  | 29 | PRS0 | PRS channel 0 asserted | 0 (Ignore) |
|  | 30 | PRS1 | PRS channel 1 asserted | 0 (Ignore) |
|  | 31 | PRS2 | PRS channel 2 asserted | 0 (Ignore) |

**Notes**:

1. HOSTSYMCRYPTO is the host cryptographic accelerator referred to as SYMCRYPTO in SixG301 reference manual.

2. The SE Major fault is triggered by any unrecoverable error in the SE code.

3. The OTP alarm is triggered if the OTP data cannot be read and verified.
