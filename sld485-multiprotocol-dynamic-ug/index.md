# Dynamic Multiprotocol User's Guide

> **Note: This section replaces *UG305: Dynamic Multiprotocol User's Guide*. Further updates to user guide will be provided here**.

Silicon Labs software is designed to be used by multiple protocols on a single wireless chip. This user's guide provides details about implementing an application using Silicon Labs' Dynamic Multiprotocol solution. Dynamic multiprotocol time-slices the radio and rapidly changes configurations to enable different wireless protocols to operate reliably at the same time.

>**Note**: The Zigbee-specific information in this document applies to version 6.10.x and lower.

Details on specific dynamic multiprotocol implementations are provided in the following application notes:

- [Dynamic Multiprotocol Development with Bluetooth and Zigbee EmberZNet SDK 7.x and Higher](https://docs.silabs.com/multiprotocol/latest/dynamic-multiprotocol-bluetooth-zigbee-sdk-7x-higher/)

- [AN1134: Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL in GSDK v2.x](https://www.silabs.com/documents/public/application-notes/an1134-bluetooth-rail-dynamic-multiprotocol.pdf)

- [Dynamic Multiprotocol Development with Bluetooth® and Proprietary Protocols on RAIL in GSDK v3.x and Higher](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ble-proprietary-on-rail/)

- [AN1209: Dynamic Multiprotocol Development with Bluetooth and Connect](https://www.silabs.com/documents/public/application-notes/an1209-dynamic-multiprotocol-connect-bluetooth.pdf)

- [Dynamic Multiprotocol Development with Bluetooth® and OpenThread in GSDK v3.x](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ble-ot-on-soc/)

## Terminology

The following lists some of the terminology specific to the dynamic multiprotocol implementation.

**Radio Abstraction Interface Layer (RAIL)**: The common API through which higher level code gains access to the EFR32 radio.

**Radio Operation**: A specific action to be scheduled. A radio operation has both a radio configuration and a priority. Each stack can request that the radio scheduler perform up to two radio operations (background receive and either Scheduled Receive or Scheduled transmit) at a time:

- **Background Receive**: Persistent receive, intended to be interrupted by Scheduled operations, and returned to after their completion.

- **Scheduled Receive**: Receive packets or calculate RSSI at a specified time and duration. (Developers working on RAIL, note that in terms of the RAIL API, “Scheduled Receive” as used in this document refers to any receive operation, other than `RAIL_StartRx`, and is not just limited in scope to `RAIL_ScheduleRx`.)

- **Scheduled Transmit**: Any one of various transmit operations including immediate transmit, scheduled (future) transmit, or CCA-dependent transmit. (Developers working on RAIL, note that in terms of the RAIL API, “Scheduled Transmit” as used in this document refers to any transmit operation, and is not limited in scope to `RAIL_StartScheduledTx`.)

**Radio Config**: Determines the state of the hardware that must be used to perform a radio operation.

**Radio Scheduler**: RAIL component that arbitrates between different protocols to determine which will have access to the radio.

**Priority**: Each operation from each stack has a default priority. An application can change default priorities.

**Slip Time**: The maximum time in the future when the operation can be started if it cannot begin at the requested start time.

**Yield**: A stack must voluntarily yield at the end of an operation or sequence of operations, unless it is performing a background receive. Until the stack yields, the scheduler will not scheduler lower priority tasks.

**RTOS (Real Time Operating System) Kernel**: The part of the operating system that is responsible for task management, and inter-task communication and synchronization.

## Architecture

Dynamic Multiprotocol makes use of the EFR32 hardware and the RAIL software as its building blocks. Zigbee, Bluetooth, and/or any other standards-based or proprietary protocols can then be built on top of these foundational layers, using Micrium to manage execution of code between different protocols. The following diagram illustrates the general structure of the software modules.

![General Dynamic Multiprotocol Software Architecture](resources/sld485-image8.png)

Beginning with version 2.0, RAIL requires the passing of a radio configuration handle to the RAIL API calls. This configuration describes various PHY parameters that are used by the stack.

The OS is an RTOS that allows stacks and application logic to share CPU execution time.

The Radio scheduler is a software library that intelligently answers requests by the stacks to perform radio operations to maximize reliability and minimize latency. API’s provided by RAIL that do not engage the radio bypass the Radio scheduler.

The RAIL core configures the EFR32 hardware in response to instructions from the radio scheduler.

## Single Firmware Image

Dynamic Multiprotocol allows a software developer to generate a single monolithic binary that is loaded onto an EFR32. Software updates are done by upgrading the entire binary. This is accomplished using the Gecko Bootloader, the details of which can be found in *UG266: Silicon Labs Gecko Bootloader User’s Guide for GSDK 3.2 and Lower*, [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) or [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-series3-and-higher/).

## Independent Stack Operation

The Silicon Labs stacks still operate independently of one another in a Dynamic Multiprotocol situation. Certain long-lived radio operations will have an impact on another protocol’s latency and compliant operation. It is up to the application to determine any special considerations for these events. See [The Radio Scheduler](./02-the-radio-scheduler.md) for more information.
