# Examples

## Using Commander tool

### Standard Debug Lock/Unlock using Simplicity Commander

This application note uses Simplicity Commander v1.19.2. The procedures and console output may be different for the other versions of Simplicity Commander. The latest version of Simplicity Commander can be downloaded from silabs.com.

```sh
commander --version
```

```sh
Simplicity Commander 1v19p2b1907

JLink DLL version: 8.44
Qt 5.15.2 Copyright (C) 2017 The Qt Company Ltd.
EMDLL Version: 0v19p19b793 mbed TLS version: 2.16.6

Emulator found with SN=440328778 USBAddr=0

DONE
```

For more information about Simplicity Commander, see [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

1. Run the security status command to get the selected device configuration.

    ```sh
    commander security status --device sixg301 --serialno 440328778
    ```

    ```sh
    SE ROM version          : 5.3
    SE Firmware version     : 3.3.2
    Serial number           : 0000000000000000781c9dfffe58959e
    Debug lock              : Disabled
    Device erase            : Enabled
    Secure debug unlock     : Disabled
    Tamper status           : OK
    Secure boot             : Disabled
    Boot status             : 0x20 - OK
    Command key installed   : False
    Sign key installed      : False
    Security state          : Production
    DONE
    ```

2. Run the security lock command to lock the selected device.

    ```sh
    commander security lock --device sixg301 --serialno 440328778
    ```

    ```sh
    WARNING: Secure debug unlock is disabled. Only way to regain debug access is to run a device erase.
    Device is now locked.
    DONE
    ```

3. Run the security lock command to lock the selected device.

    ```sh
    commander security status --device sixg301 --serialno 440328778
    ```

    ```sh
    SE ROM version        : 5.3
    SE Firmware version   : 3.3.2
    Serial number         : 0000000000000000781c9dfffe58959e
    Debug lock            : Enabled
    Device erase          : Enabled
    Secure debug unlock   : Disabled
    Tamper status         : OK
    Secure boot           : Disabled
    Boot status           : 0x20 - OK
    Command key installed : False
    Sign key installed    : False
    Security state        : Production
    DONE
    ```

4. Run the `security erasedevice` command to unlock the selected device.

    ```sh
    commander security erasedevice --device sixg301 --serialno 440328778
    ```

    ```sh
    Successfully erased device
    DONE
    ```

    > **Note**: Issue a power-on or pin reset to complete the unlock process.

5. Run the `security status` command again to check the device configuration.

```sh
commander security status --device EFR32MG21B010F1024IM32 --serialno 440328778
```

```sh
SE ROM version          : 5.3
SE Firmware version     : 3.3.2
Serial number           : 0000000000000000781c9dfffe58959e
Debug lock              : Disabled
Device erase            : Enabled
Secure debug unlock     : Disabled
Tamper status           : OK
Secure boot             : Disabled
Boot status             : 0x20 - OK
Command key installed   : False
Sign key installed      : False
Security state          : Production
DONE
```

### Secure Debug Lock Using Commander Tool

1. Run the `security status` command to get the selected device configuration.

    ```sh
    commander security status --device sixg301 --serialno 440326972
    ```

    ```sh
    commander security status --device sixg301 --serialno 440326972
    ---------------------------------------------------------------
    SE ROM version          : 5.3
    SE Firmware version     : 3.3.2
    Serial number           : 0000000000000000781c9dfffe589591
    Debug lock              : Disabled
    Device erase            : Enabled
    Secure debug unlock     : Disabled
    Tamper status           : OK
    Secure boot             : Disabled
    Boot status             : 0x20 - OK
    Command key installed   : False
    Sign key installed      : False
    Security state          : Production
    DONE
    ```

2. Run `util genkey` to generate commander private/public key pair.

    ```sh
    commander util genkey --type ecc-p256 --privkey command_key.pem --pubkey command_pubkey.pem
    ```

    ```sh
    Generating ECC P256 key pair...
    Writing private key file in PEM format to command_key.pem
    Writing public key file in PEM format to command_pubkey.pem
    DONE
    ```

3. Run the `security writekey` command to provision the Public Command Key (e.g., `command_pubkey.pem`).

    ```sh
    commander security writekey --command command_pubkey.pem --device sixg301 --serialno 440326972
    ```

    ```sh
    Device has serial number 0000000000000000781c9dfffe589591

    ================================================================================
    Please look through any warnings before proceeding.
    THIS IS A ONE-TIME command which permanently ties debug and tamper access to certificates signed by this key.
    Type 'continue' and hit enter to proceed or Ctrl-C to abort:
    ================================================================================
    continue
    Command public key stored in: C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/ device_0000000000000000781c9dfffe589591/command_pubkey.pem
    DONE
    ```

    > **Note**: The Public Command Key cannot be changed once written.

4. This step is optional. To verify the public command key written into the device's SE OTP, run the `security readkey` command.

    ```sh
    commander security readkey --command --device sixg301 --serialno 440326972
    ```

    ```sh
    50DF50A09242A49F53251xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxE6E81C6
    B3B891B1B9DBFC3D5F2D0yyyyy435DA6E8AFAF60037DA21AD7B2E1
    DONE
    ```

5. Run the `security lockconfig` command to enable the secure debug.

    ```sh
    commander security lockconfig --secure-debug-unlock enable --device sixg301 --serialno 440326972
    ```

    ```sh
    Secure debug unlock was enabled
    DONE
    ```

6. a. For the **TrustZone-unaware** application, run the `security lock` command to lock the selected device.

    ```sh
    commander security lock --device sixg301 --serialno 440326972
    ```

    ```sh
    Device is now locked.
    DONE
    ```

    b. For the **TrustZone-aware** application, run the `security lock --trustzone ####` command to set the [debug options](05-debug-unlock.md#trustzone-debug-authentication) (e.g., `1100`) and lock the selected device. The bit order of #### is SPNIDLOCK (MSB), SPIDLOCK, NIDLOCK, and DBGLOCK (LSB).

    ```sh
    commander security lock --trustzone 1100 --device EFR32MG21A010F1024 --serialno 440048205
    ```

    ```sh
    Writing debug restriction bits:
    DBGLOCK:	0
    NIDLOCK:	0
    SPIDLOCK:   1
    SPNIDLOCK:  1
    Device is now locked.
    DONE
    ```

    **Notes**:

    - The `--trustzone` option for the `security lock` command requires Simplicity Commander **≥ v1.13.3**.

    - It is strongly recommended to upgrade to SE firmware **≥ v1.2.14** (xG21 and xG22) or **≥ v2.2.1** (other Series 2 devices). For Series 3 devices, it is strongly recommended to upgrade SE firmware to ≥ v3.3.2 so that the debug options cannot be modified after the device is locked.

    - Use `commander security lock` without the `--trustzone ####` option if the default setting of debug options (`0000`) is good enough for a TrustZone-aware application.

7. Run the `security disabledeviceerase` command to disable device erase. This is an **IRREVERSIBLE** action, and should be the last step in production. (**It is recommended not to set this option while evaluating secure debug functionality.**)

    ```sh
    commander security disabledeviceerase --device sixg301 --serialno 440326972
    ```

    ```sh
    ================================================================================
    THIS IS A ONE-TIME command which Permanently disables device erase.
    If secure debug lock has not been set, there is no way to regain debug access to this device. Type 'continue' and hit enter to proceed or Ctrl-C to abort:
    ================================================================================
    continue
    Disabled device erase successfully
    DONE
    ```

    > **Note**: The debug options cannot be reset to the default value 0000 (unlock) if the device erase option is disabled.

8. Read back device status after it is securely locked.

    a. For the TrustZone-unaware application, run the security status command to check the debug lock status of the device

    ```sh
    commander security status --device sixg301 --serialno 440326972
    ```

    ```sh
    SE ROM version        : 5.3
    SE Firmware version   : 3.3.2
    Serial number         : 0000000000000000781c9dfffe589591
    Debug lock            : Enabled
    Device erase          : Enabled
    Secure debug unlock   : Enabled
    Tamper status         : OK
    Secure boot           : Disabled
    Boot status           : 0x20 - OK
    Command key installed : True
    Sign key installed    : False
    Security state        : Production
    DONE
    ```

    b. For the TrustZone-aware application, run the `security status --trustzone` command to check the full debug lock status of the device.

    ```sh
    commander security status --trustzone --device sixg301 --serialno 440326972
    ```

    ```sh
    SE ROM version          : 5.3
    SE Firmware version     : 3.3.2
    Serial number           : 0000000000000000781c9dfffe589591
    Debug lock              : Enabled
    Device erase            : Enabled
    Secure debug unlock     : Enabled

    Debug lock state: Locked

    TrustZone Config:
    Non-secure, invasive debug lock     (DBGLOCK) : Unlocked
    Non-secure, non-invasive debug lock (NIDLOCK) : Unlocked
    Secure, invasive debug lock         (SPIDLOCK) : Unlocked
    Secure, non-invasive debug lock     (SPNIDLOCK) : Unlocked

    TrustZone State:
    Non-secure, invasive debug lock state       (DBGLOCK) : Unlocked
    Non-secure, non-invasive debug lock state   (NIDLOCK) : Unlocked
    Secure, invasive debug lock state           (SPIDLOCK) : Unlocked
    Secure, non-invasive debug lock state       (SPNIDLOCK) : Unlocked

    Tamper status          : OK
    Secure boot            : Disabled
    Boot status            : 0x20 - OK
    Command key installed  : True
    Sign key installed     : False
    Security state         : Production
    DONE
    ```

### Secure Debug Unlock Using Commander tool

Use case for secure debug unlock is explained in [secure debug unlock](./05-debug-unlock.md#secure-debug-lockunlock-flow) as pictorially. The steps to securely unlock the device securely are explained below.

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

2. The Public Certificate Key (`cert_pubkey.pem`) for each device is passed to the Silicon Labs Direct Customer. The part number and serial number are also required if Direct Customer cannot access the device.If necessary, run the `security status` command to get the device serial number.

    ```sh
    commander security status --device sixg301 --serialno 440326972
    ```

    ```sh
    SE ROM version	        : 5.3
    SE Firmware version	    : 3.3.2
    Serial number	        : 0000000000000000781c9dfffe589591
    Debug lock	            : Enabled
    Device erase	        : Enabled
    Secure debug unlock	    : Enabled
    Tamper status	        : OK
    Secure boot	            : Disabled
    Boot status	            : 0x20 - OK
    Command key installed	: True
    Sign key installed	    : False
    Security state	        : Production
    DONE
    ```

3. The Direct Customer then places that Public Certificate Key in the [access certificate](05-debug-unlock.md#access-certificate). The access certificate is unique per device because it contains the unique device serial number. This certificate is generated once upon creation of the device, and thereafter, is generally only modified when the Private/Public Certificate Key pair is changed by the Product Company.

    Run the `security gencert` command with the following parameters from the Product Company to generate an unsigned access certificate (`access_certificate.extsign`) in Security Store:

    - Device part number

    - Device serial number

    - Public Certificate Key

    ```sh
    commander security gencert --device sixg301 --deviceserialno 0000000000000000781c9dfffe589591 --cert-pubkey cert_pubkey.pem --extsign
    ```

    ```sh
    Authorization file written to Security Store:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/ device_0000000000000000781c9dfffe589591/certificate_authorizations.json Cert key written to Security Store:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/ device_0000000000000000781c9dfffe589591/cert_pubkey.pem
    Created an unsigned certificate in Security Store:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/ device_0000000000000000781c9dfffe589591/access_certificate.extsign
    DONE
    ```

4. The signing of the access certificate can be done by passing an unsigned access certificate to a Hardware Security Module (HSM) containing the Private Command Key.

    In this example, the OpenSSL tool is used instead of HSM. Please install openSSL version above 3.5.0. Sign the access certificate (`access_certificate.extsign`) in Security Store with the Private Command Key (`command_key.pem`). The [access certificate signature](05-debug-unlock.md#access-certificate) is in the `cert_signature.bin` file.

    Run the `util signcert` command with the following parameters to verify the signature and generate the signed access certificate (`access_certificate.bin`):

    - Unsigned access certificate

    - Access certificate signature

    - Public Command Key

    ```sh
    commander util signcert access_certificate.extsign --cert-type access --signature cert_signature.bin
    --verify command_pubkey.pem --outfile access_certificate.bin
    ```

    ```sh
    R = D97E43FEA278207080D6D0808B46810C1167F123AF1CA9FAF2DE0F4322B97ACE
    S = FEDFEA11A3C83AFFCD5293283B13A50580862B9F651AAE08012C2BFB6BA8E697
    Successfully verified signature
    Successfully signed certificate
    DONE
    ```

5. The access certificate is passed to the Product Company. The purpose of the access certificate is to grant overall debug access capabilities to the Product Company and authorize them to allow third parties to debug the device. The Product Company can now use the access certificate to generate the Debug Unlock Token. The same access certificate can be used to generate as many Debug Unlock Tokens as necessary without having to ever go back to the Direct Customer.

6. To create the Debug Unlock Token, a debug session must be started with the device and the challenge value (which is a random number `Challenge 1` in this example) should be read out to generate the challenge response.

    Run the `security gencommand` command to generate the challenge response without debug access command signature and store it in a file (`command_unsign.bin`).

    ```sh
    commander security gencommand --action debug-unlock --unlock-param 1111 -o command_unsign.bin --nostore
    --device sixg301 --serialno 440326972
    ```

    ```sh
    Unsigned command file written to:
    command_unsign.bin
    DONE
    ```

    **Notes**:

    - The data in the `--unlock-param` option are the bits 2 to 5 of debug mode request in the challenge response.

    - The default value `1111` (reset all debug options) is in place if the `security gencommand` command does not include the `-- un-lock-param` option.

7. The challenge response is then cryptographically hashed (SHA-256) to create a digest. The digest is then signed by the Private Certificate Key to generate the debug access command signature.

    The signing of the challenge response can be done by passing an unsigned challenge response to a Hardware Security Module (HSM) containing the Private Certificate Key.

    In this example, OpenSSL is used to sign the challenge response (`command_unsign.bin`) with the Private Certificate Key (`cert_key.pem`). The debug access command signature is in the `command_signature.bin` file.

    ```sh
    openssl dgst -sha256 -binary -sign cert_key.pem -out command_signature.bin command_unsign.bin
    ```

8. Run the `security unlock` command with the access certificate (`access_certificate.bin`) from Direct Customer and debug access command signature (`command_signature.bin`) in step 7 to generate the Debug Unlock Token.

    ```sh
    commander security unlock --cert access_certificate.bin --command-signature command_signature.bin -- unlock-param 1111 --device sixg301 --serialno 440326972
    ```

    ```sh
    Certificate written to Security Store:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000781c9dfffe589591/access_certificate.bin
    R = B4F5F2628B50BBA54ADAB4EB67CD0F933FE4C01E8BA760915D5167E75330F3A2
    S = 4D7EEA1B6D8EA61F3140198B26060D9F96D32CDCC5CE58CB4611ECED0D21F9B8
    Command signature is valid
    Secure debug successfully unlocked
    Command unlock payload was stored in Security Store
    DONE
    ```

    **Notes**:

    - Put the required files in the same folder to run the command.

    - The debug access command signature can be in a Raw or Distinguished Encoding Rules (DER) format.

    - It requires Simplicity Commander Version 1.11.2 or above to support signature in DER format.

    - The data in the `--unlock-param` option are the bits 2 to 5 of debug mode request in the Debug Unlock Token. This value **MUST** be equal to the value of `--unlock-param` option in step 6.

    - The default value `1111` (reset all debug options) is in place if the `security unlock` command does not include the `--unlock- param` option.

9. **(Alternative)** The key protection is not required if the Private Certificate Key is ephemeral. Steps 6 to 8 can be implemented by running the `security unlock` command with the access certificate (`access_certificate.bin`) from the Direct Customer and Private Certificate Key (`cert_key.pem`) to generate the Debug Unlock Token.

    ```sh
    commander security unlock --cert access_certificate.bin --cert-privkey cert_key.pem --unlock-param 1111 -- device sixg301 --serialno 440326972
    ```

    ```sh
    Unlocking with unlock payload:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000781c9dfffe589591/challenge_a7be6d84d1a41321b1492e643ba626f1/unlock_payload_0000000000111110.bin
    Secure debug successfully unlocked
    DONE
    ```

    **Notes**:

    - The data in the `--unlock-param` option are the bits 2 to 5 of debug mode request in the Debug Unlock Token.

    - The default value `1111` (reset all debug options) is in place if the `security unlock` command does not include the `--unlock-param` option.

10. The Debug Unlock Token (also known as `Command unlock payload`) file (`unlock_payload_0000000000111110.bin`, where `0000000000111110` is the value of debug mode request) is stored in the Security Store. The location in Windows is `C:\Users\<userName>\AppData\Local\SiliconLabs\commander\SecurityStore\device_<Serial number>\challenge_<Challenge value>`.

    ![screenshot](resources/sld714-image68.jpeg)

    Users can also use the `security getpath` command to get the path of the Security Store or a specified device.

    ```sh
    commander security getpath --device sixg301
    ```

    ```sh
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore
    DONE
    ```

    ```sh
    commander security getpath --deviceserialno 0000000000000000781c9dfffe589591
    ```

    ```sh
    C:/Users/<username>/AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000781c9dfffe589591
    DONE
    ```

11. The Debug Unlock Token and the device are now delivered to the Debug 3rd Party.

    Run the `security gencommand` command to create the Security Store to place the Debug Unlock Token file.

    ```sh
    commander security gencommand --action debug-unlock --device sixg301 --serialno 440326972
    ```

    ```sh
    Unsigned command file written to Security Store:
    C:/Users/<userName>/AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000781c9dfffe589591/challenge_a7be6d84d1a41321b1492e643ba626f1/ unlock_command_to_be_signed22_07_2025.bin
    DONE
    ```

    Copy the Debug Unlock Token file (`unlock_payload_0000000000111110.bin`) from Product Company to the Windows Security Store `challenge_<Challenge value>` folder located in `C:\Users\<PC user name>\AppData\Local\SiliconLabs\commander\SecurityStore\device_<Serial number>challenge_<Challenge value>`.

12. The device compares the Debug Unlock Token contents with its internal serial number, challenge value, and Public Command Key to determine the token’s authenticity. If authentic, it will execute the debug access command to unlock the device; otherwise, it will ignore the command.

    Run the `security unlock` command to unlock the device.

    ```sh
    commander security unlock --unlock-param 1111 --device sixg301 --serialno 440326972
    ```

    ```sh
    Unlocking with unlock payload:
    C:/Users/<userName>AppData/Local/SiliconLabs/commander/SecurityStore/device_0000000000000000781c9dfffe589591/challenge_a7be6d84d1a41321b1492e643ba626f1/unlock_payload_0000000000111110.bin
    Secure debug successfully unlocked
    DONE
    ```

    **Notes**:

    - If the security store has multiple tokens for the selected device, use `--unlock-param` option to specify which unlock token is chosen to unlock the device.

    - Simplicity Commander will only use the token with value `1111` (error if not available) from the security store to unlock the device if the security unlock command does not include the `--unlock-param` option.

13. Run `security status --trustzone` command to check the full debug lock status of the device.

    ```sh
    commander security status --trustzone --device sixg301 --serialno 440326972
    ```

    ```sh
    SE ROM version	        : 5.3
    SE Firmware version	    : 3.3.2
    Serial number	        : 0000000000000000781c9dfffe589591
    Debug lock	            : Disabled
    Device erase	        : Enabled
    Secure debug unlock     : Disabled

    Debug lock state: Unlocked

    TrustZone Config:
    Non-secure, invasive debug lock   (DBGLOCK) : Unlocked
    Non-secure, non-invasive debug lock (NIDLOCK) : Unlocked
    Secure, invasive debug lock  (SPIDLOCK) : Unlocked
    Secure, non-invasive debug lock   (SPNIDLOCK): Unlocked

    TrustZone State:
    Non-secure, invasive debug lock state  (DBGLOCK) : Unlocked
    Non-secure, non-invasive debug lock state (NIDLOCK) : Unlocked
    Secure, invasive debug lock state   (SPIDLOCK) : Unlocked
    Secure, non-invasive debug lock state   (SPNIDLOCK): Unlocked

    Tamper status	      : OK
    Secure boot	          : Disabled
    Boot status	          : 0x20 - OK
    Command key installed : True
    Sign key installed	  : False
    DONE
    ```

14. The Debug 3rd Party can now use this same Debug Unlock Token to unlock the device (step 12), over and over again after each power-on or pin reset, until they have finished debugging the device.

15. Once the Debug 3rd Party has finished debugging, they will send the device back to the Product Company.

16. Once the Product Company receives the device, they will immediately start a debug session, roll the challenge (from `Challenge 1` to `Challenge 2` in this example), and put the device back into the secure debug lock state. Rolling the challenge will effectively invalidate any Debug Unlock Token that has been previously given to any third party.

    Run the `security rollchallenge` command and reset the device to invalidate the current Debug Unlock Token. The challenge cannot be rolled before it has been used at least once.

    ```sh
    commander security rollchallenge --device sixg301 --serialno 440326972
    ```

    ```sh
    Challenge was rolled successfully.
    DONE
    ```

    The unlock token is invalidated after rolling the challenge because any previously issued Debug Unlock Token now contains a different challenge value (`Challenge 1`) than the challenge value currently in the device (`Challenge 2`).

    The validation process of any previously issued Debug Unlock Token will always fail until a new Debug Unlock Token is issued with a current matching challenge value (`Challenge 2`).

## Using Simplicity Studio

The security operations are performed in the Security Settings of Simplicity Studio. This application note uses Simplicity Studio v5.11.0.0. The procedures and pictures may be different for the other versions of Simplicity Studio 5.

### Standard Debug lock/unlock using Simplicity Studio

1. Right-click the selected debug adapter RB (ID:J-Link serial number) to display the context menu.

   ![Debug Adapters Context Menu](./resources/sld714-image69.png)

2. Click **Device configuration...** to open the **Configuration of device: J-Link Silicon Labs (serial number)** dialog box. Click the **Security Settings** tab to get the selected device configuration.

   ![Configuration on Selected Device](./resources/sld714-image70.png)

3. Click [**Enable**] next to **Enable Debug Lock:** to lock the device. The following **Enable Debug Lock Warning** is displayed. Click [**Yes**] to confirm. This configures standard debug lock.

   ![Enable Debug Lock](./resources/sld714-image71.png)

4. The [**Enable**] controls next to **Enable Secure Debug Unlock:** and **Enable Debug Lock:** are grayed out after standard debug lock is enabled.

   ![Standard Debug Lock](resources/sld714-image72.jpeg)

5. Click [**Device Erase**] to unlock the device.

   ![Device Erase](resources/sld714-image73.png)

6. The device will return to the unlock state. Click [**OK**] to exit.

   ![Standard Debug Unlock](./resources/sld714-image74.png)

### Secure Debug Lock Using Simplicity Studio

1. Run the `util keytotoken` command to convert the Public Command Key file (PEM format) into a text file (`command_pubkey.txt`).

    Refer to [Secure Debug Lock Using Commander tool)](#secure-debug-unlock-using-commander-tool) before performing these steps.

    ```sh
    commander util keytotoken command_pubkey.pem --outfile command_pubkey.txt
    ```

    ```sh
    Writing EC tokens to command_pubkey.txt...
    DONE
    ```

2. Open **Security Settings** of the selected device

3. Click the **WriteKey** link next to **Command Key:** to open a dialog box.

   ![screenshot](./resources/sld714-write-key-link.png)

4. The **Write Command Key** dialog box is displayed.

   ![screenshot](resources/sld714-image77.png)

5. Open the `command_pubkey.txt` file generated in step 1.

    ```sh
    MFG_SIGNED_BOOTLOADER_KEY_X : 50DF50A09242A49F53251D38E1A368C82EC7CA2D33E6E81C6B3B891B1B9DBFC3
    MFG_SIGNED_BOOTLOADER_KEY_Y : D5F2D045236CBEF3CB46B13BF7527AA36A26435DA6E8AFAF60037DA21AD7B2E1
    ```

6. Copy Public Command Key (X-point `50DF...` first, then Y-point `D5F2...`) to **Command Key:** box.

   ![screenshot](./resources/sld714-image78.png)

7. Click [**Write**] to provision the Public Command Key.

8. Click [**Enable**] next to **Enable Secure Debug Unlock:** to enable the secure debug functionality.

   ![screenshot](./resources/sld714-image79.png)

9. Click [**Enable**] next to **Enable Debug Lock:** to lock the device. This configures secure debug lock.

   ![screenshot](./resources/sld714-image81.jpeg)

10. Click [**Disable**] next to **Disable Device Erase:** to disable the device erase. The following **Disable Device Erase Warning** is displayed. Click [**Yes**] to confirm.

   ![screenshot](./resources/sld714-image83.png)

   > **Important**: This is an **IRREVERSIBLE** action, and should be the last step in production. While evaluating Secure Debug functionality, it is best to not enable this option.

### Secure Debug Unlock Token Provision Simplicity Studio

Use the Debug Unlock Token file (`unlock_payload_0000000000111110.bin`) generated in [Secure Debug Unlock Using Commander Tool](#secure-debug-unlock-using-commander-tool) steps 8 or 9 to unlock the device with Simplicity Studio.

1. Open the `unlock_payload_0000000000111110.bin` file with the Hex File editor.

   ![screenshot](./resources/sld714-image85.jpeg)

2. Click **View** to open the context menu, and then select **Group By** → **Double words** to convert the token into a little-endian format.

   ![screenshot](./resources/sld714-image86.jpeg)

3. Select all (Ctrl+A) and copy (Ctrl+C) the Debug Unlock Token to a text editor.

   ![screenshot](./resources/sld714-image87.jpeg)

4. Use the text editor to remove all the spaces from the token.

5. Right-click the selected debug adapter **RB (ID:J-Link serial number)** to display the context menu.

   ![screenshot](./resources/sld714-image88.png)

6. Click **Set Unlock Token** to open the **Add Debug Unlock Token** dialog box. Enter the name (e.g., `AN1190 Token`) for this Debug Unlock Token, and copy the content in step 4 to the **Debug Unlock Token:** box. Click [**OK**] to confirm and exit.

   ![screenshot](./resources/sld714-image89.png)

   > **Note**: The Simplicity Studio can only keep one Debug Unlock Token on each WSTK.

7. Open Security Settings of the selected device as described in [Standard Debug Lock/Unlock Using Simplicity Studio](#standard-debug-lockunlock-using-simplicity-studio).

8. The token added in step 6 should be displayed on the **Crypto Profile:** field. If not, click the link next to **Crypto Profile:** to select the token from the **Crypto Profile Manager** drop-down list. The Simplicity Studio will automatically add the WSTK J-Link serial number (`-J-Link Silicon Labs (serial number)`) to the token's name.

   ![screenshot](./resources/sld714-jlink-serial-number.png)

9. Click [**Unlock Debug Port**] to use the token in **Crypto Profile:** to unlock the device (invalid token will display an error message). The device stays in the unlock state until the next power-on or pin reset. Click [**OK**] to exit.

   ![screenshot](./resources/sld714-image90.png)

10. The Simplicity IDE will automatically use the selected Debug Unlock Token in **Crypto Profile** for debugging and flashing.

   After finished debugging, open the Security Settings of the selected device as described in [Standard Debug Lock/Unlock Using Simplicity Studio](#standard-debug-lockunlock-using-simplicity-studio).

### Roll Challenge Using Simplicity Studio

1. Click [**Roll Challenge**] to generate a new challenge value to invalidate the Debug Unlock Token added in step 6. Click [**OK**] to exit.

   ![screenshot](./resources/sld714-image93.png)

2. Right-click the selected debug adapter **RB Board (ID:J-Link serial number)** to display the context menu.

   ![screenshot](./resources/sld714-image94.png)

3. Click [**Clear Unlock Token**] to delete the WSTK Debug Unlock Token from Simplicity Studio.

## Using Platform SE - Manager

Simplicity Studio 5 includes the SE Manager platform examples for Secure Tamper. Refer to the corresponding readme file for details about the SE Manager example. This file also includes the procedures to create the project and run the example about the SE Manager example. This file also includes the procedures to create the project and run the example.

<table>
    <thead>
        <tr>
            <th>Category</th>
            <th>SE Manager Platform Example</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Perform Secure Debug</td>
            <td>Platform Security - SoC SE Manager Secure Debug</td>
            <td>Sample application to:
                <ul>
                    <li>Set DEBUG_OPTION for TrustZone aware debug</li>
                    <li>Enable/Disable standard Debug Lock</li>
                    <li>Provision command key</li>
                    <li>Enable/Disable Secure Debug</li>
                    <li>Lock or Unlock debug port securely</li>
                    <li>Disable Device erase</li>
                    <li>Roll challenge</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>
