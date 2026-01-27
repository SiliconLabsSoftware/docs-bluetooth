# Selecting Suitable Connection Parameters for Apple Devices

## Introduction

For each Bluetooth connection, a set of parameters can be changed on the fly, depending on the application requirements. These include: **connection interval,** **peripheral latency**, and **supervision timeout**. For example, to minimize power consumption, you can increase the connection interval and peripheral latency. This reduces the RF duty cycle and allows the peripheral to stay longer in sleep mode if there is no need to transmit any user data.

For more details about Bluetooth connections, see [Bluetooth LE Fundamentals](https://docs.silabs.com/bluetooth/latest/bluetooth-le-fundamentals/).

The central device (typically a mobile device) controls which connection parameters are used. When the central device initiates a connection to a peripheral, it selects the default parameters that are used. The peripheral can ***request*** updating the connection parameters. However, the central can either accept or reject this request.

Depending on the central, different restrictions can apply on which parameter combinations are accepted. This document focuses on the recommendations for iOS devices specified by Apple.

## Apple Bluetooth Device Guidelines

The rules for selecting connection parameters for BLE peripherals that are defined in the document **Accessory Design Guidelines for Apple Devices** are available for download [here](https://developer.apple.com/accessories/Accessory-Design-Guidelines.pdf).

The BLE connection parameters are discussed in section **49.6 Connection Parameters**. For example, the minimum connection interval is specified as 15 ms. Note that the minimum interval allowed by the Bluetooth core specification is 7.5 ms.

At the time of writing, the latest version of the guideline document was R21 (dated 2023-10-12).
