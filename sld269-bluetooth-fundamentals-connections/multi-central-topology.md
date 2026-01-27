
# Multi-Central Topology

## Introduction

Since Bluetooth 4.1, a peripheral device can connect to multiple centrals. Silicon Labs Bluetooth stack supports connection to multiple centrals simultaneously, with a maximum number of 32 centrals. This document shows how to handle multiple connections in your Bluetooth application.

![Multi-Central Topology](resources/multi-central.png?darkModeUrl=resources/multi-central.png)

## Establishing Multiple Connections

Connections are initiated by central devices. For example, when a smartphone connects to your device, the smartphone will be the central and your device will be the peripheral.

To make connections possible, the peripheral device has to advertise itself with a *connectable* advertisement. A connectable advertisement can be started with the following command:

```c
sl_bt_advertiser_start(
        advertising_set_handle,
        advertiser_general_discoverable,
        advertiser_connectable_scannable);
```

Note, that an advertisement set must be created with *sl_bt_advertiser_create_set()* before an advertisement can be started. See the detailed description in the API documentation.

After the advertising is started, a central device can discover the peripheral and it can send a connection request to it. **When a connection request is received, the advertising is immediately stopped by the Bluetooth stack** to avoid unintended multiple connections and the connection is opened, which is signaled by an *sl_bt_evt_connection_opened* event. To enable connections for multiple centrals, re-start advertising upon receiving this event using the same command as before.

> It is worth keeping count of the live connections and not re-starting advertising after the limit of maximum number of connections is reached.

By default, four simultaneous connections are enabled in the stack. To increase the number of supported connections, go to the configuration of the **Bluetooth Core** software component and increase the *Maximum number of connections*:

![Bluetooth Core Component](resources/bluetooth-core.png?darkModeUrl=resources/bluetooth-core.png)

![Configuring the Maximum Number of Connections](resources/max-num-conn.png?darkModeUrl=resources/max-num-conn.png)

## Saving Connection Handles

When a connection is opened, the stack returns a connection handle. Later, this handle can be used to differentiate between simultaneous connections. All connection-related commands require a connection handle and all connection-related events provide a connection handle. As a result, save the connection handles for future use, e.g., in a global variable.

```c
struct {
    bd_addr device_address;
    uint8_t address_type;
    uint8_t connection_handle;
} connections[8];
uint8_t live_connections = 0;

//...

  case sl_bt_evt_connection_opened_id:

    connections[live_connections].device_address = evt->data.evt_connection_opened.address;
    connections[live_connections].address_type = evt->data.evt_connection_opened.address_type;
    connections[live_connections].connection_handle = evt->data.evt_connection_opened.connection;
    live_connections++;

  break;
```
