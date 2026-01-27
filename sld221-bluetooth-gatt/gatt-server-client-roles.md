# GATT Server and Client Roles

## Introduction

Bluetooth Low Energy is a powerful and complex technology, which is different from the classic Bluetooth with a predefined set of official profiles to choose from. Although Bluetooth Low Energy does have predefined (a.k.a. "adopted") [profiles specified by the Bluetooth SIG](https://www.bluetooth.com/specifications/gatt), they are just the tip of the iceberg, a small subset of the functionality possible to achieve with BLE.

In many (or even most) cases, the best option is to create custom profile(s) for your application because it provides ultimate flexibility without an associated cost. In fact, it can be even easier than using one of the adopted profiles because you get to define exactly how everything works rather than conforming your application into something that is already defined. Also, because there is no official generic "serial port profile" in the BLE world, such as SPP in classic Bluetooth, sometimes a custom implementation is the only option to do what you need.

To have an effective custom implementation, it is important to understand how a BLE connection works, what roles are played by the devices involved, and how data is transferred from one device to the other over the air. Many terms are used, which are usually not interchangeable and mean different things: **central, peripheral, client, server, advertise, scan, read, write, notify, and indicate**. Understanding the terminology will make it easier to describe and build your BLE application.

## Quick Overview

This document covers the following:

* **Client** devices access remote resources over a BLE link using the GATT protocol. Usually, the central is the client (but not necessarily).
* **Server** devices have a local database and access control methods and provide resources to the remote client. Usually, the peripheral is the server (but not necessarily).
* You can use **read, write, notify**, or **indicate** operations to move data between the client and the server.
  * **Read** and **write** operations are requested by the client and the server responds (or acknowledges).
  * **Notify** and **indicate** operations are enabled by the client but initiated by the server, providing a way to push data to the client.
  * **Notifications** are unacknowledged, while **indications** are acknowledged. Notifications are therefore faster but less reliable.

To learn about central and peripheral roles, see [Central and Peripheral Roles](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-connections).

## GATT Server vs. GATT Client

An important concept in BLE design is the difference between a GATT server and a GATT client (where GATT means **G**eneric **ATT**ribute profile). These roles are not mutually exclusive, though typically your device will only be a server or a client. Role(s) that your device takes depend on its intended functionality. This is a basic summary of functionalities:

* **GATT client** - a device which accesses data on the remote GATT server via read, write, notify, or indicate operations
* **GATT server** - a device which stores data locally and provides data access methods to a remote GATT client

Unlike the central/peripheral distinction defined under [Connections](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-connections), it is easy to see that one device might be both at the same time, based on how your application defines the data structure and flow for each side of the connection. While it is most common for the peripheral device to be the GATT server and the central device to be the GATT client, this is not required. **The GATT functionality of a device is logically separate from the central/peripheral role**. The central/peripheral roles control how the BLE radio connection is managed while the client/server roles are dictated by the storage and flow of data.

Most of the example projects in the SDK archive and online implement peripheral devices designed to be GATT servers. These are easy to test with our **Simplicity Connect App** (available for [Android](https://play.google.com/store/apps/details?id=com.siliconlabs.bledemo&hl=en_US) and [iOS](https://apps.apple.com/us/app/silicon-labs-blue-gecko-wstk/id1030932759)). However, there are also a few examples which implement the central end of the connection, designed to be GATT clients.

## Receive vs. Transmit - Moving Data

In BLE projects built using the Bluetooth SDK, the GATT structure can be configured using the built-in tool from Simplicity Studio, called the Bluetooth GATT Configurator. This can be found under the Configuration tools in the .slcp file. After you modify the GATT Configuration, the gatt_db.c/.h and the **gatt.xml** files are generated.

The structure and flow of data is always defined on the GATT server. The client uses whatever is exposed by the server.

If using the IAR Embedded Workbench, see the [Profile Toolkit Developer Guide](/bluetooth/{build-docspace-version}/bluetooth-profile-toolkit-developers-guide).

## GATT Structure

A GATT database implements one or more profiles, which are made up of one or more services. Each service is made up of one or more characteristics. For example, in outline form:

* Profile 1
  * Service A
    * Characteristic a
    * Characteristic b
    * Characteristic c
  * Service B
    * Characteristic d
    * Characteristic e
* Profile 2
  * Service C
    * Characteristic f
    * Characteristic g
    * Characteristic h
  * Service D
    * Characteristic i
    * Characteristic j
* Profile 3
  * Service E
    * Characteristic k
    * Characteristic l
    * Characteristic m
  * Service F
    * Characteristic n
    * Characteristic o

You can implement as many profiles, services, and characteristics as you need. These may be entirely customized, in which case they would use your own 128-bit UUIDs generated by the GATT Configurator. On the other hand, you can create a project which implements adopted specifications by referencing the Bluetooth SIG's online definitions of [profiles](https://www.bluetooth.com/specifications/gatt), services, and characteristics. If you do this, it usually makes the most sense to start at the profile level and drill down from there because the full list of characteristics includes many things that will undoubtedly be irrelevant to your design. You can combine these two as well, using some adopted profiles/services and some of your own proprietary ones.

Every BLE device acting as a GATT server must implement the official **Generic Access service**. This includes two mandatory characteristics, which are **Device Name** and **Appearance**. These are similar to the **Friendly Name** and **Class of Device** values used by classic Bluetooth. This is an example definition of an absolutely minimal GATT definition, as shown in the GATT Configurator:

![Simplicity Studio GATT Configurator](resources/sstudio-bt-conf.png?darkModeUrl=resources/sstudio-bt-conf.png)

## Attributes and Characteristics

You may occasionally hear or see the terms "attribute" and "characteristic" used interchangeably, and while this isn't entirely wrong, it isn't totally accurate and can be confusing. Remember that a **service** is made up of one or more **characteristics**. However, one single **characteristic**, generally the most specific level down to which we define our GATT structure, may be comprised of many different **attributes**.

Each **attribute** is given a unique numerical handle, which the GATT client may use to reference, access, and modify it. Every characteristic has one main attribute, which allows access to the value stored in the database for that characteristic. When you read about "writing a value to this characteristic" or "reading that characteristic's value," the read/write operations are done to the main data attribute.

Other related attributes are read-only, such as a **Characteristic User Description** attribute, some control the behavior of the characteristic, such as the **Client Characteristic Configuration** attribute which is used to enable **notify** or **indicate** operations. The BLE stack and SDK tools generate these as necessary based on the settings configured in the GATT Configurator.

Every attribute has a UUID, which may be either 16 bits (e.g., "**180a**") or 128 bits (e.g., "**e7add780-b042-4876-aae1-112855353cc1**"). All 16-bit UUIDs are defined by the Bluetooth SIG and are known as **adopted** UUIDs. All 128-bit UUIDs are custom and may be used for any purpose without approval from the Bluetooth SIG. Two very common 16-bit UUIDs that you will see are **2901**, the Characteristic User Description attribute (defined in the User description field of the GATT Editor), and **2902**, the Client Characteristic Configuration attribute (created by our SDK when either "**notify**" or "**indicate**" is enabled on a characteristic).

Note that some attribute UUIDs do not technically need to be unique. Their **handles** are always unique, but the UUIDs occasionally overlap. For example, every Client Characteristic Configuration attribute has a UUID of **0x2902**, even though there may be a dozen of them in a single GATT database. You should **always give your own custom characteristics fully unique UUIDs** but don't be alarmed if you are testing out your prototype with Bluetooth NCP Commander, you perform a Descriptor Discovery operation, and suddenly see multiple instances of the same UUID for certain things. This is normal.

## Data Transfer Methods

**read, write, notify**, and **indicate** are the **four** basic operations for moving data in BLE. The BLE protocol specification allows maximum data payload of **247 bytes** for these operations. However, for read operations, the supported size is **249 bytes**. BLE is built for low-power consumption and for infrequent short-burst data transmissions. Sending large amounts of data is possible, but usually ends up being less efficient than classic Bluetooth when trying to achieve maximum throughput. The following are a few general guidelines about the types of data transfer you will need to use:

* If the client needs to send data to the server, use **write**.
* If the client needs to get data from the server on-demand (i.e., polling), use **read**.
* If the server needs to send data to the client without the client requesting it first, use **notify** or **indicate**. (The client must **subscribe** to these updates before any data will be transferred.)
* The server can't pull data from the client directly. If this is necessary, the server must use **notify** or **indicate** to send pre-arranged request data to the client, and then the client may follow up with a **write** operation to send back whatever data that the server needs.

The above four BLE data transfer operations are described here. Commands which you must send are shown separately from the events which the stack generates. Complete API reference can be found for example through Simplicity Studio Launcher on the Documentation tab.

### Read

Read operation is requested by the GATT client on a specific attribute exposed by the GATT server. The server then responds with the requested value. In the BLE stack, these API methods are typically involved in read operations:

* `sl_bt_gatt_read_characteristic_value` **command**
  * Reads a remote attribute's value with the given handle. A single sl_bt_evt_gatt_characteristic_value event is generated if the characteristic value fits in one ATT PDU. Otherwise, more than one sl_bt_evt_gatt_characteristic_value events are generated because the firmware will automatically use the "read long" GATT procedure. **This is the most common method used by a GATT client to read individual attribute values.**

* `sl_bt_gatt_read_characteristic_value_by_uuid` **command**
  * Reads the characteristic value of a service from a remote GATT database by giving the UUID of the characteristic and the handle of the service containing this characteristic. A sl_bt_evt_gatt_characteristic_value event is generated if the characteristic value fits in one ATT PDU. Otherwise, more than one sl_bt_evt_gatt_characteristic_value events are generated because the firmware will automatically use the "read long" GATT procedure.

* `sl_bt_gatt_read_multiple_characteristic_values` **command**
  * Can be used to read the values of multiple characteristics from a remote GATT database at once. sl_bt_evt_gatt_characteristic_value events are generated as the values are returned by the remote GATT server.

* `sl_bt_evt_gatt_server_user_read_request` **event**
  * Generated **on the GATT server** when a client requests reading data from **a characteristic whose "type" value is set to "user"**. Normally, this operation is handled by the stack automatically and the characteristic value stored in RAM right on the module is sent back to the client. However, with "user" characteristics, this data storage and retrieval is entirely up to you. The stack lets you know when it has been requested and you must get it (or generate it), then send it back (or not) in whatever format is desired. This can be useful for immediate I/O status reads, for example, so that instead of maintaining a value in memory or polling all the time so it is kept "fresh," you can instead just wait for the request and read the I/O status on the spot when it comes in. (ONLY applies if **Value type="user"** is set in the GATT Editor for the given characteristic)

* `sl_bt_gatt_server_send_user_read_response` **command**
  * Used to send back the desired data when a sl_bt_evt_gatt_server_user_read_request is generated by a client's request on a "user" characteristic. This command takes a result code and the actual value, if any, to send back. (ONLY applies if **Value type="user"** is set in the GATT editor for the given characteristic)

### Write

This operation is **requested by the GATT client** on a specific attribute exposed by the GATT server, and a new value to write is provided at the same time. The server then stores the new value and (optionally) acknowledges the write operation back to the client. In the BLE stack, these API methods are typically involved in write operations:

* `sl_bt_gatt_write_characteristic_value` **command**
  * Writes a remote attribute's value on a GATT server. This performs an **acknowledged** write operation, so the server will respond when the value has been successfully written. If the given value does not fit in one ATT PDU, "write long" GATT procedure is used automatically. **This is the most common method used by a GATT client to write individual attribute values.**

* `sl_bt_evt_gatt_procedure_completed` **event**
  * This event indicates that the current GATT procedure has been completed successfully or that it has failed with an error. All GATT commands excluding sl_bt_gatt_write_characteristic_value_without_response  and sl_bt_gatt_send_characteristic_confirmation will trigger this event, so the application must wait for this event before issuing another GATT command (excluding the two aforementioned exceptions). This event is useful for controlling program flow based on whether a critical operation has finished yet or not.

* `sl_bt_gatt_write_characteristic_value_without_response` **command**

  * Writes a remote attribute's value on a GATT server. This is exactly the same as the above command except that it is **unacknowledged**. There will be no response upon completion. Like the "notify" operation, this means that it is faster (multiple unacknowledged writes may be performed within a single connection interval), but less reliable than acknowledged writes.

* `sl_bt_gatt_prepare_characteristic_value_write` **command**
  * Can be used to add a characteristic value to the write queue of a remote GATT server. This command can be used in cases where very long attributes need to be written or a set of values needs to be written atomically. At most **245** (ATT_MTU - 5) amount of data can be sent once. The sl_bt_evt_gatt_procedure_completed event occurs when the preparation is done. In all cases where the amount of data to transfer fits into the BGAPI payload the command sl_bt_gatt_write_characteristic_value is recommended for writing long values since it transparently performs the "prepare_write" and "execute_write" commands.

The following is an example of the flow of queuing writes:

1. `sl_bt_gatt_prepare_characteristic_value_write`

2. Wait for `sl_bt_gatt_prepare_characteristic_value_write` response

3. Wait for `sl_bt_evt_gatt_procedure_completed` event

4. `sl_bt_gatt_prepare_characteristic_value_write`

5. Wait for `sl_bt_gatt_prepare_characteristic_value_write` response

6. Wait for `sl_bt_evt_gatt_procedure_completed` event

7. `sl_bt_gatt_execute_characteristic_value_write(conn, 1)`

8. Wait for `sl_bt_gatt_execute_characteristic_value_write` response

9. Wait for `sl_bt_evt_gatt_procedure_completed` event

* `sl_bt_gatt_execute_characteristic_value_write` **command**
  * Executes (flags = 1) or cancels (flags = 0) any pending write operations queued with `sl_bt_gatt_prepare_characteristic_value_write`. The stack handles sending the queued writes in multiple packets automatically when you run this command and it will take as many connection intervals as necessary. The `sl_bt_evt_gatt_procedure_completed` event occurs when the execution is done. On the server side, this looks like a rapid series of write operations and is handled the same way (acknowledging each one).

* `sl_bt_evt_gatt_server_attribute_value` **event**
  * Triggered on the **GATT server** when the client writes a new value to a particular characteristic. This is the event you can use to tell when a characteristic has been updated and then respond accordingly (parse the new value for command data, set I/O pins based on the stored data, close the connection, begin reading ADC measurements, and so on).

* `sl_bt_evt_gatt_server_user_write_request` **event**
  * If type="user" is set for a given characteristic, this event will be generated instead of `sl_bt_evt_gatt_server_attribute_value`. The new written value from the client will be provided as one of the parameters of this event, but the value itself will not be stored in RAM automatically by the stack. The user needs to store or otherwise process the value and acknowledge as necessary. Parameter att_opcode describes which GATT procedure was used to change the value.

* `sl_bt_gatt_server_send_user_write_response` **command**
  * Used to send back an appropriate acknowledgment result code when a `sl_bt_evt_gatt_server_user_write_request`  event is generated by a client's write operation on a "user" characteristic. If this manual acknowledgment is necessary, call this command in the same code as the event handler for the related `sl_bt_evt_gatt_server_user_write_request`  event. (ONLY applies if **Value type="user"** is set in the GATT editor for the given characteristic).

### Notify

This operation is **initiated by the server** when a new value is written to a notify-enabled characteristic. If the client has subscribed to notifications on that characteristic, the new value is pushed to the client when it is written. Notifications are not acknowledged, hence you may send more than one notification in a single connection interval, which can be helpful for [maximizing throughput](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-system-performance/throughput). Notifications can't be enabled by the server; they must be enabled by the client to ensure data transmission. In our BLE stack, these API methods are typically involved in notify operations:

* `sl_bt_gatt_server_write_attribute_value` **command**
  * Writes a local attribute's value on the GATT server. Writing the value of a characteristic of the local GATT database will not trigger notifications or indications to the remote GATT client in case such characteristic has property of indicate or notify and the client has enabled notification or indication. Notifications and indications are sent to the remote GATT client using sl_bt_gatt_server_send_characteristic_notification command.

* `sl_bt_evt_gatt_server_characteristic_status` **event**
  * This event indicates either that a local Client Characteristic Configuration descriptor has been changed by the remote GATT client or that a confirmation from the remote GATT client was received upon a successful reception of the indication. Status_flags parameter tells whether Client Characteristic Configuration was changed or if confirmation was received (0x02). Client_config_flags has the new values for the configuration. If the updated config flag equals **1**, notifications are enabled. If it equals **2**, indications are enabled. If it equals **0**, neither is enabled.

* `sl_bt_evt_gatt_characteristic_value` **event**
  * This event indicates that the value of a characteristic in the remote GATT server was received. The event is triggered as a result of several commands e.g., `sl_bt_gatt_read_characteristic_value` and `sl_bt_gatt_read_multiple_characteristic_values` and when the remote GATT server sends indications or notifications after enabling notifications with `sl_bt_gatt_set_characteristic_notification`. The parameter att_opcode reveals which type of GATT transaction triggered this event. In particular, if the att_opcode type is sl_bt_gatt_handle_value_indication  (0x1d), the application needs to confirm the indication with `sl_bt_gatt_send_characteristic_confirmation`. **This event enables the client to know it has received an updated value.** The pushed data will be made available as part of the event's parameters.

### Indicate

An **indicate** operation is identical to a **notify** operation except that **indications are acknowledged, while notifications are not**. This increases reliability at the expense of speed. In our BLE stack, these API methods are typically involved in **indicate** operations:

* `sl_bt_gatt_server_write_attribute_value` **command**
  * Writes a local attribute's value on the GATT server. Writing the value of a characteristic of the local GATT database will not trigger notifications or indications to the remote GATT client if the characteristic has property of indicate or notify and the client has enabled notification or indication. Notifications and indications are sent to the remote GATT client using `sl_bt_gatt_server_send_characteristic_notification` command. Remember that **indications are acknowledged**, so you may do this only once within a given connection interval. It will be followed up by the `sl_bt_evt_gatt_server_characteristic_status` event with status_flags = 0x2 when the acknowledgment comes back from the GATT client.

* `sl_bt_evt_gatt_server_characteristic_status` **event**
  * This event indicates either that a local Client Characteristic Configuration descriptor has been changed by the remote GATT client, or that a confirmation from the remote GATT client was received upon a successful reception of the indication. Status_flags parameter tells whether the Client Characteristic Configuration was changed or if confirmation was received (0x02). Client_config_flags has the new values for the configuration. If the updated config flag equals **1**, notifications are enabled. If it equals **2**, indications are enabled. If it equals **0**, neither are enabled.

* `sl_bt_evt_gatt_characteristic_value` **event**
  * This event indicates that the value of a characteristic in the remote GATT server was received. This event is triggered as a result of several commands e.g., `sl_bt_gatt_read_characteristic_value`; and when the remote GATT server sends indications or notifications after enabling notifications with `gatt_set_characteristic_notification` . The parameter att_opcode reveals which type of GATT transaction triggered this event. In particular, if the att_opcode type is handle_value_indication (0x1d), the application needs to confirm the indication with `sl_bt_gatt_send_characteristic_confirmation`. **This event is how the client knows it has received an updated value**. The pushed data will be made available as part of the event's parameters.

* `sl_bt_gatt_send_characteristic_confirmation` **command**
  * Sends an indication confirmation from the GATT client back to the remote GATT server, after an indicated value has arrived at the client. The `sl_bt_evt_gatt_characteristic_value` carries the att_opcode containing handle_value_indication (0x1d), which reveals that an indication has been received and this must be confirmed with this command. Confirmation needs to be sent within 30 seconds, otherwise the GATT transactions between the client and the server are discontinued. Sometimes there are specific data flow control triggers you need to deal with very carefully. Controlling precisely when the confirmation occurs and what happens before and after it may be valuable.

* `sl_bt_evt_gatt_server_characteristic_status` **event**
  * This event indicates either that a local Client Characteristic Configuration descriptor has been changed by the remote GATT client or that a confirmation from the remote GATT client was received upon a successful reception of the indication. Very often, this event is ignored, but as mentioned in the previous item, sometimes it is valuable to know precisely when this occurs for data flow control purposes.

Typically, the GATT server functionality is provided by one device and the client functionality by a different device, but both devices can also provide both kinds of functionality. This does not usually pose any advantages for a well-designed and efficient BLE project. On the contrary, it usually complicates the implementation needlessly and is therefore not discussed here.
