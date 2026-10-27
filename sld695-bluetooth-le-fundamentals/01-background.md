# Background

Silicon Labs is developing products designed to meet the demands of customers as we move to an ever-connected world of devices in the home, what is often referred to as the IoT (Internet of Things). At a high level, the goals of IoT for Silicon Labs are to:

- Connect all the devices in the home with best-in-class networking, whether with Zigbee PRO, Thread, Bluetooth low energy technology, or other emerging standards.

- Leverage the company’s expertise in energy-friendly microcontrollers.

- Enhance established low-power, mixed-signal chips.

- Provide low-cost bridging to existing Ethernet and Wi-Fi devices.

- Enable cloud services and connectivity to smartphones and tablets that promote ease of use and a common user experience for customers.

Achieving all of these goals will increase adoption rates and user acceptance for IoT devices in the Connected Home.

Bluetooth technology is a core component of the IoT. Bluetooth was designed to offer a wireless alternative to cable connections by exchanging data using radio transmissions. One of the most popular applications for Bluetooth has been wireless audio. This uses a version of Bluetooth called BR/EDR (Bit Rate/Enhanced Data Rate) that is optimized for sending a steady stream of high quality data in a power-efficient way.

Bluetooth version 4.0 introduced Bluetooth with low energy functionality. Developers are now able to create sensors that can run on coin-cell batteries for months and even years. Some of these sensors are so efficient that the kinetic energy from flipping a switch can provide operating power. Bluetooth low energy technology is inherently different from BR/EDR. BR/EDR establishes a relatively short-range, continuous wireless connection, which makes it ideal for uses such as streaming audio from a smartphone to a headset. Bluetooth low energy technology allows for short bursts of long-range radio connections, making it ideal for IoT applications that depend on long battery life. Furthermore, Bluetooth low energy technology is built on an entirely new development framework using GATT (Generic Attributes). GATT profiles describes a use case, roles, and general behaviors based on the GATT functionality. These profiles allow developers to quickly and easily develop applications to connect devices directly to applications running on smartphones, PCs, or tablets.

Bluetooth devices can be either dual mode, supporting both BR/EDR and Bluetooth low energy technology, or single mode, supporting Bluetooth low energy technology only.

As well as ultra-low power and connectivity to smartphones, PCs, and tablets, other benefits of Bluetooth low energy technology include:

- Low cost

- Reliable and robust: AFH (Adaptive Frequency Hopping), retransmissions and 24-bit CRC (Cyclic Redundancy Checks)

- Secure: pairing, bonding, privacy, MITM (Man in the Middle) protection, and AES-128 encryption

- Supports rapid development:

  - Standardized profiles to cover key use cases (HR, HID, Glucose, Proximity, etc.)

  - Profiles can be developed as applications, supporting fast deployment

  - Vendor-specific profiles omit the need to wait for Bluetooth SIG to standardize profiles and operating system developers to integrate them

- Widely deployable: Supported by major platforms - iOS, Android 4.3 and newer, Windows 8 and 10, OSX, and Linux

The Bluetooth specification is managed by the Bluetooth SIG (special interest group). The SIG maintains a website ([https://www.bluetooth.com](https://www.bluetooth.com)) that contains both introductory information and links to specifications and other more technical details. In this guide, revisions of the specification are referred to parenthetically, where (BT5.0) means version 5.0 of the specification.

This guide provides an overview of the following aspects of Bluetooth low energy:

- Bluetooth architecture overview

- Radio features

- Basic of link layer

- Explanation of how device discovery and connections work

- Bluetooth security overview

- The Attribute Protocol

- The Generic Attribute Profile (GATT) and Bluetooth profiles
