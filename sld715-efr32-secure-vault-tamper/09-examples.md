# Examples

## Overview

The examples for HSE-SVH and Series 3 Secure Vault Anti-Tamper module are described in the following table.

| **Example** | **Device (Radio Board)** | **HSE Firmware** | **Tool** |
|-------------|--------------------------|------------------|----------|
| Provision Tamper configuration | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | SE Manager |
| Provision Public Command Key & Tamper configuration | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | Simplicity Commander |
| " | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | Simplicity Studio 5 |
| Tamper disable and Roll challenge | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | SE Manager |
| " | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | Simplicity Commander |
| Roll challenge | EFR32MG21B010F1024IM32 (BRD4181C) | Version 1.2.9 | Simplicity Studio 5 |

> **Note**: Unless specified in the example, these examples can be applied to other HSE-SVH and Series 3 Secure Vault devices.

### Using Simplicity Commander

1. This application note uses Simplicity Commander v1.14.6. The procedures and console output may be different on other versions of Simplicity Commander. The latest version of Simplicity Commander can be downloaded from [https://www.silabs.com/developers/mcu-programming-options](https://www.silabs.com/developers/mcu-programming-options/).

    ```sh
    commander --version
    ```

    ```sh
    Simplicity Commander 1v14p6b1289

    JLink DLL version: 7.70d
    Qt 5.12.10 Copyright (C) 2017 The Qt Company Ltd.
    EMDLL Version: 0v18p9b677 mbed TLS version: 2.16.6


    DONE
    ```

2. The Simplicity Commander's Command Line Interface (CLI) is invoked by `commander.exe` in the Simplicity Commander folder. The location for Simplicity Studio 5 in Windows is `C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\commander`. For ease of use, it is highly recommended to add the path of `commander.exe` to the system PATH in Windows.

3. If more than one Wireless Starter Kit (WSTK or WPK) is connected via USB, the target WSTK must be specified using the `--serialno \<J-Link serial number\>` option.

4. If the WSTK or WPK is in debug mode OUT, the target device must be specified using the --device \<device name\> option. For more information about Simplicity Commander, see [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

5. The minimum recommended version of Simplicity Commander for SixG301 devices is v1.19.2.

### Using an External Tool

The tamper disable example uses **OpenSSL** to sign the access certificate and challenge response. The Windows version of OpenSSL can be downloaded from [https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html). This application note uses OpenSSL Version 3.5.0 (Win64).

```sh
openssl version
```

```sh
OpenSSL 3.5.0 8 Apr 2025
```

The OpenSSL's Command Line Interface (CLI) is invoked by openssl.exe in the OpenSSL folder. The location in Windows (Win64) is `C:\Program Files\OpenSSL-Win64\bin`. For ease of use, it is highly recommended to add the path of `openssl.exe` to the system PATH in Windows.

## Provision Public Command Key and Tamper Configuration

The Public part of the Command Key pair can be generated from the “unsafe” private key delivered with Simplicity Studio, by Simplicity Commander, or by a Hardware Security Module (HSM). Using an HSM is recommended for production systems.

### Public Command Key Extracted from “Unsafe” Private Key using OpenSSL

External tools such as openssl can be used to generate a public key from the reference private key provided in Simplicity Studio. Note that this private key is well known and must not be used in production devices.

Run the `openssl ec` command to generate the Public Command Key from the Private Command Key.

```sh
openssl ec -in /c/SiliconLabs/SimplicityStudio/v5/developer/adapter_packs/secmgr/scripts/offline/cmd-unsafe-privkey.pem -pubout -out cmd-unsafe-pubkey.pem
```

### Command Keypair Generated Randomly Using Simplicity Commander

Run the `util genkey` command to generate the Public Command Key pair (`command_key.pem` and `command_pubkey.pem`) and Public Command Key token file (`command_pubkey.txt`).

```sh
commander util genkey --type ecc-p256 --privkey command_key.pem --pubkey command_pubkey.pem
--tokenfile command_pubkey.txt
```

```sh
Generating ECC P256 key pair...
Writing private key file in PEM format to command_key.pem
Writing public key file in PEM format to command_pubkey.pem
Writing EC tokens to command_pubkey.txt...
DONE
```

```sh
commander util keytotoken command_pubkey.pem --outfile command_pubkey.txt
```

```sh
Writing EC tokens to command_pubkey.txt...
DONE
```

### Provision Public Command Key and Tamper Configuration

1. Run the `security writekey` command to provision the Public Command Key (e.g., `command_pubkey.pem`).

    ```sh
    commander security writekey --command command_pubkey.pem --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Device has serial number 000000000000000014b457fffe0f77ce

    ================================================================================
    Please look through any warnings before proceeding.
    THIS IS A ONE-TIME command which permanently ties debug and tamper access to certificates signed by this key.
    Type 'continue' and hit enter to proceed or Ctrl-C to abort:
    ================================================================================
    continue
    DONE
    ```

    > **Note**: The Public Command Key cannot be changed once written.

2. Run the `security readkey` command to read the Public Command Key from the HSE OTP for verification with the key in step 1.

    ```sh
    commander security readkey --command --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    B1BC6F6FA56640ED522B2EE0F5B3CF7E5D48F60BE8148F0DC08440F0A4E1DCA4
    7C04119ED6A1BE31B7707E5F9D001A659A051003E95E1B936F05C37EA793AD63
    DONE
    ```

3. Run the `security genconfig` command to generate a default `user_configuration.json` file for secure boot and tamper configuration.

    ```sh
    commander security genconfig --nostore -o user_configuration.json --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Configuration file written to user_configuration.json
    DONE
    ```

    > **Note**: Simplicity Commander Version 1.14.6 or above is required to support tamper configuration for all Series 2 HSE-SVH devices.

4. Use a text editor to modify the default tamper responses in `user_configuration.json` to the desired configuration as below.

    ```sh
    {
    "OPN": "EFR32MG21B010F1024", "VERSION": "1.0.0",
    "mcu_flags": { "SECURE_BOOT_ANTI_ROLLBACK": false, "SECURE_BOOT_ENABLE": false, "SECURE_BOOT_PAGE_LOCK_FULL": false, "SECURE_BOOT_PAGE_LOCK_NARROW": false, "SECURE_BOOT_VERIFY_CERTIFICATE": false
    },
    "tamper_filter": { "FILTER_PERIOD": 10,
    "FILTER_THRESHOLD": 6,
    "RESET_THRESHOLD": 5
    },
    "tamper_flags": { "DGLITCH_ALWAYS_ON": false
    },
    "tamper_levels": { "DCI_AUTH": 0,
    "DECOUPLE_BOD": 4,
    "DGLITCH": 2,
    "FILTER_COUNTER": 1,
    "MAILBOX_AUTH": 1,
    "OTP_READ": 4,
    "PRS0": 1,
    "PRS1": 1,
    "PRS2": 2,
    "PRS3": 2,
    "PRS4": 4,
    "PRS5": 4,
    "PRS6": 7,
    "PRS7": 7,
    "SECURE_LOCK": 4,
    "SELF_TEST": 4,
    "SE_CODE_AUTH": 4,
    "SE_DEBUG": 0,
    "SE_HARDFAULT": 4,
    "SE_ICACHE": 4,
    "SE_RAM_CRC": 4,
    "SOFTWARE_ASSERTION": 4,
    "TEMP_SENSOR": 2,
    "TRNG_MONITOR": 1,
    "USER_CODE_AUTH": 0,
    "VGLITCH_FALLING": 2,
    "VGLITCH_RISING": 2,
    "WATCHDOG": 4
    }
    }
    ```

    **Notes**:

    - This example does not enable the secure boot.

    - The `SECURE_BOOT_PAGE_LOCK_FULL` and `SECURE_BOOT_PAGE_LOCK_NARROW` flags are not supported in Series 3.

5. Run the `security writeconfig` command to program the secure boot and tamper configuration to the HSE OTP. This command can be executed once per device.

    ```sh
    commander security writeconfig --configfile user_configuration.json --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    THIS IS A ONE-TIME configuration: Please inspect file before confirming:
    user_configuration.json
    Type 'continue' and hit enter to proceed or Ctrl-C to abort:
    ================================================================================
    continue
    DONE
    ```

6. Run the `security readconfig` command to check the secure boot and tamper configuration of the device.

    ```sh
    commander security readconfig --serialno 440030580
    ```

    ```sh
    MCU Flags
    Secure Boot                    : Disabled
    Secure Boot Verify Certificate : Disabled
    Secure Boot Anti Rollback      : Disabled
    Secure Boot Page Lock Narrow   : Disabled
    Secure Boot Page Lock Full     : Disabled

    Tamper Levels

    FILTER_COUNTER     : 1
    WATCHDOG           : 4
    SE_RAM_CRC         : 4
    SE_HARDFAULT       : 4
    SOFTWARE_ASSERTION : 4
    SE_CODE_AUTH	   : 4
    USER_CODE_AUTH	   : 0
    MAILBOX_AUTH	   : 1
    DCI_AUTH	       : 0
    OTP_READ	       : 4
    SELF_TEST	       : 4
    TRNG_MONITOR	   : 1
    PRS0	           : 1
    PRS1	           : 1
    PRS2	           : 2
    PRS3	           : 2
    PRS4	           : 4
    PRS5	           : 4
    PRS6	           : 7
    PRS7	           : 7
    DECOUPLE_BOD	   : 4
    TEMP_SENSOR	       : 2
    VGLITCH_FALLING	   : 2
    VGLITCH_RISING	   : 2
    SECURE_LOCK	       : 4
    SE_DEBUG	       : 0
    DGLITCH	           : 2
    SE_ICACHE	       : 4

    Tamper Filter
    Filter Period   : 10
    Filter Threshold : 6
    Reset Threshold  : 5

    Tamper Flags
    Digital Glitch Detector Always On: Disabled
    DONE
    ```

    > **Note**: The `SECURE_BOOT_PAGE_LOCK_FULL` and `SECURE_BOOT_PAGE_LOCK_NARROW` flags are not supported in Series 3.

## Tamper Disable and Roll Challenge

See Platform Sample app for PRS connections.

The tamper disable was designed with three organizations in mind:

1. The Direct Customer to whom Silicon Labs sells the chip. This chip has the Public Command Key installed in the SE OTP.

2. The Product Company is a customer of the Direct Customer. This is applicable if the Direct Customer is creating a white-labeled product for another company or a sub-component that goes into another company’s product.

3. The Debug 3rd Party could be anyone, internal or external, that the Product Company decides is qualified to debug the device.

Because the Public Command Key is installed into the SE OTP of a large number of devices and cannot be changed, the corresponding Private Command Key must be guarded by a very stringent process. If this Private Command Key is ever leaked, all the devices programmed with the corresponding Public Command Key will be compromised.

A tamper disable use case is described in the following figure, and the signing process is performed by a Hardware Security Module (HSM).

![Tamper Disable Use Case](./resources/sld715-tamper-disable-use-case.png)

The tamper disable flow moving across the time axis from left to right is explained below:

1. The Product Company creates a Private/Public Certificate Key pair for each device. Because the key pair is assigned only to a single device, the company may not need to protect the Private Certificate Key as securely as the Private Command Key by the Direct Customer.

    In this example, the Private/Public Certificate Key pair (`cert_key.pem` and `cert_pubkey.pem`) is generated by running the `util genkey` command.

    ```sh
    commander util genkey --type ecc-p256 --privkey cert_key.pem --pubkey cert_pubkey.pem
    ```

    ```sh
    Generating ECC P256 key pair...
    Writing private key file in PEM format to cert_key.pem
    Writing public key file in PEM format to cert_pubkey.pem
    DONE
    ```

2. The Public Certificate Key (`cert_pubkey.pem`) for each device is passed to the Silicon Labs Direct Customer. The part number and serial number are also required if Direct Customer cannot access the device.

    Run the security status command to get the device serial number. The `--serialno` option is for the J-Link serial number of the WSTK.

    ```sh
    commander security status --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    SE Firmware version	   : 1.2.9
    Serial number	       : 000000000000000014b457fffe0f77ce
    Debug lock	           : Disabled
    Device erase	       : Enabled
    Secure debug unlock	   : Disabled
    Tamper status	       : OK
    Secure boot	           : Disabled
    Boot status	           : 0x20 - OK
    Command key installed  : True
    Sign key installed	   : False
    DONE
    ```

3. The Direct Customer then places that Public Certificate Key in the access certificate. The access certificate is per device because it contains the unique device serial number. This certificate is generated once upon creation of the device, and thereafter, is generally only modified when the Private/Public Certificate Key pair is changed by the Product Company.

    The following two steps are **OPTIONAL** for customization of Authorizations and Tamper Authorizations.

    - (Optional) Run the `security genauth` command to generate the default certificate authorization file (`certificate_authorization.json`).

        ```sh
        commander security genauth -o certificate_authorizations.json --nostore --serialno 440030580
        ```

        ```sh
        DONE
        ```

    - (Optional) Use a text editor to modify the default Authorizations and Tamper Authorizations in the `json` file.

    Run the `security gencert` command with the following parameters from the Product Company to generate an unsigned access certificate (`access_certificate.extsign`) in Security Store:

    - Device part number

    - Device serial number

    - Public Certificate Key

    ```sh
    commander security gencert --device EFR32MG21B010F1024 --deviceserialno 000000000000000014b457fffe0f77ce
    --cert-pubkey cert_pubkey.pem --extsign
    ```

    ```sh
    Authorization file written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/ certificate_authorizations.json
    Cert key written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/ cert_pubkey.pem
    Created an unsigned certificate in Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/ access_certificate.extsign
    DONE
    ```

    **Notes**:

    - The `--extsign` option to create an unsigned access certificate is only available in Simplicity Commander Version 1.11.2 or above.

    - The unsigned access certificate is generated with the default certificate authorization file (`certificate_authorization.json`) which uses `0x0000003e` for Authorizations and `0xffffffb6` (HSE-SVH device) for Tamper Authorizations.

    - (Optional) Use `--authorization` option if the customized json file generated in the above optional steps is used.

    ```sh
    commander security gencert --device EFR32MG21B010F1024 --authorization certificate_authorizations.json
    --deviceserialno 000000000000000014b457fffe0f77ce --cert-pubkey cert_pubkey.pem –extsign –outfile access_certificate.extsign
    ```

4. The signing of the access certificate can be done by passing an unsigned access certificate to a Hardware Security Module (HSM) containing the Private Command Key.

    In this example, the OpenSSL is used to sign the access certificate (`access_certificate.extsign`) in Security Store with the Private Command Key (`command_key.pem`). The access certificate signature is in the `cert_signature.bin` file.

    ```sh
    openssl dgst -sha256 -binary -sign command_key.pem -out cert_signature.bin access_certificate.extsign
    ```

    Run the `util signcert` command with the following parameters to verify the signature and generate the signed access certificate (`access_certificate.bin`):

    - Unsigned access certificate

    - Access certificate signature

    - Public Command Key

    ```sh
    commander util signcert access_certificate.extsign --cert-type access --signature cert_signature.bin
    --verify command_pubkey.pem --outfile access_certificate.bin
    ```

    ```sh
    R = 76CDC5BA18E5248FDA5418002F250F149B449829A005D6F0726268016CC53ED4
    S = E4B8ABA2CF742B0E6CC5BA2C1023D76BEEF3C4A11DA97CC4D23459F32237A206
    Successfully verified signature
    Successfully signed certificate
    DONE
    ```

    **Notes**:

    - Put the required files in the same folder to run the command.

    - The `util signcert` command for access certificate is only available in Simplicity Commander Version 1.11.2 or above.

    - The access certificate signature can be in a Raw or Distinguished Encoding Rules (DER) format.

5. The access certificate is passed to the Product Company. The purpose of the access certificate is to grant overall debug access capabilities to the Product Company and authorize them to allow third parties to debug the device. The Product Company can now use the access certificate to generate the Disable Tamper Token. The same access certificate can be used to generate as many Disable Tamper Tokens as necessary without having to ever go back to the Direct Customer.

6. To create the Disable Tamper Token, a debug session must be started with the device and the challenge value (which is a random number `Challenge 1` in this example) should be read out to generate the challenge response.

    Run the `security gencommand` command to generate the challenge response without disable tamper command signature and store it in a file (`command_unsign.bin`).

    ```sh
    commander security gencommand --action disable-tamper --disable-param 0x00fa0000 -o command_unsign.bin
    --nostore --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Unsigned command file written to:
    command_unsign.bin
    DONE
    ```

    **Notes**:

    - The tamper disable mask (`0x00fa0000`) is based on the Tamper platform example on EFR32xG21B devices (See readme.md in the Platform sample).

    - If the `--disable-param` option is not provided, it will restore all tamper sources (`0xffffffb6`) by default.

7. The challenge response is then cryptographically hashed (SHA-256) to create a digest. The digest is then signed by the Private Certificate Key to generate the disable tamper command signature.

    The signing of the challenge response can be done by passing an unsigned challenge response to a Hardware Security Module (HSM) containing the Private Certificate Key.

    In this example, the OpenSSL is used to sign the challenge response (`command_unsign.bin`) with the Private Certificate Key (`cert_key.pem`). The disable tamper command signature is in the `command_signature.bin` file.

    ```sh
    openssl dgst -sha256 -binary -sign cert_key.pem -out command_signature.bin command_unsign.bin
    ```

8. Run the `security disabletamper` command with the access certificate (`access_certificate.bin`) from Direct Customer and disable tamper command signature (`command_signature.bin`) in step 7 to generate the Disable Tamper Token.

    ```sh
    commander security disabletamper --disable-param 0x00fa0000 --cert access_certificate.bin
    --command-signature command_signature.bin EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Certificate written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/ device_000000000000000014b457fffe0f77ce/access_certificate.bin
    R = A70834D97640A92510D151765F0EED6C6A05CB8BE81E06E905C230ED24E71659
    S = 9B69C113C2B7DEE60BF0BC7D72719F7F9465840D68EADBBB4F9BCE7A1267B936
    Command signature is valid
    Tamper successfully disabled.
    Command disable tamper payload was stored in Security Store
    DONE
    ```

    **Notes**:

    - Put the required files in the same folder to run the command.

    - The disable tamper command signature can be in a Raw or Distinguished Encoding Rules (DER) format.

    - Simplicity Commander Version 1.11.2 or above is required to support signature in DER format.

9. (Alternative) Key protection is not required if the Private Certificate Key is ephemeral. Steps 6 to 8 can be implemented by running the `security disabletamper` command with the access certificate (`access_certificate.bin`) from the Direct Customer and Private Certificate Key (`cert_key.pem`) to generate the Disable Tamper Token.

    ```sh
    commander security disabletamper --disable-param 0x00fa0000 --cert access_certificate.bin --cert-privkey
    cert_key.pem --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Certificate written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/access_certificate.bin
    Cert key written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/cert_pubkey.pem
    Created unsigned disable tamper command
    Signed disable tamper command using
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/cert_key.pem
    Tamper successfully disabled.
    Command disable tamper payload was stored in Security Store
    DONE
    ```

10. The Disable Tamper Token (aka `Command disable tamper payload`) file (`tamper_payload_111110100000000000000000.bin`, where `111110100000000000000000` is `0x00fa0000` for tamper disable mask) is stored in the Security Store. The location in Windows is `C:\Users\<PC user name>\AppData\Local\SiliconLabs\commander\SecurityStore\device_<Serial number>\challenge_<Challenge value>`.

    ![screenshot](./resources/sld715-image71.jpeg)

    Users can also use the `security getpath` command to get the path of the Security Store or a specified device.

    ```sh
    commander security getpath
    ```

    ```sh
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore
    DONE
    ```

    ```sh
    commander security getpath --deviceserialno 0000000000000000588e81fffe70350d
    ```

    ```sh
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000588e81fffe70350d
    DONE
    ```

11. The Disable Tamper Token and the device are now delivered to the Debug 3rd Party.

    Run the `security gencommand` command to create the Security Store to place the Disable Tamper Token file.

    ```sh
    commander security gencommand --action disable-tamper --disable-param 0x00fa0000
    --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Unsigned command file written to Security Store:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/challenge_8e7f73e6322edda06b62997155334f29/ disable_tamper_command_to_be_signed09_08_2021.bin
    DONE
    ```

    Copy the Disable Tamper Token file (`tamper_payload_111110100000000000000000.bin`) from Product Company to the Windows Security Store `challenge_<Challenge value>` folder located in `C:\Users\<PC user name>\AppData\Local\SiliconLabs\commander\SecurityStore\device_<Serial number>\challenge_<Challenge value>`.

12. The device compares the Disable Tamper Token contents with its internal serial number, challenge value, and Public Command Key to determine the token’s authenticity. If authentic, it will execute the disable tamper command to restore the default levels on the tamper disable mask (`0xfa000000`); otherwise, it will ignore the command.

    Run the `security disabletamper` command to disable the tamper.

    ```sh
    commander security disabletamper --disable-param 0x00fa0000 --device EFR32MG21B010F1024
    --serialno 440030580
    ```

    ```sh
    Disabling tamper with tamper payload:
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_000000000000000014b457fffe0f77ce/challenge_8e7f73e6322edda06b62997155334f29/ tamper_payload_111110100000000000000000.bin
    Tamper successfully disabled.
    DONE
    ```

    > **Note**: Users can verify the Disable Tamper Token by following steps 4 to 6 in [Tamper Disable](./08-tamper-disable.md) if the EFR32xG21B device is running in the Normal mode of the SE Manager Tamper platform example.

13. The Debug 3rd Party can now use this same Disable Tamper Token to disable the tamper (step 12), over and over again after each power-on or pin reset, until they have finished debugging the device.

14. Once the Debug 3rd Party has finished debugging, they will send the device back to the Product Company.

15. Once the Product Company receives the device, they will immediately start a debug session to roll the challenge (from `Challenge 1` to `Challenge 2` in this example). Rolling the challenge will effectively invalidate any Disable Tamper Token that has been previously given to any third party.

    Run the `security rollchallenge` command and reset the device to invalidate the current Disable Tamper Token. The challenge cannot be rolled before it has been used at least once; that is, by running the `security disabletamper` or `security unlock` command.

    ```sh
    commander security rollchallenge --device EFR32MG21B010F1024 --serialno 440030580
    ```

    ```sh
    Challenge was rolled successfully.
    DONE
    ```

    The unlock token is invalidated after rolling the challenge because any previously issued Disable Tamper Token now contains a different challenge value (`Challenge 1`) than the challenge value currently in the device (`Challenge 2`).

    The validation process of any previously issued Disable Tamper Token will always fail until a new Disable Tamper Token is issued with a current matching challenge value (`Challenge 2`).

    > **Note**: Direct Customer can directly use the Private Command Key on the connected chip to generate the Disable Tamper Token in Security Store. But it has a high risk (cannot use HSM) to leak the Private Command Key to a 3rd party when using this approach.

    ```sh
    commander security disabletamper --disable-param 0x00fa0000 --command-key command_key.pem
    --device EFR32MG21B010F1024 --serialno 440030580
    ```
