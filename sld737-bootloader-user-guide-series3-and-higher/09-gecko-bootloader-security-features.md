# Gecko Bootloader Security Features

## About Bootloader Image Security

Secure Boot and Secure Firmware Upgrade, discussed in the following sections, enables Gecko Bootloader to provide authenticity and integrity checks on the Application image, which provides a sufficient level of security for many applications. However, in systems without a hardware root of trust, no process checks the authenticity or integrity of the Gecko Bootloader itself. Its security is provided solely by the device hardware and the robustness of the software running on the device.

The native behavior of Firmware Upgrade will prevent accidental version rollback of Gecko Bootloader under normal usage conditions.

## About Application Image Security

The Gecko Bootloader can enforce security on two levels:

- Secure Boot refers to the verification of the authenticity of the application image in main flash on every boot of the device.

- Secure Firmware Upgrade refers to the verification of the authenticity of an upgrade image before performing a bootload, and optionally enforcing that upgrade images are encrypted.

### Secure Boot Procedure

When Secure Boot is enabled, the cryptographic signature of the application image in flash is verified on every boot before the application is allowed to run. Secure Boot is not enabled by default in the example configurations provided by Silicon Labs, but enabling it is highly recommended to ensure the validity and integrity of firmware images.

**Signature Algorithms**

The Gecko Bootloader supports the ECDSA-P256-SHA256 cryptographic signature algorithm. This is the ECDSA (elliptical curve digital signature algorithm) of the SHA-256 digest of the application firmware image, using the NIST P-256 (secp256r1) curve.

**Summary of Operation**

1. On boot, the bootloader checks the application image for information about whether it is signed.

2. The type of signature and signature location is determined.

3. If the type of signature does not match the requirements of the bootloader, the bootloader enters device firmware upgrade mode and prevents the application from running.

4. According to the chosen signature algorithm, the signature of the contents of flash from the beginning of the application to the location of the signature is compared to the signature at the signature location.

5. If the signatures do not match, the bootloader enters device firmware upgrade mode and prevents the application from running.

**Secure Boot using ECDSA-P256-SHA256**

For an image to be signed for Secure Boot, the application needs to contain a copy of the **ApplicationProperties\_t** struct. This struct contains information about which signature algorithm is used, and where to find the signature.

On every boot, the bootloader calculates the SHA-256 digest of the application image, from the beginning of the application to the start of the signature. The signature of the SHA-256 digest is then verified using ECDSA-P256.

If the signature is valid, the application is allowed to boot. Else, the bootloader is entered, and an application upgrade is attempted if one is available.

Simplicity Commander can be used to generate a key pair and write the public key to the device. See the [Simplicity Commander User Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/) for more information.

**Secure Boot with Application Rollback Protection**

On every boot, the application version in the **ApplicationData\_t** struct is stored at the end of the bootloader area in flash, which is used to prevent applications from being downgraded. The application version can remain the same for upgrades.

If application rollback prevention component is installed in the bootloader project, then before applying the upgrade image, the bootloader will check the application version in the application properties structure that resides inside the signed/encrypted GBL file and will only apply the OTA image to application area if the application version in ApplicationData structure is equal or higher than the highest application version last seen.

The application rollback prevention feature can be enabled in the **Bootloader Core** component by selecting the **Enable application rollback protection** option. The **Minimum application version allowed** option can be used to configure the minimum application version that should be allowed to boot.

The application versions are stored in the SE OTP area.

**Secure Boot Using a Certificate**

On Series 3 devices, a certificate-based secure boot operation is supported. The Certificate contains:

- Struct version: The version of the certificate structure.

- Public key: ECDSA-P256 public key, X and Y coordinates concatenated, used to validate the image.

- Certificate version: The version of the running certificate.

- Signature: ECDSA-P256 signature, used for the authentication of the public key and the certificate version.

The definition of the certificate struct can be found in `api/application_properties.h`.

To utilize certificate-based secure boot, configure Secure Engine to authenticate the bootloader image by configuring the certificate-based secure boot option in the Secure Engine OTP. Configure the Gecko Bootloader to enable certificate-based secure boot in the **Bootloader Core** component by selecting the **Enable certificate support** option. The Gecko Bootloader certificate must be signed by the private key pair of the public key stored in the Secure Engine OTP. For more information on the key storage, see the *Key Storage* section.

The certificate-based secure boot procedure is illustrated in the following figure.

![Certificate-Based Secure Boot Procedure](resources/sld737-image19.jpeg)

Once the certificate-based secure boot option on Secure Engine is turned on, Secure Engine verifies the Gecko Bootloader certificate. The public key stored in the certificate is used to validate the signature of the Gecko Bootloader. Secure Engine will not accept bootloader images without a certificate.

If only the secure boot option is enabled (not certificate-based) on Secure Engine, and Secure Engine identifies a certificate, the certificate will be used to validate the bootloader image. If the certificate version from the bootloader image is higher than 0 and it gets verified once, Secure Engine will never again accept direct signed bootloader images without a certificate.

The Gecko Bootloader will authenticate the direct signed application using the public key stored in the Gecko Bootloader certificate. If the application contains a certificate, Gecko Bootloader will authenticate it. The procedure is illustrated in the following figure.

![Advanced Certificate-Based Secure Boot Procedure](resources/sld737-image20.jpeg)

After authentication of the application certificate, Gecko Bootloader verifies the signature of the application using the public key from the application certificate. In addition, Gecko Bootloader compares the Gecko Bootloader certificate version against the application certificate version. All application images with certificate version lower than the certificate version of the Gecko Bootloader will be rejected. Gecko Bootloader can be configured to only allow applications with certificates to boot by configuring the **Bootloader Core** component by selecting the **Reject direct signed images** option.

The **ApplicationProperties\_t** struct contains the certificate struct **ApplicationCertificate\_t**. The certificate struct can be injected to images that contain an **ApplicationProperties\_t** with **ApplicationCertificate\_t**. To inject a certificate to an image, issue the following command from Simplicity Commander:

```C
commander convert <image file> --secureboot --keyfile <signing key> --certificate <certificate>
--outfile <signed image file with certificate>
```

### Secure Firmware Upgrade

The Gecko Bootloader supports a secure firmware upgrade process. This is achieved by using symmetric encryption to encrypt the upgrade image, and asymmetric cryptography to sign the upgrade image. Symmetric encryption provides confidentiality, and asymmetric cryptography provides integrity and authenticity. Note that encryption alone is not enough to provide authenticity.

**Encryption Algorithms**

The Gecko Bootloader supports the AES-CTR-128 encryption algorithm. The GBL upgrade file is encrypted using 128-bit AES in Counter mode with a random nonce as the initial counter value.

To make use of the OTA decryption key stored in the Secure Engine OTP, the **Use symmetric key stored in Secure Engine storage** option in the **Bootloader Core** component must be selected. Simplicity Commander can be used to generate an OTA decryption key and write the key to the device. For more information on storing the OTA decryption key on Series 3 devices, see [Simplicity Commander User Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

The Secure Engine OTP key support depends on the SE Manager component, which is enabled by default.

**Signature Algorithms**

The Gecko Bootloader supports the ECDSA-P256-SHA256 cryptographic signature algorithm. This is the ECDSA signature of the SHA-256 digest of the GBL upgrade file, using the NIST P-256 (secp256r1) curve.

**Summary of Operation**

Before starting a firmware upgrade process, the application can verify an image in storage by calling into the bootloader verification functions. For bootloaders with Communication Interface, the host device should verify the image before sending it to the NCP or RCP.

During firmware upgrade, the GBL file is parsed, and if encrypted, decrypted on-the-fly. A GBL Signature Tag in the GBL file indicates to the bootloader that the file is signed, and the signature is verified. If signature verification fails, the firmware upgrade process is aborted.

On Series 3 devices, Gecko Bootloader will authenticate the GBL signature tag using the public key stored in the bootloader certificate if the **Enable certificate support** option is selected in the **Bootloader Core** component. A GBL Certificate tag in the GBL file indicates to the bootloader that the GBL certificate tag needs to be authenticated using the public key stored in the bootloader certificate. The certificate version in the GBL certificate tag is compared with the bootloader certificate and only a version equal or higher than the bootloader certificate is accepted. Once the GBL certificate tag is authenticated, the GBL file's signature is verified using the authenticated public key from the GBL certificate tag.

## Using Application Image Security Features

This example assumes that a bootloader called **bootloader-uart-xmodem** has been built in Simplicity Studio.The relevant version can be flashed to the device using the Flash Programmer in Simplicity Studio or using Simplicity Commander.

This example provides two ways of signing the upgrade images. The first option uses Simplicity Commander to generate key material and sign data. This is suitable for development. The second option uses an external signer, such as a dedicated Hardware Security Module (HSM) to protect private key material and perform signing operations. Silicon Labs recommends using an HSM to safeguard private keys.

### Generating Keys

To use the security features of the Gecko Bootloader, encryption and signing keys need to be generated. These keys must then be written to the device. The encryption key is used with the GBL file for secure firmware upgrade. The signing keys are used both with the GBL file for secure firmware upgrade and to sign the application image for Secure Boot.

**Generating a Signing Key Using Simplicity Commander**

```C
commander util genkey --type ecc-p256 --privkey signing-key --pubkey signing-key.pub
```

This creates an ECDSA-P256 key pair for signing; `signing-key` contains the private key in PEM format and **must be kept secret from third parties**. This key will later be used to sign images and GBL files. `signing-key.pub` contains the public key in PEM format and can be used to verify GBL files using commander gbl4 info.

**Generating a Signing Key Using a Hardware Security Module**

When using a Hardware Security Module, the private key is kept secret inside the HSM. According to the instructions from your HSM vendor, have it generate an ECDSA-P256 key pair and export the public key in PEM format to the file **signing-key.pub**. Then use Simplicity Commander to convert the key to token format, suitable for writing to the Series 3 device.

```C
commander gbl keyconvert --type ecc-p256 signing-key.pub
```

**Generating an Encryption Key**

```C
commander util genkey --type aes-ccm --outfile encryption-key
```

This creates an AES-128 key for encryption in the file **encryption-key**.

**Writing Keys to the Device**

>**Note**: Refer to the sections, *Writing the AES Decryption Key* and *Writing the Public Key to the Device* in the [Simplicity Commander User Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

### Signing an Application Image for Secure Boot

If the bootloader enforces Secure Boot, the application needs to be signed to pass verification. On every boot, a SHA-256 digest of the application is calculated. The signature is verified using ECDSA-P256, with the same public key as for the GBL file signing. Signature verification failure prevents the application from booting.

Application images should contain an **ApplicationProperties\_t** struct declaring the application version, capabilities, and other metadata. If **ApplicationProperties\_t** is missing, the application image cannot be signed. For more details on adding **ApplicationProperties\_t**, see the *Application Properties* section on the *Application Interface* page.

**Using Simplicity Commander**

Signing the application can be done with the command:

```C
commander convert myapp.s37 --secureboot --keyfile signing-key --outfile myapp-signed.s37
```

**Using a Hardware Security Module**

The application can be prepared for signing by issuing the command:

```C
commander convert myapp.s37 --secureboot --extsign --outfile myapp-for-signing.s37
```

Using an HSM, sign the output file **myapp-for-signing.s37**, and supply the resulting DER-formatted signature file **signature.der** back to Simplicity Commander:

```C
commander convert myapp-for-signing.s37 --secureboot --signature signature.der --verify signing-key.pub

--outfile myapp-signed.s37
```

### Creating a Signed and Encrypted GBL Upgrade Image File from an Application

To create a GBL file from an application, use `commander gbl4 create`.

Note that, as of this writing, secure application images can only be constructed through Simplicity Commander, not through the configuration options available through Simplicity Studio.

Application images should contain an **ApplicationProperties\_t** struct declaring the application version, capabilities, and other metadata. If **ApplicationProperties\_t** is missing, the application image cannot be signed. For more details on adding **ApplicationProperties\_t**, see the *Application Properties* section on the *Application Interface* page.

**Using Simplicity Commander to Sign**

For an application called **myapp.s37**, use:

```C
commander gbl4 create myapp.gbl -–config configfile.yaml

```

Example of a config file is as follows:

```C
manifest:
 product_id: "[16 B ID] (default from app or 00000000000000000000000000000000)"
 bundle_version: "[version] (default 0x00000000)"
 min_version: "[version] (default 0x00000000)"
 hash_function: "[none | sha256] (default sha256)"
 certificate: "[path to file] (optional)"
 signing_key: "[path to file] (optional)"
se_upgrade:
 se_file: "[path to file] (optional)"
updates:
 - data: <path to file>
  app_type: "[uint32] (default from app or 0x00000000)"
  app_capabilities: "[uint32] (default from app or 0x00000000)"
  version: "[uint32] (default from app or 0x00000000)"
  compression_scheme: "[none | lz4 | lzma] (default none)"
  encryption_scheme: "[none | aes-ctr-128] (default none)"
  block_size: "[size in bytes] (default 0/disabled)"
```

This single command performs three actions:

- Creates a GBL file

- Encrypts the GBL file

- Signs the GBL file

If Secure Boot is also desired, the application must be signed using `commander convert --secureboot` prior to creating the GBL.

## System Security Considerations

The Gecko bootloader security features can be used to create a secure device, but do not create a secure system by themselves. This section goes over considerations that need to be taken when designing a secure system where the Gecko Bootloader is a component.

### Key Storage

On Series 3 devices, the decryption key and the sign key used by the Gecko Bootloader in the Secure Engine OTP. The decryption key can be provisioned in the Secure Engine OTP using Simplicity Commander or using the Secure Engine Mailbox interface. Once a key value has been programmed into the Secure Engine OTP, it cannot be changed.
