# Project Structure

This section explains the application project structure and the mandatory and optional resources that must be included in the project.

## Bluetooth Files

**Library Files**

The Bluetooth stack is provided in multiple libraries. The Silicon Labs Configurator (SLC) tool adds the necessary libraries to the application project automatically.

**RAIL**

The Bluetooth stack uses RAIL to access the radio and RAIL libraries needs to be linked with Bluetooth stack. RAIL has separate libraries for each device family and for single- and multi-protocol environments. RAIL libraries are provided in the Simplicity SDK Suite. For more information refer to [RAIL Fundamentals](/bluetooth/{build-docspace-version}/rail-fundamentals) and other RAIL documentation.

>**Note**: To ensure regulatory compliance of the radio module, the Bluetooth stack for the radio module needs to be linked together with the RAIL library and the configuration library for the radio module. These are librail_module_\<soc family>\<compiler>_release.a and librail_config\<modulename>.a.

**EMLIB** and **EMDRV**

The Bluetooth stack uses EMLIB and EMDRV libraries to access EFR32 hardware. EMLIB and EMDRV peripheral libraries are provided in source code and they must be included in the project. EMLIB and EMDRV are part of the Simplicity SDK Suite. For more details on EMLIB and EMDRV, see platform EMDRV documentation and EMLIB documentation on [https://docs.silabs.com/](https://docs.silabs.com/).

**mbed TLS**

The Bluetooth stack uses the Mbedtls library for cryptographic operations. The Mbedtls library is provided in source code and must be included in the project. Mbedtls is part of the Simplicity SDK Suite. For more details, refer to the [Mbedtls documentation](https://mbed-tls.readthedocs.io/en/latest/).

**Sleep Timer**

Sleep Timer is a platform component providing software timers, timekeeping, and date functionality. The Bluetooth stack uses it for internal event scheduling, and it must be included in the project. See [platform sleeptimer documentation](https://docs.silabs.com/mcu/latest/efr32mg13/group-SLEEPTIMER).

Note that Sleep Timer callbacks are called from the interrupt context. BGAPI functions cannot be called from the callback. Instead, the application should implement the timer task handling in the application main loop. Several simple timer components in the SDK, for example, `simple timer` and `simple timer micriumos`, implement helper functionality that also allows calling BGAPI command from its callback.

**Power Manager**

Power Manager is a platform component that manages the system's energy modes. Its main purpose is to transition the system to a low energy mode when the processor has nothing to execute. See the reference for your MCU on [https://docs.silabs.com/](https://docs.silabs.com/) under Modules \> Platform Services \> Power Manager.

**Header Files**

**sl_bt_version.h**

This file contains the Bluetooth stack version.

**API Header Files**

These files define the Bluetooth stack API.

These files serve two purposes:

1. They contain the actual Bluetooth stack API and the commands and events for the stack.

2. They provide a configuration and event management API to the Bluetooth stack.

**sl_bt_types.h**

**sl_bt_stack_init.h**

**sl_bt_api.h**

**sl_bgapi.h**

These files contain the Bluetooth stack API and the commands and events for the stack, and a configuration API for the Bluetooth stack.

**sl_bt_ncp_host_api.c**, **sl_bt_ncp_host.c**, **sl_bt_ncp_host.h**, and **sl_bt_internal.h**

These files are used when developing applications for an external host. They provide the API definitions and adaptation layer between the host application and the BGTAPI serial protocol.

## GATT Database

The GATT (Generic Attribute Profile) database is a standardized way of describing the Bluetooth profiles, services, and characteristics of a Bluetooth device. The Silicon Labs’ Bluetooth SDK provides two ways to define the GATT database:

- A static GATT database can be defined in compile time with the appropriate tools provided by the Bluetooth SDK, or can be written in XML and passed to the BGBuild tool as a pre-build task. In this case, the database structure is stored in the ROM, which means faster start-up time and lower memory usage.

- A dynamic GATT database can be defined in runtime with the appropriate BGAPI commands using the `bluetooth_feature_dynamic_gattdb` component. In this case, the database structure is stored in the RAM, which makes it more flexible. This is recommended in the NCP use case to avoid re-building the target code that runs on the Wireless Gecko.

You can also combine these two methods. For more information on how to create GATT databases and the syntax, refer to [Blue Gecko Bluetooth Smart Profile Toolkit Developer's Guide](/bluetooth/{build-docspace-version}/bluetooth-profile-toolkit-developers-guide).

**gatt_db.c** and **gatt_db.h**

The `gatt_db.c` file defines the GATT database structure and content, and is auto-generated by BGBuild or by the Visual GATT Editor. The `gatt_db.h` file includes this database and the handles of local characteristics and services. Type definitions of GATT are automatically included from *bg_gatt_db_def.h* to *gatt_db.h*.

When a dynamic GATT database is used, the handles of a local characteristic or service may change if the attribute table is updated at run time. Therefore, it is not recommended to use the constants in `gatt_db.h` unless absolutely certain that the handle values of characteristics and services are not affected by the dynamic GATT database update.

## Device Firmware Upgrade

Device Firmware Upgrade (DFU) is the process of upgrading the application either over a serial link or over-the-air (OTA). In both cases the application needs to add the following file to enable the support for DFU.

**application_properties.c**

This file includes the application properties struct that contains information about the application image, such as type, version, and security. The struct is defined in `application_properties.h` in the Gecko Bootloader API. A pre-generated file is included in Simplicity Studio projects, which can be modified to include application-specific properties. The application properties can be accessed using the Gecko Bootloader API. The following members can be updated by changing the defines:

```C
// Version number for this application (uint32_t)
#define APP_PROPERTIES_VERSION
// Unique ID (e.g. UUID or GUID) for the product this application is built for (uint8_t[16])
#define APP_PROPERTIES_ID
```

When using the OTA process with Bluetooth AppLoader, a pointer to the application properties struct needs to be set to application vector table vector 13. This is enabled automatically when using the default startup file and the struct name is `sl_app_properties`.

## NCP Applications

When developing applications for an external host, the `SL_BT_API_FULL` define needs to be defined to prevent the linker from dropping the BGAPI command implementation from the application. The define includes a full implementation of all enabled BGAPI classes in the application.

## RTOS Support

The Bluetooth stack is usually run on bare metal configuration, but the Bluetooth stack can also run on Micrium RTOS and FreeRTOS. When the application project includes an RTOS kernel, the Bluetooth RTOS adaptation component with the following files is automatically added to the application:

**sl_bt_rtos_adaptation.c**

**sl_bt_rtos_adaptation.h**

**sl_bt_rtos_config.h**

**sl_bt_rtos_adaptation.c and sl_bt_rtos_adaptation.h**

`sl_bt_rtos_adaptation.c` and `sl_bt_rtos_adaptation.h` provide the RTOS tasks for the IPC (Inter-Process Communication) with the Bluetooth stack and other RTOS tasks using CMSIS-RTOS2.

**sl_bt_rtos_config.h**

`sl_bt_rtos_config.h` is used to set the Bluetooth RTOS task priorities and the stack sizes.

When the project is generated, the SLC tools will automatically contribute the relevant Bluetooth stack configuration and initialization to enable the Bluetooth stack to work with the RTOS.

## Multiprotocol Support

When the Bluetooth Stack is used in a multiprotocol environment, the application must use the RAIL library with multiprotocol support. When the application uses multiprotocol RAIL library, the SLC tools automatically include relevant initialization for Bluetooth multiprotocol support.

## Platform Components

The Bluetooth SDK relies on many platform components that are part of the underlying Gecko Platform infrastructure of the Simplicity SDK Suite. The `autogen` folder contains the source code generated by SLC tools for initializing the hardware and processing events. The `config` folder includes hardware and stack configuration options.
