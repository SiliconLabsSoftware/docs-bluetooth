# Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL in GSDK v3.x and Higher

> **Note: This section replaces *AN1269: Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL in GSDK v3.x and Higher*. Further updates to this application note will be provided here**.

These pages provide details on how to develop a multiprotocol application running Bluetooth and a proprietary protocol at the same time, using SDKs from Gecko SDK Suite v3.x. First, the criteria for the coexistence of Bluetooth and a proprietary protocol are discussed. Then the application note guides you through how to create a new DMP application, how to configure Bluetooth and your proprietary protocol, and how to transmit and receive proprietary packets while Bluetooth is running. Finally, a Light/Switch DMP example is introduced in more details.

For background on Dynamic Multiprotocol Application development in general, and about Bluetooth task priorities and scheduling, see the [Dynamic Multiprotocol User's Guide](https://docs.silabs.com/connect/latest/multiprotocol-dynamic-ug/). It provides information about the Dynamic Multiprotocol solution, where two protocols are running on the same device in parallel, and includes general background as well as information on Bluetooth task priorities and time scheduling. You should be familiar with the principles of Dynamic Multiprotocol and with all the terms related to it. The Dynamic Multiprotocol projects require an RTOS for scheduling. Currently, the Micrium RTOS and the FreeRTOS are supported. The FreeRTOS is included in the SDK.

## Requirements

To be able to use all the features discussed in this document, you will need the following installed on your computer:

- Bluetooth SDK version 3.0.0 or higher

- (optional*) Micrium OS-5 kernel

To be able to run the Light/Switch example, you will need the following installed on your computer:

- Bluetooth SDK version 3.0.0 or higher

- Flex SDK version 3.0.0 or higher

- (optional*) Micrium OS-5 kernel

- (optional*) IAR Embedded Workbench for ARM (IAR-EWARM) (required for the Flex (RAIL) - Switch application). See the release notes for the Bluetooth SDK for the required IAR-EWARM version.

*Required only when the Micrium RTOS is used.
