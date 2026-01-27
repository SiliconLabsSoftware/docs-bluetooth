# Remote Authentication Process

Remote authentication is used to manage attestation by requesting that the device sign a challenge or [EAT](./05-r-entityattestationtoken.md) based on its secure identity.

![Remote Authentication Process](./resources/sld790-remote-authentication-process.png)

1. The remote device requests the device certificate and batch certificate from the HSE-SVH device.

2. The remote device looks up the factory certificate and root certificate from the Silicon Labs Server.

3. The remote device validates each certificate in the chain using the public key of each Issuer ([Verification for Certificates](./03-r-secureidentity.md#signing-and-verification-signing-and-verification)).

4. The remote device then sends an attestation challenge (random number) to the HSE-SVH device. The HSE-SVH device uses the Private Device Key in the Secure Key Storage on the chip to sign the challenge or EAT and sends the signature of challenge or EAT to the remote device.

5. The remote device requires a small library to validate the signature of challenge or EAT using the Public Device Key in the device certificate.
