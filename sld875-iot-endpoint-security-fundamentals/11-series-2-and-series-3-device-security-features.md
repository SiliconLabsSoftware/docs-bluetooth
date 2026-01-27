# Series 2 and Series 3 Device Security Features

Protecting IoT devices against security threats is central to a quality product. Silicon Labs offers several security options to help developers build secure devices, secure application software, and secure paths of communication to manage those devices. Silicon Labs’ security offerings were significantly enhanced by the introduction of the Series 2 products that included a Secure Engine. The Secure Engine is a tamper-resistant component used to securely store sensitive data and keys, and to execute cryptographic functions and secure services.

On Series 1 devices, the security features are implemented by the TRNG (if available) and CRYPTO peripherals.

On Series 2 devices, the security features are implemented by the Secure Engine and CRYPTOACC (if available). The Secure Engine may be hardware-based, or virtual (software-based). Throughout this document, the following abbreviations are used:

- HSE - Hardware Secure Engine

- VSE - Virtual Secure Engine

- SE - Secure Engine (either HSE or VSE)

On Series 3 devices, the security features are implemented by the Secure Engine and HOSTCRYPTO (if available). The Secure Engine may is hardware-based.

Additional security features are provided by Secure Vault. Three levels of Secure Vault feature support are available, depending on the part and SE implementation, as reflected in the following table:

| **Security Level (1)** | **SE Support** | **MCU** | **Wireless SoC (2)** |
|------------------------|----------------|---------|----------------------|
| Secure Vault Base (SVB) | N/A | EFM32JG1, EFM32PG1, EFM32JG12, EFM32PG12, EFM32GG11, EFM32GG12, EFM32TG11 | EFR32xG1, EFR32xG12, EFR32xG13, EFR32xG14 |
| Secure Vault Mid (SVM) | VSE (VSE-SVM) | EFM32PG22C | EFR32xG22C, EFR32xG27C (3) |
| Secure Vault Mid (SVM) | HSE (HSE-SVM) | - | EFR32xG21A, EFR32xG21A (Rev C), EFR32MR21A (Rev C), EFR32xG23A, EFR32xG24A, EFR32xG25A, EFR32xG28A |
| Secure Vault High (SVH) | HSE only (HSE-SVH) | EFM32PG23B, EFMPG26B, EFM32PG28B | EFR32xG21B, EFR32Xg21B (Rev C) EFR32xG23B, EFR32xG24B, EFR32xG25B, EFR32xG26B, EFR32xG28B, EFR32xG29B, SixG301 |

>Note:
>
>1. The features of different Secure Vault levels can be found in [https://www.silabs.com/security.](https://www.silabs.com/security).
>
>2. The x is a letter B, F, M, or Z.
>
>3. Unlike other VSE-SVM parts, the EFR32xG27C device has a built-in PUF.

Secure Vault Mid consists of two core security functions:

- **Secure Boot**: Process where the initial boot phase is executed from an immutable memory (such as ROM) and where code is authenticated before being authorized for execution.

- **Secure Debug access control**: The ability to lock access to the debug ports for operational security, and to securely unlock them when access is required by an authorized entity.

Secure Vault High offers additional security options:

- **Secure Key Storage**: Protects cryptographic keys by “wrapping” or encrypting the keys using a root key known only to the HSE-SVH.

- **Anti-Tamper protection**: A configurable module to protect the device against tamper attacks.

- **Device authentication**: Functionality that uses a secure device identity certificate along with digital signatures to verify the source or target of device communications.

A Secure Engine Manager and other tools allow users to configure and control their devices both in-house during testing and manufacturing, and after the device is in the field.

Silicon Labs strongly recommends installing the latest SE FW image on Series 2 and Series 3 devices and updating to the latest GSDK or SiSDK to mitigate security vulnerabilities. The latest SE FW image can be found in this Windows folder for GSDK v4.x:

`C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\gecko_sdk\util\se_release\public`

Or

`C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\simplicity_sdk\util\se_release\public`

Refer to [Production Programming of Series 2 and Series 3 Devices](https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/) for guidance on the SE firmware upgrade procedure. The latest SE firmware shipped with Series 2 and Series 3 devices and modules (if available) at the time of this writing are listed in the following table:

| **Series 2 MCU and Wireless SoC VSE - SVM** | **Shipped SE Firmware Version** |
|---------------------------------------------|---------------------------------|
| EFM32PG22C | 1.2.12 |
| EFR32xG22C | 1.2.12 |
| EFR32xG22C (Rev D) | 1.2.14 |
| EFR32xG27C | 2.2.1 |

| **Series 2 Wireless SoC HSE - SVM** | **Shipped SE Firmware Version** |
|-------------------------------------|---------------------------------|
| EFR32xG21A | 1.2.13 |
| EFR32MR21A (Rev C) | 1.2.16 |
| EFR32xG21A (Rev C) | 1.2.16 |
| EFR32xG23A | 2.1.7 |
| EFR32xG24A | 2.1.7 |
| EFR32xG25A | 2.2.1 |
| EFR32xG28A | 2.2.2 |

| **Series 2 MCU and Wireless SoC HSE - SVH** | **Shipped SE Firmware Version** |
|---------------------------------------------|---------------------------------|
| EFR32xG21B | 1.2.13 |
| EFR32xG21B (Rev C) | 1.2.16 |
| EFM32PG23B | 2.1.7 |
| EFR32xG23B | 2.1.7 |
| EFR32xG24B | 2.1.7 |
| EFR32xG25B | 2.2.1 |
| EFM32PG28B | 2.2.2 |
| EFR32xG28B | 2.2.2 |

| **Series 3 MCU and Wireless SoC HSE - SVH** | **Shipped SE Firmware Version** |
|---------------------------------------------|---------------------------------|
| SixG301 | 3.3.2 |

In support of these products Silicon Labs offers whitepapers, webinars, and documentation. The following table summarizes the key security documents:

<table class="classic">
	<colgroup>
		<col width="30%">
		<col width="45%">
		<col width="auto">
	</colgroup>
	<thead>
		<tr>
			<th>Document</th>
			<th>Summary</th>
			<th>Applicability</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/series2-secure-debug/">Series 2 and Series 3 Secure Debug</a></p></td>
			<td><p>How to lock and unlock Series 2 and Series 3 debug access, including background information about the SE</p></td>
			<td><p>Secure Vault Mid, High, and Series 3 devices</p></td>
		</tr>
		<tr>
			<td><p><a href="https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/">Series 2 and Series 3 Secure Boot with RTSL</a></p></td>
			<td><p>Describes the secure boot process on Series 2 and Series 3 devices using SE</p></td>
			<td><p>Secure Vault Mid, High, and Series 3 devices</p></td>
		</tr>
        <tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/efr32-secure-vault-tamper/">Anti-Tamper Protection Configuration and Use</a></p></td>
			<td><p>How to program, provision, and configure the anti-tamper module</p></td>
			<td><p>Secure Vault High, and Series 3 devices</p></td>
		</tr>
		<tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/series2-trustzone/">Series 2 TrustZone</a></p></td>
			<td><p>Describes the basics of TrustZone, secure and privileged programming model, and shows how to upgrade existing application to TrustZone.</p></td>
			<td><p>Series 2 devices</p></td>
		</tr>
		<tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/authenticating-devices-using-device-certificates/">Authenticating Silicon Labs Devices using Device Certificates</a></p></td>
			<td><p>How to authenticate a device using secure device certificates and signatures, at any time during the life of the product</p></td>
			<td><p>Secure Vault High, and Series 3 devices</p></td>
		</tr>
		<tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/efr32-secure-key-storage/">Secure Key Storage</a></p></td>
			<td><p>How to securely "wrap" keys so they can be stored in non-volatile storage.</p></td>
			<td><p>Secure Vault High, and Series 3 devices</p></td>
		</tr>
		<tr>
			<td><p><a href="https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/">Production Programming of Series 2 and Series 3 Devices</a></p></td>
			<td><p>How to program, provision, and configure security information using SE during device production</p></td>
			<td><p>Secure Vault Mid, High, and Series 3 devices</p></td>
		</tr>
        <tr>
			<td><p>AN1504: Series 3 Security Overview</a></p></td>
			<td><p>High level overview of the security features included in Series 3 devices</p></td>
			<td><p>Series 3</p></td>
		</tr>
        <tr>
			<td><p>AN1509: Series 3 AXiP</a></p></td>
			<td><p>How to encrypt and authenticate memory contents</p></td>
			<td><p>Series 3</p></td>
		</tr>
	</tbody>
</table>

| **Document** | **Summary** | **Applicability** |
|--------------|-------------|-------------------|
| [Programming Series 2 Devices Using the Debug Challenge Interface (DCI) and Serial Wire Debug (SWD)](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/) | How to provision and configure Series 2 devices through the DCI and how to program their internal flash memory through the SWD | Series 2 |
| [Integrating Crypto Functionality Using PSA Crypto Compared to Mbed TLS](https://docs.silabs.com/iot-security/latest/mbedtls-psa-crypto-porting-guide/) | How to integrate crypto functionality into applications using Silicon Labs implementation of PSA Crypto compared to Mbed TLS | Series 1, Series 2 and Series 3 |
