# Secure Key Storage

> **Note: This section replaces *AN1271: Secure Key Storage*. Further updates to this application note will be provided here**.

Secure Key Storage is a feature in  High devices that allows for the protection of cryptographic keys by key wrapping. User keys are encrypted by the device's root key for non-volatile storage for later usage. This prevents the need for a key to be stored in plaintext format on the device, preventing attackers from gaining access to the keys through traditional flash-extraction or application attacks, and allowing for a potentially unlimited number of keys to be securely stored in any available storage.

Series 2 devices can use TrustZone to implement Secure Key Storage, so this feature is now also available on  Mid devices.

This document describes the operation and usage of this feature, and provides comparisons with other key storage methods.

## Key Points

- Keys are encrypted or 'wrapped' with a  root key
- root key is not stored on the device, instead it is generated on each reset
- Wrapped keys are confidential to the , and can be stored in non-volatile memory safely
- Wrapped keys can be cached into  for usage at a later time
- TrustZone Secure Key Storage
