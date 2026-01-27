# Using NVM3 with Silicon Labs Bluetooth Applications

Traditionally the Bluetooth stack uses its own proprietary solution to store data in non-volatile memory, called **Persistent Store (PS Store)**. PS Store stores both data handled by the stack (such as temporary Bluetooth address, bonding keys, and so on) and user data (such as the device state) that has to be preserved on resetting the device. To learn more about PS Store, read the related section of the [Bluetooth API Reference Guide](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/).

Some of the sample applications in the Bluetooth SDK are still configured to use PS Store, while others are already configured to use NVM3. This means that:

- On Series 1 devices sample apps are configured to use PS Store, except Bluetooth Mesh NCP sample projects, where NVM3 is used by default.

- On Series 2 devices all sample apps are configured to use NVM3.

While Series 2 devices support NVM3 only, on Series 1 devices both PS Store and NVM3 can be used. This section describes how to configure NVM3 in the Bluetooth SDK, and how to switch between PS Store and NVM3 if needed.

## Configuring NVM3 in the Bluetooth SDK

**With Simplicity Studio 5 and Bluetooth SDK v3.x**

NVM3 can be configured with the Project Configurator as described in [NVM3 in the Simplicity Studio 5 Project Configurator](./03-nvm3-in-the-simplicity-studio-5-project-configurator.md).

**With Simplicity Studio 4 and Bluetooth SDK v2.x**

The Project Configurator is not available to configure NVM3 parameters. Therefore, the parameters have to be defined manually. To overwrite the default parameters:

1. Open the project settings.

2. Find the defined symbols:

   - If you use GCC as your compiler, go to C/C++ Build\>Settings\>GNU ARM C Compiler\>Symbols\>Defined Symbols

   - If you use IAR as your compiler, go to C/C++ Build\>Settings\>IAR C/C++ compiler for ARM\>Preprocessor\>Defined Symbols

3. Add any of the following defines to overwrite the default parameters:

   - NVM3_DEFAULT_NVM_SIZE

   - NVM3_DEFAULT_CACHE_SIZE

   - NVM3_DEFAULT_MAX_OBJECT_SIZE

   - NVM3_DEFAULT_REPACK_HEADROOM

You can find the description of each parameter in [NVM3 in the Simplicity Studio 5 Project Configurator](./03-nvm3-in-the-simplicity-studio-5-project-configurator.md). Be careful when providing the size of NVM3, as it must be a multiple of the flash page size. Note: On Series 1 devices the flash page size is typically 2 kB, while on Series 2 devices the flash page size is typically 8 kB. Check the Wireless Gecko Reference Manual for your device. The NVM size must be at least 3 flash pages.

## Switching from PS Store to NVM3

Beginning with Bluetooth SDK v2.13.0, both PS Store and NVM3 are supported as non-volatile memory solutions on Series 1 devices. Most sample applications are configured to use PS Store by default, but for some applications (where larger non-volatile memory is needed) NVM3 may be a better solution.

> **Note**: PS Store and NVM3 are not compatible with each other, therefore upgrading an already existing application from PS Store to NVM3 will result in losing all data stored on the device. If you have an application running in the field, it may be wiser to stay with PS Store. If you still want to upgrade, see [Using the Gecko Bootloader with the Silicon Labs Bluetooth Application](https://docs.silabs.com/bluetooth/latest/using-gecko-bootloader-with-bluetooth-apps/) for details.

> **Note**: PS Store uses only 2 flash pages (=4 kB on an EFR32BG1/12/13 device). Therefore, changing to NVM3 will affect the available space in flash. You must be particularly careful when you upgrade the firmware not to overwrite the NVM3 area with the application.

**With Simplicity Studio 5 and Bluetooth SDK v3.x**

To change your project configuration from PS Store to NVM3, simply install the **NVM3 Default Instance** component in the Project Configurator as discussed in [NVM3 in the Simplicity Studio 5 Project Configurator](./03-nvm3-in-the-simplicity-studio-5-project-configurator.md). This will automatically uninstall the (otherwise hidden) PS Store component.

**With Simplicity Studio 4 and Bluetooth SDK v2.x**

To change your project configuration from PS Store to NVM3, use the following procedure.

1. Copy the following folder with all of its content:

   `C:\SiliconLabs\SimplicityStudio\v4\developer\sdks\gecko_sdk_suite\<version>\platform\emdrv\nvm3` under the `/platform/emdrv` folder of your project.

2. Remove the following files from the project:

   - `/platform/emdrv/nvm3/src/nvm3_hal_extflash.c`

   - `/platform/emdrv/nvm3/src/nvm3_default_extflash.c` (NVM3 use with external flash is deprecated)

3. If you use Apploader in your project, also copy the NVM3 version of Apploader from:

   `C:\SiliconLabs\SimplicityStudio\v4\developer\sdks\gecko_sdk_suite\<version>\protocol\bluetooth\lib\<device>\<compiler>\binapploader_nvm3.o`

   into the `/protocol/bluetooth/lib/<device>/<compiler>` folder of your project.

4. **If you use GCC as a compiler:**

   1. Go to `Project > Properties > C/C++ Build > Settings > GNU ARM C Compiler > Includes`.

   2. Add `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/inc}` to the include directory.

   3. Go to `Project > Properties > C/C++ Build > Settings > GNU ARM C Linker > Miscellaneous`.

   4. Remove `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/<device>/<compiler>/libpsstore.a}`.

   5. Add `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/lib/libnvm3_CM4_gcc.a}`.

   **If you use IAR as a compiler:**

   1. Go to `Project > Properties > C/C++ Build > Settings > IAR C/C++ Compiler for ARM > Preprocessor`.

   2. Add `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/inc}` to the include directory.

   3. Go to `Project > Properties > C/C++ Build > Settings > IAR Linker for ARM > Library`.

   4. Remove `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/\<device\>/\<compiler\>/libpsstore.a}`.

   5. Add `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/lib/libnvm3_CM4_iar.a}`.

5. If you use Apploader, also modify:

   `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/\<device\>/\<compiler\>/binapploader.o}`

   to

   `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/\<device\>/\<compiler\>/binapploader_nvm3.o}`

6. Configure NVM3 as described in [Configuring NVM3 in the Bluetooth SDK](#configuring-nvm3-in-the-bluetooth-sdk).

## Switching from NVM3 to PS Store

It may happen that, for a reason such as backward compatibility, you have to change the configuration from NVM3 to PS Store.

**With Simplicity Studio 5 and Bluetooth SDK v3.x**

To change your project configuration from NVM3 to PS Store, simply uninstall the NVM3 Default Instance component. This will automatically install the (otherwise hidden) PS Store component. Note, that this can only be done on series 1 devices, as series 2 devices do not support PS Store. **Also, it is not possible with Bluetooth Mesh Stack in use**.

**With Simplicity Studio 4 and Bluetooth SDK v2.x**

To change your project configuration from NVM3 to PS Store, use the following procedure:

1. Remove the /platform/emdrv/nvm3 folder from your project.

2. Copy the PS Store library from:

   `C:\SiliconLabs\SimplicityStudio\v4\developer\sdks\gecko_sdk_suite\<version>\protocol\bluetooth\lib\<device>\<compiler>\libpsstore.a`

   into the `/protocol/bluetooth/lib/<device>/<compiler>` folder of your project.

3. If you use Apploader in your project, also copy the PS Store version of Apploader from:

   `C:\SiliconLabs\SimplicityStudio\v4\developer\sdks\gecko_sdk_suite\<version>\protocol\bluetooth\lib\<device>\<compiler>\binapploader.o`

   into the `/protocol/bluetooth/lib/<device>/<compiler>` folder of your project.

   > **Note**: PS Store is not supported on Series 2 devices (EFR32xG2x), therefore there is only an NVM3 version of the Apploader for these devices.

4. **If you use GCC as your compiler**

   1. Go to `Project > Properties > C/C++ Build > Settings > GNU ARM C Compiler > Includes`.

   2. Remove `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/inc}` from the include directory.

   3. Go to `Project > Properties > C/C++ Build > Settings > GNU ARM C Linker > Miscellaneous`.

   4. Add `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/<device>/<compiler>/libpsstore.a}`.

   5. Remove `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/lib/libnvm3_CM4_gcc.a}`.

   **If you use IAR as your compiler**

   1. Go to `Project > Properties > C/C++ Build > Settings > IAR C/C++ Compiler for ARM > Preprocessor`.

   2. Remove `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/inc}` from the include directories.

   3. Go to `Project > Properties > C/C++ Build > Settings > IAR Linker for ARM > Library`.

   4. Add `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/<device>/<compiler>/libpsstore.a}`.

   5. Remove `${workspace_loc:/${ProjName}/platform/emdrv/nvm3/lib/libnvm3_CM4_iar.a}`.

5. If you use Apploader, also change:

   `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/<device>/<compiler>/binapploader_nvm3.o}`

   to

   `${workspace_loc:/${ProjName}/protocol/bluetooth/lib/<device>/<compiler>/binapploader.o}`.
