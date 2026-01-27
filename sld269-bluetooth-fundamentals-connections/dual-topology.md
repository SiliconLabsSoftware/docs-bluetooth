
# Dual-Topology

## Introduction

Bluetooth Dual Topology means that a Bluetooth device can act as a central on one connection, while acting as a peripheral on another connection. Silicon Labs Bluetooth stack allows connection to multiple peripherals and centrals simultaneously, with a maximum number of 32 connections. This document shows how to handle multiple connections in your Bluetooth application, if you are expecting to be peripheral on some connections and central on others.

![Bluetooth Dual Topology](resources/sld269-image1.png?darkModeUrl=resources/sld269-image1.png)

## Establishing Multiple Connections

To create a Bluetooth connection, one of the devices has to be advertising and another one has to scan for advertisements and send a connection request. The advertiser device will become the peripheral on the connection and the scanner device will become the central.

To enable both central and peripheral functionality, the device has to act both as an advertiser and as a scanner after initialization.  A connectable advertisement can be started with *sl_bt_advertiser_start*, while scanning can be started with *sl_bt_scanner_start*. Silicon Labs' Bluetooth stack can handle scanning and advertising at the same time, however, because it needs time multiplexing, it is worth decreasing the duty cycle of the scanning from the default 100%. To decrease the duty cycle, set the scan_window shorter than the scan_interval with *sl_bt_scanner_set_timing*.

After the scanner finds an advertisement, an *sl_bt_evt_scanner_scan_report* event is generated and the scanner can decide (based on advertisement data) if it wants to connect to this device with *sl_bt_connection_open*.

![Flowchart for Simultaneous Scanning and Advertising](resources/sld269-image2.png?darkModeUrl=resources/sld269-image2.png)

When a connection is opened, an *sl_bt_evt_connection_opened* event is generated regardless of whether the connection was initiated by your device or by a remote device. To differentiate between these two use cases, the *sl_bt_evt_connection_opened* event provides a `master` parameter which is set to 1 if your device was the initiator and hence became the central, and which is set to 0 if the connection was initiated by a remote device and hence our device became the peripheral.

```c
      sl_bt_evt_connection_opened_id:

        printLog("connection opened (%s)\r\n",
                 evt->data.evt_connection_opened.master ? "central" : "peripheral");

        break;
```

To learn more about handling multiple connections, see[Multi-Peripheral Topology](./multi-peripheral-topology.md) and [Multi-Central Topology](./multi-central-topology.md).

By default, four simultaneous connections are enabled in the stack. To increase the number of supported connections, go to the configuration of the **Bluetooth Core** software component and increase the *Maximum number of connections*:

![Bluetooth Core Component](resources/bluetooth-core.png?darkModeUrl=resources/bluetooth-core.png)

![Configuring the Maximum Number of Connections](resources/max-num-conn.png?darkModeUrl=resources/max-num-conn.png)
