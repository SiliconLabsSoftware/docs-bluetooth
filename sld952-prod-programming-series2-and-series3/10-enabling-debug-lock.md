# Enabling Debug Lock

The debug lock is an important feature to prevent attackers from using the debug interface to perform unauthorized operations on the device. The following sections describe how to apply three different locks to the Series 2 and Series 3 debug interface.

## Standard Debug Lock

The following command locks the debug interface.

```sh
commander security lock --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
WARNING: Secure debug unlock is disabled. Only way to regain debug access is to run a device erase.
Device is now locked.
DONE
```

To check the debug lock status of the device, run the `security status` command:

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.14
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Enabled
Device erase	    : Enabled
Secure debug unlock : Disabled
Tamper status	    : OK
Secure boot	        : Disabled
Boot status	        : 0x20 - OK
DONE
```

## Permanent Debug Lock

The following command locks the debug interface.

```sh
commander security lock --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
WARNING: Secure debug unlock is disabled. Only way to regain debug access is to run a device erase. Device is now locked.
DONE
```

After locking the device, disable the device erase using the following command. This is an **IRREVERSIBLE** action and should be the last step in production.

```sh
commander security disabledeviceerase --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
================================================================================
THIS IS A ONE-TIME command which Permanently disables device erase.
If secure debug lock has not been set, there is no way to regain debug access to this device.
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
Disabled device erase successfully
DONE
```

To check the debug lock status of the device, run the `security status` command.

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.14
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Enabled
Device erase        : Disabled
Secure debug unlock : Disabled
Secure boot         : Disabled
Boot status         : 0x20 - OK
DONE
```

## Secure Debug Lock

The Secure Debug feature is enabled through the `security lockconfig` command. After locking the device, the security unlock command securely unlocks the device for debugging until the next device reset without erasing flash and RAM contents. For more information about Secure Debug Unlock, see [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

The following command enables the secure debug unlock.

```sh
commander security lockconfig --secure-debug-unlock enable --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Secure debug unlock was enabled
DONE
```

For **TrustZone-unaware** applications, after enabling the Secure Debug feature, lock the debug interface using the following command.

```sh
commander security lock --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Device is now locked.
DONE
```

For **TrustZone-aware** applications, after enabling the Secure Debug feature, set the debug options (e.g., `1100`) and lock the debug interface using the following command.

```sh
commander security lock --trustzone 1100 --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
Writing debug restriction bits:
DBGLOCK:	0
NIDLOCK:	0
SPIDLOCK: 1
SPNIDLOCK: 1
Device is now locked.
DONE
```

**Notes**:

- The `--trustzone` option for the `security lock` command requires Simplicity Commander **≥ v1.13.3**.

- It is strongly recommended to upgrade to SE firmware **≥ v1.2.14** (xG21 and xG22) or **≥ v2.2.1** (other Series 2 devices) so that the debug options cannot be modified after the device is locked.

- Use `commander security lock` without the `--trustzone ####` option if the default setting of debug options (`0000`) is good enough for a TrustZone-aware application.

- For more information about debug options, see the *TrustZone Debug Authentication* section in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

After locking the device, disable the device erase using the following command. This is an **IRREVERSIBLE** action and should be the last step in production.

```sh
commander security disabledeviceerase --device EFR32MG21A010F1024 --serialno 440048205
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

> **Note**: The debug options cannot be reset to the default value `0000` (unlock) if the `device erase option` is disabled.

Run the `security status --trustzone` command to check the full debug lock status of the device.

```sh
commander security status --trustzone --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.14
Serial number	    : 000000000000000014b457fffe045a8e
Debug lock	        : Enabled
Device erase	    : Disabled
Secure debug unlock : Enabled

Debug lock state: Locked

Non-secure, invasive debug lock     (DBGLOCK) : Unlocked
Non-secure, non-invasive debug lock (NIDLOCK) : Unlocked 
Secure, invasive debug lock	        (SPIDLOCK) : Locked 
Secure, non-invasive debug lock	    (SPNIDLOCK): Locked

Non-secure, invasive debug lock state     (DBGLOCK) : Unlocked
Non-secure, non-invasive debug lock state (NIDLOCK) : Unlocked
Secure, invasive debug lock state	      (SPIDLOCK) : Locked
Secure, non-invasive debug lock state	  (SPNIDLOCK): Locked

Tamper status:      : OK
Secure boot         : Disabled
Boot status         : 0x20 - OK 
DONE
```

> **Note**: For more information about Secure and Non-secure debug locks, see the *TrustZone Debug Authentication* section in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).
