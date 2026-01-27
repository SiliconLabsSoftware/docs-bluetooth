# Tamper Sources

The following table lists the available tamper sources and the default level on the EFR32xG21B. Refer to Appendix C for the other HSE-SVH and Series 3 devices. The tamper sources with the default level higher than 0 (Ignore) are always in effect even if the user does not initialize the tamper configuration in HSE OTP. Users can keep or escalate the default tamper responses (≥ 0 for Ignore and ≥ 4 for Reset) of any sources when initially configuring the part.

| **Type** | **Number** | **Name** | **Description** | **Default Level** |
|----------|------------|----------|-----------------|-------------------|
| SE Hardware | 0 | Reserved | — | — |
| " | 1 | Filter counter | Filter counter reached the configured threshold value | 0 (Ignore) |
| " | 2 | SE watchdog | Internal SE watchdog expired | 4 (Reset) |
| " | 3 | Reserved | — | — |
| " | 4 | SE RAM CRC | A 2-bit, non-correctable error in the SE RAM has occurred. | 4 (Reset) |
| " | 5 | SE hard fault | The SE core has encountered a hard fault exception indicating that an invalid memory access was attempted. | 4 (Reset) |
| " | 6 | Reserved | — | — |
| SE Software | 7 | SE software assertion | SE firmware has triggered an assertion, indicating that one of several sanity checks has failed and that normal operation cannot continue without a reset. | 4 (Reset) |
| " | 8 | SE secure boot | Secure boot of SE firmware failed | 4 (Reset) |
| " | 9 | User secure boot | Secure boot of host firmware failed | 0 (Ignore) |
| " | 10 | Mailbox authorization | Unauthorized command received over the Mailbox interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 11 | DCI authorization | Unauthorized command received over the DCI interface. This can be triggered by either (1) an incorrectly signed debug unlock or tamper disable token or (2) attempting to export a non-exportable key. | 0 (Ignore) |
| " | 12 | OTP read | OTP or flash content could not be properly authenticated. | 4 (Reset) |
| " | 13 | Reserved | — | — |
| " | 14 | Self test | A check of the integrity of the SE's internal storage failed during boot up. | 4 (Reset) |
| " | 15 | TRNG monitor | The TRNG monitor performs a number of tests on the collected entropy data. If any of these tests fail, this tamper source is triggered. | 0 (Ignore) |
| Hardware | 16 - 23 | PRS0 - 7 [1] | PRS consumer for SE Tamper 0 - 7 asserted | 0 (Ignore) |
| " | 24 | Decouple BOD [1] | Decouple Brown-Out-Detector threshold alert | 4 (Reset) |
| " | 25 | Temperature sensor [1] | SE temperature is monitored to be within 5 degrees C of the limits for the device. If the limit is exceeded, this tamper source will be triggered. | 0 (Ignore) |
| " | 26 | Voltage glitch falling | Voltage glitch detector detected a falling glitch | 0 (Ignore) |
| " | 27 | Voltage glitch rising | Voltage glitch detector detected a rising glitch | 0 (Ignore) |
| " | 28 | Secure lock | This tamper source indicates that the guarding mechanism (comparing the locks with their logical complement) of the debug port locks has failed. | 4 (Reset) |

| **Type** | **Number** | **Name** | **Description** | **Default Level** |
|----------|------------|----------|-----------------|-------------------|
| " | 29 | SE debug | Debug access to SE | 0 (Ignore) |
| " | 30 | Digital glitch | Digital glitch detector detected an event | 0 (Ignore) |
| " | 31 | SE ICACHE | The SE's instruction cache uses a checksum to verify the integrity of the data. This tamper source is triggered if the checksum is invalid. | 4 (Reset) |

**Notes**:

- [1] These tamper sources are available down to EM2. Other sources are available in EM1 and above.

- In EFR32xG21B devices, hardware tamper sources 24 to 27 can operate down to Energy Mode 3 (EM3), whereas other hardware tamper sources (16 - 23 and 28 - 31) can be active down to Energy Mode 1 (EM1).

- For devices other than EFR32xG21B see [Appendix C](./12-appendix-c-tamper-sources-reference.md) for tamper sources

- User configuration or tamper disable cannot reduce the tamper response below the default Level.

- The **User secure boot** source gets triggered if secure boot is enabled and host image verification fails. It is likely to put the device in the boot loop if users escalate the tamper response of this source to 4 (Reset).

- The **Mailbox and DCI authorizations** get triggered whenever one of the following conditions has occurred. The mailbox returns `SE_RESPONSE_AUTHORIZATION_ERROR`, and [DCI](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/) returns `AUTH_ERROR = 2`.

1. A mailbox or DCI command tries to exercise a key that it is not allowed to use (e.g., trying to export a non-exportable key).

2. A secure debug access or tamper disable request over the mailbox or DCI is invalidly signed.

3. A malformed HSE firmware upgrade over the mailbox or DCI is attempted.

- The **OTP read** gets triggered if there is an issue when decrypting and authenticating settings in HSE OTP or flash.

- The HSE has redundancy built into the locking mechanism, and the **Secure lock** source is used to detect errors in that redundancy.

- PRS inputs can allow user applications to implement additional tamper sources and feed them into the tamper response mechanism. The **PRS** tamper sources are under the control of the user application and could be reconfigured or disabled if the user application is compromised.

- The **Temperature sensor** source is not completely accurate and is generally only suitable for systems that expect to stay well within the specified temperature range. Users requiring a tighter temperature limit can implement their temperature monitor and provide the results as a tamper source via PRS.
