# Polymorphic GATT

## Introduction

Originally, Silicon Labs' Bluetooth stack implemented a static GATT database structure, which means that services and characteristics are created at compile time, not at run time. As a result, software could not change the database structure dynamically. To overcome this issue, Bluetooth SDK v2.4 introduced a new feature called *Polymorphic GATT*, which can be used to dynamically show or hide GATT services and characteristics. This new feature allows users to create a ‘superset’ GATT database with pre-defined hidden/visible services and characteristics and alter their visibility on the fly.

Silicon Labs Bluetooth SDK v3.2 introduced the ability to create the GATT database dynamically with Bluetooth APIs. These dynamically created GATT attributes can coexist with a static database generated from a GATT XML file. In this case, the attribute table of the static database is placed at the beginning of the database. When new services and characteristics are created dynamically, they are added into the attribute table after the attributes of the static database.

This feature is recommended for NCP projects. With this method, the target application where the GATT database is located does not need to be modified. Therefore, the database can be built from the host side with the APIs. In other cases, the GATT Configurator is the preferred solution. For more details, see the *Dynamic GATT Configuration* section of [GATT Configurator User’s Guide for Bluetooth LE and Bluetooth Mesh](/bluetooth/{build-docspace-version}/gatt-configurator-users-guide-ble-btmesh/03-dynamic-gatt-configuration).

> **Note**: Changing the visibility of services/characteristics should not be done during a connection because that can cause incorrect behavior if [Service Change Indication](#service-change-indications) is not enabled. The safest method is to change the visibilities when no devices are connected, or to change the visibilities after making sure that [Service Change Indication](#service-change-indications) was enabled on the connection.

## How it Works

The visibility of services and characteristics is controlled through the use of GATT *capabilities.* The usage of capabilities and their syntax in the GATT XML file is fully described in [Blue Gecko Bluetooth Profile Toolkit Developer's Guide](/bluetooth/{build-docspace-version}/bluetooth-profile-toolkit-developers-guide). Read through this guide to get a good grasp of the visibility and inheritance rules.

To summarize, each service/characteristic can declare a number of capabilities and the state of the capabilities (enable/disable) determines the visibility of those services/characteristics as a bit-wise OR operation e.g., the service/characteristic is visible when at least one of its capabilities is enabled and it is not visible when all of its capabilities are disabled.

> **Note**: If certain services/characteristics are meant to be always visible, one good approach is to have one capability that is declared by those services/characteristics which is enabled by default and untouched by the application code.

## Setting up Capabilities with GATT Configurator

Bluetooth SDK's GATT Configurator supports the polymorphic GATT database and it allows declaring *capabilities* for the whole GATT database as well as subsets for each of the services and characteristics.

Always start by declaring the GATT-level capabilities and define their default value by selecting "Custom BLE GATT" and adding the capabilities in "Capability declaration". To add a capability, press the '+' on the right-hand side and then change the capability name and default value.

![Declaring capabilities](resources/vge-gatt-level.png?darkModeUrl=resources/vge-gatt-level.png)

After those capabilities are added, they become available on each of the services and characteristics. They can be added through the drop-down list but this time you'll be shown the list of capabilities declared at the GATT-level where to pick from.

![Applying Capabilities on Services/Characteristics](resources/vge-ota.png?darkModeUrl=resources/vge-ota.png)

## Enabling/Disabling Capabilities

*Capabilities* can be enabled/disabled with the API command
```c
sl_status_t sl_bt_gatt_server_set_capabilities(uint32_t caps,
                                               uint32_t reserved);
```
where *caps* is the bit flags of each capability which should be set to 1 if the capability is to be enabled or 0 if it's to be disabled.

The auto-generated *gatt_db.h* contains the bit flag value for each of the *capabilities* you defined in the GATT Configurator, e.g.,:

```c
typedef enum {
  ota                            = 0x0001,
  temp_measure                   = 0x0002,
  temp_type                      = 0x0004,
  gattdb_all_caps = 0x0007
} gattdb_cap_t;
```

To enable *ota* and *temp_type* and disable all other capabilities, use the command call that looks like this:

```c
sl_bt_gatt_server_set_capabilities (ota | temp_type, 0);
```

## Service Change Indications

The stack monitors the local database change status and manages the service change indications for a GATT client that has enabled the indication configuration of the Service Changed characteristic. The Service Changed characteristic is part of the [Generic Attribute Service](https://www.bluetooth.com/specifications/specs/), which can be added to the GATT by checking **Generic Attribute Service** in the GATT Configurator (it can be found after selecting **Custom BLE GATT** in the GATT database). For more information, see [Service Change Indication](./service-change-indication).

## Dynamic GATT

*Dynamic GATT Database* feature is a newer way to accomplish the same flexibility and more as with the *Polymorphic GATT* feature. It is not included in projects by default, so it needs to be installed from the Software Components tab.

![Dynamic GATT Database component](resources/vge-dynamic-gatt.png?darkModeUrl=resources/vge-dynamic-gatt.png)

The Polymorphic GATT Capabilities are only usable with Services/Characteristics defined in the Static GATT, with Dynamic GATT, the provided APIs such as
```c
sl_status_t sl_bt_gattdb_start_service(uint16_t session, uint16_t service);
```
or
```c
sl_status_t sl_bt_gattdb_stop_service(uint16_t session, uint16_t service);
```
to be used to achieve the same results.
