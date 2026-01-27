# Key Provisioning

## Overview

The symmetric GBL Decryption Key is used to decrypt GBL files. All encrypted GBL images on this device must be encrypted with the same 128-bit AES key. *Provisioning the GBL Decryption Key in Simplicity Commander* below describes different ways to program the GBL Decryption Key to the Series 2 and Series 3 devices.

If the Secure Boot feature is to be used, the Public Sign Key must be provisioned to the device.

If the Secure Debug or tamper feature is to be used, the Public Command Key must be provisioned to the device.

The GBL Decryption Key (HSE device), Public Sign Key, and the Public Command Key are written to one-time-programmable (OTP) memory. Once written, they cannot be changed.

> **Note**: Silicon Labs strongly recommends provisioning these keys for future-proofing even if the device does not use the GBL Encryption, Secure Boot, and Secure Debug features.

## Provisioning the GBL Decryption Key in Simplicity Commander

To generate the text file for the GBL Decryption Key, run the command:

```sh
commander util genkey --type aes-ccm --outfile aes_key.txt
```

```sh
Using Windows' Cryptographic random number generator
DONE
```

where `aes_key.txt` contains the randomly generated AES-128 key.

To write the GBL Decryption Key to the HSE device, run the command:

```sh
commander security writekey --decrypt aes_key.txt --device EFR32MG21A010F1024 --serialno 440048205
```

This command can be executed only once per device.

```sh
Device has serial number 000000000000000014b457fffe045a8e

================================================================================
Please look through any warnings before proceeding.
THIS IS A ONE-TIME command, any encrypting of GBL files must be done with this key.
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
DONE
```

> **Note**: The GBL Decryption Key cannot be read back from the HSE OTP.

To write the GBL Decryption Key to the `Application Properties` struct of the GBL, run the command:

```sh
commander convert bootloader-uart-xmodem.s37 --aeskey aes_key.txt --outfile bootloader-uart-xmodem.s37
```

```sh
Parsing file bootloader-uart-xmodem.s37...
Writing to bootloader-uart-xmodem.s37...
Overwriting file: bootloader-uart-xmodem.s37...
DONE
```

where `bootloader-uart-xmodem.s37` is the GBL image file.

**Notes**:

- The `--aeskey` option for the convert command requires Simplicity Commander **v1.12.3** or above.

- The GBL Decryption Key can only be added to the GBL with `Application Properties` struct **v1.2** or higher (GSDK ≥ v4.1.0 or with SiSDK).

- This procedure must be implemented before signing the GBL image for Secure Boot.

To write the GBL Decryption Key to the top page of the main flash of a Series 2 VSE device, run the command:

```sh
commander flash --tokengroup znet --tokenfile aes_key.txt --device EFR32MG22C224F512 --serialno 440048205
```

```sh
Writing 8192 bytes starting at address 0x0007e000
Comparing range 0x0007E000 - 0x0007FFFF (8 KB)
Erasing range 0x0007E000 - 0x0007FFFF (1 sector, 8 KB)
Programming range 0x0007E000 - 0x0007FFFF (8 KB)
DONE
```

> **Note**: The MCU Series 2 devices (like EFM32PG22C200F512IM40) require Simplicity Commander Version 1.12.2 or above to support the flash `--tokengroup znet` command.

## Generate the Sign Key pair

Silicon Labs recommends using an HSM for the creation and storage of the Sign Key pair. The instructions for OpenSSL are shown here for simplicity.

1. Create a keypair for the SECP256r1 curve by running the following command:

    ```sh
    openssl ecparam -genkey -name prime256v1 -out sign_key.pem
    ```

2. Export the public key from this keypair using the following command:

```sh
openssl ec -in sign_key.pem -pubout -out sign_pubkey.pem
```

```sh
read EC key
writing EC key
```

## Provisioning the Public Sign Key in Simplicity Commander

To write the Public Sign Key to the device, run the command:

```sh
commander security writekey --sign sign_pubkey.pem --device EFR32MG21A010F1024 --serialno 440048205
```

where `sign_pubkey.pem` is the Public Sign Key in Privacy Enhanced Mail (PEM) format. This command can be executed only once per device.

```sh
Device has serial number 000000000000000014b457fffe045a8e

================================================================================
Please look through any warnings before proceeding.
THIS IS A ONE-TIME command, all code to be run on the device must be signed by this key.
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
DONE
```

To read the Public Sign Key on the device, run the command:

```sh
commander security readkey --sign --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
C4AF4AC69AAB9512DB50F7A26AE5B4801183D85417E729A56DA974F4E08A562C
DE6019DEA9411332DC1A743372D170B436238A34597C410EA177024DE20FC819
DONE
```

To generate the Public Sign Key token file for VSE devices, run the command:

```sh
commander util keytotoken sign_pubkey.pem --outfile sign_pubkey.txt
```

```sh
Writing EC tokens to sign_pubkey.txt...
DONE
```

To store a Public Sign Key copy on the top page of the main flash in the VSE device for ECDSA-P256-SHA256 Secure Boot, run the command:

```sh
commander flash --tokengroup znet --tokenfile sign_pubkey.txt --device EFR32MG22C224F512 --serialno 440048205
```

```sh
Writing 8192 bytes starting at address 0x0007e000
Comparing range 0x0007E000 - 0x0007FFFF (8 KB)
Erasing range 0x0007E000 - 0x0007FFFF (1 sector, 8 KB)
Programming range 0x0007E000 - 0x0007FFFF (8 KB)
DONE
```

> **Note**: The MCU Series 2 VSE devices (like EFM32PG22C200F512IM40) require Simplicity Commander Version 1.12.2 or above to support the `flash --tokengroup znet` command.

## Generate the Command Key Pair

Silicon Labs recommends using an HSM for the creation and storage of the Command Key pair. The instructions for OpenSSL are shown here for simplicity.

1. Create a keypair for the SECP256r1 curve by running the following command:

    ```sh
    openssl ecparam -genkey -name prime256v1 -out command_key.pem
    ```

2. Export the public key from this keypair using the following command:

    ```sh
    openssl ec -in command_key.pem -pubout -out command_pubkey.pem
    ```

    ```sh
    read EC key
    writing EC key
    ```

## Provisioning the Public Command Key in Simplicity Commander

To write the Public Command Key to the device, run the command:

```sh
commander security writekey --command command_pubkey.pem --device EFR32MG21A010F1024 --serialno 440048205
```

where `command_pubkey.pem` is the Public Command Key in PEM format. This command can be executed only once per device.

```sh
Device has serial number 000000000000000014b457fffe045a8e

================================================================================
Please look through any warnings before proceeding.
THIS IS A ONE-TIME command which permanently ties debug and tamper access to certificates signed by this key.
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
DONE
```

To read the Public Command Key on the device, run the command:

```sh
commander security readkey --command --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
B1BC6F6FA56640ED522B2EE0F5B3CF7E5D48F60BE8148F0DC08440F0A4E1DCA4
7C04119ED6A1BE31B7707E5F9D001A659A051003E95E1B936F05C37EA793AD63
DONE
```
