# Secure Key Storage Implementations

Users can use Secure Engine Manager (SE Manager) or PSA Crypto in the following figure to access the secure key storage on HSE-SVH devices. SE Manager APIs for secure key storage and crypto are usually not considered external APIs. PSA Crypto API abstracts the entropy sources, crypto primitives, and even advanced security features like secure key storage from the calling functions.

Silicon Labs recommends using PSA Crypto API for secure key storage and cryptography whenever possible. It makes the solution more portable and hardware agnostic. In some cases, however, setting up tamper and initializing the secure boot can only be implemented by the SE Manager APIs.

![Secure Engine Manager and PSA Crypto](./resources/sld791-secure-engine-manager-and-psa-crypto.png)

<table class="classic"><colgroup><col width="20%"><col width="80%"></colgroup><thead><tr><th>Component</th><th>Functionality</th></tr></thead><tbody><tr><td><p>EMLIB (em_se.c)</p></td><td><p>Abstracts the mailbox interface: how to construct, send and receive low-level HSE mailbox commands.</p></td></tr><tr><td><p>SE Manager</p></td><td><p>On top of EMLIB, it abstracts the HSE command set: translates function calls into mailbox messages. The SE Manager also provides thread synchronization.</p></td></tr><tr><td><p>PSA Accelerator Drivers</p></td><td><p>A translation layer to map the PSA Crypto HSE interface and crypto acceleration calls to SE Manager calls.</p></td></tr><tr><td><p>PSA Crypto API</p></td><td><p>Platform independent cryptographic hardware acceleration support by implementing standardized APIs.</p></td></tr><tr><td><p>PSA ITS Driver</p></td><td><p>The key management functionality in PSA Crypto needs access to non-volatile memory for persistent storage of plaintext or wrapped keys. NVM3 gets wrapped by this translation layer, mapping the PSA ITS (Internal Trusted Storage) interface to NVM3 calls.</p></td></tr></tbody></table>

For the SE's mailbox interface, see section *Secure Engine Subsystem* in [Series 2 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/03-r-secureelement).

For more information about NVM3, see [https://docs.silabs.com/gecko-platform/latest/driver/api/group-nvm3](https://docs.silabs.com/gecko-platform/latest/driver/api/group-nvm3).

For more information about PSA Crypto, see [Integrating Crypto Functionality Using PSA Crypto Compared to Mbed TLS](https://docs.silabs.com/iot-security/latest/mbedtls-psa-crypto-porting-guide/).

## SE Manager API

The following table lists the SE Manager APIs related to Secure Key Storage operations. The SE Manager API document can be found at [https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager).

<table class="classic"><colgroup><col width="16%"><col width="auto"></colgroup><thead><tr><th>SE Manager API</th><th>Usage</th></tr></thead><tbody><tr><td><p>sl_se_generate_key</p></td><td><p>Generate a new key and store it either in a volatile HSE storage slot or as a wrapped key.</p></td></tr><tr><td><p>sl_se_import_key</p></td><td><p>Import a plaintext key and store it either in a volatile HSE storage slot or as a wrapped key.</p></td></tr><tr><td><p>sl_se_export_key</p></td><td><p>Export a volatile or wrapped key back to plaintext if allowed. It will fail for a key that has been flagged as SL_SE_KEY_FLAG_NON_EXPORTABLE.</p></td></tr><tr><td><p>sl_se_transfer_key</p></td><td><p>Transfer a volatile or wrapped key to another storage option (volatile HSE storage slot or a wrapped key) if allowed.</p></td></tr><tr><td><p>sl_se_delete_key</p></td><td><p>Delete a key from a volatile HSE storage slot.</p></td></tr></tbody></table>

## PSA Crypto API

The following table lists the PSA Crypto APIs related to Secure Key Storage operations. The PSA Crypto API document can be found at [https://docs.silabs.com/mbed-tls/latest/](https://docs.silabs.com/mbed-tls/latest/).

For more information about PSA Crypto APIs on Secure Key Storage, see [Integrating Crypto Functionality Using PSA Crypto Compared to Mbed TLS](https://docs.silabs.com/iot-security/latest/mbedtls-psa-crypto-porting-guide/).

<table class="classic"><colgroup><col width="16%"><col width="auto"></colgroup><thead><tr><th>PSA Crypto API</th><th>Usage</th></tr></thead><tbody><tr><td><p>psa_generate_key</p></td><td><p>Generate a new plaintext or wrapped key and store it either in volatile or non-volatile memory.</p></td></tr><tr><td><p>psa_import_key</p></td><td><p>Import a plaintext key and save it in plaintext or wrapped form. It can store either in volatile or non-volatile memory.</p></td></tr><tr><td><p>psa_export_key</p></td><td><p>Export a key back to plaintext if allowed. The policy on the key must have the usage flag<code>PSA_KEY_USAGE_EXPORT</code>set.</p></td></tr><tr><td><p>psa_copy_key</p></td><td><p>Copy key material from one location to another, which may have a different lifetime (e.g., volatile to non-volatile).</p></td></tr><tr><td><p>psa_destroy_key</p></td><td><p>Destroy a key from both volatile memory and, if applicable, non-volatile storage.</p></td></tr></tbody></table>

## SE Manager API Versus PSA Crypto API

The following table compares the SE Manager APIs with PSA Crypto APIs on Secure Key Storage.

<table class="classic"><colgroup><col width="auto"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Item</th><th>SE Manager API</th><th>PSA Crypto API</th></tr></thead><tbody><tr><td><p>Availability</p></td><td><p>Only on HSE devices</p></td><td><p>Platform independent</p></td></tr><tr><td><p>API</p></td><td><p>Silicon Labs proprietary</p></td><td><p>Standardized by ARM®</p></td></tr><tr><td><p>Key Storage</p></td><td><p>Volatile (RAM) memory only</p></td><td><p>Volatile (RAM) or non-volatile (flash) memory</p></td></tr><tr><td><p>Wrapped Key Cache</p></td><td><p>Can use a volatile HSE storage slot</p></td><td><p>Not yet implemented</p></td></tr><tr><td><p>Password Protection</p></td><td><p>Can define in a key descriptor</p></td><td><p>Not yet defined in PSA Crypto</p></td></tr><tr><td><p>Custom ECC Curve</p></td><td><p>Can define in a key descriptor</p></td><td><p>Not yet defined in PSA Crypto</p></td></tr></tbody></table>

## PSA Crypto Key Types with TrustZone Secure Key Storage

The following tables describes the storage differences between key storage with and without TrustZone on SVM and SVH devices.

**Table: TrustZone Secure Key Storage (SKS) on SVM Devices**

<table class="classic"><colgroup><col width="auto"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Key Type</th><th>Storage without TrustZone SKS</th><th>Storage with TrustZone SKS</th></tr></thead><tbody><tr><td><p>Volatile Plaintext</p></td><td><p>RAM</p></td><td><p>Secure RAM (2)</p></td></tr><tr><td><p>Persistent Plaintext</p></td><td><p>NVM</p></td><td><p>Encrypted in NS NVM (2)</p></td></tr><tr><td><p>Volatile Wrapped</p></td><td><p>Not supported</p></td><td><p>Not supported</p></td></tr><tr><td><p>Persistent Wrapped</p></td><td><p>Not supported</p></td><td><p>Not supported</p></td></tr></tbody></table>

**Table: TrustZone Secure Key Storage (SKS) on SVH Devices**

<table class="classic"><colgroup><col width="auto"><col width="auto"><col width="auto"></colgroup><thead><tr><th>Key Type</th><th>Storage without TrustZone SKS</th><th>Storage with TrustZone SKS</th></tr></thead><tbody><tr><td><p>Volatile Plaintext</p></td><td><p>Plaintext key in RAM</p></td><td><p>Plaintext key in Secure RAM</p></td></tr><tr><td><p>Persistent Plaintext</p></td><td><p>Plaintext key in NVM</p></td><td><p>Encrypted plaintext key in NS NVM</p></td></tr><tr><td><p>Volatile Wrapped</p></td><td><p>Wrapped key in RAM (1)</p></td><td><p>Wrapped key in Secure RAM</p></td></tr><tr><td><p>Persistent Wrapped</p></td><td><p>Wrapped key in NVM (1)</p></td><td><p>Encrypted wrapped key in NS NVM</p></td></tr></tbody></table>

**Notes**:

- The NVM or NS NVM is at the last part of the main flash.

- It is possible to replace the wrapped key solution on the SVH device (1) with TrustZone Secure Key Storage on the SVM device (2), but this is a less secure approach.
