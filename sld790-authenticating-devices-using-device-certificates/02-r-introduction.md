# Introduction

One of the biggest challenges for connected devices is post-deployment authentication. Silicon Labs’ factory trust provisioning service with optional secure programming provides a secure device identity certificate, analogous to a birth certificate, for each individual silicon die during integrated circuit (IC) manufacturing. This enables post-deployment security, authenticity, and attestation-based health checks. The device certificate guarantees the authenticity of the device for its lifetime. When the certificate is checked, a digital signature confirms that the certificate received has not been tampered with.

Certificates can now be used to authenticate Internet of Things (IoT) devices as well as Internet servers, now that Silicon Labs’ HSE-SVH devices have both cryptographic acceleration in hardware and tamper-resistant storage to handle digital certificate operations.

The digital signature and certificates are major cryptographic tools to verify the device is authentic. These tools are described in the following sections.

## Digital Signature

The digital signature is used to protect integrity and authenticity of an electronic message.

### Digital Signature Example

Alice wants to give data to Bob, and Bob wants to make sure that the data came from Alice and has not been tampered with. Alice has a private/public key pair, and has previously given Bob her public key.

![Digital Signature](./resources/sld790-digital-signature.png)

1. Alice  generates the hash (for example SHA256) of the data.

2. Alice's private key is used to sign the hash to create a signature. The hash is signed instead of the data itself because the signing operation is slow. Therefore it is more efficient to sign the hash instead of the arbitrarily large data.

3. The signature is attached to the end of the data.

4. The data and signature are given to Bob.

5. Bob independently generates the hash of the data.

6. The signature is verified with the hash and Alice’s public key, which results in a true or false outcome indicating if the data is valid.

>**Note**: This scheme requires distribution of Alice's public key.

## Digital Certificates and Chain of Trust

In [Digital Signature Example](#digital-signature), Bob already had access to Alice’s public key, which he trusted. However, it is not always feasible to pre-share a public key with everyone for secure identity verification, and no mechanism exists to revoke or inactivate the public key in case it gets stolen.

A digital certificate is simply a small, verifiable data file that contains identity credentials and a public key. That data is then signed either with the corresponding private key, or a different private key. The digital certificate can be used to prove the ownership of a public key.

- If it is signed using the corresponding private key, it is called a self-signed certificate.

- If it is signed by another private key, the owner of that private key is acting as a Certificate Authority (CA).

- A Certificate Authority (CA) is a trusted third party by both the owner and party relying on the certificate.

Concatenation of digital certificates builds a chain of trust.

- At the root of the chain is a self-signed certificate called a root certificate or a CA certificate.

- The root or CA certificate can be used to sign another certificate.

![Digital Certificates and Chain of Trust](./resources/sld790-digital-certificates-and-chain-of-trust.png)

>**Note**: The private key is never included as part of the certificate – it must be stored separately and kept private. The security of the scheme relies on protecting the private keys.

## Digital Certificates Verification

This section illustrates the process shown in [Digital Signature Example](#digital-signature), but using digital certificates.

### Digital Certificates Verification Example

Alice wants to give data to Bob, signed with her private key. Alice has a digital certificate signed by a trusted third party (CA) in addition to her private key. Bob has a certificate from the trusted CA but nothing else is previously shared.

![Digital Certificate Verification](./resources/sld790-digital-certificate-verification.png)

1. Alice uses her private key to sign the data.

2. Alice gives the data, signature, and her certificate to Bob.

3. Bob first verifies that Alice's certificate is valid, to prove Alice is the owner of the certificate's public key. This is done by verifying that Alice's certificate contains a valid signature created by the CA.

4. Bob then verifies the signature of the data using the public key in Alice's certificate.

>**Note**: The hash process in [Digital Signature Example](#digital-signature-example) is skipped in this example.
