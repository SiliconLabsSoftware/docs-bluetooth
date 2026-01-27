# HSE Secure Key Storage

The following sections demonstrate three methods for key storage: ARM® TrustZone®, plaintext, and Secure Key Storage.

>**Note**: In the following examples, AES key usage is demonstrated. However, any other key types supported by the device can also be used for key storage.

## Key Generation and Usage

In HSE-SVH devices, cryptographic functions are performed by the HSE. In order to perform these functions, the HSE must have access to any user keys needed. Keys can be generated and used by the HSE in multiple ways:

1. External storage, in-place usage:

   1. A user generates a plaintext key and stores it in device memory.

   2. The user provides a key descriptor to the HSE that points to this key for a specific cryptographic operation.

   3. The HSE performs the cryptographic operation using this key, but does not store it in any HSE volatile storage slot.

2. External storage with HSE import:

   1. A user generates a plaintext key and stores it in device memory.

   2. The user provides a key descriptor to the HSE that points to this key, as well as a slot number to store the key.

   3. The HSE imports this key into a volatile key storage slot or can optionally save it in wrapped form in device memory.

   4. The user requests that the HSE performs a cryptographic function by providing the index of the storage slot or a pointer to the wrapped key in device memory.

3. Internal HSE key generation:

   1. The user commands the HSE to generate a new key within one of the HSE's volatile key slots or can optionally save it in wrapped form in device memory.

   2. The user requests that the HSE performs a cryptographic function by providing the index of the storage slot or a pointer to the wrapped key in device memory.

**Notes**:

- In each case, to provide persistent storage for the key, the key must be stored in non-volatile memory.

- [Plain Key Storage](#plaintext-key-storage) and [Secure Key Storage](#secure-key-storage) provide details on key generation and usage with HSE-SVH device.

## Plaintext Key Storage

### Plaintext Key Import

The simplest manner to store a key is to save it in plaintext form. The steps to store and use a key stored in plaintext form are as follows:

1. A user key is generated and imported into device memory. For persistent storage, this must be non-volatile storage, such as device flash.

2. After a device reset, the HSE volatile key storage will be empty. The plaintext key is imported ([method 2](#key-generation-and-usage)) into a slot for usage. Alternatively, the key could be used in place ([method 1](#key-generation-and-usage)) from non-volatile storage on a per-operation basis.

   ![Plaintext Key Import](./resources/sld791-plaintext-key-import.png)

### Plaintext Key Usage

In order to use the key for a cryptographic operation, the following procedure is used.

1. The user passes data to be processed (in this specific example, AES encrypted data) to the HSE.

2. The user requests that a cryptographic operation be performed on this data using one of the keys stored in the HSE volatile key storage slots ([method 2](#key-generation-and-usage)). Alternatively, the key can be passed to the HSE directly for a singular cryptographic operation ([method 1](#key-generation-and-usage)).

3. The HSE performs the cryptographic operation.

4. The output of the cryptographic operation is passed back to the user for processing.

   ![Plaintext Key Usage](./resources/sld791-plaintext-key-usage.png)

This method exposes the keys to two major vulnerabilities:

1. Access to device storage gives access to the keys. In this case, an attack that gains access to the flash contents will expose the user key.

2. Since the application has access to the keys, compromising the application or device privileges can compromise the keys. Such an attack might not directly access device memory, but take control of the application in a way that causes the application to expose the key to an attacker.

## Secure Key Storage

With Secure Key Storage, the user key, using the HSE, can be accessed in an encrypted, or 'wrapped' form. Only the HSE has access to the HSE root key used to decrypt, or 'unwrap', the wrapped key. This HSE root key is not stored on the device during power-down, but rather reconstructed after each reset. Key wrapping allows a user to securely store a key in non-volatile memory, limiting the number of keys that can be stored only by the amount of storage the user has available.

>**Note**: The reconstructed root key after each reset is IDENTICAL and UNIQUE on each HSE-SVH device.

### Wrap an External Key

To wrap an externally-generated key:

1. After power-on, the device's unique root key is reconstructed with output from the Physically Unclonable Function (PUF).

2. A user key is generated and imported into device memory. In this example, the key is imported into RAM for easy deletion, and the added security that, if device power is removed, the key will be lost.

3. The user key is passed to the HSE, where it is encrypted with the HSE's root key.

4. The wrapped key is passed back to the user application for storage in non-volatile memory (in this case, device flash).

5. The plaintext key can now be deleted from the device. From this point forward, only the HSE will have access to the plaintext key.

   ![External Key Import, Wrapping, and Storage](./resources/sld791-external-key-import-wrapping-and-storage.png)

### Generate an Internal Wrapped Key

Instead of importing an external key, the HSE can generate a new key directly into one of its volatile key storage slots. This key can then be exported in wrapped form for secure persistent storage.

1. The user requests that the HSE generates a new key into one of its storage slots using the True Random Number Generator (TRNG).

2. The key is encrypted with the HSE's root key.

3. The wrapped key is passed back to the user application for non-volatile storage (flash, in this case).

   ![Internally Generated Key Wrapping and Storage](./resources/sld791-internally-generated-key-wrapping-and-storage.png)

### Wrapped Key Import

In order to import a wrapped key into the HSE for usage:

1. The wrapped key is passed to the HSE.

2. The wrapped key is decrypted ("unwrapped") with the HSE's root key.

3. The plaintext key is stored in a volatile key storage slot.

   ![Wrapped Key Import](./resources/sld791-wrapped-key-import.png)

### Wrapped Key Usage

In order to use the key for a cryptographic operation, the same steps are followed as when using a plaintext key that has been imported into the HSE:

1. The user passes data to be processed (in this specific example, AES encrypted data) to the HSE.

2. The user requests that a cryptographic operation be performed on this data using one of the keys stored in the HSE volatile key storage slots. Alternatively, the wrapped key can be passed to the HSE directly for a singular cryptographic operation. In this case, the key will be unwrapped before being used, but will not be stored for future operations.

3. The HSE performs the cryptographic operation.

4. The output of the cryptographic operation is passed back to the user for processing.

   ![Wrapped Key Usage](./resources/sld791-wrapped-key-usage.png)

## Secure Key Storage Advantages

Secure Key Storage confers the following benefits over other key storage methods:

1. Access to device memory does not expose user keys.

2. Compromising the user application does not expose user keys, since the user application itself does not have access to the plaintext keys.

3. The number of user keys that can be securely stored is only limited by the amount of storage available to the user, including external storage.

## Operation Details

### Root Key Generation

Secure Key Storage depends on the HSE to encrypt / decrypt (wrap / unwrap) user keys with its own symmetric root key. The symmetric key used for this wrapping and unwrapping must be highly secure as it can expose all other key material in the system. The HSE key Management system uses a Physically Unclonable Function (PUF) to generate a persistent device-unique seed on power up to dynamically reconstruct this critical root key. The key is only visible to the AES encryption engine, and it is not retained when the device loses power.

### Access a Wrapped Key

By default, a key in an HSE storage slot can be exported to the application as a plaintext key. To prevent this, the user can use the key descriptor to set a user key to [non-exportable](./05-r-securekeystorageimplementation.md#secure-key-storage-implementations). This option prevents any request to export the wrapped key in plaintext from HSE, so the user application can only access the key encrypted by the HSE's root key. The HSE also tags the key with information to identify the wrapped key. Since only the HSE can access the root key to unwrap the user key, the plaintext key is non-accessible to the user application.

>**Note**: Wrapped keys are slightly larger than the equivalent plaintext key, as some additional metadata is required to identify the wrapped key to the HSE.

### Wrapped Key Storage and Usage

Once a key has been wrapped, it can be safely stored anywhere - device flash, RAM, external storage, etc. The number of keys that can be securely stored is only limited by the available storage space. A wrapped key can later be imported into a HSE volatile storage slot for usage, or used in-place. Once the key is wrapped and stored, the plaintext key available to the application can be deleted. From here, only the HSE will have the ability to unwrap and use the key.

With access to the wrapped key, the HSE can use this key in one of two ways:

1. A user can request that a cryptographic operation be performed using the key stored in memory. In this case, the HSE will import the key, unwrap it, and then perform the cryptographic operation. The key will not be stored within the HSE.

2. A user can import the wrapped key into a HSE volatile storage slot. In this case, the key is unwrapped by the HSE and stored in plaintext in a volatile slot. The user can then later request that a cryptographic function be performed by the HSE by referencing the volatile slot index. This provides a performance increase over using wrapped keys in place, as the HSE does not need to import and unwrap the key on each requested operation.

### Password Protection

When defining a key descriptor for a new key, or when importing an existing key into HSE, the user can choose to require a password to allow use of the key. The password field in the key descriptor structure is eight bytes in length. If unspecified, the key will use the default password of all zeros.

After importing a key with a password, failing to provide the correct password when performing a cryptographic operation will result in HSE returning an invalid credentials error, and no operation will be performed.
