# Dynamic Multiprotocol Development with Bluetooth and Zigbee EmberZNet SDK 7.0 and Higher

> **Note: This section replaces *AN1322: Dynamic Multiprotocol Development with Bluetooth and Zigbee EmberZNet SDK 7.0 and Higher*. Further updates to this application note will be provided here**.

This application note provides details on developing Dynamic Multiprotocol applications using Bluetooth and Zigbee in SDK 4.0 and higher. It describes how to configure applications in Simplicity Studio using Zigbee EmberZNet SDK v. 7.0 and higher. It then provides a detailed walkthrough on how the underlying code functions. For details on Dynamic Multiprotocol Application development that apply to all protocol combinations see the [Dynamic Multiprotocol User’s Guide](https://docs.silabs.com/connect/latest/multiprotocol-dynamic-ug/).

Zigbee EmberZNet SDK v7.0 introduced a component-based project architecture that replaced AppBuilder. If you are working with Zigbee EmberZNet SDK v 6.10.x or lower, see *AN1133: Dynamic Multiprotocol Developer with Bluetooth and Zigbee EmberZNet SDK 6.x and Lower*  for this information.

The example applications referenced here can be controlled either from a protocol-specific switch application or from a Bluetooth-enabled smartphone app. This application note provides details on how these examples are designed and implemented. It also describes how to generate, compile, and load example application code, and how to add dynamic multiprotocol functionality to an existing Zigbee project. The application note is intended to be used when developing your own Zigbee/Bluetooth dynamic multiprotocol implementations.

>**Note**: The Zigbee dynamic multiprotocol solution is currently only supported for SoC architectures. Support for NCP architectures has been deprecated in favor of DMP RCP. Please contact Silicon Labs Sales for more information on our multiprotocol software roadmap.

## Resources

- [Dynamic Multiprotocol User's Guide](https://docs.silabs.com/connect/latest/multiprotocol-dynamic-ug/) provides details on:

- Dynamic Multiprotocol Architecture

- Radio Scheduler operation (with examples)

- Task Priority management

- [Using Third Generation Non-Volatile Memory (NVM3) Data Storage](https://docs.silabs.com/gecko-platform/latest/using-third-generation-nonvolatile-memory/) explains how NVM3 can be used as non-volatile data storage in Dynamic Multiprotocol applications with Zigbee and Bluetooth.

## Development Environment Requirements

- Simplicity Studio 5

- SDK 4.0 or higher, which includes Zigbee EmberZNet SDK version 7.0.0 or higher and Bluetooth SDK 3.3 or higher.

- An EFR32 chip with at least 512 kB of flash (required to run all the necessary software components)

To work with the demos, download the Simplicity Connect app from Google Play Store or App Store.
