# Using Third Generation Non-Volatile Memory (NVM3) Data Storage

> **Note: This section replaces *AN1135: Using Third Generation Non-Volatile Memory (NVM3) Data Storage*. Further updates to this application note will be provided here**.

The NVM3 driver provides a means to write and read data objects (key/value pairs) stored in flash. Wear-leveling is applied to reduce erase and write cycles and maximize flash lifetime. The driver is resilient to power loss and reset events, ensuring that objects retrieved from the driver are always in a valid state. A single NVM3 instance can be shared among several wireless stacks and application code, making it well-suited for multiprotocol applications. This application note explains how NVM3 can be used as non-volatile data storage in Zigbee (EmberZNet), Open Thread, Z-Wave, Bluetooth, Connect, and WiseConnect applications.

Version 3.x of the Gecko SDK Suite, used with Simplicity Studio 5, introduced a new component-based project architecture that replaced AppBuilder. Eventually all wireless protocols will move to the component-based architecture. This document addresses both this new approach to NVM3 configuration as well as the AppBuilder configuration still in use.

## Key Points

- Key/value pair data object storage in flash
- Wear-leveling to maximize flash lifetime
- Resilient to power and reset events
- Shared by Zigbee, Connect, OpenThread, Z-Wave, and Bluetooth stacks
- Compatible with PS Store and Token APIs through wrappers
- Data upgradable from Simulated EEPROM version 2 to NVM3
