# NVM3 Default Instance

Several NVM3 instances can be created on a device and live independently of each other, but to save memory it is usually desirable to use only one NVM3 instance as each instance adds some overhead. For applications based on Silicon Labs wireless stacks, a common default instance is used. This allows Dynamic Multiprotocol (DMP) applications that combine several wireless stacks to share the same NVM3 instance.

The number of flash pages used for the NVM3 default instance is configurable, but this setting must match if an application includes several stacks that all use the default instance.

> **IMPORTANT**: When creating an application that includes an NVM3 instance for a device that already contains an NVM3 instance in flash, the number of flash pages configured for the NVM3 instance must match the number of flash pages for the NVM3 instance already found on the device. Therefore it is not possible to change the size of an NVM3 instance once it has been installed on a device, without first erasing the flash pages holding the NVM3 instance and the NVM3 objects stored there.

NVM3 has a cache to speed up access to NVM3 objects. The cache size must be set to a value greater than or equal to the number of objects found in NVM3. This includes the number of NVM3 objects created through the native NVM3 API and any objects created through higher level APIs such as the Token API. The cache must also be large enough to hold any deleted NVM3 objects. The `nvm3_countObjects()` and `nvm3_countDeletedObjects()` functions can be used to find the number of live and deleted objects in NVM3 at any given point. Silicon Labs recommends checking these functions after initialization of all NVM3 objects, both through the native NVM3 API and higher level APIs such as the token API, to figure out the correct size of the NVM3 default cache size.

## NVM3 Default Instance Key Space

NVM3 uses a 20-bit key to identify each object. To avoid using the same key for more than one object, the NVM3 key space for the default NVM3 instance has been divided into several domains as outlined in the following table. For example, NVM3 objects defined in the Zigbee EmberZNet stack should use NVM3 keys in the range 0x10000 to 0x1FFFF, while user application tokens should use keys below 0x10000. Note that any user defined NVM3 objects should be placed below 0x10000.

| **Domain** | **NVM3 Key** |
|------------|--------------|
| User | 0x00000 - 0x0FFFF |
| Zigbee EmberZNet stack | 0x10000 - 0x1FFFF |
| OpenThread stack | 0x20000 - 0x2FFFF |
| Connect stack | 0x30000 - 0x3FFFF |
| Bluetooth stack | 0x40000 - 0x4FFFF |
| Z-Wave stack | 0x50000 - 0x5FFFF |
| Bluetooth mesh stack | 0x60000 - 0x6FFFF |
| Reserved | 0x70000 - 0x7FFFF |
| Apple HomeKit | 0x80000 - 0x80FFF |
| Zigbee Cluster Library (ZCL) | 0x81000 - 0x81FFF |
| dotdot | 0x82000 - 0x82FFF |
| Platform CLI SM | 0x83000 - 0x830FF |
| Platform Crypto | 0x83100 - 0x870FF |
| Bootloader | 0x87100 - 0x871FF |
| Matter | 0x87200 - 0x87FFF |
| AWS Late Provision | 0x88000 - 0x88FFF |
| KNX IoT | 0x89000 - 0x897FF |
| Antenna Calibration (Platform) | 0x89800 - 0x8987F |
| Antenna Calibration (RAIL) | 0x89880 - 0x898FF |
| Trackers | 0x89900 - 0x899FF |
| Reserved | 0x89A00 - 0x8DFFF |
| Static device tokens - override | 0x8E000 - 0x8EFFF |
| Static secure tokens - override | 0x8F000 - 0x8FFFF |
| Wi-SUN | 0x90000 - 0x9FFFF |
| Sidewalk | 0xA0000 - 0xAFFFF |
| Wi-Fi | 0xB0000 - 0xBFFFF |
| Reserved | 0xC0000 - 0xFFFFF |
