# Device Capability

The following table lists the hardware related to cryptography hardware acceleration features on Series 1 and Series 2 devices (MCU and Wireless SoC).

| **Feature** | **Series 1** | **Series 2 - VSE** | **Series 2 - HSE** |
|-------------|--------------|--------------------|--------------------|
| TRNG | TRNG peripheral (1) | CRYPTOACC peripheral | HSE |
| Crypto Engine (2) | CRYPTO peripheral | CRYPTOACC peripheral | HSE |
| Advanced Crypto (3) | — | — | HSE-SVH |
| Secure Key Storage (4) | — | — | HSE-SVH |

>**Notes**:
>
>1. See [Table Entropy Source on Series 1 and Series 2 Devices](./06-migration-guide.md#initialization-and-random-number-generation-rng) for details of TRNG (True Random Number Generator) on Series 1 devices.
>
>2. Crypto engine supports up to 256-bit ciphers and elliptic curves.
>
>3. Advanced crypto supports up to 512-bit ciphers and 521-bit elliptic curves.
>
>4. See [Table PSA Crypto Key Lifetime Support on Series 1 and Series 2 Devices](./04-key-management-in-psa-crypto.md#key-lifetimes) for details of Secure Key Storage support on HSE-SVH devices.
