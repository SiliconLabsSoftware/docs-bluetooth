# Using NVM3 in AppBuilder-Based Applications

This section explains how NVM3 can be used for non-volatile storage in AppBuilder-based applications like EmberZNet 6.10.x and lower, EmberZNet-based DMP applications in GSDK 3.2 and lower, and Connect applications in Gecko SDK Suite 2.7 and lower.

## NVM Library Plugin

To use NVM3 with an AppBuilder-based example application, the NVM3 Library plugin should be included in the project. All PS Store and SimEE plugins should be deselected.

![NVM3 Library Plugin in AppBuilder](resources/sld770-image2.png)

The **NVM3 Library** plugin offers four plugin options:

- **Flash Pages**: Number of flash pages to use for NVM3 data storage. Must be 3 or higher. Default is 18 (36KB) for EFR32 Series-1 devices and 4 (32KB) for EFR32 Series-2 devices

- **Cache Size**: Number of objects to cache. To reduce access times, this number should be equal to or higher than the number of objects, including tokens and deleted objects, stored in NVM3 at any time.

- **Max Object Size**: Size of largest allowed NVM3 object in bytes. Must be between 208 and 4096 bytes. Note that the token API can only be used to access objects of 254 bytes or smaller. When accessing larger objects, the native NVM3 API must be used.

- **User Repack Headroom**: Headroom determining how many bytes below the forced repack limit the user repack limit is placed. The default value is 0, which means that the forced and the user repack limits are the same.

When the NVM3 Library plugin is used the **Simulated EEPROM version 2 to NVM3 Upgrade Library** or **Simulated EEPROM version 2 to NVM3 Upgrade Stub Library** must be included, as described in [SimEEv2 to NVM3 Upgrade Plugin](#simeev2-to-nvm3-upgrade-plugin).

## SimEEv2 to NVM3 Upgrade Plugin

An AppBuilder plugin (**Simulated EEPROM version 2 to NVM3 Upgrade Library**) is provided for EmberZNet applications that upgrade tokens stored in SimEEv2 to NVM3. For tokens to be successfully upgraded to NVM3, CREATOR_\* and NVM3KEY_\* defines must be added for all tokens as described in [Token API](./10-nvm3-api-options.md). The upgrade plugin will replace the SimEEv2 storage in-place with an NVM3 storage instance. The plugin does this by compacting the SimEEv2 storage down to 12 kB, and then creates an NVM3 instance in the remaining 24 kB of the original 36 kB SimEEv2 storage space. After the token data has been copied over from SimEEv2 to NVM3, the SimEEv2 storage is erased and the NVM3 instance is resized to use the entire 36 kB storage space. Apart from the code space needed for the upgrade library code, the upgrade does not require any additional flash space to the 36 kB storage area. The upgrade plugin requires that the existing SimEEv2 storage space and new NVM3 storage space are located at the same address and have the same size.

The **Simulated EEPROM version 2 to NVM3 Upgrade Library** plugin should be included to enable the upgrade as shown in the figure below. If no SimEEv2 token data is found, the upgrade plugin will look for NVM3 data, and if neither is found it will create a new NVM3 instance with tokens set to their default values. For applications that do not need to upgrade any SimEEv2 tokens, the **Simulated EEPROM version 2 to NVM3 Upgrade Stub** plugin should be included instead.

![SimEEv2 to NVM3 Upgrade Library and Stub Plugins in AppBuilder](resources/sld770-image3.png)
