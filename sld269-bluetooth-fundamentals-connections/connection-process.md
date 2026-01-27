# Understanding the Bluetooth Connection Process

## Introduction

Bluetooth ensures reliable data transfers for connected devices. A connection is required for secure data transfer. This document describes the various states that a Bluetooth device can be in and how to move between these states.

## Bluetooth States

After starting the Bluetooth stack, the device will be in an idle state. In other words, it will be non-discoverable and non-connectable. Through a call to the API function `sl_bt_advertiser_start`, the device can be made *discoverable* and *non-connectable* or *discoverable* and *connectable*. It is also possible to return the device to the idle, *non-discoverable* and *non-connectable* state by `sl_bt_advertiser_stop`.

### Non-Connectable Beacons

A device which is discoverable but non-connectable is known as a beacon. The advertising data can be seen by any device within range but it is not possible to establish a connection. This means that the advertising device’s data cannot be written. iBeacon and Eddystone standard are beacon examples. If a remote device attempts to connect to a non-connectable advertiser, the request will be dropped. No interaction is required by the user application.

### Connectable Advertisers

A device which is discoverable and connectable advertises and accepts connections from any device within range. When a connection has been established, the stack sends the event `sl_bt_evt_connection_opened` to the application. This event contains the address of the remote device, the type of address, a connection handle, the role of the device in the connection, and a bond handle to indicate whether the device is bonded or not. The event also includes a handle to indicate which advertising set the connection is associated with.

When a connection request is received, the Bluetooth stack automatically stops advertising to avoid unintended multiple connections. If multiple connections are required, advertising can be restarted after the `sl_bt_evt_connection_opened` event was received.

### Closing Connections

If a connection is closed, the event `sl_bt_evt_connection_closed`  will be sent to the application. This event includes the connection handle and the reason for disconnection. The reasons for disconnections are documented in the Bluetooth errors section of the API Reference Manual.

## Secure and Unsecure Connections

When a connection event is received(`sl_bt_evt_connection_opened`), the application can determine whether or not a bond was made with the remote device by examining the bond_handle parameter. A value of 0xFF indicates no bond, while any other value indicates a valid bond. If the local and remote devices are not bonded, the communication between them will be unencrypted and visible to any Bluetooth device within range. It is strongly recommended to secure any sensitive data.

After the connection event, at least one connection parameters event (`sl_bt_evt_connection_parameters`) will occur. This event is sent when a connection is opened and any time the connection parameters are updated. The connection parameters event includes information about the connection parameters (connection interval, latency, timeout) as well as the security mode and maximum PDU size. These are the types of security modes:

1. No security
2. No authentication, but encrypted
3. Authenticated and encrypted

Either stack of the user application can request secure connection. The stack will request a secure connection if the remote device attempts to access a protected characteristic. The user application can request a secure connection by calling `sl_bt_sm_increase_security()`. In either case, an event will be sent by the stack to the user application to indicate whether the bonding/pairing was successful (`sl_bt_evt_sm_bonded`) or unsuccessful (`sl_bt_evt_sm_bonding_failed`).

### Bonding vs Pairing

The security manager contains events and commands for controlling the security features included in the Bluetooth stack. One of these features is the ability to form new bonds (bondable mode). As shown in the diagram below, when a connection is secured, it will either be bonded and assigned a long term key (LTK), which can be used in subsequent connections or paired and assigned a short term key (STK), which will be discarded when the connection is terminated. Upon successful bonding/pairing the stack sends the event `sl_bt_evt_sm_bonded` to the application with a bond_handle as a parameter. As with the bond_handle passed to the `sl_bt_evt_connection_opened`, any value other than 0xFF indicates that the devices are bonded while a value 0xFF means that the devices are paired for the current connection.

![Bluetooth 4.x Connected State](resources/bt4-connected-state.png?darkModeUrl=resources/bt4-connected-state.png)

### Maximum Transmission Unit (MTU)

In addition to the connection opened event and the connection parameters event, a GATT MTU event is exchanged for each connection. This event tells you the size of the maximum transmission unit (MTU), which is the maximum size of any packet that can be sent between the client and server. The only special handling that may be required for this event is to use the MTU to determine whether an entire characteristic can be sent in a single read/write or if multiple writes are required. A single read/write  can be MTU – 3 bytes in length.

### Bluetooth 5 Connections

Whether a connection is secured or not, Bluetooth 5 allows the choice between 1 Mbps, 2 Mbps or Coded (125 kbps/500 kbps) PHY on a per-connection basis. PHY can be selected by calling `sl_bt_connection_set_preferred_phy`, which results in stack sending the event `sl_bt_evt_connection_phy_status` to indicate which PHY is used for the connection. The diagram below shows that the flow of the connected state is similar to that of Bluetooth 4.x except that you can now also select 2M PHY.

![Bluetooth 5 Connected State](resources/bt5-connected-state.png?darkModeUrl=resources/bt5-connected-state.png)

### Multiple Connections and Dual Mode Topology

Up to 32 simultaneous connections can be allowed. The *Maximum number of connections* parameter in the configuration of the Bluetooth Core software component can be used to limit the number of connections to less than 32 to save RAM. By default four connections are enabled.

One of the additions made in Bluetooth 4.2 is the so-called dual mode topology, which allows a device to be in central and peripheral roles simultaneously. Previously, you had to disconnect to switch roles between central and peripheral.

## Conclusion

Bluetooth connections ensure reliable and, optionally, secure data transfers. Additionally, by using connections, devices can negotiate PHY that they use to communicate.
