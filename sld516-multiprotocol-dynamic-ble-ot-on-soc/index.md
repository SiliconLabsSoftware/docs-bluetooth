# Dynamic Multiprotocol Development with Bluetooth and OpenThread on SoCs in GSDK v3.x and Higher

> **Note: This section replaces *AN1265: Dynamic Multiprotocol Development with Bluetooth and OpenThread on SoCs in GSDK v3.x and Higher*. Further updates to this application note will be provided here**.

These pages provide instructions on getting started with dynamic multiprotocol (DMP) applications using Silicon Labs OpenThread and Bluetooth running over the FreeRTOS real time operating system.

The sample application **ot-ble-dmp** is a test application that demonstrates the components that go into building a DMP application. It provides a command line interface (CLI) that allows the user to execute basic OpenThread and Bluetooth commands. It also demonstrates how the power manager component can be used to save power by allowing the device to enter low power (EM2) mode in between activities.

The term 'dynamic' in DMP refers to the fact that both protocols are operating simultaneously. The radio scheduler takes care of multiplexing the transmitted and received packets over the radio. For more information on how the radio scheduler works, see the [Dynamic Multiprotocol User’s Guide](https://docs.silabs.com/connect/latest/multiprotocol-dynamic-ug/).

The instructions assume that you have installed Simplicity Studio 5 (SSv5) and the OpenThread and Bluetooth SDKs, and that you are familiar with SSv5 and configuring, building, and flashing applications. If not, see [Silicon Labs OpenThread Quick Start Guide](https://docs.silabs.com/openthread/latest/openthread-quick-start-guide/).

## Hardware Requirements

- An EFR32 part with at least 512 kB of flash.
