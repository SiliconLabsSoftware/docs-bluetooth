# Repack Management in NVM3

## What is a Repack Operation?

A **repack** operation in NVM3 reclaims memory in non-volatile memory (NVM) by copying valid objects to new pages and erasing old pages with obsolete or deleted data. This operation is essential for maintaining available storage and ensuring the NVM3 instance can continue to store new or updated objects.

### Why is Repack Needed?

As the NVM3 fills up with new and updated objects, old versions and deleted objects accumulate, consuming memory. When there is insufficient free memory to store additional objects, a repack operation is required to release this memory by removing outdated data.

## When is Repack Triggered?

Repack can be triggered in two ways:

- **Forced repack**
  NVM3 automatically initiates a repack during a write operation if free memory falls below a critical threshold (forced threshold). This threshold represents the minimum limit and cannot be modified by the user. Forced repacks are performed internally using the `repackUntilGood` API. When a forced repack occurs, write operations may take longer, potentially impacting application timing.

- **User repack**
  Applications can manually trigger a repack by calling `nvm3_repack()`. The function will execute the repack only if the available free memory drops below the user threshold. This threshold is configurable through the `repackHeadroom` parameter and is evaluated by `nvm3_repackNeeded()`. The function `nvm3_repackNeeded()` can be used to check whether a repack is required, and `nvm3_repack()` will proceed only if this function returns `true`. For time-sensitive applications, it is recommended to trigger a user repack during periods of low activity.

## The Role of Repack Headroom

The **repack headroom** parameter (`repackHeadroom` in `nvm3_Init_t`) specifies the gap between the user-defined repack threshold and the forced repack threshold. This allows applications to reserve additional memory before a forced repack occurs, giving more control over when repacks happen.

- **Default Value**
  By default, `repackHeadroom` is configured to 0, making the user threshold equal to the forced threshold plus the maximum configured object size.

- **Increasing Headroom**
  Configuring a higher `repackHeadroom` value means user or manually triggered repacks will occur earlier, reserving more memory and reducing the risk of forced repacks during critical operations. However, this can increase the frequency of user repacks and thus flash wear.

## Best Practices for Repack Configuration

- **Call `nvm3_repack()` proactively**
  Before writing large objects or performing a batch of writes, check `nvm3_repackNeeded()` and call `nvm3_repack()` if needed. This avoids forced repacks during time-critical operations.

- **Tune `repackHeadroom`**
  Configure `repackHeadroom` according to the application's tolerance for repack latency (the amount of delay it can accept when a repack occurs during a write) and the available NVM3 memory. A smaller value allows more objects to be stored before a repack is needed but increases the risk of a forced repack during a write, adding latency. A larger value causes repacks to occur earlier, which lowers the risk of a forced repack but results in more frequent user repacks.

  Repack headroom can be configured using `NVM3_DEFAULT_REPACK_HEADROOM` macro or through the Studio UI to reserve a specific amount of memory in advance, helping to avoid forced repacks during time-critical operations.

  For example, if the application writes 512 bytes of data (including object overhead) during bootup, which is a time-sensitive operation, configure the repack headroom to 512 bytes to ensure that a forced repack does not occur during boot. Proactively check for repack before the next bootup.

  ```c
  // Example: Configure repack headroom to 512 bytes to ensure sufficient memory for a critical write
  #define NVM3_DEFAULT_REPACK_HEADROOM  512

  // Proactively check and perform a repack if needed
  if (nvm3_repackNeeded(nvm3_defaultHandle)) {
    nvm3_repack(nvm3_defaultHandle);
  }
  ```

- **Monitor Available Memory:**
  Use `nvm3_getMemInfo()` to monitor available memory and repack proactively if memory is running low.

  ```c
  // Example: Monitor available memory and trigger a repack
  nvm3_MemInfo_t memInfo;
  nvm3_getMemInfo(nvm3_defaultHandle, &memInfo);

  if (memInfo.nvm3Available < 512) {  // Threshold: 512 bytes
    nvm3_repack(nvm3_defaultHandle);
  }
  ```

  For more information, see the [NVM3 Memory Info](https://docs.silabs.com/gecko-platform/latest/platform-driver/nvm3-mem-info-t).

## Avoid Large Max Object Size Unless Needed

The `maxObjectSize` parameter has a significant impact on both the minimum required NVM3 size and the amount of usable storage before a repack is triggered. Basic storage is defined as the total size of all objects, including any overhead stored with the data. The maximum amount of data that can be stored in NVM3 depends on the number of flash pages allocated for storage and the configured maximum object size.

The maximum allowed basic storage for an NVM3 instance can be estimated using the following formula:

```c
allowed_basic_storage =
  total_nvm_size
  - ((total_nvm_size / page_size) * page_header_size)             // per-page bookkeeping
  - 2 * (page_size - page_header_size)                            // repack window
  - (max_object_size + obj_header_size_large)
  - ((page_size == 4096 && max_object_size > 4064) ? 212 : 0);    // extra buffer for 4KB pages
```

where:

- `total_nvm_size` is the total NVM3 region size in bytes
- `page_size` is the flash page size in bytes
- `page_header_size` is the page header size in bytes (20 bytes)
- `max_object_size` is the configured maximum object size in bytes
- `object_header_size_large` is the large object header size in bytes (8 bytes)

This is a theoretical limit. If the basic storage is at this limit, no memory is left for wear-levelling, and page erases will be forced for every object written. Therefore, the NVM3 instance should be configured with enough flash pages so that the maximum allowed basic storage is well above the actual basic storage required by the application.

For detailed tables showing the relationship between flash page count, page size, and max object size, see the [Storage Capacity](https://docs.silabs.com/gecko-platform/latest/platform-driver/nvm3#storage-capacity) section in the NVM3 API documentation.

> **Best Practice**: Configure `maxObjectSize` to the smallest value that still accommodates the largest object. Avoid configuring it unnecessarily high, as this reduces usable memory and increases the frequency of repacks, which can accelerate flash wear.
>
> **Tip**: If an application only occasionally needs to store a few large objects (e.g., 4096 bytes), these objects can be broken into smaller chunks (such as four 1024-byte objects). Each chunk is written separately and later read back and reassembled into the original object when needed. This approach keeps `maxObjectSize` small, maximizing usable memory and minimizing repack frequency and flash wear for the majority of the data.

## Impact of Max Object Size on Repack and Usable Storage

The maximum object size (`maxObjectSize`) directly affects the repack thresholds and the amount of usable storage before a repack is needed. A larger `maxObjectSize`:

- Increases the minimum NVM3 size required for operation.
- Reduces the amount of storage available for user data before a repack is triggered.
- May result in more frequent repacks, especially if the stored objects are much smaller than the configured maximum.
