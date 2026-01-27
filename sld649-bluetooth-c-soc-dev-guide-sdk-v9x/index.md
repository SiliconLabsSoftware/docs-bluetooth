# Silicon Labs Bluetooth C Application Developer's Guide for SDK v10.x

**NOTE: This section replaces *UG434: Silicon Labs Bluetooth C Application Developer's Guide for SDK v9.x and Higher*. Further updates to this user guide will be provided here.**

This user guide is an essential reference for everybody developing C-based applications for the Silicon Labs Wireless Gecko products using the Silicon Labs Bluetooth stack. This guide covers the Bluetooth stack architecture, application development flow, usage, and limitations of the MCU core and peripherals, stack configuration options, and stack resource usage. This version applies to the Silicon Labs Bluetooth Software Development Kit (SDK) version 10.x.

The purpose of this guide is to capture and fill in the blanks between the Bluetooth Stack API reference, Simplicity SDK API reference, and Wireless Gecko reference manuals, when developing Bluetooth applications for the Wireless Geckos. This guide exposes details that will help developers make the most out of the available hardware resources.

The guide covers the following topics:

- [Application Development Flow](./02-application-development-flow) discusses the application development flow.

- [Project Structure](./03-project-structure) reviews project structure.

- [Configuring the Bluetooth Stack and a Wireless Gecko Device](./04-configuring-bluetooth-stack-and-wireless-gecko-device) explains the project include libraries and the actual Wireless Gecko configuration in the application code.

- [Bluetooth Stack Event Handling](./05-bluetooth-stack-event-handling) is an important piece for everyone developing with the Silicon Labs Bluetooth stack, as it explains how the application runs in sync with the stack in an event-based architecture.

- [Interrupts](./06-interrupts) and [Wireless Gecko Resources](./07-wireless-gecko-resources) touch on the topics of peripherals and the chipset resources, covering what is reserved for the stack usage, how interrupts should be handled, and the stack’s memory footprint and available memory for the application.

## About this Version

The current version of Silicon Labs' Bluetooth SDK is 10.x.

Currently supported compilers are:

- GCC 12.2.1
- IAR 9.40.1

## Prerequisites

This guide assumes the current version of Silicon Labs’ Bluetooth SDK has been properly installed to the development machine (Windows, MAC OSX, or Linux), and that the reader is familiar with the quick start guides and with the SDK’s examples. Also, the reader should have a basic understanding of Bluetooth technology. For more information, see [Bluetooth LE Fundamentals](/bluetooth/{build-docspace-version}/bluetooth-le-fundamentals).

For instructions on getting started using example applications in Silicon Labs Simplicity Studio development environment, see the [Bluetooth Getting Started Guide](/bluetooth/{build-docspace-version}/bluetooth-getting-started-overview).
