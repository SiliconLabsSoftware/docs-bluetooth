# Overview

The third generation Non-Volatile Memory (NVM3) data storage driver is for storing persistent data in flash.

NVM3 is designed to work on all Silicon Labs wireless stacks running on EFR32, Series-3, and Si91x devices, as well as MCU applications running on EFM32.

Some of the main features of NVM3 are:

- Key/value pair data storage in flash

- Runtime object creation and deletion

- Persistence across power loss and reset events

- Wear-leveling to maximize flash lifetime

- Object sizes configurable up to 4096 bytes

- Configurable flash storage size (minimum 3 flash pages)

- Cache with configurable size for fast object access

- Data and counter object types

- Compatibility layers included for several Silicon Labs persistent storage APIs

- Single shared storage instance in multiprotocol applications

- Repack API to allow application to run clean-up page erases during periods with low CPU load

NVM3 is described in detail in the NVM3 Documentation on [https://docs.silabs.com/](https://docs.silabs.com/). The rest of this document assumes you are familiar with that content.

While the NVM3 API can be used directly, NVM3 is also used as the underlying storage mechanism for several other persistent storage APIs provided by Silicon Labs:

- Token API used in Zigbee EmberZNet and Connect applications

- Persistent Storage API used in Silicon Labs Bluetooth applications
