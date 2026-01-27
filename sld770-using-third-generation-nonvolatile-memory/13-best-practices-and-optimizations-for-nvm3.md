# Best Practices and Optimizations for NVM3

## Right Sizing the NVM3 Storage Region in Flash

To minimize repacks and reduce flash wear, it is important to configure the NVM3 region size appropriately. The size of the NVM3 region should be based on:

- The number of objects stored in NVM3.
- The size of each object, including metadata overhead.
- The frequency of object updates and deletions.
- The flash page size of the device.

With the addition of security on Series 3 devices, storing a data object incurs a size overhead of 8 bytes. Ensure that the NVM3 region has sufficient space to accommodate active objects, deleted objects, and spare pages for repack operations. A larger NVM3 region reduces the frequency of repacks, thereby extending the flash lifetime. The "Maximum allowed basic storage" section at [Storage Capacity](https://docs.silabs.com/gecko-platform/latest/platform-driver/nvm3#storage-capacity) can help with configuring the appropriate number of flash pages and right-sizing the NVM3 storage in flash.

## Right Sizing the Cache

The NVM3 cache size should be configured to match the number of live and deleted objects in the NVM3 region. This ensures faster access to objects and reduces the overhead of searching for objects in flash. Users can query the cache size and memory information using the `nvm3_getMemInfo()` API. This API provides details about the current memory state, including available memory, cache status, and additional cache requirements. Properly sizing the cache improves performance and reduces object lookup latency.

## Low Memory Notification Callback

Users can use the `nvm3_registerCallback()` API to register a callback function, which is triggered when the NVM3 instance detects low memory conditions or a cache overflow. The callback allows the application to handle conditions such as freeing up memory by triggering repack. It is recommended that the callback be registered before calling any NVM3 APIs. Additionally, users can deregister the callback using the `nvm3_deregisterCallback()` API.

## NVM3 Optimizations

Enabling NVM3 optimization improves the NVM3 initialization and object lookup time. NVM3 optimization can be enabled or disabled from Simplicity Studio UC for Series-2 and Series-3 devices. The optimization support has implications on code size. With optimization support enabled, the code size increases by approximately 1248 bytes. NVM3 optimization is enabled by default for Series-3 devices and disabled for Series-2 devices.

By following these best practices and leveraging the available APIs and optimizations, users can maximize the efficiency and reliability of the NVM3 storage system while maintaining high performance for their applications.
