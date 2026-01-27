# Privacy and Tracking

This section discusses privacy for mobile Bluetooth Low Energy devices.

Many Bluetooth Low Energy devices will be wearable or other mobile devices. If the device is constantly advertising the same address, it is easy for the device’s location to be tracked.

## Bluetooth Address Types

The Bluetooth specification defines several address types that are defined below. All Bluetooth addresses are 48 bits in length.

**Identity Address**: a type of address that can be used in forming a new bond. Either public addresses or random static addresses qualify as identity addresses.

**Public address**: A device’s public address is usually derived from the hardware and does not change over time.

**Resolvable private address**: This address type is random but can be resolved by a bonded device in possession of an identity-resolving key (IRK).

**Non-resolvable private address**: This is a completely random address that is only used in non-connectable advertising. The identity address cannot be determined by observers if they only know the non-resolvable private address.

**Random Static address**: All but the two upper-most bits are random. A new random static address may be chosen after a power cycle event.

**Anonymous**: Bluetooth 5.x style extended advertising supports the ability to advertise without any address being sent along with the advertising payload.

### LE Privacy

LE Privacy is a feature that was introduced in Bluetooth 4.1. When this feature is enabled, a new resolvable private address is chosen periodically by the stack. LE Privacy can be enabled by calling [sl_bt_gap_set_privacy_mode()](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-gap#sl-bt-gap-set-privacy-mode). It is recommended to use this feature on devices where tracking is a concern.

### Device Privacy

When a device is in device privacy mode, it is only concerned about its own privacy. It should accept advertising packets from peer devices that contain their Identity Addresses as well as their private address, even if the peer device has distributed its IRK.

### Network Privacy

When a device is in network privacy mode, it shall not accept advertising packets containing the Identity Address of peer devices that have distributed their IRK; that is, only resolvable private address (RPA) is accepted for peer devices that have distributed their IRK.

If the Resolvable Private Address Only characteristic is not present in the GAP service of the remote device, it may use its Identity Address over the air.

### Working with LE Privacy

#### Address Resolving List

Silicon Labs Bluetooth stack maintains an address resolving list for devices using LE privacy. Devices can be referred to by the bonding handle or by address. It is documented in the section Address Resolving List of the  [Bluetooth API Reference](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-resolving-list).

- Bonding Handle

  Adding a device to the address resolving list requires the bonding handle and privacy mode of the device. The privacy mode setting indicates whether the local device uses device privacy or network privacy for the remote device. Removing a device through its bonding handle only requires the bonding handle. Note: deleting the bonding does not remove the device from the address resolving list nor from the filter accept list.

- Address

  Adding a device to the address resolving list by its address requires the identity address of the device, the type of address, device’s IRK, and the privacy mode. The privacy mode setting indicates whether to use device privacy or network privacy for the peer device. Removing a device from the address resolving list requires the address to be removed and the address type.

#### Filter Accept List and Advertisement Filtering

Silicon Labs’ Bluetooth stack maintains a filter accept list which can used to filter out advertisements from any device that is not in the list. As with the address resolving list, devices can be added and removed from the filter accept list either by bonding handle or by address. The filtering policy can be set by using the [sl_bt_scanner_set_parameters_and_filter()](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-scanner#sl-bt-scanner-set-parameters-and-filter) API function.

- Bonding Handle

  Devices can be added to the filter accept list using the [bonding handle](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-accept-list#sl-bt-accept-list-add-device-by-bonding).

- Address

  Devices can be added to the filter accept list using the [peer address and the address type](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-accept-list#sl-bt-accept-list-add-device-by-address).

> **Note**: Series 1 devices may not support LE Privacy Feature 1.2v as it was introduced with core specification v4.2 onwards.
