# Production Programming of Series 2 and Series 3 Devices

> **Note: This section replaces *AN1222: Production Programming of Series 2 and Series 3 Devices*. Further updates to this application note will be provided here**.

This application note demonstrates how to properly program, provision, and configure Series 2 and Series 3 devices in a production environment.

Series 2 and Series 3 devices contain a Secure Engine, which runs Secure Engine firmware. When a newer version of Secure Engine firmware is released, the firmware may be upgraded either in the production programming process for devices still in manufacturing or via a field update for deployed devices. Keys must be provisioned to the Secure Engine's one-time-programmable (OTP) memory to use the Secure Boot and Secure Debug features.

For more information about Secure Engine, see [Secure Engine Subsystem](https://docs.silabs.com/iot-security/latest/series2-secure-debug/03-secure-engine-subsystem) in *Series 2 and Series 3 Secure Debug*.

## Key Points

- It is the customer's responsibility to ensure the Secure Engine firmware is up-to-date
- The Secure Engine firmware can be upgraded via the Serial Wire Debug (SWD) interface
- Secure Engine firmware is protected from downgrade
- Secure Engine's OTP memory prevents re-writing of:
  - GBL Decryption Key
  - Public Sign Key
  - Public Command Key
  - Secure Boot Enable flag and Tamper Configuration
