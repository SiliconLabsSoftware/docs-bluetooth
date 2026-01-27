# Dynamic Data Storage Implementations

This chapter introduces some of the dynamic data storage implementations offered by Silicon Labs, including SimEEv1 and SimEEv2, PS Store, and NVM3.

## SimEEv1/v2

SimEEv1/v2 can be used with EmberZNet PRO and Silicon Labs Connect (installed with the Flex SDK). SimEEv1 uses two virtual pages with a fixed total size of 8 kB, while SimEEv2 uses three virtual pages with a fixed total size of 36 kB.

One characteristic of SimEEv1/v2 is that all objects are defined with size and type at compile time, hence a new object cannot be created or deleted at runtime.

Silicon Labs provides a plugin for upgrading SimEEv1 data to SimEEv2.

Information about the SimEEv1/v2 implementations is found in *AN703: Using Simulated EEPROM Version 1 and Version 2 for the EFR32 Series 1 SoC Platform*.

## PS Store

PS Store can be used with Bluetooth devices on all platforms except for EFR32 Series 2. PS Store API commands are used to manage user data in PS keys in the flash memory of the Bluetooth device. User data stored within the flash memory is persistent across reset and power cycling of the device. The persistent store size is 2048 bytes and uses two flash pages for storage. As Bluetooth bondings are also stored in this area, the space available for user data also depends on the number of bondings the device has at the time. The size of a Bluetooth bonding is around 150 bytes. With its simple implementation and few storage flash pages, PS Store is the smallest of Silicon Labs' non-volatile storage options. PS Store allows objects to be created and deleted at runtime.

Information about the PS Store APIs may be found in the *Bluetooth API Software Reference Manual*.

## NVM3

The third generation Non-Volatile Memory (NVM3) data storage driver is an alternative to SimEEv1/v2 and PS Store. The NVM3 driver provides a means to write and read data objects (key/value pairs) stored in flash. Wear-leveling is applied to reduce erase and write cycles and maximize flash lifetime. The driver is resilient to power loss and reset events, ensuring that objects retrieved from the driver are always in a valid state. Because NVM3 can be used with several protocols, such as Bluetooth and EmberZNet PRO, it allows a single data storage instance to be shared in Dynamic Multiprotocol (DMP) applications.

Some of the main features of NVM3 are as follows:

- Key/value pair data storage in flash

- Runtime creation and deletion of objects

- Persistence across power loss and reset events

- Wear-leveling to maximize flash lifetime

- Object sizes from 0 to 4096 bytes

- Configurable flash storage size (minimum 3 flash pages)

- Cache with configurable size for fast object access

- Data and counter object types

- Compatibility layers with token and PS Store APIs provided

- Single shared storage instance in multiprotocol applications

- Repack API to allow application to run clean-up page erases during periods with low CPU load

Detailed information on NVM3 is documented in the EMDRV->NVM3 section of the [Gecko HAL & Driver API Reference Guide](http://devtools.silabs.com/dl/documentation/doxygen/). Users who are accessing NVM3 through its native API should refer to this API reference guide for information. Users who are developing dynamic multiprotocol applications should refer to [Using Third Generation Non-Volatile Memory (NVM3) Data Storage](https://docs.silabs.com/gecko-platform/latest/using-third-generation-nonvolatile-memory/).
