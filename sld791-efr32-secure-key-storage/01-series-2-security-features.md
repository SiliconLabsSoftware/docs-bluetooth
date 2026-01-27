# Series 2 Device Security Features

Protecting IoT devices against security threats is central to a quality product. Silicon Labs offers several security options to help developers build secure devices, secure application software, and secure paths of communication to manage those devices. Silicon Labs’ security offerings were significantly enhanced by the introduction of the Series 2 products that included a Secure Engine. The Secure Engine is a tamper-resistant component used to securely store sensitive data and keys and to execute cryptographic functions and secure services.

On Series 1 devices, the security features are implemented by the TRNG (if available) and CRYPTO peripherals.

On Series 2 devices, the security features are implemented by the Secure Engine and CRYPTOACC (if available). The Secure Engine may be hardware-based, or virtual (software-based). Throughout this document, the following abbreviations are used:

- HSE - Hardware Secure Engine

- VSE - Virtual Secure Engine

- SE - Secure Engine (either HSE or VSE)

Additional security features are provided by Secure Vault. Three levels of Secure Vault feature support are available, depending on the part and SE implementation, as reflected in the following table:

<table class="classic"><colgroup><col width="auto"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Level (1)</th><th>SE Support</th><th>Part (2)</th></tr></thead><tbody><tr><td><p>Secure Vault High (SVH)</p></td><td><p>HSE only (HSE-SVH)</p></td><td><p>Refer to <a href="https://docs.silabs.com/iot-security/latest/iot-endpoint-security-fundamentals/">IoT Endpoint Security Fundamentals</a> for details on supporting devices.</p></td></tr><tr><td><p>Secure Vault Mid (SVM)</p></td><td><p>HSE (HSE-SVM)</p></td><td><p>"</p></td></tr><tr><td><p>"</p></td><td><p>VSE (VSE-SVM)</p></td><td><p>"</p></td></tr><tr><td><p>Secure Vault Base (SVB)</p></td><td><p>N/A</p></td><td><p>"</p></td></tr></tbody></table>

**Notes**:

1. The features of different Secure Vault levels can be found in [https://www.silabs.com/security](https://www.silabs.com/security).
2. [IoT Endpoint Security Fundamentals](https://docs.silabs.com/iot-security/latest/iot-endpoint-security-fundamentals/).

Secure Vault Mid consists of two core security functions:

- Secure Boot: Process where the initial boot phase is executed from an immutable memory (such as ROM) and where code is authenticated before being authorized for execution.

- Secure Debug access control: The ability to lock access to the debug ports for operational security, and to securely unlock them when access is required by an authorized entity.

Secure Vault High offers additional security options:

- Secure Key Storage: Protects cryptographic keys by "wrapping" or encrypting the keys using a root key known only to the HSE-SVH.

- Anti-Tamper protection: A configurable module to protect the device against tamper attacks.

- Device authentication: Functionality that uses a secure device identity certificate along with digital signatures to verify the source or target of device communications.

A Secure Engine Manager and other tools allow users to configure and control their devices both in-house during testing and manufacturing, and after the device is in the field.

## User Assistance

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
			<td><p><a href="https://docs.silabs.com/iot-security/latest/authenticating-devices-using-device-certificates/">Authenticating Silicon Labs Devices using Device Certificates</a></p></td>
			<td><p>How to authenticate a device using secure device certificates and signatures, at any time during the life of the product</p></td>
			<td><p>Secure Vault High, and Series 3 devices</p></td>
		</tr>
		<tr>
			<td><p>Secure Key Storage (this application note)</p></td>
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

## Key Reference

Public/Private keypairs along with other keys are used throughout Silicon Labs security implementations. Because terminology can sometimes be confusing, the following table lists the key names, their applicability, and the documentation where they are used.

<table class="classic"><colgroup><col width="30%"><col width="25%"><col width="auto"></colgroup><thead><tr><th>Key Name</th><th>Customer Programmed</th><th>Purpose</th></tr></thead><tbody><tr><td><p>Public Sign key (Sign Key Public)</p></td><td><p>Yes</p></td><td><p>Secure Boot binary authentication and/or OTA upgrade payload authentication</p></td></tr><tr><td><p>Public Command key (Command Key Public)</p></td><td><p>Yes</p></td><td><p>Secure Debug Unlock or Disable Tamper command authentication</p></td></tr><tr><td><p>OTA Decryption key (GBL Decryption key) aka AES-128 Key</p></td><td><p>Yes</p></td><td><p>Decrypting GBL payloads used for firmware upgrades</p></td></tr><tr><td><p>Attestation key aka Private Device Key</p></td><td><p>No</p></td><td><p>Device authentication for secure identity</p></td></tr></tbody></table>

## SE Firmware

Silicon Labs strongly recommends installing the latest SE firmware on Series 2 and Series 3 devices to support the required security features. Refer to [Production Programming of Series 2 and Series 3 Devices](https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/) for the procedure to upgrade the SE firmware and [IoT Endpoint Security Fundamentals](https://docs.silabs.com/iot-security/latest/iot-endpoint-security-fundamentals/) for the latest SE Firmware shipped with Series 2 and Series 3 devices and modules.