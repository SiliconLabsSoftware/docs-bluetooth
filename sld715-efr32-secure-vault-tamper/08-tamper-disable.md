# Tamper Disable

For diagnostic purposes, it may be necessary to disable the tamper response. For example, if a user has configured the part to [Erase OTP](./04-tamper-responses.md#erase-otp) on external tamper detection, disabling the tamper response is required to open the unit and perform failure analysis or field service activities.

After the tamper configuration has been initialized, users can temporarily restore the tamper response to default for a set of tamper sources via a Disable Tamper Token authenticated against the Public Command Key in HSE OTP (similar to secure debug unlock). This is only possible if the Public Command Key has been provisioned in the device.

![Tamper Disable on the EFR32xG21B Devices](./resources/sld715-tamper-disable-efr32xg21b-devices.png)

**For default tamper settings on other devices, refer to [Appendix C – Tamper Sources Reference](./12-appendix-c-tamper-sources-reference.md).**

## Disable Tamper Token

The elements of the Disable Tamper Token are described in the following figure and tables.

![Disable Tamper Token](./resources/sld715-disable-tamper-token.png)

| **Element** | **Value** | **Description** |
|-------------|-----------|-----------------|
| Disable tamper command | 0xfd020001 | The command word of the Disable Tamper Token. |
| Tamper disable mask | Device-dependent | The command parameter of the Disable Tamper Token. |
| Access certificate (1) | Device-dependent | See [Access Certificate](#access-certificate). |
| Disable tamper command signature (1) | Device-dependent | See [Challenge Response](#challenge-response). |

**Note**:

1. The disable tamper command payload consists of an access certificate and a disable tamper command signature.

![Tamper Disable Mask](./resources/sld715-tamper-disable-mask.png)

**Note**: Set bit to restore the default response of the corresponding tamper source.

The Disable Tamper Token temporarily reverts all masked tamper sources in the figure above to the hard-coded configuration.

The Disable Tamper Token can only restore the escalated user-level configuration to default. It cannot degrade the default level of a tamper source.

## Access Certificate

The elements of the access certificate are described in the following figure and table.

![Access Certificate](./resources/sld715-access-certificate.png)

| **Element** | **Value** | **Description** |
|-------------|-----------|-----------------|
| Magic word | 0xe5ecce01 | A constant value used to identify the access certificate. |
| Authorizations | 0x0000003e (1) | A value used to authorize which bit in the debug mode request can be enabled for secure debug. |
| Tamper Authorizations | 0xffffffb6 (2) | A value used to authorize which bit in the tamper disable mask can be set to disable the tamper response. |
| Serial number | Device-dependent | A number used to compare against the on-chip serial number for secure debug or tamper disable. |
| Public Certificate Key (3) | Device-dependent | The public key corresponding to the Private Certificate Key (3) used to generate the signature (ECDSA-P256-SHA256) in a challenge response. |
| Access certificate signature | Device-dependent | All the content above is signed (ECDSA-P256-SHA256) by the Private Command Key corresponding to the Public Command Key in the HSE OTP. |

**Notes**:

1. The value allows all debug options to be reset for secure debug. Note that the commands for debug unlock and tamper disable are separate, so the secure debug lock will not be disabled when issuing a tamper disable command.

2. Value that sets available bits in the tamper disable mask for tamper disable.

3. The Private/Public Certificate Key is a randomly generated key pair. It can be ephemeral or retainable.

The Private Certificate Key can be used repeatedly to generate the signature in a challenge response on one device until the Private/ Public Certificate Key pair is discarded. This can reduce the frequency of access to the Private Command Key, allowing more restrictive access control on that key.

For more information about secure debug, see [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/).

![Tamper Authorizations](resources/sld715-image43.jpeg)

**Notes**:

- Set the bit to enable the corresponding bit in the tamper disable mask.

- The Disable Tamper Token will restore the default response of the corresponding tamper source if the same bit is set in the tamper disable mask and tamper authorizations. If the same bit is not set, the tamper response for that tamper source will be untouched.

## Challenge Response

The elements of the challenge response are described in the following figure and table.

![Challenge Response](./resources/sld715-challenge-response.png)

| **Element** | **Value** | **Description** |
|-------------|-----------|-----------------|
| Disable tamper command | 0xfd020001 | The command word of Disable Tamper Token. |
| Tamper disable mask | Device-dependent | The command parameter of Disable Tamper Token. |
| Challenge | Device-dependent (1) | A random value generated by the HSE. |
| Disable tamper command signature | Device-dependent (2) | All the content above is signed (ECDSA-P256-SHA256) by the Private Certificate Key corresponding to the Public Certificate Key in the access certificate. |

**Notes**:

1. The challenge remains unchanged until it is updated to a new random value by rolling the challenge. The Private Certificate Key can be reused for signing when device challenge is refreshed.

2. This signature is the final argument of the Disable Tamper Token.

## Tamper Disable Flow

The tamper disable flow is described in the following figure.

![Tamper Disable Flow](./resources/sld715-tamper-disable-flow.png)

1. Get the serial number and challenge from the HSE.

2. Generate the access certificate with device serial number.

3. Generate the challenge response with device challenge.

4. Generate the disable tamper command payload with access certificate and disable tamper command signature.

5. Send the Disable Tamper Token to the HSE.

6. Verify the disable tamper command signature using the Public Certificate Key in the access certificate.

7. Verify the serial number and the access certificate signature using the on-chip serial number and Public Command Key in the HSE OTP.

8. Restore default levels on tamper disable mask until the next power-on or pin reset.

9. Roll the challenge to invalidate the current Disable Tamper Token.

> **Note**: Refer to the [Simplicity Commander example](./09-examples.md#using-simplicity-commander) for details on how to follow this flow using Simplicity Commander.
