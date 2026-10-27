# Introduction

The Bluetooth specification makes security and privacy features optional. While using these features is not required by the specification, it is highly recommended to take advantage of the highest security features available. The Bluetooth SIG has published the following [Security and Privacy Best Practices Guide](https://www.bluetooth.com/bluetooth-resources/bluetooth-security-and-privacy-best-practices-guide/). This guide contains information for implementers of stacks as well as applications. Silicon Labs suggests following as many of these recommended best practices as possible.

## Threat Models

Several threats exist when communicating between two parties:

- Passive eavesdropping - an unauthorized third-party intercepts sensitive data.

- Man-in-the-middle (MITM) - an unauthorized third party injects or modifies data.

- Denial-of-Service through Wi-Fi coexistence.

- Tracking - an unauthorized third party can track the location of a moveable device.

- Spoofing – a device mimics the identity of a trusted device.

## Bluetooth Security Concepts

This section describes some basic concepts and terminology regarding Bluetooth security.

### Connections, Pairing and Bonding

A connection is required for reliable data exchange between two Bluetooth Low Energy devices as well as encryption and authentication, which are optional.

Pairing refers to a one-time secure relationship between two devices to establish cryptographic keys and allows data to be exchanged securely for the life of the connection. Loss of the connection results in termination of the pairing.

Bonding refers to a persistent relationship where cryptographic keys are established and stored in non-volatile memory and can exist over multiple connections.

LE Secure Connections refers to a method for exchanging cryptographic keys using the elliptic curve Diffie-Helman (ECDH) technique.

### Security Modes and Levels

Security mode 1 is the only mode supported for Bluetooth Low Energy in the Silicon Labs’ stack. The levels are as follows:

- Level 1 - no security

- Level 2 - unauthenticated pairing with encryption

- Level 3 - authenticated pairing with encryption

- Level 4 - authenticated secure connections with strong encryption (ECDH key exchange)

### GATT Database

Security in Bluetooth Low Energy is primarily controlled through GATT characteristics, which can have the following properties:

- Encrypted read/write/notify/indicate

- Authenticated read/write/notify/indicate

- Authorized read/write/notify/indicate 

A GATT client attempting to access a characteristic must support the properties required by the characteristic.
