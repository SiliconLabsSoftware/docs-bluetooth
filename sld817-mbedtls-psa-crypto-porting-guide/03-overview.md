# Overview

## Mbed TLS

Mbed TLS is a C library that implements cryptographic primitives, X.509 certificate manipulation, and the SSL/TLS and DTLS protocols.

ARM developed Mbed TLS, which was formerly known as PolarSSL. Mbed TLS has been handed over to [Trusted Firmware](https://www.trustedfirmware.org/projects/mbed-tls/) under open governance since March 2020.

For the time being, Trusted Firmware Mbed TLS is the project containing a reference implementation of the PSA Crypto API and the TLS portion of Mbed TLS. The following table lists different Mbed TLS versions supported in Simplicity Studio and Gecko SDK (GSDK) suites.

:::custom-table{width=10%,20%,10%,70%}
| **Mbed TLS** | **Gecko SDK Suite** | **PSA Crypto** | **Location in Windows** |
|--------------|---------------------|----------------|-------------------------|
| v3.2.1 | v4.3.x (Simplicity Studio 5) | Y | C:\Users\\<PC USER NAME\>\SimplicityStudio\SDKs\gecko_sdk\util\third_party\mbedtls |
| v3.2.1 | v4.2.x (Simplicity Studio 5) | Y | C:\Users\\<PC USER NAME\>\SimplicityStudio\SDKs\gecko_sdk\util\third_party\mbedtls |
| v3.1.0 | v4.1.x (Simplicity Studio 5) | Y | C:\Users\\<PC USER NAME\>\SimplicityStudio\SDKs\gecko_sdk\util\third_party\crypto\mbedtls |
| v3.0.0 | v4.0.x (Simplicity Studio 5) | Y | C:\Users\\<PC USER NAME\>\SimplicityStudio\SDKs\gecko_sdk\util\third_party\crypto\mbedtls |
| v2.26.0 | v3.2.x (Simplicity Studio 5) | Y | C:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\v3.2\util\third_party\crypto\mbedtls |
| v2.24.0 | v3.1.x (Simplicity Studio 5) | Y | C:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\v3.1\util\third_party\crypto\mbedtls |
| v2.16.6 | v3.0.x (Simplicity Studio 5) | — | C:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\v3.0\util\third_party\mbedtls |
| v2.7.12 | v2.7.8 (Simplicity Studio 4) | — | C:\SiliconLabs\SimplicityStudio\v4\developer\sdks\gecko_sdk_suite\v2.7\util\third_party\mbedtls |
:::

## PSA Crypto

**Platform Security Architecture (PSA)**

The [Platform Security Architecture](https://developer.arm.com/architectures/security-architectures/platform-security-architecture) (PSA) is made up of four key stages:

- Threat modeling

- Predefined architectural choices

- Standardized implementation

- Certification

The PSA Crypto API is one of the standardized implementation features and is discussed in the following sections.

**PSA Root of Trust (PSA-RoT)**

For an IoT product to achieve its security goals, it must meet the requirements of one of the pillars known as Root of Trust. The following figure shows the four PSA-Certified key elements that make up the Root of Trust.

![Key Requirements of PSA-RoT](./resources/sld817-image8.png)

The PSA Root of Trust (PSA-RoT) is a source of confidentiality (for example, crypto keys) and integrity. The PSA-RoT defines what it takes for a hardware or software system to be trusted.

The [Trusted Firmware-M](https://www.trustedfirmware.org/projects/tf-m/) (TF-M) offers an open-source firmware reference implementation and APIs. These resources provide developers with a trusted codebase that complies with PSA specifications and APIs that create a consistent interface to underlying Root of Trust hardware.

**PSA Functional APIs**

[PSA Certified](https://www.psacertified.org/) defines a set of PSA Functional APIs (which are implemented as part of TF-M) to access the Root of Trust features. The PSA Functional APIs provide a standardized set of vetted APIs to ensure portability and promote adherence to best practices.

- PSA Crypto APIs

- PSA Attestation APIs

- PSA Secure Storage (Internal Trusted Storage and Protected Storage) APIs

Since the Trusted boot (aka Secure boot) shown in [Figure Key Requirements of PSA-RoT](#psa-crypto) is used when booting up the device and is not used after the system is up and running, there is no need for a Trusted boot API.

The three APIs provide software developers with access to security functions to ensure interoperability across different hardware implementations of the Root of Trust. It means another hardware platform can reuse the applications in the following figure, because these APIs are standardized across various security hardware.

![PSA Functional APIs](./resources/sld817-image9.png)

[PSA Functional API Certification](https://www.psacertified.org/getting-certified/functional-api-certification/) is part of [PSA Certified](https://www.psacertified.org/), and demonstrates that software is compatible with the PSA Functional API specification. PSA Functional API Certified does not imply that a device has a security capability or is robust. Only PSA Certified Levels 1–3 can achieve this.

This application note only focuses on the PSA Crypto API.

**PSA Crypto API**

The PSA Crypto API is a low-level cryptographic API optimized for MCU and Wireless SoC. It provides APIs related to [Random Number Generation](./06-migration-guide.md#initialization-and-random-number-generation-rng) (RNG), cryptographic algorithm usage, and [key handling](./06-migration-guide.md#key-handling) (symmetric and asymmetric).

The PSA Crypto API provides developers with an easy-to-use and easy-to-learn interface to crypto primitives. It is designed for usability and flexibility and is based on the idea of a key store. The store can isolate the keys from the rest of the applications, which means keys remain opaque in [storage](./04-key-management-in-psa-crypto.md#key-lifetimes) and only accessible for usage through crypto primitives.
