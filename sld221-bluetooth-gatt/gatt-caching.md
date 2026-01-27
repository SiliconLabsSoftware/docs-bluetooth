# GATT Caching

## Introduction

Discovering the GATT database of a remote device every time a connection is made is time and energy intensive. To resolve this, most Bluetooth devices use attribute caching, i.e., after they discover the GATT database of a remote device, they store the discovered attribute handles for future use. In other words, they store a local copy of the remote database structure, or at least parts of it. This works well as long as the remote device keeps its GATT database structure unchanged. If the database structure changes, for example because of a firmware update that introduces new features, the stored attribute handles become invalid.

These are the two methods to notify peer devices about the change:

- Using Service Change Indication
- Using GATT Caching feature

Service Change Indication was introduced in Bluetooth 4.0. This solution is discussed in [Service Change Indications](./service-change-indication). The main disadvantage of this method is that requires peer devices to be bonded.

GATT caching feature was introduced in Bluetooth 5.1 to overcome the bonding problem. Using GATT caching, every device can make sure that it is storing the latest version of the GATT database structure. This is achieved by:

- Adding two new characteristics to the **Generic Attribute** service (**Database Hash** and **Client Supported Features**)
- Adding a new GATT error code (**database out-of-sync**)

## Database Hash Characteristic

The **Database Hash** characteristic stores a 128 bit value, which is a AES-CMAC hash calculated from the database structure. Any change in the database structure results in a different hash value. After discovering the database once, the client should store this value. Next time, when the client connects to the server, it should only check if the **Database Hash** has changed. If it has not, it is safe to assume that the database structure is unchanged and the cached attribute handles are still valid. If it has changed, the client needs to rediscover the GATT database.

> Note: The Database Hash must be read using GATT Read Using Characteristic UUID sub-procedure `sl_bt_gatt_read_characteristic_value_by_uuid()`.

## Database Out-of-Sync Error Code

If the client enables **Robust Caching** by setting the *Robust Caching* bit to 1 in the **Client Supported Features** characteristic, the server can send a **database out-of-sync error** code as a response to a GATT operation whenever it assumes that the client has an out-of-date database.

If the client and the server are *bonded*, the server checks if the database has changed since the last connection and notifies the client about the change via **Service Change Indication** - just like without the usage of GATT caching. The new feature is that if the client initiates a GATT operation *before* the indication is received and confirmed, the server can send a **database out-of-sync** error code. This helps avoid a race condition if the client wants to initiate a GATT operation too quickly after establishing a connection.

If the client and the server are *not bonded*, the server assumes that the client checks the **Database Hash** and/or discovers the GATT database upon each connection before initiating any GATT operation. In other words, the server assumes that the client is aware of database changes after the connection was established. If a change occurs in the database during connection, the client gets change-unaware. At the next GATT operation, the server sends a **database out-of-sync** error code to the client to signal this (except if the client reads the **Database Hash** meanwhile). From the next GATT operation, the server assumes that the client is change-aware again.

## Implementation

### Server

To enable GATT caching functionality on the server side, enable Generic Attribute Service in the GATT database as follows:

1. Open the *Bluetooth GATT Configurator* in your project.

2. Click on the Profile (by default it is *Custom BLE GATT*) in the GATT database structure.

3. On the right side, enable the *Generic Attribute Service* using the slider. You will not see the service appearing in the GATT database of the editor but it will be added to the generated gatt_db.c, i.e., it will be part of the actual GATT database.

   ![Generic Attribute](resources/generic-attribute.png?darkModeUrl=resources/generic-attribute.png)

The stack will now automatically populate the Database Hash characteristic with the database hash.

### Client

To enable GATT caching on the client side, do the following:

1. Read the *Database Hash* characteristic upon each connection and compare its value with the last stored value:

    ```C
    static uint8_t db_hash_uuid[2] = {0x2a,0x2b};
    sl_bt_gatt_read_characteristic_value_by_uuid(conn_handle, 0x0001FFFF, sizeof(db_hash_uuid), &db_hash_uuid[0]);
    ```

    If the values match, you are safe to use your stored characteristic handles. If they do not match, start a service discovery to discover characteristic handles.

2. Enable Robust Caching by writing to the *Client Supported Features* characteristic of the server:

    ```C
    static uint8_t enable_bit[1] = {0x01};
    sl_bt_gatt_write_characteristic_value(conn_handle, client_supported_features_handle, sizeof(enable_bit), &enable_bit[0]);
    ```

3. Check for *database out-of-sync* error after each GATT operation.

    ```C
    if (evt->data.evt_gatt_procedure_completed.result == SL_STATUS_BT_ATT_OUT_OF_SYNC)
     printLog("database has changed!\r\n");
    ```
