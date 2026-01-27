# Entity Attestation Token (EAT)

The device attestation service creates a token that contains a fixed set of device-specific data when requested from the caller. The device must contain an attestation key pair, which is unique per device, to sign the token. The HSE-SVH device uses the [Private Device Key](./01-series-2-security-features.md#key-reference) (aka attestation key) to sign the token, and the caller uses the Public Device Key to verify the token's authenticity.

An Entity Attestation Token (EAT) is a mini-report that is cryptographically signed. An EAT is encoded in either one of two standardized data formats: a Concise Binary Object Representation ([CBOR](https://www.rfc-editor.org/info/rfc7049)) or in the text-based format JSON. A digital signature is then used to protect its content. The technical specification defining the content of the EAT, which are claims about the hardware and the software running on a device, is specified by the Internet Engineering Task Force ([IETF](https://datatracker.ietf.org/doc/html/draft-ietf-rats-eat-11#ref-RATS.Architecture)).

An EAT is a collection of Key ID-Value pairs relating to device pedigree or any other information one wants the device to attest. Collected data can originate from the Root of Trust (RoT), any protected area, or non-protected areas.

The EAT must be signed following the structure of the CBOR Object Signing and Encryption ([COSE](https://www.rfc-editor.org/info/rfc8152)) specification. For asymmetric key algorithms, the signature structure must be COSE-Sign1. A COSE-Sign1 is a CBOR encoded, self-secured data blob that contains headers, a payload, and a signature.

The primary need for EAT verification is to check correct formatting and verify signatures as for any token. In addition, though, the verifier can operate a policy where values of some of the claims in this profile can be compared to reference values, registered with the verifier for a given deployment, to confirm that the device is endorsed by the manufacturer supply chain.

The HSE can generate the [PSA attestation token or security configuration token](./08-r-examples.md#entity-attestation-token-eat) when requested from the caller with a [challenge](./06-r-remoteauthenticationprocess.md) (Auth challenge claim below). The following tables describe EAT claims that are used in the [PSA attestation token](https://www.ietf.org/archive/id/draft-tschofenig-rats-psa-token-08.txt) and security configuration token.

>**Note**: The actual claims returned from the tokens are HSE firmware version dependent.

<table class="classic"><caption>Claims of PSA Attestation Token</caption><colgroup><col width="8%"><col width="15%"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Key ID</th><th>Claim</th><th>Description</th><th>Value</th></tr></thead><tbody><tr><td><p>-75000</p></td><td><p>Profile definition</p></td><td><p>Name of a document that describes the profile of the report.</p></td><td><p>PSA_IOT_PROFILE_1</p></td></tr><tr><td><p>-75001</p></td><td><p>Client ID</p></td><td><p>Represents the Partition ID of the caller.</p></td><td><p>See note below</p></td></tr><tr><td><p>-75002</p></td><td><p>Security lifecycle</p></td><td><p>Represents the current life cycle stage of the PSA RoT.</p></td><td><p>Device dependent</p></td></tr><tr><td><p>-75003</p></td><td><p>Implementation ID</p></td><td><p>Uniquely identifies the underlying immutable PSA RoT.</p></td><td><p>Device dependent (32 bytes)</p></td></tr><tr><td><p>-75004</p></td><td><p>Boot seed</p></td><td><p>Represents a random value created at system boot time.</p></td><td><p>Random bytes (32 bytes)</p></td></tr><tr><td><p>-75006</p></td><td><p>Software components</p></td><td><p>A list of Software components represents all the software loaded by PSA RoT.</p></td><td><p>See the software components table below.</p></td></tr><tr><td><p>-75008</p></td><td><p>Auth challenge</p></td><td><p>Input object from the caller. For example, this can be a cryptographic nonce or a hash of locally attested data. The length must be 32, 48, or 64 bytes.</p></td><td><p>Random bytes or hash (32/48/64 bytes)</p></td></tr><tr><td><p>-75009</p></td><td><p>Instance ID</p></td><td><p>Unique identifier of the instance.</p></td><td><p>Device EUI-64 unique ID with type byte 0x06 (9 bytes)</p></td></tr></tbody></table>

>**Note**:
>
> - Key ID 75001: Client ID if present. Otherwise the value 1 for a token requested by a secure bus master and -1 for a non-secure master.
>
> - Key ID 75002 (For the definitions of these lifecycle states, please refer to the ARM [Platform Security Model](https://developer.arm.com/documentation/den0128/0100/)):
>
>   - UNKNOWN (`0x0000`)
>
>   - ASSEMBLY_AND_TEST (`0x1000`)
>
>   - PSA_ROT_PROVISIONING (`0x2000`)
>
>   - SECURED (`0x3000`)
>
>   - NON_PSA_ROT_DEBUG (`0x4000`)
>
>   - RECOVERABLE_PSA_ROT_DEBUG (`0x5000`)
>
>   - DECOMMISSIONED (`0x6000`)
>
> - Key ID 75003:
>
>   - Word[0]: Die revision
>
>   - Word[1]: HSE OTP version
>
>   - Word[2]: Bit indicating it is an HSE-SVH device
>
>   - Word[3]: Production version
>
>   - Word[4:7]: Reserved (zeros)

<table class="classic"><caption>Software Components</caption><colgroup><col width="auto"><col width="auto"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Key ID</th><th>Type</th><th>Description</th><th>Value</th></tr></thead><tbody><tr><td><p>1</p></td><td><p>Measurement type</p></td><td><p>A short string represents the role of this software component.</p></td><td><p>See note below</p></td></tr><tr><td><p>2</p></td><td><p>Measurement value</p></td><td><p>Represents a hash of the invariant software component in memory at startup time.</p></td><td><p>See note below</p></td></tr><tr><td><p>4</p></td><td><p>Version</p></td><td><p>The issued software version is in the form of a text string.</p></td><td><p>See note below</p></td></tr></tbody></table>

**Notes**:

- Key ID 1:

  - HSE always exists — **PRoT**
  - If secure booted Gecko Bootloader exists at flash starting address — **BL**
  - If secure booted application exists at flash starting address — **ARoT**

- Key ID 2: SHA-256 hash (32 bytes) of the firmware (HSE, Gecko Bootloader, or application)

- Key ID 4: Version of the firmware (HSE, Gecko Bootloader, or application)

<table class="classic"><caption>Claims of Security Configuration Token</caption><colgroup><col width="8%"><col width="15%"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Key ID</th><th>Claim</th><th>Description</th><th>Value</th></tr></thead><tbody><tr><td><p>-75000</p></td><td><p>Profile definition</p></td><td><p>Name of a document that describes the profile of the report.</p></td><td><p>SILABS_1</p></td></tr><tr><td><p>-75008</p></td><td><p>Auth challenge</p></td><td><p>Input object from the caller. For example, this can be a cryptographic nonce or a hash of locally attested data. The length must be 32 bytes.</p></td><td><p>Random bytes or hash (32 bytes)</p></td></tr><tr><td><p>-75009</p></td><td><p>Instance ID</p></td><td><p>Unique identifier of the instance.</p></td><td><p>Device EUI-64 unique ID with type byte 0x06 (9 bytes)</p></td></tr><tr><td><p>-76000</p></td><td><p>SE status</p></td><td><p>Device HSE status.</p></td><td><p>Device dependent (36 bytes)</p></td></tr><tr><td><p>-76001</p></td><td><p>OTP configuration</p></td><td><p>Device HSE OTP configuration if provisioned.</p></td><td><p>Device dependent (24 bytes)</p></td></tr><tr><td><p>-76002</p></td><td><p>Sign Key</p></td><td><p>Public Sign Key in HSE OTP if provisioned.</p></td><td><p>Device dependent (64 bytes)</p></td></tr><tr><td><p>-76003</p></td><td><p>Command Key</p></td><td><p>Public Command Key in HSE OTP if provisioned.</p></td><td><p>Device dependent (64 bytes)</p></td></tr><tr><td><p>-76004</p></td><td><p>Tamper settings</p></td><td><p>Current applied tamper settings.</p></td><td><p>Device dependent (16 bytes)</p></td></tr></tbody></table>

**Notes**:

- All custom Silicon Labs claims will have a base of 76000.

- Key ID 76000: Refer to the *Get Status* section in [Programming Series 2 Devices using the Debug Challenge Interface (DCI) and Serial Wire Debug (SWD)](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/05-se-command-list#get-status) for the description (HSE-SVH) of the value.

- Key ID 76001: Refer to the *Read User Configuration* section in [Programming Series 2 Devices using the Debug Challenge Interface (DCI) and Serial Wire Debug (SWD)](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/05-se-command-list#read-user-configuration) for the description (HSE-SVH) of the value.

- Key ID 76002 and 76003: Refer to [Key Reference](./01-series-2-security-features.md#key-reference-key-reference) for Public Sign Key and Public Command Key.

- Key ID 76004: One nibble per tamper source. Refer to the *Anti-Tamper Configuration* section in [Programming Series 2 Devices using the Debug Challenge Interface (DCI) and Serial Wire Debug (SWD)](https://docs.silabs.com/iot-security/latest/efr32-dci-swd-programming/05-se-command-list#anti-tamper-configuration) for the description of the value.
