# Proven Cryptography

*Product security shall use strong, proven, updatable cryptography using open, peer-reviewed methods and algorithms.*

An important aspect of any IoT device is how secure the device is when it communicates with other devices, gateways, or the cloud. This standard mandates using proven cryptographic methods rather than attempting to implement your own.

Developers commonly secure communications such as TCP/IP connections, Bluetooth, Zigbee, or Z-Wave using the standardized and proven cryptographic methods native to the protocol. However, if a microcontroller sends sensitive information over a simple interface such as a UART to another microcontroller, it is important to realize that data should also be secured to prevent someone from snooping the UART line.

Silicon Labs offers a hardware accelerator module that provides an efficient acceleration of common cryptographic operations and allows these to be used efficiently with low CPU overhead. The cryptographic accelerator module includes hardware accelerators for the Advanced Encryption Standard (AES), optionally ChaCha20/Poly1305, Secure Hash Algorithm SHA-1 and SHA-2 (SHA-224 and SHA-256), and modular multiplication used in ECC (Elliptic Curve Cryptography) and GCM (Galois Counter Mode). The accelerator module can autonomously execute and iterate a sequence of instructions to aid software and speed up complex cryptographic functions like ECC, GCM, and CCM (Counter with CBC-MAC).

In addition to the cryptographic accelerator module, Silicon Labs includes the PSA Crypto and Mbed TLS libraries as part of the Gecko Platform SDK. Mbed TLS is open source software licensed by ARM Limited. It provides an SSL library that makes it easy to use cryptography and SSL/TLS in applications. Mbed TLS supports software implementations of all crypto algorithms that are supported by TLS 1.3 as well as a build API that allows hardware drivers to replace the software implementations when cipher accelerators are supported by the platform. Its modular framework allows for subcomponents like the crypto libraries to be incorporated into a design independently of the SSL/TLS components, saving valuable code space and runtime RAM. Mbed TLS supports SSLv3 up to TLSv1.3 communication by providing the following:

- TCP/IP communication functions: listen, connect, accept, read/write.

- SSL/TLS communication functions: init, handshake, read/write.

- X.509 functions: CRT, CRL and key handling

- Random number generation

- Hashing

- Encryption/decryption

These functions are split up into logical interfaces. They can be used separately to provide any of the above functions or to mix-and-match into an SSL server/client solution that utilizes a X.509 public key infrastructure (PKI). Examples of such implementations are provided with the source code. Components or plugins and APIs provide configuration interfaces accessible through the various SDK installations.

For more information, see the latest MCU and Peripheral Software Documentation for the target part at [https://docs.silabs.com.](https://docs.silabs.com/).
