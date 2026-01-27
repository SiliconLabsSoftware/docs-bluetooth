# Anti-Tamper Configuration

The user can provision the anti-tamper configuration in HSE OTP detailed in the following table through `sl_se_init_otp` in the [SE Manager](./03-secure-engine-manager.md). Tamper configurations must be programmed with secure boot settings and are immutable once written.

For more information on enabling the OTP tamper configuration along with the secure boot settings, see *Enabling Secure Boot and Tamper Configuration* in [Production Programming of Series 2 and Series 3 Devices](https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/09-enabling-secure-boot-and-tamper-configuration/).

| **Setting** | **Description** |
|-------------|-----------------|
| Tamper response levels | A user response level for each tamper source (1) |
| Filter settings | The tamper filter counter has two settings: trigger threshold and reset period |
| Digital Glitch Detector Always On | Bit 1 of tamper flag: 0 — Digital glitch detector runs only when the HSE is executing a command; 1 — Digital glitch detector runs even when the HSE is not performing any operations (note that this leads to increased energy consumption) |
| Keep Tamper Alive During Sleep (2) | Bit 2 of tamper flag: 0 — The tamper module stops running in sleep mode; 1 — The tamper module keeps running in sleep mode (down to EM3) |
| Reset threshold | The number of consecutive tamper resets (up to 255) before the part enters diagnostic mode (3) |

**Notes**:

1. The effective response of a tamper source is the maximum value between the default level and user level (Active level = MAX(`default level`, `user level`)). If the user sets the response of a tamper source to a level lower than the default level, the setting will still be saved to HSE OTP but does not take any effect. The HSE returns the user levels instead of active levels of all tamper sources when reading back `sl_se_read_otp` the tamper configuration from the HSE OTP.

2. This flag is not available on EFR32xG21B devices.

3. If the threshold is set to 0, the part will never enter the diagnostic mode due to tamper reset.
