# Fundamentals - Connections

Bluetooth Low Energy (BLE) is a powerful and complex technology. It is not like classic Bluetooth with a predefined set of official profiles to choose from. Although there are predefined (a.k.a. "adopted") [profiles specified by the Bluetooth SIG](https://www.bluetooth.com/specifications/gatt), these are just the tip of the iceberg, a small subset of the functionality you can achieve with BLE.

In many (or even most) cases, the best option is to create a custom profile(s) for your application because it provides ultimate flexibility and doesn't cost anything. In fact, it can be even easier than using one of the adopted profiles because you get to define how everything works rather than conforming your application into something that is already defined. Also, because there is no official generic "serial port profile" in the BLE world like SPP in classic Bluetooth, sometimes a custom implementation is the only option to do what you need.

To have an effective custom implementation, it is important to understand how a BLE connection works, what roles are played by the devices involved, and how data is transferred from one device to the other over the air. Many terms are used, they are usually not interchangeable, and mean different things: **central, peripheral, client, server, advertise, scan, read, write, notify, and indicate**. Understanding the terminology will make it easier to describe and build your BLE application.

## Quick Overview

This page covers the following:

- **Central** devices scan for other devices, and initiate connection. Usually, the central is the smartphone/tablet/PC.
- **Peripheral** devices advertise and wait for connections. Usually, the peripheral is the BGMxxx module or EFR32 device.

To learn about server and client roles, which are a bit different, see [GATT Server and Client Roles](/bluetooth/{build-docspace-version}/bluetooth-gatt/gatt-server-client-roles).

## Central vs. Peripheral - Connection Roles

An important concept in BLE connectivity is the difference between a **central** device and a **peripheral** device. First, the terms are not interchangeable with client/server. In the BLE world, the central/peripheral difference is very easy to define and recognize:

- **Central**: the BLE device which initiates an outgoing connection request to an advertising peripheral device
- **Peripheral**: the BLE device which accepts an incoming connection request after advertising

The BLE specification does not limit the number of peripherals a central may connect to, but there is always a practical limitation, especially on small embedded modules. The Silicon Labs BLE stack can support up to **32 simultaneous connections** device, when properly configured (the default is 4). The stack supports dual-topology and multi-central connections (which are part of the Bluetooth 4.1 spec), which means that a device can be simultaneously in a central and in a peripheral role and it can also connect to multiple central devices at the same time. The connection limit applies to the total number of connections regardless of the role.

The connection role, whether a device is a central or peripheral, is defined the moment the connection is made. The Silicon Labs stack is capable of acting either as a central or as a peripheral device. If a device is operating as a peripheral, it needs to advertise (accomplished in Silicon Labs' BLE stack with the *sl_bt_advertiser_start* command). If it is operating as a central, it will scan for devices (accomplished with *sl_bt_scanner_start*) and initiate a connection request to another device (accomplished with *sl_bt_connection_open*).
