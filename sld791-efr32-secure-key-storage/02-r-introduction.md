# Introduction

The HSE isolates cryptographic functions and data from the host Cortex-M33 core. It is used to accelerate cryptographic operations as well as to provide a method to securely store keys. This application note will cover the Secure Key Storage feature of the HSE-SVH devices.

The HSE contains one-time programmable memory (OTP) key storage slots for three specific keys:

1. The Public Sign Key, used for Secure Boot and Secure Upgrades

2. The Public Command Key, used for Secure Debug unlock and tamper disable

3. The Symmetric OTA Decryption Key, used for Over-The-Air updates

These keys are one-time programmable, and, after programming, are persistent for the lifetime of the device.

HSE-SVH devices also contain four volatile storage slots for any other user keys. These slots are not persistent through a reset. In the case where a key needs persistent storage, the key must be stored outside of the HSE in non-volatile storage. After a device reset, the key can be loaded into the HSE volatile key storage for usage by index, or used in-place (passed to the HSE on every requested operation). Without any secure key storage mechanism, the user key stored in non-volatile storage is opened to storage-extraction attacks (such as gaining access to and downloading device flash), as well as application-level attacks (i.e., taking control of the user application or privileges in a manner that allows access to the keys).

With Secure Key Storage, a user can only access a key from the HSE in a 'wrapped' format. In this format, the key is encrypted by a device-unique root key, only available to the HSE. This allows a user to store a key confidentially in non-volatile storage to provide key persistence. Using Secure Key Storage, the plaintext key is never stored in non-volatile memory, preventing storage-extraction attacks from obtaining the key. After a device reset, the wrapped key can be loaded into the HSE for usage without ever exposing the plaintext key to the application, which also prevents application-level attacks from exposing the key.

SVM devices can only support Secure Key Storage through the use of [TrustZone](./04-r-storagecomparisons-trustzone.md). GSDK v4.2.2 is the first version to support TrustZone software development on Series 2 devices.

Silicon Labs provides [Custom Part Manufacturing Service (CPMS)](https://www.silabs.com/developers/custom-part-manufacturing-service) to inject custom secret keys on the chips during manufacturing. For more information about CPMS, see the [Custom Part Manufacturing Service User's Guide](https://docs.silabs.com/iot-security/latest/iot-security-cpms/).
