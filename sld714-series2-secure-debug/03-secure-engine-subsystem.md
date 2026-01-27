# Secure Engine Subsystem

## Overview

The HSE refers to a separate security co-processor that provides hardware isolation between security functions and the host processor.

The VSE refers to a collection of security functions available to the host processor in Root mode if a separate security co-processor is not provided.

The SE is used to perform a series of cryptographic operations and other secure system operations as described in the following table.

| **Operation** | **VSE-SVM** | **HSE-SVM** | **HSE-SVH** | **Description** |
|---------------|-------------|-------------|-------------|-----------------|
| Unique ID | Y | Y | Y | Software can identify every device. |
| Secure Boot with RTSL | Y | Y | Y | Only boot authenticated firmware. |
| Secure Debug | Y | Y | Y | Securely lock/unlock debug ports using authorized tokens |
| Crypto Engine (1) | Y | Y | Y | Up to 256-bit ciphers and elliptic curves. |
| TRNG (1) | Y | Y | Y | Generate keys for cryptography. |
| DPA Countermeasures | — | Y | Y | Resist side channel attacks. |
| Secure Key Storage | — | — | Y | Protected by PUF technology. |
| Secure Key Management | — | — | Y | Isolate encrypted keys from application code. |
| Secure Attestation | — | — | Y | Ensure integrity and authenticity. |
| Anti-Tamper | — | — | Y | Detect tamper and protect keys/data. |
| Advanced Crypto | — | — | Y | Up to 512-bit ciphers and P-521-bit elliptic curves. |

**Note**:

1. On VSE-SVM devices, the crypto engine and TRNG (True Random Number Generator) are implemented by the CRYPTOACC (Cryptographic Accelerator) peripheral.

To start using the secure debug unlock functionality, the device needs to be provisioned. These steps include writing one time programmable (OTP) settings to the SE to determine which functionality is enabled, and uploading the Public Command Key to validate a secure debug attempt.

This application note describes how different device debug locks and unlocks are implemented through the SE on Series 2 and Series 3 devices.

The secure debug feature is implemented by Root code executed by the HSE Core. For VSE devices secure debug feature is implemented by Cortex-M33 operating in Root mode.

Silicon Labs strongly recommends installing the latest SE firmware to support the required security features. The latest SE firmware image (`.seu` and `.hex`) and release notes can be found in the Windows folder mentioned below.

`C:\Users\\<userName\>\SimplicityStudio\SDKs\Simplicity_sdk\util\se_release\public`

## Command Interface

Interaction with the SE is performed over a command interface. The command interface is available through a dedicated Debug Challenge Interface (DCI) as well as through a mailbox interface from the Cortex-M33.

Some commands may not be available at all times and may not be accessible over both interfaces. The DCI typically only contains operations for setting up a new device and for locking it down (meant for production processes), while the mailbox interface also contains commands to support cryptographic operations in HSE.

### Mailbox

The SE Manager Mailbox Interface is a mechanism used on Silicon Labs Series 2 and Series 3 devices to communicate with the Secure Engine (SE) for cryptographic and security operations. The mailbox interface allows the host processor (such as Cortex-M33) to send commands to the SE, typically for secure key storage, cryptographic operations, device provisioning, and secure identity management.

Secure Engine Manager (SE Manager) provides an abstraction of the Secure Engine's command set. The SE Manager also provides APIs for cryptographic operations and thread synchronization.

![SE Manager Mailbox interface](./resources/sld714-image17.png)

### Debug Challenge Interface (DCI)

The Debug Challenge Interface (DCI) is made available through commands in Simplicity Studio and Simplicity Commander. This is the easiest way to access and set up the different security options.

For more information about DCI, see [Programming Series 2 Devices using the Debug Challenge Interface (DCI) and Serial Wire Debug (SWD)](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/).
