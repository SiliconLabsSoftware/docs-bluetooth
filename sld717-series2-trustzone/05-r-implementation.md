# TrustZone Implementation

The goal of TrustZone implementation is to provide Secure Key Storage that can keep access to keys limited to Secure applications while at the same time allowing Non-secure applications to exercise the keys. It is an added feature for the SVM devices that do not have dedicated hardware for [Secure Key Storage](https://docs.silabs.com/iot-security/latest/efr32-secure-key-storage/) as in SVH devices.

The [PSA Crypto](https://docs.silabs.com/iot-security/latest/mbedtls-psa-crypto-porting-guide/) is placed in a Secure region to keep key material hidden from the Non-secure application. The exposed PSA Crypto APIs stay the same while the backend provides persistent key encryption and decryption similar to the key wrapping and unwrapping functionality of the SVH device.

The following items need to be considered when upgrading the existing system for Secure Key Storage with TrustZone.

- [System Configuration](#system-configuration)

- [Gecko Bootloader](#gecko-bootloader)

- [Secure Library](#secure-library)

- [TrustZone Secure Key Storage](#trustzone-secure-key-storage)

- [PSA Attestation](#psa-attestation)

- [SE Manager](#se-manager)

- [Common Vulnerabilities and Exposures (CVE)](#common-vulnerabilities-and-exposures-cve)

## System Configuration

The system configuration includes the following items:

- Enable system exceptions in the Secure state.

- Set the security attributes of different regions in the SAU and ESAU.

- Place peripherals and associated interrupts in either Secure or Non-secure applications.

- Assign the Bus Masters' security attributes.

- The system has two Secure/Non-secure pairs for the bootloader and application. The Secure part of each pair is responsible for properly configuring the split in its Secure application before branching to the Non-secure application.

>**Note**: The secure application will issue a software reset at startup (fatal error) if the device's SE firmware version is lower than the [first version](./06-r-migration.md) that supports TrustZone.

### System Exceptions

The following [system exceptions](./02-r-basic.md#type-of-exceptions) are enabled in the Secure state for the bootloader and application.

- MemManage Fault

- BusFault

- UsageFault

- SecureFault

### Main Flash Layout

The following figure is an overview of the main flash layout that covers the isolation requirements for the Secure Key Storage solution. The SAU and ESAU configurations provide the required security to the Cortex-M33 and other Bus Masters during boot and normal operation.

![Main Flash Layout](././resources/sld717-main-flash-layout.png)

1. Settings:

   - The application is set to non-executable (XN) by [Secure MPU](./02-r-basic.md#secure-attribution-unit-sau-implementation-defined-attribution-unit-idau-and-memory-protection-unit-mpu) to avoid any code execution in this area during boot.

   - The bootloader is set to non-executable (XN) by Secure MPU to avoid any code execution in this area during normal operation.

2. The ESAU configuration only uses the NSC section by setting mrb01 to the [base address of region 0](./04-r-programming.md#). The reason is that lite ESAU in other Bus Masters treats both S and NSC as a Secure attribute. For the Cortex-M33, the SAU upgrades the NSC in the ESAU to Secure. The 32 bytes region alignment of SAU also relaxes the 4 kB alignment restriction on the start address of the NSC in ESAU.

3. The whole application area is set to Secure in SAU for Cortex-M33 during boot to hide details from the bootloader NS part.

4. The ESAU cannot mark any region that comes after a Non-secure section as Secure (must be in the order of S/NSC/NS). Therefore the Secure application area does not align between the Cortex-M33 (SAU + ESAU) and other Bus Masters (lite ESAU) during boot. The secrets stored in that Secure region expose as Non-secure for other Bus Masters during boot (no such issue in normal operations). So the application must not save any plaintext secrets in that Secure region to overcome this limitation during boot.

5. The NVM storage is in the Non-secure region, so the application must [encrypt](./05-r-implementation.md#secure-library) the persistent keys before storing them in this area.

### RAM Layout

The following figure is an overview of the RAM layout used for the bootloader and application. The SAU and ESAU are used to split the RAM into a Secure and Non-secure region (Non-secure Callable is not required).

![RAM Layout](././resources/sld717-ram-layout.png)

In practice, the Secure part (bootloader or application) takes ownership of the amount of RAM it needs from the beginning of RAM and leaves the rest (up to the ESAU 4 kB alignment requirement) configured as Non-secure.

The bootloader does not know how the application partitions the RAM between Secure and Non-secure. So bootloader removes any secrets from RAM before handing control to the application.

### Info Flash and EPPB

The following figure is an overview of the Info flash and EPPB layout for the application. The Cortex-M33 core is the only Bus Master that can access the EPPB region.

![Info Flash and EPPB Layout](././resources/sld717-info-flash-and-eppb-layout.png)

### Peripheral and Device

The following figure is an overview of the peripheral and device layout for the bootloader and application. The SAU and ESAU are used to split the peripheral and device into a Secure and Non-secure region.

![Peripheral and Device Layout](././resources/sld717-peripheral-and-device-layout.png)

The Secure software is responsible for moving all peripherals and associated interrupts to the Non-secure state at startup, except for the peripherals and interrupts that need to be Secure. The Non-secure software must not include code that attempts to directly access any peripheral that is used by the Secure software.

**Peripherals owned by the Secure software (application)**

1. Security Management Unit (SMU)

   - It prevents Non-secure software from changing the configuration for the ESAUs, BMPUs, and PPUs.

   - Except for EFR32xG21 devices, some features are also available in the dedicated [Non-secure version of SMU registers](./03-r-bls.md#) (`SMU_CFGNS`).

2. CRYPTOACC (VSE devices) or SEMAILBOX (HSE devices)

   - The crypto engine is placed in the Secure domain for [Secure library](#secure-library).

3. System Configuration (SYSCFG)

   - It prevents Non-secure software from changing system configurations for Secure software.

   - Except for EFR32xG21 devices, some features are also available in the dedicated [Non-secure version of SYSCFG registers](./03-r-bls.md#) (`SYSCFG_CFGNS`).

4. Memory System Controller (MSC)

   - It prevents Non-secure software from writing to Secure flash.

**Peripheral interrupts owned by the Secure software**:

**Table**: Secure Peripheral Interrupts (Application)

<table><tbody><tr><td>VSE Device</td><td>HSE Device</td></tr><tr><td>SMU_SECURE_IRQn</td><td>SMU_SECURE_IRQn</td></tr><tr><td>SYSCFG_IRQn</td><td>SYSCFG_IRQn</td></tr><tr><td>MSC_IRQn</td><td>MSC_IRQn</td></tr><tr><td>CRYTOACC_IRQn</td><td>SEMBRX_IRQn</td></tr><tr><td>TRNG_IRQn</td><td>SEMBTX_IRQn</td></tr><tr><td>PKE_IRQn</td><td></td></tr></tbody></table>

The `PRIS` bit in the `AIRCR` register is set to 1 to place all Non-secure exceptions/interrupts in [lower priority level space](./02-r-basic.md). Therefore any Secure exceptions/interrupts can be programmed with higher priority than Non-secure ones.

The [`BMPUSEC`](./03-r-bls.md#) and [`PPUSEC`](./03-r-bls.md#) interrupt enable flags in the `SMU_IEN` register are set to enable the SMU security fault interrupts (`SMU_SECURE_IRQn`) on Bus Masters and peripherals.

**Floating Point Unit (FPU)**:

The Secure application does not use the FPU. But the Secure startup code also enables the [FPU](./04-r-programming.md) for use by the Non-secure application.

### Bus Masters

To keep all secrets from the Non-secure world, only the Bus Masters in the table below can access data in the Secure world. For the Bus Masters living in the Secure world, the secure application must configure their corresponding control interfaces in the peripheral space to Secure. The Cortex-M33 core as a Bus Master is split to run in Secure and Non-secure contexts.

**Table**: Secure Bus Masters (Application)

<table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th>Device</th><th>Secure Bus Master</th><th>Control Interface of Bus Master</th></tr></thead><tbody><tr><td><p>VSE</p></td><td><p>CRYPTOACC</p></td><td><p>CRYTPOACC</p></td></tr><tr><td><p>HSE</p></td><td><p>SEDMA or SEEXTDMA</p></td><td><p>SEMAILBOX</p></td></tr></tbody></table>

**Notes**:

- Use `SMU_BMPUSATD0` register to [configure](./04-r-programming.md#) the security attribute of a Bus Master.

- Use `SMU_PPUSATDn` register to [configure](./04-r-programming.md#) the control interface of Bus Master as a Secure peripheral.

- LDMA is set as a Non-secure Bus Master to make sure it cannot be used to copy out data from the Secure memory.

### Application Transitions

The system contains two Secure/Non-secure pairs.

1. The [bootloader pair](#bootloader pair) has a Secure bootloader and a Non-secure bootloader containing the communication interfaces.

2. The [application pair](#application pair) has a Secure application and a Non-secure application consisting of the wireless stacks (if applicable) and application layers.

As described in the preceding sections, the Secure part of these pairs is responsible for setting the security configurations of the system during startup. For the handover between Secure/Non-secure pairs, the software must restore the system so the Secure part of the other pair can execute and reconfigure the system.

The software must reconfigure the following items before transitioning to the next Secure/Non-secure pair:

- Restored all peripherals and interrupts to Secure

- Reset ESAU to default configuration (all configurable regions to Secure)

- Reset SAU to default configuration (Secure for everything)

- Reset MPU to default configuration (removes any XN)

## Gecko Bootloader

The [Gecko bootloader](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) ensures the Secure assets are protected during the boot flow and normal operation.

![Gecko Bootloader Flow](././resources/sld717-gecko-bootloader-flow.png)

1. The SAU and [Secure MPU](./02-r-basic.md) mark all the flash for application and NVM as Secure and non-executable (XN) during boot. It guards against bootloader NS code execution branching into the application code.

2. The bootloader needs to split into Secure and Non-secure software to protect secrets in the system. Secure code can access the entire flash to validate or upgrade the system.

3. For VSE devices, the [GBL Decryption Key (AES-128 key)](https://docs.silabs.com/iot-security/latest/prod-programming-series2-and-series3/) is moved from the NS memory (last page of the main flash) to the Secure part of the bootloader. The Simplicity Commander v1.13 or higher provides a feature to inject the AES-128 key to the bootloader binary file.

   ```C
   commander convert <BL image file> --aeskey <decryption key file> --outfile <BL image with decryption key>
   ```

4. The bootloader communication interfaces are placed in the NS area to support various [communication components](https://docs.silabs.com/mcu-bootloader/latest/group-Communication) below for firmware upgrades.

   - BGAPI UART

   - EZSP-SPI

   - UART XMODEM

5. The NS communication functions call into the [bootloader APIs](https://docs.silabs.com/mcu-bootloader/latest/group-Interface) placed in the bootloader NSC region. The Secure application [validates](./02-r-basic.md#) all input arguments before processing the request.

6. Before transiting from bootloader to normal operation, it resets the SAU to default configuration to make all the flash for bootloader as Secure.

7. The Non-secure application software can call [bootloader APIs](https://docs.silabs.com/mcu-bootloader/latest/group-Interface) through application NSC, and the corresponding Secure function releases the non-executable (XN) restriction on the bootloader during normal operation.

## Secure Library

The goal of the Secure library is to keep the [PSA Crypto key](#trustzone-secure-key-storage) and [attestation token](#psa-attestation) protected from malicious code on the NSPE. The following figure overviews multiple components to support the Secure library.

![Secure Library Components](././resources/sld717-secure-library-components.png)

1. The NS interfaces in NSPE are responsible for packing and passing all input arguments over the [NSC](./02-r-basic.md#) functions on wrappers in SPE.

2. The wrappers in SPE validate all input arguments before calling into the corresponding APIs in different drivers.

3. Because of the system memory layout limitation, the [flash](#main-flash-layout) for NVM3 storage is located in the NSPE. Therefore the updated PSA Internal Trusted Storage (ITS) driver needs to encrypt all crypto keys before storing them in Non-secure NVM.

4. Data stored directly using the NVM3 APIs are not encrypted.

The following table describes the new and updated components of the Secure library.

<table class="classic"><colgroup><col width="35%"/><col width="65%"/></colgroup><thead><tr><th>Component</th><th>Description</th></tr></thead><tbody><tr><td><p>SE Manager NS interface</p></td><td><p>This component contains SE Manager API callable from the NSPE. It packages the list of input arguments in the appropriate format before calling into the SE Manager wrapper's NSC functions.</p></td></tr><tr><td><p>SE manager wrapper</p></td><td><p>This component contains the interface into SE Manager exposed to the NSPE. These NSC functions grant access to the SE Manager utility API and validate all input arguments before calling into SE Manager.</p></td></tr><tr><td><p>PSA Crypto & Attestation NS interface</p></td><td><p>This component contains PSA Crypto and attestation API callable from the NSPE. It packages the list of input arguments in the appropriate format before calling into the PSA Crypto and attestation wrapper's NSC functions.</p></td></tr><tr><td><p>PSA Crypto & Attestation wrapper</p></td><td><p>This component contains the interface into PSA Crypto and attestation exposed to the NSPE. These NSC functions grant access to the entire PSA Crypto and attestation API and validate all input arguments before calling into PSA Crypto and attestation.</p></td></tr><tr><td><p>PSA attestation</p></td><td><p>This component in SPE provides the functionality required by the PSA attestation specification.</p></td></tr><tr><td><p>Encrypted PSA ITS</p></td><td><p>The PSA ITS layer builds on top of NVM3. This component is updated to support encrypted storage to secure stored keys. The encryption is based on the device's TrustZone Root Key.</p></td></tr><tr><td><p>NVM3 NS interface</p></td><td><p>This component contains NVM3 API callable from the NSPE. It packages the list of input arguments in the appropriate format before calling into the NVM3 wrapper's NSC functions.</p></td></tr><tr><td><p>NVM3 wrapper</p></td><td><p>This component contains the interface into NVM3 exposed to the NSPE. These NSC functions grant access to the NVM3 API and validate all input arguments before calling into NVM3.</p></td></tr></tbody></table>

**Notes**:

- The SE Manager NS interface, PSA Crypto NS interface, and NVM3 NS interface in the NSPE provide drop-in replacement on [SE Manager utility](#se-manager), [PSA Crypto](https://armmbed.github.io/mbed-crypto/html/about.html), and [NVM3](https://docs.silabs.com/gecko-platform/latest/driver/api/group-nvm3) APIs for existing wireless stacks and user applications.

- The NSC calls can only take a limited number of arguments, so all NSC functions take a pointer to a list of parameters to support a long list of arguments. All arguments must be validated using the [intrinsic functions](./02-r-basic.md) from CMSIS.

## TrustZone Secure Key Storage

The TrustZone Secure Key Storage provides a solution to store a user key in Secure RAM or an encrypted form in Non-secure flash.

The TrustZone Root Key stored in the SE NVM for Secure Key Storage encryption is generated or renewed by following operations.

- The TrustZone Root Key had already existed if the shipped Series 2 device with [SE firmware version](./06-r-migration.md#) supports this key.

- Generate a TrustZone Root Key when upgrading from a SE firmware version that did not support this key to the one that does.

- Renew a TrustZone Root Key after performing a [Device Erase](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

>**Note**: The TrustZone Root Key cannot be renewed if Device Erase is disabled.

The TrustZone Root Key is not exposable to the NSPE, and access to this key in SPE is different in HSE and VSE devices.

- HSE - The SPE can access the TrustZone Root Key as a built-in non-exportable key in HSE NVM.

- VSE - The SPE can access the TrustZone Root Key in Secure RAM, which is copied from VSE NVM during boot.

The TrustZone Root Key value after reset is identical to the value before reset. TrustZone Root Keys are unique on each device. The key allows a user to securely store a key in the Non-secure flash, limiting the number of keys that can be saved only by the amount of Non-secure storage. The following figure describes using the TrustZone Root Key to encrypt a plaintext key and store it in Non-secure NVM.

![root_encrypt](././resources/sld717-root-encrypt.png)

1. After power-on, the device's TrustZone Root Key is available for the SPE.

2. A user key is generated and imported into the device's Non-secure memory. In this example, the key is imported into Non-secure RAM for easy deletion, and the key is lost if device power is removed.

3. Call PSA Crypto API (`psa_import_key()` or `psa_generate_key()`) through SG in NSC to generate a key for crypto operations.

4. The plaintext key is passed to the PSA Crypto in SPE, where it is encrypted (AES-GCM) with the TrustZone Root Key.

5. The encrypted key is stored to the NVM in NSPE through the PSA ITS and NVM3 drivers.

6. The plaintext key can now be deleted from the Non-secure RAM.

7. Only the PSA Crypto in SPE can retrieve the encrypted key from NVM in NSPE and decrypt it for crypto operations in SPE.

>**Note**: Ignore steps 2 and 6 if the plaintext key is randomly generated by the PSA Crypto.

The following tables describe the storage differences between SVM and SVH devices with and without TrustZone Secure Key Storage (SKS).

<table class="classic"><colgroup><col width="40%"/><col width="30%"/><col width="30%"/></colgroup><thead><tr><th>Key Type</th><th>Storage on SVM Device</th><th>Storage on SVH Device</th></tr></thead><tbody><tr><td><p>Volatile Plaintext (without TrustZone SKS)</p></td><td><p>RAM</p></td><td><p>RAM</p></td></tr><tr><td><p>Persistent Plaintext (without TrustZone SKS)</p></td><td><p>NVM</p></td><td><p>NVM</p></td></tr><tr><td><p>Volatile Wrapped (without TrustZone SKS)</p></td><td><p>Not supported</p></td><td><p>RAM (1)</p></td></tr><tr><td><p>Persistent Wrapped (without TrustZone SKS)</p></td><td><p>Not supported</p></td><td><p>NVM (1)</p></td></tr></tbody></table>

<table class="classic"><colgroup><col width="40%"/><col width="30%"/><col width="30%"/></colgroup><thead><tr><th>Key Type</th><th>Storage on SVM Device</th><th>Storage on SVH Device</th></tr></thead><tbody><tr><td><p>Volatile Plaintext (with TrustZone SKS)</p></td><td><p>Secure RAM (2)</p></td><td><p>Secure RAM</p></td></tr><tr><td><p>Persistent Plaintext (with TrustZone SKS)</p></td><td><p>Encrypted plaintext key in NS NVM (2)</p></td><td><p>Encrypted plaintext key in NS NVM</p></td></tr><tr><td><p>Volatile Wrapped (with TrustZone SKS)</p></td><td><p>Not supported</p></td><td><p>Secure RAM</p></td></tr><tr><td><p>Persistent Wrapped (with TrustZone SKS)</p></td><td><p>Not supported</p></td><td><p>Encrypted wrapped key in NS NVM</p></td></tr></tbody></table>

**Notes**:

- The NVM or [NS NVM](#main-flash-layout) is at the last part of the main flash.

- It is possible to replace the [wrapped key](https://docs.silabs.com/iot-security/latest/efr32-secure-key-storage/) solution on the SVH device (1) with TrustZone Secure Key Storage on the SVM device (2), but this is a less secure approach.

## PSA Attestation

The device attestation service creates a token that contains a fixed set of device-specific data when requested by the caller. Each device must have a unique Initial Attestation Key (IAK) pair. The device uses the private IAK to sign the token, and the caller uses the public IAK to verify the token's authenticity.

The generation of the private IAK is different in SVM and SVH devices.

- SVM - If the private IAK does not exist in NVM3, it is randomly generated when requested from the [PSA Attestation](#secure-library) driver and saved to NVM3 through the [TrustZone Secure Key Storage](#trustzone-secure-key-storage).

- SVH - The private IAK is generated and securely stored in the HSE during chip production.

An Entity Attestation Token (EAT) is a mini-report that is cryptographically signed. An EAT is encoded in either one of two standardized data formats: a Concise Binary Object Representation ([CBOR](https://www.rfc-editor.org/info/rfc7049)) or in the text-based format JSON. A digital signature is then used to protect its content. The technical specification defining the content of the EAT, which are claims about the hardware and the software running on a device, is specified by the Internet Engineering Task Force ([IETF](https://datatracker.ietf.org/doc/html/draft-ietf-rats-eat-11)).

The EAT is a crypto-signed report card with claims. A claim is a data item that is represented as a Key-Value pair. Claims can relate to the device's pedigree or anything one wants the device to attest. Collected data can originate from the Root of Trust (RoT), any protected area, or non-protected areas.

The EAT must be signed following the structure of the CBOR Object Signing and Encryption ([COSE](https://www.rfc-editor.org/info/rfc8152)) specification. For asymmetric key algorithms, the signature structure must be COSE-Sign1. A COSE-Sign1 is a CBOR encoded, self-secured data blob that contains headers, a payload, and a signature.

The primary need for EAT verification is to check correct formation and signing as for any token. In addition, though, the verifier can operate a policy where values of some of the claims in this profile can be compared to reference values that are registered with the verifier for a given deployment, to confirm that the device is endorsed by the manufacturer supply chain.

The [PSA attestation token](https://www.ietf.org/archive/id/draft-tschofenig-rats-psa-token-21.html) (aka Initial Attestation Token - IAT) is a profiled EAT. The Series 2 device will generate this token by (Nonce claim below) unless the SE OTP is uninitialized or the [`SECURE_BOOT_ENABLE`](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/) option in SE OTP is disabled.

The following tables describe claims used in the PSA attestation token of the Series 2 device.

**Table**: Claims of PSA Attestation Token

<table class="classic"><colgroup><col width="10%"/><col width="30%"/><col width="40%"/><col width="20%"/></colgroup><thead><tr><th>Key</th><th>Claim Name (Present)</th><th>Claim Description</th><th>Claim Value</th></tr></thead><tbody><tr><td><p>265 (-75000)</p></td><td><p>Profile Definition (Must)</p></td><td><p>The Profile Definition claim encodes the unique identifier corresponds to the EAT profile.</p></td><td><p>http://arm.com/psa/2.0.0</p></td></tr><tr><td><p>2394 (-75001)</p></td><td><p>Client ID (Must)</p></td><td><p>The Client ID claim represents the security domain of the caller.</p></td><td><p>See note below (2 byes)</p></td></tr><tr><td><p>2395 (-75002)</p></td><td><p>Security Lifecycle (Must)</p></td><td><p>The Security Lifecycle claim represents the current lifecycle state of the PSA RoT.</p></td><td><p>Device dependent (2 bytes)</p></td></tr><tr><td><p>2396 (-75003)</p></td><td><p>Implementation ID (Must)</p></td><td><p>The Implementation ID claim uniquely identifies the implementation of the immutable PSA RoT.</p></td><td><p>Device dependent (32 bytes)</p></td></tr><tr><td><p>2397 (-75004)</p></td><td><p>Boot Seed (Optional)</p></td><td><p>The Boot Seed claim represents a value created at system boot time that will allow differentiation of reports from different boot sessions.</p></td><td><p>Device dependent (32 bytes)</p></td></tr><tr><td><p>2399 (-75006)</p></td><td><p>Software Components (Must)</p></td><td><p>The Software Components claim is a list of software components that includes all the software loaded by the PSA RoT.</p></td><td><p>See note below</p></td></tr><tr><td><p>10 (-75008)</p></td><td><p>Nonce (Must)</p></td><td><p>The Nonce claim is used to carry the challenge provided by the caller to demonstrate freshness of the generated token. The length must be either 32, 48, or 64 bytes.</p></td><td><p>Random nonce (32/48/64 bytes)</p></td></tr><tr><td><p>256 (-75009)</p></td><td><p>Instance ID (Must)</p></td><td><p>The Instance ID claim represents the unique identifier of the IAK. The length must be 33 bytes.</p></td><td><p>SHA-256 hash of public IAK (32 bytes) with header 0x01</p></td></tr></tbody></table>

**Notes**:

- Some claims MUST be present in a PSA attestation token.

- The keys `-7500x` were defined in a previous version of the PSA attestation token specification (`PSA_IOT_PROFILE_1` profile) that is still used in the HSE-SVH firmware.

- The actual claims returned from the tokens on the SVH device are HSE firmware version-dependent.

- Key 2394: In PSA, a security domain is represented by a signed integer where negative values represent callers from the NSPE and positive IDs represent callers from the SPE. The value 0 is not permitted.

- Key 2395 (For the definitions of these lifecycle states, refer to the ARM [PSA Security Model](https://developer.arm.com/-/media/Files/pdf/PlatformSecurityArchitecture/Architect/DEN0079_PSA_SM_ALPHA-03_RC01.pdf)):

  - UNKNOWN (`0x0000 - 0x00ff`)

  - ASSEMBLY_AND_TEST (`0x1000 - 0x10ff`)

  - PSA_ROT_PROVISIONING (`0x2000 - 0x20ff`)

  - SECURED (`0x3000 - 0x30ff`)

  - NON_PSA_ROT_DEBUG (`0x4000 - 0x40ff`)

  - RECOVERABLE_PSA_ROT_DEBUG (`0x5000 - 0x50ff`)

  - DECOMMISSIONED (`0x6000 - 0x60ff`)

- Key 2396:

  - Word[0]: Die revision

  - Word[1]: SE OTP version (return 0 for VSE SE firmware \< [v1.2.14](./06-r-migration.md#))

  - Word[2]: Security capability (not applicable to HSE-SVH device, always returns 1 in this word)

    - 0: Unknown security capability

    - 1: Security capability not applicable

    - 2: Basic security capability

    - 3: Root of Trust security capability

    - 4: HSE-SVM security capability

    - 5: HSE-SVH security capability (run HSE-SVM binary on HSE-SVH device)

  - Word[3]: Production version

  - Word[4:7]: Reserved (zeros)

- Key 2399: Each software component uses the attributes described in the following table, and some MUST be present in a software component claim.

<table class="classic"><colgroup><col width="10%"/><col width="30%"/><col width="40%"/><col width="20%"/></colgroup><thead><tr><th>Key</th><th>Attribute (Present)</th><th>Description</th><th>Value</th></tr></thead><tbody><tr><td><p>1</p></td><td><p>Measurement Type (Optional)</p></td><td><p>The Measurement Type attribute is a short string representing the role of this software component.</p></td><td><p>See note below</p></td></tr><tr><td><p>2</p></td><td><p>Measurement Value (Must)</p></td><td><p>The Measurement Value attribute represents a hash of the invariant software component in memory at startup time.</p></td><td><p>SHA-256 hash (32 bytes) of the firmware</p></td></tr><tr><td><p>4</p></td><td><p>Version (Optional)</p></td><td><p>The Version attribute is the issued software version in the form of a text string.</p></td><td><p>A string of 8 bytes</p></td></tr></tbody></table>

The following measurement types may be used for Key 1:

- "BL": a Bootloader

- "PRoT": a component of the PSA Root of Trust

- "ARoT": a component of the Application Root of Trust

- "App": a component of the NSPE application

- "TS": a component of a Trusted Subsystem

The PSA Attestation API allows access to the PSA attestation token, so an external entity can cryptographically verify the identity and trust status of the device.

**Table**: PSA Attestation API

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>PSA Attestation API</th><th>Usage</th></tr></thead><tbody><tr><td><p>psa_initial_attest_get_token</p></td><td><p>Retrieve the PSA attestation Token.</p></td></tr><tr><td><p>psa_initial_attest_get_token_size</p></td><td><p>Calculate the size of a PSA attestation Token.</p></td></tr><tr><td><p>sl_tz_attestation_get_public_key</p></td><td><p>Get the public IAK key for PSA attestation token signature verification.</p></td></tr></tbody></table>

>**Note**: The `sl_tz_attestation_get_public_key` is a Silicon Labs custom API.

## SE Manager

SE Manager is the foundation for the [Secure library](#secure-library) cryptographic operations on HSE devices. It means that SE Manager has to move into the SPE.

The following SE Manager [core](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager-core) APIs are always available in the NSPE.

<table class="classic"><colgroup><col width="40%"/><col width="20%"/><col width="20%"/><col width="20%"/></colgroup><thead><tr><th>SE Manager Core API</th><th>VSE-SVM</th><th>HSE-SVM</th><th>HSE-SVH</th></tr></thead><tbody><tr><td><p>sl_se_init</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_deinit</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_init_command_context</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_deinit_command_context</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_set_yield</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr></tbody></table>

The following SE Manager [core](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager-core) APIs expose to the NSPE through the NSC interface for the VSE devices.

<table class="classic"><colgroup><col width="40%"/><col width="20%"/><col width="20%"/><col width="20%"/></colgroup><thead><tr><th>SE Manager Core API</th><th>VSE-SVM</th><th>HSE-SVM</th><th>HSE-SVH</th></tr></thead><tbody><tr><td><p>sl_se_read_executed_command</p></td><td><p>Y</p></td><td><p>-</p></td><td><p>-</p></td></tr><tr><td><p>sl_se_ack_command</p></td><td><p>Y</p></td><td><p>-</p></td><td><p>-</p></td></tr></tbody></table>

The following SE Manager [utility](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager-util) APIs expose to the NSPE through the NSC interface for configuring the security features of HSE or VSE devices.

<table class="classic"><colgroup><col width="25%"/><col width="25%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>SE Manager Utility API</th><th>VSE-SVM</th><th>HSE-SVM</th><th>HSE-SVH</th></tr></thead><tbody><tr><td><p>sl_se_check_se_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_apply_se_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_upgrade_status_se_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_check_host_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_apply_host_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_upgrade_status_host_image</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_init_otp_key</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_read_pubkey</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_init_otp</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_read_otp</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_se_version</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_debug_lock_status</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_apply_debug_lock</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_otp_version</p></td><td><p>Y</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_write_user_data</p></td><td><p>-</p></td><td><p>Y (EFR32xG21 only)</p></td><td><p>Y (EFR32xG21 only)</p></td></tr><tr><td><p>sl_se_erase_user_data</p></td><td><p>-</p></td><td><p>Y (EFR32xG21 only)</p></td><td><p>Y (EFR32xG21 only)</p></td></tr><tr><td><p>sl_se_get_reset_cause</p></td><td><p>-</p></td><td><p>Y (EFR32xG21 only)</p></td><td><p>Y (EFR32xG21 only)</p></td></tr><tr><td><p>sl_se_get_status</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_serialnumber</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_enable_secure_debug</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_disable_secure_debug</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr></tbody></table>

<table class="classic"><colgroup><col width="25%"/><col width="25%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>SE Manager Utility API</th><th>VSE-SVM</th><th>HSE-SVM</th><th>HSE-SVH</th></tr></thead><tbody><tr><td><p>sl_se_set_debug_options</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_erase_device</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_disable_device_erase</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_get_challenge</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_roll_challenge</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_open_debug</p></td><td><p>-</p></td><td><p>Y</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_disable_tamper</p></td><td><p>-</p></td><td><p>-</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_read_cert_size</p></td><td><p>-</p></td><td><p>-</p></td><td><p>Y</p></td></tr><tr><td><p>sl_se_read_cert</p></td><td><p>-</p></td><td><p>-</p></td><td><p>Y</p></td></tr></tbody></table>


>**Note**: The NSPE cannot access the other [SE Manager APIs](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager) for cryptographic and attestation operations.

## Common Vulnerabilities and Exposures (CVE)

At this writing, the following known TrustZone CVE had been fixed in the current implementation.

- [CVE-2020-16273](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-16273): Stack sealing

- [CVE-2021-36465](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-35465): VLLDM instruction/floating-point vulnerability
