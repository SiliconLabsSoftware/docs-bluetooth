# Device Certificate Options

The HSE-SVH devices are each programmed with a device certificate during IC production. The device certificate is signed with a Public Device Key, using a Private Batch Key that can be validated against a Silicon Labs certificate chain [Verification for Certificates](./03-r-secureidentity.md#signing-and-verification) and [Certificate Chain Verification](./08-r-examples.md#certificate-chain-verification). The device private key never leaves the Secure Key Storage on the chip. Customers can create their own device certificates during their production.

Three device certificate options (standard, modified, and external) are provided to meet different requirements. Silicon Labs provides [Custom Part Manufacturing Service (CPMS)](https://www.silabs.com/developers/custom-part-manufacturing-service) to program custom certificates on your chips at the Silicon Labs factories. For more information about CPMS, see [Custom Part Manufacturing Service User's Guide](https://docs.silabs.com/iot-security/latest/iot-security-cpms/).

## Standard Device Certificate

- Comes standard with HSE-SVH devices.

- Cryptographically proves the device is an authentic Silicon Labs device.

- Does not protect against overproduction or counterfeit products that are built with authentic Silicon Labs devices.

- Signed to a Silicon Labs Certificate Authority (CA).

- The device can prove that it possesses the private key associated with the public key in its certificate by signing the response to a given challenge ([Remote Authentication Process](./06-r-remoteauthenticationprocess.md) and [Certificate Chain Verification and Remote Authentication](./08-r-examples.md#certificate-chain-verification-and-remote-authentication)).

![Standard Device Certificate](./resources/sld790-standard.png)

## Modified Device Certificate

- Available as a customization service on HSE-SVH devices (OEM custom part number).

- Cryptographically proves the device is an authentic Silicon Labs device that was produced for a specific OEM.

- Protects against overproduction by Contract Manufacturer (CM).

- Device Certificate X.509 fields can be specified, with restrictions.

- Signed to a Silicon Labs Certificate Authority (CA).

![Modified Device Certificate](./resources/sld790-modified.png)

## External Device Certificate

- Available as a customization service on HSE-SVH devices (OEM custom part number).

- Cryptographically proves the device is an authentic Silicon Labs device that was produced for a specific OEM.

- Protects against overproduction by Contract Manufacturer (CM).

- Factory Certificate is custom for each OEM.

- Device Certificate and Factory Certificate X.509 fields can be specified, with restrictions.

- Signed to a OEM Certificate Authority (CA).

- Root Certificate Authority is OEM-specified and is optional.

- Electronic delivery of all batch and device certificates signed under this OEM factory certificate is supported.

![External Device Certificate](./resources/sld790-external.png)
