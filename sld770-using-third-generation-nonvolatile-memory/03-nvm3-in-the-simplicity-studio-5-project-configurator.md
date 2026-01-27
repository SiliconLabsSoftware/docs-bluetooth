# NVM3 in the Simplicity Studio 5 Project Configurator

The Simplicity Studio 5 Project Configurator, used with Silicon Labs Bluetooth v3.x, Connect v3.x, EmberZNet Zigbee v7.x, and OpenThread SDK applications, includes an NVM3 Core component. A separate component is also provided for the NVM3 default instance, which will initialize this instance.

![screenshot](resources/sld770-image1.png)

For Si91x devices, the NVM3 default instance can be configured under NVM3 for Si91x.

![screenshot](resources/sld770-image4.png)

The **NVM3 Default Instance** component provides the following configurations:

- **Cache Size**: Number of objects to cache. To reduce access times, this number should be equal to or higher than the number of live and deleted objects stored in NVM3 at any time.

- **Max Object Size**: Size of largest allowed NVM3 object in bytes. Must be between 208 and 4096 bytes.

- **User Repack Headroom**: Headroom determining how many bytes below the forced repack limit the user repack limit is placed. The default value is 0, which means that the forced and the user repack limits are the same.

- **Default Instance Size**: Size of the NVM3 storage region in flash. This must be set to match an integer number of flash pages, 3 pages at minimum.

If the **NVM3 Default Instance** component is included in a project, the default instance will be initialized automatically during `sl\_system\_init()` if the **System Init** component is included in the project.
