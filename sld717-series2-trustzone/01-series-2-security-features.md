# Series 2 Device Security Features

Protecting IoT devices against security threats is central to a quality product. Silicon Labs offers several security options to help developers build secure devices, secure application software, and secure communication paths to manage those devices. Silicon Labs’ security offerings were significantly enhanced by the introduction of the Series 2 products that included a Secure Engine. The Secure Engine is a tamper-resistant component used to securely store sensitive data and keys, and to execute cryptographic functions and secure services.

On Series 2 devices, the security features are implemented by the Secure Engine and CRYPTOACC (if available). The Secure Engine may be hardware-based or virtual (software-based). Throughout this document, the following abbreviations are used:

- HSE - Hardware Secure Engine

- VSE - Virtual Secure Engine

- SE - Secure Engine (either HSE or VSE)

Additional security features are provided by Secure Vault. Three levels of Secure Vault feature support are available, depending on the part and SE implementation, as reflected in the following table:

<table class="classic"><colgroup><col width="30%"/><col width="30%"/><col width="40%"/></colgroup><thead><tr><th><p>Level (1)</p></th><th><p>SE Support</p></th><th><p>Part</p></th></tr></thead><tbody><tr><td><p>Secure Vault High (SVH)</p></td><td><p>HSE only (HSE-SVH)</p></td><td><p>Refer to <a href="https://docs.silabs.com/iot-security/latest/iot-endpoint-security-fundamentals/">IoT Endpoint Security Fundamentals</a> for details on supporting devices.</p></td></tr><tr><td><p>Secure Vault Mid (SVM)</p></td><td><p>HSE (HSE-SVM)</p></td><td><p>"</p></td></tr><tr><td><p>Secure Vault Mid (SVM)</p></td><td><p>VSE (VSE-SVM)</p></td><td><p>"</p></td></tr><tr><td><p>Secure Vault Base (SVB)</p></td><td><p>N/A</p></td><td><p>"</p></td></tr></tbody></table>

>**Note**: 1. The features of different Secure Vault levels can be found in [https://www.silabs.com/security](https://www.silabs.com/security).

Secure Vault Mid consists of two core security functions:

- Secure Boot: Process where the initial boot phase is executed from an immutable memory (such as ROM) and where code is authenticated before being authorized for execution.

- Secure Debug Access Control: The ability to lock access to the debug ports for operational security, and to securely unlock them when access is required by an authorized entity.

Secure Vault High offers additional security options:

- Secure Key Storage: Protects cryptographic keys by "wrapping" or encrypting the keys using a root key known only to the HSE-SVH.

- Anti-Tamper protection: A configurable module to protect the device against tamper attacks.

- Device authentication: Functionality that uses a secure device identity certificate along with digital signatures to verify the source or target of device communications.

Series 2 devices require a specific [SE firmware version](./06-r-migration.md) to support the TrustZone implementation. Refer to [Production Programming of Series 2 and Series 3 Devices](https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/) to learn how to upgrade the SE firmware and [IoT Endpoint Security Fundamentals](https://docs.silabs.com/iot-security/latest/iot-endpoint-security-fundamentals/) for the latest SE Firmware shipped with Series 2 devices and modules.

Series 2 devices use Cortex-M33 core to implement the ARMv8-M Mainline TrustZone security extension and refer to TrustZone as [Bus Level Security](./03-r-bls.md). The following table lists the configuration of TrustZone related components in the Series 2 Cortex-M33 core.

<table class="classic"><colgroup><col width="30%"/><col width="30%"/><col width="40%"/></colgroup><thead><tr><th><p>Component</p></th><th><p>Series 2 Configuration</p></th><th><p>Description</p></th></tr></thead><tbody><tr><td><p>Security Extension (TrustZone)</p></td><td><p>Enabled</p></td><td><p>The security extension cannot be disabled, and the entire memory after RESET is Secure by default.</p></td></tr><tr><td><p>Memory Protection Unit (MPU)</p></td><td><p>16 regions (maximum)</p></td><td><p>The MPU regions for both Secure and Non-secure MPUs.</p></td></tr><tr><td><p>Security Attribution Unit (SAU)</p></td><td><p>8 regions (maximum)</p></td><td><p>The SAU regions for Non-secure and Non-secure Callable.</p></td></tr></tbody></table>
