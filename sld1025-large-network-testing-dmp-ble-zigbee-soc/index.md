# Large Network Performance with Dynamic Multiprotocol Bluetooth LE and Zigbee System-on-Chips

> **Note: This section replaces *AN1425: Large Network Performance with Dynamic Multiprotocol Bluetooth LE and Zigbee System-on-Chips*. Further updates to this application note will be provided here**.

This application note summarizes the results of Zigbee/BLE dynamic multiprotocol (DMP) large network performance tests using Zigbee unicast and broadcast packets to measure the reliability, latency, and loss of a variety of scenarios. The system-on-chip test nodes are running dynamic multiprotocol software combining Zigbee with Bluetooth LE (BLE) advertising.

## Introduction

The tests and results provided are used as quality benchmarks for large network testing with DMP BLE and Zigbee. Two different scenarios were tested.

### Zigbee/BLE DMP Large Network Broadcast

The Central Node is a Zigbee coordinator (high RAM concentrator) that forms a centralized security network. The Zigbee coordinator sends out a many-to-one route request (MTORR) at a default randomized period of 60 second minimum to 300 seconds maximum. All other nodes are Zigbee router device types. All devices use a DMP stack and send out a connectable BLE advertisement with max payload once every 152.5 ms when advertising is enabled.

The purpose of this scenario is to compare Zigbee broadcast performance for when all devices are sending BLE advertisements and when they are not.

### Zigbee/BLE DMP Large Network with High Stress BLE Data Traffic

This is a multifaceted scenario that combines the test above with unicast traffic and BLE connections and high-stress BLE data traffic for 3 pairs of devices.

The purpose of this scenario is to simulate real-life usage scenarios of DMP nodes and ensure that the network remained stable.
